# Example output — tstack-design

A trimmed `docs/3 - Design/` set for a habit-tracking web app (stack from ARCHITECTURE.md if present: Next.js + shadcn/ui + Tailwind; ADR-3: WCAG 2.1 AA). Calibrate against the level of specificity and the file split — `design.md` is the canonical UX + visual spec, `design-tokens.json` is the machine source, `claude-design-prompts.md` is paste-ready hand-off. Not the exact product.

```
docs/3 - Design/
  design.md                 ← UX + visual spec (below)
  design-tokens.json        ← DTCG tokens (machine source)
  claude-design-prompts.md  ← paste into Claude Design
  previews/                 ← HTML mockups / screenshots (Route B/C or linked)
```

---

## `design.md`

### Part 1 — UX spec

**Information architecture** — two top-level areas: *Today* (the default) and *Habits* (manage/library). Settings is a leaf off the profile menu. Content model: a Habit has a name, cadence, color, and a streak; a Check is a (habit, date) toggle.

**Navigation structure** — bottom tab bar on mobile / left rail on web: `Today` · `Habits` · `Profile`. Modal layer for create/edit. No deep nesting — every primary task is ≤2 taps from Today.

**User flows**
- *Check off a habit (happy path):* Today → tap a habit card's check control → card animates to completed, streak increments. Alternate: undo within the card.
- *Add a habit:* Today → "Add habit" (FAB) → modal (name, cadence, color) → Save → returns to Today with the new card. Alternate: validation error on empty name.

**Screen / page breakdown**
- **Dashboard (Today)** — purpose: see today's habits and check them off. Layout: top bar (greeting + date) · vertical list of HabitCard · floating "Add habit". States: **populated** (cards), **empty** (illustration + "Add your first habit" CTA), **loading** (3 skeleton cards), **error** ("Couldn't load — retry").
- **Add Habit (modal)** — purpose: create a habit in <20s. Layout: modal · name field · cadence selector · color pick · Save/Cancel. States: default, **error** (inline validation on empty name), **loading** (save in flight, button spinner).

**Interaction patterns** — checkmark toggles on tap / Space / Enter with optimistic update + rollback on failure; inline field validation on blur; toasts for save/error; streak animation respects `prefers-reduced-motion`.

### Part 2 — Visual spec

**Design direction** — calm, encouraging, low-pressure. Generous whitespace, soft surfaces, one accent color used sparingly for progress. A supportive coach, not a productivity drill sergeant.

**Design tokens (summary — full set in `design-tokens.json`)**
```
color.bg #FAFAF7 · surface #FFFFFF · primary #4F46E5 · on-primary #FFFFFF
color.border #E7E5E4 · muted-fg #78716C · success #15803D · danger #C2410C
type Inter / system-ui · scale 12·14·16·20·28·40 (body lh 1.5)
space 4·8·12·16·24·32·48 · radius sm6/md10/lg16/full · elevation e1·e2·e3
```

**Component inventory**
| Component | Variants | States |
|---|---|---|
| Button | primary, ghost, destructive | default, hover, focus, disabled, loading |
| HabitCard | active, completed-today, paused | default, hover, focus |
| StreakBadge | active, broken | — |
| EmptyState · Toast | — / success, error | — |

**Accessibility (satisfies ADR-3, WCAG 2.1 AA)** — keyboard path tab-order top-bar → cards → add; text/`muted-fg` on `bg` ≥ 4.5:1 (verify with axe in CI); modal traps focus and restores to trigger, `Esc` closes; `prefers-reduced-motion` disables the streak animation.

---

## `design-tokens.json` (W3C DTCG — excerpt)

```json
{
  "color": {
    "bg":        { "$type": "color", "$value": "#FAFAF7" },
    "primary":   { "$type": "color", "$value": "#4F46E5" },
    "on-primary":{ "$type": "color", "$value": "#FFFFFF" },
    "danger":    { "$type": "color", "$value": "#C2410C" }
  },
  "space":  { "2": { "$type": "dimension", "$value": "8px" },
              "4": { "$type": "dimension", "$value": "16px" } },
  "radius": { "md": { "$type": "dimension", "$value": "10px" } }
}
```
Claude Code translates this into the chosen stack's config (Tailwind `theme` / CSS vars / Swift tokens) at build time; `tstack-build` can compute contrast ratios from it as a command-verifiable a11y check.

---

## `claude-design-prompts.md` (Route A — paste into Claude Design)

> Paste any screen prompt below, plus the token block, directly into Claude Design.

**Token block**
```
Background #FAFAF7, Surface #FFFFFF, Primary #4F46E5 (text on it #FFFFFF),
Border #E7E5E4, Muted text #78716C, Success #15803D, Danger #C2410C.
Inter / system-ui. Sizes 12–40, body line-height 1.5. Spacing 4–48 scale.
Radius sm6/md10/lg16. Soft single-layer card shadows.
```

**Screen prompt — Dashboard (high-fidelity)**
```
Design a habit-tracker dashboard, calm and encouraging with generous whitespace.
Top bar: greeting ("Good morning, Sam") and today's date. Below: a vertical list
of habit cards, each with name, a streak badge, and a large circular check control.
A floating "Add habit" button bottom-right (primary indigo). Show four versions:
(1) populated with 5 habits, two checked; (2) empty with a friendly illustration and
"Add your first habit" CTA; (3) loading with 3 skeleton cards; (4) error "Couldn't
load — retry". Use the token block. Ensure AA contrast and a clear keyboard focus ring.
```
*(For a **wireframe** pass, the same prompt drops color/brand: "low-fidelity grayscale wireframe, boxes-and-labels, no styling — focus on layout, hierarchy, and the four states.")*

**Brand constraints**
```
Accent indigo (#4F46E5) is the only saturated color — progress and primary action only.
No red except genuine errors/destructive actions. Voice: warm and brief, never guilt-tripping.
```

---

## Adopted draft (when `tstack-ingest` staged existing designs)

`docs/_adopted/design.draft.md` — quarantined, unratified; `tstack-design` ratifies it into the set above and deletes it:

```markdown
## Adoption status
- Source: ./mockups/*.html + brand.pdf, adopted 2026-06-25.
- **Unratified draft — not a canonical doc.** Ratify via `tstack-design`, which authors `docs/3 - Design/` from this.

## Open gaps
- Mockups show the populated state only — empty / loading / error states missing.
- No focus order or keyboard path specified — needs an a11y pass to satisfy ADR-3.
- Colors are embedded in HTML; no token names — confirm the semantic roles for `design-tokens.json`.
```
