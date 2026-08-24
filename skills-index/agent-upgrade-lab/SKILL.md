---
name: agent-upgrade-lab
description: Deploy recipes + install status for top agent-upgrading OSS projects (mem0, letta, deer-flow, nanobot, browser-use, camofox, ECC, oh-my-claudecode, evals). Use when user wants to make an agent smarter, add memory, stealth browsing, multi-agent, or observability.
---

# Agent Upgrade Lab

Curated scan: /var/minis/workspace/agent-upgrades.md · Expert roles: skill `agency-expert-roles`

## INSTALLED in this Minis (2026-08-23)
- Skills: anthropics pack (docx/pptx/xlsx/pdf/mcp-builder/frontend-design/webapp-testing/...), superpowers (`sp-*`: brainstorming, systematic-debugging, TDD, writing-plans, verification-before-completion...), taste pack (`taste-*`), agency roles index.
- MCP servers: context7 (lib docs), deepwiki (ask any GitHub repo), gitmcp (repo docs fetch) — all keyless, verified alive.
- Pending VPS: mem0 memory layer, camofox stealth browser (VPS sshd was refusing at install time).

## Deploy recipes

### mem0 (memory layer w/ semantic retrieval)
Needs an OpenAI-compatible LLM (extraction) + embeddings provider. On user VPS (950MB RAM): pip venv + `pip install mem0ai fastapi uvicorn`, vector_store=faiss local, llm base_url=http://127.0.0.1:20128/v1 (9router, free oc model). BLOCKER last attempt: no /v1/embeddings route on 9router → fix = set a free Gemini API key (text-embedding-004) as MEM0_EMBEDDINGS_KEY env, then deploy. Expose FastAPI on 127.0.0.1:8021; wrap as MCP or call via curl from sessions.

### letta (stateful agents platform)
Docker compose on a bigger box (needs Postgres ~500MB+). Not viable on 950MB VPS alongside xray+chrome.

### bytedance/deer-flow — deep-research superagent
Python 3.12 + Node 22 backend/frontend; heavy. Run on Vercel/Railway instead of VPS, or locally with pnpm.

### HKUDS/nanobot — lightweight personal agent
`pip install nanobot` style, WebUI included; good candidate to self-host on VPS if RAM freed (~300MB).

### Browser automation
- sandbox: use built-in browser_use tool first.
- VPS stealth (CF battles): **camofox-browser** — Camoufox(Firefox)+playwright; needs ~400MB RAM headroom; install: pip install camoufox[geoip] && python -m camoufox fetch; complements FlareSolverr-Go already on .212.
- vercel-labs/agent-browser: CLI `npx agent-browser` — runs anywhere Node runs.

### Multi-agent
- oh-my-claudecode: teams-first orchestration for Claude-Code-style agents — reference for our own loops.
- ruflo: swarm meta-harness; openai/swarm: minimal patterns worth copying into skills.

### Evals/observability
langwatch / openlit / RagaAI Catalyst — point OpenTelemetry traces at them when we run production agents on VPS.

## Methodology must-reads
humanlayer/12-factor-agents (25k★) · obra/superpowers docs · anthropics/skills authoring guide (skill `skill-creator-anthropic`).
