# You.com Web Intelligence API (ydc)

Paid web search/answer/research API. Key: `$200` free credit account (user: Start free $100 + work-email bonus). Balance dashboard: https://you.com/platform/analytics

## Key resolution
- Env var `YOU_API_KEY` (app settings) OR file `/var/minis/workspace/.secrets/youcom.env` (chmod 600, `YOU_API_KEY=...`). Never echo the key.

## CLI (installed at /usr/local/bin/ydc)
```
ydc search "query" [count]        # web+news results w/ snippets — ~$0.005/call
ydc search-full "q" [count]       # + full-page markdown extraction ($1 per 1k pages)
ydc answer "question"             # cited NL answer, {answer, citations[], results}
ydc research "input" [effort]     # multi-hop report: lite|standard|deep|exhaustive|frontier
ydc finance "question" [effort]   # finance-optimized research (earnings/filings)
ydc contents URL [URL...]         # clean markdown/html of pages
```

## Raw endpoints (auth header `X-API-Key: <key>`)
- POST https://ydc-index.io/v1/search  body {"query","count","extraction":{...}}
- POST https://api.you.com/v1/answer   body {"query"}
- POST https://api.you.com/v1/research body {"input","research_effort"} → {output:{content,sources}}
- POST https://api.you.com/v1/finance_research same shape as research
- POST https://ydc-index.io/v1/contents body {"urls":[...]}
- OLD base api.ydc-index.io GET /search is DEAD → returns Forbidden. Use POST /v1 above.
- Search response may include `results.web` AND `results.news` when query says "news".

## MCP (native Minis)
- Server entry "you" in /var/minis/mcp-servers/servers.json → https://api.you.com/mcp with `Authorization: Bearer $$YOU_API_KEY` (requires YOU_API_KEY env var set in Settings → Environments).
- Tools: you-search, you-contents, you-answer, you-research, you-finance.
- Keyless fallback (search only, 100 q/day): https://api.you.com/mcp?profile=free

## When to use what
- Quick fact w/ citations → `answer`. Deep multi-source brief → `research lite/standard`. Stock/market question → `finance`. Need raw URLs/snippets for own RAG → `search` (+extraction only if truly needed, costs more).
