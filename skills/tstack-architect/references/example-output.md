# Example output: `docs/ARCHITECTURE.md` + `docs/DECISIONS.md` (excerpt)

Excerpts from realistic outputs for the **Slink** link shortener. Shows the structure and specificity expected. Real architecture docs for this scope total ~600–900 lines across all files; this is an illustrative slice.

---

# `docs/ARCHITECTURE.md` (excerpt)

## Architecture Philosophy

Three non-negotiable principles:

1. **Redirect latency is sacred.** Every architectural decision is judged against "does this keep p95 < 50ms globally?" If it doesn't, it lives off the hot path.
2. **Plan tier separation is enforced at the data layer.** Free vs Pro limits are queries, not UI state.
3. **AI features fail open.** If the slug-suggestion service is down, link creation still works — manual entry is always available.

## Tech Stack

| Layer | Choice | Rationale (ADR ref) |
|---|---|---|
| Frontend | Next.js 16 App Router | Server Components keep the dashboard small; Cache Components for analytics rollups | ADR-6 |
| Language | TypeScript strict | Catches model/API drift at compile time | ADR-7 |
| Styling | Tailwind v4 + shadcn/ui | Default; founder has prior fluency | ADR-8 |
| Database | Postgres 16 (Neon) | Transactional links + serverless scaling fits the budget | ADR-9 |
| Edge runtime | Cloudflare Workers | Redirect path; global p95 < 50ms achievable | ADR-10 |
| Click events | Cloudflare D1 (replicated to Postgres nightly) | Hot writes at edge; analytical queries on Postgres | ADR-11 |
| Auth | Clerk | Managed, supports SAML for future Team tier | ADR-12 |
| Error tracking | Sentry | ADR-2 (observability) | |
| Product analytics | PostHog | ADR-2 | |
| Background jobs | Inngest | DNS verification polling, nightly aggregation, GDPR deletion | ADR-13 |
| AI provider | Anthropic Claude (Haiku) for slug suggestions | Cost + latency match the AI strategy spec | ADR-5 |
| Test runner | Vitest (unit) + Playwright (e2e) + axe (a11y) | Per TESTING.md | |
| Deploy | Vercel (app) + Cloudflare (Workers) | App is Server-Component-heavy; redirect needs edge | ADR-14 |

## Repo Structure

```
slink/
├── apps/
│   ├── web/              # Next.js — dashboard, marketing, API routes
│   └── redirect/         # Cloudflare Worker — the redirect hot path
├── packages/
│   ├── db/               # Drizzle schema + migrations (shared)
│   ├── shared/           # Types, validation schemas, link utilities
│   └── eval/             # AI slug eval harness + golden set
├── docs/                 # The TStack doc set
├── evals/
│   └── slug-suggestions-v1.jsonl
└── .github/workflows/    # CI: typecheck, test, lint, deploy preview
```

## Data Flow

```
[ Newsletter reader ]
        │
        │  GET https://links.riley.fm/abc123
        ▼
┌──────────────────────┐
│ Cloudflare Worker    │  ◄── reads from D1 (slug → destination)
│ (redirect service)   │     writes click event to D1 (append-only)
└──────────────────────┘
        │ 302 Location: https://riley.beehiiv.com/...
        ▼
[ Newsletter reader → destination ]


[ Creator dashboard ]
        │
        │  Next.js App Router (apps/web)
        ▼
┌──────────────────────┐      ┌──────────────────┐
│ Server Components    │ ───► │ Postgres (Neon)  │  Users, Domains, Links, daily click rollups
│ + Server Actions     │      └──────────────────┘
└──────────────────────┘      ┌──────────────────┐
        │                     │ Clerk            │  Auth, session
        │                     └──────────────────┘
        │                     ┌──────────────────┐
        │                     │ Claude (Haiku)   │  AI slug suggestions (via fetch in Server Action)
        │                     └──────────────────┘
        ▼
┌──────────────────────┐
│ Inngest              │  DNS poll, nightly aggregation (D1 → Postgres), GDPR purge
└──────────────────────┘
```

**Security boundary (ADR-1):** All authenticated app routes enforce `user_id` in the WHERE clause via Drizzle helpers; row-level policies in Postgres back this up as defense-in-depth. The redirect Worker is unauthenticated by design (anyone can click a public short link) but never returns user-identifying data.

**Observability boundary (ADR-2):** Every Server Action and every Worker invocation emits a structured log with `request_id` (correlation). Sentry catches unhandled errors in both. PostHog tracks dashboard interactions (page views, key actions, conversion events).

## Module Boundaries

- `apps/web` may import from `packages/{db,shared,eval}`. Never the reverse.
- `apps/redirect` (Worker) imports only from `packages/shared` — must stay tiny for cold-start performance. Drizzle is not in this bundle; Workers reads D1 directly via the binding.
- `packages/db` owns schema and migrations. Schema changes follow the migration runbook in `docs/2 - Specs/database-schema.md`.

## Deployment Topology

- **Production:** Vercel for `apps/web` (auto-deploy on main); Cloudflare for `apps/redirect` (Wrangler deploy in CI). Neon Postgres production branch; Cloudflare D1 production database.
- **Preview:** Vercel preview per PR; redirect Worker preview via Wrangler; Neon branch DB per PR (auto-provisioned via the Vercel-Neon integration).
- **Local:** `pnpm dev` runs Next.js + Wrangler dev for the Worker against a local D1; Neon dev branch for Postgres.

---

# `docs/DECISIONS.md` (excerpt — first 5 ADRs)

## ADR-1: Security posture

**Status:** Accepted. 2026-05-15.

**Context:** Slink stores user accounts, custom domain mappings, and click events that include hashed IPs and referrer hosts. EU users are expected — GDPR applies.

**Decision:**
- Secrets via Vercel env vars (production) + Cloudflare Workers secrets (Worker production); `.env.local` for dev.
- Auth: Clerk (Google + email password). No custom credential handling.
- Authorization: RBAC at app layer (`free`, `pro`, `team` plans gate features); ownership at row layer (`user_id` WHERE clauses + Postgres RLS as defense-in-depth).
- No PII at the edge: redirect Worker logs hashed IPs only, never raw.
- Dependency scanning: GitHub Dependabot + `pnpm audit` in CI.

**Consequences:** Adds Clerk as a critical-path vendor (acceptable; pricing model fits). RLS adds a Postgres complexity layer (mitigated by helper functions documented in CONVENTIONS.md).

**Alternatives considered:** NextAuth (rejected — more glue code for less feature delta); custom auth (rejected — not core competence).

## ADR-2: Observability posture

**Status:** Accepted. 2026-05-15.

**Decision:**
- Logs: structured JSON with `request_id` correlation. Vercel logs for `apps/web`; Cloudflare logs for `apps/redirect`. Both pipe to Better Stack via log drains.
- Errors: Sentry SDK in both apps; sourcemaps uploaded in CI.
- Product analytics: PostHog (cloud).
- No metrics platform in v1 — Cloudflare Analytics + Vercel Analytics dashboards cover what v1 needs. Revisit at >5k MAU.

## ADR-3: Accessibility posture

**Status:** Accepted. 2026-05-15.

**Decision:** WCAG 2.1 AA across all user-facing dashboard screens. Marketing pages aim for AA. Branded 404 page for non-existent short links: AA. axe-core run in CI on every PR against key dashboard routes; failures block merge.

## ADR-4: Privacy & data handling

**Status:** Accepted. 2026-05-15.

**Decision:**
- Data residency: app data in US (Neon's `us-east`); click event hot writes globally distributed via D1, with EU-region D1 if EU traffic exceeds 25% of requests (revisit at month 3).
- Retention: User account data — indefinite while account active; deleted within 30 days of account deletion. Click events — full detail for 30 days, then aggregated to daily rollups, raw events purged. Server logs — 30 days.
- Deletion API: self-serve account delete in the dashboard. Triggers Inngest job that cascades to Postgres + D1 + Clerk + Sentry within 30 days; user receives confirmation email.
- Compliance: GDPR (cookie consent banner from day 1; right to deletion via the API above; data export available as CSV).

## ADR-5: AI/LLM strategy

**Status:** Accepted. 2026-05-15.

**Decision:** Use Claude (Haiku tier) for slug suggestions. Prompts versioned in `apps/web/lib/ai/prompts/`. Eval set at `evals/slug-suggestions-v1.jsonl`; CI runs evals on prompt changes and blocks merge if quality drops > 5% from baseline. Per-user free-tier cap of 5 AI suggestions/month; Pro = unlimited (soft-capped at 500/mo with alert at 80%). Total monthly LLM budget cap at $200 for v1; PagerDuty alert at 75%. On any error/timeout/filter: fall back to manual slug input with placeholder text. Privacy: only the page title + meta description is sent to the model, never the user's email or account info. Anthropic's standard data retention applies (no fine-tuning on customer data).

(remaining ADRs ADR-6 through ADR-14 — tech-stack choices — omitted from this excerpt)
