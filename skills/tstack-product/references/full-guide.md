# Product Requirements Guide — Creating PRODUCT.md

> **Source of truth:** `SKILL.md` is authoritative for the *process* of this stage (section order, early AI-feature detection, the three acceptance-criteria formats, the completion sign-off gate). This guide carries the longer-form templates, the deep dive on writing acceptance criteria, and troubleshooting. If the two diverge, follow SKILL.md and fix the guide.

## Purpose

This document guides the creation of `docs/PRODUCT.md` — the product requirements document that defines *what* gets built and *why*. PRODUCT.md is the foundation that all technical documentation derives from, so it's created in its own session before any technical docs.

Give this file to Claude Code along with your completed business brief from step 1. Claude will work through the requirements section by section, producing a comprehensive PRODUCT.md that feeds directly into ARCHITECTURE.md, API.md, and the rest of the technical documentation.

---

## How to Use This

### What You Provide

Your completed `business-brief.md` from step 1. Claude Code reads the brief and translates it into structured product requirements.

### What You Get

A `docs/PRODUCT.md` file that covers: product overview, target user, business model, every feature with user flows and acceptance criteria, data models, edge cases, security requirements, UI/UX descriptions, and explicit scope boundaries.

---

## Claude's Approach

Claude follows this process in order:

1. **Read the business brief and confirm understanding.** Summarize the product, target user, and core value proposition in its own words. Flag any gaps that will block requirements — unclear features, missing user flows, ambiguous scope boundaries. Get these resolved before writing.

2. **Work through PRODUCT.md section by section.** Start with the product overview, then features and user flows, then data models, then acceptance criteria. Present each major section for user review before moving to the next. Don't write the entire document in one pass.

3. **Ask questions when the brief is ambiguous** rather than guessing. For product decisions (feature priority, scope boundaries, user flow details), present options and let the user decide. These are product choices that shape everything downstream.

4. **After the document is complete, review for completeness:**
   - Every feature has acceptance criteria (what "done" looks like)
   - Every data model is defined (entities, relationships, field types)
   - Scope boundaries are explicit (what's in v1 vs. deferred)
   - Edge cases and error states are documented
   - UI/UX descriptions are specific enough for a developer to implement from

---

## What PRODUCT.md Should Contain

**Named `PRODUCT.md`** (not `PRD.md`) for consistency — no acronyms in file names.

**Should contain:**
- Product overview and target user
- Business model / monetization
- Feature descriptions with user flows (step-by-step user journeys)
- Data model definitions (entities, relationships, field types)
- Acceptance criteria per feature
- Edge cases and error states
- Security requirements
- UI/UX descriptions (screen inventory, navigation structure)
- What's in scope vs. explicitly out of scope

**Should NOT contain:**
- Implementation details (that's ARCHITECTURE.md and specs)
- API endpoint definitions (that's API.md)
- Code patterns (that's CONVENTIONS.md)

**Tip:** Version this file (e.g., "Phase 1 v1.1"). When you iterate on requirements, bump the version and note what changed.

**Two cross-cutting requirements every applicable feature must carry** (checked in SKILL.md's final review):
- **Accessibility** — every consumer-facing feature states an explicit a11y criterion (e.g., "fully keyboard-navigable; WCAG 2.1 AA on contrast/labels/focus order"). This pairs with `tstack-architect`'s ADR-3. Don't leave it implicit and don't write a vaguer bar than ADR-3 commits to.
- **Data retention & deletion** — every feature that stores user data states how long it's kept and how it's deleted. This pairs with ADR-4 (privacy). "Indefinite while account active; purged 30 days after account deletion" is concrete; "we store it" is not.

---

## Writing Acceptance Criteria (the load-bearing part)

Acceptance criteria are what make a feature *buildable and verifiable*. `tstack-roadmap` turns them into milestone "Done when" criteria and `tstack-build` proves each one with a real command — so a criterion a command can't check is a defect that surfaces much later. Pick the format that matches the feature's behavior.

### 1. Deterministic features → Given/When/Then

CRUD, business rules, validation, navigation, permissions. The output is a fixed function of the input.

```
Given a logged-in user with no projects,
when they visit /projects,
then they see the empty state with a "Create project" CTA.
```

Write the happy path, the main error path, and any isolation rule (e.g., "user A cannot see user B's records") as separate criteria.

### 2. AI / LLM / ML features → eval-based criteria

Chat, recommendations, classification, generation, search relevance — anything non-deterministic. **Given/When/Then does not work here** because the same input can produce different valid outputs. Use three required parts:

**a. Eval set** — a defined, *versioned, on-disk* set of inputs you measure against. Not "some test prompts."
- Reference it by path: `evals/support-questions-v1.jsonl`.
- **Sizing:** enough to make the quality number stable, not exhaustive. ~30–50 hand-labeled cases is a realistic v1 floor for a focused feature; broad/high-stakes features want 100+. The rule is "would adding 10 more cases swing the score meaningfully?" — if yes, it's too small. Say the size in the criterion.
- Version it (`-v1`) so a later change to the set is visible rather than silently moving the goalposts.

**b. Quality bar** — a measurable target on that set. This is where teams wrongly reach for soft language. Make it a number:
- Graded-correctness: "≥85% top-1 accuracy vs. labeled answers."
- **Subjective-but-measurable** (relevance, helpfulness, tone): a Likert average *is* acceptable when the rubric and rater are defined — "mean relevance ≥4.0/5, rated by the rubric in `evals/grader.md`" (an LLM grader or a human; name which). The test is not "is it objective?" but "is it a reproducible number with a defined rater and rubric?" "Feels relevant" fails because nothing reproduces it; "≥4.0/5 on this rubric" passes.
- Latency belongs here too if it's part of the bar: "p95 ≤ 2.5s over the eval set."

**c. Fallback behavior** — what the feature does on model error, timeout, low confidence, or content-filter trip. This part *is* deterministic, so give it its own Given/When/Then.

```
Eval set:   evals/recipe-suggestions-v1.jsonl (50 prompts, hand-labeled ideal outputs)
Quality bar: ≥80% of suggestions rated "relevant or better" by the grader prompt in evals/grader.md
Fallback:   Given a model error or timeout >5s, when the user requests suggestions,
            then show "We couldn't generate suggestions right now — here are popular recipes"
            with the trending list.
```

### 3. Performance / latency features → measurable thresholds

"Search returns in ≤200ms p95 over 10k indexed documents." State the percentile and the conditions (data volume, concurrency) — a latency number without them isn't reproducible.

### The one rule across all three

If you can't express a criterion in one of these forms, the feature isn't specified yet — clarify it. Soft criteria ("feels fast", "works well", "is intuitive") never count.

---

## Troubleshooting

### Business Brief Doesn't Translate to Requirements

If the business brief is too vague to write specific requirements, go back:

```
The business brief doesn't specify enough about [data model / user flows /
integrations] to write PRODUCT.md. Let's go back to the brief and flesh out
that section before continuing.
```

### Requirements Are Too Thin

If a feature description reads like a single sentence with no user flow or acceptance criteria, push for depth:

```
The "invoice generation" feature needs more detail. Walk me through the user
flow step by step: what triggers it, what the user sees, what data is required,
what the output looks like, and what happens when something goes wrong.
```

### Scope Is Creeping

If PRODUCT.md is growing beyond what's realistic for v1, re-anchor on the business brief's v1 scope:

```
We're adding features that weren't in the v1 scope from the business brief.
Let's review: which of these are truly v1, and which should move to the
"Explicitly Deferred" section?
```

### Missing Acceptance Criteria

If features are described but don't have testable completion criteria:

```
Every feature needs acceptance criteria — specific, testable conditions that
define "done." Go back through each feature and add criteria, picking the format
that fits: Given/When/Then for deterministic features, eval-set + quality-bar +
fallback for AI/ML features, measurable thresholds for performance. See "Writing
Acceptance Criteria" above.
```

Watch specifically for an **AI feature wearing deterministic clothes** — a "smart suggestions" or "auto-categorize" feature with a Given/When/Then criterion. That's a sign the non-determinism wasn't acknowledged; convert it to eval-based criteria.

---

## After PRODUCT.md Is Complete

Commit the file to your project:

```bash
git add docs/PRODUCT.md
git commit -m "docs: create PRODUCT.md from business brief"
```

**Start a fresh Claude Code session** for the next step. This is intentional — Claude Code produces better technical docs when it reads PRODUCT.md as a finalized artifact from disk rather than as something it just wrote in the same session.

Next steps:
- Run the `tstack-architect` skill to create ARCHITECTURE.md, API.md, CONVENTIONS.md, TESTING.md, DECISIONS.md, breakout specs, AGENTS.md, and CLAUDE.md.
- Then run `tstack-roadmap` to create the build-order roadmap.
