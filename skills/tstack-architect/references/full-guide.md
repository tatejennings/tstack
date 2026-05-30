# Technical Documentation Guide

A complete reference for creating the technical documentation that Claude Code needs to build your project. Give this file to Claude Code along with your completed PRODUCT.md — Claude will work through each document in order, producing the full documentation set before any code is written.

> **Source of truth:** `SKILL.md` is authoritative for the *process and shape* of this stage — the four foundational ADRs (security, observability, accessibility, privacy), the optional AI ADR, the opinionated-default tech-stack approach, mandatory TESTING.md and DECISIONS.md, and AGENTS.md ownership. This guide provides the longer-form **templates, per-file content rules, and troubleshooting**. Where the two ever disagree, follow SKILL.md and treat the discrepancy as a guide bug to fix.

---

## Overview

PRODUCT.md defines *what* gets built. This step defines *how* — the system architecture, API contracts, coding standards, architecture decisions, and detailed specs. These documents serve two audiences: **you** (the developer making decisions) and **Claude Code** (the agent implementing them).

Start a **fresh Claude Code session** for this step. Claude Code reads your finalized PRODUCT.md from disk, which produces better technical docs than continuing from the session that created it.

---

## Claude's Approach

Claude follows this process in order:

1. **Read PRODUCT.md and the business brief.** Confirm understanding of the product, its features, data models, and technical constraints (from the Technical Context section of the business brief). Flag any gaps that would block technical decisions.

2. **Work through each document in the Step-by-Step order** (ARCHITECTURE.md first, CLAUDE.md last). Complete one document at a time. Present each document for user review before moving to the next.

3. **Lead with the opinionated default, not a menu.** Per SKILL.md, ask the target platform and the team's ecosystem first, then propose the opinionated default stack for that ecosystem and ask the user to confirm or override. Only expand a choice into a 2-3 option tradeoff comparison when the user signals they want to deviate — don't open every decision as a three-way menu (that creates paralysis). Either way, record the accepted choice (default-confirmed or overridden) in DECISIONS.md with its rationale and a `Chosen: <date>` stamp; silent override is the only thing that's not allowed.

4. **Manage session context proactively.** For complex projects, the full doc set may exceed a single session's effective context. Because DECISIONS.md is written *first* (the foundational ADRs gate everything else), a natural breakpoint is after the four core docs (ARCHITECTURE/API/CONVENTIONS/TESTING) — start a fresh session for breakout specs and AGENTS.md, pointing the new session at the docs created so far.

5. **After all documents are created, cross-reference check.** Does ARCHITECTURE.md align with API.md? Do breakout specs cover all PRODUCT.md features? Does DECISIONS.md capture every significant technical choice? Flag inconsistencies for the user to resolve.

---

## Documentation Structure

### The `/docs` Folder

```
project-root/
├── AGENTS.md                    # Agent instructions — single source of truth for all AI tools
├── CLAUDE.md                    # Claude Code pointer → "See @AGENTS.md"
├── docs/
│   ├── PRODUCT.md               # Product requirements (what & why) — already created
│   ├── ARCHITECTURE.md          # System overview, tech stack, data flow
│   ├── API.md                   # API contracts, endpoints, request/response shapes
│   ├── CONVENTIONS.md           # Code style, naming, file organization rules
│   ├── TESTING.md               # Test strategy, coverage floor, a11y/security checks
│   ├── DECISIONS.md             # Architecture Decision Records (ADRs)
│   ├── ROADMAP.md               # Build-order milestones with dependencies
│   ├── 1 - Discovery/           # Business brief, market research, competitive analysis
│   └── 2 - Specs/               # Breakout specs for detailed topics
│       ├── database-schema.md
│       ├── authentication.md
│       └── ...
```

### What Each File Does

| File | What It Contains | When Claude Code Reads It |
|---|---|---|
| **AGENTS.md** | Project overview, tech stack, doc pointers, common commands, key code patterns, conventions quick-reference — the single source of truth for all AI coding agents | Every session (referenced by CLAUDE.md) |
| **CLAUDE.md** | Points to AGENTS.md. Can include Claude-specific overrides if needed, but generally just a pointer. | Every session (auto-loaded from repo root) |
| **PRODUCT.md** | User stories, features, data models, acceptance criteria, edge cases, UI flows | Before implementing any feature |
| **ARCHITECTURE.md** | Component relationships, tech stack rationale, repo structure, module boundaries, data flow diagrams, deployment topology | Starting new features, understanding system design, refactoring |
| **API.md** | Every endpoint with method, path, request body, response shape, auth requirements, rate limits | Building any endpoint or client that calls one |
| **CONVENTIONS.md** | Code patterns, naming rules, file organization, anti-patterns to avoid (cross-references TESTING.md rather than duplicating it) | Any code generation task |
| **TESTING.md** | Unit/integration/e2e split + frameworks, coverage floor, test location, mocking, what's in CI vs local, how a11y/security/the four ADRs are verified | Writing tests; defining a milestone's "Done when" verification |
| **DECISIONS.md** | Past architectural choices — what was decided, what was rejected, why. Always opens with the four foundational ADRs (+ AI ADR if applicable) | When Claude suggests something you've already evaluated |
| **ROADMAP.md** | Sequenced milestones with dependencies, spec references, and "done when" criteria | Before starting any milestone; to understand what's next |
| **1 - Discovery/** | Business context, market research, competitive analysis | Rarely by Claude Code; primarily for human reference |
| **2 - Specs/** | Detailed breakout specs for topics that need more depth than the high-level docs provide | Before implementing the specific system that spec covers |

---

## File Specifications

### AGENTS.md (Repo Root)

**Location:** Repo root, alongside CLAUDE.md.

**Purpose:** The single source of truth for all AI coding agents (Claude Code, Codex, Cursor, Windsurf, Xcode Agent, etc.). Every agent-readable instruction lives here. Tool-specific files (CLAUDE.md, .cursorrules, etc.) simply point to this file.

**Structure:**

```markdown
# AGENTS.md — {Project Name}

## Project Overview
Brief description (2-3 sentences) of what the project does.

## Tech Stack
- Language/framework with versions
- Database
- Key dependencies and services
- Build tools

## Documentation
| Doc | When to Read |
|---|---|
| `docs/PRODUCT.md` | Before implementing any feature — contains requirements, user flows, acceptance criteria |
| `docs/ARCHITECTURE.md` | When starting a new feature, refactoring, or needing to understand system design |
| `docs/API.md` | When building any endpoint or client that calls one |
| `docs/CONVENTIONS.md` | Before writing any code — code style, naming, file organization |
| `docs/TESTING.md` | Before writing tests or verifying a milestone — test strategy, coverage floor, a11y/security checks |
| `docs/DECISIONS.md` | Before suggesting alternatives to existing patterns — check if it's already been evaluated |
| `docs/ROADMAP.md` | Before starting any milestone — build order, dependencies, "done when" criteria |
| `docs/2 - Specs/*` | Before implementing the specific system that spec covers — detailed blueprints |

## Architecture Principles
The 3-5 non-negotiable rules for this project (e.g., "database enforces access control",
"server-side only for sensitive operations", "entity isolation is absolute").

## Common Commands
Build, test, lint, dev server, database reset, type generation — whatever a developer
runs daily.

## Code Conventions (Quick Reference)
The top 10 rules from CONVENTIONS.md that apply to every task. Keep it scannable.

## Key Patterns
2-3 annotated code snippets showing the standard patterns for common operations
(API route structure, database query pattern, etc.)

## Current Focus
Roadmap not generated yet — run `tstack-roadmap`.
```

`tstack-architect` initializes `## Current Focus` to the placeholder above. Once `tstack-roadmap` runs, it updates the block to point at ROADMAP.md's Status section. After that the block is a **static pointer** — it doesn't change per milestone, because the live "what's next" lives in ROADMAP.md's Status, which `tstack-build` keeps current.

**Rules:**
- Don't enumerate every file in the project — agents can explore
- Don't paste full specs — point to them
- Do include common commands (agents can't guess your build/test scripts)
- Do include the 2-3 most important code patterns (saves agents from inventing their own)
- `tstack-architect` owns this file. Downstream skills (`tstack-roadmap`, `tstack-build`) update **only** the `## Current Focus` block — they never restructure the rest.

> **iOS projects:** Include Swift version, minimum deployment target, SwiftUI vs UIKit, dependency manager, async patterns (`async/await` vs Combine), Xcode build commands, and any Run Script phases that affect codegen.

---

### CLAUDE.md (Repo Root)

**Location:** Must be at the project root. Claude Code auto-loads this file at the start of every session.

**Purpose:** Points Claude Code to AGENTS.md. Keeps all agent instructions in one place so you don't maintain duplicate content across tool-specific files.

**Contents:**

```markdown
See @AGENTS.md
```

That's it. If you ever need Claude-specific overrides (rare), add them below the reference:

```markdown
See @AGENTS.md

## Claude-Specific Notes
- {Any Claude Code-only behavior, if needed}
```

**Why this pattern:** Other AI tools use different config files (.cursorrules, Codex config, etc.) but they all need the same project context. AGENTS.md is the canonical source. Each tool's config file just points to it. One place to update, every tool stays in sync.

---

### ARCHITECTURE.md

**Purpose:** Defines *how the system fits together*. The bridge between product requirements and implementation.

**Should contain:**
- Architecture philosophy / guiding principles
- High-level data flow diagram (ASCII art works well for Claude Code)
- Tech stack table with rationale for each choice
- Repository structure (full directory tree with annotations)
- Module boundaries — what runs where and why
- Service communication patterns (what calls what)
- Database overview — table inventory, relationship summary, NOT full SQL (that goes in a spec)
- Key integration points (webhooks, pipelines, sync protocols)
- Deployment topology and environments

**Should NOT contain:**
- Full SQL migrations (that's `2 - Specs/database-schema.md`)
- Endpoint definitions (that's API.md)
- Business requirements (that's PRODUCT.md)

---

### API.md

**Purpose:** The contract between services. Every endpoint that exists or will exist.

**For each endpoint, document:**
- Method + path
- Auth requirements
- Request body (with TypeScript-style type annotations or JSON examples)
- Response body (success and error shapes)
- Query parameters for GET endpoints
- Side effects (e.g., "triggers background job", "sends email")
- Rate limits

**Include a standard error response format** at the top so every endpoint follows it.

**Include a rate limiting table** summarizing limits by endpoint category.

---

### CONVENTIONS.md

**Purpose:** The coding standards. Claude Code follows these on every task without being asked.

**Should contain:**
- Language-specific rules (naming, typing, error handling, imports)
- File organization rules (one component per file, colocated tests, no barrel files, etc.)
- Framework-specific patterns (Server Components by default, validation at API boundary, etc.)
- Database conventions (naming, timestamps, soft deletes, FK behavior)
- Git conventions (branch naming, commit message format)
- Testing conventions (what to test, how to test, where tests live)
- Explicit anti-patterns ("never do X because Y")

**If the project spans multiple languages** (e.g., TypeScript + Swift), include a section for each language.

> **iOS projects:** Specify `async/await` vs Combine preference, `@Observable` vs `ObservableObject`, SwiftData vs Core Data, and whether you use `@MainActor` explicitly.

---

### TESTING.md

**Purpose:** The test strategy. Mandatory at every right-sizing level. It's what `tstack-roadmap` and `tstack-build` lean on to make "Done when" criteria command-verifiable, so it must name concrete, runnable checks — not aspirations.

**Should contain:**
- **Test levels and frameworks** — unit / integration / e2e split, with the framework chosen for each (e.g., Vitest for unit+integration, Playwright for e2e; pytest; swift test).
- **Coverage floor** — a specific number, not "high coverage" (e.g., "85% statements / 75% branches"). State what's measured and where it's enforced.
- **Where tests live** — alongside code (preferred for 2026 web stacks) vs a separate tree.
- **Mocking / test-data strategy** — what gets mocked, how fixtures/factories are built, how the DB is seeded/reset.
- **CI vs local** — which checks run on every PR vs on demand.
- **How the foundational ADRs are verified** — concretely: accessibility via `axe` in CI (ADR-3), security via dependency scanning / secret scanning (ADR-1), observability smoke-check that errors reach the tracker (ADR-2), privacy/deletion path covered by a test (ADR-4).

**Should NOT contain:** the conventions for writing code (that's CONVENTIONS.md — cross-reference it), or per-feature acceptance criteria (those live in PRODUCT.md and become milestone "Done when" criteria).

---

### DECISIONS.md

**Purpose:** Architecture Decision Records. Prevents Claude Code from suggesting things you've already evaluated and rejected. **Written first**, before the other technical docs.

**Always opens with the four foundational ADRs**, in this order, captured before any other doc is written (see SKILL.md for the questions each asks):
- **ADR-1: Security posture** — sensitive/regulated data, secret storage, auth provider, authorization model
- **ADR-2: Observability posture** — log destination + format, error tracker, metrics (or explicit "none in v1")
- **ADR-3: Accessibility posture** — WCAG target for consumer-facing UI, or explicit minimal-obligation note for CLI/API
- **ADR-4: Privacy & data handling** — residency, retention per data type, deletion capability, compliance regimes
- **ADR-5: AI/LLM strategy** — *only if* the product uses AI; pairs with `docs/2 - Specs/ai-strategy.md`

Tech-stack choices follow as ADR-6, ADR-7, … — each with a `Chosen: <date>` stamp and a revisit trigger.

**For each decision:**

```markdown
## ADR-{N}: {Decision Title}

**Date:** {Month Year}
**Status:** Accepted

**Context:** What problem needed solving.

**Options considered:**
1. Option A — brief description
2. Option B — brief description
3. Option C — brief description

**Decision:** Which option was chosen.

**Rationale:** Why this option won. Be specific about the technical reasons.

**Tradeoff:** What you gave up. Be honest — every decision has a cost.
```

Good ADRs save time. When Claude Code suggests "why not use X?", you can point to the ADR that already evaluated X.

---

### 2 - Specs/ (Breakout Specs)

**Purpose:** Detailed specifications for topics that need more depth than the high-level docs provide. These are NOT limited to user-facing features — they cover any area of the system that benefits from a dedicated, detailed document.

**Common spec topics:**
- Database schema (full SQL migrations, RLS policies, indexes)
- Encryption / security implementation
- Email ingestion pipeline
- File import/export pipelines
- Matching or scoring algorithms
- Offline sync protocols
- Background job definitions
- Admin system implementation
- Authentication flows (when complex)

**When to create a spec:** If a section of ARCHITECTURE.md or PRODUCT.md is getting too long or too detailed, break it out into a spec. The high-level doc should summarize and link to the spec: *"Full schema in `2 - Specs/database-schema.md`."*

**Spec format:** Each spec should be self-contained enough that Claude Code can read it and implement what it describes. Include:
- Overview of what the spec covers
- Reference back to which PRODUCT.md sections it implements
- Technical details (schemas, algorithms, prompt templates, data flows)
- Error handling table (what can go wrong, how to handle it)
- Monitoring / observability notes (what to track)

---

### ROADMAP.md

**Purpose:** Build-order milestone sequence. Tells Claude Code what to build next, what it depends on, and what "done" looks like.

**See the `tstack-roadmap` skill** for the complete process of creating this file from the other documentation.

**Key properties:**
- Milestones sequenced by **technical dependencies**, not by PRODUCT.md section order
- Each milestone scoped to **1-2 focused days** of work
- Each milestone includes: description, dependencies, doc references, definition of done
- No status indicators on individual milestones — all progress tracked in a single **Status section at the bottom** listing completed milestones and what's up next
- No task-level breakdown within milestones (Claude Code's Tasks system handles that)
- Includes dependency graph diagram, parallelization notes, and critical path

---

## Setting Up the Technical Docs

### Step-by-Step

**Not every project needs every document.** Skip any file that doesn't apply — if there's no API, skip API.md. If there are no complex subsystems, skip breakout specs. Only create what's useful. ARCHITECTURE.md, CONVENTIONS.md, and AGENTS.md are the essential core for this step.

1. **Write ARCHITECTURE.md.** Translate product requirements into system design. Choose the tech stack, define the repo structure, draw the data flow, establish module boundaries. This is where PRODUCT.md's "what" becomes "how."

2. **Write API.md.** Define every endpoint contract. Request shapes, response shapes, auth, errors. If the project has a mobile app, the API doc is the contract between server and client.

3. **Write CONVENTIONS.md.** Establish code style before any code exists. This prevents inconsistency from the first line.

4. **Write TESTING.md.** Define the test strategy — levels, frameworks, coverage floor, and how the foundational ADRs are verified. Mandatory at every size; `tstack-build` verifies milestones against it.

5. **Write DECISIONS.md.** In practice this is written *first* (the four foundational ADRs gate everything), but ensure every "we chose X over Y" from the steps above has an ADR with a `Chosen: <date>` stamp.

6. **Write breakout specs** for anything that needs more detail. Database schema is almost always a spec. Pipelines, sync protocols, and complex algorithms usually need specs too. If the product uses AI, `2 - Specs/ai-strategy.md` is mandatory regardless of size.

7. **Create AGENTS.md** at the repo root. This is the single source of truth for all AI agents and `tstack-architect` owns it. Point to all the docs (including TESTING.md and DECISIONS.md). Include commands and key patterns. Note: the AGENTS.md template references `docs/ROADMAP.md` — that file doesn't exist yet and that's expected. It's created in the next stage by `tstack-roadmap`.

8. **Create CLAUDE.md** at the repo root with contents: `See @AGENTS.md`. If you use other AI tools, create their config files the same way (e.g., `.cursorrules` pointing to AGENTS.md).

**Stop here.** The roadmap is created separately by the `tstack-roadmap` skill in a new session.

### 9. Validate Documentation Consistency

Before moving to roadmap generation, verify the docs are internally consistent. Run through this checklist (Claude can do this, or you can do it manually):

1. Every feature in PRODUCT.md has corresponding coverage in API.md (for endpoints), ARCHITECTURE.md (for system components), or a breakout spec (for complex subsystems)
2. Every endpoint in API.md traces back to a feature or flow in PRODUCT.md
3. Every significant tech stack choice in ARCHITECTURE.md has an ADR in DECISIONS.md (with a `Chosen:` date)
4. DECISIONS.md opens with the four foundational ADRs (+ AI ADR if the product uses AI)
5. TESTING.md's a11y/security checks align with ADR-3 and ADR-1
6. Breakout specs reference which PRODUCT.md sections and ARCHITECTURE.md components they implement
7. AGENTS.md correctly points to all created docs (including TESTING.md and DECISIONS.md) and includes accurate common commands

Catching inconsistencies now is much cheaper than discovering them during implementation.

---

## Working with Claude Code

For detailed guidance on Claude Code prompting patterns, session management, context hygiene, and git workflow during implementation, see the `tstack-build` skill's `references/full-guide.md`.

**Key principles for the documentation phase:**

- **Start fresh sessions** for unrelated tasks — context pollution degrades output quality
- **Don't load all docs at once.** CLAUDE.md tells Claude Code *when* to read each file. Let it load docs on demand.
- **Use `.claudeignore`** to exclude build artifacts, large assets, and environment files from Claude Code's context.
- **Commit frequently.** Claude Code works better with clean git state. Treat Claude Code output like a PR from a teammate — review diffs before accepting.

---

## Troubleshooting

### Context Degrades Partway Through

If Claude Code's output quality drops noticeably after creating 3-4 documents (vague content, inconsistencies with earlier docs), the session context is getting too large. Split the work:

```
Let's pause here. Commit what we have. I'll start a fresh session for the
remaining documents. Read the docs we've created so far before continuing.
```

### Claude Bakes In a Tech Stack Silently

Claude should *propose* its opinionated default explicitly and let you confirm or override — not quietly bake a choice into ARCHITECTURE.md as if it were settled. Re-anchor it:

```
Stop. Don't silently commit the stack. State your default for the database and
why, then let me confirm or override. If I push back, give me 2-3 options with
tradeoffs (cost, complexity, team familiarity). Record the decision in
DECISIONS.md with a Chosen: date.
```

### Docs Are Too Thin for Implementation

If a document reads like a summary rather than an implementation reference (e.g., ARCHITECTURE.md that says "we use a database" without specifying which one), push for specifics:

```
This ARCHITECTURE.md is too high-level for Claude Code to implement from.
Add: specific tech stack versions, directory structure with file annotations,
data flow diagram, and module boundary definitions. Claude Code needs enough
detail to write code without guessing.
```
