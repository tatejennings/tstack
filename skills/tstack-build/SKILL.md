---
name: tstack-build
description: Executes an approved milestone plan for a TStack-managed project — implements the plan with frequent commits, verifies against the milestone's "Done when" criteria, commits and merges the feature branch, and updates docs/ROADMAP.md status. Use when an approved plan exists at docs/plans/{id}.md (typically opened in a fresh session) or the user says "build it", "implement the plan", "ship this milestone", "execute the plan". Do not use to plan a milestone — that's tstack-plan-milestone. Input is an approved plan + feature branch + docs/ROADMAP.md. Output is shipped code, merged branch, and updated roadmap status. Ends cleanly; the next milestone is planned with tstack-plan-milestone in a fresh session.
---

# tstack-build

You are running TStack's per-milestone execution stage. The plan is already approved and the feature branch is checked out (`tstack-plan-milestone` did that). Your job is to implement, verify, merge, and update the roadmap. You do not re-litigate the plan unless it turns out to be wrong.

## Prereq check

Required state:

- `docs/ROADMAP.md` exists
- An approved implementation plan exists in the repo at `docs/plans/{id}.md` (written by `tstack-plan-milestone`). If it was committed, this works even when a cloud agent or a different machine — not the session that planned it — is doing the build.
- The current branch is a `milestone/*` branch (not `main`)

**Load the plan first.** Read `docs/plans/{id}.md` for the milestone you're building. If it's missing and there's no approved plan in context: stop and tell the user to run `tstack-plan-milestone` first. Don't try to plan-as-you-go — that's the failure mode this split exists to prevent. The plan file carries the "Done when" criteria copied from ROADMAP.md; still open `docs/ROADMAP.md` to confirm they match (if they've drifted, the roadmap is authoritative — flag it).

## Approach

### 1. Implement the plan

Work through the plan in order. **Commit frequently** — after each meaningful piece works, commit with a descriptive message tied to the plan step. Frequent commits are clean rollback points if a later step goes wrong.

For larger milestones, work in chunks (steps 1–3, then 4–6, then 7+) and verify incrementally. Don't try to land the whole milestone in one push.

If the plan turns out to be wrong mid-build (a missing piece, an unanticipated constraint, an underspecified edge case):
- For small course corrections, adjust and continue
- For anything that affects what the milestone *delivers* (scope creep, missing prerequisite, spec gap): stop and tell the user to either re-enter `tstack-plan-milestone` for this milestone, or run `tstack-specify-feature` if PRODUCT.md is wrong

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

If a criterion can't be verified by any of the above, the criterion is not testable. Stop and tell the user the roadmap entry has a soft criterion that needs rewriting before the milestone can be shipped. (This should be rare — `tstack-roadmap` now checks testability at write time. If you hit one here, it slipped through; fixing it is a roadmap edit, not a build judgment call.)

**Default is all-or-nothing: every criterion must PASS before the milestone ships.** If any criterion is FAIL: fix it. Don't move on. Don't mark the milestone done.

**Waiving a criterion (controlled escape hatch).** A criterion may be deferred *only* with explicit user sign-off, and only when it's a genuine scope split rather than a defect being hidden. When the user signs off:
1. Record it in the verification report as `Result: DEFERRED — {reason} (user-approved {date})`, not PASS.
2. Append a follow-up milestone to `docs/ROADMAP.md` (via `tstack-specify-feature` rules, or a direct append with explicit dependency) that carries the deferred criterion, so it isn't lost.
3. Note the known gap in the merge commit body.

Never silently mark a milestone done with a failing or unmet criterion. A waiver is a visible, tracked decision — not a soft pass.

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

**Team projects** — open a PR instead of merging directly. Use `gh pr create` if available, targeting `main`. If the project records review requirements of its own — in `AGENTS.md`, `CONTRIBUTING.md`, or `DECISIONS.md` (e.g., "requires one approval + passing CI") — follow them; TStack doesn't impose any. Put the verification report (from step 2) in the PR body so reviewers see the proof. **The milestone is not `Completed` until the PR actually merges** — don't update ROADMAP status (step 4) on PR open. If CI or review is still pending, tell the user the milestone is "in review" and pause the loop there; resume step 4 once it merges.

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

### 5. Report and stop

Report completion and point at what's next — then **end the session here**:

> {id} shipped and merged. Up next: {next-id} — {next-name}.
>
> **Plan it in a fresh session** — start a new session and run `tstack-plan-milestone` (or say "plan the next milestone"). A fresh window per milestone is the default: it stops context from piling up across milestones, which is what keeps a long roadmap sustainable. The roadmap on disk (`Completed:` / `Up next:`) is the only state the next session needs.

Do not auto-advance into `tstack-plan-milestone` or start planning the next milestone in this session on your own — the fresh-session boundary is the point. If the user *explicitly* wants to keep going now, that's their call; don't make it for them.

## Reference handoff

`references/full-guide.md` is the full implementation guide. The execution-relevant sections are "Build", "Verify Against Done When Criteria", "Merge", "Update Roadmap Status", and the prompting patterns for "Resuming After a Break" and "Struggling Mid-Milestone." Read those when the situation calls for them.

For a realistic verification-report example showing the quoted-command-output discipline across six different criteria types (unit tests, curl, Playwright, manual UI, axe a11y), read `references/example-output.md`.

## When the plan was wrong

Don't silently expand scope. If during build you find:

- The milestone is bigger than the plan accounted for → propose splitting (`Mxa` / `Mxb`) and re-enter `tstack-plan-milestone` for the second half
- A feature is misspecified in PRODUCT.md or API.md → stop, run `tstack-specify-feature` to update the docs, regenerate the relevant roadmap entries, then re-enter `tstack-plan-milestone`
- The plan referenced a doc section that doesn't exist → flag the gap, get the user to fill it before continuing

Silent overreach is the failure mode. Surface the issue, don't paper over it.

## Hand-back

After step 5, end the session cleanly — the next milestone is planned in a fresh session, not chained on here. The project state on disk reflects exactly what's been shipped: `Completed:` and `Up next:` in ROADMAP.md are the source of truth for what to do next, so nothing is lost across the boundary.
