---
name: cf-bypass
description: Permanent Cloudflare-challenge & captcha solving toolkit for all projects — self-hosted FlareSolverr-Go API on user's VPS (95.182.91.212:8191, localhost) + puppeteer-core kit for form automation (signups, reCAPTCHA v3 tokens, Turnstile pages). Use whenever a project hits a Cloudflare interstitial/managed challenge, cf_clearance is needed, or a signup/verification flow must be automated with a real Chrome.
version: 1.0.0
---

# CF-Bypass Toolkit

## What's installed (permanent, on VPS 95.182.91.212)

| Piece | Path | Purpose |
|---|---|---|
| FlareSolverr-Go v0.8.1 | `/usr/local/bin/flaresolverr-go` | HTTP API that solves Cloudflare challenges, returns cookies+UA+HTML |
| chrome-headless-shell 152 | `/opt/chrome-headless-shell-linux64/chrome-headless-shell` | Lightweight browser used by solver |
| Full Chrome-for-Testing 152 | `/opt/chrome-linux64/chrome` | Heavier stealth option (1GB-RAM box: use sparingly) |
| systemd unit | `/etc/systemd/system/cfsolver.service` | Runs solver, `BROWSER_POOL_SIZE=1`, `MAX_MEMORY_MB=450` |
| Node kit | `/root/cfkit/` | puppeteer-core scripts (`serv00-register.js` pattern) |

## Solver API (FlareSolverr-compatible)

```bash
# health
curl http://127.0.0.1:8191/health

# solve a CF-protected GET
curl -s -X POST http://127.0.0.1:8191/v0 -H 'Content-Type: application/json' \
  -d '{"cmd":"request.get","url":"https://TARGET.example","maxTimeout":60000}'
# -> solution.cookies (incl. cf_clearance), solution.userAgent, solution.response (HTML)

# POST through challenge: {"cmd":"request.post","postData":"a=b&c=d", ...}
# sessions: {"cmd":"sessions.create"} then "request.get" with "session":"<id>"
```

From OUTSIDE the VPS keep it localhost-only; tunnel when needed:
`ssh -L 8191:127.0.0.1:8191 root@95.182.91.212`

Env knobs (unit file): `BROWSER_POOL_SIZE`, `MAX_MEMORY_MB`, `DEFAULT_TIMEOUT`, `HEADLESS`,
`PROXY_URL`/`PROXY_LIST`, `CLEARANCE_CACHE_ENABLED`, `API_KEY_ENABLED`+`API_KEY`.

## Signup / reCAPTCHA-v3 pattern (puppeteer-core)

reCAPTCHA v3 tokens are domain-bound → generate them ON the target page, not remotely:

```js
const browser = await puppeteer.launch({
  executablePath:'/opt/chrome-headless-shell-linux64/chrome-headless-shell',
  args:['--no-sandbox','--disable-gpu','--disable-dev-shm-usage']});
const page = await browser.newPage();
await page.goto(url,{waitUntil:'domcontentloaded',timeout:120000}); // networkidle2 often never fires
await page.waitForSelector('#form input[name=x]',{timeout:60000});
// fill fields, then click submit — page's own JS calls grecaptcha.execute() itself
```

Gotchas learned:
- `waitUntil:'networkidle2'` times out on pages loading recaptcha/analytics → use `domcontentloaded`.
- Long SSH sessions to this VPS drop under load → run jobs with
  `setsid nohup node x.js > /tmp/x.log 2>&1 < /dev/null &`, then poll the log in short SSH calls.
- Google reCAPTCHA is UNREACHABLE from the Minis sandbox (000 responses) → any flow needing
  grecaptcha/Turnstile MUST run on the VPS, not in sandbox browser_use.
- Sandbox browser CAN pass plain forms (no captcha); Turnstile-managed challenges may still fail
  headless → fall back to solver API or full-Chrome run.
- VPS has ~950MB RAM: one chrome at a time; `OOMScoreAdjust=500` set in unit.

## Temp-email companion (for signups)
- Worker: https://temp-mail.vickfmr.workers.dev — `POST /api/pub/new` → `{addr}`;
  `GET /api/inbox?addr=<a>&mode=public`; body via `/api/pub/msg`. Telegram bot @tempmailersaz_bot.
