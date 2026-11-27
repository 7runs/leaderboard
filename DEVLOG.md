
## 2026-11-27T10:04:22+00:00 — GitHub source installation

Replaced registry installation instructions with clone/build setup and consistent source CLI invocation. Source access limitations are explicit. Historical benchmark run data remains unchanged. Validation: fresh engine source build and starter 14/14 with HTML/JSON reports; adapter/workflow checks recorded with the final commit. Commit: this commit (`docs: install 7runs from GitHub source`).

Validation completed 2026-11-27T10:05:18+00:00: starter 14/14; command adapter and exact workflow run/report/check shell steps 21/21; HTTP adapter 21/21; HTML and JSON present. Full anonymous hosted Actions execution remains gated on engine visibility. No model API calls.

## 2026-11-27T10:30:39+00:00 — Organization purchase wording

Commit: this commit (`docs: point organization licensing to purchase pricing`). Updated organization-license links to the current pricing/availability dialog. Install commands, measured results and engine license terms remain unchanged. Checked all Markdown and local target fragments.

## 2026-11-27T10:51:29+00:00 — Source toolchain security alignment

Commit: this commit (`fix: align guide with security-tested source`). Updated the Node prerequisite to22.13+ (22.x) or24+. The profile example workflow now pins engineb01c0747ea12cf697489604fb356b4f98b58c9e3 instead of the earlier dependency lockfile. Engine validation: clean install/build,238 tests pass/5 live skipped, lint pass, full dependency audit0 known advisories; starter14/14, command/HTTP/exact CI run/report/check steps21/21 with HTML/JSON. Anonymous hosted execution remains gated on engine visibility. Historical benchmark data is unchanged.
