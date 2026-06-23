---
name: tstack-roadmap
description: Reads the full docs/ tree and produces docs/ROADMAP.md — a dependency-sequenced list of milestones with "Read before starting" doc pointers and "Done when" criteria. Always mandates an M0 — Infrastructure baseline milestone covering CI, secrets, deployment skeleton, and observability bootstrap. Use when ARCHITECTURE.md and PRODUCT.md exist and the user asks "what do we build first", "sequence the work", or "make a roadmap". Input is docs/PRODUCT.md + docs/ARCHITECTURE.md + docs/CONVENTIONS.md + docs/DECISIONS.md + docs/TESTING.md (API.md and breakout specs optional); output is docs/ROADMAP.md. Hands off to tstack-plan-milestone.
---

# tstack-roadmap

You are running TStack's roadmap stage. You read every doc the previous skills produced and synthesize them into a strict, dependency-sequenced list of milestones that an implementer (or another instance of you, via `tstack-plan-milestone` → `tstack-build`) can pick up one at a time.

## Prereq check

Required inputs:

```
docs/PRODUCT.md
docs/ARCHITECTURE.md
docs/DECISIONS.md
docs/TESTING.md
docs/CONVENTIONS.md
```

Optional but used if present:

```
docs/API.md
docs/2 - Specs/*.md         (including ai-strategy.md if the product uses AI)
docs/1 - Discovery/business-brief.md
```

If a required input is missing: stop and tell the user to run `tstack-architect` first. (The required set assumes `tstack-architect` has run at the current TStack v0.2 — older projects predating mandatory DECISIONS/TESTING may need a re-run.)

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

### M0 — Infrastructure baseline is mandatory

Every roadmap **must** begin with `M0 — Infrastructure baseline`. Do not generate a roadmap without it. This milestone covers the things that, if deferred, create compounding pain across every feature milestone:

- **CI workflow** — GitHub Actions (or equivalent) running typecheck + tests + lint on every PR
- **Secrets management** — wired up per ADR-1's choice (e.g., Vercel env vars, Doppler, Vault); no secrets in the repo
- **Deployment skeleton** — at minimum, a working deploy of the empty app to the chosen platform (per ARCHITECTURE.md's deployment topology) so subsequent milestones have somewhere to land
- **Observability bootstrap** — per ADR-2: log destination working, error tracker integrated (e.g., Sentry SDK installed and reporting from a deliberate test error), basic dashboard or log view accessible
- **Linting + formatting** — per CONVENTIONS.md: lint/format configured and enforced in CI
- **Test runner** — per TESTING.md: test runner installed, one trivial passing test, CI runs it

**Split M0's "Done when" into two labeled groups** — what the implementation itself proves, vs. what the owner must set in an external console. This is tool-agnostic — the same split holds whether a human or a CI runner does the work — and it keeps a milestone from looking "not done" because a GitHub/host toggle the code can't flip is still pending:

- ***Implementation satisfies*** (provable by the build/test/lint/CI run itself):
  - The CI workflow file is committed; pushing a PR triggers it; it runs typecheck + lint + tests and the status appears on the PR
  - CI fails when lint fails
  - A deliberate test error reaches the error tracker
  - The test runner runs one trivial passing test
- ***Owner configures externally*** (provider console — GitHub/host/Apple settings; marked done by attestation, not a command):
  - Secrets are set in the deploy platform's secret store (none in repo)
  - Any provider-side deploy/distribution setup the platform requires

  *(TStack does not add branch protection or required-review rules to M0. If the user wants them, that's their call to make in their own repo settings — don't suggest them.)*

Refuse to save the roadmap if M0 is absent or its scope is missing any of the above bullets. If the user pushes back ("I don't need CI for a weekend project"), point them at the ADRs they recorded in `tstack-architect` and ask which bullet they want to formally defer with an ADR-style explanation. Don't silently drop infrastructure.

### iOS: the infrastructure baseline is `i0`

When the project's primary or only platform is iOS, emit the mandatory baseline as **`i0 — iOS infrastructure baseline`** (the iOS analog of M0 — it satisfies the M0 mandate above; you don't also emit a `M0`). When the project has *both* a backend/web workstream and an iOS client, keep `M0` for the shared/backend baseline and add `i0` for the iOS workstream, with `i0` depending on `M0` where it consumes shared CI/secrets.

`i0`'s scope is the iOS analog of M0's bullets:
- **CI** running `xcodebuild build` + `swift test` (or `xcodebuild test`) + SwiftLint/SwiftFormat on every PR
- **Signing & secrets** approach decided and wired (no secrets in repo)
- **Buildable app skeleton** — an archivable app target that launches (the "deployment skeleton" analog)
- **Crash/error reporting** bootstrapped per ADR-2 (e.g. a deliberate test error surfaces in the chosen reporter)
- **Test runner** with one trivial passing test, run in CI

`i0`'s "What gets built" describes the baseline as a normal, hand-implementable milestone — *"Stand up the iOS baseline: project skeleton, CI, signing/secrets, crash reporting, and a passing test."* Don't name or assume any particular scaffolding tool — how the baseline gets stood up is the implementer's choice; the milestone only specifies the deliverable. Use the same **implementation-vs-external split** (above) for `i0`'s "Done when": building/testing/linting and a test crash reaching the reporter are *implementation satisfies*; Xcode signing certificates and App Store Connect/TestFlight distribution are *owner configures externally (Apple settings)*.

5. **Naming conventions:**
   - Server/web milestones: `M0`, `M1`, `M2`, …
   - Mobile milestones (separate workstream): `i0`, `i1`, … (or other letter if not iOS)
   - Mobile milestones depend on the server milestones whose APIs they consume — make that dependency explicit.

6. **Include sections:**
   - Header (overview, how to use, prefix conventions)
   - Dependency Graph Diagram (ASCII, parallel tracks if multi-workstream)
   - Milestone Entries (in build order)
   - Parallelization Notes (which milestones can run simultaneously; critical path)
   - **Status** section at the bottom: `Completed:` (none yet), `Up next: M0 — …`, and a `Docs last synced:` line (see below)

   The Status section header carries a sync marker so drift is visible at a glance:

   ```markdown
   ## Status
   Docs last synced: <commit short-SHA or date this roadmap was generated/re-sequenced from the docs>
   Completed: none yet
   Up next: M0 — Infrastructure baseline
   ```

   `tstack-specify-feature` appends milestones surgically without re-running this skill, so the marker tells a reader when the roadmap was last fully reconciled against `docs/`. If PRODUCT.md/ARCHITECTURE.md changed after that point and the change wasn't a surgical `tstack-specify-feature` edit, the roadmap may be stale — re-run `tstack-roadmap` to re-sequence.

7. **Cross-check before saving:**
   - The infrastructure baseline is present — `M0 — Infrastructure baseline`, or `i0 — iOS infrastructure baseline` for an iOS-primary project — with all its bullets covered (or formally deferred with explicit rationale), and its "Done when" split into *implementation satisfies* vs *owner configures externally*
   - Every feature milestone lists the baseline (`M0`/`i0`) directly or transitively in its dependencies
   - Every "Read before starting" reference points to a doc/section that exists
   - **Every "Done when" criterion is testable at write time.** For each criterion, name the command or test that would prove it (e.g., `curl …`, `npm test path`, `playwright test`, `axe` scan). If you can't name one — the criterion is soft ("looks good", "feels fast", "works well") — rewrite it now into something a command can verify, or split out the measurable part. Don't defer this to `tstack-build`; a soft criterion discovered at build time is a roadmap defect caught too late.
   - No milestone exceeds ~2 days of focused work (split if so)
   - No milestone is so granular it's not independently shippable (combine if so)

8. **Save to `docs/ROADMAP.md`.** Commit: `docs: generate ROADMAP.md from project docs`.

9. **Update the `## Current Focus` block** that `tstack-architect` created in `AGENTS.md`. `tstack-architect` owns `AGENTS.md`; downstream skills only update the designated `## Current Focus` block — never restructure the rest of the file. Set it to:

   ```markdown
   ## Current Focus
   Check `docs/ROADMAP.md` — see the Status section at the bottom for the current milestone.
   ```

   If `AGENTS.md` has no `## Current Focus` block (older project), add one. If `AGENTS.md` doesn't exist at all, write the block into `CLAUDE.md` instead.

## Reference handoff

For the full milestone template, "Read before starting" rules, "Done when" rules, and troubleshooting (too-large/too-granular milestones, missing dependencies), read `references/full-guide.md`.

For a realistic example showing the dependency graph format, the mandatory M0 — Infrastructure baseline structure, and the milestone-entry shape, read `references/example-output.md`.

## Handoff

> Roadmap complete — `docs/ROADMAP.md` written with {N} milestones across {workstreams}.
> Up next: M0 — {name}.
>
> **Next: run `tstack-plan-milestone`** (or say "plan milestone M0"). No fresh session needed — `tstack-plan-milestone` reads only the docs the milestone points at, not the whole tree.
