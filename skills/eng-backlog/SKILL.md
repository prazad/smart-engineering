---
name: eng-backlog
description: One discovery-sweep pass — find the single next unbuilt item (unshipped plan in docs/plans/, ready-labeled issue, or clear self-contained TODO), run the eng-auto pipeline on it, report, exit. Drive on a cadence with native /loop or /schedule for a continuous backlog burner.
argument-hint: "[source filter: plans | issues | todos — or an explicit item]"
---

# Backlog

Loops that only execute still need someone to find the work. This is the discovery half: **pick exactly one item, run it through `/eng-auto`, exit.** Continuity comes from re-invocation, not from this skill running forever.

**Input:** <input> $ARGUMENTS </input> — optional source filter or an explicit item (which skips discovery).

## Discover (in priority order)

1. **Unbuilt plans:** files in `docs/plans/` with no matching branch or PR (search branch names and open/merged PRs for the plan's slug) and no `> Blocked:` marker at the top. Oldest first.
2. **Ready issues:** `gh issue list` filtered to a readiness label (`ready`, `good first issue`) — only if the repo actually uses such labels; skip issues that already have a linked branch or PR.
3. **Clear TODOs:** `TODO`/`FIXME` markers only when self-contained and unambiguous (the comment says exactly what to do). Anything requiring interpretation stays in the report as a suggestion, not a pick.

**Never auto-pick** items touching auth, payments, data migrations, or deletion of user data — list those in the report for the user to launch explicitly.

Nothing eligible → report "backlog clear" (plus anything deliberately skipped and why) and exit.

## Execute

Invoke `/eng-auto` on the picked item — it owns branching, quality gates, shipping, and CI watch. State the pick and the reason in one line before starting.

## If blocked

- **Plan item:** prepend `> Blocked: <reason> (<date>)` to the plan file so the next pass skips it instead of retrying forever. The user removes the marker to re-queue.
- **Issue or TODO:** report the blocker; don't comment on the issue or edit the code comment — external writes and re-scoping are the user's call.

## Report

The pick and why it won, the pipeline outcome (PR URL + CI state, or blocked-with-reason), and a one-line preview of what the next pass would pick. Then exit — one item per pass keeps each PR reviewable and each failure attributable.
