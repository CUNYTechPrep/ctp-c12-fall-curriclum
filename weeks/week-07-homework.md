# Week 7 Homework — One Honest Async Screen

*Assigned the week-7 session · one screen of your product that reads from
your own data and writes back to it · loading, empty, error, and no
double-submit · a gated PR, reviewed by a teammate, merged before the
week-8 session · no page for it yet? build it in a sandbox route*

# tl;dr

Pick one screen of your product that reads something and changes
something. Make it honest about time: it says when it's waiting, says
when there's nothing yet, says when something failed (and keeps what
the user typed), and never does the same thing twice because someone
clicked twice. Then prove all of that by slowing the network, stopping
the server, and double-clicking on purpose.

## The waiter who disappears

You order. The waiter walks into the kitchen, and the door swings shut.
Now you don't know anything. Is the food coming? Did they hear you? Is
the kitchen even open? So you flag down a second waiter and order again,
and twenty minutes later two plates show up and you're paying for both.

Every `fetch` is a trip to the kitchen. The browser sends the order and
the door swings shut, and for some amount of time nobody knows anything.
An honest screen is a good waiter. It says the order's in (loading). It
says the kitchen doesn't have anything for you yet (empty). It comes
back and says they're out of salmon, and it doesn't throw away your
order while it tells you (error, input kept). And when you wave at it
twice, it doesn't put the order in twice (no double-submit).

Remember the todo app's "Adding…" button that stayed stuck forever when
the server died? That's the waiter who never came back out. This week
you build the one who does.

## Now yours

**1 · Pick the screen.** One that reads (a list, a detail page, a
dashboard) and writes (a create form, a toggle, a delete). Last week's
create form plus the list it adds to is the obvious pair. If the screen
doesn't exist yet, use a sandbox route,
`apps/web/app/sandbox/<screen>/page.tsx`, same as last week.

**2 · Decide where the read happens, and say why.** You've now seen both:

- **On the server**, in a Server Component, with `loading.tsx` or a
  `<Suspense>` boundary for the waiting part. This is the default, and
  it's what Callback does. The data arrives with the HTML.
- **In the browser**, in a client component, when the data depends on
  something only the browser knows: what the user just typed, a filter,
  a timer, a live connection.

Pick one for your screen and write one sentence in the PR about why.
"Because that's how the AI wrote it" isn't a reason. It's a thing to
check.

**3 · Write the four states before you build them.** Extend your week-5
state diagram, or start a new one. Every read has *loading*, *empty*,
*error*, and *has data*. Every write has *idle*, *pending*, *failed*, and
*done*. Write down what the user sees in each one. The starter's own
standard, in `docs/specs/web.md`, is a good bar to copy: a skeleton for
loading, an empty state with a primary action, and an error with a way
to retry.

**4 · Build it.** Some pieces that do most of the work:

- Next.js gives you `loading.tsx` and `error.tsx` per route folder.
  `error.tsx` has to be a client component, and it gets a `reset`
  function for the retry button.
- For writes through a Server Action, React's `useActionState` hands you
  an `isPending` flag, and `useFormStatus` gives the same `pending` to a
  submit button inside the form. Disable the button while pending, and
  that's most of your double-submit protection.
- If a write fails, the form keeps what the user typed. Clearing inputs
  happens on success, never before.
- A `fetch` that never comes back is an error, not a loading state that
  lasts forever. Catch it.

**5 · Prove it, three ways.** Each one goes in the PR as a screenshot or
a short screen recording:

1. **Slow it down.** DevTools → Network → throttle to *Slow 4G*, then
   reload. You should see your loading state, not a blank page.
2. **Break it.** Stop the dev server, then submit. You should see your
   error state, with the user's input still there.
3. **Double it.** Click submit twice, fast. Then press Enter twice. You
   should get one row, not two.

If any of the three surprises you, that's the most valuable part of the
assignment. Write down what you expected and what happened, then fix it.

**6 · The review.** The four moves again, and for this PR "run it" means
the reviewer does all three proofs themselves. The best real question
is about failure: *what happens if this request never comes back?*

## Done means

The PR is merged with a teammate's review. The screen shows a loading
state, an empty state, and an error state that keeps the user's input.
Double-clicking does nothing extra. The PR description has the
one-sentence answer to "server or browser, and why," plus the three
proofs.

## The interview question

"How do you handle loading and error states?" Every frontend interview
asks some version of this, and "I add a spinner" is the answer that ends
the conversation early. You have a better one now. Every async thing has
four states, and you design all four before you build. Reads happen on
the server unless the browser knows something the server doesn't. Writes
disable while pending, keep input on failure, and never run twice. And
you test it by breaking it on purpose. Then point at this PR.

## What we didn't cover

Optimistic updates, where the screen changes before the server answers
and rolls back if it fails (React's `useOptimistic` is built for this).
Client-side data libraries like TanStack Query, with caching,
deduplication, and refetching. Streaming a slow part of a page behind a
fast one. All real, all worth it once your screen is honest without
them. A waiter who's fast but sometimes lies is worse than one who's
slow and always tells you the truth.

So, which of the three proofs broke your screen first? Put it at the top
of the PR. 🔰
