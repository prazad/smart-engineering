---
name: eng-debug
description: Reproduce a bug, trace the complete causal chain to root cause, fix with a regression test that proves it. Use for broken behavior, failing tests, error messages, or stack traces; use eng-plan/eng-work for features.
argument-hint: "[bug description, error message, stack trace, failing test path, or issue reference]"
---

# Debug

**Input:** <bug> $ARGUMENTS </bug>

## Core discipline

1. **Investigate before fixing.** No fix until you can explain the full causal chain from trigger to symptom with zero gaps. "Somehow X leads to Y" is a gap.
2. **Predictions test hypotheses.** For each uncertain link in the chain, state something else that must also be true if the hypothesis holds — then check it. A fix that "works" while a prediction fails means you patched a symptom.
3. **One change at a time.** Changing multiple things to "see if it helps" is shotgun debugging — stop and re-derive the chain instead.
4. **When stuck, diagnose why you're stuck** — wrong layer? unreproduced assumption? stale state? — don't just try harder.

## Phase 0 — Triage

- Issue reference given (`#123`, tracker URL): fetch it including **the full comment thread** — later comments often contain narrowed scope, failed attempts, or a pivoted root-cause theory that invalidates the opening post.
- If the user signals prior failed attempts ("keeps failing", "I've tried"), ask what they already tried before investigating — the one case where asking first beats reading code first.
- **Trivial fast-path:** if the cause is immediately readable (missing import, obvious typo/nil, one-line fix) — state cause and fix, confirm, apply with a regression test if behavior-bearing, done.
- Otherwise, no questions — investigate. Ask only when a genuine ambiguity blocks investigation and code/tests can't resolve it.

## Phase 1 — Reproduce

Get a reliable reproduction before theorizing: run the failing test, hit the endpoint, execute the repro steps. If it won't reproduce, that IS the finding — investigate environment/data/timing differences before touching code. Capture the exact failure output as the baseline.

Also check `docs/solutions/` for this symptom — a documented past diagnosis may shortcut everything.

## Phase 2 — Root cause

1. Trace the code path from trigger to symptom, reading actual code (not docs, not memory).
2. Form ranked hypotheses. For each: what evidence supports it, what prediction would confirm it.
3. Test the top hypothesis with the cheapest decisive probe (targeted log, narrower test, bisect, data inspection).
4. **Causal chain gate:** before moving to the fix, write the chain explicitly — `trigger → step → step → symptom` — and confirm every link is verified, not assumed. If a link is assumed, probe it.
5. If two hypothesis rounds fail, escalate systematically: `git log`/`git bisect` for regressions, widen to config/data/environment, question the reproduction itself.

## Phase 3 — Fix

1. Confirm workspace state (uncommitted work, correct branch — create `fix/<slug>` if on the default branch).
2. **Regression test first:** write or update a test that fails for exactly the diagnosed reason. Watch it fail. This proves the diagnosis and prevents recurrence.
3. Apply the minimal fix at the root cause — not where the symptom surfaced, unless that genuinely is the cause.
4. Watch the regression test pass; run the surrounding suite for collateral damage.
5. Check for **siblings**: does the same broken pattern exist elsewhere (grep for it)? Fix or flag them.

## Phase 4 — Handoff

Summarize: symptom → root cause (the verified chain) → fix → regression test → siblings checked. If the diagnosis was non-obvious or the pattern likely recurs, suggest `/eng-learn` to capture it — a debugging session that ends in a documented learning is worth double.

If asked for **diagnosis only**, stop after Phase 2 with the summary; don't edit code.
