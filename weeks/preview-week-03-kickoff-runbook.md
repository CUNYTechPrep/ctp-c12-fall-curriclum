# Code Kickoff Runbook — Week 3 Project Time

*The night your data model stops being a drawing. ~35 minutes of project
time, one shared screen per team, everyone's editor open.*

**The bar, stated once:** by the end of tonight, your team's **first
migration is MERGED through the gate** and the app boots against your
schema. That's it. Seed data and first endpoints are this week's homework —
if you get to them tonight, you're ahead, not on schedule.

**What "blank" means:** your repo's `main` has an *empty* Prisma schema —
generator and datasource only, no models. Nothing gets replaced tonight;
something gets born. (The full worked example lives on the course
starter's `example/todo` branch — your week-1 work-block clone — whenever
you want to see what a finished schema looks like; that's what your jigsaw
expert studied.)

---

## Before class (strongly encouraged)

Turn your week-2 entity sketch into a draft `schema.prisma` — on a branch,
not merged. Teams that arrive with a draft spend tonight *reviewing* instead
of *typing*. Your data-layer expert from the jigsaw is the natural driver;
the design-review feedback you got from your TA is the punch list.

## The ritual (one driver, everyone reading)

**1 · Branch.**

```bash
git switch -c kickoff/schema
```

**2 · Write the schema.** Open `packages/db/prisma/schema.prisma` and turn
the sketch into models. Ground rules from the design review apply: every
user-owned table carries the owning id; name your relations; timestamps
(`createdAt`/`updatedAt`) on everything; start with fewer models than you
think you need — migrations exist so you can add more next week.

**3 · Generate the migration.** From `packages/db/`:

```bash
npx prisma migrate diff --from-empty \
  --to-schema-datamodel prisma/schema.prisma --script \
  > prisma/migrations/0001_init.sql
```

**4 · Read the SQL.** The whole team, on the shared screen, before anything
runs. This is tonight's actual lesson: the migration *is* the schema, in the
language the database speaks. Can you point at the line that creates each
table from your sketch? The line that enforces each relation?

**5 · PR it, review it, merge it.** Push the branch, open the PR, and a
teammate who is *not* the driver reviews it in hunk — the checklist applies
to migrations too (pull it, run it, read it, ask one real question: "why is
this field optional?" is a great one). Merge through the gate. **Kickoff is
done when this PR is merged.**

**6 · While the review is fresh:** write section 4 of your team charter —
what blocks an approval, expected response time on review requests,
comment conventions. The charter is this week's homework
([charter-guide.md](../charter-guide.md)); ten minutes on section 4 tonight
saves a week-6 argument.

## This week's homework

**The team charter** — template, worked example, and how-to in the repo
root; merged into your team repo by the week-4 session. Plus the
follow-through: the demo seed (idempotent — model it on the example
branch's seed) and your first scoped queries, each as a gated PR. By next
session Prisma Studio should show your product's demo data.

## When it goes sideways (TA triage)

| Symptom | Fix |
|---|---|
| Migration SQL looks wrong / missing a table | The schema and sketch disagree — fix `schema.prisma`, regenerate (step 3 overwrites; that's fine pre-merge) |
| `pnpm dev` boot fails on migration | Read the error — it names the SQL line. Usually a bad enum value or a self-referencing relation missing `@relation` names |
| Old example data haunting the local db | `pnpm db:reset`, restart dev |
| Team still arguing about the model at :20 | Merge the *smallest defensible version* tonight; model the disputed entity next week as migration 0002. Shipping a small migration beats debating a big one |
| Merged after class instead of in class | Fine. The bar is merged *this week*, tonight is the target — log it in the TA thread either way |

## TA end-of-night checklist

- [ ] Every team: `kickoff/schema` PR exists
- [ ] Merged (or a concrete blocker logged in the TA thread with an owner)
- [ ] Reviewed by a non-driver teammate, in hunk
- [ ] `/api/health` green against the new schema on at least the driver's machine
- [ ] Charter section 4 (review norms) drafted; whole charter due by wk 4
