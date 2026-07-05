---
name: eng-ship
description: Commit the current work, push, and open a pull request with a clear description — optionally watching CI to green. Use when work is done and reviewed; handles commit-only if asked.
argument-hint: "[optional: 'commit' for commit-only, or extra context for the PR description]"
---

# Ship

**Input:** <args> $ARGUMENTS </args> — `commit` means stop after committing.

## Commit

1. `git status` + `git diff` first: know exactly what's being committed. If the tree mixes unrelated work, split into separate logical commits — stage each explicitly by path, never `git add .` / `git add -A`.
2. **Never commit to the default branch.** If on it, create a branch named from the change (`feat/<slug>`, `fix/<slug>`) first.
3. Message: conventional format, **classified by intent, not diff shape** — remedying broken/missing behavior is `fix:` even when implemented by adding code; reserve `feat:` for genuinely new capability. Include the narrowest useful scope: `fix(parser): handle empty frontmatter`. Subject ≤72 chars, imperative; body only when the *why* isn't obvious from the diff.
4. Never use `!` or `BREAKING CHANGE:` without explicit user confirmation — it triggers a major version bump.
5. Sanity gate before committing: secrets/keys in the diff, leftover debug output, files that shouldn't be tracked. Stop and flag rather than commit these.

If args said `commit`, report the commit hash and stop here.

## Push and PR

1. `git push -u origin <branch>`.
2. Check for a PR template (`.github/PULL_REQUEST_TEMPLATE.md`) and follow it. Otherwise:
   - **Title:** conventional, matching the primary commit's intent.
   - **Body:** what changed and why (2-5 sentences), how it was verified (test evidence, not "tests pass"), link the plan (`docs/plans/...`) and any issue (`Closes #N`), and call out anything reviewers should scrutinize.
3. `gh pr create` with that title/body. Report the PR URL.

## CI watch (when asked, or when invoked by /eng-auto)

Poll `gh pr checks <n>` in the background until conclusive. On failure: read the failing log, fix the root cause locally (never delete/skip a test or loosen an assertion to force green), push, re-watch. After 3 distinct failed repair attempts, stop and report the failure with logs rather than thrashing. End state to report: PR URL + checks green, or blocked-with-diagnosis.
