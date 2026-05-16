# Example output: `tstack-build` verification report

Realistic verification report from shipping **Slink** M3 (Link CRUD). Demonstrates the **quoted-command-output** discipline — every "Done when" criterion is verified by a real command whose output is pasted back into the conversation. No "verified ✓" without evidence.

This is the artifact `tstack-build` produces between step 2 (Verify) and step 3 (Merge).

---

## Verification for M3 — Link CRUD (manual slugs)

### Criterion 1: All Link CRUD endpoints exist with Zod validation and per-user WHERE clauses; cross-user isolation works

Command: `pnpm test packages/db/src/queries/links.test.ts`

Output:
```
 RUN  v3.0.5 packages/db

 ✓ packages/db/src/queries/links.test.ts (8 tests) 312ms
   ✓ listLinks > returns only the requesting user's links 47ms
   ✓ listLinks > filters by archived flag 38ms
   ✓ createLink > inserts with user_id from session 41ms
   ✓ createLink > rejects when slug already exists for same domain 35ms
   ✓ updateLink > rejects when link belongs to another user 39ms
   ✓ archiveLink > sets archived=true and clears no other fields 36ms
   ✓ Zod schema > rejects malformed URL with field-level error 38ms
   ✓ Zod schema > rejects slug >50 chars 38ms

 Test Files  1 passed (1)
      Tests  8 passed (8)
   Duration  428ms
```

Cross-user isolation specifically: `updateLink > rejects when link belongs to another user` — the test creates user A's link, attempts to update from user B's session, expects a `RowNotFound` error (because the WHERE clause filters out non-owned rows). All 8 tests pass.

Result: **PASS** — endpoints + Zod + per-user WHERE confirmed by passing test suite.

### Criterion 2: Slug uniqueness per domain_id; 409 with specific error code on conflict

Command: `curl` against running dev server (two attempts; second is the duplicate).

Output:
```
$ curl -X POST http://localhost:3000/api/links \
    -H "Authorization: Bearer $TOKEN_RILEY" \
    -H "Content-Type: application/json" \
    -d '{"domainId":"d-1","slug":"issue-47","destination":"https://riley.fm/47"}'

HTTP/1.1 201 Created
content-type: application/json

{"data":{"id":"l-9k2","slug":"issue-47","destination":"https://riley.fm/47"}}

$ curl -X POST http://localhost:3000/api/links \
    -H "Authorization: Bearer $TOKEN_RILEY" \
    -H "Content-Type: application/json" \
    -d '{"domainId":"d-1","slug":"issue-47","destination":"https://riley.fm/47-alt"}'

HTTP/1.1 409 Conflict
content-type: application/json

{"error":"slug already taken on this domain","code":"SLUG_CONFLICT","field":"slug"}
```

Result: **PASS** — 409 returned with `code: "SLUG_CONFLICT"` matching CONVENTIONS.md § Error response format.

### Criterion 3: Free-tier link limit enforced

Command: `pnpm playwright test apps/web/tests/integration/links-limits.spec.ts`

Output:
```
Running 3 tests using 1 worker

  ✓ links-limits.spec.ts:8 → free user sees "4 of 5 links used" at 4/5 (1.2s)
  ✓ links-limits.spec.ts:24 → free user at 5/5 sees upgrade CTA and create button is disabled (1.4s)
  ✓ links-limits.spec.ts:42 → API returns 402 PLAN_LIMIT when free user tries 6th link via API (0.8s)

  3 passed (3.6s)
```

Result: **PASS** — UI gating + API gating both verified.

### Criterion 4: Search filters client-side, no API calls

Command: manual verification with browser devtools (no automated test for this — too brittle to assert "no network request happened during a keystroke," but verifiable by inspection).

Output (devtools Network tab transcript during the search interaction):
```
14:48:02  GET  /links               200  3.1 KB   (initial list load)
14:48:14  [user types "newsletter" in search box, 10 keystrokes]
14:48:14  (no requests)
14:48:14  (no requests)
... (8 more keystrokes, no requests)
14:48:18  [list updates to show only matching links]
```

Result: **PASS** — verified by inspection. Manual check; called out per `tstack-build` SKILL.md's "manual UI flow" guidance.

### Criterion 5: Archived links return 410 Gone via redirect Worker

Command: archive a known link, then `curl` the redirect Worker.

Output:
```
$ curl -X DELETE http://localhost:3000/api/links/l-9k2 \
    -H "Authorization: Bearer $TOKEN_RILEY"

HTTP/1.1 200 OK
{"data":{"id":"l-9k2","archived":true}}

$ curl -i https://slink-redirect-preview.workers.dev/issue-47 \
    -H "Host: links.riley.fm"

HTTP/1.1 410 Gone
content-type: text/html
cache-control: public, max-age=60

<!DOCTYPE html><html><head><title>Link archived — Slink</title>...
```

Result: **PASS** — archive flips the flag; redirect returns 410 (not 404) so analytics can distinguish "never existed" from "removed."

### Criterion 6: a11y — list view keyboard-navigable; axe passes

Command: `pnpm playwright test apps/web/tests/a11y/links-page.spec.ts`

Output:
```
Running 2 tests using 1 worker

  ✓ links-page.spec.ts:6 → links page passes axe-core checks (1.8s)
  ✓ links-page.spec.ts:18 → links list is fully keyboard-navigable: tab through, enter to edit, escape to cancel (2.1s)

  2 passed (4.0s)
```

axe results detail (from the first test's logged output):
```
Violations: 0
Passes: 47
Incomplete: 1 (color-contrast — passes manual review; axe flags one tooltip that meets AA at 4.6:1)
```

Result: **PASS** — zero violations; the one "incomplete" is a known axe false-positive on a tooltip with verified 4.6:1 contrast (AA threshold is 4.5:1).

---

## Summary

| Criterion | Result |
|---|---|
| 1. CRUD + Zod + per-user WHERE | PASS |
| 2. Slug uniqueness, 409 with code | PASS |
| 3. Free-tier link limit | PASS |
| 4. Client-side search, no API calls | PASS |
| 5. Archive → 410 Gone | PASS |
| 6. a11y / axe / keyboard | PASS |

**All 6 criteria PASS. Ready to merge M3.**

(Then `tstack-build` proceeds to step 3: commit final state, push branch, merge to main, update ROADMAP.md Status section.)
