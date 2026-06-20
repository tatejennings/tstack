# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

This repo *is* the **TStack** Claude Code plugin. It's not an application, it's not a documentation kit — it's a plugin you install into other projects so the TStack skills become available there: seven chained skills, one adoption on-ramp (`tstack-ingest`), plus three optional off-chain companions (`tstack-design`, `tstack-status`, `tstack-wrap`). The deliverable is the contents of `.claude-plugin/` and `skills/`.

Most edits here are editorial: tightening a `SKILL.md`, updating a `references/full-guide.md`, or refining the README. There is no build, test, or lint.

## Architecture

- `.claude-plugin/plugin.json` — plugin manifest (name, version, author). Skills are auto-discovered from `skills/`, not enumerated here.
- `skills/tstack-<stage>/SKILL.md` — the concise procedural top-level for each skill. Frontmatter `description` is load-bearing — it drives auto-triggering. ~120–200 lines max.
- `skills/tstack-<stage>/references/full-guide.md` — the long-form guide. Source of edge-case detail; SKILL.md tells Claude when to consult it. Each carries a header naming SKILL.md as the source of truth. The `discover`/`roadmap`/`build` guides are still the preserved pre-plugin prose; the `architect` and `product` guides have been actively maintained (de-staled/thickened) and are no longer verbatim.

The chain: `discover → product → architect → roadmap → plan → build`. Plus `specify` as the iteration loop, which hands off to `plan`.

**`tstack-ingest` is a chain on-ramp, not an off-chain skill.** It's an alternate entry point for projects that already have docs: it adopts the user's existing material and **hands off into the chain** (to `tstack-product`), the way `specify` hands off to `plan`. Because it has a required handoff, it is *not* off-chain. Don't reclassify it as off-chain, and don't fold its adoption logic into another skill — `tstack-discover` only *detects existing docs and forwards* to ingest; it never distills them itself.

Three skills sit **off-chain** (optional, invokable at any point, no required handoff): `tstack-design` (UX spec + ready-to-paste Claude Design prompts; reads PRODUCT.md if present), `tstack-status` (strictly read-only project inspector — reports status and doc drift, writes nothing), and `tstack-wrap` (end-of-session doc sweep — writes undocumented decisions/gotchas to the right doc, but never commits and never adds/renumbers milestones). When editing an off-chain skill, keep it off-chain — don't wire it into the linear flow; keep `tstack-status` from mutating anything, and keep `tstack-wrap` from committing or touching roadmap milestones.

**Cross-skill contracts (don't regress these when editing):**
- **`AGENTS.md` is owned by `tstack-architect`.** Only it writes the full file. `tstack-roadmap` and `tstack-build` update *only* the `## Current Focus` block; they never restructure the rest.
- **The approved plan is written in-repo at `docs/plans/{milestone-id}.md`.** `tstack-plan` writes it; `tstack-build` reads it. It lives in the project (not a local user folder) on purpose: once committed (a manual step, left to the user), milestones can be planned ahead and built later by a cloud agent or a different machine. The schema lives in `tstack-plan/SKILL.md`.
- **`ROADMAP.md` has one full-regenerator (`tstack-roadmap`) and one append-only editor (`tstack-specify`).** Roadmap carries a `Docs last synced:` marker; specify advances it with a `(surgical: …)` annotation and never renumbers existing milestones. `tstack-status` reads that marker to flag drift.
- **`tstack-roadmap` validates "Done when" testability at write time** — every criterion must map to a runnable command before save, not be discovered as soft at build time.
- **`tstack-ingest` writes only to `docs/_adopted/` — never a canonical doc.** Adoption quarantines distilled material as an *unratified draft* (`docs/_adopted/PRODUCT.draft.md`, `docs/_adopted/*-notes.md`); nothing in `docs/_adopted/` satisfies any chain skill's prereq, so unverified content can't flow into the loop. The owning skill authors the canonical doc *from* the draft through its own gate — `tstack-product` reads `PRODUCT.draft.md`, authors `docs/PRODUCT.md`, and deletes the draft on save. Ingest never writes `PRODUCT.md`/`ARCHITECTURE.md`/`DECISIONS.md`/`ROADMAP.md`/`AGENTS.md` and never fabricates — it carries the source across and names what's missing in a visible `## Open gaps` block. This generalizes the old `tstack-discover` "distill existing material" branch (now removed; discover forwards to ingest). Don't let ingest write canonical files, and don't add a competing distill path elsewhere.

**Foundational decisions live in `tstack-architect`.** Before generating any technical doc, the skill asks four mandatory questions (security, observability, accessibility, privacy) — each becomes an ADR in DECISIONS.md. Optionally a fifth question for AI/LLM products. DECISIONS.md and TESTING.md are now mandatory outputs at every right-sizing level. When editing `tstack-architect/SKILL.md`, preserve this order: prereq check → foundational ADRs → AI question → right-sizing → approach.

**`tstack-roadmap` mandates an `M0 — Infrastructure baseline` milestone** (CI, branch protection, secrets, deployment skeleton, observability bootstrap, lint, test runner). The skill refuses to save a roadmap without it. Don't relax this when editing.

## Design Invariants — the Agentic Loop

TStack is a harness for product-build loops. Three properties are what make the loop trustworthy over a long project; treat them as invariants and don't regress them when editing. (See the README's "The agentic loop" section for the user-facing version.)

- **Verification is the engine — keep it external and quoted.** `tstack-build` checks every "Done when" criterion by running a real command and pasting its output; `tstack-roadmap` refuses criteria that aren't command-verifiable at write time. Don't soften either — a self-asserted "✓" is not a signal.
- **State lives on disk, not in the session.** The docs + `docs/plans/{id}.md` + ROADMAP's Status section are the loop's durable memory; that's why plans are in-repo and why `.tstack/state.json` was removed (see BACKLOG). Don't move cross-session coordination back into session memory.
- **Fresh session per milestone is the iteration boundary.** `tstack-plan` and `tstack-build` end and recommend a fresh session rather than auto-chaining (the v0.3.1 fix). Don't reintroduce same-window plan→build→next-milestone glide. Any future autonomous runner (BACKLOG #5) is opt-in re-automation *on top of* this default, not a reversal of it.

## Editorial Conventions (Match These When Editing)

- **Filenames use full words, no acronyms.** `PRODUCT.md`, not `PRD.md`.
- **Every SKILL.md follows the same shape:** frontmatter → Prereq check → Approach → Reference handoff → Handoff. Don't break the symmetry — it makes the chain readable.
- **Frontmatter `description`** must lead with what the skill does, then "Use when…" triggers, then explicit Input/Output, then handoff. The Input clause is what makes Claude check prerequisites before triggering.
- **`tstack-product` and `tstack-specify` are mutually exclusive triggers.** Product creates initial PRODUCT.md; specify amends an existing one. Both descriptions carry an explicit negative phrase to disambiguate.
- **Verification in `tstack-build` requires quoted command output.** Every "Done when" criterion is checked by running a real command and pasting its output into the conversation. "Verified ✓" without output is not acceptable — match this discipline when editing the verification section.
- **AI-feature acceptance criteria use eval-based format**, not Given/When/Then. Required parts: eval set (versioned, on disk), quality bar (measurable threshold), fallback (deterministic). `tstack-product` enforces this — match the format when adding examples.
- **Always update `CHANGELOG.md`.** Any change to a skill's behaviour, a new/removed skill, or a manifest/README change adds an entry under the `## [Unreleased]` heading (Added / Changed / Fixed / Removed). This is part of every edit, not an afterthought. When tagging a release, move Unreleased entries under the new version heading and bump `version` in `.claude-plugin/plugin.json`.

## What NOT to Do

- Don't add a build system, tests, or CI — there's no executable code.
- Don't add `AGENTS.md` to this repo's root. The framework *produces* AGENTS.md in consumer projects, but the plugin itself isn't a consumer of itself.
- Don't enumerate skills in `plugin.json` unless plugin loading turns out to require it (currently it doesn't).
- Don't rewrite a `references/full-guide.md` unless asked — prefer editing the SKILL.md. The `discover`/`roadmap`/`build` guides remain preserved pre-plugin prose; keep them that way absent a request. (`architect` and `product` guides were intentionally reconciled against their SKILL.md — when you touch those, keep them consistent with the SKILL.md, which is the source of truth.)

## Common Tasks

- **Tweak a skill's behavior:** edit the relevant `skills/tstack-<name>/SKILL.md`. Keep it concise; move detail into the reference. Add a `CHANGELOG.md` Unreleased entry.
- **Update the auto-triggering of a skill:** edit only the `description:` field in the SKILL.md frontmatter. Test by saying the trigger phrases in a consumer project.
- **Add a new skill:** create `skills/tstack-<name>/SKILL.md` following the established shape. Update README's lifecycle diagram and skill table, and the skill-count wording in `plugin.json` / `marketplace.json`. Decide where it fits in the chain (or whether it's a parallel iteration skill like `tstack-specify`, or an optional off-chain companion like `tstack-design` / `tstack-status`). Add a `CHANGELOG.md` Unreleased entry.
- **Test the plugin:** see README's Quick Start. From a scratch directory, `/plugin install <this-repo-path>` and walk the chain.
- **Track future work:** add to `BACKLOG.md` at the repo root. Solo-dev shorthand list of upcoming features, content gaps, and process items.
