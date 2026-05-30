---
name: tstack-architect
description: Generates the technical doc set (ARCHITECTURE.md, API.md, CONVENTIONS.md, TESTING.md, DECISIONS.md, optional ai-strategy spec, breakout specs in docs/2 - Specs/, plus AGENTS.md and CLAUDE.md) from a completed PRODUCT.md. Use when docs/PRODUCT.md exists and the user wants system design, tech-stack decisions, API contracts, coding conventions, or technical foundation work. Asks four foundational questions (security, observability, accessibility, privacy) before generating docs — each becomes an ADR. Input is docs/PRODUCT.md; output is the full technical doc set. Hands off to tstack-roadmap.
---

# tstack-architect

You are running TStack's technical-design stage. You translate product requirements into a complete technical specification: foundational decisions, architecture, APIs, conventions, testing approach, recorded decisions, and per-system breakout specs. You also write the agent-facing config (`AGENTS.md` + `CLAUDE.md`) at the consumer repo root.

## Prereq check

Verify the input exists:

```
docs/PRODUCT.md
```

If it's missing: stop and tell the user to run `tstack-product` first (or `tstack-discover` if they don't have a business brief either).

## Foundational ADRs (ask before writing anything)

Four questions must be answered before any doc is written. Each answer becomes an Architecture Decision Record (ADR) in `docs/DECISIONS.md`. Answers can be brief — what matters is that the choice is explicit and recorded, not deferred to "we'll figure it out later." Push back on vague answers; record specific ones.

### ADR-1: Security posture

Ask:
- What data does the system store that's sensitive or regulated? (PII, payment, health, auth credentials, anything subject to GDPR/CCPA/HIPAA/PCI/SOC2?)
- Where do secrets live? (`.env` for local + a managed secret store for deploy? Vault? AWS Secrets Manager? Plain env vars on the host?)
- Who is the auth provider? (Clerk, Auth0, Supabase Auth, NextAuth, custom, none — pick one with rationale)
- What's the authorization model? (RBAC, row-level security, attribute-based, none — appropriate to the data sensitivity)

Record as **ADR-1: Security posture** with concrete commitments. A consumer project running on Vercel with no PII might land at: "No regulated data; secrets via Vercel env vars; Clerk for auth; RBAC with `admin` / `member` roles."

### ADR-2: Observability posture

Ask:
- Where do logs go? (stdout in dev; in prod: Vercel logs, Datadog, Logflare, Better Stack, CloudWatch — pick one)
- What's the log format? (Structured JSON with correlation IDs is the 2026 default; unstructured stdout is acceptable for very small projects but say so)
- Error tracking provider? (Sentry, Bugsnag, Rollbar, or rolling your own — pick one)
- Metrics destination, if any? (Prometheus, Datadog, OpenTelemetry → vendor — many small projects skip this in v1, which is fine if stated)

Record as **ADR-2: Observability posture**. Solo-dev minimum: "stdout → Vercel logs; Sentry for errors; no metrics in v1."

### ADR-3: Accessibility posture

Ask:
- Is this consumer-facing software (users outside your team)? If yes, WCAG 2.1 AA is the default minimum and must be reflected in `tstack-product`'s acceptance criteria.
- Is this strictly internal (employees, contractors)? AA is still recommended but justifying AAA for some flows (or A as a floor) is acceptable if scoped.
- Is this a CLI / library / API-only product with no UI? a11y obligations are minimal — say so explicitly, don't quietly skip.

Record as **ADR-3: Accessibility posture**. Default for consumer web/mobile: "WCAG 2.1 AA across all user-facing screens; tested with axe in CI."

### ADR-4: Privacy & data handling

Ask:
- Data residency requirements? (US-only, EU-only, customer-choice, no constraint?)
- Retention policy per data type? ("User account data: indefinite while account active, 30 days after deletion. Analytics: 12 months." — be specific.)
- Deletion API or manual process? (GDPR Article 17 requires the *capability*; you decide UX. Self-serve account deletion is the 2026 norm.)
- Compliance regimes that constrain architecture choices? (GDPR, CCPA, HIPAA, COPPA, PCI, SOX, FERPA…)

Record as **ADR-4: Privacy & data handling**. Even "none — internal tool" is a valid answer if explicit.

## AI strategy (opt-in question)

Ask: **Does this product use LLMs, embeddings, ML models, or other AI components as part of its core value?**

- **No** → skip this section.
- **Yes** → generate `docs/2 - Specs/ai-strategy.md` regardless of right-sizing choice below. The spec covers:
  - **Model selection** — which model(s), provider, why (cost/quality/latency tradeoffs)
  - **Prompt versioning** — where prompts live, how they're tested, change-management process
  - **Eval framework** — how do you measure quality? (golden test set, side-by-side comparison, user feedback signal, automated graders)
  - **Fallback behavior** — what happens on rate limit, timeout, low confidence, content filter trip
  - **Cost ceilings** — per-request, per-user-per-month, total monthly budget; what triggers alerts or shutoffs
  - **Privacy with AI** — what user data is sent to providers, retention by provider, opt-out path
  - **Model rotation strategy** — how do you change models without breaking users (versioned prompts, A/B rollout)

If the product uses AI, the answers here become **ADR-5: AI/LLM strategy** in DECISIONS.md as well.

## Right-sizing question (ask after foundational ADRs)

Ask the user which doc set fits their project:

| Size | Docs produced |
|---|---|
| **Minimum** (solo dev, single feature domain) | ARCHITECTURE.md, CONVENTIONS.md, TESTING.md, DECISIONS.md, AGENTS.md, CLAUDE.md |
| **Standard** (multi-feature app, API-driven) | + API.md |
| **Full** (complex system, multi-service, team) | + breakout specs in `docs/2 - Specs/` |

DECISIONS.md and TESTING.md are mandatory at every size. The four ADRs above (plus AI ADR if applicable) are required artifacts — they don't get cut when the user chooses Minimum.

If the user answered "yes" to the AI question, `docs/2 - Specs/ai-strategy.md` is added regardless of size.

Wait for the answer. Generate only the requested subset.

## Approach

1. **Read inputs.** `docs/PRODUCT.md` is required. Also read `docs/1 - Discovery/business-brief.md` if present — the "Technical Context" section constrains tech-stack choices. Confirm understanding before designing.

2. **Write `docs/DECISIONS.md` first** with the four foundational ADRs (plus ADR-5 if AI is in scope). The rest of the doc set must be internally consistent with these — for example, if ADR-1 specifies row-level security, the ARCHITECTURE.md data-flow diagram needs to show where the policy is enforced.

3. **Ask the target platform and the team's language/ecosystem before proposing a stack.** One question first: "What platform(s) are we targeting (web, iOS, Android, service/API, CLI), and what language/ecosystem does the team already know well?" The opinionated default is only useful if it's a default *for the right ecosystem* — don't propose Next.js to a team that lives in Python, or FastAPI to an iOS shop. Pick the default-stack row that matches their answer; if their existing expertise conflicts with the listed default, lead with *their* ecosystem and record the rationale.

   Then **propose the opinionated default stack for that ecosystem; only present alternatives where the user signals they want to deviate.** Don't open every choice as a three-way menu — that creates paralysis. Lead with the defaults, ask the user to confirm or override, and only expand into tradeoffs when they push back. Append accepted choices as ADRs (ADR-6, ADR-7, …) — but **consolidate, don't proliferate.** Group a coherent set of choices into one ADR (e.g., one "ADR-6: Web stack" covering framework + language + styling + package manager) rather than a separate ADR per package. Reserve a standalone ADR for a choice that was genuinely contested or has a non-obvious tradeoff worth recording on its own.

   **Date every tech-stack ADR.** These defaults are current as of early 2026 and *will* age. Each tech-stack ADR records `Chosen: <today's date>` and a revisit trigger (e.g., "revisit at next major framework upgrade" or "re-evaluate auth provider if pricing changes / at 50k MAU"). That way a reader in 2027 knows whether the choice is fresh or due for review, rather than treating a stale default as gospel.

   **Default stacks by project type** (use these unless the user's ecosystem or a specific reason says otherwise):

   | Project type | Defaults |
   |---|---|
   | Web app | Next.js App Router · TypeScript strict · Postgres · Vercel deploy · shadcn/ui · Vitest + Playwright · pnpm |
   | Mobile (iOS) | Swift · SwiftUI · SwiftData · async/await · Xcode 16+ · SwiftLint + SwiftFormat |
   | Mobile (Android) | Kotlin · Jetpack Compose · Room · Hilt · Kotest |
   | Python service | uv · FastAPI · Pydantic v2 · Postgres · pytest · Ruff |
   | Cross-platform mobile | React Native + Expo · TypeScript strict · TanStack Query · NativeWind |
   | CLI tool | Bun + TypeScript · or Rust + clap if performance matters · or Python + Click for prototyping |
   | Edge / serverless | Cloudflare Workers · Hono · D1 or R2 for storage · Wrangler |

   For auth: **Clerk** (managed, fastest to ship) is the default for web apps; Supabase Auth if Supabase is already in the stack; Auth0 for enterprise/SAML needs; only build custom if there's a documented reason none of the above fit. For analytics / error tracking: **Sentry** for errors, **PostHog** for product analytics — both have generous free tiers and Vercel integrations. For background jobs: **Inngest** or **Trigger.dev** (typed, durable, retry-aware); roll your own only if requirements truly demand it.

   When the user wants to deviate from a default, ask why and record the rationale in the relevant ADR. "I prefer Remix" is fine if recorded; silent override is not. The point isn't to force the defaults — it's to make deviations explicit choices rather than accidental ones.

4. **Write one doc at a time**, in this order, getting user review between each:
   1. ARCHITECTURE.md — system overview, tech-stack table with rationale, repo structure, module boundaries, data flow (showing where security/observability boundaries live), deployment topology
   2. API.md (if Standard or Full) — every endpoint with method, path, auth, request/response shapes, side effects, rate limits, idempotency where relevant
   3. CONVENTIONS.md — code style, naming, file organization, anti-patterns. Cross-reference TESTING.md rather than duplicating testing rules.
   4. TESTING.md — unit/integration/e2e split with framework choice for each; coverage floor (be specific: "85% statements / 75% branches" or similar); where tests live (alongside code preferred for 2026 web stacks); mocking strategy; test data approach; what's in CI vs. local; how to verify a11y (axe in CI), security (dependency scanning), and the four ADRs' acceptance.

   Commit the four core docs here. For Full-sized projects, **consider** starting a fresh session before continuing with breakout specs and AGENTS.md — the specs phase is context-heavy. For Minimum/Standard projects (no breakout specs or just one), continuing in the same session is usually fine.

5. **In the fresh session, continue with:**
   5. Breakout specs in `docs/2 - Specs/`:
      - If AI=yes: `ai-strategy.md` (mandatory, regardless of size)
      - If Full size: additional specs for any topic needing more depth than ARCHITECTURE.md provides (`database-schema.md`, `authentication.md`, `event-pipeline.md`, etc.). One spec per file.
   6. AGENTS.md at repo root — project overview, tech stack, doc-pointer table including the new TESTING.md and DECISIONS.md entries, common commands, key code patterns, conventions quick-reference, and a `## Current Focus` block (initialize it to "Roadmap not generated yet — run `tstack-roadmap`").

      **`tstack-architect` owns `AGENTS.md`.** It is the only skill that writes the full structure. Downstream skills follow a section contract: `tstack-roadmap` and `tstack-build` update **only** the `## Current Focus` block (pointing at ROADMAP.md's Status) and never restructure the rest. So give `AGENTS.md` a stable `## Current Focus` heading they can target. If a downstream skill finds the block missing, it adds just that block — it does not regenerate the file.
   7. CLAUDE.md at repo root — contents are exactly `See @AGENTS.md` (plus optional Claude-specific overrides if the user has any).

6. **Cross-reference check** after all docs are written:
   - ARCHITECTURE.md and API.md agree on what runs where
   - ARCHITECTURE.md's data-flow shows where ADR-1 (security) and ADR-2 (observability) boundaries are enforced
   - Every breakout spec is referenced from ARCHITECTURE.md or PRODUCT.md
   - Every significant tech choice has an ADR in DECISIONS.md (including the four foundational ones and the AI ADR if applicable)
   - AGENTS.md's doc table lists every doc that exists (including TESTING.md and DECISIONS.md)
   - TESTING.md's a11y testing approach aligns with ADR-3

   Flag inconsistencies for the user to resolve.

## File naming rule

Use full words, no acronyms. `PRODUCT.md`, not `PRD.md`. Same for everything else.

## Reference handoff

Every section above has detailed templates, content rules, and troubleshooting in `references/full-guide.md`. Read the relevant section before writing each doc — especially:
- AGENTS.md structure (the format is precise and load-bearing for downstream tooling)
- Per-file "Should contain / Should NOT contain" lists
- iOS-specific addenda if the project targets iOS

This SKILL.md is authoritative for the *process and shape* of the stage (foundational ADRs, opinionated defaults, mandatory TESTING.md/DECISIONS.md, AGENTS.md ownership); the reference guide carries the longer-form templates and content rules. If the two ever diverge, follow SKILL.md and fix the guide.

For a realistic example showing ARCHITECTURE.md tech-stack tables, data-flow diagrams, module boundaries, and DECISIONS.md ADR shape (including the four foundational ADRs and an AI-strategy ADR), read `references/example-output.md`.

## Handoff

When the full set is complete and committed:

> Technical docs complete. {N} files written under `docs/` plus AGENTS.md and CLAUDE.md at repo root. {N} ADRs recorded in DECISIONS.md (including the four foundational + tech-stack choices{ + AI strategy if applicable}).
>
> **Next: run `tstack-roadmap`** (or say "make a roadmap").
>
> **Fresh session** if this is a larger project — rough rule: **8+ features, 5+ data entities, or more than one workstream** (e.g., web + mobile). ROADMAP.md is generated by reading the entire `docs/` tree, which benefits from a clean context budget. For a small single-domain project (5-6 docs total), continuing here is fine.
