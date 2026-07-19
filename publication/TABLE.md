# 7runs flakiness leaderboard — Phase 1-A · publication tables

Whole-system flakiness (harness+prompt+model). Task: security review of a code diff.
7 reps/case, judge `claude-sonnet-5` double-judged on the claude-agent transport,
subscription auth (`env -u` — the $5 key was never billed). `llm_rubric`-only checks;
no precision/recall. Cost = price-table model cost × measured tokens (modeled). Claude
Code cost = n/a (shell adapter). **Not published — maintainer notices pending.**

## Headline: model axis on the 3 hard cases (Opus → Sonnet-5 → Haiku-4-5)

| Agent | Opus | Sonnet-5 | Haiku-4-5 |
|---|---|---|---|
| security-auditor (wshobson) | 100% [85,100] · $0.0343 | 95% [77,99] · $0.0202 | **76% [55,89]** · $0.0100 |
| code-reviewer (wshobson) | 100% [85,100] · $0.0365 | 100% [85,100] · $0.0204 | **67% [45,83]** · $0.0097 |
| security-auditor (VoltAgent) | 100% [85,100] · $0.0392 | 100% [85,100] · $0.0235 | **67% [45,83]** · $0.0118 |
| Claude Code (restricted Read/Grep/Glob) | 100% [85,100] · n/a | 100% [85,100] · n/a | **76% [55,89]** · n/a |

*Sonnet-5 cost is **promotional** $2/$10 per MTok (through 2026-08-31); at the
standard $3/$15 (from 2026-09-01) the Sonnet points are **$0.0303, $0.0306, $0.0353**.
Opus cost recomputed from stored tokens (Wave 2 stored the `opus` alias, absent from
the price table).*

## Per-case pass (passes / 7), hard cases

| Agent | Model | obfuscated-cmdi | fp-bait (SAFE) | multifile-xss |
|---|---|:--:|:--:|:--:|
| security-auditor (wshobson) | opus | 7/7 | 7/7 | 7/7 |
| security-auditor (wshobson) | sonnet-5 | 6/7 | 7/7 | 7/7 |
| security-auditor (wshobson) | haiku | 7/7 | 2/7 | 7/7 |
| code-reviewer (wshobson) | opus | 7/7 | 7/7 | 7/7 |
| code-reviewer (wshobson) | sonnet-5 | 7/7 | 7/7 | 7/7 |
| code-reviewer (wshobson) | haiku | 7/7 | 0/7 | 7/7 |
| security-auditor (VoltAgent) | opus | 7/7 | 7/7 | 7/7 |
| security-auditor (VoltAgent) | sonnet-5 | 7/7 | 7/7 | 7/7 |
| security-auditor (VoltAgent) | haiku | 7/7 | 0/7 | 7/7 |
| Claude Code (restricted Read/Grep/Glob) | opus | 7/7 | 7/7 | 7/7 |
| Claude Code (restricted Read/Grep/Glob) | sonnet-5 | 7/7 | 7/7 | 7/7 |
| Claude Code (restricted Read/Grep/Glob) | haiku | 7/7 | 2/7 | 7/7 |

## Wave 1 reference (3 *easy* cases — all-ceiling, why we escalated)

| Agent | Model | Pass | Wilson CI | Flaky |
|---|---|---:|:--:|:--:|
| security-auditor (wshobson) | opus | 100% | [85,100] | 0/3 |
| code-reviewer (wshobson) | opus | 100% | [85,100] | 0/3 |
| security-auditor (VoltAgent) | opus | 100% | [85,100] | 0/3 |
| Claude Code (restricted Read/Grep/Glob) | sonnet-5 | 95% | [77,99] | 1/3 |
