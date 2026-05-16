# How to Generate a Build-Order Roadmap

You are creating a `docs/ROADMAP.md` file from existing project documentation. This file sequences every piece of work into milestones ordered by technical dependencies, where each milestone points back to the source docs that define what gets built.

Read this entire file before starting. Then read all project documentation. Then generate the roadmap.

---

## Inputs

Read all of these files before generating the roadmap:

| File | What to Extract |
|---|---|
| **PRODUCT.md** | Every feature, user flow, data model, and acceptance criterion. These become the "what" of each milestone. |
| **ARCHITECTURE.md** | Tech stack, repo structure, module boundaries, data flow. These determine which components depend on which. |
| **API.md** | Every endpoint contract. Milestones that build endpoints must reference these. |
| **CONVENTIONS.md** | Code style rules. Referenced by milestones when they introduce a new pattern domain. |
| **DECISIONS.md** | Architecture Decision Records. Referenced when a milestone touches an architecturally significant choice. |
| **2 - Specs/** | All breakout specs (database schema, encryption, pipelines, sync protocols, etc.). These are the detailed blueprints that milestones point to. |

Also check `1 - Discovery/` for any business context that affects prioritization, but milestones don't directly reference discovery docs.

---

## Generation Process

### 1. Identify Buildable Units

Extract every distinct piece of work from the documentation. A buildable unit:

- Can be implemented and tested independently
- Has clear inputs (dependencies) and outputs (what it enables)
- Takes roughly 1-2 focused days of work

Examples: "Database schema and migrations," "Entity CRUD API with encryption and RLS," "Email ingestion pipeline," "iOS voice capture flow."

### 2. Build the Dependency Graph

For each buildable unit, determine what must exist before it can be built.

**Technical dependencies** (most important):
- Database schema before any CRUD operations
- Authentication before any authenticated endpoints
- Encryption module before any encrypted data writes
- Background job infrastructure before any async pipelines
- API endpoints before client UI that calls them
- Server-side features before mobile features that consume them

**Data dependencies:**
- Can't build matching/reconciliation without at least one data source
- Can't build reports without the data they report on
- Can't build scoring/analytics without the underlying records

**Shared component dependencies:**
- Reusable UI components (shells, layouts) before pages that use them
- Shared libraries (validation schemas, types) early in the sequence

**Independence signals** — things that DON'T depend on each other:
- Features operating on different tables with no FK relationships
- Admin system vs. user-facing system (separate auth, separate routes)
- On-device mobile features (speech, camera, GPS) vs. server features

### 3. Sequence into Milestones

Arrange buildable units into a sequence respecting the dependency graph:

1. **Foundation first.** Monorepo setup → database schema → auth → encryption.
2. **Backend before frontend.** API endpoints before the UI that consumes them.
3. **Server before mobile.** Server milestones establish APIs. Mobile milestones consume them.
4. **Infrastructure before features.** Background job skeleton before pipelines. Storage config before uploads.
5. **Simple CRUD before complex pipelines.** Entity management before reconciliation.
6. **Data producers before data consumers.** Capture creation before reconciliation. Imports before reports.
7. **Core features before polish.** Analytics, notifications, feedback, admin come after core user flows.
8. **Hardening last.** Security audit, cross-user testing, rate limiting as the final milestone.

### 4. Identify Parallelization

Note which milestones can run simultaneously because they share dependencies but don't depend on each other. Common examples:
- Encryption module + auth system
- Multiple independent CRUD endpoints
- Independent mobile capture flows (voice, photo, mileage)
- Admin system vs. user-facing features

---

## Output Format

Structure `docs/ROADMAP.md` exactly as follows:

### Header

A "Phase" corresponds to a major product version (Phase 1 = v1, Phase 2 = v2, etc.). Milestones are the individual build steps within a phase. Most projects start with Phase 1.

```markdown
# Phase {N} Build Roadmap

**Purpose:** Build-order sequence for Phase {N}. Each milestone is scoped to ~1-2 focused days. This file tells you what to build next, what it depends on, and what "done" looks like.

**How to use:** Check the Status section at the bottom for the current milestone. Read the listed specs for that milestone, enter plan mode, and plan the implementation.

**Convention:** Server/web milestones use `M` prefix. If the project has a mobile app, mobile milestones use a separate prefix (e.g., `i` for iOS). Mobile milestones depend on specific server milestones — the API must exist before the client is built.
```

### Dependency Graph Diagram

An ASCII diagram showing the milestone sequence with arrows for dependencies. Group related milestones visually. If the project has multiple workstreams (e.g., server + iOS), show them as parallel tracks with cross-dependency arrows.

### Milestone Entries

Each milestone follows this template. Do NOT include status indicators on individual milestones — status is tracked in a single section at the bottom.

```markdown
## M{N} — {Short Descriptive Name}

**What gets built:** One or two sentences describing the deliverable. Be specific about what
exists when this milestone is complete (endpoints, UI screens, pipeline steps, modules).

**Dependencies:** M{X}, M{Y} — which milestones must be complete first.
If none, say "None — starting point."

**Read before starting:**
- `docs/ARCHITECTURE.md` — {specific section}
- `docs/API.md` — {specific section}
- `docs/2 - Specs/{spec-file}.md` — {full file or specific section}
- `docs/PRODUCT.md` — Section {N} ({section name})
- `docs/DECISIONS.md` — ADR-{N} ({decision name}) [only if relevant]
- `docs/CONVENTIONS.md` — {specific section} [only if milestone introduces a new pattern]

**Done when:**
- {Specific, testable criterion}
- {Another specific criterion}
- {Criteria should be binary pass/fail}
- {Include isolation/security criterion if applicable}
```

### "Read before starting" Rules

- Always include the relevant spec file(s) — the primary implementation reference
- Include PRODUCT.md sections for acceptance criteria and user flow context
- Include API.md sections when the milestone builds endpoints
- Include ARCHITECTURE.md when the milestone establishes a new structural pattern
- Include DECISIONS.md only when the milestone touches an architecturally significant decision
- Include CONVENTIONS.md only for the first milestone of a new domain (first Swift code, first Inngest function, etc.)
- Be specific about which *section* of each file to read — don't say "full file" unless the entire file is genuinely relevant

### "Done when" Rules

- Every criterion must be objectively testable — no subjective language
- Include at least one happy-path criterion ("endpoint returns correct response")
- Include at least one error-path criterion ("invalid input returns 400 with structured error")
- Include at least one isolation criterion when relevant ("user A cannot see user B's data")
- For encrypted fields: include a round-trip criterion ("encrypt → store → read → decrypt returns original")
- For background jobs: include both success and failure criteria
- Don't over-specify — these are completion checks, not exhaustive test cases

### Parallelization Notes

After all milestones, include a section documenting which milestones can run simultaneously and why. Also identify the critical path — the longest dependency chain that determines minimum total build time.

### Status Section

The **last section** of the roadmap tracks progress. This is the only place status is recorded — not on individual milestones. Format:

```markdown
---

## Status

**Completed:**
(none yet)

**Up next:** M0 — {Name}
```

As milestones are completed, they get added to the completed list and "Up next" is updated:

```markdown
---

## Status

**Completed:**
- M0 — Monorepo & Tooling Setup
- M1 — Database Schema & Migrations
- M2 — Auth & Profiles

**Up next:** M3 — Field-Level Encryption
```

This keeps milestone definitions clean and puts all progress tracking in one scannable place.

---

## After Creating the Roadmap

Add a `## Current Focus` section to `AGENTS.md` (or `CLAUDE.md` if no AGENTS.md exists):

```markdown
## Current Focus
Check `docs/ROADMAP.md` — see the Status section at the bottom for the current milestone.
```

---

## Troubleshooting

### Milestones Are Too Large

If a milestone looks like more than 2 focused days of work, it's too big. Ask Claude Code to split it:

```
M4 covers too much — CRUD endpoints, encryption, RLS policies, and tests.
Split it into M4a (CRUD without encryption) and M4b (add encryption + RLS).
Each should be completable in 1-2 days.
```

### Milestones Are Too Granular

If you have 30+ milestones for a Phase 1 product, the roadmap is over-decomposed. Consolidate:

```
These milestones are too granular. Combine related work — M3 (create users table),
M4 (add user CRUD), and M5 (add user validation) should be a single milestone
"M3 — User Management."
```

### Dependency Graph Seems Wrong

If Claude Code has a milestone depending on something that should be independent, challenge it:

```
Why does M7 (notification emails) depend on M5 (file upload)? These seem
independent — emails don't require file upload. Remove the dependency and
note they can run in parallel.
```

### Milestones Reference Docs That Don't Exist

If a "read before starting" list points to a spec or PRODUCT.md section that wasn't created in step 2, either the doc is missing or the reference is wrong:

```
M6 references "docs/2 - Specs/payment-processing.md" but that spec doesn't
exist. Either create it now or update M6 to reference the correct source for
payment processing details.
```

