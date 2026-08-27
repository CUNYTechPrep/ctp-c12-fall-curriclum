# The Expert Jigsaw — Week 1 Homework

*Your first homework. Claimed at tonight's pod meet; due before week 2.*

The starter has **eight aspects**. Each pod member claims one — first come
in your pod thread, no two members on the same one — and becomes the
pod's expert on it. Between you, your pod covers the whole machine.
(Pods are week 1's temporary groups; teams form in week 2, around the
pitches — so every team you might join will have studied all eight.) All
of it runs against the **course starter's `example/todo` branch** — the
clone you made in week 1's study hall is your study copy (your team's
repo, created in week 2, starts as the blank skeleton; the example stays
upstream).

| # | Aspect | Where it lives | The question you'll answer for your pod |
|---|--------|----------------|------------------------------------------|
| 1 | The request path | `apps/web` | What happens between a click and a render? |
| 2 | The domain door | `packages/domain` | Where's the one place input becomes trusted? |
| 3 | The data layer | `packages/db` · `apps/db-server` · `apps/migrate` | Why does `pnpm dev` start a database — and what are the three doors? |
| 4 | Identity | `packages/auth` | Who am I, according to the app — and why is that safe for now? |
| 5 | The background half | `apps/worker` | How does work happen when no request is in flight? |
| 6 | The cloud seams | `packages/services` | What changes when we point at real Azure? |
| 7 | Tests & CI | `tests/` · `.github/workflows` | What does a green check prove — and what doesn't it? |
| 8 | The docs & agents system | `docs/` · `AGENTS.md` · `.opencode/` | Where does a decision live, vs a behavior, vs a procedure? |

## How to study

Your aspect has a **six-question study set** in
[expert-jigsaw-study-questions.md](expert-jigsaw-study-questions.md). Work
the questions **in order** — they escalate from *run it* to *trace it* to
*why* to a final **experiment you predict before running**.

The rules of the game:

- **Verify everything by running the code.** The mentor agent (ask it
  anything — it lives in `.opencode/`) and the READMEs are both fair
  sources; neither is truth until you've executed it.
- **The last question of every set is an experiment.** Predict out loud
  first, then do it. Being wrong is the useful outcome.

## What you deliver

**The teach-back** *(week 2, in class, in pods, ~4 minutes)* — walk your
pod through 2–3 of your six questions, your choice. Expect follow-ups from
the rest of the set. No slides; your editor and the running app are the
medium. From week 2 through kickoff, you're the first-stop for this layer —
for your pod, and then for the team you join.

While you study, keep a note of every question you could only answer by
**reading source** — never from the docs. Each one is a documentation gap.
Bring the worst one to your teach-back: *what you wish had been written
when you started* is the sharpest thing an expert can say about their
layer.
