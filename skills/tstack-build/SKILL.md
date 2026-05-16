---
name: tstack-build
description: Executes an approved milestone plan for a TStack-managed project — implements the plan with frequent commits, verifies against the milestone's "Done when" criteria, commits and merges the feature branch, and updates docs/ROADMAP.md status. Use when a milestone plan has been approved (typically immediately after tstack-plan finishes) or the user says "build it", "implement the plan", "ship this milestone", "execute the plan". Do not use to plan a milestone — that's tstack-plan. Input is an approved plan + feature branch + docs/ROADMAP.md. Output is shipped code, merged branch, and updated roadmap status. Hands back to tstack-plan for the next milestone.
---

# tstack-build

You are running TStack's per-milestone execution stage. The plan is already approved and the feature branch is checked out (`tstack-plan` did that). Your job is to implement, verify, merge, and update the roadmap. You do not re-litigate the plan unless it turns out to be wrong.

## Prereq check

Required state:

- `docs/ROADMAP.md` exists
- An approved implementation plan exists in conversation context (or in `~/.claude/plans/...` if the user wants to resume)
- The current branch is a `milestone/*` branch (not `main`)

If you're on `main` or have no approved plan: stop and tell the user to run `tstack-plan` first. Don't try to plan-as-you-go — that's the failure mode this split exists to prevent.

## Approach

### 1. Implement the plan

Work through the plan in order. **Commit frequently** — after each meaningful piece works, commit with a descriptive message tied to the plan step. Frequent commits are clean rollback points if a later step goes wrong.

For larger milestones, work in chunks (steps 1–3, then 4–6, then 7+) and verify incrementally. Don't try to land the whole milestone in one push.

If the plan turns out to be wrong mid-build (a missing piece, an unanticipated constraint, an underspecified edge case):
- For small course corrections, adjust and continue
- For anything that affects what the milestone *delivers* (scope creep, missing prerequisite, spec gap): stop and tell the user to either re-enter `tstack-plan` for this milestone, or run `tstack-specify` if PRODUCT.md is wrong

### 2. Verify against "Done when" criteria

**Mandatory format: every criterion is verified by running a real command and quoting its output back into the conversation.** "I checked it" / "looks good" / "verified ✓" without quoted output does not count and must not be accepted.

Open `docs/ROADMAP.md`, find this milestone's "Done when" list, and produce a verification report in this exact shape:

````
## Verification for M{id} — {name}

### Criterion 1: {paste the criterion verbatim}
Command: `<the command run>`
Output:
```
<paste the actual output, truncated to the relevant ~20 lines>
```
Result: PASS / FAIL — {one-line reason}

### Criterion 2: …
(repeat for every criterion)
````

Per-criterion command guidance:

- **API endpoints / happy path:** `curl` (or the project's test command) with a real request; show status code + response body.
- **Type safety / build health:** `tsc --noEmit` (or `swift build`, `cargo check`, etc.); show "0 errors" or the failure.
- **Unit / integration tests:** the project's test command (`vitest run`, `pytest`, `swift test`, …); show the pass/fail summary line.
- **Isolation criteria** ("user A cannot see user B's data"): run the cross-account test with two distinct auth tokens; show both responses side by side.
- **Encryption round-trip:** encrypt → store → read → decrypt; show input value, stored ciphertext (truncated), and output value.
- **Background jobs:** trigger the job; show both the success log and a forced-failure log (or its absence with a "no error scenario possible" note).
- **Manual UI flows** (last resort, only when no automated path exists): describe the exact click sequence, paste a screenshot path or console output captured during it, and call out that this is manual.

If a criterion can't be verified by any of the above, the criterion is not testable. Stop and tell the user the roadmap entry has a soft criterion that needs rewriting before the milestone can be shipped.

If any criterion is FAIL: fix it. Don't move on. Don't mark the milestone done.

### 3. Commit final state and merge

Once every criterion passes:

```bash
git add -A
git commit -m "feat: complete {id} — {name}"
git push origin {branch}
```

**Solo projects** — merge directly:

```bash
git checkout main && git merge {branch} && git push origin main && git branch -d {branch}
```

**Team projects** — open a PR and follow the team's review workflow instead. Use `gh pr create` if available. The rest of the loop is the same after merge.

### 4. Update roadmap status

Edit the **Status section at the bottom** of `docs/ROADMAP.md` — this is the only place status is tracked. Do not put status indicators on individual milestone entries.

- Move the just-finished milestone into `Completed:` (append, preserve order)
- Update `Up next:` to the next milestone whose dependencies are all in Completed

Commit to `main`:

```bash
git add docs/ROADMAP.md
git commit -m "docs: {id} complete, up next {next-id}"
git push origin main
```

### 5. Offer to continue

Ask the user:

> {id} shipped and merged. Up next: {next-id} — {next-name}. Want to plan it now?

- If yes → hand back to `tstack-plan` for the next milestone
- If no → end cleanly. They can return later by invoking `tstack-plan` or saying "plan the next milestone."

## Reference handoff

`references/full-guide.md` is the full implementation guide. The execution-relevant sections are "Build", "Verify Against Done When Criteria", "Merge", "Update Roadmap Status", and the prompting patterns for "Resuming After a Break" and "Struggling Mid-Milestone." Read those when the situation calls for them.

For a realistic verification-report example showing the quoted-command-output discipline across six different criteria types (unit tests, curl, Playwright, manual UI, axe a11y), read `references/example-output.md`.

## When the plan was wrong

Don't silently expand scope. If during build you find:

- The milestone is bigger than the plan accounted for → propose splitting (`Mxa` / `Mxb`) and re-enter `tstack-plan` for the second half
- A feature is misspecified in PRODUCT.md or API.md → stop, run `tstack-specify` to update the docs, regenerate the relevant roadmap entries, then re-enter `tstack-plan`
- The plan referenced a doc section that doesn't exist → flag the gap, get the user to fill it before continuing

Silent overreach is the failure mode. Surface the issue, don't paper over it.

## Hand-back

After step 5, you either loop into `tstack-plan` for the next milestone (continuous flow) or end the session cleanly. Either way, the project state on disk reflects exactly what's been shipped: `Completed:` and `Up next:` in ROADMAP.md are the source of truth for what to do next.
