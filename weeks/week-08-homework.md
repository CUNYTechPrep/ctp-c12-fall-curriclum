# Week 8 Homework — Real Sign-In

*Assigned the week-8 session · replace the dev identity stub in your team
repo with real GitHub sign-in · one gated PR, reviewed by a teammate,
merged before the week-9 session · pair up: proving it takes two GitHub
accounts · the worked example is Callback's sign-in, built in this week's
open build session*

# tl;dr

Since week 1 your app has known *who* a request is for, but it has never
asked for proof. This week it does. You'll register a GitHub OAuth app,
add the four tables Auth.js needs, and change the inside of
`currentUserId()` so identity comes from a session row instead of a header
anyone can type. Then two people sign in from two browsers and prove
neither can see the other's data.

## The coat check

A coat check works because of what you *don't* get. You hand over your
coat and get back a small numbered ticket. The ticket isn't the coat, and
it isn't your name. It's a random number that only means something to the
person behind the counter, who keeps the real list. Lose the ticket and
the attendant can void it. Forge one and you're guessing among millions.

That's a database session. The cookie is the ticket: long, random, and
meaningless by itself. The `Session` table is the attendant's list. Every
request, the server looks the ticket up and finds out whose it is. When
you sign out, the row gets deleted, and yesterday's cookie is a ticket for
a coat that isn't there anymore.

The stub you've been using works differently. It's a coat check that asks
"what's your name?" and hands over whatever coat matches. Tonight we fix
the counter, and nothing else changes. Every page and action you've
written since week 4 keeps calling `currentUserId()`.

## The setup guide

Work on a branch: `git switch -c auth/github-sign-in`.

**1 · Register a GitHub OAuth app.** Go to
[github.com/settings/developers](https://github.com/settings/developers),
click *OAuth Apps*, then *New OAuth App*.

- **Homepage URL:** the URL you open your app at. That's
  `http://localhost:3000` on your own machine, or
  `https://<your-codespace>-3000.app.github.dev` in a Codespace.
- **Authorization callback URL:** the same URL plus
  `/api/auth/callback/github`. It has to match *exactly*: protocol, host,
  port, path.

Then *Generate a new client secret* and keep the tab open, because GitHub
only shows it once. Each teammate registers their own development app,
since everyone's dev URL is different.

**2 · Install.** From the repo root:

```bash
pnpm --filter @project/auth add next-auth@beta @auth/prisma-adapter "@project/db@workspace:*"
```

`next-auth@beta` is Auth.js v5, the version with the Next.js App Router
API used in class. The auth package needs `@project/db` because the
adapter writes sessions through your Prisma client.

**3 · Secrets.** From `apps/web/`:

```bash
npx auth secret
```

That generates `AUTH_SECRET` and writes it to your env file. Make sure it
ended up in `apps/web/.env.local`, then add the other two values from step
1:

```bash
AUTH_SECRET=…            # from npx auth secret
AUTH_GITHUB_ID=…         # the OAuth app's Client ID
AUTH_GITHUB_SECRET=…     # the client secret you generated
AUTH_URL=https://<your-codespace>-3000.app.github.dev   # Codespaces only
```

Next loads `apps/web/.env.local` itself, so these reach the app even
though turbo filters the shell's environment. Check that it's ignored
before anything else: `git check-ignore apps/web/.env.local` should print
the path back. The starter's `.gitignore` already covers `.env*`. If a
secret ever lands in a commit, regenerate it on GitHub right away, because
deleting the commit doesn't un-leak it.

**4 · The tables.** In `packages/db/prisma/schema.prisma`, your `User`
gets four fields (keep everything it already has), and three new models
arrive, copied from the Auth.js Prisma adapter:

```prisma
model User {
  // …everything you already have, plus:
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  accounts      Account[]
  sessions      Session[]
}

model Account {
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@id([provider, providerAccountId])
}

model Session {
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@id([identifier, token])
}
```

Skip a `name` or `email` line if your `User` already has it. The adapter
expects `User.id` to be a `String`. If yours is an `Int`, stop here and
talk to your TA before going further.

**5 · The migration.** From `packages/db/`. Use your next number, which
might be `0003` or later:

```bash
git show main:packages/db/prisma/schema.prisma > /tmp/before.prisma
npx prisma migrate diff --from-schema-datamodel /tmp/before.prisma \
  --to-schema-datamodel prisma/schema.prisma --script > prisma/migrations/0002_auth.sql
```

Read the SQL before anything runs, same as week 3: three new tables, a few
new `User` columns, foreign keys back to `User`. Then from the root:

```bash
pnpm prisma:generate
pnpm db:reset    # stop dev first; the db-server applies migrations on boot
pnpm dev
```

**6 · The config.** Create `packages/auth/src/auth.ts`:

```ts
import NextAuth from "next-auth";
import GitHub from "next-auth/providers/github";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { prisma } from "@project/db";

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [GitHub],
  session: { strategy: "database" },
  callbacks: {
    session({ session, user }) {
      session.user.id = user.id;
      return session;
    },
  },
});
```

Export it from `packages/auth/src/index.ts`, next to `currentUserId`:

```ts
export { handlers, auth, signIn, signOut } from "./auth";
```

If TypeScript says `id` doesn't exist on `session.user`, the Auth.js
TypeScript guide shows the short module augmentation that fixes it.

**7 · The route.** Create
`apps/web/app/api/auth/[...nextauth]/route.ts`:

```ts
import { handlers } from "@project/auth";
export const { GET, POST } = handlers;
```

**8 · The swap.** Replace the inside of
`packages/auth/src/current-user.ts`:

```ts
import { auth } from "./auth";

export async function currentUserId(): Promise<string | null> {
  const session = await auth();
  return session?.user?.id ?? null;
}
```

Notice the return type: `string` became `string | null`. Run
`pnpm typecheck` and the compiler hands you a list of every caller that
has to decide what "nobody" means. Pages redirect to `/login`, and actions
and route handlers return the standard
`{ error: { code: "UNAUTHENTICATED", message } }` with a 401. Then:

```bash
git grep -n "x-user-id"
```

Nothing should print in code or tests. If a doc still describes the old
stub (the starter's `docs/specs/auth.md` does), update it in the same PR,
because the spec rides along. Delete the `DEV_USER_ID` fallback too. Dev
signs in like production now.

**9 · A way in and a way out.** A `/login` page with a sign-in button,
and a sign-out button in your layout's header. Both are forms whose action
is an inline Server Action:

```tsx
<form action={async () => { "use server"; await signIn("github"); }}>
  <button type="submit">Sign in with GitHub</button>
</form>
```

`signOut()` is the same shape. Last week's Origin check guards both of
them.

## Prove it

Every one of these goes in the PR, as a screenshot or a sentence.

1. **It remembers you.** Sign in, refresh, close the tab, come back. Still
   signed in.
2. **It forgets you.** Sign out, then press back. Nothing private renders,
   because every page asks again.
3. **The cookie is a ticket, not a name.** Signed in, open DevTools →
   Application → Cookies, change one character of
   `authjs.session-token`, reload. You're nobody. (On HTTPS the cookie is
   named `__Secure-authjs.session-token`.)
4. **Two people, zero bleed.** Your teammate signs into *your* running app
   with *their* GitHub account. In a Codespace, set port 3000's visibility
   to public for the test and set it back after. Each of you creates
   something, and neither of you can see the other's. Then try opening
   their item's URL from your session. It should be a 404.

## When it goes sideways

| What you see | What it usually means |
|---|---|
| GitHub says `redirect_uri` is not associated with this application | The callback URL in step 1 doesn't exactly match the URL you're on. Check http vs https, the port, and the Codespace name. In a Codespace, set `AUTH_URL`. |
| `MissingSecret` | `AUTH_SECRET` isn't loaded. It belongs in `apps/web/.env.local`. Restart `pnpm dev`. |
| An adapter error about a missing table | The migration didn't apply. Stop dev, `pnpm db:reset`, start dev, and watch for `applied migration` in the log. |
| `Invalid Server Actions request` on sign-in | Week 7's Codespaces fix: `allowedOrigins` in `next.config.ts`, plus `globalPassThroughEnv` in `turbo.json`. |
| Signed in, but `currentUserId()` returns null | The `session` callback in step 6 is missing, so the id never reaches the session. |

## Reviewing one of these

Same four moves, and the checklist is the last slide from tonight: identity
from the session only, every read scoped, every write checked on the
server, foreign ids get a 404, secrets in `.env.local`, two people with
zero bleed. Your reviewer runs proofs 3 and 4 themselves. The best real
question to ask: *where else does this app trust something the browser
sent?*

## Done means

The PR is merged with a teammate's review. `git grep x-user-id` finds
nothing in code or tests. All four proofs are in the description. `.env.local` isn't in
the diff.

## The interview question

"What's a session?" sounds basic, and that's why it's a good question.
The weak answer is "it's how you stay logged in." You have a better one
now. A session is server-side state tied to a random token the browser
holds in a cookie. HttpOnly keeps scripts from reading it, Secure keeps it
off plain HTTP, and SameSite keeps other sites from sending it. The server
looks the token up on every request, and signing out deletes the record,
so a stolen or stale cookie stops working. Then say when you'd pick a JWT
instead, and what you'd give up.

## What we didn't cover

Roles and permissions, like admins or shared workspaces. Passkeys, which
the adapter supports through its `Authenticator` model. Email magic links,
more providers, and rate-limiting sign-in attempts. Also worth knowing:
the Auth.js project is now part of Better Auth, as the banner on
authjs.dev says. The ideas from tonight carry over one to one: sessions,
cookies, OAuth, and identity from the server only.

So, where else does your app trust something the client sent? Find one
before your reviewer does, and put it at the top of the PR. 🔰
