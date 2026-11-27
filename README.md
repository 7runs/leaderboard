# 7runs flakiness leaderboard — Phase 1-A

A reproducible, **whole-system flakiness** benchmark for AI code-review agents. Every
number here is **harness + prompt + model as one system** — not a model-vs-model
claim. Two agents that share an underlying model can still differ in reliability
because their prompts and scaffolding differ; that difference is what this measures.

We used **7runs** to repeat each test and report pass rates and flaky cases. The [benchmark report](https://7runs.dev/leaderboard) summarizes the results. The suites, fixtures, raw runs and judge transcripts are available here.

## Test your own agent

Build the CLI from its GitHub source with Git, Node.js 20 or later and npm. npm installs dependencies; the 7runs engine is not installed from the npm registry.

> The engine repository is currently private while public access is being prepared. The following commands require repository access; anonymous installation is not available yet.

```sh
git clone https://github.com/7runs/7runs.git ../7runs-engine
cd ../7runs-engine
npm ci
npm run build:cli
export SEVENRUNS_CLI="$PWD/dist/index.js"
cd -
```

From this benchmark checkout, run a suite with `node "$SEVENRUNS_CLI" run <suite.yaml>` and generate its reports with `node "$SEVENRUNS_CLI" report latest`. Use the starter quickstart below first to check installation without model credentials. Published model-backed benchmark suites require your own configured model access and can incur model-provider costs.

Follow the [public quickstart](https://github.com/7runs/.github/blob/main/profile/README.md) to run the starter suite and verify 14/14 passing executions. Then use the [command and HTTP examples](https://github.com/7runs/.github/tree/main/examples/generic-adapter) to connect your own agent, set the answer you expect, and generate HTML and JSON reports. To run the checks in your repository, follow the [GitHub Actions setup and report guide](https://github.com/7runs/.github/blob/main/examples/github-actions/README.md).

Solo builders can test their own projects for free, including paid apps. Use by or on behalf of an organization requires a separate license. [Request an organization license](https://7runs.dev/#request-license).

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
| `*-claude-code` | Claude Code (`claude -p`) | — | [Wave-specific configuration](./RESULTS.md) |

The three subagents run via the Claude Agent SDK with `settingSources: []` (isolated).
In Wave 1, **Claude Code** ran with the local configuration. For Waves 2 and 3,
its read-only toolset was restricted to Read/Grep/Glob with skills and MCP disabled.
[Configuration details](./RESULTS.md).

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
