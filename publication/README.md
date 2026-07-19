# Publication data pack — Phase 1-A

Machine-readable data and rendered tables behind the results (see
[`../RESULTS.md`](../RESULTS.md)).

| File | What it is |
|---|---|
| [`data.json`](./data.json) | Machine-readable full results — 16 rows (4 agents × waves/models), each with per-case passes/n/CI/stability, overall pass rate + Wilson CI, flakiness, disputed, and mean cost/exec. The source of truth for the tables below. |
| [`TABLE.md`](./TABLE.md) | The headline **model-axis** table (Opus→Sonnet→Haiku × pass rate + cost), the per-case pass matrix, and the Wave 1 easy-case reference. |
| [`haiku-fp-bait-histograms.md`](./haiku-fp-bait-histograms.md) | Per-agent histograms of the Haiku false-positive collapse on the safe parameterized-query case, with the verified failure flavor. |

Run IDs are listed in [`../RESULTS.md`](../RESULTS.md); the raw records they refer to
are under [`../runs/`](../runs/).

**Note on cost:** Sonnet-5 figures use the promotional $2/$10 per-MTok pricing (through
2026-08-31); `data.json` carries the standard $3/$15 figures (from 2026-09-01) too.

**Labeling:** the Claude Code row is "restricted to Read/Grep/Glob, no skills/MCP" —
never "out-of-the-box."
