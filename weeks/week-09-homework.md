# Week 09

*Assigned the week-09 session · two parts · part 1 is a team PR: a smoke
test of your live URL, written as a runbook and merged before the week-11
session · part 2 is solo: your own free Postgres server, with a personal
copy of the starter pointed at it*

# tl;dr

Your app has a real URL now. Part 1: walk it like a stranger would, with
a checklist, and write the checklist down as a runbook your team can run
again after every deploy. Part 2: provision a Postgres server of your own
on Azure's free tier, point a personal copy of the starter at it, and
watch `/api/health` say `db: "ok"` from a database you built. That's the
provisioning rep: everything the class server did for you, done yourself,
once.

## The pre-flight walkaround

Before every flight, a pilot walks around the plane. It's the same walk
every time, from the same checklist, even after ten thousand flights:
tires, flaps, lights, the pitot tube. Not because they don't trust the
mechanics, but because the checklist catches the thing nobody would think
to look at today. Then it's signed, so the next crew knows it happened.

Your deploy pipeline already runs the first item, which is `/api/health`
before any traffic moves. That's a good start and a very short checklist.
A health endpoint can say `db: "ok"` while sign-in is broken, secrets
leak into the page, and the first visitor after a quiet hour waits ten
seconds. So you walk around it yourself, write down what you checked, and
sign it with a date.

## Part 1 · Smoke-test your live URL (team, one PR)

Run every check against the **live URL**, not localhost. Write down the
result of each one as you go, including the ones that fail.

1. **Health.** `curl -i https://<your-app>/api/health` returns `200` and
   `"db":"ok"`.
2. **HTTPS.** Open `http://<your-app>`, without the s. You should end up
   on `https://` with the padlock, or be refused outright. Either is a
   pass. A page served over plain `http://` is a fail.
3. **Sign in, for real.** Sign in with GitHub on the live URL, using the
   production OAuth app, not your dev one. In DevTools → Application →
   Cookies, the session cookie is `__Secure-authjs.session-token`, with
   *HttpOnly* and *Secure* checked.
4. **Two people, zero bleed, in production.** A teammate signs in on
   their phone. Each of you creates something, and neither of you can see
   the other's. Their item's URL from your session is a 404.
5. **The core flow on a phone.** Do your product's main job start to
   finish on an actual phone, on cellular, not wifi.
6. **Cold start.** Leave the app alone until it scales to zero (give it
   at least ten minutes), then load it and time the first response. Write
   the number down. That's what your first user of the day waits.
7. **No secrets in the page.** View source and search the JavaScript in
   DevTools → Sources for your `AUTH_GITHUB_SECRET` value. Nothing. Then
   from the repo root, `git ls-files | grep -i env` lists `.env.example`
   and nothing else.
8. **Logs.** `az containerapp logs show -n <app> -g <resource-group>
   --follow`, then load a page. Your request shows up. Now you know where
   to look at 11pm.
9. **Rollback, before you need it.** `az containerapp revision list -n
   <app> -g <resource-group> -o table`. Write down the name of the revision
   serving right now, because that's your last good one.

**The PR.** Add `docs/runbooks/smoke-test-production.md`, written the way
Callback's runbook README says runbooks are written: numbered steps,
imperative mood, no prose. The checks above, adapted to your product's
real flows, with the exact commands. Under them goes a results table
with today's date, who ran it, pass or fail for each check, and the
cold-start time. If anything failed, file it as an issue and link it.
Your reviewer runs the runbook themselves before approving, because a
runbook nobody else can follow isn't one.

## Part 2 · Provision your own free Postgres (solo)

**1 · An Azure account that costs nothing.** Sign up for
[Azure for Students](https://azure.microsoft.com/free/students/) with
your CUNY email. No credit card. It includes $100 of credit, plus 12
months of free services, which include *750 hours of Flexible Server:
Burstable B1MS Instance, 32 GB storage, and 32 GB backup storage.* 750
hours is one server running all month, every month, for a year.

**2 · A budget alert, before anything else.** Cost Management → Budgets
→ add a budget for $5 with an email alert. If anything ever starts
costing money, you'll hear about it the same day, not at the end of the
month.

**3 · Create the server.** Portal → *Create a resource* → **Azure Database
for PostgreSQL flexible server**.

- **Resource group:** a new one, like `rg-c12-<you>`. When the year's
  over, deleting this one group deletes everything.
- **Workload type:** *Development*. Then under *Configure server*, check
  that compute is **Burstable, B1ms**. This is the free one. Anything
  bigger is not.
- **Storage:** 32 GB, with autogrow **off**. Storage can grow but never
  shrink, and past 32 GB you pay.
- **High availability** and **geo-redundant backup:** off.
- **Authentication:** PostgreSQL authentication. Pick an admin name and a
  long password, and save both in your password manager.
- **Networking:** public access, then *Add current client IP address*.
  The firewall only lets in addresses you list. If you're working in a
  Codespace, its outbound address is different from your laptop's, and it
  can change: run `curl -s ifconfig.me` in the Codespace and add that
  address too. Don't add `0.0.0.0–255.255.255.255`. That's "everyone on
  the internet," with your password as the only lock.

Create it. It takes a few minutes, which is a good time to do step 4.

**4 · Your own copy of the starter.** On
[github.com/CUNYTechPrep/ctp-starter](https://github.com/CUNYTechPrep/ctp-starter),
click *Use this template* to create a repo under your own account. Clone
it, then `pnpm install` and `pnpm prisma:generate`.

**5 · Point it at your server.** Build the connection string from the
server's *Overview* page:

```bash
# apps/web/.env.local (ignored by .env* in the starter's .gitignore)
DATABASE_URL=postgresql://<admin>:<password>@<server>.postgres.database.azure.com:5432/postgres?sslmode=require
PG_POOL_MAX=1
```

If your password has symbols like `@`, `:`, or `/`, they have to be
URL-encoded in the string (`@` becomes `%40`), or the URL means something
else. Then run only the web app, since you don't need the local Postgres:

```bash
pnpm dev:web
curl -s localhost:3000/api/health     # {"status":"ok","db":"ok"}
```

That `db: "ok"` is a query running in a data center, on a server you
built, through the same seam your team's app uses in production. Same
code as week 1, one variable different.

**6 · Break it on purpose.** Change one character of the password in
`.env.local`, restart, and curl again: `503` and `db: "error"`. That's the
exact failure the deploy pipeline's smoke test catches before a bad
revision ever takes traffic. Fix it, restart, green again.

**7 · Keep it free.** One B1ms server, under 32 GB, and that's it. A
second server running at the same time uses up the 750 hours twice as
fast. You can stop the server from its *Overview* page when you're not
using it, but Azure starts a stopped flexible server again automatically
after seven days. Check Cost Management → Cost analysis after a day: it
should say $0.

## Done means

**Part 1:** the smoke-test runbook is merged, with a dated results table
and an issue for every failed check, reviewed by a teammate who ran it.
**Part 2:** a screenshot of your own `/api/health` saying `db: "ok"`
against your server, and your budget alert set. Post both in your team
channel.

## The resume line

If it's true of your team's pipeline, and by now it should be, this line
is yours:

> Deployed a Next.js + PostgreSQL app to Azure Container Apps with
> GitHub Actions CI/CD: images built in CI and pushed to GHCR,
> health-gated revisions, zero-downtime promotion, and one-command
> rollback.

Be ready to defend every word of it. "Walk me through what happens when
you merge to main" is the interview question it invites, and you can now
answer it file by file: the workflow, the image, the registry, the
revision at 0%, the health check, the promotion, and the runbook for when
it goes wrong.

## What we didn't cover

Custom domains, preview environments for every PR, infrastructure as
code (Bicep or Terraform), and autoscaling past one replica. All real,
all later. Get one honest deploy working, with a walkaround you can
repeat, before any of them.

So, which check failed first on your live URL? Put it at the top of the
runbook's results table, with what you did about it. 🔰
