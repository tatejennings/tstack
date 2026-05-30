# Example output — tstack-design

A trimmed `docs/2 - Specs/design.md` for a habit-tracking web app (stack from ARCHITECTURE.md: Next.js + shadcn/ui + Tailwind; ADR-3: WCAG 2.1 AA). Calibrate against the level of specificity and the explicit human / Claude-Design split — not the exact product.

---

## Part 1 — For the human reader

### Direction
Calm, encouraging, low-pressure. Generous whitespace, soft surfaces, one accent color used sparingly for progress. Personality: a supportive coach, not a productivity drill sergeant.

### Tokens
```
color.bg            #FAFAF7      color.danger        #C2410C
color.surface       #FFFFFF      color.success       #15803D
color.primary       #4F46E5      color.border        #E7E5E4
color.on-primary    #FFFFFF      color.muted-fg      #78716C
type.family         Inter / system-ui
type.scale          12 · 14 · 16 · 20 · 28 · 40   (1.5 line-height body, 1.2 headings)
space.scale         4 · 8 · 12 · 16 · 24 · 32 · 48
radius              sm 6 · md 10 · lg 16 · full
elevation           e1 subtle card · e2 popover · e3 modal
```

### Components
| Component | Variants | States |
|---|---|---|
| Button | primary, ghost, destructive | default, hover, focus, disabled, loading |
| HabitCard | active, completed-today, paused | default, hover, focus |
| StreakBadge | active, broken | — |
| EmptyState | — | — |
| Toast | success, error | — |

### Key screens

**Dashboard** — purpose: see today's habits and check them off.
- Layout: top bar (greeting + date) · vertical list of HabitCard · floating "Add habit" button.
- States: **populated** (list of cards), **empty** (illustration + "Add your first habit" CTA), **loading** (3 skeleton cards), **error** ("Couldn't load your habits — retry").

**Add Habit** — purpose: create a habit in <20s.
- Layout: modal · name field · cadence selector · color pick · save.
- States: default, **error** (inline validation on empty name), **loading** (save in flight, button shows spinner).

### Accessibility (satisfies ADR-3, WCAG 2.1 AA)
- Full keyboard path: tab order top-bar → cards → add button; checkmark toggles on Space/Enter.
- All text/`color.muted-fg` on `color.bg` meets ≥4.5:1; verify with axe in CI.
- Modal traps focus, restores to trigger on close; `Esc` closes.
- `prefers-reduced-motion` disables the streak animation.

---

## Part 2 — For Claude Design

> Paste any screen prompt below, plus the token block, directly into Claude Design.

**Token block**
```
Background #FAFAF7, Surface #FFFFFF, Primary #4F46E5 (text on it #FFFFFF),
Border #E7E5E4, Muted text #78716C, Success #15803D, Danger #C2410C.
Inter / system-ui. Sizes 12–40, body line-height 1.5. Spacing 4–48 scale.
Radius sm6/md10/lg16. Soft single-layer card shadows.
```

**Screen prompt — Dashboard**
```
Design a habit-tracker dashboard, calm and encouraging with generous whitespace.
Top bar: greeting ("Good morning, Sam") and today's date. Below: a vertical list
of habit cards, each showing habit name, a streak badge, and a large circular
check control. A floating "Add habit" button bottom-right (primary indigo).
Show four versions: (1) populated with 5 habits, two already checked;
(2) empty state with a friendly illustration and "Add your first habit" CTA;
(3) loading with 3 skeleton cards; (4) error with a "Couldn't load — retry" panel.
Use the token block. Ensure AA contrast and a clear keyboard focus ring.
```

**Screen prompt — Add Habit modal**
```
Design an "Add habit" modal over a dimmed dashboard. Fields: habit name (text),
cadence (daily / weekdays / custom segmented control), color swatch picker.
Primary "Save habit" + ghost "Cancel". Show three versions: default empty,
inline validation error on the name field, and a saving state (button spinner,
fields disabled). Use the token block; modal corners radius lg, e3 elevation.
```

**Brand constraints**
```
Accent indigo (#4F46E5) is the only saturated color — use it for progress and
the primary action only. No red except for genuine errors/destructive actions.
Voice: warm and brief, never guilt-tripping.
```
