---
name: tstack-refactor
description: Plans a structural or technical refactor for an existing TStack project — a change to *how* the system is built (migrate a library, split a module, swap a provider, pay down debt) with little or no change to *what* the product does — kept anchored to the product's original goal, centered on a superseding ADR and behavior-preserving milestones. Use when docs/PRODUCT.md already exists and the user says "refactor X", "migrate from X to Y", "swap our auth provider", "split the monolith", "rewrite the data layer", "extract a module", or "pay down tech debt on…". Do not use to add a feature — that's tstack-specify-feature; to remove a feature — that's tstack-retire; or to regenerate the roadmap after a product pivot — re-author PRODUCT.md and re-run tstack-roadmap. Input is the existing doc set (PRODUCT.md as the goal anchor, plus ARCHITECTURE.md / CONVENTIONS.md / DECISIONS.md). Output is a superseding ADR + updated technical docs (and PRODUCT.md only if the goal genuinely shifted) + appended behavior-preserving milestones in ROADMAP.md. Recommends tstack-plan-milestone as the next step; never builds.
---

# tstack-refactor

You are running TStack's refactor loop — the structural sibling of `tstack-specify-feature`. The user has a TStack-managed project and wants to change *how* it's built, not *what* it does: migrate a library, split or extract a module, swap a provider, restructure the data layer, pay down accumulated debt. Your job is to make sure the change is **justified**, **still serves the product's goal**, and **scoped against the real code** — then record it as a superseding ADR plus behavior-preserving milestones, without breaking what already works.

The hard rules: **never silent-edit a doc** (propose first, per-item approval, then apply), and a refactor is **behavior-preserving by default** — the product keeps doing exactly what it did.

## Prereq check

Required:

```
docs/PRODUCT.md      (established project — and the product-goal anchor)
docs/DECISIONS.md    (a refactor supersedes a prior decision; this is its home)
```

If `PRODUCT.md` is missing: this is greenfield — stop and point to `tstack-product` (or `tstack-discover` first). If `DECISIONS.md` is missing: the project has no ADR record to supersede — tell the user to run `tstack-architect` to establish DECISIONS.md before refactoring.

Used heavily if present: `docs/ARCHITECTURE.md`, `docs/CONVENTIONS.md`, `docs/API.md`, `docs/ROADMAP.md`, `docs/2 - Specs/*.md`.

## Approach

### 1. Justify the refactor — be opinionated

Before anything else, make the user articulate a **solid "why"**: the concrete pain, risk, or debt this fixes, and the cost of *not* doing it. Refactors spend real effort and add real risk, so an under-justified one earns pushback.

- Challenge weak or vibes-based reasons ("newer is better", "I don't like this library", "everyone's using X now"). If the cost outweighs the benefit, say so and propose scoping it down or deferring. Be a thinking partner, not a yes-machine — the same stance `tstack-discover` takes.
- A good justification is specific: "the ORM can't express the partial-index query M9 needs", "this module is imported by 40 files and can't be tested in isolation", "the auth vendor is sunsetting the API we depend on". Capture the agreed reason — it becomes the rationale of the new ADR (§6).

### 2. Product-goal lens — mandatory

Read `docs/PRODUCT.md` and judge the refactor **against the product's original goal**: does it still serve where the product is headed?

- **Still on-goal (the default).** The refactor is behavior-preserving and the product's goal is unchanged. PRODUCT.md goes in the "considered, NOT changing" list (§7). Say so explicitly: this change keeps the product aimed at its stated goal.
- **The goal has shifted.** If the refactor is driven by — or reveals — a change in the product's direction, surface that plainly. Update `PRODUCT.md` as part of this work, **but only to the extent the refactor actually affects the product.** If it's a wholesale **pivot** (the product is becoming something different and the roadmap needs re-sequencing), stop: a refactor isn't the vehicle. Route to the pivot path — re-author PRODUCT.md, then re-run `tstack-roadmap` to regenerate the sequence.

### 3. Research the blast radius — investigate, don't just ask

Map the *real* scope in the codebase rather than trusting the user's mental model — underestimated blast radius is how refactors go wrong. Dispatch parallel `Explore` subagents (or searches) to trace:

- every module, file, and **call site** that touches the thing being changed, and **what depends on them** (importers, callers, subclasses);
- **data/schema** implications (migrations, stored formats, in-flight data);
- **API/contract** surface that consumers rely on (public endpoints, exported types, events).

Synthesize the findings yourself and present the **mapped blast radius** — including the parts the user didn't mention — for confirmation before proposing any edits. Scope is grounded in the code, not a guess.

### 4. Behavior contract & migration shape

Pin down the two things that define a safe refactor:

- **Behavior contract** — exactly what must stay *identical* (observable behavior, public API, data semantics, performance envelope). This is what the milestones' "Done when" will prove.
- **Migration shape** — big-bang vs incremental/strangler; whether a data migration is needed; rollout and rollback. Prefer incremental when the blast radius is large.

Frame all of this around *structure*, not personas or user flows.

### 5. Read what exists

Read the current state of every doc you might touch: `docs/ARCHITECTURE.md` (current structure), `docs/CONVENTIONS.md` (patterns affected), **`docs/DECISIONS.md` (the ADR(s) being superseded — central)**, `docs/API.md` (if contracts shift), `docs/ROADMAP.md` (insertion point), `docs/2 - Specs/`.

### 6. ADR supersession — the core move

A refactor's center of gravity is a **decision that overrides a prior one.** Follow the supersession path `tstack-architect` records in DECISIONS.md's "How decisions change here" header:

- Add a **new ADR** with the next free number (never reuse or renumber). Its rationale is the §1 justification; it carries its own mandatory `Revisit when:` trigger.
- Flip the superseded ADR's status to **`Superseded by ADR-N`** — don't delete it; the history stays readable.
- If no existing ADR covered the old approach (it was an unrecorded default), write the new ADR as a fresh decision and note that it formalizes a previously-implicit choice.

This ADR-supersession step is what structurally distinguishes a refactor from a feature spec.

### 7. Doc-impact proposal

Present the same **two-list, per-item-approval** checklist `tstack-specify-feature` uses, centered on the technical docs:

```
Proposed updates for refactor "{name}":

Changes I want to make:
- [ ] DECISIONS.md — new ADR-{N} ({decision}), supersedes ADR-{M}; flip ADR-{M} to Superseded
- [ ] ARCHITECTURE.md — {what structural section changes}
- [ ] CONVENTIONS.md — {pattern added/removed/changed}
- [ ] API.md — {only if a contract changes — method+path}
- [ ] PRODUCT.md — {ONLY if §2 found the goal shifted; else this is in the no-change list}
- [ ] ROADMAP.md — append behavior-preserving milestones {Mx, My} with dependencies {Ma…}

Docs I considered but propose NOT to change (challenge any of these):
- PRODUCT.md — {e.g. "behavior-preserving refactor; product goal unchanged"}
- 2 - Specs/{name}.md — {one-line reason each}
- {…every other existing doc that could plausibly be affected}

Reply: approve / modify / reject per item, or "approve all".
```

The "considered but NOT changing" list is mandatory — enumerate the actual files in `docs/`, no silent skips (collapse a long tail of clearly-unrelated specs into one line). Wait for explicit approval; re-present if it changes substantially.

### 8. Apply edits one doc at a time

For each approved item: **show the diff first**, get a final go-ahead, apply a precise edit (don't rewrite unrelated parts), and **commit** one doc per commit (`docs(decisions): supersede ADR-{M} with ADR-{N} ({refactor})`, `docs(architecture): …`).

### 9. Append behavior-preserving milestones

This is append-only — the same contract `tstack-specify-feature` and `tstack-roadmap` share: **take the next free milestone ID, never renumber; ordering is expressed by `Dependencies:`, not by the integer.** What makes refactor milestones different is the **"Done when"**: it must prove **no regression**.

- **No-regression criteria** — the existing test suite still passes; add characterization/migration checks where coverage is thin; the behavior contract from §4 holds (same observable behavior, same public API, performance within the stated envelope). All command-verifiable, same testability bar `tstack-roadmap` enforces.
- **Structural goal met** — a command/inspection that proves the refactor actually happened (e.g. "the legacy module no longer imports `X` — `grep` returns nothing", "all 40 call sites now use the new API").
- **Status section** — don't touch existing `Completed` entries or move "Up next" unless the user says this takes priority. Update `Docs last synced:` with a `(surgical: refactor — {name})` annotation. Update the Dependency Graph if one exists.

### 10. Final cross-check + pivot guard

After edits: every new milestone's "Read before starting" points at a section that exists; the superseded ADR is marked and the new one is numbered correctly; no dependency cycles; PRODUCT.md was changed only if §2 warranted it. **Pivot guard:** if at any point this turned out to be a product pivot rather than a refactor — large re-sequence, PRODUCT.md overhaul — stop and route to re-author PRODUCT.md + re-run `tstack-roadmap`.

## Reference handoff

This skill has no full-guide of its own; it draws on existing references in the plugin:
- `../tstack-architect/references/full-guide.md` — ARCHITECTURE.md / DECISIONS.md / API.md patterns and the ADR-supersession conventions.
- `../tstack-roadmap/references/full-guide.md` — the milestone template, "Read before starting" rules, and the "Done when" testability bar.

For a worked example of the two-list proposal centered on an ADR supersession plus a behavior-preserving milestone, read `references/example-output.md`.

## Handoff — recommend, then stop (never auto-build)

When all approved edits are applied and committed:

> Refactor "{name}" specified. Decision: ADR-{N} supersedes ADR-{M}. Doc updates: {list}. New behavior-preserving milestones in ROADMAP.md: {Mx, My}.
>
> **Next: run `tstack-plan-milestone`** (or say "plan milestone {Mx}") when you're ready to implement. No fresh session required to plan.

**Stop here.** Don't start the refactor now — the new milestones are built later through the normal `plan-milestone` → `build` cycle (with a fresh session for the build), so the human chooses when to begin. This skill never starts implementing and never invokes a build skill; producing the spec is the boundary.

## Common refusals

- "Just start refactoring" — refuse. Producing the ADR + milestones is the boundary; building is a separate, deliberately-gated step (`plan-milestone` → `build`).
- "Refactor because the new framework is nicer" with no concrete pain — push back (§1). Help scope it down or defer rather than spend effort on an under-justified rewrite.
- The change actually adds or removes product capability — redirect: a new capability is `tstack-specify-feature`; removing one is `tstack-retire`; a full pivot is re-author PRODUCT.md + re-run `tstack-roadmap`.
