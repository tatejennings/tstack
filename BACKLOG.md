# BACKLOG

Things to do next, ordered by rough priority. This is solo-dev shorthand — when something gets picked up, move it to a real branch and update this file when it ships.

## Shipped — review-driven hardening pass

From a deep review of the skill chain (see `REVIEW.md`). All advisory; needs the end-to-end verification under "Verify v0.2" before tagging.

- **Seam fixes:** in-repo plan artifact at `docs/plans/{id}.md` (`plan` writes, `build` reads — commit it to hand planned-ahead milestones to a cloud agent); `roadmap`↔`specify` contract (roadmap regenerates + `Docs last synced:` marker, specify is append-only); `AGENTS.md` ownership (architect owns; roadmap/build touch only `## Current Focus`); `tstack-roadmap` description now lists CONVENTIONS.md.
- **Quality gates:** `roadmap` checks "Done when" testability at write time; `product` detects AI features early + has a completion sign-off; `build` has a controlled criterion-waiver policy + concrete team-PR flow.
- **Guides:** architect `full-guide.md` de-staled (TESTING.md + foundational ADRs added, opinionated-default contradiction fixed, dead `0x-*.md` refs removed); product `full-guide.md` thickened (deep acceptance-criteria section incl. AI eval depth + subjectivity). Both carry a "SKILL.md is source of truth" header.
- **Polish:** tech-stack defaults now dated + ask-ecosystem-first; standardized session-boundary thresholds; discover has when-NOT-to-run + brief-ready gate + operationalized no-WebSearch fallback.
- **New off-chain skills:** `tstack-design` (UX spec + paste-ready Claude Design prompts) and `tstack-status` (read-only inspector with doc-drift detection).

## Verify v0.2

- [ ] End-to-end test in a scratch repo — full chain (`discover` → `product` → `architect` → `roadmap` → `plan` → `build`) for a small real product. Confirm: the four foundational ADRs are produced, TESTING.md is generated, M0 is enforced, AI branch fires correctly when answered yes, verification report is properly quoted.
- [ ] Negative tests from the v0.2 plan: invoke `tstack-roadmap` without M0, expect refusal; invoke `tstack-specify` and check the "considered, no change" line appears.
- [x] Cut `0.2.0` — `plugin.json` bumped, `CHANGELOG.md` `[0.2.0]` dated. **Still to do:** `git tag v0.2.0` on GitHub, and run the verification above (shipped ahead of it).

## Evolution pass — from user feedback (2026-06)

Four improvements surfaced by a real-world weekend build (user got to milestone 22 with ~20 more queued). All validated against current best-practice research; tackle one at a time, deeply. Order = rough priority.

1. [x] **`tstack-wrap` — session doc-sweep skill** *(shipped in v0.3.0).* New off-chain companion that closes out a work session: sweeps for undocumented decisions / tradeoffs / gotchas / operational events (stale branches, deferred criteria, manual steps) from the session's work and writes each gap to the right home (DECISIONS.md ADR, CONVENTIONS.md, a code comment, ROADMAP.md Status). Does not commit; routes new scope/milestones to `tstack-specify`. Built from a user's end-of-session sweep prompt ("I catch something almost every time"). `tstack-status` only *detects* drift — nothing currently *closes* it, and staleness is the chain's #1 risk.
2. [ ] **Tame auto-chaining + recommend fresh session per milestone.** `tstack-plan` step 7 and `tstack-build` step 5 currently invite a continuous one-window plan→build→plan loop ("Same session is fine", "typically immediately after tstack-plan finishes"). User reported this bloats context more than expected (and the agent sometimes auto-advances milestones). Flip the default handoff to *stop and recommend a fresh session*; the in-repo plan artifact already makes fresh context cheap. Loop-hygiene, not a discipline change.
3. [ ] **Delivery skills — UAT / manual testing / user docs.** The chain ends at "merged + roadmap updated"; there's no handoff-to-a-human stage, so the user built their own: install/UAT ("how do I install/test, what should work?"), optional manual testing (env setup + steps), and end-user docs (what it does, how to install/use). Decide shape in its own session — one `tstack-deliver` vs three skills (`tstack-uat`, `tstack-manual-test`, `tstack-userdocs`). Adjacent to (but distinct from) the backlogged `tstack-audit`, which is production-readiness, not human handoff.
4. [ ] **Loop-engineering positioning pass.** Editorial only. Reframe README + CLAUDE.md around the agentic loop: docs = durable external context, the "Done when" verify step = the engine, fresh session per milestone = the iteration. ("Loop engineering" is a soft community term; the substance — strong external verification + state-on-disk — is already TStack's core. Make it legible.) No behavior change.

## New skills (deferred from v0.2)

- [ ] `tstack-audit` — production-readiness review. Runs over an established TStack project and checks: are ADRs honored in code? Is observability actually wired? Are a11y tests in CI? Surfaces drift between docs and reality.
- [ ] `tstack-techdebt` — periodic tech-debt audit. Reviews DECISIONS.md for ADRs that have aged poorly (e.g., "we picked Mongo for v1, now we have transactional pain"). Flags them and suggests a remediation milestone for ROADMAP.md.
- [ ] `tstack-retire` — feature retirement. Inverse of `tstack-specify`: walks the doc set to remove a deprecated feature cleanly (PRODUCT.md scope, API.md endpoints, specs, ROADMAP entries, code paths).

## Content gaps to close

- [ ] `references/full-guide.md` refresh pass. **Architect and product guides done** in the hardening pass (de-staled + source-of-truth headers). **Still pending:** the discovery guide's "web search works best on Claude.ai" line and a check of the roadmap/build guides for SKILL.md contradictions.
- [ ] AGENTS.md format in `tstack-architect/references/full-guide.md` is dated. Should cover: MCP server configs, hook patterns, skill recommendations, model-routing preferences, worktree conventions for parallel agent work.
- [ ] Extract inline templates (business-brief, PRODUCT, ARCHITECTURE, API, ROADMAP, milestone-entry) into separate `references/*-template.md` files. Currently embedded in full-guide.md; harder to update.
- [ ] Add a 5th and 6th foundational ADR question: **performance budgets** (Core Web Vitals targets, server p95 latency, mobile Lighthouse minimums) and **cost ceilings** (monthly infra + AI budget, per-user cost target).
- [ ] Make i18n/l10n a 7th foundational ADR for consumer-facing products.
- [ ] Database migrations / zero-downtime-deploy strategy as a mandatory `2 - Specs/migrations.md` for any project with persistence.

## Bigger pieces

- [ ] **Monorepo support.** Currently the framework assumes a flat repo. Add guidance for `apps/web` + `apps/api` + `packages/*` layouts — where docs live, whether PRODUCT.md is shared or per-package, how ROADMAP.md handles cross-package milestones. At minimum, a one-page reference; ideally a Minimum/Standard/Full split that's monorepo-aware.
- [ ] **iOS-first variant.** The current opinionated stack table is web-leaning. A real-world iOS project would want a Swift-specific flavor of TESTING.md (XCTest, ViewInspector, snapshot testing), a different M0 (App Store Connect setup, TestFlight, code signing), and architecture patterns that reflect SwiftUI/SwiftData.
- [ ] **Marketplace listing.** Add `homepage` and `repository` fields to `plugin.json`. Publish to the Claude Code marketplace once it's stable.
- [ ] **Docs site beyond README.** Once there's >3 examples and a couple of variants (web/iOS), the README will outgrow itself. Cover with a docs site (Mintlify, Fumadocs, or just a `docs/` site on Vercel).

## Process / housekeeping

- [ ] After a real consumer-project run, decide if `tstack-architect` needs splitting into `tstack-architect-core` + `tstack-architect-specs` (the v0.2 plan said reassess after one real-world run; this is the trigger).
- [ ] Decide whether to keep the `git mv`-preserved `references/full-guide.md` files indefinitely or replace them with rewritten content as the SKILL.md files drift further from the originals.
- [ ] Set up Dependabot or Renovate on the plugin repo (low priority — there are no real dependencies, but worth doing if examples grow).

## Maybe / not sure

- [ ] **Trigger-collision integration tests.** Could write a small harness that simulates user phrasing and checks which skill fires. Probably overkill for a solo plugin; revisit if collisions become a real problem.
- [ ] **`.tstack/state.json`** — we removed every reference in v0.2 because it was a ghost artifact. If we ever want skills to coordinate across sessions automatically (vs. relying on doc presence), this is where it'd come back. Wait until the use case is concrete.
- [ ] **Visual asset generation** — could `tstack-architect` produce a Mermaid version of ARCHITECTURE.md's data-flow diagram instead of ASCII art? Trade: prettier rendering vs. another tool dependency. Not now.
- [ ] **Plugin self-test.** A skill that runs the full chain against a built-in fixture product and asserts the outputs are well-formed. Useful for CI of the plugin itself. Probably not worth it until others contribute.

---

## How this file is used

- New idea or critique → add a bullet here.
- Picking something up → move to a branch, work it, ship it, delete the bullet (or move to a `## Shipped in vX.Y` section if context matters).
- Yearly review → prune anything that hasn't moved in 12 months.
