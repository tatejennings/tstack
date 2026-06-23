# TStack — Brand & Design System

> Source of truth for TStack's visual identity. Derived from `images/banner.jpg`.
> Use this when regenerating the banner, the lifecycle infographic, social cards, or
> any other TStack visual so everything stays on-brand. The **Style Block** at the
> bottom is the copy-paste prompt fragment for AI image generation (Higgsfield /
> nano-banana-pro / Recraft etc.).

---

## 1. Brand essence

TStack is a developer tool, but its personality is **warm, human, and hand-made** — the
opposite of a cold enterprise SaaS. The visual language is a **hand-drawn crayon /
colored-pencil sketch on textured cream paper**: friendly, approachable, a little playful,
clearly made by a person. It says *"great apps start with great docs"* without feeling
corporate.

Three adjectives to hold onto: **warm, crafted, calm.**

---

## 2. Color palette

| Role | Name | Hex | Notes |
|---|---|---|---|
| Primary | **TStack Orange** | `#EA8A57` | The signature terracotta. Logo "T", mascots, key underlines, accents. |
| Primary (bright) | **Ember** | `#F59153` | Lighter, more saturated orange for highlights / fills in light. |
| Primary (deep) | **Clay** | `#D9743F` | Darker shade for shadows, outlines on orange, emphasis. |
| Ink | **Charcoal** | `#1A1A18` | Near-black, slightly warm. Headlines ("Stack"), body text, sketch linework. |
| Surface | **Paper** | `#F0EDE0` | The cream paper ground. Everything sits on this. |
| Surface (light) | **Paper Light** | `#F7F3E8` | Lighter paper / page highlights, document cards. |
| Neutral | **Pencil Gray** | `#B9B4A6` | Muted gray pencil strokes, secondary linework, faint grids. |
| Accent | **Sage** | `#9CAA72` | Sparingly — plants, "go" / positive ticks. Never a primary. |

**Usage ratio (rough):** ~70% paper, ~18% ink linework, ~10% orange, ~2% sage. Orange is a
*spotlight* color — overusing it kills the warmth. Keep large fields paper-cream.

---

## 3. Typography

The banner uses a **hand-lettered / marker** look, not a clean geometric sans.

- **Display / headlines:** chunky hand-drawn marker letterforms with slightly uneven weight
  (the "TStack" wordmark). Ink charcoal, with the leading "T" in orange.
- **Body (in-illustration):** a lighter hand-printed style, like neat felt-tip handwriting.
- **Emphasis:** key words are **underlined in a single orange marker stroke** (e.g.
  _foundation_, _blueprint_, _clarity and consistency_). This underline is a signature move —
  reuse it to highlight terms.

For real UI / README text (outside images), the repo just uses GitHub's default Markdown
font — keep the hand-drawn look confined to the illustrations.

---

## 4. Logo & wordmark

- **Mark:** three offset stacked rounded squares (a literal "stack") — the **top tile orange**,
  the lower tiles cream/white with charcoal outlines and a soft drop shadow. Reads as layers /
  a documentation stack.
- **Wordmark:** "**T**Stack" — orange **T**, charcoal **Stack**, set tight, hand-drawn marker.
- **Lockup:** mark sits to the **left** of the wordmark, vertically centered.
- **Clear space:** keep at least one stacked-tile height of empty paper around the lockup.

---

## 5. Mascot

The friendly orange characters are the heart of the brand.

- **Form:** a round **orange disc/ball** body (think a glowing orange coin) with a **white
  multi-point sunburst / spark** on the face — a stylized starburst that doubles as eyes/smile,
  an affectionate nod to the Claude/Anthropic spark. Small **white gloved cartoon hands**, simple
  black dot eyes, cheerful expression.
- **Personality:** helpful, eager, collaborative — shown working together at a desk over
  documents, gesturing, holding pens, sipping from `{}` code-stamped mugs.
- **Use:** as guides/narrators of a process. They can point at steps, carry documents, or react.
  Keep them few (1–3) and expressive rather than crowding the scene.

---

## 6. Iconography & motifs

Simple, single-weight hand-drawn line icons in charcoal with occasional orange fill:

- 🚀 rocket (initialize / launch), ☑️ checklist (document), 🎯 target (build better)
- Document pages with wavy "text" lines, little flow boxes, `</>` and `{ }` code marks
- Coffee mugs (often stamped `{}`), sticky notes (pale yellow), potted plants, pushpins
- Faint pencil-sketched UI in the background (an "APP PLAN" whiteboard with ticked items)

These props build the "thoughtful workspace" scene. Scatter them lightly; don't clutter.

---

## 7. Medium & texture

- **Ground:** visible **paper grain / canvas texture**, cream.
- **Line:** crayon / colored-pencil / felt-tip — slightly rough, organic, **never vector-crisp**.
- **Shading:** soft directional pencil hatching; gentle, diffuse shadows (no hard edges, no gloss).
- **Edges:** hand-drawn wobble. Avoid perfectly straight lines, gradients, glows, or 3D renders.

---

## 8. Layout principles

- **Generous cream negative space.** Let the paper breathe.
- **Left-to-right reading flow** for process/story scenes (mascots and docs arranged along a line).
- **One focal orange moment** per composition; supporting elements stay charcoal-on-cream.
- **Underline-emphasis** for the 2–4 words that carry the message.
- Banner/infographic aspect ratio: wide **~2.5:1** (`16:9` or `21:9` works for README headers).

---

## 9. Do / Don't

**Do**
- Keep it hand-drawn, warm, and paper-textured.
- Reserve orange for the few things that matter.
- Use the mascots to *guide* the viewer through a process.
- Keep any text short and legible — large, few words.

**Don't**
- Don't go flat-vector, corporate-gradient, neon, glassy, or 3D-rendered.
- Don't fill the page with orange.
- Don't crowd in tiny unreadable labels (AI renderers garble dense text — keep nodes sparse).
- Don't introduce new accent colors beyond the palette above.

---

## 10. Style Block (copy-paste for AI image generation)

Paste this as the *style* portion of any TStack image prompt, then add the specific
composition/content after it:

```
STYLE: warm hand-drawn illustration, crayon and colored-pencil on textured cream paper
(#F0EDE0), visible paper grain. Friendly, cozy, slightly playful — not corporate.
Charcoal (#1A1A18) hand-sketched linework with organic wobble; soft pencil hatching and
gentle diffuse shadows; NO vector-crispness, NO gradients, NO glow, NO 3D render.
Signature accent color is a warm terracotta orange (#EA8A57 / #F59153) used sparingly for
emphasis only — most of the canvas stays cream. Key words underlined with a single orange
marker stroke. Optional mascots: round orange disc characters with a white sunburst/spark
face, small white gloved hands, cheerful. Hand-lettered marker typography for any headings;
neat felt-tip handwriting for labels. Keep all text SHORT, LARGE, and clearly legible.
```

---

*Maintaining the look:* when the lifecycle changes, regenerate the infographic with the Style
Block above plus the current step list — don't hand-redraw. The exact recipe is in §11.

---

## 11. Regenerating the lifecycle infographic (workflow)

The README images (`images/lifecycle.jpg` and `images/agentic-loop.jpg`) are **AI-generated**,
not hand-drawn, so they can be re-made from a prompt whenever the chain changes. This is the
deliberate trade for dropping the Mermaid diagram: editing is a re-generate, not a redraw.

**Tooling:** Higgsfield MCP → `generate_image`, model **`nano_banana_pro`** (chosen for accurate
text rendering and diagrams). Settings: `aspect_ratio: "16:9"`, `resolution: "2k"`, `count: 2`
(generate a couple, pick the best). Cost ≈ 2 credits per image.

**Character match:** the mascot is locked to the banner by passing **`images/mascot-ref.png`**
(a tight crop of the banner's left mascot) as a reference image. Upload it once per session with
`media_upload` → `PUT` the bytes → `media_confirm`, then pass the returned `media_id` as
`medias: [{ value: <media_id>, role: "image" }]`. Without the reference, the model invents a
slightly different sunburst face.

**Prompt = character lock + Style Block (§10) + composition.** Keep node labels to ONE short word
each — AI renderers garble dense text. The composition used for the current `lifecycle.jpg`:

```
[character lock: "Use the EXACT character from the reference image and do not redesign it: a
round terracotta-orange disc body with a bold WHITE multi-point sunburst/spark on its face, two
small black dot eyes, a cheerful open smile, and small white gloved cartoon hands."]

[Style Block from §10]

COMPOSITION — a wide horizontal pipeline infographic. Title top center in orange marker:
"TStack Lifecycle". A left-to-right row of four paper note-cards: "DISCOVER", "PRODUCT",
"ARCHITECT", "ROADMAP", joined by short orange arrows. To the right, two cards "PLAN" and "BUILD"
inside a small rounded loop of two orange arrows, note beneath: "repeat per milestone". The
reference mascot stands beside the row handing a document along the line. A small curved orange
return arrow loops up into PLAN, labeled "SPECIFY · add features". Bottom: faint label
"optional · anytime" above four paper tags: "DESIGN", "STATUS", "WRAP", "AUTOPILOT". Far left, a
small entry arrow noted "have docs? → INGEST". Generous cream negative space.
```

For `agentic-loop.jpg`, swap the composition for: title "The agentic loop"; a short ascending
ramp of the four setup cards labeled "one-time setup" feeding a large central circular loop of
two orange arrows between "PLAN" and "BUILD" ("the build loop" in the middle); the mascot to the
right holding a ticked checklist; caption "fresh session each milestone".

**Post-process** (before committing): auto-trim the uniform cream margin, resize to ~1800px wide
(lifecycle) / ~1400px (loop), save as progressive JPEG q90. Keeps each file < 300 KB and the text
crisp enough to read. (The original generation was done with a short PIL script: diff against the
paper color → `getbbox` → crop with ~24px pad → `LANCZOS` resize → `JPEG quality=90`.)

**When the chain changes:** edit the node list in the composition above, regenerate with the same
model/settings + the mascot reference, post-process, and replace the file in place. Update this
section if the composition itself changes.

### Rendered graph (what `lifecycle.jpg` currently depicts)

This is the canonical record of the nodes baked into the current images — the diffable source of
truth the `/refresh-lifecycle` skill checks the live `skills/tstack-*` set against. **Keep this
list in lockstep with the rendered image:** if you regenerate with a different node set, update
this block in the same change.

```
main chain (row):   discover → product → architect → roadmap → plan ⇄ build
iteration arrow:    specify → plan        (labeled "SPECIFY · add features")
on-ramp (entry):    ingest                (labeled "have docs? → INGEST", feeds product)
off-chain shelf:    design · status · wrap · autopilot   (labeled "optional · anytime")
loop note:          "repeat per milestone" on the plan ⇄ build cycle
```

Total: 12 skills, matching `skills/tstack-*`. If a skill is added/removed/renamed, or its
chain-vs-on-ramp-vs-off-chain role changes, the graph is **stale** and must be regenerated.
