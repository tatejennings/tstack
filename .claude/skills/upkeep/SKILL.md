---
name: upkeep
description: >-
  MANUAL INVOCATION ONLY (via /upkeep). Repo-local maintenance skill that deep-reviews THE
  TSTACK PLUGIN REPO ITSELF for internal consistency, sync-point drift, design-invariant
  regressions, AND currency against the live Claude ecosystem (current Claude model
  IDs/tiers, current Claude Code features, current default tech-stack major versions). This
  is NOT a consumer-project audit (that is the planned tstack-audit) and NOT tstack-status /
  tstack-wrap — it runs on the plugin's own source. Do NOT auto-trigger on phrases like
  "review my project" or "check the docs". Input is this repo's skills/, manifests, README,
  CLAUDE.md, CHANGELOG.md. Output is a prioritized review report at reviews/upkeep-YYYY-MM-DD.md
  plus a chat summary; it auto-fixes ONLY safe deterministic sync mismatches on explicit
  per-item approval. Hands off to nothing.
disable-model-invocation: true
---

# upkeep

You are running the TStack plugin's **self-review**. You review **this repository — the
TStack plugin itself** — not a consumer project. You are manual-only: a human typed
`/upkeep`. Your job is to catch (1) internal inconsistency and drift between the repo's
many duplicated facts, (2) regressions of the framework's design invariants and cross-skill
contracts, and (3) staleness against the fast-moving Claude ecosystem and tech-stack
defaults. You **report first**; you apply only safe, mechanical fixes, and only after
explicit per-item approval.

**You are not** `tstack-audit` (a planned skill that audits a *consumer's* product for
production-readiness), nor `tstack-status` / `tstack-wrap` (which run on *consumer* projects
via the shipped plugin). Those operate on someone's product repo; you operate on the plugin's
own source and never ship.

## Prereq check

1. Confirm the working directory is the TStack plugin repo: `.claude-plugin/plugin.json`
   exists and its `name` is `"tstack"`. If not, stop and say this skill only runs inside the
   TStack plugin repo.
2. **Read `CLAUDE.md` fresh** and treat its **Design Invariants**, **Cross-skill contracts**,
   and **Editorial Conventions** sections as the *authoritative checklist source* for Group A
   below. Do **not** hardcode those rules into this skill — they evolve, and CLAUDE.md is the
   single source of truth. If CLAUDE.md has changed since this skill was written, follow
   CLAUDE.md.
3. Record today's date, the current `version` from `plugin.json`, and the Claude Code version
   if observable. These go in the report header.

## Audit dimensions

Two groups. **Group A is deterministic** (answerable from the repo alone — no web). **Group B
needs live, fetched knowledge** because the base model's training cutoff lags the real
ecosystem.

### Group A — internal consistency (repo-only)

- **A1 — Skill-shape conformance.** Every `skills/tstack-*/SKILL.md` follows the mandated shape
  (frontmatter → Prereq check → Approach → Reference handoff → Handoff) and its frontmatter
  `description` leads with what-it-does → "Use when…" → Input/Output → handoff. Flag missing or
  out-of-order sections. (Source of the rule: CLAUDE.md § Editorial Conventions.)
- **A2 — Design-invariant regressions.** Per CLAUDE.md § Design Invariants: verification stays
  external and quoted (`tstack-build` pastes real command output; `tstack-roadmap` refuses
  non-command-verifiable "Done when"); state lives on disk, not session (no reintroduced
  `.tstack/state.json` or session-memory coordination); fresh session per milestone (plan/build
  end and recommend a fresh session, no auto-chain glide). Quote the offending line if softened.
- **A3 — Cross-skill contracts.** Per CLAUDE.md § Cross-skill contracts: `AGENTS.md` written in
  full only by `tstack-architect` (roadmap/build touch only `## Current Focus`); approved plan
  at `docs/plans/{id}-plan.md` (plan writes, build reads); `ROADMAP.md` has one regenerator
  (roadmap) + one append-only editor (specify) with a `Docs last synced:` marker;
  `tstack-ingest` writes only to `docs/_adopted/`; four foundational ADRs in `tstack-architect`;
  `M0` (and `i0` for iOS) mandatory in `tstack-roadmap`.
- **A4 — Editorial conventions.** Full-word filenames (no `PRD.md`); `tstack-product` vs
  `tstack-specify-feature` keep their mutually-exclusive negative-phrase triggers; AI-feature
  acceptance criteria use the eval-based format (eval set + quality bar + fallback), not
  Given/When/Then.
- **A5 — Sync-point drift** (the known drifters — these are the highest-yield auto-fixables):
  - `version` in `.claude-plugin/plugin.json` vs the latest released heading in `CHANGELOG.md`.
  - The skill-count wording ("Seven-skill chain", "three … off-chain companions") in
    `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `README.md`, and
    `CLAUDE.md` — all four must agree with each other and with reality.
  - The README **mermaid lifecycle diagram**, the **skill table**, and the **repo-layout tree**
    must each enumerate exactly the skills present. Diff each against `ls skills/`.
- **A6 — CHANGELOG discipline.** `## [Unreleased]` exists; recent behavior/manifest/README
  changes have entries. (This is also the gate for your own auto-fixes — see Report-vs-fix.)

### Group B — external currency (web + the `claude-api` skill)

- **B1 — Claude model currency.** Are the Claude model tiers/IDs TStack references current?
  Touchpoints include `tstack-architect/references/example-output.md` (the "Claude (Haiku)"
  slug-suggestion ADR and tech-stack rows), `tstack-discover/references/example-output.md`
  (cost line), and the AI-strategy guidance in `tstack-architect/SKILL.md`. **Use the built-in
  `claude-api` skill** as the authoritative source for current model IDs/tiers, cross-checked
  with a fetched Anthropic docs page.
- **B2 — Claude Code feature currency.** TStack uses plan mode and a `Stop` hook but does not
  reference subagents, MCP, output styles, or the Skill tool. Flag where a *current* Claude Code
  feature should plausibly be referenced (e.g. should plan/build mention subagents for parallel
  exploration? should the AI-strategy spec mention MCP?), and verify the README hook jq paths
  against the current Claude Code hook schema (the README itself warns these field names shift).
- **B3 — Tech-stack major-version currency.** Re-validate the default-stack table in
  `tstack-architect/SKILL.md` and the dated phrasing ("current as of early 2026", "2026
  default") against current **major** versions (Next.js, Xcode, Pydantic, Tailwind, etc.).
  **Vendor pricing / free-tier drift is out of scope** — too noisy to verify authoritatively.

## Execution model

Run Claude to its fullest — fan out, and verify currency against fetched sources, never memory.

1. **Phase 0 — snapshot** (main thread): the prereq check above.
2. **Phase 1 — parallel fan-out:** launch one subagent per dimension **in a single batch**
   (Group-A agents are fast repo reads; Group-B agents do the slow network work concurrently).
   Group-B agents **must** invoke the `claude-api` skill and/or `WebFetch` the primary source —
   a search snippet alone is not enough; fetch the authoritative page.
3. **Phase 2 — adversarial freshness gate:** before any "X is stale" claim graduates from
   candidate to confirmed, the agent must (a) establish the **live current value** from an
   authoritative fetched source, (b) quote the **repo's value** with `file:line`, and (c) show
   live ≠ repo. Anything that fails the gate is reported as **unconfirmed — needs human check**,
   never as a fix. (This holds you to the same external-verification bar the repo enforces on
   `tstack-build`.)
4. **Phase 3 — synthesis:** merge, de-duplicate, prioritize by the severity ladder, and split
   findings into auto-fixable vs recommend-only.

## Report-vs-fix policy

You are **report-first**. You apply edits only after explicit per-item approval, in two tiers.

**MAY auto-fix on approval — safe, deterministic, one-right-answer:**
- A5 sync-point mismatches: a skill-count word that disagrees with `ls skills/`; a README
  diagram/table/tree that omits or misnames a skill that demonstrably exists; a
  `version` ↔ CHANGELOG heading transcription mismatch.
- **Every applied fix carries its `## [Unreleased]` CHANGELOG entry in the same change.** If a
  fix touches a manifest (`plugin.json` / `marketplace.json`), note whether a version bump is
  warranted but **never bump unilaterally** — release tagging is a human decision.

**Recommend-only — never auto-applied (judgment / editorial):**
- Changing a default model tier, default stack, or pinned version (B1/B3); deciding whether to
  *add* feature references like subagents/MCP/output styles (B2); rewording dated guidance.
- **Invariant or contract regressions (A2/A3).** Flag these loudly as P0, but never auto-fix —
  repairing one requires understanding *why* it regressed; a mechanical revert could be wrong.

## Output

Write the report to **`reviews/upkeep-YYYY-MM-DD.md`** (today's date). Never overwrite a prior
run — the `reviews/` folder is a diffable history (the first entry, `upkeep-2026-05-30.md`, is
the original advisory review, preserved). Report structure:

1. **Run header** — date, repo `version`, Claude Code version observed, the model IDs/feature
   facts you treated as current, each with its source URL or `claude-api`.
2. **Executive summary** — counts by severity.
3. **Group A findings** — table: dimension · `file:line` · expected · found · severity · auto-fixable?
4. **Group B findings** — table: item · repo value (`file:line`) · current live value · source · verdict (confirmed-stale / fresh / unconfirmed) · recommendation.
5. **Auto-fix manifest** — the exact deterministic edits proposed, each with the CHANGELOG entry it would add.
6. **Recommend-only list** — judgment items with rationale.
7. **Open questions.**

Always also print a tight **chat summary** + an approval prompt (e.g. "N findings: X auto-fixable
sync mismatches, Y confirmed-stale currency items, Z unconfirmed. Approve auto-fixes? (all /
specific items / none)"). Apply approved fixes, then report what changed.

**Severity ladder:** P0 invariant/contract regression · P1 confirmed currency staleness ·
P2 sync-point drift · P3 editorial/shape · P4 unconfirmed currency (human-check).

## Handoff

Hands off to nothing. Manual, repo-local maintenance utility — run it before a release, after a
big edit, or whenever you want to confirm the plugin is still internally coherent and current.
