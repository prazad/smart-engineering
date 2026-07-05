# Review: Every's compound-engineering plugin → this rewrite

Reviewed: `compound-engineering` v3.17.1 (28 skills, ~28,000 lines of skill markdown).
Date: 2026-07-05.

## Verdict

The plugin's core ideas are excellent; its execution is optimized for Every's needs, not a solo developer's. The philosophy (each unit of work makes the next easier), the loop (plan → work → review → compound), and several specific disciplines are genuinely best-in-class. But roughly 70-80% of the content is overhead a personal Claude Code user pays for on every invocation: multi-platform portability machinery, artifact-contract ceremony, company-internal tooling, and defensive prose against failure modes of weaker harnesses.

## What was worth keeping (and was kept)

1. **The compounding loop itself.** `docs/solutions/` learnings read as grounding by plan/review/debug is the single highest-leverage idea in the plugin. Kept intact in `/eng-learn` with the same category taxonomy, simplified frontmatter, and the same dedup-before-write rule.
2. **Evidence-first execution** (ce-work). The "watch the test fail before implementing" table — use existing failing test / update wrong-expectation test / write smallest new failing test / record deliberate no-test exception — is the best encoding of TDD-for-agents I've seen. Kept nearly verbatim in `/eng-work`.
3. **The system-wide test check** (ce-work): "what fires when this runs — two levels out? do tests exercise the real chain or is every layer mocked? can failure leave orphaned state?" Kept.
4. **Causal-chain debugging discipline** (ce-debug): no fix until the trigger→symptom chain has zero gaps; predictions for uncertain links; one change at a time; the trivial-bug fast-path. The strongest skill in the plugin, kept almost whole in `/eng-debug`.
5. **Verified review findings** (ce-code-review): every finding backed by a concrete failure scenario before being reported; severity ranking; plan-fidelity checking. Kept in `/eng-review`, with its 16 reviewer personas collapsed into 6 lenses.
6. **Parallel subagent safety** (ce-work): file-overlap is necessary but not sufficient — shared types, migrations, lockfiles, environment singletons force serialization; bounded worker packets; orchestrator owns commits; inspect real diffs, not worker reports. Kept, condensed 5:1.
7. **Right-sizing** everywhere: trivial work gets no ceremony, deep work gets more structure.
8. **Commit intent classification** (fix vs feat by intent not diff shape; never `!`/`BREAKING CHANGE` without confirmation). Kept in `/eng-ship`.

## What was dropped, and why

1. **Cross-platform portability machinery** (~25% of core-skill text): interaction-method fallbacks for Codex/Pi/Antigravity, `SKILL_DIR` anchoring protocol, pre-resolution `!`-backtick patterns with triple fallbacks, per-harness worktree integration notes. You run Claude Code; the rewrite calls AskUserQuestion, Agent, and TaskCreate directly.
2. **The artifact contract system**: `ce-unified-plan/v1`, `artifact_readiness` states, `product_contract_source`, stable R/A/F/AE ID registries, superseded-sibling resolution, HTML output mode with a parallel rendering reference tree. Brainstorm and plan are two skills manipulating one contracted artifact with handoff states — for one person this is pure ceremony. The rewrite merges them into `/eng-plan` (Phase 1 is the brainstorm, skipped when requirements are already clear) writing plain markdown. Kept unit IDs (U1, U2) only, since `/eng-work` and `/eng-review` genuinely use them.
3. **The shared repo-profile cache**: byte-duplicated scripts/schemas across 8 skills with a parity test guarding drift — infrastructure whose maintenance cost exists *because* the plugin forbids cross-skill imports. A solo repo doesn't re-derive its profile often enough to need a git-keyed cache.
4. **Every-internal skills**: ce-riffrec-feedback-analysis (their feedback tool), ce-proof (their doc product), ce-promote, ce-product-pulse, ce-dogfood, ce-sweep, ce-test-xcode. Also the meta/niche tier: ce-pov, ce-strategy, ce-ideate, ce-explain, ce-compound-refresh, ce-optimize, ce-polish, ce-worktree (Claude Code's Agent tool worktree isolation covers it), ce-setup, ce-doc-review (folded into `/eng-review`'s plan-fidelity lens), ce-commit + ce-commit-push-pr (merged into `/eng-ship`).
5. **Defensive prose against weak-model failure modes**: multi-paragraph warnings about mode tokens without paths, progress-like readiness values, commented-YAML parsing traps. Real bugs for a plugin distributed to thousands of users across four harnesses; noise for one strong model on one machine.
6. **Handoff-menu contracts** ("Mandatory Completion Contract", blocking menus after every artifact). Replaced with a one-line next-step suggestion.

## Consolidation map

| Every (28 skills) | Rewrite (8 skills) |
|---|---|
| ce-brainstorm + ce-plan + ce-doc-review | `/eng-plan` (review lens moved to `/eng-review`) |
| ce-work + ce-worktree | `/eng-work` |
| ce-simplify-code | `/eng-simplify` |
| ce-code-review + ce-resolve-pr-feedback | `/eng-review` |
| ce-debug | `/eng-debug` |
| ce-compound (+ refresh, sweep) | `/eng-learn` |
| ce-commit + ce-commit-push-pr | `/eng-ship` |
| lfg | `/eng-auto` |
| remaining 13 skills | dropped |

## Size

~28,000 lines of skill markdown → ~700. Every line in the rewrite passes the plugin's own deletion test (a rule Every's AGENTS.md preaches better than its skills practice): states a falsifiable constraint, counters a known default tendency, or supplies non-obvious domain knowledge.

## What the rewrite adds

- **Learnings-aware review**: `/eng-review` treats a recurring documented mistake as high-severity — closing the loop Every leaves implicit.
- **Sibling check in debug**: after a root-cause fix, grep for the same broken pattern elsewhere.
- **`/eng-auto` learns**: the autopilot ends by capturing a lesson when the run produced one, so even hands-off runs compound.
