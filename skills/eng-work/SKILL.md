---
name: eng-work
description: Execute an implementation plan or a concrete work request end-to-end with evidence-first testing and incremental commits. Use when implementing from docs/plans or building something with clear scope; use eng-debug for open-ended bugs.
argument-hint: "[plan path or work description; blank = newest plan in docs/plans/]"
---

# Work

Execute systematically. The goal is a **finished, verified feature** — not perfect process.

**Input:** <input> $ARGUMENTS </input>

## Phase 0 — Triage

- **Plan path given:** read it. Treat it as a decision artifact, not a script — note `Scope boundaries` (refer back when implementation pulls toward adjacent work), `Deferred to implementation` questions, and each unit's pattern references.
- **Blank:** pick the newest file in `docs/plans/`; confirm it's the intended one before starting.
- **Bare prompt:** scan the affected area, then route by complexity:
  - *Trivial* (1-2 files, no behavior change): implement directly, no task list.
  - *Small/medium* (clear scope, <10 files): build a task list from your own discovery, proceed.
  - *Large* (cross-cutting, 10+ files, or touches auth/payments/migrations): recommend `/eng-plan` first. Honor the user's choice.

## Phase 1 — Setup

1. **Branch:** never work on the default branch without explicit user confirmation. If on it, create `feat/<slug>` or `fix/<slug>` (pull first). If already on a sensibly named feature branch, stay.
2. **Task list:** create one task per implementation unit with TaskCreate, keyed `U1: <name>`, with dependencies. Mark in_progress/completed as you go.
3. **If any plan ambiguity blocks starting, ask now** — once, up front. Then execute without renegotiating scope.

## Phase 2 — Execute each unit

For each task in dependency order:

1. **Idempotency check:** if the unit's work already exists and satisfies its verification criteria (prior branch/session), verify it, mark complete, move on. Never silently reimplement.
2. **Read the pattern files** the plan names. Mirror their conventions exactly.
3. **Evidence before implementation.** Find existing tests for the files being changed, then choose:

   | Situation | Action |
   |-----------|--------|
   | An existing test already fails for the intended behavior | Use it as the red evidence |
   | An existing test asserts the old expectation | Update it, watch it fail, then implement |
   | No test covers the behavior | Write the smallest failing test first, watch it fail for the right reason |
   | Testing genuinely inappropriate (pure config, styling, generated files) | Note the reason and the replacement verification (smoke run, visual check) |

   Never write the test and the implementation in the same step. A test you never saw fail proves nothing.
4. **Implement** the smallest change that makes the evidence pass, following existing conventions.
5. **System-wide check** (skip for leaf-node changes): What fires when this runs — callbacks, middleware, hooks, event handlers two levels out? Do the tests exercise the real chain, or is every layer mocked? Can a failure mid-flow leave orphaned state? Is the same behavior exposed through another interface that needs parity? If any answer is uncomfortable, add one integration test with real objects through the full chain.
6. **Run tests** for the affected area. Fix failures immediately — never start the next unit on a broken tree.
7. **Commit** when the unit is complete and tests pass: stage only that unit's files (never `git add .`), conventional message from the unit's goal. If the message would be "WIP", don't commit yet.

## Parallel execution (structured multi-unit plans)

When the plan has 3+ units and some are **independent** (per the plan's dependency notes), dispatch them as parallel subagents via the Agent tool with `isolation: "worktree"`:

- **Safety check first:** units that touch the same files, shared types/interfaces, migrations, lockfiles, or an environment singleton (dev server, database) run **serially**. File-disjoint is necessary but not sufficient — reason about shared contracts too.
- Give each worker a bounded packet: the unit's goal, files, pattern refs, test scenarios, verification — plus the evidence-first rules above. Not "read the whole plan."
- Require each worker to report changed paths **and** its verification evidence (test added/changed, red observed, result) — that evidence is not reconstructable from the tree.
- Cap at ~3 concurrent workers. After a batch: inspect actual diffs (not just reports), merge branches in dependency order, run tests, commit per unit, clean up worktrees. On a merge conflict, abort the merge and re-run that unit serially against the merged tree.
- If a batch produces out-of-scope edits or repeated conflicts, stop parallelizing and finish serially.

## Phase 3 — Finish

1. Every ~3 units, scan recent changes for duplication that emerged across units; consolidate if real (subagent workers can't see each other's patterns).
2. When all tasks are complete: run the full test suite, then hand off — suggest `/eng-simplify` (if the diff is substantial), then `/eng-review`, then `/eng-ship`.
3. Report honestly: what shipped, verification evidence per behavior-bearing unit, any deliberate no-test exceptions with reasons, anything deferred.

When invoked by `/eng-auto`: skip all interactive questions (resolve with the plan's stated assumptions), skip the handoff suggestion, and return a structured summary — status, changed files, units completed, verification evidence, blockers — without shipping. The caller owns review and PR.

## Pitfalls

- **80% done syndrome** — finish the feature; don't move on early.
- **Testing at the end** — test continuously.
- **Scope creep** — the plan's boundaries are authority; log discovered-but-out-of-scope work as a note, don't do it.
- **Human-time re-scoping** — never split execution into multi-day "phases" or ask the user to pick a subset of units. Context pressure is solved by subagents, not sessions.
