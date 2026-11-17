# Week 11 Homework — Make It Answerable

*Assigned the week-11 session · part 1 is a team PR: turn on telemetry in
your team repo · part 2 is solo: your own Application Insights, one
request followed end to end, one query you wrote, and one alert you've
seen fire · both merged before the week-12 session · your app does not
need to be deployed for any of it*

# tl;dr

Give every request in your app a tracking number. Then send the scans to
Azure, find one request there, ask one question of your own in a query,
and set one alert that you've watched go off. After this, "it's slow" is
a thing you look up.

## The tracking number

When you ship a package, you don't call the truck. You look up one number
and read the scans: picked up, arrived, out for delivery, and how long it
sat in each place.

Your app handles requests the same way, with nothing to look up. A page
is slow for somebody, and the only tools you have are a guess and a
`console.log` you'd have to deploy to read. So this week every request
gets a number (a trace id), every step it takes gets scanned (the query,
the log lines, how long each took), and the scans get kept somewhere you
can search.

## Part 1 · Turn it on (team, one PR)

One teammate does this in a PR of its own, and everybody branches from it.
The worked example is Callback's observability PR, which makes the same
changes. Three of these steps exist only because the build fails without
them, so don't skip the odd-looking ones.

**1 · Install.**

```bash
pnpm --filter @project/web add @microsoft/opentelemetry
```

pnpm will stop with `Ignored build scripts: protobufjs`, and it writes a
placeholder line into `pnpm-workspace.yaml`. Replace that line, under
`allowBuilds`, with a real answer, then finish the install:

```yaml
  "protobufjs": false
```

```bash
pnpm install
pnpm --filter @project/log add @opentelemetry/api @opentelemetry/api-logs
```

**2 · Two files in `apps/web/`.** Next calls `register()` once when the
server starts, before the first request.

```ts
// apps/web/instrumentation.ts
export async function register() {
  if (process.env.NEXT_RUNTIME === "nodejs") {
    await import("./instrumentation.node");
  }
}
```

```ts
// apps/web/instrumentation.node.ts
import { useMicrosoftOpenTelemetry } from "@microsoft/opentelemetry";

const connectionString = process.env.APPLICATIONINSIGHTS_CONNECTION_STRING;

if (connectionString) {
  useMicrosoftOpenTelemetry({ azureMonitor: { azureMonitorExporterOptions: { connectionString } } });
} else if (process.env.TELEMETRY_TO_CONSOLE === "1") {
  useMicrosoftOpenTelemetry({ enableConsoleExporters: true });
}
```

With neither variable set, telemetry is off and your terminal looks the
way it always has.

**3 · One line in `apps/web/next.config.ts`.** Add the package to the
list that's already there, so Node loads it instead of the bundler:

```ts
serverExternalPackages: ["@electric-sql/pglite", "pglite-prisma-adapter", "@microsoft/opentelemetry"],
```

**4 · The logger.** Replace the body of `packages/log/src/index.ts`. The
`mixin` stamps every line with the request's trace id. The hook hands the
same line to OpenTelemetry, which is how it reaches Azure.

```ts
import pino from "pino";
import { trace } from "@opentelemetry/api";
import { logs, type AnyValueMap } from "@opentelemetry/api-logs";

const level = process.env.LOG_LEVEL ?? (process.env.NODE_ENV === "production" ? "info" : "debug");

// pino's level numbers (10–60) → OpenTelemetry's severity numbers (1–24).
const severity = (pinoLevel: number) => Math.min(21, Math.max(1, (pinoLevel / 10 - 1) * 4 + 1));

export const log = pino({
  level,
  mixin() {
    const traceId = trace.getActiveSpan()?.spanContext().traceId;
    return traceId ? { traceId } : {};
  },
  hooks: {
    logMethod(args, method, levelNumber) {
      const [first, second] = args as unknown[];
      const fields = typeof first === "object" && first !== null ? (first as Record<string, unknown>) : {};
      const attributes: AnyValueMap = {};
      for (const [key, value] of Object.entries(fields)) {
        attributes[key] = value instanceof Error ? (value.stack ?? String(value)) : (value as AnyValueMap[string]);
      }
      logs.getLogger("app").emit({
        severityNumber: severity(levelNumber),
        severityText: pino.levels.labels[levelNumber],
        body: typeof first === "string" ? first : typeof second === "string" ? second : "",
        attributes,
      });
      method.apply(this, args);
    },
  },
});
```

**5 · Typecheck.** Run `pnpm typecheck`. If it fails in
`packages/db/src/client.ts` with *Argument of type 'Pool' is not
assignable*, you have two copies of the Postgres types. The error names
both versions. Pin the older one under `overrides` in
`pnpm-workspace.yaml`, then `pnpm install` again:

```yaml
  "@types/pg": 8.15.6
```

**6 · See it, before Azure.** Add one structured log line to a route you
care about, fields first and message last:

```ts
log.info({ userId, count: jobs.length }, "jobs listed");
```

Then run the app with telemetry printed to the terminal, and load that
route:

```bash
TELEMETRY_TO_CONSOLE=1 pnpm dev
```

It prints a lot. In it, find three things with the same `traceId`: the
request (`name: 'GET /your-route'`), the query (`'db.query.text'`), and
your log line. That's one tracking number on three scans.

**7 · Document it.** Add `APPLICATIONINSIGHTS_CONNECTION_STRING`,
`TELEMETRY_TO_CONSOLE`, and `OTEL_SERVICE_NAME` to `.env.example`, each
with a comment saying what it does. The PR includes the updated
`pnpm-lock.yaml`.

## Part 2 · Follow one request on Azure (solo)

**1 · Your own Application Insights.** In the Azure portal, with your
Azure for Students subscription: *Create a resource* → **Application
Insights**. Put it in the resource group you made in week 9, pick a
region near you, and let it create its Log Analytics workspace. The first
5 GB a month is free, and a class project won't get near that.

**2 · Point your app at it.** Copy the **Connection String** from the
resource's Overview page into `apps/web/.env.local`:

```bash
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=...;IngestionEndpoint=...
OTEL_SERVICE_NAME=<your-team>-<you>
```

It isn't a password, but anyone who has it can send data into your
resource, so it stays out of git like everything else in that file. Run
`pnpm dev` and use your app for a couple of minutes. Click the slow thing
twice.

**3 · Find one request.** In the portal, open your resource. *Live
metrics* shows requests as they happen. Then open *Search*, find one of
the requests you just made (new data takes a few minutes to show up), and
click into it. You should see the request, the query inside it with its
time, and your log line. Screenshot that.

**4 · Ask one question of your own.** Open *Logs*. Start from this, then
change it until it answers something you actually want to know about your
app:

```kusto
requests
| where timestamp > ago(24h)
| summarize count(), p50 = percentile(duration, 50), p95 = percentile(duration, 95) by name
| order by p95 desc
```

Which route is slowest at p95? Which query runs the most? What did one
user do in the last hour? Save the query in the repo as
`docs/queries/<the-question>.kql`, with a comment at the top that says the
question in plain words and what the answer was when you ran it.

**5 · One alert, and press its test button.** In your resource, *Alerts*
→ *Create alert rule*. Signal: **Failed requests**. Condition: greater
than 0 over 5 minutes. Action: an email to you. Ten metric alerts a month
are free.

Same rule as last week: an alert you've never seen fire is one you're
taking on faith. So break something on purpose. Change one character of
the database password in `.env.local`, restart, load `/api/health` a few
times, and wait for the email. Screenshot it. Then fix the password.

## The PR (solo)

Your `.kql` file is the change. The description carries the receipts:

- the screenshot of one request with its query and log line
- the question your query answers, and what you learned from the answer
- the screenshot of the alert that fired

Your reviewer runs your query against their own Application Insights and
tells you whether the answer surprised them.

## Done means

Your team's repo starts telemetry when the app boots, and logs carry a
trace id. You have a merged PR with one saved query, and screenshots of a
request followed end to end and an alert you watched fire.

## The interview question

"A user says the app is slow. What do you do?" The weak answer is a list
of things you'd try. Yours is a procedure: find the request, read where
its time went, check whether it's everyone or one person, and look at
what changed. Then say how you'd know it was fixed.

## What we didn't cover

Fixing what you find: indexes, query plans, and the N+1 query are the
watch-ahead material, and they're where most slow requests end up.
Dashboards and workbooks, load testing, telemetry from the browser, and
what logging costs once an app is busy. All real, all later. You can't
fix the slow thing until you can see which thing is slow.

So, which request did you follow, and where did its time actually go? 🔰
