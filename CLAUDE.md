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
- **`*-viewer.html`** (not yet implemented for SFD) — would reformat an
  *incoming* Winlink message back into a readable HTML view. Reference
  examples showing the Initial/Viewer split exist under `WAforms/` (e.g.
  `WA Field Situation Report Initial.html` / `...viewer.html`), which were
  pulled in as design references, not code owned by this project.
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
  back into the form) is a known TODO — see `SFD/acs_issues.md` and
  `SFD/issue1.md` for the current punch list of unfinished features.

To add or modify a form field: update the `name=`/`id=` in the `-form.html`
input, add a matching `<var name>` in the corresponding `-template.txt`, and
(if the Save/Load JSON structure is affected) update the field list inside
`createJsonObjectFromFormElements()` in the form's inline script.

## Installing/testing a form in Winlink

Copy the form's `-form.html` and `-template.txt` into Winlink Express's
`Global Folders\Templates` directory, then compose a new message and select
the template under "General Templates". Outside Winlink, the HTML file can be
opened directly in a browser to test layout and the Save Data button, but
Submit/Load will not function (see `SFD/README.md` for the live-hosted
example link and full user-facing instructions).

## Site generation

`SFD/myst.yml` configures a [MyST](https://mystmd.org/) build (`SFD/index.md`
→ `SFD/_build/`) for the SFD subsection of the docs site; the root site uses
plain docutils-rendered HTML (`index.html` from `index.md`). Treat
`SFD/_build/` as generated output, not something to hand-edit.
