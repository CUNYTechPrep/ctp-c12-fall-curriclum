# Week 4 Homework — An Endpoint From Your Schema

*Assigned the week-4 session · one model from your schema becomes two
routes · a gated PR, reviewed by a teammate, merged before the week-5
session · the worked example is the `example/todos` branch of the starter*

# tl;dr

Pick one model from the schema you merged in week 3. Give it a Zod schema
in `packages/domain`, two queries scoped by the current user, and a route
file with `GET` and `POST`. Every error comes back as
`{ error: { code, message } }`. Prove it with `curl`, once as yourself and
once as a stranger, then open the PR. That's the whole assignment, the rest
of this page is how.

## A function with a stranger on the other end

An endpoint is a function. Input, side effects, output, same as anything
else you've written. The only new thing is who typed the input. Until now
every value that reached your database came from you, or from a seed file
you wrote. From this week on it comes from a request, and a request is a
stranger at the door with a bag. You don't know what's in the bag. You're
going to look before it comes inside.

That's all validation is. Zod is how we look. And once you see the endpoint
this way, the shape of the file writes itself: check who's asking, check
what they brought, do the work as *them*, say what happened in one
consistent voice. The starter calls that the server entry ritual
(`docs/specs/web.md`), and it's the same order every time, on every route,
for the rest of the semester.

## Read the example first

Before you write a line, read the todo version on the `example/todos`
branch, in this order, the way you'd read any function from its inputs
to its outputs:

```
packages/domain/src/schemas/todo.ts      what a stranger is allowed to send
packages/domain/src/queries/todos.ts     what we do with it, scoped by userId
apps/web/app/api/todos/route.ts          the door: identity, Zod, query, errors
packages/domain/tests/todo-schema.test.ts   the contract, written down
```

It's about eighty lines total. Notice how the route never touches
`req.json()` directly once it's parsed, the query only ever sees
`parsed.data`. Notice that every query takes `userId` as an argument, no
exceptions, and that the one that can fail returns `null` and lets the
route decide what null means. Cool! Those three habits are the homework.
Your model is different, the habits aren't.

## Now yours

Say your model is `Deck` (swap in your own). Naming: the Zod schema is
the verb plus the noun, `CreateDeck`; the queries are `listDecks` and
`createDeck`; the route is the plural noun, `/api/decks`.

**1 · The schema.** `packages/domain/src/schemas/deck.ts`:

```ts
import { z } from "zod";

export const CreateDeck = z.object({
  title: z.string().trim().min(1, "Title is required").max(120),
  exam: z.string().trim().min(1),
});

export type CreateDeckInput = z.infer<typeof CreateDeck>;
```

Only the fields a stranger is allowed to set. Not `id`, not `userId`, not
`createdAt`. Those are ours. If your Prisma model has twelve columns and
three of them come from the request, the Zod schema has three fields.
This is the part most people get backwards the first time: the Zod schema
is not a copy of the Prisma model, it's the *doorway* into it.

**2 · The queries.** `packages/domain/src/queries/decks.ts`:

```ts
import { prisma } from "@project/db";
import type { CreateDeckInput } from "../schemas/deck";

export function listDecks(userId: string) {
  return prisma.deck.findMany({ where: { userId }, orderBy: { createdAt: "desc" } });
}

export function createDeck(userId: string, input: CreateDeckInput) {
  return prisma.deck.create({ data: { userId, ...input } });
}
```

`userId` is the first argument to both, and it's in the `where` and the
`data`. There is no version of these functions without it. If your model
writes a history row like `JobEvent`, do it in one `$transaction` the way
`createTodo` does.

Export all of it from `packages/domain/src/index.ts`. The barrel is empty
on `main` on purpose; you're the first thing in it.

**3 · The route.** `apps/web/app/api/decks/route.ts`. Copy the todos route
and rename, honestly. The ritual is the same:

```ts
export async function POST(req: Request) {
  const userId = await currentUserId();            // 1 · who's asking
  if (!userId) return unauthenticated();

  let body: unknown;                                // 2 · what they brought
  try { body = await req.json(); } catch { return badJson(); }
  const parsed = CreateDeck.safeParse(body);
  if (!parsed.success) return validation(parsed.error);

  const deck = await createDeck(userId, parsed.data); // 3 · do it, as them
  return Response.json({ deck }, { status: 201 });    // 4 · say so
}
```

`body` is typed `unknown` and stays that way until Zod says otherwise.
That's not ceremony, that's the type system refusing to let you reach into
the bag. And every early return is the same shape:
`{ error: { code, message } }`, 400 for a bad body, 401 for no user, 404
for a thing that isn't yours. Not 403. A stranger asking about your deck
gets the same answer as a stranger asking about a deck that doesn't exist,
because "it exists but it's not yours" is information, and we don't hand
it out.

**4 · The contract, written down.** `packages/domain/tests/deck-schema.test.ts`,
modeled on the todo one, three or four cases: a good input, an empty
string, a missing field, a wrong type. Change the `test` script in
`packages/domain/package.json` from the placeholder echo to `vitest run`,
then `pnpm test` from the root. The schema test is the cheapest test you'll
write all semester and it's the one that stops a teammate from quietly
changing what the API accepts.

**5 · Prove it at the door.** With `pnpm dev` running:

```bash
curl -s localhost:3000/api/decks
curl -s -X POST localhost:3000/api/decks -H 'content-type: application/json' \
  -d '{"title":"Baruch transfer","exam":"BUS 1000"}'
curl -s -X POST localhost:3000/api/decks -H 'content-type: application/json' -d '{}'
curl -s -X POST localhost:3000/api/decks -d 'not json'
```

Predict all four before you run them. Then the one that matters most:

```bash
curl -s localhost:3000/api/decks -H 'x-user-id: someone-else'
```

Until week 8 the identity stub reads that header, so this is you,
knocking on your own door as a stranger. The list should be empty. If it
isn't, a query is missing its `userId`, and that's the bug you'd rather
find tonight than in week 10 with real users. Paste the five commands and
their responses into the PR description. That's your review's "run it"
step, done for the reviewer in advance.

**6 · The spec, and the table.** One file from
`docs/specs/_template-feature.md` at `docs/specs/<domain>/<thing>.md`,
with an endpoint table in it, one row per route: path, method, input
schema, error codes, what it's scoped by. Two rows tonight. Every endpoint
you add for the rest of the semester adds a row, and by week 9 that table
is your product's API, documented, which is a thing most teams never have.

## Done means

The PR merged into `main` with a teammate's review, and the reviewer
actually ran the stranger curl (ask them). `pnpm test` green. Two rows in
the endpoint table. If your product needs a `PATCH` or `DELETE` this week
too, go ahead, it's the same ritual with a `[id]` folder, and the todos
branch has one to copy from. Two is the bar, not the ceiling.

## What we didn't cover

Pagination, query-string filtering, and a `GET /api/decks/[id]`. All real,
all next week or the week after, all the same ritual. Build the two you
need tonight. When you need the third one you'll know.

So, what's one input your product accepts right now that it shouldn't?
Which line refuses it by next session? Tell your reviewer in the PR. 🔰
