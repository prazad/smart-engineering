---
name: eng-plan
description: Turn a feature idea, bug report, or rough request into an implementation-ready plan. Combines requirements clarification and technical planning in one right-sized pass. Use when asked to plan, break down, or think through work before building it.
argument-hint: "[feature description, requirements, or path to an existing plan to update]"
---

# Plan

Produce a plan an implementer can execute confidently without the plan writing the code for them. Plans capture **decisions, not code**: approach, boundaries, files, risks, and test scenarios.

**Input:** <request> $ARGUMENTS </request>

If the request is empty, ask: "What would you like to plan?" and wait.

## Right-size first

Classify the work before doing anything else:

| Depth | Signals | What you produce |
|-------|---------|------------------|
| **Trivial** | 1-2 files, no design decisions | No plan file. Say so and suggest `/eng-work` directly. |
| **Light** | Well-bounded, low ambiguity, one component | Compact plan: goal, files, approach, test scenarios. Skip research subagents. |
| **Standard** | Normal feature or bounded refactor with real decisions | Full plan below. |
| **Deep** | Cross-cutting, high-risk (auth/payments/migrations/data), or ambiguous | Full plan + explicit risk section + flow/edge-case pass. |

Escalate Light → Standard if research reveals external contract surfaces (public APIs, CI config, env vars, shared types consumed elsewhere).

## Phase 1 — Clarify requirements (only if unclear)

If the request already states what to build and why, skip this phase. Otherwise ask **one question at a time** with AskUserQuestion until you can state:

- **Problem:** what's broken or missing, for whom
- **Success criteria:** how we'll know it works
- **Scope boundary:** explicit non-goals
- **Blocking unknowns:** anything that changes the architecture if answered differently

Stop asking as soon as you can fill those four. Do not run a full interview for work whose shape is obvious.

## Phase 2 — Research (before structuring, never after)

1. **Past learnings:** If `docs/solutions/` exists, grep it for topics matching this work. A prior lesson that applies is the highest-value input a plan can have — cite it.
2. **Codebase patterns:** Find the 2-3 existing files most similar to what will be built. The plan will tell the implementer to mirror them. For Standard/Deep work, dispatch an Explore agent for this; for Light work, search inline.
3. **Project conventions:** Use the project's active instructions and conventions already in your context (test commands, structure, naming).
4. **External research (conditional):** Search the web only when (a) the user asked for it, (b) the domain is high-risk (security, payments, external APIs, migrations), or (c) the codebase has no local pattern for this layer. If strong local patterns exist, say "codebase has solid patterns for this, skipping external research" and move on.

Resolve open technical questions here. Ask the user only when the answer materially changes architecture, scope, or risk and can't be responsibly inferred. Defer genuinely execution-time unknowns explicitly — don't fake certainty.

## Phase 3 — Write the plan

Path: `docs/plans/YYYY-MM-DD-<kebab-slug>.md` (create the directory if needed). Use today's real date.

```markdown
# <type>: <Title>

**Date:** YYYY-MM-DD
**Depth:** light | standard | deep

## Problem
What's broken/missing and for whom. 2-4 sentences.

## Success criteria
Observable outcomes, not vibes.

## Scope boundaries
Non-goals. What this deliberately does NOT do.

## Approach
The key decisions and WHY — chosen over what alternative. Reference
learnings from docs/solutions/ that shaped a decision.

## Implementation units
### U1: <name>
- **Goal:** one meaningful change, landable as an atomic commit
- **Files:** repo-relative paths to create/modify + test file paths
- **Pattern to mirror:** existing file(s) to follow
- **Test scenarios:** specific inputs → expected outcomes (happy path,
  edge cases, error paths — enough that the implementer invents nothing)
- **Verification:** how to prove this unit works

### U2: ...
(ordered by dependency; note which units are independent of each other)

## Risks
(Standard/Deep only) What could go wrong, and the mitigation.

## Deferred to implementation
Questions intentionally left for execution time, so they inform
rather than surprise.
```

Rules:
- **Repo-relative paths everywhere.** Never absolute paths.
- Units are commit-sized: not 2-minute micro-steps, not multi-concern epics.
- Every feature-bearing unit names its test file and enumerated scenarios.
- If a diagram genuinely clarifies architecture or sequencing, use a mermaid block; otherwise skip it.

## Phase 4 — Confidence check and handoff

Before presenting, self-check: could an implementer with no session context execute each unit from its section alone? If a unit fails that test, fix it now.

Then present a 5-line summary (problem, approach, unit count, key risk, plan path) and ask what's next:
1. Start building (`/eng-work <plan-path>`)
2. Revise the plan
3. Done for now

When invoked by `/eng-auto` (pipeline mode), skip all interactive questions, resolve ambiguity with explicit stated assumptions in the plan, and return the plan path instead of asking.
