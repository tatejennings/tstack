# BACKLOG

Open work only, grouped by priority. Solo-dev shorthand. When something ships, delete its bullet — `CHANGELOG.md` is the historical record, this file holds only what's left to do.

## Now — pick up next

- [ ] **Verify the chain end-to-end in a scratch repo** (quality gate). Full chain (`discover` → `product` → `architect` → `roadmap` → `plan` → `build`) for a small real product. Confirm: the four foundational ADRs are produced, TESTING.md is generated, M0 is enforced, the AI branch fires when answered yes, and the verification report is properly quoted. Plus negative tests: invoke `tstack-roadmap` without M0 → expect refusal; invoke `tstack-specify` → check the "considered, no change" line appears.
- [ ] **Delivery skills — UAT / manual testing / user docs** (new skill[s]). The chain ends at "merged + roadmap updated"; there's no handoff-to-a-human stage. Three needs: install/UAT ("how do I install/test, what should work?"), optional manual testing (env setup + steps), and end-user docs (what it does, how to install/use). Decide shape in its own session — one `tstack-deliver` vs. three skills (`tstack-uat`, `tstack-manual-test`, `tstack-userdocs`). Adjacent to but distinct from `tstack-audit` (production-readiness, not human handoff).

## Next

- [ ] **`tstack-ingest` v2 — adoption-aware `tstack-architect`** (extends the shipped on-ramp). v1 is product-grade only: foreign architecture/design docs are staged as `docs/_adopted/*-notes.md` and always routed to `tstack-architect`, which asks its four ADR questions fresh (no adoption of technical docs — this is what structurally prevents ADR/M0 smuggling). v2 would let architect *ratify* a rich foreign ARCHITECTURE.md from a draft (the same pattern product uses), while still forcing the four ADR questions. Reviewers flagged foreign arch docs rarely clear the bar and this reopens the ADR-enforcement surface — so only do it if real users show up with adopt-grade architecture docs. Decide at build time.
- [ ] **`tstack-audit`** (new skill) — production-readiness review. Runs over an established TStack project and checks: are ADRs honored in code? Is observability actually wired? Are a11y tests in CI? Surfaces drift between docs and reality.
- [ ] **`tstack-techdebt`** (new skill) — periodic tech-debt audit. Reviews DECISIONS.md for ADRs that have aged poorly (e.g., "we picked Mongo for v1, now we have transactional pain"). Flags them and suggests a remediation milestone for ROADMAP.md.
- [ ] **`tstack-retire`** (new skill) — feature retirement. Inverse of `tstack-specify`: walks the doc set to remove a deprecated feature cleanly (PRODUCT.md scope, API.md endpoints, specs, ROADMAP entries, code paths).
- [ ] **`references/full-guide.md` refresh pass** (content gap). Still pending: the discovery guide's "web search works best on Claude.ai" line and a check of the roadmap/build guides for SKILL.md contradictions. (Architect and product guides already done.)
- [ ] **AGENTS.md format in `tstack-architect/references/full-guide.md` is dated** (content gap). Should cover: MCP server configs, hook patterns, skill recommendations, model-routing preferences, worktree conventions for parallel agent work.
- [ ] **Extract inline templates** (content gap) — business-brief, PRODUCT, ARCHITECTURE, API, ROADMAP, milestone-entry — into separate `references/*-template.md` files. Currently embedded in full-guide.md; harder to update.
- [ ] **Add a 5th and 6th foundational ADR question** (content gap): **performance budgets** (Core Web Vitals targets, server p95 latency, mobile Lighthouse minimums) and **cost ceilings** (monthly infra + AI budget, per-user cost target).
- [ ] **Make i18n/l10n a 7th foundational ADR** (content gap) for consumer-facing products.
- [ ] **Migrations / zero-downtime-deploy strategy as a mandatory `2 - Specs/migrations.md`** (content gap) for any project with persistence.

## Later

- [ ] **Monorepo support** (bigger). **Partly shipped in v0.5.0** — `tstack-architect` now emits a platform-at-root layout (`web/` / `apple/` / `android/` + shared `packages/`) that's expansion-ready by default, so the framework no longer assumes a single flat app. Remaining: where docs live in a multi-package repo, whether PRODUCT.md is shared or per-package, and how ROADMAP.md handles cross-package milestones; ideally a Minimum/Standard/Full split that's monorepo-aware.
- [ ] **iOS-first variant** (bigger). The current opinionated stack table is web-leaning. **The iOS infrastructure baseline shipped in v0.5.0** — `tstack-roadmap` emits an `i0 — iOS infrastructure baseline` milestone (App Store Connect, TestFlight, code signing) with the implementation-vs-external "Done when" split. Remaining: a Swift-specific flavor of TESTING.md (XCTest, ViewInspector, snapshot testing) and architecture patterns that reflect SwiftUI/SwiftData.
- [ ] **Marketplace listing** (infra). Add `homepage` and `repository` fields to `plugin.json`. Publish to the Claude Code marketplace once it's stable.
- [ ] **Docs site beyond README** (bigger). Once there's >3 examples and a couple of variants (web/iOS), the README will outgrow itself. Cover with a docs site (Mintlify, Fumadocs, or just a `docs/` site on Vercel).
- [ ] **Reassess `tstack-architect` split** (process) into `tstack-architect-core` + `tstack-architect-specs`. The v0.2 plan said reassess after one real-world run; that's the trigger.
- [ ] **Decide fate of the `git mv`-preserved `references/full-guide.md` files** (process) — keep them indefinitely, or replace them with rewritten content as the SKILL.md files drift further from the originals.
- [ ] **Set up Dependabot or Renovate** (infra, low priority) — no real dependencies yet, but worth doing if examples grow.

## Maybe / not sure

- [ ] **Trigger-collision integration tests.** Could write a small harness that simulates user phrasing and checks which skill fires. Probably overkill for a solo plugin; revisit if collisions become a real problem.
- [ ] **`.tstack/state.json`** — we removed every reference in v0.2 because it was a ghost artifact. If we ever want skills to coordinate across sessions automatically (vs. relying on doc presence), this is where it'd come back. Wait until the use case is concrete.
- [ ] **Visual asset generation** — could `tstack-architect` produce a Mermaid version of ARCHITECTURE.md's data-flow diagram instead of ASCII art? Trade: prettier rendering vs. another tool dependency. Not now.
- [ ] **Plugin self-test.** A skill that runs the full chain against a built-in fixture product and asserts the outputs are well-formed. Useful for CI of the plugin itself. Probably not worth it until others contribute.

---

## How this file is used

- New idea or critique → add a bullet to the right tier (Now / Next / Later / Maybe).
- Picking something up → move to a branch, work it, ship it, then **delete the bullet**. `CHANGELOG.md` is the shipped record; this file holds only open work.
- Periodic review → re-tier anything whose priority has shifted; prune stale Maybe items.
