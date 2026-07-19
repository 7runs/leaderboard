# Haiku false-positive collapse — `fp-bait-parameterized-like` histograms

The case is a **SAFE** parameterized `LIKE` query that *looks* like SQL string
concatenation. Correct behavior: report **no** vulnerability. `█`=pass (correctly
safe), `░`=fail (asserted a vulnerability). Opus/Sonnet handle it; Haiku collapses.

| Agent | Opus | Sonnet-5 | Haiku-4-5 | Haiku failure flavor |
|---|---|---|---|---|
| security-auditor (wshobson) | ███████ 7/7 | ███████ 7/7 | ██░░░░░ **2/7** | 4/5 failures assert a vuln in the *pre-diff* code (flaky) |
| code-reviewer (wshobson) | ███████ 7/7 | ███████ 7/7 | ░░░░░░░ **0/7** | 5/7 failures assert a vuln in the *pre-diff* code (stable_fail) |
| security-auditor (VoltAgent) | ███████ 7/7 | ███████ 7/7 | ░░░░░░░ **0/7** | 7/7 failures assert a vuln in the *pre-diff* code (stable_fail) |
| Claude Code (restricted Read/Grep/Glob) | ███████ 7/7 | ███████ 7/7 | ██░░░░░ **2/7** | 4/5 failures assert a vuln in the *pre-diff* code (flaky) |

**Reading it:** at Haiku the two *real*-vuln positives stay ~7/7 (recall intact);
only this safe case collapses (precision lost). The dominant failure isn't misreading
the new parameterized code — it's **fabricating a SQL injection in the old `SELECT … LIMIT 20`** that had no user input. Honest caveat: some of this is rubric
strictness (Haiku often calls the *new* code parameterized but is failed for the
over-claim about the past).
