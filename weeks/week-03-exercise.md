# Code Kickoff Runbook — Week 3 Project Time

*The night your data model stops being a drawing. ~35 minutes of project
time, one shared screen per team, everyone's editor open.*

**The bar, stated once:** by the end of tonight, your team's **first
migration is MERGED**

---

## Team coding! (One driver, everyone contributing)

**1 · Branch.**

```bash
git switch -c kickoff/schema
```

**2 · Write the schema.** Open `packages/db/prisma/schema.prisma` and turn
the sketch into models. Ground rules from the design review apply: every
user-owned table carries the owning id; name your relations; timestamps
(`createdAt`/`updatedAt`) on everything; start with fewer models than you
think you need — migrations exist so you can add more next week.

**3 · Generate the migration.** From `packages/db/` — the skeleton has no
`migrations/` folder yet, so create it first:

```bash
mkdir -p prisma/migrations
npx prisma migrate diff --from-empty \
  --to-schema-datamodel prisma/schema.prisma --script \
  > prisma/migrations/0001_init.sql
```

**4 · Read the SQL.** The whole team, on the shared screen, before anything
runs. This is tonight's actual lesson: the migration *is* the schema, in the
language the database speaks. Can you point at the line that creates each
table from your sketch? The line that enforces each relation?

**5 · Run it.** From the repo root; stop `pnpm dev` first if it's running.

```bash
pnpm prisma:generate     # regenerate the typed client
pnpm db:reset            # deletes .pgdata — local db starts over
pnpm dev                 # boot — the db-server applies migrations/ on startup
```

The db-server logs `applied migration 0001_init.sql`; `/api/health` says
`db:"ok"` (it did on the empty schema too — the log line is the proof).
To see the tables, from `packages/db/`:

```bash
DATABASE_URL=postgresql://postgres:postgres@127.0.0.1:5433/postgres npx prisma studio
```

Empty tables with your names on them. That's your product.

**6 · PR it, review it, merge it.** Push the branch, open the PR, and a
teammate who is *not* the driver reviews it in hunk — the checklist applies
to migrations too (pull it, run it, read it, ask one real question: "why is
this field optional?" is a great one). Merge through the gate. **Kickoff is
done when this PR is merged.**

**7 · While the review is fresh:** write section 4 of your team charter —
what blocks an approval, expected response time on review requests,
comment conventions. The charter is this week's homework
([charter-guide.md](../charter-guide.md)); ten minutes on section 4 tonight
saves a week-6 argument.

## This week's homework

**The team charter** — template, worked example, and how-to in the repo
root; as a canvas in your team channel by the week-4 session.

**The Reflection** - What are the advantages and disadvantages of using Prisma over SQL for database interaction?

**Watch Ahead** - The watchahead playlist for this week and reflection
