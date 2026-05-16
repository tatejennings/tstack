# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

This repo *is* the **TStack** Claude Code plugin. It's not an application, it's not a documentation kit — it's a plugin you install into other projects so the seven TStack skills become available there. The deliverable is the contents of `.claude-plugin/` and `skills/`.

Most edits here are editorial: tightening a `SKILL.md`, updating a `references/full-guide.md`, or refining the README. There is no build, test, or lint.

## Architecture

- `.claude-plugin/plugin.json` — plugin manifest (name, version, author). Skills are auto-discovered from `skills/`, not enumerated here.
- `skills/tstack-<stage>/SKILL.md` — the concise procedural top-level for each skill. Frontmatter `description` is load-bearing — it drives auto-triggering. ~120–200 lines max.
- `skills/tstack-<stage>/references/full-guide.md` — the verbatim long-form guide. Source of edge-case detail; SKILL.md tells Claude when to consult it.

The chain: `discover → product → architect → roadmap → plan → build`. Plus `specify` as the iteration loop, which hands off to `plan`.

**Foundational decisions live in `tstack-architect`.** Before generating any technical doc, the skill asks four mandatory questions (security, observability, accessibility, privacy) — each becomes an ADR in DECISIONS.md. Optionally a fifth question for AI/LLM products. DECISIONS.md and TESTING.md are now mandatory outputs at every right-sizing level. When editing `tstack-architect/SKILL.md`, preserve this order: prereq check → foundational ADRs → AI question → right-sizing → approach.

**`tstack-roadmap` mandates an `M0 — Infrastructure baseline` milestone** (CI, branch protection, secrets, deployment skeleton, observability bootstrap, lint, test runner). The skill refuses to save a roadmap without it. Don't relax this when editing.

## Editorial Conventions (Match These When Editing)

- **Filenames use full words, no acronyms.** `PRODUCT.md`, not `PRD.md`.
- **Every SKILL.md follows the same shape:** frontmatter → Prereq check → Approach → Reference handoff → Handoff. Don't break the symmetry — it makes the chain readable.
- **Frontmatter `description`** must lead with what the skill does, then "Use when…" triggers, then explicit Input/Output, then handoff. The Input clause is what makes Claude check prerequisites before triggering.
- **`tstack-product` and `tstack-specify` are mutually exclusive triggers.** Product creates initial PRODUCT.md; specify amends an existing one. Both descriptions carry an explicit negative phrase to disambiguate.
- **Verification in `tstack-build` requires quoted command output.** Every "Done when" criterion is checked by running a real command and pasting its output into the conversation. "Verified ✓" without output is not acceptable — match this discipline when editing the verification section.
- **AI-feature acceptance criteria use eval-based format**, not Given/When/Then. Required parts: eval set (versioned, on disk), quality bar (measurable threshold), fallback (deterministic). `tstack-product` enforces this — match the format when adding examples.

## What NOT to Do

- Don't add a build system, tests, or CI — there's no executable code.
- Don't add `AGENTS.md` to this repo's root. The framework *produces* AGENTS.md in consumer projects, but the plugin itself isn't a consumer of itself.
- Don't enumerate skills in `plugin.json` unless plugin loading turns out to require it (currently it doesn't).
- Don't rewrite `references/full-guide.md` content unless asked — those are validated prose, preserved verbatim from the pre-plugin guides. Edit the SKILL.md instead.

## Common Tasks

- **Tweak a skill's behavior:** edit the relevant `skills/tstack-<name>/SKILL.md`. Keep it concise; move detail into the reference.
- **Update the auto-triggering of a skill:** edit only the `description:` field in the SKILL.md frontmatter. Test by saying the trigger phrases in a consumer project.
- **Add a new skill:** create `skills/tstack-<name>/SKILL.md` following the established shape. Update README's lifecycle diagram and skill table. Decide where it fits in the chain (or whether it's a parallel iteration skill like `tstack-specify`).
- **Test the plugin:** see README's Quick Start. From a scratch directory, `/plugin install <this-repo-path>` and walk the chain.
