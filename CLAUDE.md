# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Winlink message forms (and supporting docs) developed by Seattle ACS (Auxiliary
Communications Service), primarily a **Seattle Fire Department Incident Card**
form under `SFD/`. There is no build system, package manager, linter, or test
suite — this is plain HTML/CSS/JavaScript and Markdown, hand-edited and tested
by opening files directly in a browser or installing them into Winlink Express.

## Branch structure (important)

- `main` — canonical source: `README.md`, `SFD/README.md`,
  `SFD/SFD_incident_card-form.html`, `SFD/SFD_incident_card-template.txt`,
  and `SFD/images/`. This is the branch to base form/content changes on.
- `gh-pages` — published GitHub Pages site (https://rjleveque.github.io/SeattleACS_wl-forms/).
  It contains everything from `main` plus generated site output
  (`index.html`, `SFD/_build/`, docutils/MyST HTML) and a lot of in-progress
  scratch material that is untracked in git (draft form variants like
  `SFD_test*/`, `SFD_incident_card-draft*/`, downloaded reference forms under
  `WAforms/`, slide decks, meeting notes). Don't assume everything present on
  `gh-pages` is meant to be permanent or reviewed — check `git status` before
  treating untracked files here as part of the project.
- When making substantive changes to the form itself, prefer working from/on
  `main` and let the maintainer decide when to sync to `gh-pages`, unless
  asked specifically to update the published site.
- `claude` — working branch used to commit Claude Code's changes, based on
  `main`, for eventual review and merge back into `main`.

## Winlink form architecture

Each Winlink custom form is a pair (sometimes trio) of files that must keep
their field names in sync:

- **`*-form.html`** — the "Initial" form: an HTML `<form>` a user fills out
  when composing a message. Every input has a `name=` attribute. Winlink
  Express posts the form data to `action="http://{FormServer}:{FormPort}"`,
  which is a Winlink-internal placeholder substituted at runtime (this is why
  Submit does nothing when the file is opened as a plain webpage — only
  Save/Load work outside Winlink).
- **`*-template.txt`** — defines how Winlink turns submitted form field
  values into the actual outgoing message: `To:`, `Subject:`, and `Msg:`
  lines with `<var fieldname>` placeholders that must exactly match the
  HTML `name=` attributes (case-sensitive). The first line
  (`Form: <initial-html>[,<viewer-html>]`) declares which HTML file(s) the
  template pairs with.
- **`*-viewer.html`** — reformats a *received* Winlink message back into a
  readable HTML view. Implemented as `SFD_incident_card-viewer.html`.
  Reference examples showing the Initial/Viewer split exist under
  `WAforms/` (e.g. `WA Field Situation Report Initial.html` /
  `...viewer.html`), which were pulled in as design references, not code
  owned by this project. Two things are easy to get wrong when wiring up a
  viewer:
  - The template's `Form:` line must list *both* files, comma-separated
    (`Form: x-form.html,x-viewer.html`), or Winlink Express will never
    attach the XML form-data file to the outgoing message, and the
    recipient's viewer will never get invoked at all.
  - Viewer files use **`{var fieldname}`** (curly braces) to insert
    submitted field values — a different syntax from the `<var fieldname>`
    used in `*-template.txt`. Both must still match the HTML `name=`
    attribute exactly (case-sensitive). This bit us once already: the
    template had `<var Status>` (capital S) referring to a radio group
    actually named `status` (lowercase), which would have silently failed
    to substitute — always grep the form's `name="..."` attributes to
    confirm exact case before adding a new `<var>`/`{var}` reference.
  - The viewer has no live `<form>` fields to read from, so any JS-driven
    formatting (checkbox glyphs, conditional bold, the status box) needs
    its inputs fed via hidden fields whose `value="{var fieldname}"` is
    substituted by Winlink at merge time, then read the same way the input
    form reads its own live fields. `SFD_incident_card-viewer.html` reuses
    the input form's print JS (`markX`, `updateStatusBox`,
    `populatePrintCard`) verbatim against those hidden fields — if the
    print layout changes in one file, update the other to match, since the
    code is duplicated rather than shared.
- `WAforms/` (downloaded, third-party forms) is a useful source of working
  patterns to copy before reimplementing something from scratch — check there
  first. Example: the `Checkifinbrowser()` pattern used consistently across
  files there (e.g. `ICS213RR_WA_Initial.html`) — a hidden
  `<input id="testCall" value="{MsgSender}">` whose value only gets
  substituted by Winlink Express at load time; if the literal string `{Msg`
  is still present, the form is being viewed in a plain browser (not
  Winlink), so the Submit button is hidden and an alert is shown. SFD's form
  now uses this same pattern.
- Checkboxes are implemented as a visible `<input type="checkbox">` paired
  with a hidden `<input type="hidden" name="...">` whose value is toggled via
  an inline `onclick` handler (there's an open question in
  `SFD/issue1.md` about doing this the more idiomatic way with
  `addEventListener`, which the author couldn't get working — see the
  `checkbox` branch for a WIP attempt).
- The "Save Data" button serializes current form values to a JSON `.txt` file
  client-side (`saveFormInputElementsToFile()` in the form's `<script>`)
  named with a timestamp, so field data can be saved offline and later
  transferred to a Winlink-connected machine. "Load Data" (reading that JSON
  back in via `loadFormInputElementsFromFile()` / `populateFormFromJsonObject()`)
  is implemented too. See `SFD/acs_issues.md` and `SFD/issue1.md` for other,
  still-unfinished items on the punch list.
- The "Print Card" button (see the dedicated section below) reproduces the
  physical card's layout as a printable page, available from both the input
  form and the viewer.

To add or modify a form field: update the `name=`/`id=` in the `-form.html`
input, add a matching `<var name>` in the corresponding `-template.txt`
(and `{var name}` in `-viewer.html` if it should show up there too), and (if
the Save/Load JSON structure or the print card is affected) update
`createJsonObjectFromFormElements()`/`populateFormFromJsonObject()` and/or
`populatePrintCard()` in the form's inline script — remembering to mirror any
`populatePrintCard()`/CSS change into `-viewer.html` as well, since that code
is duplicated, not shared, between the two files.

## Print Card feature

`SFD_incident_card-form.html` and `SFD_incident_card-viewer.html` each have a
"Print Card" button that calls `window.print()` against a hidden
`#printSheet` subtree (`.print-only`, only shown via `@media print`) styled
to mimic the physical SFD Incident Card, followed by a `#printExtra` block of
information that doesn't fit on the physical card (ACS message number,
To/Submitted-by, remarks, print timestamp in UTC and local time). Notes for
anyone touching this again:

- The physical card is **4in x 6in**. It turned out *not* to need rotating:
  an unrotated 6in-wide `#printCard` fits comfortably on an 8.5in-wide
  letter-portrait page, so `#printCard` and its wrapper `.card-rotate-wrap`
  are both sized 6in x 4in (landscape) with no `transform: rotate(...)`.
  An earlier attempt to rotate a 4in x 6in portrait card into place ran into
  print-rendering quirks (see below) and was abandoned as unnecessary — don't
  re-introduce rotation without a concrete reason.
- **`.card-rotate-wrap`'s size must exactly match `#printCard`'s (6in x
  4in).** It was briefly left at the old rotated dimensions (4in x 6in)
  after rotation was dropped; because the wrapper used
  `flex; align-items:center` to center the card, the mismatched 6in height
  silently added an invisible ~1in gap above *and* below the card. If the
  card ever appears to have unexplained extra space around it, check this
  first.
- Getting the card's position exactly reproducible on the page (e.g. so its
  bottom edge lands precisely at half the physical page height, for
  fold-and-cut instructions) requires accounting for **both** the `@page`
  margin *and* the browser's default `body` margin — add `body { margin: 0; }`
  inside `@media print` and then use a fixed top margin on
  `.card-rotate-wrap` computed from the `@page margin` value, rather than
  relying on `margin: 0 auto` / document flow, which shifts depending on
  what content precedes it.
- The print dialog's own **Scale / "Fit to page" setting is outside CSS
  control** and can silently shrink the whole printed page (diagnosed once
  by noticing the printed card was exactly 0.875x the intended size in both
  dimensions — the print dialog's Scale had been left off 100%/"Actual
  Size"). If a user reports the printed card is the wrong size but by a
  uniform ratio, check this before touching CSS.
- `overflow: hidden` combined with `transform: rotate(...)` and percentage
  based `position: absolute` offsets did not behave reliably in this
  browser's print renderer (`open` launches the OS default browser on
  macOS, likely Safari) — clipping was computed from the pre-transform box,
  not the rotated one. This is why rotation was abandoned rather than
  fixed; if it's ever revisited, expect this quirk.
- I have no browser/print-preview tool available in an agent session,
  so changes to this feature can only be verified by the human tester
  actually printing/previewing — don't claim a visual/layout result is
  correct without that confirmation.
- The multi-line card fields (`location`, `actions`, `PNBT`) each have a
  hard max-line cap enforced on input (reverting the field to its last
  valid value if a keystroke/paste would exceed the cap), plus a matching
  on-screen `max-width` (in inches, set per `#id`) chosen to approximate
  the printed box's proportions so wrapping on screen previews what will
  happen when printed. Current caps: Location 4 lines (box `max-width:
  8.25in`), Actions & Needs 9 lines (same `8.25in` width as Location),
  PNBT 2 lines (`4.125in`, half of Location's width).
- Line counting is done by a shared `countTextareaLines(textarea)` helper
  (duplicated in both files, like the rest of the print JS). It reads
  `scrollHeight` against the computed `line-height`/padding — but a
  textarea's default height is driven by its `rows` attribute, so
  `scrollHeight` alone always reports the `rows`-sized height regardless
  of how much text is actually present. The helper works around this by
  temporarily setting `textarea.style.height = '0px'` (forcing the box
  smaller than its content) immediately before reading `scrollHeight`,
  then restoring the original inline height. Setting the height to
  `'auto'` instead does **not** work for this — `'auto'` is the same
  default sizing driven by `rows`, so it doesn't force a re-measurement
  (a real bug hit and fixed this session; watch for it if this helper is
  ever copied elsewhere).
- Each of these fields uses a two-tier print font size: the normal size
  fits up to its max-minus-one line count; a smaller size (toggled via a
  `.small-text` class the JS adds to the `#print-*` element) only kicks
  in at the true max line count, so a full field never gets clipped by
  its row's fixed `@media print` height. The exact point sizes
  (Location 12pt→8pt, Actions 12pt→8pt, PNBT 13pt→10pt) were tuned
  against the maintainer's actual print-test feedback rather than pure
  CSS-geometry arithmetic — a naive pt/line-height calculation from the
  row heights predicted the 12pt baseline would already overflow at 3
  lines, but real printing showed it fits fine, so don't rederive these
  numbers from scratch without a real print test backing the change.
- The collapse risk/partial/full marks (meant to mimic "circle this
  word" on the physical card) are rendered as bold text inside a
  rounded-corner border box via a `.collapse-mark-active` class the JS
  toggles onto the `#print-collapse-*-mark` span, not literal bracket
  characters — if adding similar "mark visually" indicators elsewhere on
  the card, prefer this class-toggle + CSS border pattern over building
  the marker into the text content.

## Installing/testing a form in Winlink

Copy the form's `-form.html`, `-viewer.html` (if present), and
`-template.txt` into Winlink Express's `Global Folders\Templates` directory,
then compose a new message and select the template under "General
Templates". The viewer file also needs to be present on any *recipient's*
machine for their Winlink Express to render it. Outside Winlink, the HTML
files can be opened directly in a browser to test layout, the Save Data
button, and the Print Card button, but Submit/Load and the `{var ...}` /
`{FormServer}` substitutions will not function (see `SFD/README.md` for the
live-hosted example link and full user-facing instructions).

## Site generation

`SFD/myst.yml` configures a [MyST](https://mystmd.org/) build (`SFD/index.md`
→ `SFD/_build/`) for the SFD subsection of the docs site; the root site uses
plain docutils-rendered HTML (`index.html` from `index.md`). Treat
`SFD/_build/` as generated output, not something to hand-edit.
