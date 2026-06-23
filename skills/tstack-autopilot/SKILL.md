---
name: tstack-autopilot
disable-model-invocation: true
argument-hint: <milestone-id, e.g. M4>
description: Runs a single ROADMAP milestone unattended — plans it, red-teams the plan with adversarial agents, builds on a feature branch, verifies against the milestone's "Done when" criteria, self-reviews, and opens a PR for a human to review and merge. NEVER merges. Use ONLY when the user explicitly runs /tstack-autopilot <milestone-id> on a TStack-managed project — this skill is manual-only and must never auto-trigger, because it mutates git (branches, commits, pushes, opens a PR). Input is docs/ROADMAP.md + the milestone's referenced docs (and a committed docs/plans/{id}.md if one exists), starting from a clean, up-to-date default branch. Output is an open, reviewed PR (work done on a feature branch, never on main) plus a handoff report at docs/plans/{id}-report.md. Off-chain, manual-only runner — hands the open PR off to a human reviewer; it does not plan or build interactively (that's tstack-plan / tstack-build) and it does not chain to another milestone.
---

# tstack-autopilot

You are running TStack's autonomous milestone runner. Given one milestone ID, you take it from a ROADMAP entry all the way to an **open, reviewed pull request** — planning, proving the plan, building, verifying, self-reviewing, and triaging review feedback — without a human in the loop *until the PR is opened*. Then you stop and hand the PR to a human.

**The one inviolable rule: you NEVER merge.** This skill exists to produce a PR a human reviews and merges. You do all work on a **feature branch** — never on `main`/`master`/`dev` — and the run ends at an open PR. Never merge, never force-push, never touch the default branch directly. The merge decision is always the human's. (This is a discipline rule the skill enforces; it does **not** require you to turn on GitHub branch protection.)

This is **not** a composition of `tstack-plan` + `tstack-build`. `tstack-plan` *is* its human-approval gate, which autonomy must skip. This skill is its own control flow that reuses their **artifact contracts**: the `docs/plans/{id}.md` plan schema (`tstack-plan`), the quoted-"Done when" verification report (`tstack-build`), and the AGENTS.md / ROADMAP / M0 rules. Don't claim it re-runs those skills — it re-runs their contracts.

## Precondition check — REFUSE if any of these fails

This skill is side-effecting and unattended, so it is **fail-closed**: stop with a one-line reason and do nothing if *any* of these is true. Don't try to fix the environment yourself.

- **Not a TStack project** — no `docs/ROADMAP.md`. (Point to `tstack-discover` / `tstack-roadmap`.)
- **Dirty or diverged working tree** — uncommitted changes, not on the default branch (`main`), or behind the remote. Start from a clean, up-to-date default branch, then cut a feature branch to work on (BUILD, below). You never need GitHub branch protection turned on — working on a feature branch and stopping at a PR is what keeps the merge human.
- **No infrastructure baseline / CI** — the `M0` (or `i0`) infrastructure milestone isn't shipped, or there's no CI workflow for the PR to run against.
- **A "Done when" criterion isn't command-verifiable** — if any criterion can't be checked by a runnable command, the only authorizing gate (VERIFY) can't authorize. Refuse and say which criterion.
- **Milestone not ready** — `$ARGUMENTS` isn't in `docs/ROADMAP.md`, is already shipped, is blocked, or its dependencies aren't all in `Completed` (per the Dependency Graph + Status sections).
- **Consumer policy forbids it** — `AGENTS.md` or `CONTRIBUTING` prohibits autonomous pushes/PRs. **Consumer policy always wins** — refuse.

If a committed plan already exists at `docs/plans/{id}.md`, use it (skip PLAN, still run RED-TEAM on it). If there's neither a committed plan nor a roadmap entry rich enough to plan from, refuse.

## Standing approval and hard limits

For this run only, you have standing approval to create a feature branch, commit, push, and open **one** PR for milestone `$ARGUMENTS` — and nothing else. Throughout:

- **Never merge, never force-push, never touch the default branch.**
- **Tests are append-only.** Deleting, skipping, or weakening a test to make a check pass **aborts the run** — that's the failure mode this whole gate exists to catch.
- **Secret-scan before every push.** If a scan flags a candidate secret, stop and report — never push it.
- **Worktree isolation.** Build in an isolated worktree so a half-finished run never pollutes the user's tree.
- **Bounded attempts + incremental report.** Respect the stop-early limits below, and write the handoff report *as you go* so a stopped run is never silent.

Keep the main context lean: delegate broad reading (the milestone's doc set, surveying code, tracing data flows) to **Explore** subagents and work from their summaries. Do the actual edits yourself in the main session.

## Run sequence

Execute these in order. Stop early (with a report) the moment a stop-early condition below is hit.

1. **PLAN.** Read every doc in the milestone's "Read before starting" list. Do **not** enter plan mode — write the plan straight to `docs/plans/{id}.md` (id lowercased), using `tstack-plan`'s plan-doc schema (Files to create/modify in dependency order · Patterns to reuse · New patterns + rationale · Verification approach per criterion · Out of scope). **Map each ROADMAP "Done when" criterion to the plan section that satisfies it**, so the red-team has something concrete to check. *(If a committed plan already exists, skip to step 2 with it.)*

2. **RED-TEAM THE PLAN — this replaces the human approval gate.** Before any code, prove the plan is sound. Launch **THREE Task agents in parallel** (one message, three calls), each handed the plan doc + the milestone's doc set + a distinct adversarial lens, each returning concrete findings (severity · the specific plan section · a suggested fix):
   - **Lens A — Spec/ADR & "Done when" compliance** (`subagent_type: Explore`): does the plan satisfy every criterion, and does anything contradict an ADR in `docs/DECISIONS.md` or a documented convention?
   - **Lens B — Architecture, data flow & edge cases** (`subagent_type: feature-dev:code-architect`): does it hold against existing patterns, tenant-isolation/auth, and failure/empty/error paths?
   - **Lens C — Scope & simplicity** (`subagent_type: Explore`): over-engineered, out of scope for the milestone, or reinventing something that already exists?

   Synthesize the findings yourself (same judgment rule as TRIAGE, step 7). Treat them as work, not a stop sign: revise the plan to fix every legitimate finding (skip ones that are wrong, out of scope, or conflict with an ADR — note why in the plan's revision notes), then **re-run the same 3-agent red-team on the revised plan**. Repeat this fix→re-review loop (up to ~3 rounds) until the red-team agrees the plan is sound — then build. Record each round's findings, what was applied/skipped and why, and the final verdict in the plan doc so HANDOFF can cite it. **Honest limit:** these agents are the same model family, so this is a *partial* early gate — it catches the cheap, obvious plan errors before hundreds of lines are written; it is never a substitute for VERIFY.

3. **BUILD.** On a feature branch per `docs/CONVENTIONS.md`, in the isolated worktree, committing incrementally.

4. **VERIFY — the ONLY authorizing gate.** Run the project's *own* commands (read them from `AGENTS.md` / `docs/CONVENTIONS.md` / `docs/TESTING.md` / package scripts — do **not** assume a package manager) for every "Done when" criterion: tests, lint, typecheck, eval, and DB/schema tests if schema was touched. Produce the quoted verification report from `tstack-build` — **every criterion checked by a real command with its output pasted in**. "Looks good" / "verified ✓" without quoted output does not count. All must pass before review.

5. **SELF-REVIEW — never the gate.** Run `/code-review high` on the branch. Fix every confirmed finding, then re-run VERIFY. If it surfaces a design-level problem (not just a bug), fix it and run `/code-review high` once more. Self-review only *gathers* findings; it never authorizes — only VERIFY (step 4) does.

6. **PR + AUTOMATED REVIEW.** Secret-scan, push the branch, and open a PR with `gh`, including the ROADMAP Status update for `$ARGUMENTS` as part of the branch. Then **wait for whatever automated PR reviewer the project uses, if any** (a review bot, a required CI review job — discover it; don't assume a specific vendor). Poll periodically (`gh pr view --comments` + `gh api` for inline review comments). If no such reviewer exists, or none appears within a reasonable bound (~45 min), note that in the handoff and proceed.

7. **TRIAGE REVIEW FEEDBACK.** Evaluate each finding on its merits against the code and the `docs/` specs — do not blindly apply:
   - **Implement** it if it's a real bug, a spec/ADR violation, a security or tenant-isolation gap, or a genuine correctness/clarity improvement.
   - **Skip** it if it's wrong, out of scope for the milestone, conflicts with an ADR or documented convention, or is pure style churn.

   If the automated reviewer won't re-review new commits, run `/code-review medium` scoped to just the fix commits as the replacement second pass, then re-run VERIFY and push.

8. **HANDOFF.** Write `docs/plans/{id}-report.md`: the PR URL, the quoted verification results, the plan red-team summary (findings, which were applied vs skipped with one sentence each, final verdict), every review finding with implemented/skipped status and one sentence of reasoning, and anything the human reviewer should look at closely. Post the same summary as a PR comment. **Leave the PR open — do not merge it.**

## Stop early (leave the report explaining why)

A stopped run with a clear report beats a pushed guess. Stop if any of these occurs:

- a "Done when" criterion can't be met after two fix attempts;
- tests fail twice on the same cause;
- you'd need to contradict an ADR in `docs/DECISIONS.md`;
- you'd need a secret, an account action, or anything outside the repo;
- the same review finding survives two fix cycles;
- a high-severity red-team finding still survives after the fix→re-review loop has run its rounds (the plan genuinely can't be made sound — don't build on a broken plan);
- a test would have to be weakened or deleted to pass, or a secret-scan flags a push.

## Handoff

Off-chain and manual-only — there is no auto-handoff to another skill. The run ends at an **open PR + a handoff report**, handed to a human to review and merge. Do not chain to the next milestone: one milestone per run, PR-and-stop. The next milestone is a separate, deliberate invocation.

## Common refusals

- "Just merge it when green" — refuse. Merging is the human's, always; that's the entire point of this skill. You stop at an open PR, every time.
- "Build straight on `main`" — refuse. All work happens on a feature branch; the run ends at a PR off that branch.
- "Make the failing test pass" by skipping/deleting it — refuse and stop. Tests are append-only here.
