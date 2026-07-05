---
name: eng-auto
description: Autonomous full pipeline — plan, implement, simplify, review + fix, ship, and watch CI to green, hands-off. Use when handing over a well-described feature or an existing plan and stepping away.
argument-hint: "[feature description or plan path]"
---

# Auto

Run the full loop without stopping for input. The user is stepping away; every decision gets made with stated assumptions rather than questions.

**Input:** <request> $ARGUMENTS </request>

If the input is empty, or so vague that any plan would be a guess at *what the user wants* (not merely *how to build it*), stop immediately and ask — that's the one legitimate question. How-to ambiguity is resolved autonomously with recorded assumptions.

## Pipeline

Each stage invokes the named skill in pipeline mode (no interactive questions; assumptions recorded in the artifacts). Announce each stage transition in one line so the transcript reads as a progress log.

1. **Plan** — skip if the input is an existing implementation-ready plan path. Otherwise invoke `eng-plan`; ambiguities become explicit `## Assumptions` in the plan.
2. **Work** — invoke `eng-work` in pipeline mode: implement all units with evidence-first testing and incremental commits; return the structured summary. If work returns `blocked`, stop the pipeline and report — do not improvise around a blocker.
3. **Simplify** — invoke `eng-simplify` on the branch diff when it's non-trivial (≥30 changed lines).
4. **Review + fix** — invoke `eng-review` with the plan path, then apply Critical/High/Medium fixes (root-cause fixes only), re-running tests after each batch.
5. **Ship** — invoke `eng-ship`: commit anything pending, push, open the PR, then watch CI and repair failures (max 3 distinct repair attempts).
6. **Learn** — if the run produced a non-obvious lesson (a diagnosis, a repeated review finding, a CI trap), invoke `eng-learn` to capture it. Skip when there's genuinely nothing durable — most clean runs have nothing; don't manufacture a learning.

## Guardrails

- Never work on the default branch; the pipeline creates its own feature branch (or worktree if the workspace is dirty with unrelated changes).
- Never force-push, never touch branches other than the pipeline's own.
- Quality gates are not skippable for speed: red-test evidence in work, verified findings in review, root-cause CI repairs in ship.
- If any stage fails twice in a row on the same error, stop and report state honestly (branch, commits, what passed, what's broken) instead of looping.

## Final report

One message: PR URL, CI status, plan path, units completed, review findings fixed vs deferred, assumptions made along the way, and any learning captured. The user should be able to decide merge/revise from this message alone.
