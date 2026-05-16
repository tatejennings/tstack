# Implementation Guide — Building from the Roadmap

## Purpose

You have a `ROADMAP.md` with sequenced milestones, dependencies, and "done when" criteria. This guide covers the practical workflow for implementing each milestone with Claude Code — from starting a feature branch to merging a completed milestone.

---

## The Milestone Loop

Every milestone follows the same cycle:

```
1. Update ROADMAP.md Status section → set "Up next" to this milestone
2. Create a feature branch
3. Start Claude Code → plan mode → read specs → plan
4. Review the plan → adjust → approve
5. Build (Claude Code implements)
6. Verify against "done when" criteria
7. Commit, push, merge
8. Update ROADMAP.md Status section → move milestone to "Completed", set next "Up next"
```

---

## Step by Step

### 1. Pick the Next Milestone

Open `docs/ROADMAP.md` and scroll to the Status section at the bottom. The "Up next" milestone is what you're starting. Make sure its dependencies are all in the "Completed" list.

Commit this change to `main` so the roadmap reflects current state.

### 2. Create a Feature Branch

Use a consistent naming convention:

```bash
# Pattern: milestone/{id}-{short-description}
git checkout -b milestone/m4-entity-crud
```

For iOS milestones:

```bash
git checkout -b milestone/i2-entity-list
```

If a milestone spans both server and client work, use the milestone ID:

```bash
git checkout -b milestone/m21-sync-endpoint
```

### 3. Start Claude Code in Plan Mode

Open Claude Code and start with plan mode (Shift+Tab). Your first prompt should tell Claude Code exactly what milestone you're working on and which docs to read.

**Example prompt — starting a backend milestone:**

```
We're starting milestone M2 — Auth & User Profiles.

Read these docs before planning:
- docs/ARCHITECTURE.md — Auth section
- docs/API.md — Auth endpoints, Profile endpoints
- docs/2 - Specs/authentication.md — full spec
- docs/PRODUCT.md — Section 2.1 (User accounts)

Then plan the implementation. List the files you'll create or modify, in what order,
and how you'll verify each piece works.
```

**Example prompt — starting a frontend milestone:**

```
We're starting milestone M5 — Dashboard UI.

Read these docs before planning:
- docs/PRODUCT.md — Section 4.1 (Dashboard), Section 5.3 (Dashboard Screens)
- docs/CONVENTIONS.md — Component patterns, file organization
- docs/API.md — Dashboard data endpoints

Then plan the implementation. List the components, pages, and hooks you'll create,
the order you'll build them, and how we'll verify each piece works.
```

**Example prompt — starting a mobile milestone:**

```
We're starting milestone i3 — Voice Capture.

Read these docs before planning:
- docs/PRODUCT.md — Section 4.2 (Voice Capture Flow), Section 5.2 (Capture Screens)
- docs/PRODUCT.md — Section 16.1 (Voice Capture Edge Cases)

Then plan the implementation. List the Views, ViewModels, and Services you'll create,
the order you'll build them, and how we'll test each piece.
```

### 4. Review the Plan

Claude Code will produce an implementation plan. Before approving, check:

- **Does it cover all "done when" criteria?** Cross-reference with ROADMAP.md.
- **Is the file structure consistent with CONVENTIONS.md?** (naming, location, patterns)
- **Does it introduce any new dependencies?** If so, is that justified?
- **Is the order sensible?** (utilities before consumers, schemas before routes, etc.)
- **Is anything missing?** Error handling, validation, RLS tests, edge cases?

Push back if needed:

```
Your plan doesn't include RLS testing — the "done when" criteria requires that
user A cannot see user B's entities. Add a verification step for that.
```

```
You're creating a new helper in lib/utils.ts but we already have a similar pattern
in lib/supabase/server.ts. Use the existing pattern instead.
```

```
Move the Zod schema to a shared location — the iOS sync endpoint will need the
same validation shapes later.
```

### 5. Build

Switch out of plan mode (Shift+Tab) and tell Claude Code to implement:

```
Plan looks good. Implement it, starting with step 1.
```

For larger milestones, you can work through the plan in chunks:

```
Start with steps 1-3 (the database helpers and encryption integration).
Run the tests after each step. We'll do the API routes next.
```

**Commit frequently during implementation.** After each meaningful piece works:

```
Commit this with a descriptive message. We just finished the entity encryption
helpers and they're passing tests.
```

This gives you clean rollback points if something goes sideways later.

### 6. Verify Against "Done When" Criteria

Before considering the milestone complete, walk through every criterion in ROADMAP.md explicitly:

```
Let's verify this milestone is complete. Here are the "done when" criteria from
ROADMAP.md for M4:

- GET /api/entities returns user's entities with decrypted legal_name, masked ein, decrypted members
- POST /api/entities encrypts legal_name, ein, members before insert
- PATCH /api/entities/:id updates fields (re-encrypts changed encrypted fields)
- DELETE /api/entities/:id soft-deletes (sets is_active = false)
- Zod validation on POST/PATCH request bodies
- RLS: user A cannot see user B's entities (test with two auth sessions)
- API error responses follow the standard { error, code, details } format

Run through each one and verify it works. For the RLS test, create two test users
and confirm cross-user access is blocked.
```

### 7. Merge

Once everything passes:

```bash
# Make sure everything is committed
git add -A
git commit -m "feat: complete M4 — entity CRUD with encryption and RLS"

# Push and create PR (or merge directly for solo projects)
git push origin milestone/m4-entity-crud

# Merge to main
git checkout main
git merge milestone/m4-entity-crud
git push origin main

# Clean up
git branch -d milestone/m4-entity-crud
```

> **Team projects:** Replace the direct merge above with your team's PR workflow — create a PR from the milestone branch, run CI, get code review, then merge. The rest of the milestone loop stays the same.

### 8. Update Roadmap Status

Open the Status section at the bottom of `docs/ROADMAP.md`. Move the milestone to "Completed" and update "Up next":

```markdown
**Completed:**
- M0 — Monorepo & Tooling Setup
- M1 — Database Schema & Migrations
- M2 — Auth & Profiles
- M3 — Field-Level Encryption
- M4 — Entity CRUD + RLS          ← just finished

**Up next:** M5 — Account CRUD + Entity-Account Links
```

Commit this to `main`:

```bash
git add docs/ROADMAP.md
git commit -m "docs: M4 complete, up next M5"
git push origin main
```

---

## Prompting Patterns

### Starting a Session

Always ground Claude Code in where you are:

```
We're working on milestone M10 — Statement Import (CSV). We're on branch
milestone/m10-csv-import. Read ROADMAP.md for the milestone details, then
read the specs listed there.
```

### Resuming After a Break

If you're picking up where you left off:

```
We're continuing milestone M10 — Statement Import (CSV). Check the current
state of the code and tell me what's been implemented so far vs. what's
remaining from the "done when" criteria in ROADMAP.md.
```

### When Something Isn't Working

Be specific about what's failing:

```
The Chase CSV parser is incorrectly parsing negative amounts as positive.
The raw CSV line looks like: "01/15/2026","PAYMENT RECEIVED",-500.00
Look at the parser in lib/statements/parsers/chase.ts and fix the sign handling.
```

Not:

```
The CSV parser is broken, fix it.
```

### When You Want to Explore Before Building

```
Before we implement the reconciliation scorer, I want to think through edge cases.
Read docs/2 - Specs/reconciliation.md and walk me through what happens when:
1. Two captures match the same statement record
2. A refund comes in for a previously matched transaction
3. The same vendor charges the same amount two days in a row
```

### When You Want Tests First

```
Before implementing the encryption module, write the test file first.
Test cases should cover:
- Round-trip encrypt/decrypt for strings, JSON, and amounts
- Different encryptions of the same plaintext produce different ciphertext
- Missing encryption key throws a clear error
- Key ID is embedded in the ciphertext payload

Then implement the module to make the tests pass.
```

### When You Need to Fix Something from a Previous Milestone

Don't fix it on the current milestone branch. Create a hotfix:

```bash
git stash  # save current work
git checkout main
git checkout -b fix/m4-entity-rls-policy
```

Fix, merge, then return:

```bash
git checkout main
git merge fix/m4-entity-rls-policy
git checkout milestone/m10-csv-import
git rebase main  # pick up the fix
git stash pop    # restore current work
```

---

## Git Workflow Summary

```
main
 │
 ├── milestone/m0-monorepo-setup        (branch, build, merge)
 ├── milestone/m1-database-schema       (branch, build, merge)
 ├── milestone/m2-auth-profiles         (branch, build, merge)
 ├── fix/m1-missing-index               (hotfix if needed)
 ├── milestone/m3-encryption            (branch, build, merge)
 ├── milestone/m4-entity-crud           (branch, build, merge)
 └── ...
```

### Commit Message Convention

```
feat: complete M4 — entity CRUD with encryption and RLS
fix: M1 — add missing index on entities.user_id
docs: M4 complete, up next M5
refactor: M4 — extract encryption helpers to shared package
test: M4 — add RLS cross-user isolation tests
```

Prefix with the type, reference the milestone. Keep it scannable in `git log`.

### When to Commit

- After each working piece within a milestone (not just at the end)
- After writing tests that pass
- Before attempting anything risky or experimental
- Before running `/clear` or `/compact` in Claude Code

### When to Branch Off Main vs. Off Another Branch

- **Always branch off `main`.** Each milestone branch starts fresh from the latest main.
- **Never stack milestone branches** (branching m5 off m4's branch). If m5 depends on m4, merge m4 first.
- **Exception:** If two milestones are parallelizable (noted in ROADMAP.md), they can both branch off the same main commit and be merged independently.

---

## Parallel Milestones

When ROADMAP.md identifies milestones that can run in parallel, you can work on them simultaneously using git worktrees:

```bash
# Create a worktree for each parallel milestone
git worktree add ../project-m8-inngest milestone/m8-inngest-setup
git worktree add ../project-m3-encryption milestone/m3-encryption

# Run separate Claude Code sessions in each worktree
cd ../project-m8-inngest && claude
cd ../project-m3-encryption && claude
```

Or work on them sequentially on separate branches — the point is they don't block each other, so you can switch between them if you get stuck on one.

---

## Session Management

### When to Start a Fresh Session

- Starting a new milestone
- After merging and switching branches
- When Claude Code's context seems polluted (repeating mistakes, forgetting conventions)
- After running `/compact` and noticing degraded output quality

### When to Continue a Session

- Still working on the same milestone
- Just took a short break
- Iterating on something Claude Code just built

### Context Hygiene

```
# Clear context between unrelated tasks
/clear

# Compact if the session is long but you want to keep going
/compact Focus on the CSV parser implementation and test results

# Check what's using context
/context
```

---

## Troubleshooting

### Claude Code Keeps Deviating from Conventions

Re-anchor it:

```
Stop. Read docs/CONVENTIONS.md, specifically the naming conventions and
file organization sections. The file you just created doesn't follow our patterns.
Rename it and restructure to match.
```

### Claude Code Suggests Changing an Architecture Decision

Point to the ADR:

```
We already evaluated that approach. See docs/DECISIONS.md — ADR-002 explains
why we chose application-level encryption over database-level. Don't change this.
```

### A Milestone Is Taking Longer Than Expected

If you're past 2 days on a milestone, it's probably too big. Split it:

1. Identify what's already working
2. Commit and merge what's done as a partial milestone
3. Update ROADMAP.md — add the partial completion to "Completed" with a note, add a new milestone for the remainder

```
# In ROADMAP.md, update the milestone and add a new one:

## M10 — Statement Import (CSV)
Note: Chase and Amex parsers complete. Remaining bank parsers moved to M10b.

## M10b — Statement Import (Remaining Parsers)
...

# In the Status section at the bottom:
**Completed:**
- ...
- M10 — Statement Import (CSV) (partial — Chase + Amex only)

**Up next:** M10b — Statement Import (Remaining Parsers)
```

### You Disagree with the Plan

Say so before Claude Code starts building:

```
I don't think we need a separate service class here. The parsing logic is simple
enough to live in the API route handler directly. Simplify the plan — inline the
parsing and skip the service abstraction.
```

It's much cheaper to fix a plan than to fix an implementation.

---

## Updating Documentation During Implementation

Documentation is created before code, but implementation will reveal things the docs got wrong or didn't anticipate. When that happens:

1. **Update the doc first, then continue building.** Don't let implementation drift from documentation — that's how you end up re-explaining context every session.

2. **Keep changes scoped.** Update the specific section that's wrong, not the whole document. If an API endpoint needs an extra field, update that endpoint in API.md. Don't rewrite the entire file.

3. **Record significant changes in DECISIONS.md.** If you're reversing or modifying a previous ADR (e.g., "we said we'd use JWT but switched to session tokens"), add a new ADR explaining why. Mark the old one as `Superseded by ADR-{N}`.

4. **Check downstream impact.** If the change affects other milestones, review ROADMAP.md. A schema change in M4 might affect the "done when" criteria for M7. Update them now, not when you get to M7.

5. **Commit doc changes separately.** Use `docs:` prefix in your commit message so doc updates are easy to find in git log:
   ```
   docs: update API.md — add pagination params to GET /entities
   ```
