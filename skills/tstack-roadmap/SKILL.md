---
name: tstack-roadmap
description: Reads the full docs/ tree and produces docs/ROADMAP.md — a dependency-sequenced list of milestones with "Read before starting" doc pointers and "Done when" criteria. Use when ARCHITECTURE.md and PRODUCT.md exist and the user asks "what do we build first", "sequence the work", or "make a roadmap". Input is docs/PRODUCT.md + docs/ARCHITECTURE.md (API.md, DECISIONS.md, and specs optional); output is docs/ROADMAP.md. Hands off to tstack-build.
---

# tstack-roadmap

You are running TStack's roadmap stage. You read every doc the previous skills produced and synthesize them into a strict, dependency-sequenced list of milestones that an implementer (or another instance of you, via `tstack-build`) can pick up one at a time.

## Prereq check

Required inputs:

```
docs/PRODUCT.md
docs/ARCHITECTURE.md
```

Optional but used if present:

```
docs/API.md
docs/CONVENTIONS.md
docs/DECISIONS.md
docs/2 - Specs/*.md
docs/1 - Discovery/business-brief.md
```

If a required input is missing: stop and tell the user to run `tstack-architect` first.

## Repo-self guard

If `.claude-plugin/plugin.json` exists in cwd, refuse.

## Approach

1. **Read every doc in `docs/`** before doing anything else. Extract:
   - From PRODUCT.md: every feature, user flow, data model, acceptance criterion
   - From ARCHITECTURE.md: tech stack, repo structure, module boundaries, data flow (these determine which components depend on which)
   - From API.md: every endpoint (each is a candidate milestone reference)
   - From CONVENTIONS.md: which milestones introduce a new pattern domain
   - From DECISIONS.md: ADRs to reference when a milestone touches an architecturally significant choice
   - From `2 - Specs/`: detailed blueprints — milestones point to specific specs

   Don't reference `1 - Discovery/` directly from milestones; use it only for context on prioritization.

2. **Identify buildable units.** Extract every distinct piece of work. A buildable unit:
   - Can be implemented and tested independently
   - Has clear inputs (dependencies) and outputs (what it enables)
   - Takes roughly 1–2 focused days of work

3. **Build the dependency graph.** For each unit, determine what must exist first:
   - **Technical:** schema → CRUD; auth → authenticated endpoints; encryption module → encrypted writes; API → UI that calls it; server → mobile that consumes it
   - **Data:** matching/reconciliation needs at least one source; reports need data they report on; scoring needs underlying records
   - **Shared components:** reusable UI shells before pages; shared libraries (validation schemas, types) early

   Flag independence — features on different tables with no FK relationships *don't* depend on each other and can run in parallel.

4. **Sequence into milestones.** Use the milestone template from `references/full-guide.md` § Milestone Entries. Each entry contains:
   - **What gets built** (1–2 sentences, concrete deliverable)
   - **Dependencies** (M-IDs that must be complete first, or "None — starting point")
   - **Read before starting** (specific doc sections — not whole files)
   - **Done when** (binary, testable criteria — happy path + error path + isolation where relevant)

   Do NOT put status on individual milestones. Status lives only in the Status section at the bottom.

5. **Naming conventions:**
   - Server/web milestones: `M0`, `M1`, `M2`, …
   - Mobile milestones (separate workstream): `i0`, `i1`, … (or other letter if not iOS)
   - Mobile milestones depend on the server milestones whose APIs they consume — make that dependency explicit.

6. **Include sections:**
   - Header (overview, how to use, prefix conventions)
   - Dependency Graph Diagram (ASCII, parallel tracks if multi-workstream)
   - Milestone Entries (in build order)
   - Parallelization Notes (which milestones can run simultaneously; critical path)
   - **Status** section at the bottom: `Completed:` (none yet) and `Up next: M0 — …`

7. **Cross-check before saving:**
   - Every "Read before starting" reference points to a doc/section that exists
   - Every "Done when" criterion is binary
   - No milestone exceeds ~2 days of focused work (split if so)
   - No milestone is so granular it's not independently shippable (combine if so)

8. **Save to `docs/ROADMAP.md`.** Commit: `docs: generate ROADMAP.md from project docs`.

9. **Add a Current Focus pointer** to `AGENTS.md` (or `CLAUDE.md` if AGENTS.md isn't present):

   ```markdown
   ## Current Focus
   Check `docs/ROADMAP.md` — see the Status section at the bottom for the current milestone.
   ```

## Reference handoff

For the full milestone template, "Read before starting" rules, "Done when" rules, and troubleshooting (too-large/too-granular milestones, missing dependencies), read `references/full-guide.md`.

## Handoff

> Roadmap complete — `docs/ROADMAP.md` written with {N} milestones across {workstreams}.
> Up next: M0 — {name}.
>
> **Next: run `tstack-build`** (or say "start milestone M0"). No fresh session needed — `tstack-build` is the implementation loop and doesn't need to re-read the whole doc set.

Optionally update `.tstack/state.json` `stage` to `"roadmap"`.
