# Changelog

All notable changes to the TStack plugin are recorded here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); this project uses
[semantic versioning](https://semver.org/).

When you edit a skill or change plugin behaviour, add an entry under **Unreleased**.
Move the Unreleased entries under a new version heading when you tag a release
(and bump `version` in `.claude-plugin/plugin.json`).

## [Unreleased]

### Changed
- **iOS-aware structure and infrastructure milestones.** `tstack-architect` still emits a complete, buildable repo tree but now labels it a *"baseline an implementer may restructure (record as an ADR + update this section)"* rather than a fixed contract — never "TBD", never omitted (feedback item **T2**). `tstack-roadmap` now splits every infrastructure milestone's "Done when" into two labeled groups — ***implementation satisfies*** (provable by the build/test/lint/CI run) vs ***owner configures externally*** (GitHub/host/Apple console settings, attested not command-checked) — so a milestone stops reading as "not done" because of a provider toggle the code can't flip, and a real implementation gap can't hide behind one (**T3**). For iOS-primary projects, `tstack-roadmap` now emits the mandatory baseline as **`i0 — iOS infrastructure baseline`** (the iOS analog of M0: `xcodebuild`/`swift test`/SwiftLint in CI, signing/secrets, archivable app skeleton, crash reporting, test runner), described as a normal hand-implementable milestone and using the same implementation-vs-external split (**T6**). `tstack-architect` now gives **each deployable app its own top-level folder at the repo root** (`web/`, `apple/`, `android/`) — not nested under an `apps/` wrapper, and never the lone app dumped loose at the root — with shared cross-platform code under `shared/`/`packages/`. This holds even for a single app, so expanding to a second platform later is additive, not a restructure (platform-at-root suits T-Stack's typically-polyglot projects, where a Swift app and a web app share no workspace tool and an `apps/` parent only adds nesting). The Apple folder is named `apple/` rather than `ios/` since one Swift codebase often spans iOS/iPadOS/macOS/watchOS. The Slink example across all skills' `references/example-output.md` is updated to root-level folders to match. Updates both skills' SKILL.md, `references/full-guide.md`, and `references/example-output.md`.
- **`tstack-architect` now frames foundational and stack ADRs as authoritative *but revisable*.** Every ADR (the four foundational ones *and* every tech-stack ADR) now carries a mandatory, prominent `Revisit when:` trigger — previously only tech-stack ADRs did. DECISIONS.md opens with a standard **"How decisions change here"** header documenting a low-ceremony supersession path (add a new ADR that supersedes the old, flip the old one's status to `Superseded by ADR-N`, update affected docs) and an **append-only numbering rule** (next ADR = highest existing + 1, never renumbered or reused) — so a later implementer (another agent, a teammate, or future-you adopting a DI framework) can override a default cleanly without colliding with a future T-Stack ADR or reading like rule-breaking. CONVENTIONS.md generation now separates genuine anti-patterns (correctness/security/bug-class rules) from *project choices* (stack/structure decisions), which are stated as defaults-with-rationale that point to their ADR rather than branded inviolable prohibitions. Addresses feedback items **T1** and **T5**. Updates SKILL.md, `references/full-guide.md`, and `references/example-output.md`.

## [0.4.0] — 2026-06-20

### Added
- **`tstack-ingest` — adoption on-ramp for projects that already have docs.** Reads a user's pre-existing material (a PRD, discovery notes, design/architecture docs in any format), reflects intent back to catch a stale-after-pivot doc, classifies coverage (Present / Thin / Missing with named gaps), and distills it into an *unratified draft* at `docs/_adopted/` — never a canonical doc. `tstack-product` then authors `docs/PRODUCT.md` from the draft through its existing requirements gate and deletes the draft on save, so adopted content still passes verification before entering the loop. Foreign technical/design material is staged as `docs/_adopted/*-notes.md` and routed to `tstack-architect` (which asks its four ADR questions fresh — adoption can't smuggle a project past them). It's a **chain on-ramp** (required handoff to `tstack-product`), not an off-chain skill.

### Changed
- **`tstack-product` is adoption-aware.** If `docs/_adopted/PRODUCT.draft.md` exists, it ratifies the draft through its full gate (resolving every `## Open gaps` item) instead of re-interviewing, and removes the draft on save. Remains the sole author of `PRODUCT.md`.
- **`tstack-discover` forwards existing docs to `tstack-ingest`.** Its old "distill existing material into business-brief.md in one pass" branch is removed; when the user already has docs (or points discover at a file/folder), discover detects and hands off to ingest rather than distilling itself — a single home for adoption, no trigger collision.
- **CLAUDE.md** records the adoption contract (`tstack-ingest` writes only to `docs/_adopted/`, never canonical files) and classifies ingest as a chain on-ramp distinct from the three off-chain companions.
- **README, `plugin.json`, `marketplace.json`** updated for the new skill (lifecycle diagram on-ramp node, skill table, repo-layout tree, and count wording).

## [0.3.2] — 2026-06-14

### Changed
- **Repositioned around the agentic loop (docs only, no behavior change).** README gains a "The agentic loop" section naming the three properties that make TStack's loop trustworthy — durable on-disk context, verification as the engine, and a fresh session per iteration — with an honest note on the "loop engineering" / context-engineering / harness terminology. CLAUDE.md records the same three as **design invariants** future edits must preserve. Also fixed a stale "two off-chain companions" reference (now three — includes `tstack-wrap`).

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
