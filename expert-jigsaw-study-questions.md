# The Expert Jigsaw — Study Questions

One aspect per team member; between you, cover all four quadrants (see the
syllabus). All questions are asked of the **course starter's `example/todo` branch**
(your week-1 work-block clone) — the skeleton with one small app (username sign-in, todos, image attachments, a
thumbnail worker, live notifications) flowing through every layer. For your
aspect, work through its questions **in order** — they escalate from *run
it* to *trace it* to *why* to *break it*. Rules of the game:

- **Verify everything by running the code.** The mentor agent and the
  READMEs are both fair; neither is truth until you've executed it.
- **The last question of every set is an experiment.** Predict out loud
  first, then do it. Being wrong is the useful outcome.
- **Doc gaps are findings:** any question you could only answer by
  reading source — never from the docs — is a documentation gap. Note the
  worst one; it goes in your teach-back.
- **Teach-back (wk 2, ~4 min):** walk your team through 2–3 of these, your
  choice — but expect follow-ups from the rest of the set.

---

## 1 · The request path (`apps/web`)

1. Open the app signed out. What happens, and *where in the code* is that
   redirect decided? Then sign in and load the list: what files does that
   request touch, in order, from browser to database and back?
2. Where is the decision made about what data to fetch — client or server?
   How can you tell from the code?
3. What's the difference between a page and a route handler here? Which one
   serves `/api/health`, and why is it that kind?
4. What does a request see when the database is down? Try it: stop the DB,
   load `/` and `/api/health`, compare what each tells you.
5. Why does `/api/health` exist at all — who calls it besides you?
6. **Experiment:** rename a route folder. Predict: what breaks first — the
   build, or the request? Verify, then put it back.

## 2 · The domain door (`packages/domain`)

1. Find the schema that validates creating a todo. `curl` a POST with an
   empty title: what exact shape comes back, and with what status? Where is
   that shape decided — and why one shape everywhere?
2. The schema lives in a package the browser *could* import. Where does it
   actually run today? What would running it in the form buy you — and what
   does the server-side check guarantee no matter what the form does?
3. Sign in as `Ada`, then as `ada`. One account or two? Find the single
   line that makes it so — and say why that's validation's job, not the
   login form's.
4. Only the web app may import this package. What problem does that boundary
   prevent? (The ADR index will point you to the reasoning.)
5. How does a query in this package know whose todos to return? What stops
   it from returning someone else's? Find the test that proves it.
6. **Experiment:** `curl` a PATCH with `{"done": "true"}` — string, not
   boolean. Predict the status code and body, then verify.

## 3 · The data layer (`packages/db`, `apps/db-server`, `apps/migrate`)

1. What exactly starts when `pnpm dev` starts "your own Postgres server"?
   Where does its data live on disk?
2. What are the three doors in the database client, and when is each used?
3. What happens to migrations on boot? And why is editing an
   already-applied migration file *never* correct — what would actually go
   wrong?
4. Run `pnpm db:seed` twice. Why didn't the second run duplicate anything?
   Find the two different mechanisms that make that true (hint: users and
   todos are protected differently).
5. Where does the connection string live — and what's the *only* thing that
   changes when this app points at a cloud database in week 10?
6. **Experiment:** delete `.pgdata/`. Predict the full recovery path —
   including what happens to your session cookie — then run it.

## 4 · Identity (`packages/auth`)

1. Where does `currentUserId()` get its answer? What writes that value, and
   when? Trace one sign-in from the form to the cookie landing.
2. There's no password. So what, exactly, does signing in *prove*? Separate
   the two ideas this package's README insists on — which one does the app
   have today?
3. What's the rule about client-supplied user ids anywhere in the app? Find
   one query that proves the rule.
4. Why does asking for another user's todo return 404 and not 403? What
   would a 403 leak?
5. When real sign-in arrives (week 8), what changes — and what stays
   exactly the same? Which seam absorbs the swap? And find the guard that
   keeps *this* sign-in out of production — quote it.
6. **Experiment:** run the sanctioned attack. Sign in as yourself, edit the
   `session` cookie in devtools to another user's id, refresh. Predict
   first: what will you see, and what will `curl` on a todo id you *don't*
   own return? Why is being someone not the same as escalating?

## 5 · The background half (`apps/worker`)

1. How does the worker find out there's work to do? Attach an image and
   trace the job from enqueued to done — name every file it passes through.
2. Why is the worker a separate process instead of more code in the web
   app? What can it do that a request handler shouldn't?
3. What database does the worker talk to, and why does "a second client"
   matter when it writes the thumbnail?
4. Attach an image while the worker is *stopped*. What does the UI honestly
   tell you? What happens when the worker comes back — and where was the
   job waiting in between?
5. A job fails halfway through (bad image, crash). Read the worker's loop
   carefully: what does this design choose to *lose*, and what trail does
   it leave? Would you make the same choice?
6. **Experiment:** kill the worker mid-job. Predict the state of the queue,
   the database, and the UI, then look at all three.

## 6 · The cloud seams (`packages/services`)

1. What is Azurite standing in for? What starts it, and where does its data
   live?
2. Trace an upload: where do the bytes end up, and what goes in the
   database instead? Why are those split? (An ADR has the reasoning.)
3. A thumbnail finishes and a toast appears — with no polling anywhere.
   Trace that notification backwards: toast → SSE stream → Postgres. What
   does `pg_notify` buy that a `setInterval` wouldn't?
4. State the seam pattern in one sentence. Inventory every seam in the app
   — database, blob, queue, notify, identity. What's the switching env var
   for each?
5. What breaks if Azurite isn't running — and how does the app tell you?
   Is the failure honest?
6. **Experiment:** stop Azurite mid-session. Predict which features fail
   and how loudly, then verify.

## 7 · Tests & CI (`tests/`, `.github/workflows`)

1. What runs when you run `pnpm test` — and why does it need no running
   database or emulator? What is PGlite in this picture?
2. What exactly does CI run on every PR? When the check is red, where do
   you look first?
3. Find the integration test that proves users can't see each other's
   todos. What, specifically, would have to break in the app for it to
   fail?
4. What does a green check *not* prove? Find one real behavior the suite
   doesn't cover (the thumbnail pipeline is a good hunting ground).
5. "Tests that can't fail don't count." Pick a test, delete the behavior it
   tests, run the suite. Did it fail? What did that teach you about the
   test?
6. **Experiment:** make the smallest code change you can that turns CI red.
   What does that tell you about where the gate actually is?

## 8 · The docs & agents system (`docs/`, `AGENTS.md`, `.opencode/`)

1. What are the four document kinds, and what's each one's lifecycle? Which
   of them are you allowed to edit, and which never?
2. A PR changes a behavior. What's the rule about which document rides
   along in the same PR?
3. State the one-way linkage rule. If you wanted a spec's history, where
   would you look?
4. What does AGENTS.md tell a coding agent that a README doesn't? Ask the
   `mentor` agent something about the repo — can you tell which document
   its answer came from?
5. Which reviewer agent looks at a change under `docs/specs/`, and what
   would it flag?
6. **Experiment:** ask the mentor agent one question about a *teammate's*
   aspect, then verify its answer by running code. Did it hold up? Bring
   the verdict to the roundtable.
