---
name: tstack-ingest
description: Adopts a project that already has documents into TStack — reads pre-existing material (a written PRD, discovery notes, design or architecture docs in their own format, not TStack's), reflects intent back, classifies coverage, and distills it into a quarantined draft at docs/_adopted/ that the owning chain skill then ratifies into a canonical doc. Use when the user arrives with docs already written and says "adopt my project", "I already have a PRD/docs", "import my existing requirements", "we didn't start in TStack", or points you at a file/folder of docs. Reachable directly or via a tstack-discover hand-off — same outcome either way. Do not use for a rough unwritten idea (that's tstack-discover), to change an already-TStack project that has docs/PRODUCT.md (that's tstack-specify), or to merely report state (that's tstack-status, which is read-only — ingest writes). Input is the user's existing docs; output is a draft at docs/_adopted/ + a coverage report. Hands off to tstack-product.
---

# tstack-ingest

You are running TStack's adoption on-ramp. The user already has product thinking written down — a PRD, discovery notes, a design or architecture doc — but it's in their format, not TStack's. Your job is to **map what they have onto the TStack doc set, distill it into a draft, and route them to the right stage of the chain** so they resume mid-stream instead of starting over.

The hard rule that keeps the loop trustworthy: **you never write a canonical doc.** You write only to `docs/_adopted/` — a quarantine that satisfies no chain skill's prerequisite. The owning skill (`tstack-product`) authors the real `docs/PRODUCT.md` from your draft, through its own verification gate. You distill and route; you do not ratify. Never fabricate — carry across what the source says, and name what it's missing as an explicit gap.

## Prereq check (soft)

- **Foreign material + no owner-authored `docs/PRODUCT.md`** → primary path (below).
- **`docs/PRODUCT.md` already exists** (owner-authored) → this is already a TStack project. Stop and redirect: to *change* docs, run `tstack-specify`; to *inspect* state, run `tstack-status`. Adopting more foreign material into an established project is out of scope for v1 — don't reconcile here.
- **Nothing to adopt** (no written material, just a rough idea) → stop and point to `tstack-discover` for the discovery interview.

## Approach

Follow in order. Steps 3–5 are where the value is — don't skip to writing.

1. **Gather the source material.** Find what the user has:
   - **Scan** common locations one level deep: `docs/`, `doc/`, `notes/`, `spec/`, `specs/`, `design/`, and root `*.md` / `*.txt`. Present an inventory ("here's what I found") and ask the user to confirm which are real product/design/architecture docs.
   - **Accept pointers** — the user can name a file or folder directly (this is also how a `tstack-discover` hand-off arrives).
   - **Off-disk docs** (Notion, Google Docs, Confluence): ask the user to export into the repo or give a `WebFetch`-able URL **before** falling back to paste — pasting a large doc into chat is lossy and burns context budget.
   - **Large sources:** work section by section rather than ingesting one giant blob.
   - **PDF/binary:** ask the user to export to text or paste the relevant parts; never transcribe from a binary you can't read cleanly.

2. **Reflect intent back (mandatory).** Before distilling anything, summarize what the material says they're building — product, user, core value — and ask: **"Is this still current?"** A rigorous PRD can still describe a product the user has since pivoted away from. This one question catches a stale doc that the coverage check alone would wave through.

3. **Map source → TStack stages, at the section level.** A single file often spans stages — a "spec" mixing requirements and technical design. Split it: requirements/flows/data → **product material** (becomes the draft); architecture/stack/API/decisions → **staged notes** (reference only, routed to `tstack-architect`). Surface the proposed seam and let the user confirm before you act. When a section is ambiguous, prefer staging it as notes (recoverable) over folding a guess into the draft.

4. **Assess coverage.** For the product material, classify against what `tstack-product` requires, naming the *specific* missing elements (no opaque score):
   - **PRODUCT-grade** — features with user flows **and** testable acceptance criteria, defined data models (entities, relationships, field types), explicit v1 scope boundaries. Any non-deterministic (AI/LLM/ML) feature lacking an eval set / quality bar / fallback → name it as a gap.
   - **Discovery-grade only** — rough notes, soft or absent criteria, no data models. Still adoptable: it seeds a lower-fidelity draft, with the soft areas marked as gaps for `tstack-product` to harden.
   - Report it as a short Present / Thin / Missing summary (see `references/example-output.md`).

5. **Per-item approval.** Propose what you'll do with each piece of source, using the two-list shape (mirroring `tstack-specify`):

   ```
   Proposed adoption from "{source}":

   Distil into docs/_adopted/PRODUCT.draft.md:
   - [ ] Features + flows from {source §X} (PRODUCT-grade)
   - [ ] Data models from {source §Y} (PRODUCT-grade)
   - [ ] Onboarding flow from {source §Z} (Thin — no acceptance criteria; flagged as gap)

   Stage as notes for tstack-architect (docs/_adopted/architecture-notes.md):
   - [ ] Stack + infra choices from {source §W} — not transcribed; architect asks its ADR questions fresh

   Considered, not adopting (challenge any of these):
   - {source §V} — {one-line reason, e.g. "internal meeting notes, not requirements"}

   Reply approve / modify / reject per item, or "approve all".
   ```

   Surface any **source-vs-source contradictions** here (PRD says one data model, design doc implies another) and make the user resolve them — never silently pick a winner. Wait for explicit approval.

6. **Write the draft(s) to `docs/_adopted/`.** Create the directory if needed. The product draft is `docs/_adopted/PRODUCT.draft.md`; staged notes are `docs/_adopted/{topic}-notes.md`. Every draft opens with two **visible** sections (not hidden comments):

   ```markdown
   ## Adoption status
   - Source: {file/folder/URL}, adopted {date}.
   - **Unratified draft — not a canonical doc.** Ratify via `tstack-product`, which authors `docs/PRODUCT.md` from this.

   ## Open gaps
   - {specific, actionable gap, e.g. "Onboarding feature has no acceptance criteria — needs Given/When/Then"}
   - {e.g. "Recommendations feature is AI-driven but has no eval set / quality bar / fallback"}
   ```

   Then the distilled content below, organized toward PRODUCT.md's section order. Carry the source's substance across faithfully; do not invent anything to fill a gap — that's what the Open gaps list is for. Commit: `adopt: distilled {source} into draft (unratified, {N} gaps)`.

7. **Route.** Hand off to `tstack-product` (it reads the draft and authors the canonical doc). If you staged architecture/design notes, mention they're waiting at `docs/_adopted/` for when the chain reaches `tstack-architect`.

## Reference handoff

The downstream owners define the shapes your draft feeds into — model the draft on them:
- `../tstack-product/references/full-guide.md` — PRODUCT.md section order, acceptance-criteria formats (Given/When/Then, eval-based, measurable thresholds), data-model conventions. This is the bar your coverage assessment encodes.
- `../tstack-architect/references/full-guide.md` — what architecture/decisions material looks like, so you can recognize and stage it as notes (not transcribe it).

For a realistic coverage report and a sample `PRODUCT.draft.md` with its Adoption status + Open gaps headers, read `references/example-output.md`.

## Handoff

When the draft is written and committed:

> Adopted `{source}` → `docs/_adopted/PRODUCT.draft.md` ({N} open gaps named inside). {If staged: Architecture/design notes parked at `docs/_adopted/` for the architect stage.}
>
> **Next: run `tstack-product`** (or say "write the PRD") — it authors `docs/PRODUCT.md` from this draft, resolving the open gaps through its requirements gate, then continues the chain.
>
> **Fresh session** recommended for `tstack-product` on a larger adoption — it benefits from a clean context budget to ratify the draft properly.
