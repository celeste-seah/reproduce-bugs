# Product config template

A living artifact, not a one-time form. Resolve it in this order before Phase 2 (see [SKILL.md](SKILL.md)):

1. **Look first.** Search near the product's bug tracker for something matching the schema below — a sibling page, a block on the tracker's parent page, wherever this product's config was last stored. If found, use it as-is.
2. **Ask only for gaps.** Anything genuinely missing, ask the user for. Don't make them restate what's already documented.
3. **Persist it.** If none existed, create one before finishing this run, so the next run on this product can skip straight to step 1.

Store it somewhere discoverable from the tracker alone, and note where, so "look first" is actually fast next time.

```md
# Product config: <product name>

- **App under test**: <URL(s) — staging/prod/local as relevant>
- **Official supported config**: <device/OS/browser/install state considered "correct">
- **Login / credentials path**: <how to authenticate as a test user — decision tree if it branches by scenario (e.g. real call vs no call, new branch vs staging), including where OTP/2FA comes from in each case>
- **Tracker**: <link to the bug tracker, and its schema — property/field names if they differ from the generic set in REFERENCE.md#page-template>
- **Type / category options**: <the product-specific values for the tracker's classification field, if it has one>
- **Known environment quirks**: <facts discovered during past repro runs that would otherwise need rediscovering each time — a test number's actual behaviour, a login step that runs twice, a device with no matching emulator profile, and so on>
```

Keep entries short. If a field doesn't apply to the product, say so explicitly rather than leaving it blank, so it reads as confirmed-absent rather than not-yet-filled-in.

## Write back what you learn

Every product-specific fact belongs here, never in this skill's own files (SKILL.md, EMULATORS.md, REFERENCE.md). Those stay usable for any product; this file is where one product's particulars accumulate. When Phase 2D turns up something new, append it to **Known environment quirks** — dated if it might go stale — rather than letting it evaporate at the end of the session.
