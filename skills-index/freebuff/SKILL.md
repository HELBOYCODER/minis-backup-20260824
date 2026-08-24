# Freebuff — free AI coding agent (non-interactive driver)

Freebuff (npm `freebuff`, by CodebuffAI) = free terminal coding agent. Models: DeepSeek V4 Flash (default), GPT-5.6 Luna, MiMo 2.5 (unlimited fallback). Free w/ text ads. Login account: HELBOY CODER (quilt-that-refract@duck.com).

## Setup already done in this sandbox
1. `npm i -g freebuff` → downloads glibc binary to `/root/.config/manicode/freebuff`.
2. Alpine is musl → binary won't run directly. Fix in place:
   - Debian/Ubuntu arm64 libs extracted to `/opt/debroot/usr/lib/aarch64-linux-gnu` (from ubuntu-base-22.04.5 arm64 tarball).
   - `/root/.config/manicode/freebuff` is now a **sh wrapper**: execs `$L/ld-linux-aarch64.so.1 --library-path $L freebuff.bin "$@"`. Real binary renamed `freebuff.bin`. If an update re-downloads the raw binary, redo the mv+wrapper dance.
3. Auth stored at `/root/.config/manicode/credentials.json` (authToken). Re-login: run `freebuff login` (background), open printed URL in sandbox browser, click "Continue with GitHub" (GitHub OAuth works there; Google doesn't).

## Running tasks non-interactively (the trick)
CLI has NO prompt arg — TUI only. Driver: `/var/minis/workspace/freebuff-driver/fb_run.py`, wrapper `/usr/local/bin/fb-task`.

```sh
printf 'TASK TEXT\n' > /tmp/task.txt
setsid nohup fb-task <project_dir> /tmp/task.txt [max_sec=900] [idle=75] > /tmp/fb-driver.log 2>&1 &
```

How it works:
- pty.fork spawns TUI (50x160), waits for "Enter a coding task" marker, types prompt+\r, confirms start via tool-name keywords in stream.
- **Completion signal = structured session log**, not screen scraping: watches newest dir under `/root/.config/manicode/projects/*/chats/*` created after launch; polls `log.jsonl` for `"Main prompt finished"` event; grabs `fullResponse` from the `shouldEndTurn:true` End-agent event.
- Result answer → `/tmp/fb-answer.txt`; raw TUI dump → `/tmp/fb-tui-last.log`; project files changed in place. I then verify/run code myself.

## Critical gotchas
- **bash inside freebuff FAILS in PRoot** ("terminal command broker … protocol response was missing"). Prompt template auto-appends: "don't run commands, only create/edit files" (disable with FB_ALLOW_BASH=1). Agent writes code; sandbox shell verifies.
- Quota: ~5 premium sessions/day (DeepSeek/Luna), falls back to MiMo 2.5 "unlimited". Iran = limited-mode country but full mode worked here.
- Chat logs also contain every write_file content — useful to recover what changed without git.
- Session dirs map cwd→short name (`fb-test`, not full path) — don't build watch path from cwd, glob all projects.
