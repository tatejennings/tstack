# Product Requirements Guide — Creating PRODUCT.md

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
define "done." Go back through each feature and add criteria. Format:
"Given [context], when [action], then [expected result]."
```

---

## After PRODUCT.md Is Complete

Commit the file to your project:

```bash
git add docs/PRODUCT.md
git commit -m "docs: create PRODUCT.md from business brief"
```

**Start a fresh Claude Code session** for the next step. This is intentional — Claude Code produces better technical docs when it reads PRODUCT.md as a finalized artifact from disk rather than as something it just wrote in the same session.

Next steps:
- See `03-create-technical-docs.md` for creating ARCHITECTURE.md, API.md, CONVENTIONS.md, DECISIONS.md, breakout specs, AGENTS.md, and CLAUDE.md.
- See `04-generate-roadmap.md` for creating the build-order roadmap.
