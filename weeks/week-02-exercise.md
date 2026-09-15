# Week 2 Project Time — Your Product's Relations, on Paper First

*~15 minutes in class, finished as part of this week's homework · your
team, one sheet of paper (or one shared canvas) · started the night your
team forms*

Your team just adopted a pitch. Before anyone types `schema.prisma`,
draw what the product *is*. Paper first, on purpose: it's faster to erase,
easier to argue with, and nobody anchors on the first name typed.

---

## 1 · Nouns first

Say the product out loud in one sentence (it's on your adopted pitch).
Underline the nouns. Those are your candidate **entities** — most products
at your stage have 3 to 6. More than 8? You're designing v2; cross some
out (that's Week-9 pitch fodder).

## 2 · One card each

For each entity: its name (singular), and 3–5 fields you're *sure* about.
Skip timestamps and ids — they're free. If you can't name three fields,
it might not be an entity; if a field is a list, it might be an entity.

## 3 · Draw the lines

Connect the entities that relate, and label each line with its
cardinality, in words: "a Trip *has many* Stops," "a Stop *belongs to one*
Trip." Every line must survive being read aloud in a sentence that makes
sense. The line everyone argues about is the most valuable one on the
page — that argument is design review, happening early, for free.

## 4 · The ownership question

Circle the entity that answers: *whose data is this?* Almost every product
scopes its data to a user or a team — one circle, one arrow from it to
what it owns. You met this rule in the starter (the line that decides
whose todos a query returns). Your product has the same line; draw it now.

## 5 · Predict one problem

Before you leave: each member names one place this model might be wrong —
a relation that could go many-to-many, an entity that might split, a field
that might be its own table. Write them in the margin. You'll check them
against real queries in week 3.

---

## Done means

A photo of the sheet posted in your team channel tonight. It doesn't need
to be right — it needs to exist, because this week's homework (**the
schema doc**: entities → draft Prisma models, PR'd into your team repo)
starts from exactly this page, and the design review will read both.
