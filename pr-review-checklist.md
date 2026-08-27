# PR Review Checklist v1

*Taught in 10 minutes in week 2; used every week after. This is the whole
checklist on purpose — v1 fits on an index card. The job-grade version
(adversarial reading, subtle breakage, AI hallucinations) arrives week 11.*

---

## The four moves

**1 · Pull it.** Check out the branch. A review from the diff page alone is
a guess with a green button.

**2 · Run it.** Boot it, click the thing, hit the endpoint. Does it do what
the PR *says* it does? You are the first user of this change.

**3 · Read it — in hunk.** `hunk diff` against the base branch; `[` and `]`
walk you hunk by hunk. Read **every hunk** — approval means "I read all of
it," not "the parts that looked interesting." If the author used an agent,
this is where "the agent wrote it" stops being anyone's explanation: now
*you've* read it, so now *two* people own it.

**4 · Ask one real question.** Every review, at least one comment that isn't
"LGTM." A real question is one you actually want answered: "what happens if
this list is empty?" · "why a string here and an enum there?" · "I can't
follow this function from its name — what does it do?" Not knowing is a
contribution: **"I can't follow this" is one of the most valuable comments a
reviewer can make.**

## The ground rules

- **Comment on the code, not the person.** "This query isn't scoped" — yes.
  "You forgot to scope" — no. Same fact, different sentence; the first one
  is about the work.
- **Approve = I read every hunk and ran it.** Don't spend your approval
  cheaper than that; your teammate is counting on it — the gate means their
  homework doesn't merge without you.
- **Blocking is normal.** "Requesting changes" is a reviewer doing the job,
  not an insult. What blocks vs. what's a nit goes in your team charter at
  kickoff.
- **15 minutes stuck means ask** — reviews included. Can't tell if a change
  is right? Say so in the comments and pull in a teammate or TA. A stuck
  review left silent blocks the author twice.
