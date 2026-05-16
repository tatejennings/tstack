---
name: tstack-plan
description: Plans the next milestone for a TStack-managed project. Reads docs/ROADMAP.md to identify which milestone is up, creates the feature branch, reads every doc the milestone's "Read before starting" section names, enters plan mode, and produces an approved implementation plan. Use when docs/ROADMAP.md exists and the user says "plan milestone Mx", "let's plan the next milestone", "what should we build next", "enter plan mode for M4", or otherwise wants a doc-grounded implementation plan before writing code. Input is docs/ROADMAP.md + the docs that milestone references. Output is an approved implementation plan and a feature branch ready to build on. Hands off to tstack-build.
---

# tstack-plan

You are running TStack's per-milestone planning stage. Your job is to take a roadmap entry and turn it into an implementation plan grounded in the specific docs that milestone points at — not in your own assumptions about what the user probably wants. The output is an *approved* plan and a clean feature branch. You don't write feature code in this skill. That's `tstack-build`.

## Prereq check

Required:

```
docs/ROADMAP.md
```

If missing: stop. Tell the user this skill is for TStack-managed projects only. If they want to start one, run `tstack-discover`. If they have docs but no roadmap, run `tstack-roadmap`.

Check the Status section of `docs/ROADMAP.md`. If "Up next" is empty: the project is done — tell the user. If they want to add a feature instead, point them to `tstack-specify`.

## Repo-self guard

If `.claude-plugin/plugin.json` exists in cwd, refuse. This is a plugin repo, not a TStack consumer project.

## Approach

### 1. Confirm the milestone

Read the Status section at the bottom of `docs/ROADMAP.md`. The "Up next" entry is the default target. Verify with the user:

> Planning M{N} — {name}. Dependencies {Mx, My} are in Completed. Proceed?

If the user wants a different milestone, accept it — but verify its listed dependencies are all in Completed before continuing. Refuse to plan a milestone whose dependencies aren't done.

### 2. Create the feature branch

Branch off `main` (or the project's default) so plan mode operates on a clean working tree:

```bash
git checkout main && git pull
git checkout -b milestone/{id-lowercased}-{short-desc}
```

Naming pattern: `milestone/m4-entity-crud`, `milestone/i2-entity-list` (iOS workstream), `milestone/m21-sync-endpoint` (cross-cutting).

### 3. Read the milestone's docs

The milestone's entry in `docs/ROADMAP.md` lists a "Read before starting" section. Read **exactly** what it lists — specific sections, not whole files unless the entry says so. The roadmap was generated to minimize required reading; trust it.

Typical reads:
- `docs/ARCHITECTURE.md` — relevant section
- `docs/API.md` — relevant endpoints
- `docs/2 - Specs/{spec}.md` — the implementation blueprint
- `docs/PRODUCT.md` — section with acceptance criteria
- `docs/DECISIONS.md` — relevant ADRs (only if listed)
- `docs/CONVENTIONS.md` — relevant section (only if milestone introduces a new pattern domain)

Also read `docs/AGENTS.md` if present — it carries project-wide conventions every plan should respect.

### 4. Enter plan mode and produce a plan

Switch into plan mode. The plan must contain:

- **Files to create or modify**, in dependency order (lowest level first — schema, then helpers, then API routes, then UI)
- **Existing patterns to reuse** — name the file path. Before inventing anything, search for an existing helper, schema, validation, or component you can extend
- **New patterns to introduce**, with rationale tying back to a spec or ADR
- **Verification approach** for each piece — what test to write, what curl/manual check to run, how you'll confirm it works
- **Tradeoffs the user should weigh** before approving (alternative approaches, scope boundaries, performance vs. simplicity)
- **Out-of-scope explicitly** — anything the milestone is *not* covering, so it doesn't creep in during build

Cross-check the plan against the milestone's "Done when" criteria in ROADMAP.md. Every criterion must be addressed by a specific step in the plan. If a criterion isn't covered, your plan is incomplete — fix it before presenting.

### 5. Review with the user, adjust, approve

Present the plan. Expect pushback. Common categories to incorporate:

- "Use the existing pattern in `<file>` instead of inventing a new one."
- "Move shared types/schemas to a common location — they'll be needed again."
- "Smaller commits — break this into two phases."
- "You missed the case where X."

Iterate until the user approves. Do not exit plan mode prematurely.

### 6. Hand off

When the plan is approved:

> Plan approved for M{N} — {name}. Feature branch `milestone/{id}-{desc}` is ready. {N} files to create/modify, {N} verification checks.
>
> **Next: run `tstack-build`** (or just say "build it" / "ship it"). Same session is fine — `tstack-build` picks up from the approved plan and feature branch.

Do not write any code in this skill. Hand the approved plan + branch to `tstack-build`.

## Reference handoff

The full implementation guide lives at `../tstack-build/references/full-guide.md`. It covers planning patterns, "Read before starting" semantics, "Done when" verification approaches, and prompting patterns. Read the planning-relevant sections before producing the plan — especially the prompting patterns for "Starting a Session" and "Plan Phase."

## Common refusals

- The user says "just start building" — politely refuse. Plan-first is the discipline that makes the milestone loop work. The plan takes 5 minutes and saves an hour of misdirection.
- The user wants to plan a milestone whose dependencies aren't in Completed — refuse and list the missing dependencies.
- During planning, the user reveals the feature is misspecified or the roadmap entry is wrong — stop and tell them to run `tstack-specify` to update the doc set first. Don't paper over spec gaps in the plan.
