# Team Charter — Night Shift *(worked example)*

*This is a finished charter for a fictional team — what "done" looks like
before you write yours (week 3's homework). Copy the shape, not the answers —
your team's answers will differ, and that's the point. Blank version:
[charter-template.md](charter-template.md) · how-to:
[charter-guide.md](charter-guide.md).*

---

## 1 · Team & Project

**Team name:** Night Shift

**Project (adopted pitch):** StudyBuddy — spaced-repetition flashcards for
CUNY transfer-credit exams, built by the people who have to take them.

**Section:** Fri 6:30 · **TA:** Priya

**Members:**

| Name | GitHub | Email |
|------|--------|-------|
| Ada Lovelace (pitcher) | @adalovelace |
| Grace Hopper | @gracehopper | grace@… |
| Hector Garcia | @hgarcia | hector@… |
| Keyshawn Seymour | @kseymour | key@… |

### Roles & responsibilities

Roles rotate weekly so nobody becomes "the one who always…". The
**stand-up lead** runs Tuesday's 15 minutes and posts the notes.
The **review captain** is first responder on every PR opened that week
(others can still review — the captain just guarantees nobody waits).
The **demo owner** keeps `main` deployable and runs the team's status
share when it's our turn. Rotation is in the team channel's pinned
message; whoever has it, has it — no swapping without a message.

Standing ownership (from the jigsaw): each member is the first stop for
questions in their aspect above. First stop, not sole owner — anyone can
change anything, but you ask the expert *before* you rewrite their layer.

Everyone, every week: one homework PR merged, one review given, stand-up
attended or an async update posted *before* it starts.

## 2 · The Product

*From the adopted pitch, sharpened as a team.*

**The problem:** Transfer students lose credits because requirement info is
scattered across three CUNY sites and stale PDFs, and there's no practice
material for the placement exams that decide what transfers.

**Who it's for:** CUNY students planning a transfer, starting with
BMCC → Baruch business majors (Ada's own path — we can test on real people).

**Three core features (the MVP):**

1. Decks by exam — a student picks a target exam and gets a curated deck.
2. Spaced review — cards resurface on a schedule; streaks are visible.
3. Share a deck — a link a study partner or advisor can open, read-only.

**What ships by Week 13 (demo day):** A stranger opens our URL, signs in
with GitHub, picks the "Baruch business transfer" deck, studies ten cards,
closes the tab, comes back tomorrow and the app knows which cards are due.

**Out of scope / v2 ideas (Week-9 pitch fodder):** user-authored cards,
mobile app, AI-generated decks, any social features beyond a share link.

## 3 · Working Agreement

**Where we talk:** `#team-night-shift` in the course Slack. Decisions get
a 📌; everything else scrolls.

**Response window:** 24 hours on weekdays, 48 on weekends. "Seen, will
answer tonight" counts as a response.

**When we meet (outside class):** Tuesday 8:00 pm stand-up (15 min, hard
stop). One optional pairing block Thursday 7–8 pm for whoever's stuck.

**Availability notes:** Grace works Sat/Sun and is offline both days.
Hector is in Puerto Rico Oct 9–14 (async only). Keyshawn has a Wednesday
night class — never schedule then. Ada is the early riser; message her
before 10 pm or wait until morning.

**How we decide when we disagree:** Try to agree in ten minutes. If we
can't, build the *smaller* version first and revisit after it's merged —
real code settles arguments that opinions can't. Product-scope calls go to
Ada as pitcher; technical calls go to whoever owns that layer.

**Definition of done:** Merged into `main` through the gate, CI green,
reviewed by someone who pulled and ran it, and it works at the preview URL
— not "works on my machine."

### Rituals

| Ritual | When | Shape |
|--------|------|-------|
| Stand-up | Tue 8:00 pm, 15 min | Each person: merged / in review / blocked. Blockers become a named owner before we hang up. |
| Team review (in class) | Every session, ~15 min of project time | One member's PR on the screen; the four moves (pull it, run it, read it, ask one real question). Comments filed as real review comments. |
| Async check-in | Thu, in the channel | One line each: what's in flight, anything that'll slip. Replaces a meeting, not a conversation. |
| Retro | Midterm (wk 7) + before demo day | 20 minutes: keep / stop / start. The charter gets edited on the spot — that's the output. |
| Planning | Sunday night, async, 10 min | Next week's PRs claimed in the channel by name, one issue each. If you can't name your PR on Sunday, that's the first thing to say at stand-up. |

We use GitHub issues as our board: one issue per PR, assigned to one
person, closed by the merge. If it isn't an issue, it isn't planned.

## 4 · Code & Review Norms

*Completed together at the week-3 kickoff, after our first migration
review.*

**Branch & PR flow:** `main` is protected. Branch from `main` as
`yourname/short-thing`, open a PR early (draft is fine), request the review
captain plus one. Squash-merge; the PR title is the commit message, so
write it like one.

**What blocks approval:** the reviewer couldn't run it; a query that isn't
scoped by the current user; a migration that edits an earlier migration
instead of adding a new one; AI-generated code the author can't explain
when asked. Style never blocks — leave a `nit:` and approve.

**Review response time:** first response within 24 hours on weekdays. If
you can't review in time, say so in the PR so the captain reroutes it —
silence is the only unacceptable answer.

**Comment conventions:** `nit:` (take it or leave it) · `q:` (a real
question — answer before merge) · `blocker:` (must change) · `praise:`
(say what's good; it's how we learn what to repeat). One `blocker:` per
real problem, not a wall of them.

## 5 · AI Working Norms

**Course policy (not optional):** no AI-generated code gets merged unread.
The PR author owns every line they open, wherever it came from. AI
explanations get verified by running the code.

**How we use AI as a team:** as a pair while building and an explainer we
verify. Every PR description says which parts were AI-drafted, in one
line — not as a confession, so the reviewer knows where to read slowest.

**What we never delegate to AI:** the schema and migrations (hand-typed,
per kickoff), anything touching user scoping, and the review itself — a
reviewer reads the diff, not a summary of it.

**What we build by hand first:** each layer's first instance. The first
endpoint, the first component, the first test in a file are typed; AI
accelerates the second one.

## 6 · When Things Go Wrong

Stuck protocol (course default): 15 minutes stuck → post in the team
thread → still stuck at stand-up → TA → office hours.

**If someone can't deliver on time:** say it in the channel the moment you
know — a Tuesday "I'm not going to make it" is a plan; a Friday silence is
a problem. The review captain redistributes; the missed PR moves to next
week, and the person owes the review captain a coffee, not an apology.

**If we have a conflict:** name it at stand-up, out loud, kindly. If it's
still there next stand-up, Priya (TA) mediates. If it's about the product
direction, it's Ada's call as pitcher and we move on.

## 7 · Commitment

We wrote this together, we mean it, and we'll revisit it at midterm and
update what isn't working.

| Signed | Date |
|--------|------|
| Ada Lovelace | Sep 18, 2026 |
| Grace Hopper | Sep 18, 2026 |
| Hector Garcia | Sep 18, 2026 |
| Keyshawn Seymour | Sep 18, 2026 |
