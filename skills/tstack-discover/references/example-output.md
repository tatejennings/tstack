# Example output: `docs/1 - Discovery/business-brief.md`

This is a realistic example of what `tstack-discover` produces for a small but non-trivial product. Read this to calibrate "specific enough" before producing your own brief. Anonymized but representative — every section here corresponds to a real decision the founder made.

---

# Slink — Business Brief

## Executive Summary

Slink is a link shortener built for content creators (newsletter writers, podcasters, YouTubers) who manage 50–500 outbound links per month. Existing tools (Bitly, Rebrandly) optimize for marketing teams or enterprise; creators are an underserved middle. Slink differentiates on branded domains in the free tier, per-link referrer analytics, and AI-suggested slugs that match the linked content. v1 ships in 8–10 weeks for solo dev.

## Problem Statement

Creators publish 5–20 links per piece of content. Tracking which links drive subscribers, listens, or clicks is currently a mess: bit.ly's free tier doesn't allow branded domains, Rebrandly's pricing jumps from $0 to $24/mo, and neither integrates well with newsletter platforms. Creators end up either (a) using ugly bit.ly URLs that hurt brand, (b) paying for tools built for enterprise marketing teams, or (c) cobbling together UTM parameters and a spreadsheet.

## Target User

**Primary:** Solo or small-team content creators publishing weekly+:
- Newsletter writers (Beehiiv / Substack / Ghost), 2k–50k subscribers
- Podcasters with 1k–20k weekly listens
- YouTubers in the 10k–500k subscriber range
- Technical sophistication: comfortable with CSV exports, can configure DNS, doesn't write code

**Secondary:** Two-person creator businesses (creator + editor/manager). Need shared link library.

Budget: $5–$15/mo for "tools that earn their keep." Above $15 needs clear ROI.

## Market Landscape

### Direct Competitors

| Competitor | What They Do | Pricing | Strengths | Weaknesses |
|---|---|---|---|---|
| Bitly | URL shortener + analytics | Free (no custom domain) / $8/mo / $29/mo | Trusted brand, deep analytics | No custom domain in free; UI bloated; built for marketers |
| Rebrandly | Branded short links | Free (5 links) / $24/mo | Custom domains in free tier | $24 jump is steep; limited free tier |
| Dub.co | OSS modern shortener | Free (10 domains) / $24/mo | Modern UX, good API | Aimed at agencies; analytics paywalled |
| TinyURL | Simple shortener | Free | No signup needed | No analytics, no custom domain |

### Adjacent Products

- **UTM builders** (Google Campaign URL Builder) — manual, no analytics dashboard
- **Newsletter analytics** (Beehiiv built-in) — tied to one platform, no cross-channel rollup
- **Linktree / Beacons** — link-in-bio, not the same job

### Market Opportunity

Bitly's free tier removed custom domains in 2022, pushing budget-conscious creators to alternatives. Most alternatives still target marketing teams. There's a clear gap for "Pro tier for creators, $9/mo" — better than Bitly free, cheaper than Rebrandly Pro.

## Value Proposition

**One-sentence pitch:** "Branded short links and per-link analytics for creators, at a price that doesn't assume you have a marketing budget."

**Differentiators:**
1. Branded domain in free tier (5 links/mo) — direct shot at Bitly's most-complained-about restriction
2. AI slug suggestions based on the linked page's content — never type `my-newsletter-issue-47` again
3. Newsletter-platform integrations (Beehiiv, Substack, ConvertKit) — paste your draft, get shortened links inline

**Wedge:** AI slug suggestions. Faster than typing, looks more professional than auto-generated, no competitor offers it.

## Product Overview

### Core Features (v1)

1. **Account creation & branded domain setup** — Clerk auth; DNS verification flow for custom domain
2. **Link creation** — paste URL, get short link; optional custom slug; optional AI suggestion
3. **Link management** — list view, edit destination, archive, search
4. **Per-link analytics** — total clicks, unique visitors, referrer breakdown, geographic rollup (country-level), time-series chart (last 30 / 90 days)
5. **Redirect service** — fast (<50ms p95) redirect with click tracking
6. **CSV export** — for off-platform analysis
7. **AI slug suggestions** — LLM reads linked page title/meta, proposes 3 slug options

### Explicitly Deferred (v2+)

- Team workspaces (planned post-launch — clear path via tstack-specify-feature)
- Mobile app (web is responsive; native deferred until >5k MAU)
- Real-time click stream (batched analytics fine for v1)
- A/B testing of destination URLs
- QR code generation

### User Flows

**Onboarding (first 5 min):** sign up via Clerk → add custom domain (or use `slink.app`) → DNS verify → create first link → see redirect work.

**Daily use:** open dashboard → paste a URL → (optional) accept AI slug suggestion → copy short link → paste into newsletter. Repeat 5–20×.

**Weekly review:** click into a link → see referrer breakdown → identify what drove the most clicks → adjust next issue's framing.

## Business Model

| Tier | Price | Links/mo | Custom domains | Analytics retention | AI slugs |
|---|---|---|---|---|---|
| Free | $0 | 5 | 1 | 7 days | 5/mo |
| Pro | $9/mo | Unlimited | 3 | 1 year | Unlimited |
| Team | $29/mo | Unlimited | 10 | 2 years | Unlimited |

Unit economics: hosting + redirect serving estimated at $0.15/active user/mo; LLM costs ~$0.04 per AI slug suggestion (Haiku-class model). Pro gross margin ~88% at scale; Free tier needs throttling on AI to avoid blow-up.

Expansion path: Team → Agency tier (post-v1).

## Go-to-Market

- **First 10 users:** founder's newsletter audience + 5 personal asks
- **First 100:** Product Hunt launch (week 4 post-v1); creator-economy Twitter; cross-post on Beehiiv community
- **Launch:** Product Hunt + Hacker News "Show HN"
- **Timeline:** 10 weeks to v1, 2-week beta, public launch at week 12

## Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|
| Bitly adds custom domains to free tier | High | Low | Lean harder on AI slugs and integrations; not a price war we'd win |
| LLM costs explode on free tier | Medium | Medium | Hard rate limits (5 AI slugs/mo free); switch to cheaper model if margin collapses |
| Redirect SLA breaks (slow/downtime) | High | Low | Edge deploy from day 1 (Cloudflare Workers); status page; rollback plan |
| DNS verification too friction-heavy | Medium | Medium | Offer slink.app subdomain as default; custom domain optional |
| AI slug suggestions are bad | Medium | Medium | Manual override always available; eval set of 50 prompts to track quality |

## Success Metrics

| Timeframe | Metric | Target |
|---|---|---|
| 3 months | Paid conversions | 50 Pro subscribers |
| 6 months | MRR | $750 |
| 12 months | Paid users | 250 (mix of Pro + Team), MRR $3k |

Kill signals: <10 Pro by month 3 (positioning failure); >40% AI-slug rejection rate (feature failure); >5% redirect error rate (technical failure).

Double-down signals: organic Bitly migrators >25% of signups; >$1k MRR by month 4.

## Technical Context

- **Team:** solo founder, full-stack TS background, prefers Next.js + Postgres
- **Existing code:** none, greenfield
- **Constraints:** must run on managed infra (no ops time budget); redirect latency <50ms p95 globally
- **Compliance:** EU users present → GDPR applies (cookie consent, data deletion, EU data residency for redirect logs if possible)
- **Deploy preference:** Vercel for app + Cloudflare Workers for redirect edge

## Open Questions

- Should free-tier links expire after 90 days of inactivity (cost control) or stay forever (trust)?
- How to handle abuse / malware short-link reports? Auto-suspend on report + manual review?
- Single vs multi-region Postgres in v1? (Single fine for v1; revisit at 1000 paid users.)

## Appendix

- Competitor pricing screenshots: see `2025-research/`
- Beehiiv community feedback threads: `2025-research/beehiiv-feedback.md`
- LLM cost back-of-envelope: `2025-research/ai-cost-model.md`
