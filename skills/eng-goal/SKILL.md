---
name: eng-goal
description: Iterate toward an objectively verifiable goal — run the verification gate, fix the root cause, re-run, repeat until green or the attempt budget is spent. Use when the end state is checkable by a command (tests pass, build clean, lint zero) and you want convergence, not a single pass.
argument-hint: "[goal] --verify \"<command>\" [--max N]"
---

# Goal

A bounded convergence loop: **generator proposes, gate decides.** The verification gate — not your judgment — is the only authority on whether the goal is met.

**Input:** <input> $ARGUMENTS </input> — a goal statement, optionally `--verify "<command>"` and `--max N` (default 5).

## Setup — establish the gate

1. **Gate command given:** use it verbatim.
2. **No gate given:** derive the narrowest objective command that proves the goal (a specific test path beats the whole suite; `tsc --noEmit` beats "looks type-safe"). State the derived gate in one line before starting.
3. **Goal not objectively checkable** ("make it cleaner", "improve UX"): stop and say so — offer either a concrete proxy gate to agree on, or `/eng-plan` for work that needs human judgment. A loop without a real gate just burns attempts.

## Run the gate first

Before touching anything, run the gate and read the failure:

- **Already green:** report that and stop. Nothing to converge on — the loop is idempotent.
- **Red:** capture the exact failure output. This is the baseline every attempt is measured against.
- **Gate itself is broken** (command not found, wrong path): fix the gate, not the code, and restate it.

## Iterate (up to the budget)

Each attempt:

1. **Diagnose from the gate's output** — trace to root cause, not the nearest symptom. Two consecutive attempts failing on the *same* error means local pokes aren't working: step back and build the causal chain (as `/eng-debug` would) before attempt three.
2. **Smallest fix** that addresses the diagnosis, following existing conventions.
3. **Re-run the gate.** Log the attempt in one line: what changed, how the failure output moved (fixed / same / different error).
4. **Commit** on meaningful green progress (a sub-goal passing), conventional message — so a later interruption resumes from evidence, not memory.

**Never satisfy the gate by weakening it** — deleting or skipping tests, loosening assertions, broadening lint ignores, lowering coverage thresholds. That is failing the goal while forging the certificate. If the gate itself is genuinely wrong, say so and stop for the user's call.

## Stop conditions

- **Gate green:** report the goal, the gate command, final passing output, attempts used, and commits made.
- **Budget spent:** stop and report honestly — the attempt log, the best current diagnosis, and what a human should look at. A loop that fails loudly is doing its job; one that thrashes past its budget is not.
- **Blocked on user input** (missing credentials, ambiguous intent, gate dispute): stop immediately with the specific question.

For recurring cadence (re-check every N minutes), drive this skill with native `/loop` — each invocation is idempotent, so a green gate exits immediately.
