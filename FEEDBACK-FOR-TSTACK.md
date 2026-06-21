# Feedback for T-Stack — discussion + prompts

> **▶ Start here.** Open Claude Code in the T-Stack repo and paste:
> *"Read FEEDBACK-FOR-TSTACK.md top to bottom for context. Then start on Prompt 1
> (revisable decisions) — and per the file, show me the plan before editing any files."*
> Then go prompt-by-prompt (approve Prompt 1 before Prompt 2).

> Scratch artifact (untracked). Open a Claude session **in the T-Stack repo** and work
> through the prompts at the bottom. Derived from bootstrapping Atlas (2026-06) where
> T-Stack's doc set and the `xcode-project-bootstrapper` ("Bootstrap") collided.
> Companion: `FEEDBACK-FOR-BOOTSTRAP.md` (run in the Bootstrap repo).

## How to use this file

1. **Read it whole first** — it's small and self-contained; you don't need the
   originating project or any prior chat.
2. **Then work the prompts at the bottom one at a time** — each is a separate workstream
   and ends with "show the plan before editing." Don't fire them all at once.
3. **Verify against this repo's current code as you go** — items are observations from a
   2026-06 run; some specifics may have moved. Treat them as a design backlog to discuss
   and confirm, not settled work.
4. Item IDs (`T1`, `S2`, …) are shared with the companion file so cross-refs line up.
   `T*` = T-Stack-owned, `B*` = Bootstrap-owned, `S*` = shared. This file details the `T*`
   and `S*` items; the `B*` items live in `FEEDBACK-FOR-BOOTSTRAP.md`.

## Background — where this came from

One real run: a **T-Stack** doc set (PRODUCT / DECISIONS / ARCHITECTURE / ROADMAP /
CONVENTIONS / TESTING) for **Atlas**, a native iOS app, was implemented by the
**xcode-project-bootstrapper** ("Bootstrap") in its T-Stack mode, milestone **M0**
(infrastructure baseline).

The core collision: T-Stack's docs committed to **zero third-party dependencies**, a
**flat single-target** layout, and **MV with no DI framework** (ADR-6, reinforced as
anti-patterns in CONVENTIONS). Bootstrap is built around **vendored Forge DI + modular
Swift packages**. The owner chose Bootstrap's modular baseline — which required
**superseding an Accepted ADR mid-build** (recorded as ADR-8) and left ARCHITECTURE.md's
Repository Structure factually **stale**. Those symptoms drive every item below.

Key terms: **M0/i0** = the infra-baseline milestone; **ADR-6** = T-Stack's zero-dep/flat
decision; **ADR-8** = the Bootstrap override that adopted Forge+modular; **Forge** = a
small first-party-Swift DI framework Bootstrap vendors in.

## Principle this all serves (read first)

**T-Stack and Bootstrap are independent tools.** T-Stack must produce a *complete*
architecture a human could implement by hand with **no Bootstrap**. So none of the
changes below ask T-Stack to *defer* or *leave gaps* for Bootstrap to fill — T-Stack
keeps deciding everything. The single thing to change is **rigidity**: T-Stack's
decisions should be **authoritative but revisable** (explicit revisit triggers, a
low-ceremony supersession path), so that *any* implementer — Bootstrap, or a human who
later adopts a DI framework — can override one cleanly instead of treating it as a
violation. T-Stack never assumes Bootstrap will run; it just stops writing its decisions
in stone.

## The friction, in one line

Every foundational decision T-Stack emitted was framed as **immutable** — ADRs marked
*Accepted*, choices reinforced as *anti-patterns* — so when the owner picked Bootstrap's
Forge+modular baseline, a legitimate choice required *superseding an Accepted ADR
mid-build* and reading like rule-breaking.

---

## Discussion — what to change in T-Stack

### T1 — Frame foundational/stack ADRs as authoritative *but revisable* — HIGH
ADR-6 ("zero dependencies", flat MV-no-DI) was *Accepted* and CONVENTIONS branded
"third-party dependency" an **anti-pattern**. Keep deciding these — just:
- Give every foundational/stack ADR an explicit, generous **"Revisit when…"** trigger
  (ADR-6 already had one; make it universal and prominent).
- Document a **low-ceremony supersession path** ("to change this, add an ADR-N that
  supersedes it and update the affected docs") so an override is a sanctioned move.
- Stop encoding a *choice* as an inviolable *convention/anti-pattern* (e.g. "no DI
  framework"). State preferences as defaults with rationale, not prohibitions.

### T2 — Keep authoring Repository Structure, but mark it a baseline — MED
T-Stack should still emit a complete, buildable repository tree (a Bootstrap-free project
needs it). Just label it: *"Baseline layout; an implementer may restructure — record the
change as an ADR + update this section."* Then if Bootstrap restructures, **Bootstrap**
owns updating it. Don't mark it "TBD".

### T3 — Split `i0`/`M0` "Done when" into implementation vs external config — HIGH
M0 mixed criteria any code-only implementation can satisfy (build, test, lint, OSLog)
with things it can't (GitHub Actions actually firing on a PR, branch protection,
TestFlight). Split every infra milestone's "Done when" into:
- **[implementation satisfies]** — verifiable by the build/test/lint itself.
- **[owner configures externally]** — GitHub/Apple settings, marked as such.

This is tool-agnostic (true for a human implementer too). Note: the *"empty package
graph"* criterion is **correct for T-Stack's own zero-dep design** — only an implementer
that supersedes the dependency ADR reworps it (and that implementer updates the criterion;
see S2). Don't pre-weaken it.

### T4 — Avoid hard source paths in architecture prose — MED
ARCHITECTURE.md/CONVENTIONS referenced concrete files (`Atlas/Support/Logging.swift`,
`Atlas/Data/…`, `Atlas/Stores/…`) and milestones leaned on them. Under any restructure
they break. Keep "Read before starting" pointing at **doc sections** (mostly already
true) and describe the data layer by **logical role**, not file path.

### T5 — Reserve ADR numbering for an implementer — MED
Expect the implementer to add ≥1 ADR (the DI/structure decision). Either reserve a range
(`ADR-i1…`) or a section, or document that implementer ADRs continue the sequence — so
Bootstrap's ADR-8 can't collide with a future T-Stack ADR-8.

### T6 — Emit an explicit iOS infra-baseline milestone (closed loop) — MED
For iOS, `tstack-roadmap` should emit *"i0 — iOS infrastructure baseline"* whose plan
names the implementer (e.g. "run Bootstrap `/bootstrap`, or stand up the baseline by
hand") with implementation-scoped "Done when". This pairs with Bootstrap's proposed
**`bootstrap-tstack`** skill (see the Bootstrap doc) and `tstack-mode.md §9`.

### T7 — Regenerate stale guidance after later chain steps — LOW
Generated `AGENTS.md` said ROADMAP was *"not generated yet"* although it exists, and its
Common Commands assumed a repo-root project. Refresh the ROADMAP pointer / Current Focus
once roadmap exists, and reconcile command paths with the agreed project location (S1).

### S2 (T-Stack side) — make `tstack-status` aware of sanctioned implementer edits — HIGH
`tstack-status` currently flags any post-sync edit to monitored docs as **drift**, which
pressures an implementer to leave docs stale rather than fix them. Teach it to recognize
**sanctioned surgical edits** (e.g. a sync-marker note like `(surgical: iOS scaffold …)`,
or edits to an agreed allowlist of sections) and report them as *intentional re-sync*, not
drift. This is what lets Bootstrap update docs without fear.

### S3 (T-Stack side) — emit a machine-readable handshake of T-Stack's decisions — MED
Write a marker (e.g. `docs/.tstack.json` or a ROADMAP header) recording T-Stack's **own
decisions** — `target: ios`, `di: none/manual`, `structure: flat`, `deps: none` — as
authoritative defaults. An optional implementer reads it to detect intent (instead of
sniffing four ADRs) and, if it diverges, supersedes the relevant decision via an ADR.
T-Stack stays the decider; the marker is just a clean read surface.

---

## Prompts to start in the T-Stack repo

Paste these into a Claude session opened in the T-Stack repo, one workstream at a time.

### Prompt 1 — revisable decisions (T1, T5)
```
In this repo, the product workflow emits foundational/stack ADRs (security,
observability, accessibility, privacy + tech-stack) via the tstack-architect skill.
Today they read as immutable: marked "Accepted" and reinforced as "anti-patterns" in
CONVENTIONS, so an optional downstream implementer that chooses differently (e.g. adds a
DI framework) has to "supersede an Accepted ADR," which reads like breaking the rules.

Keep the architect making complete, standalone decisions — do NOT make it defer. Instead,
update the architect's ADR template and CONVENTIONS generation so that:
1. every foundational/stack ADR carries an explicit, prominent "Revisit when…" trigger;
2. there's a documented, low-ceremony supersession path;
3. choices are stated as defaults-with-rationale, not inviolable anti-patterns;
4. ADR numbering reserves room (or a documented convention) for an implementer to add
   their own ADR without collision.
Show me the proposed template + CONVENTIONS changes before editing.
```

### Prompt 2 — iOS-aware structure & milestones (T2, T3, T6)
```
For iOS projects this workflow is often implemented by an external Xcode bootstrapper.
Make the following changes, keeping T-Stack fully standalone (the architecture must still
be hand-implementable with no bootstrapper):
1. tstack-architect: keep emitting a complete Repository Structure, but label it a
   "baseline an implementer may restructure (record as an ADR + update this section)."
2. tstack-roadmap: split every infrastructure milestone's "Done when" into
   "[implementation satisfies]" vs "[owner configures externally: GitHub/Apple settings]".
3. tstack-roadmap: for iOS, emit an explicit "i0 — iOS infrastructure baseline" milestone
   whose plan names the implementer and uses implementation-scoped Done-when.
Propose the edits to each skill before applying.
```

### Prompt 3 — drift-awareness + handshake (S2, S3, T4, T7)
```
Two integration changes so an optional downstream implementer can keep docs accurate
without tripping drift detection:
1. tstack-status: recognize "sanctioned surgical edits" to monitored docs (e.g. a
   sync-marker note pattern, or an agreed section allowlist) and report them as intentional
   re-sync, not drift.
2. Emit a machine-readable handshake (docs/.tstack.json or a ROADMAP header) recording
   T-Stack's own decisions: target, di, structure, deps. These are authoritative defaults
   an implementer reads and may supersede via an ADR.
Also: stop putting hard source-file paths in architecture prose (use logical roles), and
refresh generated AGENTS.md guidance after roadmap exists. Show the plan first.
```
