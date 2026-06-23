# Example output — tstack-status

A realistic Status-mode report in chat. Calibrate against the scannable, headed shape and the explicit drift flag — not the specific project.

---

**Project status — RecipeBox** *(read-only; nothing changed)*

**Shipped (4)**
- M0 — Infrastructure baseline
- M1 — Auth + user model
- M2 — Recipe CRUD
- M3 — Recipe search

**In progress**
- On branch `milestone/m4-meal-planner`. Plan at `docs/plans/m4.md`. Not yet merged.

**Up next**
- M4 — Meal planner *(in progress, above)*. Dependencies M2, M3 ✓ — unblocked.
- After M4: M5 — Shopping list. Depends on M4 → currently **blocked** until M4 ships.

**Mandatory docs** — all present ✓ (PRODUCT, ARCHITECTURE, CONVENTIONS, TESTING, DECISIONS, ROADMAP). API.md present (optional).

**⚠ Doc drift**
- `PRODUCT.md` was edited 2026-05-26, after ROADMAP.md's last sync (`Docs last synced: 2026-05-18`). The roadmap may not reflect the latest requirements. If that edit added scope, re-run `tstack-roadmap`; if it was a surgical `tstack-specify-feature` change, you can ignore this.

**Suggested next action**
- Finish and verify M4, then merge (`tstack-build`). But first, resolve the drift on `PRODUCT.md` — confirm whether the roadmap needs re-sequencing.

---

### Example — early-stage project (no roadmap yet)

**Project status — Untitled** *(read-only)*

- `docs/1 - Discovery/business-brief.md` ✓
- `docs/PRODUCT.md` ✓
- No `ARCHITECTURE.md`, no `ROADMAP.md`.

You're two stages into the chain (discovery + product done). **Next action: run `tstack-architect`** to produce the technical doc set, then `tstack-roadmap`.
