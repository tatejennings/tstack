---
name: tstack-product
description: Turns a completed business brief into docs/PRODUCT.md — the product requirements doc with features, user flows, data models, and acceptance criteria (Given/When/Then for deterministic features, eval-based for AI/LLM features, measurable thresholds for performance). Use when docs/1 - Discovery/business-brief.md exists and the user wants product requirements, says "let's write the PRD", or asks what to build next after discovery. Do not use to add features to an existing PRODUCT.md — that's tstack-specify-feature. Input is the business brief; output is docs/PRODUCT.md. Hands off to tstack-design for UI products (then tstack-architect), or straight to tstack-architect for headless products.
---

# tstack-product

You are running TStack's product-requirements stage. You read the business brief and translate it into a precise, testable PRODUCT.md that defines *what* gets built and *why*. Every downstream technical doc derives from this one — so be specific, not aspirational.

## Prereq check

Before doing anything else, check the inputs:

```
docs/_adopted/PRODUCT.draft.md   ← adoption path (check first)
docs/1 - Discovery/business-brief.md   ← normal path
```

**Adoption path:** if `docs/_adopted/PRODUCT.draft.md` exists, the user adopted an existing project via `tstack-ingest`. Author PRODUCT.md from that draft instead of (or in addition to) the brief — see the **adoption-aware branch** in step 1. You are still the sole author of `docs/PRODUCT.md`; the draft is unratified input, not a finished doc.

**Normal path:** require `docs/1 - Discovery/business-brief.md`. If it's missing *and* there's no adopted draft: stop and tell the user to run `tstack-discover` first (or `tstack-ingest` if they already have docs).

If `docs/PRODUCT.md` already exists: stop and ask if they want to (a) replace it (run me), or (b) add a feature to it (run `tstack-specify-feature` instead — that's the right tool for iteration). Default to (b).

## Approach

1. **Read the input from disk** and summarize what you understood: the product, target user, core value prop. Flag any gaps that will block requirements writing — unclear features, missing user flows, ambiguous scope. Resolve those with the user before writing.

   **Adoption-aware branch.** If `docs/_adopted/PRODUCT.draft.md` exists, read it *first* — it carries content `tstack-ingest` distilled from the user's own docs, plus a visible `## Open gaps` list. Treat the draft as **unratified input, not a finished doc**: confirm and extend its content through this skill's full gate (steps 2–5) rather than re-interviewing from scratch, and treat every entry in its Open gaps list as a required item to resolve with the user before save (a soft criterion, a missing data model, an AI feature with no eval set). Where a brief also exists, reconcile the two. **After PRODUCT.md is saved and signed off, delete (or archive) `docs/_adopted/PRODUCT.draft.md`** so the quarantine doesn't linger as stale state — note the removal in the save commit.

   **Flag AI/LLM/ML features now, not later.** As you read, identify every feature whose behavior is non-deterministic — chat, recommendations, classification, generation, search relevance, anything model-driven. Name them explicitly back to the user ("X and Y look like AI features"). This matters because those features take **eval-based** acceptance criteria (step 4), not Given/When/Then, and `tstack-architect` will need an AI-strategy ADR for them downstream. Catching them here prevents a soft criterion like "suggestions feel relevant" from slipping through to `tstack-build`, where it can't be verified.

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

4. **Acceptance criteria are mandatory.** Every feature ends with testable criteria. Pick the format that matches the feature's behavior:

   - **Deterministic features** (CRUD, business rules, validation, navigation): use Given/When/Then.
     > Given a logged-in user with no projects, when they visit /projects, then they see the empty state with a "Create project" CTA.

   - **AI / LLM / ML / non-deterministic features** (chat, recommendations, classification, generation, search relevance): use eval-based criteria. Three required parts:
     1. **Eval set:** A defined, versioned set of inputs (e.g., "the 100-question support-question benchmark in `evals/support-v1.jsonl`")
     2. **Quality bar:** A measurable target on that set (e.g., "≥85% top-1 accuracy graded against the labeled answers" / "human-rated relevance ≥4.0/5 averaged over the set" / "latency p95 ≤ 2.5s")
     3. **Fallback behavior:** What the feature does when the model fails, times out, returns low-confidence, or trips a content filter. This is deterministic and gets its own Given/When/Then.

     Example AI acceptance criterion:
     > **Eval set:** `evals/recipe-suggestions-v1.jsonl` (50 prompts, hand-labeled ideal outputs).
     > **Quality bar:** ≥80% of suggestions rated "relevant or better" by the grader prompt in `evals/grader.md`.
     > **Fallback:** On model error or timeout >5s, show "We couldn't generate suggestions right now — here are popular recipes" with the trending list.

   - **Performance/latency features**: use measurable thresholds (e.g., "search returns in ≤200ms p95 over 10k indexed documents").

   If you can't write criteria in any of these forms, the feature isn't specified yet — go back and clarify. Soft criteria like "feels fast" or "feels relevant" do not count.

5. **Final review** before saving:
   - Every feature has acceptance criteria in the appropriate format (Given/When/Then, eval-based, or measurable threshold)
   - Every consumer-facing feature includes an explicit accessibility criterion (e.g., "fully keyboard-navigable; meets WCAG 2.1 AA on contrast/labels/focus order")
   - Every feature that stores user data states its retention and deletion behavior
   - Every AI feature has a defined eval set, quality bar, and fallback
   - Every data model is defined (entities, relationships, field types)
   - Scope boundaries are explicit
   - Edge cases and error states are documented (including rate-limit, abuse, multi-tenancy isolation if applicable)
   - UI/UX descriptions are specific enough for a developer to implement from

   Then **get an explicit completion sign-off before saving.** Present the assembled PRODUCT.md (or a section-by-section recap) and ask directly: "Is this complete, or is anything missing?" Don't treat reaching the last section as done.
   - If the user names a missing feature or flow → iterate within PRODUCT.md; add it and re-run the checklist. Stay in this skill.
   - If the gap reveals the *brief itself* is wrong or incomplete (a whole audience, market, or value-prop was never explored) → stop and send them back to `tstack-discover` to fix the brief, rather than inventing product direction here.
   Only save once the user confirms it's complete.

6. **Save to `docs/PRODUCT.md`.** Commit: `docs: create PRODUCT.md from business brief`. **Adoption path:** also remove `docs/_adopted/PRODUCT.draft.md` in the same commit and use `docs: author PRODUCT.md from adopted draft` instead. (Leave any `docs/_adopted/*-notes.md` staged for the architect stage — don't delete those here.)

## What PRODUCT.md should NOT contain

- Implementation details (that's `tstack-architect`'s output)
- API endpoint definitions (that's API.md, written next)
- Code patterns (that's CONVENTIONS.md, written next)

If the user pushes you to add architecture or APIs here, redirect — those belong in `tstack-architect`.

## Reference handoff

For the full template, troubleshooting (thin requirements, scope creep, missing acceptance criteria), and section-by-section guidance, read `references/full-guide.md`.

For a realistic PRODUCT.md excerpt showing the expected level of detail — including a deterministic + AI mixed-acceptance-criteria example — read `references/example-output.md`.

## Handoff

> PRODUCT.md complete — committed to `docs/PRODUCT.md`.
>
> **Next:**
> - **UI product** (PRODUCT.md §6 lists real consumer-facing screens) → run **`tstack-design`** to design the screens and token set, *then* `tstack-architect` — so the frontend-stack ADR and ADR-3 (accessibility) come out design-informed instead of guessed.
> - **Headless product** (CLI / library / API — no UI) → go straight to **`tstack-architect`** (no design step).
>
> **Fresh session** if this is a larger project — rough rule: **8+ features, 5+ data entities, or more than one workstream** (e.g., web + mobile). `tstack-architect` (and `tstack-design`) produce several docs and benefit from a clean context budget. For a small single-domain project, continuing here is fine.
