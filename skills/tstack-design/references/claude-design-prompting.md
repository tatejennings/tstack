# Prompting Claude Design — reference

> **Source of truth:** `SKILL.md` is authoritative for `tstack-design`'s behaviour. This reference
> backs the **Route A** brief — what Claude Design is, and how to write `claude-design-prompts.md`
> so it pastes cleanly and keeps components consistent. If this and `SKILL.md` ever disagree,
> follow `SKILL.md`.

## What Claude Design is

- A **browser** design tool at **`claude.ai/design`** — runs on **Mac, Windows, and Linux**.
  **There is no native app to install** (any `.dmg`/`.exe` claiming to be it is not from Anthropic).
- Launched April 2026 (Anthropic Labs, research preview), powered by **Claude Opus 4.7**; available
  on Pro / Max / Team / Enterprise plans.
- A **chat + canvas** surface: describe a screen, it drafts, you iterate in chat or on the canvas.
  Goes wireframe → high-fidelity → interactive prototype; exports to Canva / PDF / PPTX / HTML and
  hands off to Claude Code.
- **Its platform is unrelated to the product's target platform.** It designs native macOS/iOS apps,
  web apps, and CLI dashboards alike. Never deprioritize Route A because the product is "a Mac app"
  or "not a web app" — that distinction does not exist here.

## Fidelity & credits

- **Start at wireframe.** It's cheaper, faster to iterate, and the right altitude when design runs
  *before* `tstack-architect` (the architecture-informing pass). Rerun **high-fidelity** once flows settle.
- Re-running the same prompts at a higher fidelity later should change **only** the fidelity line in
  the kickoff — everything else stays put.

## The kickoff prompt (paste once, first)

One message that establishes the **design system as the shared source of truth** for the whole session:

- The **tokens** (color roles, type scale, spacing, radius, elevation) with the rule *never hard-code values*.
- The **brand & voice** constraints and the **accessibility bar** (the WCAG level from ADR-3).
- The **component inventory as a shared library** — name the components and their variants/states.
- End with a planning ask, e.g. *"Before designing, ask me any questions about the UX/UI that would
  make these screens as effective as possible."* This makes Claude Design plan before spending tokens.

## Per-screen prompts (one message each, send as-is)

Each is **self-contained** but references the kickoff rather than re-pasting tokens. Include:

- **Purpose** of the screen, and **layout regions / hierarchy**.
- The **full state set as separate frames** — populated, empty, loading, error (Claude Design produces
  thoughtful UX only when states are explicit).
- The screen's **accessibility** requirement (focus order, keyboard path, focus trap for modals).
- The standing reuse line: *"using the components from my first message — reuse, don't recreate; if you
  extend a shared component here, apply that change across all pages."*

Keep each prompt one block the user pastes and sends — **never** loose fragments they must stitch together.

## Cross-page consistency (the part that's easy to get wrong)

Pasting screens one at a time makes Claude Design create each as an **independent page**, so a component
refined on a later screen leaves earlier screens stuck with the placeholder version. The documented fix:

1. **Establish the component library first** (the kickoff does this — the "design system first" pattern).
2. **Every screen prompt reuses it** ("build using the components defined above; reuse, don't recreate").
3. **Propagate refinements** — when a component changes, tell Claude Design to *"apply this across the
   full design,"* its explicit back-propagation capability.
4. **Run a final consistency pass** that reconciles all pages to one canonical version of each component.
5. **No cross-session memory** — re-paste the kickoff at the start of any new session.

Resist re-editing the design system after every screen; update it only when a pattern recurs, a token
isn't working, or you see Claude Design drifting from the intent.

## Refinement modes

- **Inline canvas comments** for component-level tweaks ("is this progress indicator clear?").
- **Direct canvas edits** for quick spacing / color / position changes.
- **Chat** for structural changes (layout, navigation, color scheme) — then *"apply across the full design."*

## Heavier-weight alternative (don't wire in by default)

Claude Design's **`/design-sync`** (runs in Claude Code) does two-way design-system sync — *pull* imports
the codebase's real components into Claude Design so every generated screen uses them; *push* sends
implemented code back to the canvas. It's a stronger consistency mechanism than manual re-pasting, but it
adds a tool dependency — mention it to users who want it; keep this skill's default the paste-based loop.

## Sources

- [Introducing Claude Design by Anthropic Labs](https://www.anthropic.com/news/claude-design-anthropic-labs)
- [Get started with Claude Design — Claude Help Center](https://support.claude.com/en/articles/14604416-get-started-with-claude-design)
- [Set up your design system in Claude Design — Claude Help Center](https://support.claude.com/en/articles/14604397-set-up-your-design-system-in-claude-design)
- [Claude Design: the complete setup & workflow guide (2026) — Design Systems Collective](https://www.designsystemscollective.com/claude-design-the-complete-setup-workflow-guide-2026-5de41e62fd4c)
- [Prompting best practices — Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
