# Example output — tstack-ingest

Two artifacts: the **coverage report** you present in chat, and the **draft** you write to `docs/_adopted/`. Calibrate against the shape and the honesty (named gaps, nothing fabricated) — not the specific project.

---

## 1. Coverage report (in chat)

**Adopting — RecipeBox** *(from `notes/prd-v2.md` + `design/screens.md`)*

**Intent check** — The material describes a recipe-organizer web app for home cooks, with AI recipe suggestions and a meal planner. You confirmed this is still current. ✓

**Product material → `PRODUCT.draft.md`**
- **Recipe CRUD** — *Present (PRODUCT-grade).* Flows + Given/When/Then criteria + `Recipe` data model all in the source.
- **Recipe search** — *Present.* Flow + measurable threshold ("≤300ms p95") present.
- **AI suggestions** — *Thin.* Feature described, but no eval set / quality bar / fallback → flagged as a gap (it's a non-deterministic feature; `tstack-product` needs eval-based criteria).
- **Meal planner** — *Thin.* Flow present, acceptance criteria soft ("should be easy to rearrange") → flagged.
- **Data models** — *Present* for `Recipe`, `User`; *Missing* for `MealPlan` (referenced but never defined) → flagged.

**Staged as notes → `architecture-notes.md`** *(not transcribed — `tstack-architect` will ask its ADR questions fresh)*
- Stack + hosting choices from `prd-v2.md §8`.
- A rough data-flow sketch from `design/screens.md`.

**Design material → `design.draft.md`** *(routed to `tstack-design`, which ratifies it into `docs/3 - Design/`)*
- Screen layouts + navigation from `design/screens.md` (mockups) — flagged: states beyond the happy path (empty/loading/error) not shown.

**Considered, not adopting**
- `notes/standup-2026-03.md` — meeting notes, not requirements.

**Conflict to resolve before I write:** `prd-v2.md §3` says recipes can't be edited after sharing; `design/screens.md` shows an "Edit" button on shared recipes. Which is canonical?

→ Once you resolve that and approve, I write the drafts and route you to `tstack-product` (with the design draft waiting for `tstack-design` after product).

---

## 2. `docs/_adopted/PRODUCT.draft.md` (excerpt)

```markdown
## Adoption status
- Source: `notes/prd-v2.md`, adopted 2026-06-20. *(Screen designs from `design/screens.md` staged separately in `design.draft.md` for `tstack-design`.)*
- **Unratified draft — not a canonical doc.** Ratify via `tstack-product`, which authors
  `docs/PRODUCT.md` from this and resolves the gaps below through its requirements gate.

## Open gaps
- **AI suggestions** has no eval set, quality bar, or fallback — non-deterministic feature,
  needs eval-based acceptance criteria.
- **Meal planner** acceptance criteria are soft ("easy to rearrange") — needs Given/When/Then.
- **`MealPlan` data model** is referenced but never defined (entities, relationships, field types).
- No accessibility criteria stated on any consumer-facing feature.
- No retention/deletion behavior stated for stored user data.

---

## Product Overview
RecipeBox — a recipe organizer for home cooks. {carried from prd-v2.md §1}

## Features

### Recipe CRUD
{flow carried from prd-v2.md §2}
**Acceptance criteria** (from source):
> Given a logged-in user, when they save a recipe with a title and ≥1 ingredient,
> then it appears in their recipe list.

### AI suggestions
{description carried from prd-v2.md §5}
> ⚠ Open gap: no eval set / quality bar / fallback. tstack-product to define.

## Data Models
### Recipe  {carried — entities, fields, relationships from prd-v2.md §6}
### MealPlan  ⚠ Open gap: referenced in §4 but undefined.
```

Note what the draft does **not** do: it never invents the missing `MealPlan` schema or writes acceptance criteria the source didn't have. Gaps are named, not filled — filling them is `tstack-product`'s job, through its gate.
