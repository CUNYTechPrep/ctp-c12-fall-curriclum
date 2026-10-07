# Week 6 Homework — Build One Accessible Component

*Assigned the week-6 session · one interactive piece your product has or
will need · an accessibility brief, then the component, built to a known
pattern · a gated PR, reviewed by a teammate, merged before the week-7
session · no page for it yet? build it in a sandbox route*

# tl;dr

Find the one interactive piece of your product that would be hardest to
use without a mouse or without eyes. It can be something you've built or
something that's next on your board. Write down who it could block and
what they'd need, look up the standard pattern for it, then build it that
way and prove it works with a keyboard and a screen reader. One
component, done right, and the brief that explains why it's right.

## Pour the curb cut with the sidewalk

Walk to any corner in the city and look down. That little ramp where the
sidewalk meets the street is a curb cut. They were fought for by
wheelchair users, and then everybody started using them: strollers,
suitcases, delivery carts, you on a bad knee.

Here's the part that matters for us. A curb cut poured with the sidewalk
costs almost nothing. A curb cut added later means a jackhammer, a
permit, and a closed street. Tonight's audit was the jackhammer: we found
the todo app's problems after they shipped and fixed them one by one.
This week you pour concrete instead. You pick a component before (or
while) you build it, figure out who it could leave out, and build it so
it doesn't.

Most of that work is cheap, if you do it first. A `<button>` instead of a
`<div>`, a visible label, a name that works alone. The expensive problems
are the ones you didn't think about until a user found them.

## Now yours

**1 · Find your challenge.** Look at your product's MVP features in the
charter and at what's next on your board, then pick the interactive
piece that's hardest to use without a mouse or without eyes. This table
covers most of what CTP products need:

| If your product has… | The challenge | Learn it from |
|---|---|---|
| A delete confirm, or any popup | Focus moves in, Escape closes, focus comes back | [APG Dialog (Modal)](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/) |
| A "⋯" button with actions | Arrow keys inside, Escape out | [APG Menu Button](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/) |
| Tabs, or a segmented filter | One tab stop, arrows between tabs | [APG Tabs](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/) |
| Show/hide details | Saying open or closed | [APG Disclosure](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/) |
| An on/off setting | Switch or checkbox, and its state | [APG Switch](https://www.w3.org/WAI/ARIA/apg/patterns/switch/) |
| Search with suggestions | The hardest one on this list | [APG Combobox](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/) |
| A form with errors | Labels, and errors you can hear | [WebAIM: Forms](https://webaim.org/techniques/forms/) + tonight's slides |
| Toasts, "Saved!", live updates | A live region that exists first | [APG Alert](https://www.w3.org/WAI/ARIA/apg/patterns/alert/) + tonight's slides |
| Icon-only buttons (trash, edit) | A name, when there's no text | [APG Button](https://www.w3.org/WAI/ARIA/apg/patterns/button/) |
| Images, avatars, charts | A text alternative that says what matters | [WebAIM: Alt Text](https://webaim.org/techniques/alttext/) |
| Drag to reorder | A way to do it without dragging | WCAG 2.5.7 Dragging Movements |

If your piece isn't on the list, the [APG pattern
index](https://www.w3.org/WAI/ARIA/apg/patterns/) has 30 of them. Pick
the closest one.

**2 · Write the brief first.** Before any code, in the PR description
(open it as a draft now), answer four questions:

```markdown
**The component:** delete-confirm dialog for a job
**Who it could block:** keyboard users if focus doesn't move in or come
back; screen-reader users if the dialog has no name, or the page behind
it is still readable
**The pattern:** APG Dialog (Modal), via the native <dialog> element
**Keyboard table:**
| Key | What happens |
|---|---|
| Enter on "Delete" | dialog opens, focus moves to "Cancel" |
| Tab / Shift+Tab | cycles inside the dialog only |
| Escape | closes it, focus returns to "Delete" |
```

The keyboard table is the whole spec. If you can't fill it in, you
don't understand the component yet, and that's exactly the moment to
find out.

**3 · Learn the solution from the source.** Every APG pattern page has
the same two sections that matter: *Keyboard Interaction* and *WAI-ARIA
Roles, States, and Properties*. Read both. Then read the APG's [Read Me
First](https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/), whose
headline rule is "No ARIA is better than Bad ARIA." Native elements
first. `<button>`, `<dialog>` with `showModal()`, `<details>`, and a real
`<input type="checkbox">` do most of the work before you write a single
`aria-` attribute.

Ask your AI pair too, it's good at this. Then check what it gives you
against the APG page, line by line. AI loves to sprinkle ARIA on things
that already had the right role, and every extra attribute is something
you now have to keep true.

**4 · Build it.** If the page exists, build it there. If it doesn't, give
it a page of its own: `apps/web/app/sandbox/<component>/page.tsx` is a
real route at `/sandbox/<component>` (the folder is the URL, week 5), and
you can render the component there with fake props. The component itself
goes in `components/`, where the real page will pick it up later. Use
tonight's slides as the checklist: role, name, state, focus you can see,
contrast, rem.

**5 · Prove it.** Walk your keyboard table row by row with the mouse in a
drawer. Then do one pass with VoiceOver (⌘F5) or NVDA, and paste what it
said into the PR:

```text
opened:   "Delete job? dialog. Cancel, button."
Escape:   "Delete Acme · Frontend Intern, button."   (focus came back)
```

Stretch: a test that finds your component the way assistive tech does,
by role and name, with Testing Library:
`getByRole("button", { name: "Delete Acme · Frontend Intern" })`. If that
query works, the name is real. The starter doesn't ship Testing Library
or a browser-like test environment yet, so adding them is part of the
stretch.

**6 · The review.** Same four moves. "Run it" means your reviewer walks
the keyboard table themselves, every row, and doesn't take the list's
word for it. The best real question is about focus: *where does focus go
after this?* Ask it about every row.

## Done means

The PR is merged with a teammate's review. The description has the
brief, the keyboard table, and a screen-reader transcript. Every row of
the table works for someone who isn't the author. If the component lives
in a sandbox route, there's an issue on the board for wiring it into the
real page.

## The interview question

"How do you approach accessibility?" comes up more than you'd think, and
"I'd add ARIA" is the answer that loses the room. You have a better one
now: you design it in. Pick native elements first, because they come
with roles and behavior. Write down the keyboard interaction before you
build, from a known pattern. Test with a keyboard and a screen reader,
then run axe for what you missed. Then point at this PR, because a brief
with a keyboard table is a design doc most working engineers have never
written.

## What we didn't cover

Reduced motion (`prefers-reduced-motion`), forced-colors mode, captions
for video, and full WCAG conformance are all real, all later. So is auditing
a whole product, once there's a whole product to audit. Pour one curb cut
right this week.

So, what were you about to build that someone couldn't have used? Put
your answer at the top of the brief. 🔰
