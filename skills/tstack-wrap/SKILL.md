---
name: tstack-wrap
description: Closes out a work session by sweeping for things that should be documented but aren't — decisions, tradeoffs, gotchas, or operational events (stale branches, deferred criteria, manual steps, new dependencies) from the session's work — then writes each genuine gap to its right home (a dated ADR in DECISIONS.md, a pattern in CONVENTIONS.md, a code comment, or ROADMAP.md's Status section) and reports what went where. Does not commit, and never adds or renumbers roadmap milestones. Use at the end of a session or when the user says "before we wrap", "did we miss documenting anything", "sweep for doc gaps", "capture loose ends", or "wrap up". Off-chain utility that *writes* targeted doc updates — unlike tstack-status, which only reports drift read-only and changes nothing. Input is the session's work + git log + the docs/ tree; output is doc edits + a chat report. Hands off to nothing.
---

# tstack-wrap

You are running TStack's session wrap-up. Real work generates knowledge that never makes it to disk — a tradeoff settled in conversation, a gotcha discovered mid-build, a branch left dangling, a criterion quietly deferred. That knowledge is the project's most perishable context: it's clear now and gone next session. Your job is to catch it before the session closes and write it where it belongs, so the docs stay a faithful, agent-readable record of reality.

**This skill is optional and off-chain.** It's *not* a step in `discover → product → architect → roadmap → plan → build`. Nothing hands off to it and it blocks nothing. Run it whenever a session has produced real work — most naturally right before you close a session, or any time the user wants to make sure nothing slipped through undocumented.

**It writes, but it stays in its lane.** Unlike `tstack-status` (strictly read-only — it only *reports* drift), this skill *fixes* gaps by writing targeted doc updates. But it only writes small, in-lane records (an ADR, a convention, a comment, a status note). It never restructures docs, never adds or renumbers roadmap milestones, and never commits — anything bigger gets routed to the skill that owns it.

## Prereq check (soft)

Works on any project; degrades gracefully:
- If `docs/` exists → full sweep against the doc set.
- If there's no `docs/` at all → say it isn't a TStack project, do a lighter pass (git log + code comments only), and point to `tstack-discover` if they want the full chain.

Nothing here is mandatory. The sweep is only as useful as the session that preceded it — if no real work happened, say so and stop rather than inventing findings.

## Approach

### 1. Gather what happened this session

Build a picture of the session's work from durable signals, not just memory:

- **The conversation** — decisions made, tradeoffs weighed, problems hit and how they were solved, anything the user said "we should remember / note / come back to."
- **Git** — `git log` since the session started (or since ROADMAP.md's `Docs last synced:` marker, whichever is tighter); `git status` for uncommitted changes and stale/unmerged `milestone/*` branches; `git diff` on `docs/` to see what's already been written.

### 2. Identify candidate facts

From that picture, list everything that *might* deserve a durable home:

- **Decisions & tradeoffs** — "we chose X over Y because Z", a default that was overridden, a library picked or rejected.
- **Gotchas** — a non-obvious constraint, a footgun, a workaround that isn't self-explanatory in the code.
- **Operational events** — a stale or unmerged branch, a criterion deferred (was it recorded as DEFERRED?), a manual step taken outside the normal flow, a new dependency added, an env/config quirk.
- **Convention shifts** — a new pattern introduced that future code should follow.

### 3. Filter to genuine gaps

For each candidate, check whether it's **already captured** — in the docs, in a commit message, or in a code comment. Drop anything that's already recorded. A good sweep surfaces only true gaps; don't re-report what's already on disk.

### 4. Route each gap to its right home

Map every surviving gap to exactly one destination, and **respect the ownership contracts** (this is the load-bearing constraint — getting it wrong corrupts the doc set other skills depend on):

| Gap type | Home | Format / constraint |
|---|---|---|
| A genuine decision or tradeoff | **`docs/DECISIONS.md`** | Append a new **dated ADR**, matching the existing ADR format in that file. |
| A new code-level pattern future code should follow | **`docs/CONVENTIONS.md`** | Add to the relevant section in that file's style. |
| A gotcha local to a specific spot in the code | **A code comment** | At the exact line/function; keep it short and matter-of-fact. |
| A status note, stale branch, or already-handled deferral | **`docs/ROADMAP.md` — Status section only** | Note it in Status. **Never add or renumber milestones.** |
| Project-wide focus shift | **`AGENTS.md` — `## Current Focus` block only** | `tstack-architect` owns the rest of the file; touch nothing else. |

**Route out-of-lane work — don't do it here.** If a gap is really *new scope* — a feature, a behavior change, a new milestone, a doc restructure — do **not** write it. Surface it and recommend the right skill:
- New feature / changed product behavior → **`tstack-specify`**.
- The roadmap needs re-sequencing (not just a status note) → **`tstack-roadmap`**.

### 5. Confirm borderline items, then write

Present the routed gaps as a short list — each with its proposed destination — before writing. Clear-cut records (an obvious ADR, an obvious comment) you can write directly. For anything **borderline** — is this a real decision or just a passing comment? does this belong in DECISIONS or CONVENTIONS? — ask the user first. (This mirrors `tstack-specify`'s per-item discipline: surface the judgment call rather than guessing.) Write only the confirmed ones.

### 6. Report what went where — and stop

Close with a scannable report: each gap, where it was written (file + section), and any items you routed out to another skill instead. Then **stop**.

**Do not commit.** Leave the working tree staged-or-unstaged exactly as the user prefers to review it — committing is their call (the same convention `tstack-plan`/`tstack-build` follow for their artifacts). If they want it committed, that's a separate, explicit ask.

## Hard rules

- **Never commit.** Write files; let the user review and commit.
- **Never add or renumber roadmap milestones.** Status-section notes only; new milestones go through `tstack-specify` / `tstack-roadmap`.
- **Never restructure `AGENTS.md`.** Only the `## Current Focus` block, if anything.
- **Don't invent findings.** Only write gaps grounded in the session's actual work and confirmed against disk.
- **Route, don't overreach.** New scope is a recommendation, not an edit.

## Reference handoff

For a realistic end-to-end run — candidates gathered, already-documented ones filtered out, one ADR written, one code comment, one borderline item confirmed with the user, one gap routed out to `tstack-specify`, and the final "what went where" report — read `references/example-output.md`.

## Handoff

Off-chain utility — no required next step. End with the report and, if anything was routed out, the single most useful follow-up:

> Swept the session. Wrote {n} gaps: {1-line each — file + what}. Routed {n} out (e.g., "the CSV-export idea is new scope — run `tstack-specify` to spec it"). Nothing committed — review and commit when ready.
