---
name: eng-simplify
description: Behavior-preserving cleanup of recently changed code — reuse existing helpers, remove duplication and dead code, reduce needless abstraction. Use after fresh implementation and before review, or on a file that keeps absorbing complexity.
argument-hint: "[optional: file/dir to target; blank = current branch diff]"
---

# Simplify

Refine what was just written so review reads the intended change, not the noise around it. **Behavior-preserving only** — bug-hunting belongs to `/eng-review`.

**Scope:** <target> $ARGUMENTS </target> — blank means the branch diff (`git diff <merge-base>...HEAD` + uncommitted). Only touch files in scope; simplification of unrelated code is scope creep, note it and move on.

## Passes

1. **Reuse before invention.** For each new helper/util/type in the diff, grep the repo for an existing equivalent. Duplicating an existing capability is the highest-value fix here.
2. **Duplication within the diff.** Same shape written 2-3 times across the new code (common after parallel subagent work) → extract once, at the same abstraction level the codebase already uses.
3. **Needless indirection.** Single-use abstractions, pass-through wrappers, config for things with one value, premature generality ("might need it later" — delete it; git remembers).
4. **Dead weight.** Unused imports/params/branches, commented-out code, debug leftovers, comments that narrate the next line instead of stating a constraint.
5. **Naming.** Names that lie or that diverge from the codebase's established vocabulary.

**Do not:** change behavior, rename public APIs, reformat untouched code, or "improve" architecture beyond the diff. If a real structural problem is visible but out of scope, report it instead of fixing it.

## Verify and report

Run the affected tests after each pass — a simplification that breaks a test is a behavior change; revert it. Commit as `refactor: simplify <area>` if the work is already committed, or leave staged with the pending work if not.

Report: fixes applied (grouped by pass), net line delta, anything flagged-not-fixed. If nothing needed simplifying, say exactly that — do not invent churn to look useful.
