---
name: reproduce-bugs
description: Reproduce reported bugs from a source thread (Slack, ticket, bug bash doc) and document them end to end in a tracker, one row/page per bug, with an exact-vs-ideal reproduction split and a triage pass. Use when the user wants to reproduce bugs from a thread or bug bash, write up bug reports in a tracker, or asks to run a bug repro/triage workflow.
---

# Bug tracker & reproduction workflow

Product-agnostic workflow for turning a raw bug report into a reproduced, triaged, written-up tracker entry.

This skill carries no product config of its own. Before starting Phase 2, resolve the product config per [CONFIG-TEMPLATE.md](CONFIG-TEMPLATE.md) — look for one that already exists before asking the user to restate anything. Don't proceed to Phase 2 on assumed defaults.

## Phase 1 — Collect reports

Pull every distinct bug mentioned in the source thread/doc. For each one, capture before doing anything else:
- what the reporter said, **verbatim** (don't paraphrase, it loses detail that matters for repro)
- who reported it, and their device/OS/browser if stated or inferable from a screenshot
- any screenshot or recording attached

Create one tracker row per bug at this stage, even before attempting reproduction. Leave reproduction fields blank until Phase 2 is done.

## Phase 2 — Reproduce

Try to reproduce each bug for real, not just corroborate from the reporter's own screenshot.

**A. Environment.** Follow the login/credentials path confirmed for this product. If it isn't confirmed yet, ask before proceeding rather than guessing.

**B. Device.** You'll usually need an emulator/simulator (or a resized desktop browser) to get a real screenshot rather than trusting the reporter's own. Set one up per [EMULATORS.md](EMULATORS.md) (Android, iOS, desktop) before attempting reproduction. Don't skip straight to "not reproducible" for lack of a device.

**C. Exact vs ideal.** Test both, and label the result clearly as one or the other:
- **Exact**: the reporter's actual device, OS, browser, and install state
- **Ideal**: the officially supported configuration for this product

These can genuinely disagree. When they do, document both, don't pick one as "the" answer.

Capture a real screenshot from the device/emulator/simulator you actually tested on, never a placeholder or the reporter's original image.

If reproduction fails, don't force it. Record that it wasn't reproduced and move on. Best-effort only.

**D. Learn.** If reproducing this bug surfaced an environment fact not already in the product config — a device with no matching emulator profile, a login step that behaves oddly, test data with a quirky known behaviour, anything a future run on this product would otherwise have to rediscover — write it back into that product's config now, per [CONFIG-TEMPLATE.md](CONFIG-TEMPLATE.md). That fact belongs there, never in this skill's own files.

## Phase 3 — Write it up

Every bug gets its own tracker page, following the page template in [REFERENCE.md](REFERENCE.md#page-template). Consistency matters more than any individual field: if the template changes, update every existing page to match.

## Phase 4 — Triage

Fill in root cause (if findable), options to fix, severity, effort, and a triage priority. These are judgement calls, not something to automate away, flag your reasoning so it can be challenged.

## Phase 5 — Cleanup

Tear down anything spun up for reproduction — kill emulator/simulator processes, close opened browser tabs or dev servers. See [REFERENCE.md](REFERENCE.md#cleanup). A run isn't done while it's left stray processes running.

## Known gotchas

Structural limitations, not one-off mistakes, worth checking before assuming the app itself is broken or wasting time working around them. See [REFERENCE.md](REFERENCE.md#known-gotchas) (Slack image extraction, evidence upload) and [EMULATORS.md](EMULATORS.md) (device-specific gotchas: PWA install caveats, coordinate scaling, icon placement, device substitution, clock correlation, non-native form inputs, native dialogs).
