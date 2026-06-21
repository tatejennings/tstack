# TStack — Deep Skill Review & Recommendations

> Advisory review of the seven TStack skills: their goals, where they're strong, where the seams leak, and two candidate new skills. Nothing in the plugin is changed by this document — it's a working artifact to drive future edits and BACKLOG entries.
>
> Reviewed: all seven `SKILL.md` files, their `references/`, `README.md`, `BACKLOG.md`, and both manifests. Findings are cross-referenced against `BACKLOG.md` so already-tracked work is labelled and not double-counted.

---

## 1. Overall assessment

The chain — `discover → product → architect → roadmap → plan → build`, plus `specify` as the iteration loop — is genuinely well-designed. Each skill does one thing, prereq-checks its input file, and hands off cleanly by name. Three disciplines stand out as best-in-class and should be treated as the framework's crown jewels:

- **M0 mandatory infrastructure** in `tstack-roadmap` (refuses to save a roadmap without CI, branch protection, secrets, deploy skeleton, observability, lint/format).
- **Eval-based AI acceptance criteria** in `tstack-product` (eval set + measurable quality bar + deterministic fallback — not Given/When/Then hand-waving).
- **Quoted-command-output verification** in `tstack-build` (every "Done when" criterion proven with a real command and its pasted output; "Verified ✓" alone is rejected).

The frontmatter `description` fields are strong, specific triggers, and the `product` vs `specify` mutual exclusion is correctly disambiguated with explicit negative phrasing.

**The weaknesses are not in the goal of any individual skill — every skill's purpose is sound. They live in the *seams between skills* and in *reference-material consistency*.** That's where this review focuses.

---

## 2. Per-skill goal check

Each skill's core goal is sound. Below is the one highest-leverage issue for each.

| Skill | Goal verdict | Top issue |
|---|---|---|
| **discover** | Sound | WebSearch-unavailable fallback is a UX cliff ("paste competitor research" / switch to claude.ai); no explicit "brief-is-ready" quality gate before commit; no "when NOT to run" guidance (user who already has a finished spec should be redirected to `product`). |
| **product** | Sound | Its `full-guide.md` is the **thinnest (123 lines) yet produces the most load-bearing doc**; the AI eval-criteria framework isn't surfaced in the description, so an AI-heavy product can slip through with soft criteria; "this PRODUCT.md is complete" exit criteria are implicit. |
| **architect** | Sound (most rigorous) | Its `full-guide.md` **self-admits staleness** (SKILL.md line 155: "The reference guide predates the foundational-ADRs and TESTING.md additions"); ADR count grows unbounded with no consolidation guidance; the "2026 default stack" is year-stamped but the choice isn't dated in the ADR; defaults are proposed before asking language/team, so a Python team gets a Next.js default. |
| **roadmap** | Sound (M0 is excellent) | **"Done when" testability is validated too late** — roadmap writes the criteria, `build` discovers the untestable ones milestones later; and an **internal description↔prereq mismatch** on CONVENTIONS.md (see §3.4). |
| **plan** | Sound (plan-first is enforced) | **The "approved plan" artifact is never formally defined or persisted** — it lives in conversation context with a vague `~/.claude/plans/...` fallback; no granularity heuristics; no own guide (points at build's). |
| **build** | Sound (verification is the strongest part of the suite) | Inherits the undefined plan-artifact problem (what *is* the "approved plan" input, and how does it survive a session restart?); no partial-success / criterion-waiver policy (all-or-nothing); team PR workflow is referenced but not integrated. |
| **specify** | Sound (two-list proposal is excellent UX) | **It appends to ROADMAP.md while `roadmap` regenerates ROADMAP.md** → drift, with no documented contract; the "considered but not changing" list gets verbose on spec-heavy projects; mid-roadmap milestone-insertion logic is underspecified. |

---

## 3. Cross-cutting themes (the high-leverage findings)

These are the issues worth fixing first, because each one spans multiple skills.

### 3.1 Reference material is inconsistent and partly stale
`architect/references/full-guide.md` is explicitly self-flagged as predating the foundational-ADR and TESTING.md additions. `product`'s guide is the thinnest (123 lines) precisely where depth matters most. `plan` and `specify` have no guide at all (by design — they point at siblings — but with no bridge text telling the reader where to look).

→ **Establish a single source-of-truth policy:** `SKILL.md` is authoritative; `full-guide.md` is reference-only and must never contradict its SKILL.md. Re-base the stale architect guide against the current SKILL.md.
*(BACKLOG already tracks "refresh full-guide" and "extract inline templates." This finding reframes the urgency: a guide that openly contradicts its skill is a correctness risk, not just tidiness.)*

### 3.2 The plan artifact is a continuity gap *(net-new)*
`plan → build` hands off an "approved plan" with no defined format and no durable location. If the session ends between planning and building, the artifact is gone — `build`'s only fallback is a vague mention of `~/.claude/plans/...`, and nothing in `plan` is responsible for putting it there.

→ **Define a persisted location and a minimal schema** (e.g. `.tstack/plans/{milestone-id}.md`, or a `docs/`-local file): files-to-touch in dependency order, patterns reused vs. introduced, per-criterion verification approach, out-of-scope. Make `plan` responsible for writing it and `build` for reading it. This directly enables the BACKLOG `.tstack/state.json` cross-session-coordination idea.

### 3.3 ROADMAP.md has two writers with no contract *(net-new)*
`roadmap` *regenerates* ROADMAP.md from the whole doc tree; `specify` *appends* milestones to the same file and edits the Status section. Nothing documents how they coexist, so a project that alternates between them can silently drift out of sync with the docs the roadmap was synthesized from.

→ **Write the contract down:** `specify` is surgical/append-only; a full re-sequence requires re-running `roadmap`; add a "docs last synced: <commit/date>" marker to ROADMAP.md so drift is visible at a glance.

### 3.4 CONVENTIONS.md — description contradicts the prereq block *(net-new, verified)*
`roadmap`'s **frontmatter description** lists inputs as "PRODUCT.md + ARCHITECTURE.md + DECISIONS.md + TESTING.md (API.md and breakout specs optional)" — CONVENTIONS.md is absent. But the skill's own **Prereq check** lists `docs/CONVENTIONS.md` as a *required* input (line 19), and step 1 of the approach reads it ("From CONVENTIONS.md: which milestones introduce a new pattern domain"). CONVENTIONS.md *is* produced at every right-sizing tier, so the requirement is correct — the **description is simply out of sync with the body**.

→ **Add CONVENTIONS.md to the frontmatter description's input list.** Trivial fix, but the description is the load-bearing trigger contract, so it should match what the skill actually requires.

### 3.5 AGENTS.md ownership is fuzzy
`architect` creates AGENTS.md; `roadmap` appends a "Current Focus" pointer; `build` and `specify` touch status. No single skill owns the file or its section layout.

→ **Define one owner (architect) and a section contract** — who writes which section, and that downstream skills only update a designated "Current Focus / Status" block rather than restructuring. BACKLOG's "update AGENTS.md format" item should absorb this.

### 3.6 "Done when" testability is verified too late
`roadmap` writes "Done when" criteria; `build` is where an untestable/soft criterion is finally caught — potentially many milestones after it was written.

→ **Move the check upstream:** `roadmap`'s pre-save cross-check should assert that every criterion maps to a concrete runnable command or test, not merely that referenced docs exist.

### 3.7 Tech-stack defaults aren't dated or skill-matched
The "2026 default stack" is explicitly year-stamped but the resulting ADR doesn't record *when* the choice was made or *when to revisit*. Defaults are also proposed before the skill knows the team's language.

→ **Record "chosen as of <date>" + a revisit trigger in the tech-stack ADR**, and **ask language/team preference before proposing the default stack** so the opinionated default is for the right ecosystem.

### 3.8 Session-boundary guidance is vague and inconsistently strong
Across skills the advice ranges from "consider a fresh session" to "fresh session recommended" to "no fresh session needed," all gated on undefined phrases like "non-trivial scope."

→ **Replace with concrete thresholds** (feature/entity/endpoint counts) or rough context-budget heuristics, and standardize the wording.

### 3.9 AI-feature detection isn't propagated upstream
`architect` treats AI as a first-class concern (ADR-5 + `ai-strategy.md`); `product` does not surface it in its description, so an AI feature can reach PRODUCT.md with a soft criterion that only fails later in `build`.

→ **Have `product` detect AI/LLM features from the brief early** and force eval-based criteria at that point, closing the loop before architecture.

---

## 4. Prioritized recommendations

**P1 — correctness / seams**
- Define and persist the plan artifact (§3.2).
- Document the `roadmap` ↔ `specify` ROADMAP.md contract (§3.3).
- Fix the CONVENTIONS.md description↔prereq mismatch (§3.4).
- Resolve AGENTS.md ownership (§3.5).

**P2 — quality of output**
- Validate "Done when" testability at roadmap time (§3.6).
- Surface AI criteria in `product` (§3.9).
- Thicken `product`'s guide and re-base the stale `architect` guide (§3.1).

**P3 — polish**
- Date tech-stack defaults + ask language first (§3.7).
- Concrete session-boundary thresholds (§3.8).
- `discover` "when-not-to-run" + brief-ready gate.
- `build` partial-success / criterion-waiver policy.

---

## 5. Candidate new skills

You already have three in BACKLOG (`tstack-audit`, `tstack-techdebt`, `tstack-retire`); those are out of scope here. Below are the two requested directions.

### 5.1 `tstack-design` — design/UX spec **(optional, off-chain, callable at any point)**

**Why it exists.** Frontend work currently has no upstream source of truth. `build` *verifies* accessibility, but nothing upstream *defines* the UI it's checking. This skill fills that gap.

**Deliberately not a chain step.** Unlike the other skills, `tstack-design` is **optional and outside the `discover → … → build` flow**. No skill hands off to it and it blocks nothing. It can be invoked at *any* point in a project's life:
- *before* `architect` — to inform the frontend stack choice,
- *after* `roadmap` — to flesh out a specific UI milestone,
- *mid-`build`* — when a particular screen needs a design pass.

Its only prerequisite is "there's a product to design for": it reads `PRODUCT.md` if present, otherwise works from the user's description.

**Standard output — `docs/2 - Specs/design.md`** (human-readable): design tokens, component inventory, key screen layouts, interaction states (incl. empty/error/loading), responsive breakpoints, and accessibility patterns tied to the architect's accessibility ADR (ADR-3).

**Headline output — Claude Design hand-off material.** Beyond the human spec, the skill produces **ready-to-paste prompts and structured context that feed directly into Claude Design**:
- **Per-screen generation prompts** — purpose, layout intent, key components, content & states, tone/visual direction.
- **Design-token set** — color, type, spacing, radius, elevation.
- **Component inventory** — names, variants, states.
- **Brand/style constraints** — any fixed visual rules.

These are formatted so the user can drop them straight into Claude Design to generate or iterate the actual UI. The skill **explicitly labels which sections are "for Claude Design" vs. "for the human reader,"** so the hand-off material is unambiguous.

**Triggers:** "design the UI", "create a design system", "give me design prompts", "spec a screen", or any moment a consumer-facing surface needs design work.

**Handoff:** none fixed (it's off-chain). It *may* suggest re-running `roadmap`/`plan` if it surfaces new UI milestones, but never requires it. Natural integration point with the existing `frontend-design` and Figma skills.

### 5.2 `tstack-status` — read-only project inspector / onboarding

**Why it exists.** Directly addresses the continuity and drift gaps in §3.2–§3.5. It's the suite's missing read-only "inspector," safe to run anytime.

**Status mode.** Reads ROADMAP.md status + git state + the `docs/` tree and reports: what's shipped, what's up next, blocked milestones, missing mandatory docs, and **doc drift** (e.g. PRODUCT.md modified after ROADMAP.md was generated — exactly the §3.3 failure mode made visible).

**Onboard mode.** Generates contributor onboarding — how to set up, how to run, where the docs live, and the current focus — for a developer joining a TStack-managed project.

**Triggers:** "where are we", "project status", "what's left", "onboard a new dev", "is anything out of sync".

**Output:** a concise chat report and/or `docs/STATUS.md`. Pairs naturally with the BACKLOG `.tstack/state.json` idea — `tstack-status` would be its primary reader.

---

## 6. Appendix

### 6.1 Reference-guide line counts (verified)
The imbalance is concrete: the most load-bearing doc (`product`) has the leanest guide, and two skills have none.

| Skill | `full-guide.md` lines |
|---|---|
| build | 467 |
| architect | 369 *(self-flagged stale)* |
| discover | 269 |
| roadmap | 247 |
| product | **123** *(thinnest; most load-bearing output)* |
| plan | — *(none; points to build's guide)* |
| specify | — *(none; points to product/roadmap/architect guides)* |

### 6.2 Already-tracked vs. net-new (so nothing is double-counted)

| Finding | Status |
|---|---|
| Refresh / re-base stale `full-guide.md` (§3.1) | **In BACKLOG** ("refresh full-guide", "keep vs rewrite") — reframed as correctness risk |
| Extract inline templates to `references/*-template.md` | **In BACKLOG** |
| AGENTS.md format update (§3.5) | **In BACKLOG** ("update AGENTS.md format") — extend to cover *ownership* |
| Extra foundational ADRs (perf budgets, cost ceilings, i18n) | **In BACKLOG** |
| `.tstack/state.json` cross-session coordination (§3.2) | **In BACKLOG** — `tstack-status` would be its reader |
| `tstack-audit` / `tstack-techdebt` / `tstack-retire` | **In BACKLOG** (out of scope here) |
| Plan-artifact definition & persistence (§3.2) | **Net-new** |
| `roadmap` ↔ `specify` ROADMAP.md contract (§3.3) | **Net-new** |
| CONVENTIONS.md description↔prereq mismatch (§3.4) | **Net-new** |
| "Done when" testability checked at roadmap time (§3.6) | **Net-new** |
| Date tech-stack defaults + ask-language-first (§3.7) | **Net-new** |
| Concrete session-boundary thresholds (§3.8) | **Net-new** |
| AI-feature detection in `product` (§3.9) | **Net-new** |
| `tstack-design` (off-chain, Claude Design output) | **Net-new** |
| `tstack-status` (inspector / onboarding) | **Net-new** |
