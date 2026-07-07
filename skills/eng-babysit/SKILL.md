---
name: eng-babysit
description: One idempotent pass over open PRs — repair red CI, address review comments, flag what's ready — then report and exit. Designed to run on a cadence via native /loop or /schedule (e.g. "/loop 15m /eng-babysit").
argument-hint: "[PR number(s), or blank = my open PRs]"
---

# Babysit

One pass, then exit cleanly. The cadence lives outside this skill (`/loop`, `/schedule`); the skill's job is to leave every PR strictly better than it found it and say exactly what state things are in.

**Input:** <input> $ARGUMENTS </input> — specific PR number(s), or blank for `gh pr list --author @me --state open`.

## Triage each PR

| State | Action |
|-------|--------|
| CI still running | Skip — note "in flight", don't touch |
| CI red | Repair (below) |
| Unresolved review comments | Address (below) |
| Approved + green | Report **ready to merge** — never merge; that's the user's call |
| Draft | Skip unless its CI is red |
| Merge conflict with base | Rebase/merge base in, resolve only mechanical conflicts; judgment conflicts go in the report |

Nothing open → report "nothing to babysit" and exit.

## CI repair

1. Check out the branch (worktree if the local tree has unrelated changes).
2. Read the failing log; fix the **root cause** locally — never delete/skip a test or loosen an assertion to force green.
3. Push, note the fix in the report.
4. **Budget: 2 repair attempts per PR per pass.** Before attempting, check whether the head commits are this skill's own fixes from a previous pass for the *same* failure — if so, don't stack another guess; escalate in the report with the log excerpt and your best diagnosis.

## Review comments

- **Clear, in-scope change requests:** implement, push, reply on the thread stating what changed (commit ref included).
- **Questions:** answer factually on the thread if the answer is in the code/plan; otherwise draft a reply in the report for the user to send.
- **Scope changes or judgment calls** ("should this also handle X?"): never decide autonomously — surface in the report.

## Guardrails

- Never force-push, never merge, never close PRs, never touch PRs outside the given scope.
- Every push is a normal commit on the PR's own branch; conventional message.
- If the same PR has been escalated in two consecutive passes with no user action, just restate it in one line — don't re-attempt.

## Report

One line per PR: number, state (in-flight / repaired / commented / ready-to-merge / **blocked-on-you**), and the action taken. Lead with anything blocked on the user. Then exit — a loop body that doesn't terminate isn't a loop body.
