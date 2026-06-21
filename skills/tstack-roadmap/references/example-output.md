# Example output: `docs/ROADMAP.md` (excerpt)

Excerpt from the **Slink** roadmap. Shows the mandatory `M0 — Infrastructure baseline`, the dependency graph format, and the milestone-entry shape. Real roadmap for Slink v1 has ~12 milestones; this excerpt shows M0–M3 plus the Status section.

---

# Slink — Phase 1 Roadmap

## Overview

12 milestones across one workstream (web). Estimated 8–10 calendar weeks for solo dev at 4–6 focused hours per day. Critical path runs M0 → M1 → M2 → M3 → M5 → M9 (launch-readiness). M4 (AI slugs), M6 (analytics dashboard polish), and M11 (CSV export) can parallelize against M5 once their dependencies clear.

## How to Use

Check the **Status section at the bottom** for the current milestone. Read the listed specs for that milestone, run `tstack-plan` to produce an approved plan, then `tstack-build` to ship.

Milestone IDs use `M` prefix for the web workstream. There's no separate mobile workstream in v1.

## Dependency Graph

```
M0 — Infrastructure baseline
   │
   ├──► M1 — Database schema + Auth integration
   │          │
   │          ├──► M2 — Domain management + DNS verification
   │          │          │
   │          │          └──► M3 — Link CRUD (manual slugs)
   │          │                     │
   │          │                     ├──► M4 — AI slug suggestions
   │          │                     │
   │          │                     └──► M5 — Redirect Worker + click events
   │          │                                │
   │          │                                └──► M6 — Analytics dashboard
   │          │
   │          └──► M7 — Stripe + plan-tier enforcement
   │
   └──► M8 — Sentry / PostHog wiring (parallel with M1)
              │
              └──► M9 — GDPR account-deletion flow
                         │
                         └──► M10 — Soft-launch (beta cohort)
                                    │
                                    └──► M11 — CSV export
                                               │
                                               └──► M12 — Public launch (Product Hunt)
```

---

## M0 — Infrastructure baseline

**What gets built:** The non-negotiable platform plumbing that every later milestone depends on. CI workflow, branch protection, secrets management, deployment skeleton, observability bootstrap, lint/format, and test runner.

**Dependencies:** None — starting point.

**Read before starting:**
- `docs/ARCHITECTURE.md` — § Tech Stack, § Repo Structure, § Deployment Topology
- `docs/DECISIONS.md` — ADR-1, ADR-2 (security & observability postures drive M0's specifics)
- `docs/CONVENTIONS.md` — § Lint/format rules
- `docs/TESTING.md` — § CI matrix, § Test runner configuration

**Done when:**

*Implementation satisfies (provable by the build/test/lint/CI run):*
- Pushing a PR triggers GitHub Actions; CI runs typecheck (`tsc --noEmit`), lint (`pnpm lint`), unit tests (`pnpm test`), and a Playwright smoke test. The CI status appears on the PR.
- Vercel preview deploys for `web` succeed on every PR.
- Wrangler deploys `redirect` to a `*.workers.dev` preview from CI.
- A deliberate `throw new Error("M0 sentry test")` in a test route surfaces in Sentry within 60s.
- `pnpm test` runs one trivial passing unit test; `pnpm lint` passes; `pnpm typecheck` passes; `gitleaks` runs in CI and finds no secrets.

*Owner configures externally (GitHub/Vercel/Cloudflare consoles — attested, not command-checked):*
- `main` branch protection requires the `ci` check to pass and disallows force-push.
- Secrets are set in the deploy platforms (Vercel env vars for app, Wrangler secrets for Worker, Neon connection string for DB) — none in repo.

**Estimated effort:** 1–2 focused days.

---

## M1 — Database schema + Auth integration

**What gets built:** Postgres schema for `User`, `Domain`, `Link` (per PRODUCT.md § Data Models). Drizzle migrations runnable from CI. Clerk integrated; `/` is a marketing page; `/app/*` requires auth and provisions a `User` row on first login.

**Dependencies:** M0.

**Read before starting:**
- `docs/PRODUCT.md` — § Data Models
- `docs/2 - Specs/database-schema.md` — full file
- `docs/ARCHITECTURE.md` — § Auth (Clerk integration shape)
- `docs/DECISIONS.md` — ADR-1, ADR-9, ADR-12

**Done when:**
- `pnpm db:migrate` runs cleanly against a fresh Neon dev branch.
- All three tables exist with the columns, constraints, indexes, and RLS policies in the schema spec — verifiable via `\d+ users`, `\d+ domains`, `\d+ links` in `psql`.
- Signing up via Clerk creates a `User` row with the Clerk `user_id`. Verifiable: sign up, query Postgres, see the row.
- `/app/*` redirects unauthenticated users to Clerk sign-in.
- Cross-user isolation test: two users; user A's query returns only A's rows; demonstrated in `tests/integration/auth-isolation.spec.ts` (passing).
- a11y: Clerk sign-in / sign-up screens are AA-compliant (axe-core passes).

**Estimated effort:** 1–2 days.

---

## M2 — Domain management + DNS verification

**What gets built:** UI to add a custom domain. Inngest job polls DNS for CNAME match every 30s up to 10 min. Plan-tier limit enforcement (Free=1, Pro=3, Team=10).

**Dependencies:** M1.

**Read before starting:**
- `docs/PRODUCT.md` — § F-1 (Account Creation & Branded Domain Setup)
- `docs/API.md` — `POST /api/domains`, `POST /api/domains/:id/verify`
- `docs/ARCHITECTURE.md` — § Background jobs (Inngest patterns)
- `docs/2 - Specs/dns-verification.md` — full file
- `docs/DECISIONS.md` — ADR-13 (Inngest)

**Done when:**
- `POST /api/domains` with `{ host: "links.riley.fm" }` creates an unverified `Domain` row; UI shows the required CNAME record.
- `POST /api/domains/:id/verify` schedules an Inngest job that resolves the CNAME and updates `verified_at`. Demonstrated: add a real test domain pointing to the correct target, click verify, see verified within 60s.
- DNS mismatch shows expected vs. actual in the UI.
- Free-tier user with 1 domain attempting to add a 2nd: blocked with upgrade CTA.
- a11y: verification status uses `aria-live="polite"`; axe passes.

**Estimated effort:** 2 days.

---

## M3 — Link CRUD (manual slugs)

**What gets built:** Dashboard list view, create form (manual slug only — AI deferred to M4), edit destination, archive, search.

**Dependencies:** M2.

**Read before starting:**
- `docs/PRODUCT.md` — § F-2 (Link Creation), § Data Models / `Link`
- `docs/API.md` — `GET/POST/PATCH /api/links`
- `docs/CONVENTIONS.md` — § Server Actions, § Form validation
- `docs/TESTING.md` — § Integration test patterns

**Done when:**
- All `Link` CRUD endpoints exist with Zod validation and per-user `WHERE` clauses (defense-in-depth: RLS confirmed in Postgres logs).
- Slug uniqueness enforced per `domain_id`; duplicate attempt returns 409 with `{ error: "slug taken", code: "SLUG_CONFLICT" }`.
- Free-tier link limit (5/mo) enforced at the API; UI shows progress + upgrade CTA at 4/5.
- Search filters the list client-side (no API call) for the user's own links — demonstrated by network tab.
- Soft-delete: archived links return 410 Gone via redirect; admin UI to unarchive within 90 days.
- a11y: list view is keyboard-navigable; create form labeled correctly; axe passes.

**Estimated effort:** 1–2 days.

(milestones M4–M12 omitted from this excerpt)

## Parallelization Notes

- **M8 can start in parallel with M1** (different code paths — observability wiring is independent of schema).
- **M4 (AI slugs) and M5 (Redirect Worker) can run in parallel** once M3 is shipped — different runtimes, no shared code beyond `packages/shared`.
- **M11 (CSV export) can start any time after M6.**

Critical path (longest dependency chain to launch): **M0 → M1 → M2 → M3 → M5 → M6 → M9 → M10 → M12.** Estimated 14–18 focused days end-to-end if executed serially; closer to 10–12 with parallel execution.

---

## Status

**Completed:**
(none yet)

**Up next:** M0 — Infrastructure baseline

---

## Aside: what an `i0` looks like (not part of Slink — iOS reference only)

Slink is web-only, so its baseline is `M0`. If a project targets iOS, the mandatory baseline is emitted as `i0 — iOS infrastructure baseline` instead (see SKILL.md § iOS). Shown here for reference — note the same implementation-vs-external split, with Apple's console settings on the external side:

```markdown
## i0 — iOS infrastructure baseline

**What gets built:** Stand up the iOS baseline: an archivable SwiftUI app target that launches,
CI running build + test + lint on every PR, signing/secrets approach, and crash reporting.

**Dependencies:** None — starting point. (For a web+iOS project, depends on the shared M0.)

**Read before starting:**
- `docs/ARCHITECTURE.md` — § Repo Structure (the `apple/` folder), § Tech Stack
- `docs/DECISIONS.md` — ADR-1, ADR-2 (security & observability postures)
- `docs/CONVENTIONS.md` — § Swift conventions
- `docs/TESTING.md` — § Test runner, § CI

**Done when:**

*Implementation satisfies (provable by the build/test/lint/CI run):*
- Pushing a PR triggers CI; it runs `xcodebuild build`, `xcodebuild test` (or `swift test`), and
  SwiftLint/SwiftFormat. CI status appears on the PR.
- The app target builds and archives cleanly (`xcodebuild -scheme App archive` succeeds).
- One trivial XCTest/Swift Testing test passes in CI.
- A deliberate test crash surfaces in the chosen crash reporter (e.g. Sentry/Crashlytics).

*Owner configures externally (Apple/GitHub consoles — attested, not command-checked):*
- Signing certificates + provisioning profiles set up in the Apple Developer account / Xcode.
- App Store Connect record created; a build distributes to TestFlight.
- `main` branch protection requires the CI check and disallows force-push.

**Estimated effort:** 1–2 focused days.
```
