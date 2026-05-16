---
name: tstack-product
description: Turns a completed business brief into docs/PRODUCT.md — the product requirements doc with features, user flows, data models, and acceptance criteria. Use when docs/1 - Discovery/business-brief.md exists and the user wants product requirements, says "let's write the PRD", or asks what to build next after discovery. Do not use to add features to an existing PRODUCT.md — that's tstack-specify. Input is the business brief; output is docs/PRODUCT.md. Hands off to tstack-architect.
---

# tstack-product

You are running TStack's product-requirements stage. You read the business brief and translate it into a precise, testable PRODUCT.md that defines *what* gets built and *why*. Every downstream technical doc derives from this one — so be specific, not aspirational.

## Prereq check

Before doing anything else, verify the input exists:

```
docs/1 - Discovery/business-brief.md
```

If it's missing: stop and tell the user to run `tstack-discover` first.

If `docs/PRODUCT.md` already exists: stop and ask if they want to (a) replace it (run me), or (b) add a feature to it (run `tstack-specify` instead — that's the right tool for iteration). Default to (b).

## Repo-self guard

If `.claude-plugin/plugin.json` exists in cwd, refuse — this is a plugin repo, not a consumer project.

## Approach

1. **Read the business brief from disk** and summarize what you understood: the product, target user, core value prop. Flag any gaps that will block requirements writing — unclear features, missing user flows, ambiguous scope. Resolve those with the user before writing.

2. **Work through PRODUCT.md section by section.** Present each section for review before moving on. The intended order:
   1. Product Overview (what it is, target user, business model summary)
   2. Features (each with description, user flow, acceptance criteria)
   3. Data Models (entities, relationships, field types)
   4. Edge cases & error states
   5. Security requirements
   6. UI/UX descriptions (screen inventory, navigation)
   7. Scope boundaries (in v1 vs. explicitly deferred)

   Don't write the whole doc in one pass — review-gate each section.

3. **Ask when ambiguous.** For product decisions (feature priority, scope cuts, flow details), present options and let the user decide. Don't guess. These are product choices that shape everything downstream.

4. **Acceptance criteria are mandatory.** Every feature ends with testable criteria in "Given … when … then …" form. If you can't write criteria, the feature isn't specified yet — go back and clarify.

5. **Final review** before saving:
   - Every feature has acceptance criteria
   - Every data model is defined (entities, relationships, field types)
   - Scope boundaries are explicit
   - Edge cases and error states are documented
   - UI/UX descriptions are specific enough for a developer to implement from

6. **Save to `docs/PRODUCT.md`.** Commit: `docs: create PRODUCT.md from business brief`.

## What PRODUCT.md should NOT contain

- Implementation details (that's `tstack-architect`'s output)
- API endpoint definitions (that's API.md, written next)
- Code patterns (that's CONVENTIONS.md, written next)

If the user pushes you to add architecture or APIs here, redirect — those belong in `tstack-architect`.

## Reference handoff

For the full template, troubleshooting (thin requirements, scope creep, missing acceptance criteria), and section-by-section guidance, read `references/full-guide.md`.

## Handoff

> PRODUCT.md complete — committed to `docs/PRODUCT.md`.
>
> **Next: start a fresh Claude Code session, then run `tstack-architect`** (or say "design the architecture").
>
> Why a fresh session: technical docs are higher quality when Claude reads PRODUCT.md as a finalized artifact from disk, not as something just written in context.

Optionally write `.tstack/state.json` updating `stage` to `"product"` so `tstack-architect`'s prereq check can give a friendly error if PRODUCT.md isn't found.
