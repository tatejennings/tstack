---
name: tstack-design
description: Designs the UX and visual interface for a TStack product and writes the docs/3 - Design/ set — design.md (information architecture, navigation, user flows, per-screen breakdown with all states, visual direction, component inventory, accessibility tied to ADR-3), design-tokens.json (W3C DTCG), a paste-ready claude-design-prompts.md for Claude Design, and previews/. The conditional UI design stage of the chain — runs between tstack-product and tstack-architect for products with a consumer-facing UI (so the frontend-stack ADR and ADR-3 are design-informed), and is also invokable any time (after roadmap, mid-build) for screen-level work. Always asks the design level (wireframe vs high-fidelity) on first run — in either mode, including when adopting — never assuming high-fidelity. Two modes — GENERATE (then a route: a Claude Design brief [works at both fidelities — a wireframe pass still produces wireframe Claude Design prompts], in-repo HTML previews, or a Figma round-trip) and LINK-BACK/ADOPT (you designed in Claude Design/Figma, or tstack-ingest staged a draft — record the link/screenshots into the design set). Use when the user says "design the UI", "design/spec the screens", "create a design system", "wireframe this", "give me Claude Design prompts", "I designed this in Claude Design / link my designs", or "I have Figma mockups/screenshots to bring in". Skip for headless products (CLI/library/API — no UI). Not a requirements skill — to define WHAT to build use tstack-product/tstack-specify-feature. Input is docs/PRODUCT.md (+ docs/ARCHITECTURE.md frontend stack & ADR-3 if present, or docs/_adopted/design.draft.md when adopting). Output is the docs/3 - Design/ set. Hands off to tstack-architect.
---

# tstack-design

You are running TStack's design/UX stage. You turn product intent into a concrete UX + visual specification, a machine-readable token set, and ready-to-paste material for **Claude Design** — giving frontend work an upstream source of truth that `tstack-build` can later verify against (especially accessibility). You own the `docs/3 - Design/` set; no other skill writes it.

**Conditional UI stage with a retained off-chain mode.** For a product with a consumer-facing UI this runs **between `tstack-product` and `tstack-architect`** — so the frontend-stack ADR and ADR-3 (accessibility) are *design-informed* rather than guessed. It is **conditional** (skipped entirely for headless products) and a **recommendation, never a gate**: it blocks nothing, and you can also invoke it any time — before architect, after roadmap, or mid-build when a single screen needs a design pass. Like `tstack-ingest` (an on-ramp, not off-chain), it has a place in the chain without ever forcing one.

## When to run / when to skip

- **Run** for any product with a consumer-facing UI (web or mobile). Recommended right after `tstack-product`, before `tstack-architect`.
- **Skip** for headless products — APIs, CLIs, libraries, backend services with no UI. If asked anyway, say so and stop rather than inventing UI for something that has none.

## Prereq check (soft)

Work from whatever exists:

- **`docs/_adopted/design.draft.md` exists** → the user adopted existing designs via `tstack-ingest`. Take the **adopt/ratify** path (step 5) — but **still choose fidelity first (step 1)**: adopting existing material is *not* a reason to assume high-fidelity. Author the `docs/3 - Design/` set from the draft at the chosen level, then delete it.
- `docs/PRODUCT.md` — preferred. Use its features, flows, and §6 screen inventory as the design's requirements. If absent, work from the user's description (and offer that `tstack-product` would give firmer ground).
- `docs/ARCHITECTURE.md` — if present, read the **frontend stack**; express tokens and components in that system's vocabulary (shadcn/ui, Tailwind, SwiftUI), not against it. When design runs *before* architect the stack isn't chosen yet — keep tokens stack-neutral (that's why `design-tokens.json` is DTCG).
- `docs/DECISIONS.md` — if present, honor **ADR-3 (accessibility)**: meet at least the committed WCAG bar, never propose a weaker one.

## Approach

1. **Choose the design level (fidelity) — always ask this first, in every mode.** Two choices, nothing else:
   - *Wireframe / UX-structural* — structure, layout, IA, flows, state placement, and the component inventory, plus a **wireframe-fidelity Claude Design brief**; no brand/color tokens or visual polish. **Not a lock-in:** re-run the skill for a high-fidelity pass once the structure settles (this skill is invokable any time).
   - *High-fidelity visual* — everything in the wireframe pass **plus** the full visual layer: design tokens, brand, styled components, and a high-fidelity Claude Design brief.

   Default to **wireframe** when running *before architect* (the cheap, architecture-informing pass) and **high-fidelity** for a UI milestone — but **state the default and let the user confirm or override**. **Ask even when adopting an ingest-staged draft or bringing back a link** — pre-existing design material is never a reason to assume high-fidelity; the user decides at what level you author the `docs/3 - Design/` set. Do **not** offer "UX now, visuals later" as a separate third option — it produces the same immediate deliverable as the wireframe pass and the rerun is always available, so it's just the wireframe choice. Skip this question only when re-running against an existing `docs/3 - Design/` set whose level is already settled.

2. **Pick a mode.**
   - **Generate** — produce the design from PRODUCT.md (route in step 3, then steps 4 & 6).
   - **Link-back / adopt** — the user already has designs (a Claude Design / Figma link, screenshots) or `tstack-ingest` staged a draft (steps 5 & 6).

3. **Generate — choose a route** (how the artifact gets produced — default **A**):
   - **A · Claude Design (recommended; works at *both* fidelities).** Claude Design is Anthropic's browser design tool at **`claude.ai/design`** (Mac/Windows/Linux — no native app to install). It goes wireframe → high-fidelity, so **a wireframe pass still routes through it** — never defer Claude Design just because the visual layer isn't decided yet. *Its platform is unrelated to the product's target platform:* it designs native macOS/iOS apps, web apps, and CLI dashboards alike — never steer away from it because the product is "a Mac app" or "not web." Write a **structured, paste-ready** brief to `docs/3 - Design/claude-design-prompts.md`: **one kickoff prompt** (the design system + the **component inventory as the shared library**, ending by asking Claude Design to plan), then **one self-contained, send-as-is prompt per screen** that reuses that library, then a **consistency-pass prompt** — each a single block the user pastes and sends, never loose fragments to assemble. The fidelity from step 1 sets what the kickoff carries:
     - **High-fidelity:** the design system = tokens + brand + a11y + the component library; screen prompts carry full visual detail.
     - **Wireframe:** the design system is the **structural** system — layout/spacing/hierarchy primitives + the component inventory + the a11y intent, with **color/brand tokens omitted** until the high-fidelity rerun; each screen prompt asks for *low-fidelity grayscale, boxes-and-labels, focused on layout, hierarchy, and the full state set* (see `references/example-output.md` for the exact wireframe swap). You **still write `claude-design-prompts.md`** — wireframe is a fidelity of the brief, not a reason to skip it.

     The kickoff carries the standing rule *reuse shared components, don't recreate them; apply any component refinement across every page* (Claude Design otherwise makes each pasted screen an independent page and earlier screens keep placeholder components). Claude Design has **no cross-session memory** — the file tells the user to re-paste the kickoff when starting a new session. The user generates in Claude Design, then **links the result back** (step 5). It's a separate surface you can't invoke directly — your job is the best brief + the link-back loop, not out-designing it.
   - **B · In-repo HTML previews.** Write self-contained HTML per screen (all states) to `docs/3 - Design/previews/`. Be honest about fidelity: an HTML **wireframe** is fully adequate; a high-fidelity preview is a *rough scaffold, not the final design* — use Route A for hi-fi quality. Good for an instant look without leaving the session. (Use the **frontend-design** skill as the engine.)
   - **C · Figma round-trip.** Only if a **Figma MCP is connected** — build the screens in Figma (lo-fi or hi-fi) and pull screenshots back into `previews/`. Offer it when you detect Figma; never assume it.

4. **Author the design content** (structure below) and, at high fidelity, the **token set.** Write `docs/3 - Design/design-tokens.json` in **W3C DTCG** format as the canonical, stack-agnostic token source; mirror a readable summary in `design.md`. A pure **wireframe** pass may defer **only** the token JSON / brand-color layer until the visual pass — it does **not** defer the Claude Design brief or the UX content, both of which are produced now.

5. **Link-back / adopt.**
   - **A brought-back link or screenshots** (lightweight, no ingest): record it in the `## Design source` block at the top of `design.md` (URL + tool + date + pointer to `previews/`), save any screenshots/exports to `docs/3 - Design/previews/`, and reconcile any edits the user made back into `design.md`. If the design set doesn't exist yet, author it from the brought-back material + PRODUCT.md. You own `docs/3 - Design/` and write it directly — no `docs/_adopted/` quarantine for a simple link.
   - **An ingest-staged draft** (`docs/_adopted/design.draft.md` exists): ratify it — read the draft + its `## Open gaps`, resolve every gap through this skill's steps, author the `docs/3 - Design/` set **at the chosen fidelity (step 1)**, record the source in `## Design source`, and **delete the draft on save** (`docs: author 3 - Design from adopted design`).

6. **Write the `docs/3 - Design/` set.** Create the folder if needed. Commit: `docs: add design for {product/screen}`.
   - `design.md` — the canonical UX + visual spec (structure below).
   - `claude-design-prompts.md` — the send-as-is Claude Design brief (Route A), at the chosen fidelity: a paste-once kickoff + one prompt per screen + a consistency pass.
   - `design-tokens.json` — DTCG tokens (high-fidelity pass; deferred for a pure wireframe).
   - `previews/` — HTML mockups / screenshots (Routes B/C, or linked screenshots).
   - `screens/{name}.md` — per-screen breakouts **only** for large multi-screen apps (right-sizing; mirrors architect's breakout specs).

7. **Round-trip awareness.** Whenever you change a `design.md` that carries a `## Design source` link, **ask**: "this design is linked to `{URL}` — want to update the linked Claude Design to match these changes?" Regenerate the updated prompts into `claude-design-prompts.md` for the user to paste. Never auto-push to the external tool.

## What `design.md` contains

The canonical, human-readable design doc — **UX first, then visual** (the Claude Design brief lives separately in `claude-design-prompts.md`):

- **`## Design source`** — only if linked: URL + tool + date + pointer to `previews/`.
- **Part 1 — UX spec**
  - *Information architecture* — content model, screen map / sitemap, grouping & hierarchy.
  - *Navigation structure* — nav pattern (tabs / sidebar / stack / breadcrumb), the screen/route graph, global vs contextual nav.
  - *User flows* — ordered steps across screens per key task (happy path + key alternates).
  - *Screen / page breakdown* — per screen: purpose, layout regions & hierarchy, components used, and the **full state set** (populated / empty / loading / error). Empty/error/loading are first-class; `tstack-build` expects them.
  - *Interaction patterns* — forms, validation, feedback, transitions, motion intent.
- **Part 2 — Visual spec**
  - *Design direction* — tone, density, personality.
  - *Design tokens (summary)* — readable mirror of `design-tokens.json` (semantic color roles, type scale, spacing, radius, elevation).
  - *Component inventory* — each component with variants and states.
  - *Accessibility patterns* — focus order, keyboard nav, contrast against the token set, labelling, reduced-motion — the a11y criteria that satisfy ADR-3 downstream.

**Seam with `PRODUCT.md §6`:** product says *which* screens exist and what each must do (requirements); design owns the deeper navigation model, IA, and per-screen layout/state breakdown. Don't duplicate the screen list — deepen it. For large apps, break the per-screen breakdown out to `screens/{name}.md`.

## Reference handoff

This skill pairs with the environment's design tooling — it defines *what* to design; they help *produce* it:
- **frontend-design** — translating the spec (or a Route-B preview) into production-grade component code.
- The **Figma** skills — building or syncing a design system / screens in Figma from this spec (Route C).

For what Claude Design is and how to write effective prompts for it — fidelity/credits, the clarifying-question kickoff, per-screen structure, **keeping components consistent across pages**, and refinement — read `references/claude-design-prompting.md` before authoring `claude-design-prompts.md`.

For a realistic example of the `docs/3 - Design/` set — `design.md`, the DTCG token set, and the `claude-design-prompts.md` material — read `references/example-output.md`.

## Handoff

When the design set is written and committed:

> Design written to `docs/3 - Design/` — {n} screens, {n} components, a DTCG token set, and (Route A) a `claude-design-prompts.md` ready to paste into **Claude Design**.
>
> **Next: run `tstack-architect`** — it reads the design set so the **frontend-stack ADR** and **ADR-3 (accessibility)** are design-informed. (If you ran this mid-build for one screen, just continue — nothing forces a chain step.)
>
> {If Route A: once you've generated in Claude Design, bring the **link or screenshots** back and I'll record them in `design.md`'s `## Design source` and reconcile any changes.}
