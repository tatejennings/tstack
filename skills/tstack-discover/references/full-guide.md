# Product Discovery & Business Brief Guide

## Purpose

This document defines a structured process for turning a rough product idea into a comprehensive business brief. The output is a `business-brief.md` file that feeds directly into the Claude Code Project Documentation Guide as the foundation for all technical documentation.

Give this file to Claude along with a short description of your product idea. Claude will act as a senior product manager — asking questions, researching the market, challenging assumptions, and helping you think through the product before any technical decisions are made.

---

## How to Use This

### What You Provide

A short description of your product idea. It can be as rough as a few sentences:

> "I want to build an app that helps people with multiple LLCs track their expenses and stay compliant. Think QuickBooks but for people with 5+ businesses."

Or as detailed as a few paragraphs. The less you provide, the more questions Claude will ask. That's fine — the questions are the point.

> **Web search:** The competitive research step works best with a Claude interface that has web search enabled (Claude.ai, Claude Code). If web search isn't available, do your own competitor research and include it with your product description — Claude will still run the full discovery process.

### What Claude Does

1. **Reads your description** and identifies what's clear vs. what's missing
2. **Researches the market** — competitors, adjacent products, market size signals
3. **Asks structured questions** across 9 discovery areas (below)
4. **Challenges assumptions** — flags risks, suggests alternatives, pushes back where needed
5. **Synthesizes everything** into a business brief document

### What You Get

A `business-brief.md` file that covers: what the product is, who it's for, why it matters, how it makes money, what the competition looks like, what's in scope for v1, and what's explicitly out of scope. This file becomes the input for creating PRODUCT.md, ARCHITECTURE.md, and the rest of the technical documentation.

---

## Claude's Approach

Claude follows this process in order. The discovery conversation is the most valuable part — don't skip straight to writing the brief.

1. **Start by reflecting back what it understands.** Summarizes the product idea in its own words — what it does, who it's for, and what problem it solves. Calls out anything that's unclear or ambiguous. This lets you correct misunderstandings early.

2. **Research the competitive landscape.** Before asking questions, Claude searches the web for:
   - Direct competitors (products solving the same problem)
   - Adjacent products (products solving a related problem that users might use instead)
   - Market signals (is this market growing? are people complaining about existing solutions?)
   - Pricing benchmarks (what do competitors charge?)

   Presents findings as a brief competitive summary. Flags competitors you should be aware of, and notes gaps in the market your product could fill.

3. **Ask discovery questions.** Works through the 9 discovery areas below. Groups them into 2-3 areas per round, starting with the most foundational. Waits for your answers before moving to the next round.

   Adapts questions based on what you've already said. Skips questions you've already answered. Goes deeper on areas where your answers reveal uncertainty or risk.

   **Claude should be opinionated.** If something sounds like it won't work, it says so and explains why. If it sees a better approach, it suggests it. If you're building something that already exists and does it well, it tells you. You need a thinking partner, not a yes-machine.

4. **After discovery is complete, writes the business brief.** Uses the template at the bottom of this document. Presents it for your review before finalizing.

---

## Discovery Areas

### 1. Problem & Market
- What specific problem does this solve? (be precise — "manages expenses" is too vague)
- Who has this problem? How many of them are there?
- How are they solving it today? What's painful about current solutions?
- Why now? What's changed that makes this the right time?
- Is this a vitamin (nice to have) or a painkiller (must have)? What's the cost of NOT solving it?

### 2. Target User
- Describe the primary user in detail. Job title, company size, daily workflow, technical sophistication.
- Is there a secondary user? (e.g., the primary user's accountant, team, admin)
- What's their budget for this kind of tool? Do they already pay for something?
- Where do they hang out? (communities, conferences, publications — this informs go-to-market)
- What would make them switch from their current solution? What would prevent them from switching?

### 3. Core Value Proposition
- If you had to explain this product in one sentence to your target user, what would you say?
- What's the single most important thing this product does better than alternatives?
- What are 2-3 key differentiators? (things competitors can't easily copy)
- Is there a "wedge" — a single feature so compelling that users will adopt just for that, then discover the rest?

### 4. Product Scope (v1)
- What are the absolute minimum features for v1? (be ruthless — what can you cut?)
- What features do users expect but you're intentionally deferring? Why?
- What's the primary platform? (web, mobile, desktop, API)
- Is there an offline requirement?
- What integrations are essential for v1 vs. nice-to-have?
- What does the user's first 5 minutes look like? (onboarding flow)
- What does a typical daily/weekly usage session look like?

### 5. Business Model
- How does this make money? (subscription, usage-based, freemium, marketplace, etc.)
- What's the pricing structure? What tier boundaries make sense?
- Is there a free tier? What's included vs. gated?
- What's the target price point? How does it compare to competitors?
- What are the unit economics? (cost to serve one user — hosting, API calls, support)
- Is there a path to expansion revenue? (users paying more over time)

### 6. Go-to-Market
- How will the first 10 users find this product?
- How will the first 100?
- Is there a beta/waitlist strategy?
- What does "launch" look like? (Product Hunt, community post, direct outreach, etc.)
- Are there partnerships or channels that could accelerate growth?
- What's the timeline? When do you want v1 in users' hands?

### 7. Risks & Assumptions
- What are you assuming that might be wrong? (e.g., "users will pay $X", "the market is big enough")
- What's the biggest technical risk? (something that might be harder than expected)
- What's the biggest market risk? (something about user behavior or competition)
- What regulatory or compliance considerations exist?
- What happens if a large competitor builds this feature? (e.g., QuickBooks adds multi-entity support)

### 8. Success Criteria
- What does success look like at 3 months? 6 months? 12 months?
- What's the one metric that matters most? (active users, revenue, retention, transactions processed, etc.)
- What would make you kill this project? (what failure signal would you act on?)
- What would make you double down? (what success signal would trigger more investment?)

### 9. Technical Context
- What's the team's technical background? (languages, frameworks, platforms they know best)
- Is there an existing codebase, or are we starting from scratch?
- Any existing systems or services this needs to integrate with? (databases, APIs, auth providers)
- Hosting or deployment preferences or constraints? (cloud provider, region, budget, self-hosted)
- Performance requirements? (expected concurrent users, response time targets, data volume)
- Any regulatory or compliance constraints that affect technical choices? (data residency, encryption at rest, audit logging, HIPAA, SOC 2)

---

## Business Brief Template

After completing discovery, Claude writes the brief using this structure:

```markdown
# {Product Name} — Business Brief

## Executive Summary
2-3 paragraphs: what the product is, who it's for, what problem it solves, and why now.

## Problem Statement
The specific pain point. Include quotes or scenarios that illustrate the problem.
Quantify the cost of the problem when possible (time wasted, money lost, risk exposure).

## Target User
Primary persona with demographics, workflow, pain points, current tools, and budget.
Secondary personas if applicable.

## Market Landscape

### Direct Competitors
| Competitor | What They Do | Pricing | Strengths | Weaknesses |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

### Adjacent Products
Products that solve a related problem or that users currently cobble together.

### Market Opportunity
Size signals, growth trends, underserved segments, and the gap your product fills.

## Value Proposition
One-sentence pitch, key differentiators, and the wedge feature (if applicable).

## Product Overview

### Core Features (v1)
Prioritized feature list with brief descriptions. Group by user-facing category.

### Explicitly Deferred (v2+)
Features intentionally excluded from v1 with rationale for deferral.

### User Flows
Primary user journeys described step by step. (These feed directly into PRODUCT.md later.)

## Business Model
Pricing tiers, free vs. paid boundaries, unit economics, expansion revenue path.

## Go-to-Market
First 10 users, first 100 users, beta strategy, launch plan, timeline.

## Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|
| ... | ... | ... | ... |

## Success Metrics
| Timeframe | Metric | Target |
|---|---|---|
| 3 months | ... | ... |
| 6 months | ... | ... |
| 12 months | ... | ... |

## Technical Context
Team background, existing systems, hosting preferences, performance requirements,
and any regulatory constraints that will shape architecture decisions.

## Open Questions
Anything unresolved that needs more research or user feedback.

## Appendix
Competitor screenshots, market data sources, reference links.
```

---

## After the Brief Is Complete

The business brief becomes the first file in `docs/1 - Discovery/business-brief.md`. From there, the workflow is:

1. **Business brief** → feeds into creating **PRODUCT.md** (product requirements)
2. **PRODUCT.md** → feeds into **ARCHITECTURE.md** (system design), **API.md** (contracts), **CONVENTIONS.md** (code style)
3. **Architecture decisions** → captured in **DECISIONS.md**
4. **Complex topics** → broken out into **2 - Specs/** files
5. **AGENTS.md** created at repo root pointing to all docs
6. **CLAUDE.md** created with `See @AGENTS.md`
7. **All docs together** → fed into Claude Code to generate **ROADMAP.md**
8. **Implementation begins** following the roadmap

See `02-create-product-doc.md` for the next step: creating PRODUCT.md.
See `03-create-technical-docs.md` for creating the technical documentation.
See `04-generate-roadmap.md` for the roadmap generation process.

---

## Tips for Getting the Most Out of Discovery

- **Don't rush through the questions.** The discovery conversation often surfaces insights that change the product direction. A 30-minute conversation here can save weeks of building the wrong thing.
- **Be honest about what you don't know.** "I'm not sure" is a valid answer. Claude will help you figure it out or flag it as an open question.
- **Push back on Claude's pushback.** If Claude challenges an assumption and you have a good reason, say so. The goal is a rigorous brief, not consensus.
- **Keep the brief living.** Update it as you learn from beta users. The brief isn't a contract — it's a snapshot of your best thinking at this moment.
- **Don't skip the competitor research.** Even if you think you know the market, Claude's web search often surfaces competitors or market signals you missed.

---

## Troubleshooting

### Claude Produces a Shallow Brief

The brief reads like a generic template with your product name plugged in. Push back:

```
This brief is too generic. The competitive analysis doesn't mention specific
products or pricing. The user flows are vague. Go deeper — research actual
competitors, name them, and describe specific user workflows step by step.
```

### Claude Agrees with Everything

Claude should be a thinking partner, not a yes-machine. If it's not pushing back:

```
You're being too agreeable. Challenge my assumptions. What's the weakest part
of this idea? What competitor am I underestimating? Where is my scope too
ambitious for v1?
```

### Web Search Isn't Available

If you're using an interface without web search, do your own competitor research first and include it with your product description. Claude will still run the full discovery process — it just won't be able to research the market independently.

### Discovery Is Taking Too Long

If you're past the third round of questions and the conversation feels circular, it's time to write the brief:

```
We've covered enough ground. Write the business brief now using everything
we've discussed. Flag anything still unresolved in the Open Questions section.
```
