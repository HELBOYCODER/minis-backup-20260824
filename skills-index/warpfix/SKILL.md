---
name: warpfix
description: AI-powered CI auto-repair via WarpFix (warpfix.org) using the user's Pro API key. Use when a project needs CI failure diagnosis/repair, fixing broken tests, resolving dependency conflicts, runtime error fixes, CI failure prediction, or repo health checks (doctor). Triggers: "fix ci", "خطای CI", "failing tests", "dependency conflict", "predict failure", "warpfix".
version: 1.0.0
---
# WarpFix — AI CI Auto-Repair

Autonomous CI repair SaaS for GitHub Actions/GitLab. Detects CI failures, generates patches, validates in sandbox, opens fix PRs with a confidence score (0–100).

**User's key is Pro**: unlimited repairs/repos. Key is in GLOBAL.md (`USER_KEY` under "WarpFix").

## API (no CLI install needed)

Auth = `api_key` field inside the JSON body (NOT a header).

```bash
curl -s -X POST https://api.warpfix.org/warp/command \
  -H "Content-Type: application/json" \
  -d '{"command":"<CMD>","api_key":"'"$WARPFIX_API_KEY"'","context":{}}'
```

| Command | Use |
|---|---|
| `doctor`-style check | use `repair-last` + `/health` |
| `fix-ci` | repair latest CI failure |
| `fix-tests` | fix broken tests |
| `fix-deps` | resolve dependency conflicts |
| `fix-runtime` | fix runtime errors |
| `predict-failure` | predict likely failures from diff/history |
| `repair-last` | last repair (status, error_type, confidence, pr_url) |

- Health: `GET https://api.warpfix.org/health` → `{status:"healthy"}`
- Async responses return `{status:"queued", job_id}` → track via `warp/command` or dashboard.
- 429 = plan limit (shouldn't happen on Pro).
- Docs: https://warpfix.org/docs · Free no-signup tools: https://warpfix.org/tools

## Workflow

1. Get git context (repo URL, branch) and include it in `context` if available.
2. Start with `repair-last` / repo CI logs to identify the failing workflow.
3. Run the matching command; poll for job completion.
4. Report back: status, confidence score, PR URL. If confidence < threshold, recommend manual review.

## Repo config (.warpfix.yaml)

```yaml
version: 1
repair:
  auto_fix: true
  confidence_threshold: 0.75   # float 0–1 (NOT 0–100)
quality_gates:
  min_confidence: 0.8
  require_tests: true
  max_patch_lines: 50
review:
  auto_comment: true
ignore:
  paths: ["dist/**"]
```

## PR chat commands

Mention `@warpfix` in PR comments: `explain`, `security`, `test`, `refactor`, `performance`, plus slash commands `/fix-ci`, `/fix-tests`, `/fix-deps`, `/predict-failure`, `/repair-last`.

## CLI alternative

```bash
npm i -g warpfix && warpfix login <key>
warpfix doctor | fix-ci | fix-tests | fix-deps | fix-runtime | predict-failure | status <job-id>
```

Env vars: `WARPFIX_API_KEY`, `WARPFIX_API_URL` (default https://api.warpfix.org).
