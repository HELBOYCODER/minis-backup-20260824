# Global Memory

## GitHub-first problem solving (user standing instruction, 2026-08-23)
- Whenever the agent needs something it doesn't have or doesn't know how to do (tool, method, workaround, integration): **search GitHub FIRST** (`gh search repos` / code search), then elsewhere (Exa/web) if not found. GitHub almost always has a ready solution.
- Every reusable technique/شگرد learned while solving a problem → register as a Minis skill (`/var/minis/skills/<name>/SKILL.md`) so the same approach is always reusable in other projects/sessions.

## WarpFix — use for projects (user instruction, 2026-08-22)
- User asked: whenever a project needs CI repair / test fix / dependency fix / failure prediction, USE WarpFix with his key — the key is on a **Pro plan** (unlimited repairs, unlimited repos).
- API: POST https://api.warpfix.org/warp/command  body: {"command":"<fix-ci|fix-tests|fix-deps|fix-runtime|predict-failure|repair-last>","api_key":USER_KEY,"context":{}}. CLI: `npm i -g warpfix && warpfix login <key>` (default API https://api.warpfix.org).
- USER_KEY = wf_e25f01c5e7c5f0bc67b677a032b208c6baa1ec431e6861c8 (user-provided explicitly for this purpose; validated OK 2026-08-22).
- Repo config: `.warpfix.yaml` in repo root, schema version: 1 (repair.confidence_threshold 0–1, quality_gates, review.auto_comment, ignore.paths). Docs: warpfix.org/docs.

## Auto-continue on interruption
- If a chat is cut off mid-task and internet is available, automatically continue the task without requiring user prompting. Apply to all chats and tasks.