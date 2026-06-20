# BACKLOG

Open work only, grouped by priority. Solo-dev shorthand. When something ships, delete its bullet — `CHANGELOG.md` is the historical record, this file holds only what's left to do.

## Now — pick up next

- [ ] **Ingest existing docs — adopt a project mid-chain** (new skill; name TBD — `tstack-adopt` / `tstack-ingest` / `tstack-onboard-docs`). Today the chain assumes you start from a rough idea and walk every stage. Many users arrive with docs already written — discovery notes, a PRD, design docs, even an architecture sketch — in *their* format, not TStack's. This skill reads what they have, maps it onto the TStack doc set (business-brief / PRODUCT / ARCHITECTURE / DECISIONS / design.md / …), reports the gap ("you have product + design, missing architecture + roadmap"), and **resumes the chain at the right step** rather than forcing a from-scratch restart. Decide shape in its own session. Open questions: (1) map vs. regenerate — convert their content into TStack-shaped docs, or keep theirs and only fill gaps? (2) confidence/coverage check — how to tell a real PRODUCT.md from a thin one, and flag sections too sparse to build on. (3) entry-point routing — does it hand off to `product` / `architect` / `roadmap` / `plan` depending on what's present? (4) overlap with `tstack-specify` (amends an existing TStack project) and `tstack-status` (reports drift) — this is the *adoption* front door for non-TStack docs, keep it distinct. Off-chain front-door utility that feeds *into* the chain.
- [ ] **Verify the chain end-to-end in a scratch repo** (quality gate). Full chain (`discover` → `product` → `architect` → `roadmap` → `plan` → `build`) for a small real product. Confirm: the four foundational ADRs are produced, TESTING.md is generated, M0 is enforced, the AI branch fires when answered yes, and the verification report is properly quoted. Plus negative tests: invoke `tstack-roadmap` without M0 → expect refusal; invoke `tstack-specify` → check the "considered, no change" line appears.
- [ ] **Delivery skills — UAT / manual testing / user docs** (new skill[s]). The chain ends at "merged + roadmap updated"; there's no handoff-to-a-human stage. Three needs: install/UAT ("how do I install/test, what should work?"), optional manual testing (env setup + steps), and end-user docs (what it does, how to install/use). Decide shape in its own session — one `tstack-deliver` vs. three skills (`tstack-uat`, `tstack-manual-test`, `tstack-userdocs`). Adjacent to but distinct from `tstack-audit` (production-readiness, not human handoff).

## Next

- [ ] **Autonomous milestone runner (opt-in)** (new skill) — *name TBD; candidates `tstack-relay` (best metaphor: builds, then hands the PR baton to a human), `tstack-run`, `tstack-solo`, `tstack-cruise`, `tstack-drive`. Not "autopilot" (overused).* Generalized from the personal `sorted-loop-poc/.claude/commands/autopilot.md`. Takes a milestone ID, runs unattended, opens a PR — **never merges**. Fills TStack's missing autonomous mode. **Now unblocked: both sequencing prerequisites have shipped — the fresh-session default (v0.3.1) and the loop-engineering positioning it embodies (v0.3.2).** **Red-teamed (3 adversarial agents); the design below is the corrected one — do not regress it.**
   - **Reuse contracts, not skills.** "Compose tstack-plan + tstack-build" is fiction — `tstack-plan` *is* its human-approval gate, which autonomy must skip. Reuse the *artifact contracts*: the `docs/plans/{id}.md` schema, the quoted "Done when" verification-report format, and the AGENTS.md/ROADMAP/M0 rules. The planning + loop control flow is new code; don't claim single-source-of-truth via skill composition.
   - **Plan phase = generate → adversarially verify** (the key design directive). Generate the plan, then run **parallel red-team agents with diverse lenses** (architecture / safety / spec-&-ADR-conformance) to poke holes; proceed only if it survives, else abort with a report. This *replaces the human approval gate* — restoring the cheap early catch (wrong plan caught before hundreds of lines are written). Honest limit: red-team agents are the same model family → a *partial* gate, never a substitute for the build-time gate.
   - **Verify phase = the only authorizing gate.** Quoted "Done when" command output. **Self-review (`/code-review`) gathers findings but is NEVER the gate** (same-model, correlated, incentivized to exit).
   - **Primitive & invocation (decided):** **independent from `tstack-build`** (not a `--unattended` flag — keeps build simple, avoids leakage). **Manual-only — a command, not an auto-triggering skill** (side-effecting/git-mutating; must never auto-fire). Ship under `skills/<name>/SKILL.md` with `disable-model-invocation: true` (preferred, for packaging consistency) or `.claude/commands/<name>.md`; load-bearing property is manual-only. The **engine lives in a script/workflow**, not the markdown — the never-merge invariant and the bounds (retries, budget) are guaranteed there, not in prose.
   - **Fail-closed preconditions — REFUSE if any fails:** dirty tree / not on default branch / behind remote; no branch protection on `main` (else the "human merge gate" is unenforceable); M0 not shipped / no CI; **any "Done when" criterion not command-verifiable**; milestone deps not all Completed, or no committed plan at `docs/plans/{id}.md`; AGENTS.md/CONTRIBUTING forbids autonomous push → **consumer policy wins**.
   - **Enforcement (mechanism, not prose):** worktree isolation; on-disk attempt/token-time budget + resume protocol (never silently restart); secret-scan before push; never force-push / never touch `main` / never merge; tests append-only (weakening/deleting a test aborts); write the handoff report incrementally.
   - **Generalize off the personal version:** project's *own* test/lint/typecheck commands (not hard-coded pnpm); "wait for whatever automated PR reviewer exists, if any" (not Codex-specific); path `docs/plans/` (not `docs/3 - Plans/`).
   - **Sequencing:** one milestone per run, PR-and-stop, never chains.
   - **Open risk (product red-team dissent):** autonomy may be off-brand for a plan-first/anti-vibe-coding framework; some argue this stays a personal command, not a shipped skill. Decide at build time, don't assume. Distinct from `tstack-audit` (production-readiness review, not a build runner).
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

- [ ] **Monorepo support** (bigger). Currently the framework assumes a flat repo. Add guidance for `apps/web` + `apps/api` + `packages/*` layouts — where docs live, whether PRODUCT.md is shared or per-package, how ROADMAP.md handles cross-package milestones. At minimum, a one-page reference; ideally a Minimum/Standard/Full split that's monorepo-aware.
- [ ] **iOS-first variant** (bigger). The current opinionated stack table is web-leaning. A real-world iOS project would want a Swift-specific flavor of TESTING.md (XCTest, ViewInspector, snapshot testing), a different M0 (App Store Connect setup, TestFlight, code signing), and architecture patterns that reflect SwiftUI/SwiftData.
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
