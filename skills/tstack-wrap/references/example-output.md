# Example output — tstack-wrap

A realistic end-of-session sweep. Calibrate against the shape: gather → surface candidates (with the already-documented ones filtered out) → confirm the borderline one → write the in-lane records → route the out-of-lane one → report. Nothing is committed. The specific project doesn't matter.

---

## The sweep (what the skill says in chat)

**Wrapping up — swept the session against git + docs. Here's what I found.**

I looked at: this session's work (M4 — meal planner), `git log` since `Docs last synced: 2026-06-08` (7 commits), `git status`, and `git diff docs/`.

**Already documented — nothing to do (3):**
- The `mealPlan` schema and the `/api/meal-plans` endpoints — already in `docs/API.md` and the M4 commit messages. ✓
- M4 marked Completed, M5 set as Up next — already in `ROADMAP.md` Status. ✓
- The Zod validation pattern — already covered by the existing convention in `CONVENTIONS.md`. ✓

**Genuine gaps — proposed writes (4):**

1. **Decision:** we dropped the third-party meal-recommendation API and went with a simple rules engine, because the API's latency (~1.2s p95) blew the M4 "list renders < 300ms" criterion. → new dated ADR in `docs/DECISIONS.md`.
2. **Gotcha:** the planner's date math assumes the user's local timezone, not UTC — off-by-one bugs if you forget. Not obvious from the code. → code comment in `lib/mealPlan/dates.ts`.
3. **Operational:** `milestone/m4-meal-planner` is merged but the local branch wasn't deleted — it's stale. → note in `ROADMAP.md` Status.
4. **Borderline — need your call:** mid-build we talked about letting users *share* a meal plan via a public link. We didn't build it. Is that (a) a passing idea, or (b) something to capture? If (b), it's new scope — that's a `tstack-specify` job, not something I'll write into the docs here.

> *User: (4) is real, let's not lose it. Write the other three.*

**Done. Here's what went where:**

- **`docs/DECISIONS.md`** — appended **ADR-009: Rules engine over third-party recommendation API** (dated 2026-06-14), with the latency rationale and a revisit trigger.
- **`lib/mealPlan/dates.ts`** — added a comment above `addPlanDays()` flagging the local-timezone assumption.
- **`docs/ROADMAP.md`** (Status section) — noted the stale local branch `milestone/m4-meal-planner` (merged, safe to delete).
- **Routed out:** the public-share-link idea is new scope — **run `tstack-specify`** to spec it and add a milestone. I did *not* write it into the docs.

**Nothing committed** — review the changes and commit when you're ready.

---

## What the written records look like

**ADR appended to `docs/DECISIONS.md`** (matches the file's existing ADR format):

```markdown
## ADR-009: Rules engine over third-party meal-recommendation API

Chosen: 2026-06-14
Status: Accepted

**Context.** M4 required meal recommendations. We prototyped the FoodAPI
recommendation endpoint but measured ~1.2s p95 latency, which breaks the M4
"meal-plan list renders < 300ms" criterion.

**Decision.** Build a small in-app rules engine (dietary tags + pantry match)
instead of calling the external API.

**Consequences.** Lower-quality recommendations than the API, but deterministic
and fast. Revisit if (a) recommendation quality becomes a product complaint, or
(b) we add server-side caching that makes the API latency acceptable.
```

**Comment added in `lib/mealPlan/dates.ts`:**

```ts
// NOTE: plan dates are computed in the user's LOCAL timezone, not UTC.
// Converting to UTC here causes off-by-one day errors at midnight boundaries.
export function addPlanDays(start: Date, n: number): Date {
```

**Status note in `docs/ROADMAP.md`** (Status section only — no milestone added or renumbered):

```markdown
## Status
Completed: M0, M1, M2, M3, M4
Up next: M5 — Shopping list
Docs last synced: 2026-06-08
Notes: local branch `milestone/m4-meal-planner` is merged but not deleted (stale; safe to `git branch -d`).
```

---

### Example — light pass, no `docs/` yet

**Swept the session.** No `docs/` tree, so this isn't a TStack-managed project — I checked `git log` and the code only.

- One gotcha worth keeping: the auth middleware silently no-ops if `SESSION_SECRET` is unset (no error thrown). → added a comment in `middleware/auth.ts`.
- Nothing else undocumented.

If you want the full living-docs treatment (DECISIONS, ROADMAP, drift tracking), run `tstack-discover` to start the chain. **Nothing committed.**
