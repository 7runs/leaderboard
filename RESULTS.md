# Phase 1-A results

- **[Wave 1](#wave-1--easy-cases-2026-07-18)** — 3 easy cases (2026-07-18)
- **[Wave 2](#wave-2--hard-discriminating-cases-2026-07-18)** — 3 hard discriminating cases (2026-07-18)

---

## Wave 1 — easy cases (2026-07-18)

Run label `wave1`. 7 reps × 3 cases per agent (21 executions each, 84 total).
Judge: `claude-sonnet-5`, **double-judged on the `claude-agent` transport** (D21).
Auth: **subscription** — `ANTHROPIC_API_KEY` stripped from the run process tree
with `env -u` (never billed). No infra errors; every execution recorded.

## Leaderboard

| Agent | Model | Pass rate | Wilson 95% CI | Flaky cases | Disputed |
|---|---|---:|:--:|:--:|:--:|
| security-auditor (wshobson) | opus (claude-opus-4-8) | **100%** (21/21) | [85%, 100%] | 0/3 | 0 |
| code-reviewer (wshobson) | opus (claude-opus-4-8) | **100%** (21/21) | [85%, 100%] | 0/3 | 0 |
| security-auditor (VoltAgent) | opus via `inherit` (claude-opus-4-8) | **100%** (21/21) | [85%, 100%] | 0/3 | 0 |
| Claude Code itself (`claude -p`) | claude-sonnet-5 (pinned) | **95%** (20/21) | [77%, 99%] | **1/3** | 0 |

Even at 100%, the Wilson CI floor is 85% — with N=21 you cannot claim "perfectly
reliable," only "no failure observed in 21 tries." That honesty band is the point.

### Per-case (passes / 7)

| Case | sec-auditor (wsh) | code-reviewer (wsh) | sec-auditor (Volt) | Claude Code |
|---|:--:|:--:|:--:|:--:|
| `sqli-string-concat` (pos) | 7/7 | 7/7 | 7/7 | 7/7 |
| `cmdi-shell-true` (pos) | 7/7 | 7/7 | 7/7 | 7/7 |
| `clean-readme-doc` (neg) | 7/7 | 7/7 | 7/7 | **6/7 — flaky** |

## What surprised me

1. **The only flakiness in the whole wave came from Claude Code itself — and it
   wasn't a wrong security call.** On 6/7 reps it correctly cleared the benign
   README diff. On rep 5 it never reviewed the diff at all: it emitted
   "housekeeping notes" that `/security-review` (a skill) couldn't run because the
   sandbox "isn't a git repository," and that a Linear MCP connector needed
   authorization. The judges (correctly) failed it — there was no security
   assessment in the output.

2. **Root cause is environment leakage, and it cleanly separates the two adapters.**
   The three subagents run through the `claude-agent` **SDK adapter with
   `settingSources: []`** — fully isolated from the user's Claude Code config — and
   were 100% stable. "Claude Code itself" runs through the **shell adapter
   (`claude -p`)**, which inherits the *real* user config: skills, MCP connectors,
   and all. That ambient surface is exactly what derailed rep 5. So this data
   point is partly measuring *config-driven distraction*, not just review skill.

3. **Two-of-three subagents are literally the same model (Opus) on the same task,
   yet the comparison is still meaningful** — it isolates prompt/scaffolding. Here
   both wshobson and VoltAgent security-auditors, and even the general
   code-reviewer, were rock-solid on these three cases. To find where they diverge
   we need harder cases (subtle vulns, near-misses) — see next steps.

4. **Zero disputes across 84 rubric checks** — the two judges agreed every time.
   That's expected on unambiguous cases; it also means the judge itself added no
   observed noise here.

## Caveats / next steps

- **Fairness of the Claude Code entry:** for a *published* number it should run
  against an **isolated config** (clean `CLAUDE_CONFIG_DIR`, no user skills/MCP),
  or the leakage should be reported as an explicit property of "out-of-the-box
  Claude Code." Flagged, not silently averaged.
- **Ceiling effect:** three cases at 100% don't rank anything. Wave 2 needs harder,
  more discriminating cases (obfuscated injection, false-positive bait, multi-file
  diffs) to pull the agents apart.
- Not published. Per the ethics rule, maintainers get notified with their numbers
  and one week to respond before anything goes public.

## Reproducibility

| Suite | Run ID |
|---|---|
| `suite-security-auditor-wshobson.yaml` | `20260718T222953Z-ftlp7i` |
| `suite-code-reviewer-wshobson.yaml` | `20260718T223240Z-ql5qal` |
| `suite-security-auditor-voltagent.yaml` | `20260718T223537Z-cjlqow` |
| `suite-claude-code.yaml` | `20260718T223819Z-7nelyz` |

Records under `.7runs/runs/<id>/` (git-ignored) hold the suite snapshot, every raw
agent output, and both judge transcripts per check.

---

## Wave 2 — hard discriminating cases (2026-07-18)

Run label `wave2`. 7 reps × **3 hard cases** per agent (21 each, 84 total). Same
methodology as Wave 1 (judge `claude-sonnet-5`, double-judged on the `claude-agent`
transport; subscription auth via `env -u`; no infra errors). **All four agents run
on `claude-opus-4-8`** this time, so differences would be attributable to
prompt/scaffolding, not the model.

The three cases were designed to be *discriminating*, not just harder:
`obfuscated-cmdi` (a real command injection disguised as a safe arg vector,
`os.system(" ".join(parts))`, with an unused-`shlex` red herring),
`fp-bait-parameterized-like` (a **safe** parameterized `LIKE` query that *looks*
like SQL string-concatenation — false-positive bait), and `multifile-xss` (a real
reflected XSS visible only by reading **two** files together).

### Leaderboard

| Agent | Model | Pass rate | Wilson 95% CI | Flaky | Disputed |
|---|---|---:|:--:|:--:|:--:|
| security-auditor (wshobson) | opus (claude-opus-4-8) | **100%** (21/21) | [85%, 100%] | 0/3 | 0 |
| code-reviewer (wshobson) | opus (claude-opus-4-8) | **100%** (21/21) | [85%, 100%] | 0/3 | 0 |
| security-auditor (VoltAgent) | opus (claude-opus-4-8) | **100%** (21/21) | [85%, 100%] | 0/3 | 0 |
| Claude Code (`claude -p`, restricted to Read/Grep/Glob, no skills/MCP) | opus (claude-opus-4-8) | **100%** (21/21) | [85%, 100%] | 0/3 | 0 |

Every case: 7/7 for every agent. Wilson floor is 85% at N=21 — "no failure in 21
tries," not "provably perfect."

### The Claude Code row is a *restricted* configuration — not out-of-the-box

That row is **`claude -p` limited to `Read`/`Grep`/`Glob` with no skills and no
MCP** — deliberately not "out-of-the-box Claude Code." Why: the requested clean-
`CLAUDE_CONFIG_DIR` isolation is **infeasible on this machine** — the subscription
token is bound to the default `~/.claude` location, so any custom config dir reads
as "Not logged in" and (because the judge shares the run's process env) also breaks
the judge's auth. Symlinking the account file and mirroring the config minus
`plugins/` both still failed auth (3 attempts). Isolation was therefore done
*structurally* instead: default permission mode + `--allowedTools "Read Grep Glob"`
means the Skill/SlashCommand/MCP tools are unavailable, so no skill (e.g. Wave 1's
`/security-review`) or MCP connector can run; `--strict-mcp-config` ignores any MCP
config. The trade-off is a real one — this measures Claude Code's *review ability
under a read-only toolset*, not its default agentic behavior.

### What the results say (and don't)

1. **Isolation fixed the Wave 1 flake.** Wave 1's Claude Code scored 95% (20/21)
   because one rep went off-task on an env-leaked `/security-review` skill + MCP
   prompt. Wave 2, isolated: **100% (21/21)**, and a scan of all 21 outputs found
   **0** skill/MCP/login leak markers. The failure mode was environmental, and
   removing the environment removed it.
2. **The "hard" cases still did not discriminate — and the passes are genuine, not
   lucky.** Spot-checks confirm agents engaged the traps explicitly: on the
   false-positive bait they named it ("looks like it might be SQL injection … but
   the value is passed as a bound parameter"), and on the multi-file case they
   traced the XSS across both files ("functionally deceptive refactor … reflected
   XSS"). At Opus, all four systems are simply reliable on unambiguous vulns, even
   obfuscated or split across files.
3. **What this implies for the leaderboard:** obfuscation/multi-file difficulty is
   *not* the discriminating axis for Opus-class agents. To actually rank these
   apart, a Wave 3 needs one of: (a) **genuinely ambiguous** cases where even
   experts split (flakiness then shows as the model flip-flopping) — harder to
   score with a crisp rubric; or (b) the **same agents on smaller/faster models**
   (haiku/sonnet), where prompt/scaffolding quality is more likely to matter.
   Difficulty-that-has-a-clear-answer gets solved; difficulty-that-is-ambiguous is
   where flakiness lives.

Nothing is published; the maintainer-notification ethics rule stands.

### Reproducibility

| Suite | Run ID |
|---|---|
| `wave2/suite-security-auditor-wshobson.yaml` | `20260718T234914Z-h58rkd` |
| `wave2/suite-code-reviewer-wshobson.yaml` | `20260718T235226Z-2mym8k` |
| `wave2/suite-security-auditor-voltagent.yaml` | `20260718T235548Z-e4igaz` |
| `wave2/suite-claude-code.yaml` | `20260718T235904Z-me415n` |

---

## Wave 3 — model axis (2026-07-18)

Run label `wave3`. Same **3 byte-identical hard cases** as Wave 2, all 4 agents,
swept across two smaller models (`claude-sonnet-5`, then `claude-haiku-4-5`) — only
the agent model changes; judge stays `claude-sonnet-5` (claude-agent transport),
7 reps, subscription via `env -u`. 8 suites × 3 × 7 = **168 executions, 0 infra
errors**. Model overrides were smoke-verified to actually take effect (haiku ran on
haiku, not a silent fallback). Claude Code uses the same restricted isolation as
Wave 2 (Read/Grep/Glob, no skills/MCP).

### Cost-vs-reliability curve (mean $/execution × pass rate)

| Agent | Opus | Sonnet-5 | Haiku-4-5 |
|---|---|---|---|
| security-auditor (wshobson) | 100% · $0.0343 | **95%** · $0.0202 | **76%** · $0.0100 |
| code-reviewer (wshobson) | 100% · $0.0365 | 100% · $0.0204 | **67%** · $0.0098 |
| security-auditor (VoltAgent) | 100% · $0.0392 | 100% · $0.0235 | **67%** · $0.0119 |
| Claude Code (restricted Read/Grep/Glob) | 100% · n/a | 100% · n/a | **76%** · n/a |

- Cost is the price-table model cost (Opus 5/25, **Sonnet 2/10**, Haiku 1/5 per Mtok)
  × measured tokens — a *modeled* API cost; actual execution was on subscription.
- **Sonnet-5 pricing is promotional** ($2/$10 per MTok) **through 2026-08-31**; it
  reverts to $3/$15 on 2026-09-01. At the standard rate the Sonnet points would be
  **$0.0303, $0.0306, $0.0353** respectively (i.e. the promo figures × 1.5). The
  price table (`7runs.yaml`) now carries the promotional 2/10.
- **Opus cost was recomputed from stored token counts**: Wave 2 recorded the model
  as the `opus` alias, which isn't in the price table, so 7runs reported cost=null
  there (a real normalization gap — `normalizeModelId` doesn't map alias→snapshot
  id; noted, not fixed without a decision).
- **Claude Code has no cost point** — the shell adapter (`claude -p`) can't capture
  token usage. Reliability only.

### The headline: the model axis discriminated where the case axis could not

At **Opus** every agent was 100% (Wave 2, ceiling). Drop the model and they separate:

- **Sonnet-5 ≈ near-ceiling** (95–100%) at **~$0.020/exec under the current
  promotional pricing** (~40% below Opus); at the standard rate from 2026-09-01
  (~$0.030) the savings shrink to marginal, since token counts dominate and Opus and
  Sonnet use similar tokens. Either way **Sonnet is the sweet spot**: near-Opus
  reliability at ≤ Opus cost.
- **Haiku craters to 67–76%** at ~$0.010/exec (~3× cheaper) — and the collapse is
  **almost entirely one case: the false-positive bait** (the *safe* parameterized
  `LIKE` query). Per-case at Haiku: the two *real*-vuln positives
  (`obfuscated-cmdi`, `multifile-xss`) stay ~7/7 for everyone, while `fp-bait`
  falls to 2/7, 0/7, 0/7, 2/7.

**Smaller models keep recall, lose precision.** Haiku still *catches* the real
injections and the multi-file XSS — it just can't *restrain* from flagging safe
code. Verified failure mode: of the Haiku `fp-bait` failures, almost all
(4/5, 5/7, **7/7**, 4/5) assert a vulnerability in the *pre-diff* code — i.e. Haiku
fabricates/over-attributes a SQL injection in the old `SELECT … LIMIT 20` that had
no user input at all, rather than reading the new code as unsafe. Honest caveat:
part of this is rubric strictness — Haiku often correctly calls the *new* code
parameterized, but is failed for over-claiming a (nonexistent) vuln in the old
state; a reviewer could debate whether that should count. Either way it's a real
discernment gap that Opus/Sonnet don't have.

### Do the two same-name auditors finally separate?

**Numerically yes, but not robustly.** wshobson vs VoltAgent security-auditor:
Sonnet — 95% vs 100% (VoltAgent ahead, on one flaky `obfuscated-cmdi` miss by
wshobson); Haiku — **76% vs 67%** (wshobson ahead, `fp-bait` 2/7 vs 0/7). The winner
**flips** between Sonnet and Haiku and the Wilson CIs overlap heavily ([55,89] vs
[45,83]). Conclusion: smaller models *do* expose differences, but N=21 is not enough
to crown one auditor more robust — the gap is within noise. A larger rep count or
more cases would be needed to make it stick.

### Other signals

- **First disputes in the whole leaderboard appear at Haiku:** Claude Code / Haiku
  logged 1 `disputed` (the two sonnet-5 judges disagreed on one messy Haiku output).
  Judge noise shows up only at the smallest agent model.
- **`stable_fail` vs `flaky` on the bait:** code-reviewer and VoltAgent go 0/7 on
  `fp-bait` at Haiku (consistently wrong — reliably trigger-happy), while wshobson
  and Claude Code go 2/7 (flaky). Consistent-but-wrong is arguably worse than flaky.

### Reproducibility

| Suite | Sonnet run | Haiku run |
|---|---|---|
| security-auditor-wshobson | `20260719T001818Z-3q1bbi` | `20260719T003216Z-js5tai` |
| code-reviewer-wshobson | `20260719T002230Z-7sfxo1` | `20260719T003546Z-zkivh2` |
| security-auditor-voltagent | `20260719T002601Z-gn55m0` | `20260719T003934Z-2dv7il` |
| claude-code | `20260719T002918Z-29ue38` | `20260719T004319Z-93n7a4` |
