---
name: tstack-product
description: Turns a completed business brief into docs/PRODUCT.md — the product requirements doc with features, user flows, data models, and acceptance criteria (Given/When/Then for deterministic features, eval-based for AI/LLM features, measurable thresholds for performance). Use when docs/1 - Discovery/business-brief.md exists and the user wants product requirements, says "let's write the PRD", or asks what to build next after discovery. Do not use to add features to an existing PRODUCT.md — that's tstack-specify. Input is the business brief; output is docs/PRODUCT.md. Hands off to tstack-architect.
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

6. **Save to `docs/PRODUCT.md`.** Commit: `docs: create PRODUCT.md from business brief`.

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
> **Next: run `tstack-architect`** (or say "design the architecture").
>
> Fresh session recommended for non-trivial projects — `tstack-architect` produces several docs and benefits from a clean context budget. For small projects, continuing here is fine.
