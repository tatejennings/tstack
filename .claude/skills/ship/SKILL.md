---
name: ship
description: >-
  Runs the COMPLETE release procedure for the TStack plugin when the user says "ship",
  "ship it", or "release" (the plugin / a new version). Does the whole thing end-to-end —
  version bump, CHANGELOG + BACKLOG updates, branch→merge to main, annotated tag, and a
  rendered GitHub Release — not just a commit. Use ONLY for releasing THIS repo (the TStack
  plugin); not for shipping a consumer project. Input is the repo's uncommitted/committed
  work since the last tag; output is a tagged, pushed release + a GitHub Release page.
---

# ship

You are running the TStack plugin's **release procedure**. When the user says "ship" /
"ship it" / "release", run the **complete** sequence below — don't stop at committing. The
push, tag, and GitHub-Release steps are outward-facing but are **pre-authorized by the ship
request itself**; do all steps without re-asking the sequence.

This skill releases **the plugin repo itself**. It is not a consumer-facing skill and does
not ship in the plugin (it lives in `.claude/`, outside `skills/`).

## Prereq check

1. Confirm cwd is the TStack plugin repo (`.claude-plugin/plugin.json`, `name` == `"tstack"`).
2. Confirm `gh auth status` is healthy before the release step (you'll need it for step 5).
3. Look at what's changed since the last tag (`git log <lasttag>..HEAD`, `git status`) so you
   know what's being released and can write accurate notes. If behavioral work is still
   uncommitted, commit it first (or confirm the user already did).

## Steps (in order)

1. **Bump the version** in `.claude-plugin/plugin.json` using semver:
   - new skill / new user-facing feature → **minor** bump (e.g. 0.4.0 → 0.5.0)
   - bug/doc fix only → **patch** bump
   - breaking change → **major** bump
   Pick the level from what actually shipped; mention which and why.

2. **Update docs:**
   - In `CHANGELOG.md`, move the `## [Unreleased]` entries under a new dated heading
     `## [X.Y.Z] — YYYY-MM-DD` (today's date), leaving an empty `## [Unreleased]` above it.
   - Update `BACKLOG.md`: delete bullets for anything fully shipped; for partially-shipped
     "bigger" items, trim the done part and note "partly shipped in vX.Y.Z" with what remains.
   - Keep skill-count wording consistent across `README.md`, `marketplace.json`, and
     `plugin.json` **only if** the skill topology changed this release.

3. **Commit on a short-lived branch, then fast-forward merge to `main`** (repo guidance: don't
   commit straight to the default branch):
   ```
   git checkout -b release/vX.Y.Z
   git add <the release files>        # NOT untracked scratch files (e.g. FEEDBACK-*.md)
   git commit -m "Ship vX.Y.Z — <one-line summary>"   # end with the Co-Authored-By trailer
   git checkout main
   git merge --ff-only release/vX.Y.Z
   git branch -d release/vX.Y.Z
   ```
   End the commit message with: `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`

4. **Tag and push:**
   ```
   git tag -a vX.Y.Z -m "vX.Y.Z — <summary>"
   git push origin main
   git push origin vX.Y.Z
   ```
   Remote is `git@github.com:tatejennings/tstack.git`.

5. **Create the GitHub Release.** Extract the matching CHANGELOG section as notes and publish,
   marking it latest:
   ```
   awk '/^## \[X\.Y\.Z\]/{f=1;next} /^## \[/{if(f)exit} f' CHANGELOG.md > /tmp/relnotes.md
   gh release create vX.Y.Z --title "vX.Y.Z — <summary>" --notes-file /tmp/relnotes.md --latest
   ```

Report the release URL when done.

## Notes

- **Why this is one trigger word:** the user wants "ship" to produce a clean, communicable
  release — version + changelog + tag + a rendered GitHub Release — because TStack is shared
  with a team (Releases give a "what's new" surface + Watch→Releases notifications).
- GitHub Releases are **communication-only**; they do **not** affect plugin install (the
  marketplace pulls from the repo/branch, not Releases).
- The CHANGELOG-move + version-bump part is also documented in this repo's `CLAUDE.md`; this
  skill adds the branch→merge, tag, and `gh release` steps around it.

## Handoff

Hands off to nothing. Manual release utility for this repo.
