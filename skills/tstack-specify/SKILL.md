---
name: tstack-specify
description: Specifies a new feature for an existing TStack project. Interviews the user about the change, proposes which docs need updates (PRODUCT.md, ARCHITECTURE.md, API.md, breakout specs) with per-item approval before editing, then appends new milestones to docs/ROADMAP.md. Use when docs/PRODUCT.md already exists and the user says "let's add X feature", "I want to spec out a new capability", "this product needs to change", "add a feature", or "extend the product". Do not use to create the initial PRODUCT.md — that's tstack-product. Input is the existing doc set; output is updated docs + new milestone entries in ROADMAP.md. Hands off to tstack-build.
---

# tstack-specify

You are running TStack's iteration loop. The user has a TStack-managed project that's already past initial setup, and they want to add or change a feature. Your job is to weave that feature through every doc it affects and append the right milestones to ROADMAP.md — without touching anything that doesn't need to change.

The hard rule: **never silent-edit a doc.** Always propose first, get per-item approval, then apply.

## Prereq check

Required:

```
docs/PRODUCT.md
```

If it's missing: this is greenfield. Stop and tell the user to run `tstack-product` (or `tstack-discover` first if they don't have a brief either).

Optional but used heavily if present: `docs/ARCHITECTURE.md`, `docs/API.md`, `docs/CONVENTIONS.md`, `docs/DECISIONS.md`, `docs/ROADMAP.md`, `docs/2 - Specs/*.md`.

## Repo-self guard

If `.claude-plugin/plugin.json` exists in cwd, refuse. This is the TStack plugin repo, not a consumer project.

## Approach

### 1. Feature interview

Ask focused questions about the new feature. Cover at minimum:
- **What** is being added or changed? (one-sentence summary)
- **Who** is it for? (existing user, new persona, both?)
- **User flow** — walk through it step by step: what triggers it, what the user sees, what data is required, what the output is, what happens on error
- **Data** — does it need new entities, new fields on existing entities, or no schema change?
- **Surface area** — UI screens affected, API endpoints needed, background jobs involved
- **Acceptance criteria** — what "done" looks like, in "Given … when … then …" form

Adapt: skip what the user already gave you in their initial request. Go deeper where answers are vague.

### 2. Read what exists

Before proposing edits, read the current state of every doc you might touch:

- `docs/PRODUCT.md` — find the feature category this fits under; check for conflicting acceptance criteria
- `docs/API.md` (if present) — see the existing endpoint patterns and error format
- `docs/ARCHITECTURE.md` — see if the feature crosses a module boundary or requires a new component
- `docs/DECISIONS.md` — see if any ADR constrains this feature's implementation
- `docs/ROADMAP.md` — find the right insertion point (dependency-aware, not just bottom-append)
- `docs/2 - Specs/` — see existing spec topics; decide if a new spec is needed

### 3. Doc-impact proposal

Present a checklist. Use this exact format so the user can give per-item approvals fast:

```
Proposed updates for "{feature name}":

- [ ] PRODUCT.md — {what changes: new feature section under §X, updated data model for User (adds `timezone`), updated acceptance criteria for §Y}
- [ ] ARCHITECTURE.md — {no change | what changes}
- [ ] API.md — {no change | which endpoints get added/modified, with method+path}
- [ ] CONVENTIONS.md — {no change | which new pattern is being introduced}
- [ ] DECISIONS.md — {no change | which new ADR is being added, ADR-N: "..."}
- [ ] 2 - Specs/ — {no change | new file 2 - Specs/{name}.md, or edits to existing spec X}
- [ ] ROADMAP.md — append milestones {Mx (server-side), My (UI), iz (mobile)} with dependencies on {Ma, Mb}

Reply: approve / modify / reject per item. Type "approve all" if everything looks right.
```

Wait for explicit approval per line. Modify the proposal based on user pushback. Re-present if it changes substantially.

### 4. Apply edits one doc at a time

For each approved item:

1. **Show the diff first.** Read the current section, draft the change, and present the diff before saving. For new sections, show the full new content. Get a final go-ahead on each diff.
2. **Apply the edit.** Use precise edits — don't rewrite unrelated parts of the file.
3. **Commit.** One commit per doc with a descriptive message: `docs(product): add {feature} feature spec`, `docs(api): add {endpoint} endpoints for {feature}`, etc.

If you're creating a new breakout spec, follow the template style of existing specs in `docs/2 - Specs/`. Read one of them first to match formatting.

### 5. Append milestones to ROADMAP.md

This is the most surgical edit. Rules:

- **Dependency-aware insertion** — a new milestone may depend on existing infrastructure milestones (e.g., the encryption module from M3). Make those dependencies explicit. Don't just append to the bottom; place it where dependencies are satisfied.
- **Reuse the milestone template** from `references/full-guide.md` of the `tstack-roadmap` skill (i.e., `../tstack-roadmap/references/full-guide.md` from the plugin's perspective; in a consumer project, read the patterns in the existing ROADMAP.md).
- **Numbering** — continue the existing sequence. If existing milestones go up to M13, start at M14 (or `i7` for the iOS workstream).
- **Required fields per milestone:** What gets built, Dependencies, Read before starting (point at the *just-updated* doc sections), Done when (binary criteria).
- **Status section** — do not touch existing Completed entries. Do not move the "Up next" pointer unless the user explicitly says this new feature is taking priority over the current "Up next".
- Update the Dependency Graph Diagram at the top of ROADMAP.md if the diagram exists.

### 6. Final cross-check

After all edits are applied:
- Every new milestone's "Read before starting" reference points to a section that exists
- Every new endpoint in API.md is referenced from at least one milestone
- The new feature has acceptance criteria in PRODUCT.md (testable, "Given … when … then …")
- No two milestones list each other as a dependency (no cycles)

Flag anything off for the user.

## Reference handoff

For deep templates, this skill draws on existing references in the plugin:
- `../tstack-product/references/full-guide.md` — for the PRODUCT.md section style, acceptance-criteria format, data model conventions
- `../tstack-roadmap/references/full-guide.md` — for the milestone template, "Read before starting" rules, "Done when" rules
- `../tstack-architect/references/full-guide.md` — for ARCHITECTURE.md, API.md, and spec patterns

In a consumer project, those references are inside the installed plugin — you can read them, but more practically, model the new content on the patterns already present in the consumer's docs.

## Handoff

When all approved edits are applied and committed:

> Feature "{name}" specified. Doc updates: {list}. New milestones in ROADMAP.md: {Mx, My, …}.
>
> **Next: run `tstack-build`** (or say "start milestone {Mx}") to implement.
>
> No fresh session required — `tstack-build` reads the roadmap on entry.
