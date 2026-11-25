# 7runs flakiness leaderboard — Phase 1-A

A reproducible, **whole-system flakiness** benchmark for AI code-review agents. Every
number here is **harness + prompt + model as one system** — not a model-vs-model
claim. Two agents that share an underlying model can still differ in reliability
because their prompts and scaffolding differ; that difference is what this measures.

We used **7runs** to repeat each test and report pass rates, flaky cases and regressions. The [benchmark report](https://7runs.dev/leaderboard) summarizes the results. The suites, fixtures, raw runs and judge transcripts are available here.

To try the engine locally, install the CLI with Node.js 20 or later.

```sh
npm install --global @7runs/7runs@0.1.0
```

The [local quickstart](https://7runs.dev/#local) runs two starter cases seven times each and produces HTML and JSON reports. The benchmark suites and their recorded results are documented below.

Solo builders can test their own projects for free, including paid apps. Use by or on behalf of an organization requires a separate license. [Ask about an organization license](mailto:hello@7runs.dev?subject=7runs%20organization%20license).

## The task (identical for every agent)

"Review this diff for security issues and report your findings." Cases are
byte-identical across all suites in a wave, so results are directly comparable.

- **Wave 1** (`fixtures/`, `suite-*.yaml`) — 3 easy cases (SQLi, command injection,
  clean README). Three agents passed 21/21. Claude Code passed 20/21.
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

**Full report: [7runs.dev/leaderboard](https://7runs.dev/leaderboard)**

On the three hard cases, each agent passed 21/21 reviews on Opus. On Haiku the
four agents passed all 56 reviews of the two vulnerable diffs, but only 4 of the
28 reviews of the safe SQL query. [Read the three-wave results](./RESULTS.md).

## Verify a score

The [publication tables](./publication/) list per-case scores, confidence intervals
and run IDs. Select an entry in the [run index](./runs/README.md), then open
`results.jsonl` in that run’s folder. Each line contains one repetition and its
pass verdict. The corresponding `raw/<case>/<rep>.json` contains the agent
output and both judge transcripts. Each suite and its fixture diffs are included
in this repository.

Method note preserved for the record: the published runs used Claude **subscription**
auth (`env -u ANTHROPIC_API_KEY` so billing could not silently route to an API key).

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
