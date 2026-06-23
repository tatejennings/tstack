---
name: refresh-lifecycle
description: >-
  Repo-local maintenance skill that regenerates THIS repo's README lifecycle infographic
  (images/lifecycle.jpg) and agentic-loop illustration (images/agentic-loop.jpg) when the
  TStack chain changes. Use when the user says "/refresh-lifecycle", "regenerate the
  lifecycle graph/diagram/infographic", "the chain changed — update the diagram", or "the
  lifecycle image is out of date". Runs a no-cost DRIFT CHECK first (diffs the live
  skills/tstack-* set against the rendered-graph manifest in images/BRANDING.md §11) and only
  regenerates if stale. ONLY for the TStack plugin repo itself — NOT a consumer-facing skill,
  NOT tstack-status/tstack-wrap. Depends on the Higgsfield MCP being connected. Input is
  skills/tstack-*, images/BRANDING.md, README.md; output is updated image(s) + manifest +
  README alt-text + a CHANGELOG entry. Hands off to nothing (commit via /ship or by hand).
---

# refresh-lifecycle

You are maintaining **this repository's** README diagrams — the lifecycle infographic
(`images/lifecycle.jpg`) and the agentic-loop illustration (`images/agentic-loop.jpg`). These
are AI-generated raster images, so keeping them in sync with the chain is a *re-generate*, not
a redraw.

This skill is **repo-local and not shipped** — it lives in `.claude/`, outside `skills/`, so
the plugin loader never publishes it (same as `/upkeep` and `/ship`). It maintains the plugin's
own README; it is useless in a consumer project. Do not move it under `skills/`.

**`images/BRANDING.md` is the source of truth for content.** The brand Style Block (§10), the
generation recipe (§11), and the rendered-graph manifest (§11 → *Rendered graph*) live there.
This skill *orchestrates* those steps — it does **not** restate the prompts. If a prompt or the
composition needs to change, edit `BRANDING.md §11` and follow it; don't fork the wording here.

---

## Preflight (fail-closed)

Stop with a clear message if any of these fail:

1. **Right repo.** `images/BRANDING.md` and `skills/tstack-*` both exist. If not, you're not in
   the TStack plugin repo — refuse.
2. **Higgsfield available.** Run `ToolSearch` for `generate_image` / `media_upload`. If the
   Higgsfield MCP isn't connected, **do not improvise another generator** — report that
   regeneration needs Higgsfield and point the user at `images/BRANDING.md §11` for the manual
   path. (The drift check in Step 1 still works without it — run that and report.)
3. **Tree awareness.** This skill overwrites tracked images. You may run with a dirty tree, but
   note it; never commit (that's `/ship` or the user's call).

---

## Step 1 — Drift check (always; no credits)

This is the cheap gate. Run it first and **never spend credits if the graph is already current.**

1. List `skills/tstack-*`. For each, read its `SKILL.md` frontmatter `description` and classify
   its role using the cross-skill contracts in `CLAUDE.md`:
   - **main chain**: discover, product, architect, roadmap, plan, build
   - **iteration**: specify (hands into plan)
   - **on-ramp**: ingest (feeds product)
   - **off-chain**: design, status, wrap, autopilot
   Classify by *role*, not by exact directory name — a rename (e.g. `tstack-plan` →
   `tstack-plan-milestone`) keeps the same node and the same one-word image label (`PLAN`).
2. Read the **Rendered graph** manifest in `images/BRANDING.md §11` — the canonical record of
   what the current images depict.
3. **Diff** the live set/roles against the manifest. Report one of:
   - ✅ **In sync** — every node and role matches. State that the images are current and
     **stop**. Offer a forced regen only if the user explicitly wants a style refresh (say so;
     it still costs credits).
   - ⚠️ **Stale** — list exactly what diverged: skill **added** / **removed** / **role changed**
     (these change the picture) vs a pure **rename** with an unchanged one-word label (the image
     itself is unaffected — only the manifest/README text need a text update, *no* regen).
4. Decide scope from the diff:
   - `lifecycle.jpg` depicts the **whole** graph (chain + ingest + specify + off-chain shelf) →
     regenerate it for any node/role change.
   - `agentic-loop.jpg` depicts **only** the 6-step chain + plan⇄build loop → regenerate it only
     if a *main-chain* node changed; ingest/specify/off-chain changes don't touch it.

If only text drifted (rename, unchanged label), skip to Step 4 (records only).

---

## Step 2 — Regenerate (only if a node/role changed, or forced)

Follow `images/BRANDING.md §11` exactly. In brief:

1. **Cost confirm.** Call `generate_image` with `get_cost: true` for one image and tell the user
   the total (≈2 credits/image × 2 variants × affected images). Get a go-ahead before
   submitting real jobs — this spends the user's Higgsfield credits.
2. **Mascot lock.** Ensure `images/mascot-ref.png` exists (if missing, re-crop the banner's left
   mascot per §11). Upload it once: `media_upload` → `PUT` the bytes to the presigned URL →
   `media_confirm`. Keep the returned `media_id`.
3. **Generate.** `nano_banana_pro`, `aspect_ratio: "16:9"`, `resolution: "2k"`, `count: 2`, with
   `medias: [{ value: <media_id>, role: "image" }]`. Use the §11 composition prompt(s) for the
   affected image(s), with the node list updated to the new chain. **One short word per node** —
   dense labels garble.
4. Poll, download variants, and **present them for the user to pick** (don't auto-select).

---

## Step 3 — Post-process (per BRANDING.md §11)

For each chosen image: auto-trim the uniform cream margin → resize (`lifecycle` ~1800px wide,
`agentic-loop` ~1400px) → save progressive **JPEG q90** → overwrite the file **in place** at
`images/lifecycle.jpg` / `images/agentic-loop.jpg`. Verify the result visually (Read it) to
confirm no label was clipped and the text is legible.

---

## Step 4 — Sync the records

Whatever changed (regen or text-only), leave the repo self-consistent:

1. **Manifest.** Update the *Rendered graph* block in `images/BRANDING.md §11` to the new node
   set/labels/roles. This is mandatory — it's what the next drift check reads.
2. **README alt-text.** If nodes changed, update the affected image's alt text in `README.md`
   (the lifecycle and/or agentic-loop `![…]` line) so the description matches the picture.
3. **CHANGELOG.** Add an entry under `## [Unreleased]` (per `CLAUDE.md`'s "always update
   CHANGELOG" rule) naming what changed and why (e.g. "regenerated lifecycle.jpg — added
   tstack-foo off-chain companion").

---

## Handoff

Report: drift verdict, which image(s) were regenerated (or that none were), credits spent, and
the records updated. Remind the user nothing is committed — they commit by hand or via `/ship`.
Do **not** chain into a release.
