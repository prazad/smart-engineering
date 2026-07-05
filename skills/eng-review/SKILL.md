---
name: eng-review
description: Multi-lens review of the current branch diff (or a PR) for correctness, security, test quality, maintainability, and project-convention violations — findings verified before reporting, with optional fix application. Use before merging.
argument-hint: "[optional: PR number, base ref, plan path to verify against, or 'fix' to apply findings]"
---

# Review

Review the diff, verify every finding against the actual code, report ranked findings. Reviews catch **the pattern, not just the bug**.

**Input:** <args> $ARGUMENTS </args>

## Scope

- Default: `git diff <merge-base-with-default-branch>...HEAD` plus uncommitted changes.
- PR number given: `gh pr diff <n>` and `gh pr view <n>`.
- A plan path given (or an obvious matching plan in `docs/plans/`): also verify **plan completeness** — every implementation unit's success criteria actually met, every stated test scenario actually covered. Unbuilt requirements are findings.

If the diff is purely mechanical (formatting, lockfile, generated files), say so and stop.

## Size the review

- **Small diff** (<150 changed lines, no sensitive surface): review inline yourself with all lenses in one pass.
- **Standard/large diff, or any sensitive surface** (auth, payments, migrations, crypto, input handling, concurrency): dispatch parallel subagents, one per applicable lens, each given the diff scope and lens brief below.

## Lenses

Apply every lens that matches the diff; skip lenses with no matching surface.

1. **Correctness** — logic errors, off-by-ones, nil/None handling, race conditions, broken edge cases, error paths that swallow or double-handle failures. For each suspect, trace the concrete failing input.
2. **Security** — injection, authz gaps (IDOR, missing checks), secrets in code, unsafe deserialization, SSRF, path traversal. Only on diffs touching input handling, auth, network, files, or queries.
3. **Tests** — do new behaviors have tests that would actually fail on regression? Over-mocked tests that prove nothing? Missing error-path and boundary coverage? Assertions that assert nothing?
4. **Maintainability** — duplication of existing code in the repo (grep before believing something is new), needless complexity, dead code, misleading names, comments that narrate instead of explain constraints.
5. **Project conventions** — violations of the project's active instructions and conventions already in context, plus applicable learnings: grep `docs/solutions/` for topics matching the diff; a documented past mistake recurring is a high-severity finding.
6. **Plan fidelity** (only when a plan is in scope) — requirements without implementation, implementation without requirements (scope creep), verification criteria unmet.

## Verify, then rank

**Every candidate finding gets verified before it's reported.** Re-read the actual code; construct the concrete failure scenario (inputs/state → wrong outcome). A finding you can't state a failure scenario for is dropped or explicitly marked speculative. This step is what separates review from linting — do not skip it under time pressure.

Deduplicate across lenses, then rank:

| Severity | Meaning |
|----------|---------|
| **Critical** | Data loss, security hole, breaks core flow — must fix before merge |
| **High** | Real bug on a plausible path, or a documented learning violated |
| **Medium** | Wrong-but-survivable: missing error handling, weak tests, real duplication |
| **Low** | Polish; note only when the fix is cheap |

## Report

Lead with the verdict: merge-ready, or N findings blocking. Then findings ranked most-severe first, each with `file:line`, one-sentence defect, and the concrete failure scenario. No filler praise; a clean diff gets a short "verified clean across N lenses" with what was checked.

## Applying fixes

Apply fixes only when asked (`fix` in the args, the user says so afterward, or invoked by `/eng-auto`):

- Fix Critical/High/Medium findings; batch by file; re-run tests after each batch; commit with a message naming what was fixed.
- **A fix must address the root cause, not silence a symptom** (never delete a failing test to make it pass, never broaden an exception handler to hide an error).
- Report each finding's outcome: fixed / skipped (why) / turned out fine on second look.

If a finding reveals a lesson worth keeping (a pattern likely to recur), suggest `/eng-learn` after fixes land.
