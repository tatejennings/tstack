# Example output: `tstack-refactor` (excerpt)

A worked refactor on the **Slink** project (the link-shortener used across the example docs). The user wants to **replace Inngest with Cloudflare Queues** for the DNS-verification polling job. This excerpt shows the justification check, the product-goal lens, the blast-radius map, the two-list doc-impact proposal centered on an **ADR supersession**, and one **behavior-preserving** milestone. (Slink's `docs/DECISIONS.md` records `ADR-13: Inngest for background jobs`.)

---

## 1. Justification (agreed after pushback)

> **You:** Why move off Inngest? "It's another vendor" alone isn't enough to justify the churn.
> **User:** The DNS-verify job is the only Inngest use, we're already on Cloudflare for the redirect Worker, and Inngest's free tier caps us at a run volume we'll hit at ~2k domains. Consolidating onto Cloudflare Queues removes a vendor *and* the ceiling.

Concrete pain (single-use dependency + a real volume ceiling + platform consolidation) → **justified.** This becomes ADR-14's rationale.

## 2. Product-goal lens

Slink's goal (PRODUCT.md) is "branded short links with reliable custom-domain setup." DNS verification must keep working *identically* — same 60s-ish verification, same UI states. **Behavior-preserving; product goal unchanged → PRODUCT.md is NOT touched.**

## 3. Blast radius (mapped from the code, via Explore agents)

- `apps/web/src/jobs/verify-dns.ts` — the Inngest function (the core).
- `apps/web/src/app/api/domains/[id]/verify/route.ts` — enqueues the job (1 call site).
- `apps/web/src/lib/inngest.ts` — client setup; **only consumer** of the `inngest` dep.
- `wrangler.toml` / CI — needs a Queue binding + consumer Worker (new infra).
- Tests: `tests/integration/dns-verify.spec.ts` — the behavior contract to keep green.
- **No** data/schema impact; **no** public API/contract change (the `verify` endpoint signature is unchanged).

## 4. Behavior contract & migration

- **Stays identical:** `POST /api/domains/:id/verify` response shape; verification completes within ~60s; UI `aria-live` states unchanged.
- **Migration shape:** incremental — stand up the Queue consumer alongside Inngest, cut the enqueue over behind a flag, then delete Inngest. Rollback = flip the flag back.

## 7. Doc-impact proposal

```
Proposed updates for refactor "DNS-verify: Inngest → Cloudflare Queues":

Changes I want to make:
- [ ] DECISIONS.md — new ADR-14 (Cloudflare Queues for background jobs), supersedes ADR-13; flip ADR-13 to "Superseded by ADR-14". Revisit when: a job needs fan-out/retries Queues can't express.
- [ ] ARCHITECTURE.md — § Background jobs: replace the Inngest description with the Queue producer/consumer topology.
- [ ] CONVENTIONS.md — § Background jobs: the "enqueue from a route, process in a consumer Worker" pattern replaces the Inngest-function pattern.
- [ ] ROADMAP.md — append M13 (behavior-preserving migration) depending on M2.

Docs I considered but propose NOT to change (challenge any of these):
- PRODUCT.md — behavior-preserving refactor; DNS verification behaves identically; product goal unchanged.
- API.md — the /verify endpoint contract is unchanged.
- 2 - Specs/dns-verification.md — the verification *logic* is unchanged; only the runner moves. (Flag me if you want the runner named here.)
- 8 other specs (auth, encryption, schema, …) — unrelated to this refactor.

Reply: approve / modify / reject per item, or "approve all".
```

## 9. Appended milestone (behavior-preserving)

```
## M13 — Migrate DNS-verify job to Cloudflare Queues

**What gets built:** Move the DNS-verification polling off Inngest onto a Cloudflare Queue
(producer in the web app, consumer Worker), behind a cutover flag, then remove Inngest.

**Dependencies:** M2 (DNS verification must exist to migrate).

**Read before starting:**
- `docs/DECISIONS.md` — ADR-14 (and superseded ADR-13)
- `docs/ARCHITECTURE.md` — § Background jobs (updated topology)
- `docs/2 - Specs/dns-verification.md` — the unchanged verification logic

**Done when:**
*No regression (the behavior contract holds):*
- `tests/integration/dns-verify.spec.ts` passes unchanged: add a real test domain, click verify, see `verified` within 60s.
- `POST /api/domains/:id/verify` returns the same shape as before (contract test green).
- DNS mismatch still surfaces expected-vs-actual in the UI.

*Structural goal met:*
- `grep -r "inngest" apps/web/src` returns nothing; `inngest` removed from `package.json`.
- `wrangler.toml` declares the Queue + consumer; CI deploys the consumer Worker.
- The cutover flag is removed after the Queue path is confirmed in preview.

**Estimated effort:** 1 day.
```

> Refactor specified. Decision: **ADR-14 supersedes ADR-13.** New milestone: **M13**.
> **Next: run `tstack-plan-milestone`** when ready to implement. **Stop here — the refactor isn't started now; it's built later through the normal plan → build cycle.**
