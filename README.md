> **Publication in progress** — the maintainers of the agents under test have been
> notified with their results and have a **one-week response window** before these
> numbers are finalized. Corrections/responses will be incorporated.

# 7runs flakiness leaderboard — Phase 1-A

A reproducible, **whole-system flakiness** benchmark for AI code-review agents. Every
number here is **harness + prompt + model as one system** — not a model-vs-model
claim. Two agents that share an underlying model can still differ in reliability
because their prompts and scaffolding differ; that difference is what this measures.

Built with **7runs**, a CLI for testing and evaluating AI agents (golden suites,
N-rep runs, Wilson-CI reliability reports, regression detection). The CLI lives at
**[7runs.dev](https://7runs.dev)**; the product repository opens at launch.

## The task (identical for every agent)

"Review this diff for security issues and report your findings." Cases are
byte-identical across all suites in a wave, so results are directly comparable.

- **Wave 1** (`fixtures/`, `suite-*.yaml`) — 3 easy cases (SQLi, command injection,
  clean README). All agents hit the ceiling → we escalated.
- **Wave 2** (`wave2/`) — 3 *hard* cases: obfuscated command injection, a **safe**
  parameterized-`LIKE` query (false-positive bait), and a multi-file XSS. Still all
  100% at Opus.
- **Wave 3** (`wave3/`) — the same 3 hard cases across **Sonnet-5 and Haiku-4-5**
  (the model axis). This is where agents finally separate.

## Agents under test

| Suite | Agent | Source | See |
|---|---|---|---|
| `*-security-auditor-wshobson` | security-auditor | wshobson/agents | [PROVENANCE.md](./PROVENANCE.md) |
| `*-code-reviewer-wshobson` | code-reviewer | wshobson/agents | [PROVENANCE.md](./PROVENANCE.md) |
| `*-security-auditor-voltagent` | security-auditor | VoltAgent/awesome-claude-code-subagents | [PROVENANCE.md](./PROVENANCE.md) |
| `*-claude-code` | Claude Code (`claude -p`) | — | restricted config (below) |

The three subagents run via the Claude Agent SDK with `settingSources: []` (isolated).
The **Claude Code** entry runs headless `claude -p` **restricted to Read/Grep/Glob
with no skills and no MCP** — deliberately *not* "out-of-the-box Claude Code" (a clean
`CLAUDE_CONFIG_DIR` was infeasible; the isolation is done via a tool allowlist). See
[RESULTS.md](./RESULTS.md).

## Method

- **7 reps per case** (one run proves nothing — variance is the headline metric).
- **Judge:** `llm_rubric`, **double-judged**, on the `claude-agent` transport, judge
  model `claude-sonnet-5`. A rubric encodes the correct verdict per case, so scoring
  is format-agnostic across heterogeneous free-form output. Disagreement → `disputed`.
- **No precision/recall** — free-form output has no reliable deterministic
  `positive_when`; per the tool's "never fabricate metrics" rule that section is
  omitted (cases are still labeled positive/negative for documentation).
- **Metrics:** overall pass rate with **Wilson 95% CI**, flakiness, `disputed`.

## Results

**[RESULTS.md](./RESULTS.md)** — the three-wave writeup with the headline finding:
the model axis discriminates where case difficulty did not (Opus 100% → Sonnet
95–100% → **Haiku 67–76%**, the collapse concentrated on the false-positive bait —
smaller models keep recall, lose precision).

- Machine-readable data + tables: [`publication/`](./publication/)
  (`data.json`, `TABLE.md`, `haiku-fp-bait-histograms.md`).
- **Raw run records for replay:** [`runs/`](./runs/) — per rep, the agent output,
  token usage, and **both judge transcripts** for every rubric check. (For privacy,
  absolute local paths and the verbose SDK message stream are stripped; see
  [`runs/README.md`](./runs/README.md).)

## Reproduce

Requires the **7runs CLI** ([7runs.dev](https://7runs.dev)). From the repo root:

```bash
# Wave 2 (Opus, as authored):
for s in security-auditor-wshobson code-reviewer-wshobson security-auditor-voltagent claude-code; do
  env -u ANTHROPIC_API_KEY 7runs run "wave2/suite-$s.yaml" --label wave2
done
# Wave 3 (model axis):
for s in security-auditor-wshobson code-reviewer-wshobson security-auditor-voltagent claude-code; do
  for m in sonnet haiku; do
    env -u ANTHROPIC_API_KEY 7runs run "wave3/suite-$s-$m.yaml" --label wave3
  done
done
```

**`env -u ANTHROPIC_API_KEY`** matters: these runs use Claude **subscription** auth,
and the key's presence would route billing to the key even on the SDK path — stripping
it for the run's process tree forces subscription auth.

## Licensing

- **Code, suites, and docs we authored:** MIT — see [LICENSE](./LICENSE).
- **Benchmark data** (`runs/`, `RESULTS.md`, `publication/` data): **CC-BY-4.0** —
  see [LICENSE-DATA](./LICENSE-DATA). Attribution keeps re-use of the numbers honest.
- **Vendored agents** (`agents/`): under their **upstream** licenses (MIT at time of
  vendoring); this repo does not relicense them. See [PROVENANCE.md](./PROVENANCE.md).

## Ethics (non-negotiable)

Before these numbers are publicized, each agent's maintainers are notified with their
results and given one week to respond. This repo is reproducible precisely so those
conversations are evidence-based.
