---
name: tstack-build
description: Drives the per-milestone implementation loop for a TStack-managed project — branch → plan → build → verify → merge → update roadmap status. Use when docs/ROADMAP.md exists and the user says "start milestone Mx", "build the next milestone", "implement M4", or is ready to ship a roadmap entry. Do not use for ad-hoc feature work in repos without docs/ROADMAP.md. Input is docs/ROADMAP.md + the specs that milestone references; output is shipped code, a merged branch, and updated roadmap status. Repeats until the roadmap is done.
---

# tstack-build

You are running TStack's implementation loop. Each invocation drives a single milestone from a feature branch through to merge and roadmap update. You're disciplined about plan-first work, frequent commits, and explicit verification against "Done when" criteria.

## Prereq check

Required:

```
docs/ROADMAP.md
```

If missing: stop. Tell the user this skill is for TStack-managed projects only. If they want to start one, run `tstack-discover`. If they have docs but no roadmap yet, run `tstack-roadmap`.

Also check the Status section of `docs/ROADMAP.md`. If "Up next" is empty: the project is done — tell the user.

## Repo-self guard

If `.claude-plugin/plugin.json` exists in cwd, refuse. This is a plugin repo, not a TStack consumer project.

## Approach — the milestone loop

Follow these steps strictly in order. Don't skip the plan or the verification.

### 1. Confirm the milestone

Read the Status section of `docs/ROADMAP.md`. The "Up next" entry is what you're building. Verify with the user before proceeding:

> Starting M{N} — {name}. Dependencies {Mx, My} are in Completed. Proceed?

If the user wants a different milestone, accept that and use it. But check its listed dependencies are completed first.

### 2. Update roadmap state & branch

Move the milestone's name into the active position in your todo for this session. Then create the feature branch:

```bash
git checkout main && git pull
git checkout -b milestone/{id-lowercased}-{short-desc}
```

Naming: `milestone/m4-entity-crud`, `milestone/i2-entity-list` (iOS), `milestone/m21-sync-endpoint` (cross-cutting).

### 3. Plan (enter plan mode)

Enter plan mode. Read every doc the milestone's "Read before starting" section names — specific sections only, not whole files unless the entry says so. Then plan: list every file to create or modify, in what order, and how you'll verify each piece. Surface tradeoffs the user should weigh before approving.

### 4. Get the plan approved

Review with the user. Adjust. Common pushback to expect and incorporate:
- "Use the existing pattern in `<file>` instead of inventing a new one."
- "Move shared types/schemas to a common location — they'll be needed again."
- "Smaller commits — break this into two phases."

Do not exit plan mode until the plan is approved.

### 5. Build

Implement the approved plan. **Commit frequently** — after each meaningful piece works, commit with a descriptive message. Frequent commits are clean rollback points if a later step goes wrong.

For larger milestones, work through the plan in chunks (steps 1–3, then 4–6, etc.) so verification happens incrementally.

### 6. Verify against "Done when" criteria

This is mandatory and explicit. Read the milestone's "Done when" list in ROADMAP.md and walk through every criterion. For each:
- Run the relevant test/command
- Show the user the result
- For isolation criteria (e.g., "user A can't see user B's data"), actually run the cross-account test, don't just assert it works
- For encryption round-trip: encrypt → store → read → decrypt and verify the original is returned

If any criterion fails: fix it. Don't move on.

### 7. Merge

Once every criterion passes:

```bash
git add -A
git commit -m "feat: complete {id} — {name}"
git push origin {branch}
```

For solo projects, merge directly:

```bash
git checkout main && git merge {branch} && git push origin main && git branch -d {branch}
```

For team projects, open a PR instead and follow the team's review workflow.

### 8. Update roadmap status

Edit the Status section at the bottom of `docs/ROADMAP.md`:
- Move the just-finished milestone into `Completed:`
- Update `Up next:` to the next ready milestone (its dependencies must all be in Completed)

Commit to `main`:

```bash
git add docs/ROADMAP.md
git commit -m "docs: {id} complete, up next {next-id}"
git push origin main
```

### 9. Offer to continue

Ask the user if they want to start the next milestone now. If yes, loop back to step 1. If no, end cleanly.

## Reference handoff

`references/full-guide.md` has the full guide: branching conventions for different milestone shapes, prompting patterns (starting, resuming, struggling, handling scope creep), more verification examples, and troubleshooting.

## When the roadmap evolves mid-build

If during planning or build the user realizes the milestone needs to change (scope creep, missing prerequisite, a feature was misspecified): **stop and run `tstack-specify`** instead. Don't silently expand scope — feature changes go through the doc-update flow so PRODUCT.md, API.md, and ROADMAP.md stay consistent.

## Handoff

After each milestone:

- Roadmap not done → loop back to step 1 with the new "Up next."
- Roadmap done → "All milestones shipped. If you want to add features, run `tstack-specify`."

`tstack-specify` is the natural follow-up once the initial roadmap is complete and the user wants to extend the product.
