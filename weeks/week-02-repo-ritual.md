# The Team Repo Ritual — exact clicks

*Week 2 project time · ~30 minutes · one repo per team, created the night
your team forms — for the whole semester. Homework lands here as PRs from
tonight on; at week-3 kickoff your schema lands in the blank skeleton; this
same repo deploys in week 10 and demos in December.*

---

## 1 · Create the repo (one member drives, screen shared)

1. Open the course template: `⟨github.com/…/starter⟩`
2. Click the green **Use this template** → **Create a new repository**
3. **Owner:** the driving member's account. **Name:** your team name,
   kebab-case (e.g. `night-shift-app`)
4. Visibility: **Private** → **Create repository**

Your new repo is the **blank skeleton** — default branch only, empty
schema, all the infrastructure. That's correct: the worked example
(`example/todo`) stays upstream in the course starter repo, which everyone
cloned in week 1's study hall — that clone was your jigsaw study copy.

## 2 · Add your people

**Settings → Collaborators → Add people:**

- every teammate — role **Admin**
- the instructor + your section's TAs — so we can see your work, not to
  grade it (there are no grades)

## 3 · Protect main — the merge gate

**Settings → Branches → Add branch protection rule**, pattern `main`:

- ☑ **Require a pull request before merging**
- ☑ **Require approvals** — set to **1**

That's the whole gate, and it does more than it looks: GitHub won't let you
approve your own PR, so "1 approval" *means* "a teammate read your code."
No self-merge, by construction. **Homework counts when it's MERGED** —
which means reviewed — all semester.

## 4 · Everyone: clone

Every member, on their own machine:

```bash
git clone ⟨your-team-repo⟩ && cd ⟨your-team-repo⟩
```
