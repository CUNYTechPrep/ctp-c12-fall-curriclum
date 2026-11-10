# Week 10 Homework — One Test That Can Fail

*Assigned the week-10 session · solo, in your team repo · one test for one
promise, with proof that the test can fail · one gated PR, reviewed by a
teammate, merged before the week-11 session · plus one tiny team PR first,
so your tests actually run in CI*

# tl;dr

Pick one promise your app makes. Write the test that tries to break it.
Then break the code on purpose and watch your test go red, on your laptop
and in CI. A test you've never seen fail is a test you're taking on faith.

## The test button

A smoke detector has a test button because looking at it tells you
nothing. One with a dead battery looks exactly like one that works, right
up until the fire. So you press the button, it screams, and now you know.

A green test is the same. Green could mean the code is right. It could
also mean the test checks nothing, or that it never ran. In class we saw
both: a test that stayed green with the user filter deleted, and a suite
of real tests that CI never called. This week you press the button on one
test of your own, and you keep the receipt.

## Step 0 · Wire the detector in (team, once)

Open `package.json` and `.github/workflows/ci.yml` in your team repo and
read them side by side.

```json
"test": "turbo run test",
"test:integration": "vitest run",
```

`pnpm test` runs the `test` script inside each package under `apps/` and
`packages/`. Your integration tests live at the repo root, in
`tests/integration/`, and `pnpm test:integration` is the command that runs
them. If `ci.yml` only calls `pnpm test`, nothing in `tests/integration/`
has ever run on a PR.

One teammate fixes that in a PR of its own, a few lines long, before
anyone starts their test:

```yaml
      - name: Test
        run: pnpm test

      - name: Integration tests
        run: pnpm test:integration
```

Merge it, and everybody branches from there. If your team already did
this in project time, you're done with step 0.

## Step 1 · Pick a promise

One sentence, from your spec, that you'd be embarrassed to break. Some
that most of your apps make:

1. User A can't see, change, or delete user B's things.
2. Bad input is rejected before it reaches the database.
3. Deleting hides a thing without destroying it.
4. A change that implies history writes its history in the same
   transaction.
5. A signed-out request gets a 401, not data.

Check that nobody's testing yours already, then say which one you're
taking in your team channel, so five people don't write the same test.

## Step 2 · Write the test that tries to break it

Add a file under `tests/integration/`, named for the promise. The setup
is the same in every file: point the database client at PGlite, which is
a real Postgres running inside the test, and import your code after that.

```ts
import { describe, it, expect, beforeAll } from "vitest";

beforeAll(async () => {
  process.env.PGLITE_DATA_DIR = "memory://";
  delete process.env.DATABASE_URL;

  const { prisma } = await import("@project/db");
  await prisma.user.create({ data: { id: "user-a", name: "A" } });
  await prisma.user.create({ data: { id: "user-b", name: "B" } });
}, 30000);

describe("notes are private", () => {
  it("keeps user B's note out of user A's list", async () => {
    const { createNote, listNotes } = await import("@project/domain");

    await createNote("user-a", { title: "mine" });
    await createNote("user-b", { title: "not mine" });

    const titles = (await listNotes("user-a")).map((n) => n.title);
    expect(titles).toContain("mine");
    expect(titles).not.toContain("not mine");
  });
});
```

Notice the two assertions. `not.toContain` alone would pass if the list
came back empty every time, which is a different bug, and one this test
would wave through. The first assertion is there so the second one means
something.

Your names and your schema will be different, so adapt it. Two rules
carry over whatever you're testing. The test makes everything it needs,
and never leans on what another test left behind. And the assertion is
about the promise: `expect(result).toBeDefined()` is not a claim about
anybody's privacy.

Run just your file while you work:

```bash
pnpm test:integration tests/integration/notes-private.test.ts
```

The worked example is Callback's
[`tests/integration/jobs.test.ts`](https://github.com/CUNYTechPrep/callback/blob/main/tests/integration/jobs.test.ts),
which tests three promises this way.

## Step 3 · Press the test button

Now break the promise. Find the one line that keeps it (the user filter
in the query, the validation rule, the `deletedAt` check) and delete it.
Run your test.

Red? Good. Copy the failure output, you'll need it. Put the line back and
watch it go green again.

Still green? Then your test isn't testing the promise, and you just found
that out for the price of one deleted line. Fix the test, not the claim,
and press the button again.

## Step 4 · See it red in CI

A test that fails on your laptop and passes in CI protects your laptop.
So do it once more where it counts, on your PR branch:

```bash
# with the line deleted again
git commit -am "break: remove the user filter, on purpose"
git push                      # watch the PR's check go red
git revert HEAD --no-edit
git push                      # and green again
```

This is a branch, so nothing you break here reaches main. Grab the link
to the red run from the PR's Checks tab before you move on.

## The PR

The description carries the receipt:

- the promise, in one sentence
- the line you deleted to break it
- the failure output from step 3
- the link to the red CI run from step 4

Your reviewer doesn't just read the test. They pull the branch and press
the button a different way: break the same promise somewhere else, or
with a different line, and see whether your test notices. If it doesn't,
that's the review comment.

## Stretch · Make it a rule

Right now a red X is advice, and the merge button still works. In your
repo's Settings → Branches (or Rules), require the status check named
`checks` on `main`, after the job in `ci.yml`. GitHub only offers this on
private repos with a paid plan, so if yours says no, write that down and
move on.

## Done means

Your team's `ci.yml` runs `pnpm test:integration`. Your PR is merged with
one new test, a description that shows it failing locally and in CI, and
a reviewer who tried to sneak a broken promise past it.

## The interview question

"How do you test your code?" Most answers are a list of tools. Yours is a
story now: here's a promise my app makes, here's the test that tries to
break it, here's how I know that test can fail, and here's what stops a
PR from merging when it does.

## What we didn't cover

Browser tests that click through the real UI (Playwright), coverage
percentages and what they do and don't tell you, tools that press the
test button for you (mutation testing, StrykerJS), and evals for AI
features beyond the one slide. All real, all later. One test you trust
beats twenty you've never seen fail.

So, did your test go red the first time you pressed the button? 🔰
