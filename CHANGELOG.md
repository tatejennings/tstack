# Changelog

All notable changes to the TStack plugin are recorded here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); this project uses
[semantic versioning](https://semver.org/).

When you edit a skill or change plugin behaviour, add an entry under **Unreleased**.
Move the Unreleased entries under a new version heading when you tag a release
(and bump `version` in `.claude-plugin/plugin.json`).

## [Unreleased]

## [0.3.1] — 2026-06-14

### Changed
- **Per-milestone loop now defaults to a fresh session per step instead of chaining in one window.** `tstack-plan` ends after writing the plan and recommends building in a fresh session (no rolling straight into `tstack-build`); `tstack-build` ends after shipping and recommends planning the next milestone in a fresh session (no auto-advancing). The on-disk plan (`docs/plans/{id}.md`) + ROADMAP status already carry all state across the boundary, so nothing is lost. Same-session work is still allowed when the user explicitly asks. Addresses user-reported context bloat from the skills auto-chaining plan→build→next-milestone. README Quick Start updated to match.

## [0.3.0] — 2026-06-14

### Added
- `tstack-wrap` — optional, off-chain session doc-sweep skill. At the end of a session it sweeps the work just done (conversation + `git log` + `docs/`) for undocumented decisions, tradeoffs, gotchas, and operational events (stale branches, deferred criteria, manual steps), then writes each genuine gap to its right home — a dated ADR in `DECISIONS.md`, a pattern in `CONVENTIONS.md`, a code comment, or `ROADMAP.md`'s Status section. It **does not commit**, never adds or renumbers roadmap milestones, and routes new scope to `tstack-specify`. Complements `tstack-status` (which only *detects* drift, read-only) by *closing* the gap. Addresses the doc-staleness risk surfaced by user feedback. Manual by default; the README documents an optional opt-in `Stop` hook for consumers who want a wrap-up nudge at session end.

## [0.2.0] — 2026-05-30

### Added
- `tstack-design` — optional, off-chain skill that produces a design/UX spec (`docs/2 - Specs/design.md`) plus ready-to-paste prompts and context for Claude Design. Invokable at any point; not part of the chain.
- `tstack-status` — optional, off-chain, read-only inspector. Reports project status, planned-ahead plans, missing mandatory docs, and **doc drift** (e.g. PRODUCT.md edited after ROADMAP.md's last sync). Writes nothing.
- Foundational ADRs in `tstack-architect` (security, observability, accessibility, privacy) + optional AI/LLM ADR; mandatory `TESTING.md` and `DECISIONS.md` at every right-sizing level.
- `M0 — Infrastructure baseline` mandate in `tstack-roadmap` (refuses to save a roadmap without it).
- Eval-based AI/LLM acceptance criteria (eval set + quality bar + fallback) in `tstack-product`; deep acceptance-criteria guidance incl. the subjectivity question.

### Changed
- **Plan artifact** persisted in-repo at `docs/plans/{id}.md` (`tstack-plan` writes, `tstack-build` reads). Committing is a manual step so planned-ahead milestones can be handed to a cloud agent.
- **`ROADMAP.md` ownership contract:** `tstack-roadmap` regenerates and carries a `Docs last synced:` marker; `tstack-specify` is append-only (never renumbers).
- **`AGENTS.md` ownership:** `tstack-architect` owns the file; `tstack-roadmap`/`tstack-build` update only the `## Current Focus` block.
- `tstack-roadmap` validates "Done when" testability at write time; description now lists `CONVENTIONS.md`.
- `tstack-product` flags AI features early and requires a completion sign-off before saving.
- `tstack-architect` asks the team's ecosystem before proposing a stack, dates each tech-stack ADR, and consolidates related choices into one ADR.
- `tstack-build` gained a controlled criterion-waiver policy and a concrete team-PR flow.
- `tstack-discover` gained a when-NOT-to-run redirect, a brief-ready gate, and an operationalized no-WebSearch fallback.
- Standardized the fresh-session threshold across the setup-chain handoffs.
- De-staled the `architect` `full-guide.md` (added TESTING.md + foundational ADRs, fixed the opinionated-default contradiction, removed dead pre-plugin file references) and thickened the `product` `full-guide.md`; both now name SKILL.md as the source of truth.

## [0.1.0]

### Added
- Initial release: the TStack skill chain — `tstack-discover`, `tstack-product`, `tstack-architect`, `tstack-roadmap`, `tstack-plan`, `tstack-build`, plus the `tstack-specify` iteration loop.
