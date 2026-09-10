# Reference

## Page template

Every bug page follows this structure:

- **Summary** (callout) — one-line description of the bug
- **Reported by** / **Reproduced on** (side by side, as columns)
  - *Reported by*: name, guessed OS/device + reasoning, link to the original message, and the reporter's **original comment quoted verbatim as its own line** (not folded into your own sentence)
  - *Reproduced on*: split into as many columns as needed, exact-match column(s) and ideal/supported-case column(s), each stating pass/fail and linking a real screenshot
- **Root cause**
- **Options to fix**
- **Severity / Effort / Triage** — one-line restatement of the tracker's own property values, so the page stands alone without opening the row's properties

**Generic fields** (reusable for any product): Reported by, Screenshots, Reproduced on (exact + ideal), Root cause, Options to fix, Severity, Effort, Triage, Status.

**Product-specific values** live in a `Type` (or equivalent) field's options, read these off the tracker's own schema for the product at hand, don't invent them.

## Evidence upload cookbook

The same four-step pattern works for attaching a real screenshot to a tracker page, regardless of tracker platform:

1. **Upload** the file to wherever the tracker lives (a two-step create-then-send upload, a direct attach call, whatever that platform's API offers).
2. **Get back an embeddable reference** — the upload step should hand you something insertable into the page content (a file ID, an embeddable URL, a markdown snippet).
3. **Embed it** in the write-up at the right spot in the page template (next to the reproduction steps it's evidence for).
4. **Replace the placeholder** — a targeted content edit swapping out "not yet reproduced" text for the real write-up plus embed, not a full-page rewrite.

Do this once per screenshot that matters to the write-up; don't attach every intermediate screenshot taken along the way.

## Cleanup

Before calling a repro run finished, tear down anything spun up to do it: kill emulator/simulator processes, close browser tabs or dev servers you opened. Leaving these running doesn't break anything immediately, but it accumulates across runs and makes it harder to tell a stray process from one still in use.

## Known gotchas

Structural limitations worth knowing up front, so you don't waste time working around them as if they were one-off mistakes:

- **Slack images can't be extracted.** There's no tool path to pull raw image bytes/URLs out of a Slack thread, only to view them inline. If you need a real screenshot, reproduce the bug yourself and capture your own.

Device/emulator-specific gotchas (coordinate scaling, icon placement, device substitution, clock correlation, non-native inputs, PWA install caveats, native dialogs) live in [EMULATORS.md](EMULATORS.md), since they're tied to the device setup itself rather than the tracker/writeup side of the workflow.
