# Provenance — vendored agents

The three external agents under test are vendored **unmodified** into `agents/`
for reproducibility, each pinned to the exact upstream commit it was fetched from
(2026-07-18). To refresh, re-fetch from the pinned commit (or bump the SHA
deliberately and note it here — never edit a vendored file in place).

| Local file | Upstream repo | Commit (pinned) | Path in repo | Frontmatter `model` |
|---|---|---|---|---|
| `agents/wshobson-security-auditor.md` | [wshobson/agents](https://github.com/wshobson/agents) | `c4b82b0ad771190355eb8e204b1329732a18449a` | `plugins/security-compliance/agents/security-auditor.md` | `opus` |
| `agents/wshobson-code-reviewer.md` | [wshobson/agents](https://github.com/wshobson/agents) | `c4b82b0ad771190355eb8e204b1329732a18449a` | `plugins/comprehensive-review/agents/code-reviewer.md` | `opus` |
| `agents/voltagent-security-auditor.md` | [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) | `947b44ca0c58d606b084e9cb1a2389335b49278b` | `categories/04-quality-security/security-auditor.md` | `inherit` → resolves to `claude-opus-4-8` at run time |

Notes:
- `wshobson/agents` ships the same agent name under several plugins; the copies
  chosen here are the security-/review-focused plugins. If they diverge upstream,
  the pinned commit above is authoritative.
- The VoltAgent agent declares `model: inherit`. With no orchestrator model to
  inherit, the Claude Agent SDK resolves it to the account default, observed as
  `claude-opus-4-8` in the run transcripts. It is **not** overridden — the model
  is part of the system under test.
- Licensing: both upstream repos are MIT-licensed at the time of vendoring. These
  files remain under their upstream license; this repo's MIT license does not
  relicense them. See each upstream repo's `LICENSE`.

## Fetch commands (reproducible)

```bash
WSHA=c4b82b0ad771190355eb8e204b1329732a18449a
VSHA=947b44ca0c58d606b084e9cb1a2389335b49278b
curl -sL "https://raw.githubusercontent.com/wshobson/agents/$WSHA/plugins/security-compliance/agents/security-auditor.md" -o agents/wshobson-security-auditor.md
curl -sL "https://raw.githubusercontent.com/wshobson/agents/$WSHA/plugins/comprehensive-review/agents/code-reviewer.md"   -o agents/wshobson-code-reviewer.md
curl -sL "https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/$VSHA/categories/04-quality-security/security-auditor.md" -o agents/voltagent-security-auditor.md
```
