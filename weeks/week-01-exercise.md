# Week 1 Study Hall — Clone it. Run it. Start your aspect.

*~55 minutes · in pods, but on your own machine*

---

## 0 · What you're cloning

The **course starter**, on its **`example/todo` branch** — the skeleton with
one small, complete app flowing through every layer (username sign-in,
todos, image attachments, a thumbnail worker, live notifications). Your
*team* repo comes next week, once teams form around the pitches (it starts
as the blank skeleton — no example in it). This clone is yours, for
breaking — and it's your **jigsaw study copy** for this week's homework, so
keep it.

```bash
git clone https://github.com/CUNYTechPrep/ctp-starter.git starter && cd starter
git switch example/todo
```

## 1 · Get it running (the health check)

```bash
pnpm install
pnpm prisma:generate     # builds the typed database client
pnpm dev                 # web + YOUR OWN Postgres server + Azurite
```

Then, in a **second terminal**:

```bash
pnpm db:seed             # creates users @ada and @grace with starter todos
```

Stuck anywhere in this step? That's not falling behind — week 1's study
hall doubles as install triage. Flag a TA immediately; nobody leaves
tonight unable to run the app.

## 2 · Prove it runs

- Open `http://localhost:3000`. You didn't ask for `/login`, but that's
  where you are. (File that away — someone in your pod owns "the request
  path" for homework.)
- Sign in as `ada`. Her todos are there because *you seeded them* — the
  data came from your own database server, the one `pnpm dev` started.
- Open `http://localhost:3000/api/health`. Read the JSON. Both facts it
  reports will matter later.

## 3 · Change one visible thing

Find any string or color in `apps/web` — a heading, a button label — change
it, save, and watch the browser update. Trivial on purpose: you've now
proven the whole loop *editor → dev server → browser* works on your machine,
and you know at least one file that's really the UI.

## 4 · Start your aspect (the rest of the block)

Open your claimed aspect's six-question study set in
[expert-jigsaw-study-questions.md](../expert-jigsaw-study-questions.md)
and start working it, **in order**, right now, while the instructor and
your pod are in the room. Tonight's realistic target is the first one or
two questions — the *run it* tier. The point of starting here instead of
at home: the first confusion hits while you can just *ask*.

Two habits to practice from question one:

- **Verify everything by running the code** — the mentor agent and the
  READMEs are fair sources; neither is truth until you've executed it.
- When you get stuck, note *what you predicted vs what happened* before
  asking. That sentence is the question.

---

## This week, at home (part of your jigsaw study)

Two experiments from tonight's demo, before your deeper questions — both
are predict-first:

- **Stop the database.** Write your prediction first, both parts: what
  will the todo page show when the database is down? What will
  `/api/health` say? Then: `Ctrl+C` the `pnpm dev` terminal, start only
  the web app — `pnpm dev:web` — and load both URLs. Was the failure
  *honest* — did it tell you what was actually wrong, in plain words?
  Restore with `Ctrl+C` → `pnpm dev`.
- **Trace the seed.** Where does `pnpm db:seed` get its data? Follow the
  trail from the root `package.json` to the actual rows-to-be. Then
  predict: what happens if you run it a *second* time? Run it, and find
  *why* in the code — users and todos are protected from duplication by
  **two different mechanisms**. Name both.

## Done early? (stretch)

- Sign in as `grace`. Different todos. Find the line of code that decides
  *whose* todos a query returns — you're looking at the most important rule
  in the app.
- Start the background half: `pnpm worker` in a third terminal, attach an
  image to a todo, and watch: queued → processing → toast. No refresh, no
  polling. Someone in your pod is about to become the expert on how.

## Done means

You ran the app, changed it, and started your aspect's study questions. Before week 2: the two at-home experiments,
your aspect studied for the teach-back, and **your pitch + choices in the
project interest spreadsheet**. If any step still fails on your machine,
tell a TA **before you leave tonight**.
