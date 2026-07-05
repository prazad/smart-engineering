# Smart Engineering

A lean, Claude-Code-native rewrite of the compound-engineering workflow: **each unit of work should make the next one easier.**

8 skills, no ceremony. The loop: plan → work → simplify → review → ship → learn — and the learnings feed the next plan.

## Skills

| Skill | Use it for |
|-------|-----------|
| `/eng-plan` | Turn an idea or bug report into an implementation-ready plan (requirements Q&A + technical planning in one right-sized pass) |
| `/eng-work` | Execute a plan or clear request — evidence-first tests, incremental commits, parallel subagents for independent units |
| `/eng-simplify` | Behavior-preserving cleanup of the fresh diff before review |
| `/eng-review` | Multi-lens verified review (correctness, security, tests, maintainability, conventions, plan fidelity) |
| `/eng-ship` | Commit → push → PR → optionally watch CI to green |
| `/eng-debug` | Reproduce → causal chain to root cause → fix proven by a regression test |
| `/eng-learn` | Capture the lesson into `docs/solutions/` so future sessions start smarter |
| `/eng-auto` | The whole pipeline hands-off: plan, build, review+fix, ship, watch CI |

## The compounding mechanism

`/eng-learn` writes one-page solution docs to `docs/solutions/<category>/` with greppable frontmatter. `/eng-plan`, `/eng-review`, and `/eng-debug` all grep that directory as grounding — so a lesson captured once changes every future plan, review, and debugging session in the repo. That return arrow is the whole point.

## Typical usage

```text
# Feature loop
/eng-plan add rate limiting to the API
/eng-work
/eng-simplify
/eng-review
/eng-ship
/eng-learn        # if something non-obvious was discovered

# Bug
/eng-debug checkout webhook creates duplicate invoices

# Hands-off
/eng-auto docs/plans/2026-07-05-rate-limiting.md
```

## Install

```bash
# From a local marketplace or directly:
claude plugin install smart-engineering@<marketplace>
# Or add this directory to your marketplace config and install from there.
```

## Design

This is a from-scratch rewrite informed by Every's [compound-engineering plugin](https://github.com/EveryInc/compound-engineering-plugin), keeping its best ideas (the compound loop, evidence-first execution, verified review findings, causal-chain debugging) at roughly 15% of the size, single-platform, with zero cross-skill infrastructure. Full rationale in [docs/REVIEW.md](docs/REVIEW.md).
