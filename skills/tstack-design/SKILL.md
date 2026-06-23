---
name: tstack-design
description: Produces a design/UX spec (docs/2 - Specs/design.md) plus ready-to-paste prompts and structured context for Claude Design — design tokens, component inventory, key screen layouts, interaction/empty/error/loading states, and accessibility patterns tied to ADR-3. Optional and off-chain — invoke it at ANY point in a TStack project (before architect, after roadmap, or mid-build) when a consumer-facing surface needs design work. Use when the user says "design the UI", "create a design system", "give me design prompts", "spec a screen", "design material for Claude Design", or wants the look-and-feel defined. Input is docs/PRODUCT.md if present (else the user's description) + docs/ARCHITECTURE.md's frontend stack and ADR-3 if present; output is docs/2 - Specs/design.md. Not part of the discover→build chain; hands off to nothing by default.
---

# tstack-design

You are running TStack's design/UX stage. You turn product intent into a concrete design specification **and** into material a human can paste straight into Claude Design to generate the actual interface. You give frontend work an upstream source of truth that `tstack-build` can later verify against (especially accessibility).

**This skill is optional and off-chain.** It is *not* a step in `discover → product → architect → roadmap → plan → build`. No skill hands off to it, it blocks nothing, and you can run it whenever a consumer-facing surface needs design — before `tstack-architect` (to inform the frontend stack), after `tstack-roadmap` (to flesh out a UI milestone), or mid-`tstack-build` (when a screen needs a design pass).

## When to run / when to skip

- **Run** for any product with a consumer-facing UI (web or mobile) that needs look-and-feel, a design system, or screen-level design defined.
- **Skip** for headless products — APIs, CLIs, libraries, backend services with no UI. If asked anyway, say so and stop rather than inventing UI for something that has none.

## Prereq check (soft)

Nothing is mandatory — this skill works from whatever exists:

- `docs/PRODUCT.md` — preferred. Use its features, user flows, and screen inventory as the design's requirements. If it's absent, work from the user's description (and offer that `tstack-product` would give you firmer ground).
- `docs/ARCHITECTURE.md` — if present, read the **frontend stack** (e.g., Next.js + shadcn/ui + Tailwind, or SwiftUI). Design tokens and component choices should fit that stack, not fight it.
- `docs/DECISIONS.md` — if present, honor **ADR-3 (accessibility)**: the design must meet at least the committed WCAG bar. Don't propose a weaker one.

## Approach

1. **Establish the design direction.** Confirm the product's tone and audience, then propose an opinionated visual direction (mood, density, personality — e.g., "calm, editorial, lots of whitespace" vs "dense, utilitarian, data-first"). Get the user to confirm or redirect before going wide. Don't open with a blank canvas.

2. **Define the design tokens.** A concrete, named set the implementation and Claude Design both consume:
   - Color (semantic roles: background, surface, primary, on-primary, muted, border, success/warn/danger — not just hex swatches)
   - Typography scale (families, sizes, weights, line-heights)
   - Spacing scale, radius scale, elevation/shadow steps
   - If `ARCHITECTURE.md` names a system (shadcn/ui, Tailwind, native), express tokens in that system's vocabulary.

3. **Inventory the components.** List the reusable components the screens need (Button, Input, Card, Nav, Modal, EmptyState, …), each with its variants and states (default/hover/focus/disabled/loading). Pull the set from PRODUCT.md's flows, not from imagination.

4. **Lay out the key screens.** For each primary screen/flow in PRODUCT.md: purpose, layout structure (regions/hierarchy), the components it uses, and the **full state set** — populated, empty, loading, error. Empty/error/loading states are first-class, not afterthoughts; `tstack-build` will expect them.

5. **Specify accessibility patterns.** Concrete patterns that satisfy ADR-3: focus order, keyboard navigation, contrast adherence to the token set, labelling, motion-reduction. These become the a11y acceptance criteria downstream.

6. **Write `docs/2 - Specs/design.md`.** Two clearly separated parts (label them explicitly):
   - **For the human reader** — the spec above (direction, tokens, components, screens, a11y), readable and reviewable.
   - **For Claude Design** — the ready-to-paste hand-off material (next section). Mark this section so the user knows exactly what to drop into Claude Design.

   If `docs/2 - Specs/` doesn't exist, create it. Match the formatting of any existing specs in that folder. Commit: `docs: add design spec for {product/screen}`.

## Claude Design hand-off material (the headline output)

Under a clearly marked **"For Claude Design"** section, produce material the user can paste directly into Claude Design to generate or iterate the UI:

- **Per-screen generation prompts** — one self-contained prompt per key screen: its purpose, layout intent, the components it uses, the content and every state (populated/empty/loading/error), and the visual direction/tone. Written so it stands alone when pasted.
- **The design-token set** — colors, type, spacing, radius, elevation, in copy-pasteable form.
- **The component inventory** — names, variants, states.
- **Brand/style constraints** — any fixed rules (logo, palette locks, voice) Claude Design must respect.

Keep this section verbatim-pasteable: a reader should be able to copy one screen's prompt + the token block into Claude Design without editing.

## Reference handoff

This skill pairs with the broader design tooling available in the environment:
- The **frontend-design** skill — for translating the spec into production-grade component code.
- The **Figma** skills — for building or syncing a design system / screens in Figma from this spec.

Lean on those once the spec and Claude Design material exist; this skill defines *what* to design, they help *produce* it.

For a realistic example of the two-part output — the human-readable spec and the "For Claude Design" paste-ready prompts — read `references/example-output.md`.

## Handoff

This skill is off-chain, so there's no required next step. End with what you produced and the optional follow-ups:

> Design spec written to `docs/2 - Specs/design.md` — {n} screens, {n} components, token set, and a "For Claude Design" section ready to paste.
>
> Optional next steps: paste the per-screen prompts into **Claude Design** to generate the UI; or, if this surfaced UI work that isn't yet on the roadmap, run `tstack-specify-feature` to add it (or `tstack-roadmap` to re-sequence). Nothing here forces a chain step.
