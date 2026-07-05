---
name: eng-learn
description: Capture a solved problem, hard-won diagnosis, or durable decision into docs/solutions/ so future sessions start smarter. Use after fixing a non-obvious bug, settling a pattern/tooling decision, or whenever a lesson would otherwise be relearned from scratch.
argument-hint: "[optional: what was learned; blank = extract from this session]"
---

# Learn

Write the one document that saves the next session from rediscovering this. This is the compounding step: `eng-plan`, `eng-review`, and `eng-debug` all read `docs/solutions/` as grounding, so what you write here changes future behavior.

**Input:** <topic> $ARGUMENTS </topic> — if blank, extract the most valuable lesson from the current session (the non-obvious diagnosis, the decision with durable rationale, the mistake that cost real time).

## Worth capturing / not worth capturing

**Capture:** non-obvious root causes, misleading symptoms, decisions with rationale that will be questioned later, patterns that must be followed (or avoided) repo-wide, environment/tooling traps.

**Skip:** anything derivable from the code itself, one-off facts, generic best practices any competent agent already knows, session-specific state. If the lesson is really a missing convention, propose adding a line to the project's root instructions file instead — that's always-loaded, which beats discoverable.

## Dedup first

Grep `docs/solutions/` for the topic before writing. If an existing doc covers the same problem, **update it** (append the new variant, correct what changed) — never create a near-duplicate.

## Write one file

Path: `docs/solutions/<category>/<kebab-slug>.md`. Categories — pick the narrowest fit, create the directory if new:

- Bug track: `build-errors/`, `test-failures/`, `runtime-errors/`, `performance/`, `database/`, `security/`, `integration/`, `logic-errors/`
- Knowledge track: `architecture-patterns/`, `design-patterns/`, `tooling-decisions/`, `conventions/`, `workflow/`

```markdown
---
module: <area of the codebase>
problem_type: <e.g. runtime_error, tooling_decision, convention>
tags: [searchable, keywords, someone-with-this-problem-would-grep]
date: YYYY-MM-DD
---

# <Title stating the problem or decision, not the fix>

## Symptom / Context
What you'd observe (exact error text when applicable — that's what
gets grepped) or the situation that forced the decision.

## Root cause / Decision
The verified explanation, or the decision and what it was chosen over.

## Solution
What actually fixed/settled it. Concrete: code shape, command, config.

## Prevention
How to not hit this again — the check, the pattern, the rule of thumb.
```

Quality bar: the reader is a future agent with zero session context and a symptom in hand. Front-load exact error strings and searchable terms. Keep it under a page — a learning nobody reads because it's a wall of text compounds nothing.

## Finish

Confirm: path written (or updated), one-line hook of what it saves next time. One file per invocation — if the session holds two distinct lessons, write the more valuable one and offer the second.
