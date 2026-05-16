# Example output: `tstack-specify` doc-impact proposal

Realistic example of the proposal `tstack-specify` produces when a **Slink** user (post-launch, after M0–M12 shipped) asks to add team workspaces. Shows the two-list format with per-item approval, the "considered but no change" line, and the milestone-insertion that follows.

---

## Feature interview (summary)

User asked: "I want to add team workspaces — multiple people in one account, shared link library, role-based permissions."

After the interview, the feature shape is:

- **What:** Team workspaces. A `Workspace` owns `Links` and `Domains` (currently owned by `User`). A `User` can be a member of multiple workspaces with one role (`owner`, `editor`, `viewer`).
- **Who:** Existing Pro users who upgrade to Team ($29/mo). Secondary persona: the "editor" who never had a Slink account before.
- **User flow:** Workspace owner invites a teammate by email → invitee gets email link → signs up via Clerk (if new) → joins the workspace → sees the shared link library scoped to that workspace.
- **Data:** New `Workspace` and `WorkspaceMember` tables. Existing `Link.user_id` and `Domain.user_id` migrate to `workspace_id` (every existing single-user account becomes a workspace with one owner).
- **Acceptance criteria highlights:** Editor can create/edit links but not change billing or member roles. Viewer can only see (read-only). Workspace count per Team account: 1 (Team = one workspace; "agency" tier with multiple workspaces is a separate future feature).

## Doc-impact proposal

```
Proposed updates for "Team workspaces":

Changes I want to make:
- [ ] docs/PRODUCT.md — new feature section F-10 (Team Workspaces); updated data model (Workspace, WorkspaceMember entities; Link/Domain ownership shift from user_id to workspace_id); updated business-model paragraph (Team tier becomes "1 workspace + up to 5 members")
- [ ] docs/API.md — new endpoints: POST/GET /api/workspaces, POST /api/workspaces/:id/members, DELETE /api/workspaces/:id/members/:userId, PATCH /api/workspaces/:id/members/:userId (role change). Existing /api/links and /api/domains gain a workspace_id scope in their queries (documented breaking-change note for clients on the v1 API; mitigation: API v2 with workspace_id required, v1 grandfathered for 90 days)
- [ ] docs/2 - Specs/team-workspaces.md — new file: ownership model, RLS rewrite, invite flow, role enforcement matrix, edge cases (last owner can't leave, deleted member's links stay with workspace)
- [ ] docs/DECISIONS.md — new ADR-15: "Team workspaces as a separate ownership entity (not multi-user-shared records)" with rationale, alternatives considered (per-record sharing was rejected as exponentially harder)
- [ ] docs/ROADMAP.md — append M13 (workspace data model migration), M14 (invite flow + member management UI), M15 (role enforcement across all link/domain endpoints), M16 (Stripe Team-tier billing changes). Dependencies: M13 depends on a stable v12 (current state). M14 depends on M13. M15 depends on M14. M16 can start in parallel with M14.

Docs I considered but propose NOT to change (challenge any of these if you disagree):
- docs/ARCHITECTURE.md — no change. Workspaces are an in-app ownership concept; no new module, no new runtime, no new data store. The existing data-flow diagram still describes what runs where. (If we added cross-workspace analytics or workspace switcher real-time sync, this would change.)
- docs/CONVENTIONS.md — no change. No new code patterns; the workspace_id WHERE clauses follow the existing per-user pattern.
- docs/TESTING.md — no change. The two-user integration test fixture extends naturally to a two-workspace fixture; no new test infrastructure needed. The existing axe + Playwright setup covers the new screens.
- docs/2 - Specs/database-schema.md — no change to the existing spec; the new tables get their own spec (team-workspaces.md). Cross-reference will be added inside team-workspaces.md.
- docs/2 - Specs/dns-verification.md — no change. Domains move from user_id to workspace_id but the verification flow itself doesn't change.
- docs/2 - Specs/ai-strategy.md — no change. AI slug suggestions remain per-link; no workspace-level concept needed in v1 of teams.

Reply: approve / modify / reject per item. Type "approve all" if everything looks right, or call out a "NOT to change" item you think I'm wrong about.
```

## User pushback (in actual usage, this is where the user challenges items)

User replied:
> approve all except: I think ARCHITECTURE.md *does* need a small update — we're shifting the security boundary. ADR-1 says "row ownership via user_id WHERE clauses." That's now "via workspace_id WHERE clauses, with workspace membership lookup as a join." That's a meaningful architectural change to the data-access layer. Add ARCHITECTURE.md back in.

`tstack-specify` accepts the pushback and re-presents:

```
Updated proposal — adding ARCHITECTURE.md back in:

- [ ] docs/ARCHITECTURE.md — § Security boundary updated: "row ownership enforced via workspace_id WHERE clause + workspace membership join. RLS policies updated to reflect workspace ownership." § Data Flow diagram annotation: the auth check happens at workspace-membership lookup, not just user-id match.
```

User: "approve all."

## Applying edits (one at a time, with diff preview)

For each approved item, `tstack-specify` reads the current section, drafts the change, presents the diff, gets a final go-ahead, applies the edit, and commits. Example for one of the docs (showing the pattern, not the full content):

```
$ git diff docs/PRODUCT.md
@@ -210,6 +210,32 @@
 ## F-9: CSV Export
 ...
+## F-10: Team Workspaces
+
+(...new section content here, ~80 lines...)
+
+**Acceptance criteria:**
+- Given an owner with a pending invite, when the invitee accepts, then they appear in the member list with role "editor" by default.
+- Given a workspace with one owner, when the owner tries to leave, then the action is blocked with "Transfer ownership first."
+- ...
```

User confirms. `tstack-specify` applies the edit and commits: `docs(product): add F-10 team workspaces feature spec`.

## Appending milestones to ROADMAP.md

Inserts M13–M16 *between* the existing "Up next" pointer and the "Completed" list, in dependency order. Updates the dependency graph at the top of ROADMAP.md. Status section is **not** touched — the user can promote M13 to "Up next" themselves when they're ready to start.

Sample new milestone entry appended:

```markdown
## M13 — Workspace data model migration

**What gets built:** New `Workspace` and `WorkspaceMember` tables (per `docs/2 - Specs/team-workspaces.md`). Migration that converts every existing `User`-owned `Link` and `Domain` into a personal workspace owned by that user. Existing endpoints continue to work via a backwards-compatible shim until M15 ships v2.

**Dependencies:** None within the new milestones (depends on the v12 stable state, which is Completed).

**Read before starting:**
- `docs/2 - Specs/team-workspaces.md` — § Ownership Model, § Migration Plan
- `docs/PRODUCT.md` — § F-10 (Team Workspaces)
- `docs/DECISIONS.md` — ADR-15

**Done when:**
- `pnpm db:migrate` runs cleanly forward and backward on a fresh test DB.
- Migration on a snapshot of prod data preserves every Link and Domain row (counts match pre/post).
- Existing API endpoints (v1) continue to work for all existing users (verified by running M3 + M5 test suites against the migrated DB).
- New `Workspace` and `WorkspaceMember` tables have RLS policies matching ADR-15 — cross-workspace isolation test passes.
- A11y: no UI in this milestone, but the audit log of who-migrated-when is human-readable for support purposes.

**Estimated effort:** 2 days.
```

## Handoff

> Feature "Team workspaces" specified. Doc updates: PRODUCT.md, API.md, ARCHITECTURE.md, DECISIONS.md (ADR-15), new spec 2 - Specs/team-workspaces.md, ROADMAP.md (M13–M16 appended).
>
> **Next: run `tstack-plan`** (or say "plan milestone M13") to start implementation. M13 is the data migration; M14 (invite flow + UI), M15 (role enforcement), M16 (billing) follow in sequence.
