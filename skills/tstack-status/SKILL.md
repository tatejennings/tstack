---
name: tstack-status
description: Read-only inspector for a TStack-managed project. Reads the docs/ tree, ROADMAP.md status, and git state and reports project status in chat — what's shipped, what's up next, blocked or in-progress milestones, missing mandatory docs, and doc drift (e.g. PRODUCT.md edited after ROADMAP.md was last synced). On request it also generates contributor onboarding. Writes nothing and changes nothing. Use when the user says "where are we", "project status", "what's left", "what's the state of this project", "onboard a new dev", "is anything out of sync", or wants a project state report. Input is the docs/ tree + git; output is a chat report. Off-chain utility — hands off to nothing.
---

# tstack-status

You are running TStack's status inspector. You read the project's docs and git state and report where things stand. This is the suite's read-only lens — it makes the project's state, and any drift between docs, visible at a glance.

**Hard rule: this skill is strictly read-only.** Never write, edit, create, or commit anything. No file changes, no git mutations. If the report reveals something to fix, *recommend* the skill that fixes it (`tstack-specify`, `tstack-roadmap`, `tstack-plan`) — don't fix it here. This skill is off-chain and safe to run anytime.

## Prereq check (soft)

Works on any project; degrades gracefully:
- If `docs/ROADMAP.md` exists → full status report.
- If docs exist but no ROADMAP.md → report how far through the chain the project is and what to run next.
- If there's no `docs/` at all → say it isn't a TStack project yet and point to `tstack-discover`.

## Modes

Pick based on what the user asked; default to **Status**.

### Status mode (default)

Read (read-only) and report:

1. **Inputs to gather**
   - `docs/ROADMAP.md` — the **Status** section (`Completed:`, `Up next:`, `Docs last synced:`) and each milestone's dependencies + "Done when".
   - Git — current branch, recent commits, any `milestone/*` branch in progress. (`git status`, `git branch`, `git log --oneline -15`.)
   - The `docs/` tree — which docs exist.
   - File modification times via git (`git log -1 --format=%cs -- <file>`) to detect drift.

2. **Report (in chat) these sections:**
   - **Shipped** — milestones in `Completed:`.
   - **Up next** — the `Up next:` milestone and whether its dependencies are all in `Completed` (if not → **Blocked**, name the missing deps).
   - **In progress** — if on a `milestone/*` branch, which milestone and whether its plan exists at `docs/plans/{id}.md`. Also surface any `docs/plans/*.md` for milestones not yet built (planned-ahead work waiting for a builder).
   - **Missing mandatory docs** — flag any absent from: PRODUCT.md, ARCHITECTURE.md, CONVENTIONS.md, TESTING.md, DECISIONS.md, ROADMAP.md. (API.md and breakout specs are optional — don't flag them.)
   - **Doc drift** — the key check (below).
   - **Suggested next action** — the single most useful next skill to run, given the above.

3. **Doc drift check.** Compare when the upstream docs last changed against ROADMAP.md's `Docs last synced:` marker (and ROADMAP.md's own last-modified date as a fallback). If `PRODUCT.md`, `ARCHITECTURE.md`, `API.md`, or a breakout spec was modified *after* the roadmap was last synced — and the change wasn't a surgical `tstack-specify` edit — the roadmap may no longer reflect the docs. Flag it explicitly:

   > ⚠ Drift: `PRODUCT.md` was edited 2026-05-20, after ROADMAP.md's last sync (2026-05-12). The roadmap may be stale — consider re-running `tstack-roadmap`.

   If the marker is absent (older project), say so and fall back to comparing git modification dates.

### Onboard mode (on request)

When the user wants to bring someone new in ("onboard a new dev", "how does someone get started here"), produce a chat onboarding briefing from the docs:
- What the product is (1–2 sentences from PRODUCT.md).
- The stack and key commands (from AGENTS.md / ARCHITECTURE.md / CONVENTIONS.md).
- Where the docs live and when to read each (mirror AGENTS.md's doc table).
- Current focus (from ROADMAP.md's Status section).

Output it in chat. If the user wants it saved as a file, that's a separate ask — tell them this skill doesn't write, and offer to do it outside the skill or via `tstack-architect` (which owns AGENTS.md).

## Output

Chat only. Keep it scannable — short headed sections, not prose. Do not write a STATUS.md or any file.

For a realistic example of a Status-mode report (including a doc-drift flag and an early-stage project with no roadmap), read `references/example-output.md`.

## Handoff

Off-chain utility — no handoff. Close by pointing at the single most useful next action you identified (e.g., "Up next is M4 and unblocked — run `tstack-plan`," or "Drift detected — re-run `tstack-roadmap` first").
