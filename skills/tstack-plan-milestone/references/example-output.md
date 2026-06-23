# Example output: `tstack-plan-milestone` plan-mode output

Realistic example of an approved plan for **Slink** milestone M3 (Link CRUD). This is what gets presented to the user inside plan mode, refined through review, and then approved before handing off to `tstack-build`.

---

## Plan: M3 — Link CRUD (manual slugs)

**Branch created:** `milestone/m3-link-crud` (off `main`, clean working tree).

**Docs read:**
- `docs/PRODUCT.md` — § F-2 (Link Creation), § Data Models / `Link`
- `docs/API.md` — `GET /api/links`, `POST /api/links`, `PATCH /api/links/:id`
- `docs/CONVENTIONS.md` — § Server Actions, § Form validation, § Error response format
- `docs/TESTING.md` — § Integration test patterns

Cross-checked plan against M3's "Done when" criteria — every criterion is addressed by a specific step below.

## Files to create or modify (dependency order)

### Layer 1: schema + validation (foundation)

1. **`packages/shared/src/schemas/link.ts`** (new) — Zod schemas: `LinkCreateInput`, `LinkUpdateInput`, `LinkListQuery`. Used by both the form (client-side validation) and the API (server-side validation). Reuses the URL validator already in `packages/shared/src/validators/url.ts`.

2. **`packages/db/src/queries/links.ts`** (new) — Drizzle helpers: `listLinks(userId, opts)`, `createLink(userId, input)`, `updateLink(userId, linkId, patch)`, `archiveLink(userId, linkId)`. Every helper enforces `WHERE user_id = $userId`. RLS is the defense-in-depth, but the helpers are the primary fence.

### Layer 2: API routes (the contract)

3. **`web/app/api/links/route.ts`** (new) — `GET` (list, filtered by `?archived=false`) and `POST` (create). Uses the schemas + queries. Returns the standard `{ data | error }` shape from `web/lib/api/response.ts`. Plan-tier limit enforced via `web/lib/auth/limits.ts` (existing helper from M2's domain limit; extend with a `linksPerMonth` field).

4. **`web/app/api/links/[id]/route.ts`** (new) — `PATCH` (update destination), `DELETE` (archive).

### Layer 3: UI (the surface)

5. **`web/app/(dashboard)/links/page.tsx`** (new) — Server Component, fetches links via `listLinks(userId, …)`. Renders `<LinkList>` (Client Component for search/filter).

6. **`web/components/links/LinkList.tsx`** (new) — list view with client-side search (no API call); empty state with concrete CTA.

7. **`web/components/links/CreateLinkForm.tsx`** (new) — Server Action submission; optimistic update on success; surfaces 409 (slug taken) inline with form retention. **No** AI slug button yet — that's M4. Manual slug input only.

8. **`web/components/links/LinkRow.tsx`** (new) — single row in the list with inline edit-destination + archive.

### Layer 4: tests

9. **`packages/db/src/queries/links.test.ts`** (new) — unit tests for each query helper. Two-user fixture confirms cross-user isolation (user A's query returns 0 rows for user B's links).

10. **`web/tests/integration/links-crud.spec.ts`** (new) — Playwright: create → list → edit → archive → confirm 410 from redirect Worker on archived slug.

11. **`web/tests/integration/links-limits.spec.ts`** (new) — Free user at 4/5: UI shows progress. At 5/5: create blocked with upgrade CTA. API returns 402 `{ code: "PLAN_LIMIT" }`.

## Patterns reused (not reinvented)

- **`web/lib/api/response.ts`** — existing helper for `{ data, error, code }` shape. Used by every endpoint in this milestone.
- **`web/lib/auth/limits.ts`** — existing limit-check helper from M2. Extending with `linksPerMonth`, not duplicating.
- **`packages/shared/src/validators/url.ts`** — existing URL validator. Reused in `LinkCreateInput`.
- **`web/components/ui/*`** (shadcn) — reusing `Input`, `Button`, `Card`, `Dialog`. Zero net new shadcn additions.

## Patterns introduced (with rationale)

- **Server Component → Client Component split for list views.** First milestone with a search-heavy list; establishing the pattern here so M6 (analytics) can follow it. ADR-15 will record this if not already noted.
- **`packages/db/src/queries/{entity}.ts` convention.** First per-entity query file. Going forward every entity gets one. Documented in CONVENTIONS.md after this milestone ships.

## Verification approach (per Done-when criterion)

| Criterion | How to verify |
|---|---|
| CRUD endpoints with Zod + per-user WHERE | `curl` each endpoint with two distinct auth tokens; show second user can't see/edit/delete first user's link |
| Slug uniqueness per `domain_id`; 409 on conflict | `curl` POST same slug twice on same domain; show 409 + correct error code |
| Free-tier link limit | Run `links-limits.spec.ts` in Playwright; show pass output |
| Search filters client-side, no API call | Open the list, type in search, show network tab with no new requests |
| Soft-delete → 410 Gone via redirect | Archive a link, hit redirect Worker; show `HTTP/1.1 410 Gone` |
| a11y: list view keyboard-navigable; axe passes | Run axe-core via Playwright; show 0 violations |

## Tradeoffs the user should weigh before approving

1. **Optimistic UI updates** in `CreateLinkForm`. Trade: faster feel, occasional rollback flash on 409. Default: yes, optimistic — the 409 case is rare and we handle it. Alternative: no optimism, slight UI delay. **Recommendation: optimistic.**

2. **Client-side search only** (no server-side `?q=`). Works for users with <500 links (Pro/Team limit territory). Will need server-side at higher tiers — but that's not v1 scope. **Recommendation: client-side only, document the limit.**

3. **Archive vs hard-delete.** PRODUCT.md says archive at v1 (recoverable for 90 days). Plan respects this. **No tradeoff to discuss.**

## Out of scope for M3 (explicitly)

- AI slug suggestions — that's M4.
- Bulk operations (multi-select archive) — post-launch.
- Link folders / tags — not in v1.
- CSV import — M11 territory.
- Domain switcher on the create form — single-domain users only in M3 (most users); multi-domain users see a domain dropdown, but it lists their existing domains only (no inline create).

---

**Plan approved by user at 14:32.** Handing off to `tstack-build` in this session.
