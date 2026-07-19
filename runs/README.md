# Run records (published for replay)

Immutable records for the three published waves. **Privacy:** absolute local paths,
usernames, and the verbose SDK message stream (`transcript.messages`) have been
removed/normalized; the audit-relevant fields remain — per rep: the agent `output`,
token `usage`, and every `llm_rubric` check with **both judge transcripts**
(`verdict`, `reasoning`, raw text, prompt, model, transport). `transcript` keeps the
summary (subtype, num_turns, total_cost_usd, modelUsage).

Each `runs/<id>/` has `meta.json` (suite snapshot), `results.jsonl` (one line per
execution), `report.json` (computed metrics), and `raw/<case>/<rep>.json`.

| Run ID | Wave | Suite |
|---|---|---|
| `20260718T222953Z-ftlp7i` | wave1 | leaderboard-security-auditor-wshobson |
| `20260718T223240Z-ql5qal` | wave1 | leaderboard-code-reviewer-wshobson |
| `20260718T223537Z-cjlqow` | wave1 | leaderboard-security-auditor-voltagent |
| `20260718T223819Z-7nelyz` | wave1 | leaderboard-claude-code-itself |
| `20260718T234914Z-h58rkd` | wave2 | leaderboard-w2-security-auditor-wshobson |
| `20260718T235226Z-2mym8k` | wave2 | leaderboard-w2-code-reviewer-wshobson |
| `20260718T235548Z-e4igaz` | wave2 | leaderboard-w2-security-auditor-voltagent |
| `20260718T235904Z-me415n` | wave2 | leaderboard-w2-claude-code-restricted |
| `20260719T001818Z-3q1bbi` | wave3 | leaderboard-w3-security-auditor-wshobson-sonnet |
| `20260719T002230Z-7sfxo1` | wave3 | leaderboard-w3-code-reviewer-wshobson-sonnet |
| `20260719T002601Z-gn55m0` | wave3 | leaderboard-w3-security-auditor-voltagent-sonnet |
| `20260719T002918Z-29ue38` | wave3 | leaderboard-w3-claude-code-sonnet |
| `20260719T003216Z-js5tai` | wave3 | leaderboard-w3-security-auditor-wshobson-haiku |
| `20260719T003546Z-zkivh2` | wave3 | leaderboard-w3-code-reviewer-wshobson-haiku |
| `20260719T003934Z-2dv7il` | wave3 | leaderboard-w3-security-auditor-voltagent-haiku |
| `20260719T004319Z-93n7a4` | wave3 | leaderboard-w3-claude-code-haiku |
