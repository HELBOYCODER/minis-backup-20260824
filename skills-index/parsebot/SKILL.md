# Parse.bot Skill — Web Scraping API Agent Toolkit

## Overview
[Parse.bot](https://parse.bot) turns ANY website into a structured REST API + MCP server.  
Key: `pmx_8bd5b8eff4d1b1031e82b6173ad59d07` (env `PARSE_API_KEY`)

**Free plan**: 200 credits, 5 req/min rate limit. Enough for hundreds of calls.

## How it works
- **Marketplace** — 3,300+ pre-built APIs for popular sites (Zillow, Amazon, Torob, booking, etc.). Search/call them instantly.
- **Build custom** — POST /dispatch with a URL + description → agent builds a typed API (~1-2 min).
- **MCP Server** — `POST https://api.parse.bot/mcp` with `X-API-Key`. Connect to Claude, Cursor, any agent.
- **Read & Act** — GET endpoints for scraping, POST endpoints for actions (book, fill form, reserve).

## CLI Usage (`parsebot`)
```bash
# Search marketplace (no API key needed)
parsebot search "torob iran" --limit 3
parsebot endpoints "flight prices" --limit 5

# Get listing detail + canonical_scraper_id
parsebot detail <listing_id>

# Call canonical directly (any API key works)
curl -X POST https://api.parse.bot/scraper/{canonical_id}/{endpoint} \
  -H "X-API-Key: $PARSE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"params":"here"}'

# Subscribe (get your own pinned copy)
parsebot subscribe <listing_id>

# Build custom API from URL
parsebot build "https://example.com" "get product prices and availability"
parsebot wait <task_id>
parsebot call <scraper_id> <endpoint> '{"param":"value"}'

# Revise in English
parsebot revise <task_id> "add an endpoint to search by category"

# MCP tools
parsebot mcp marketplace_search '{"q":"hotels"}'
parsebot mcp call_endpoint '{"scraper_id":"...","endpoint":"search","params":{"location":"NYC"}}'
```

## Key differences vs browser-based scraping
- **100× faster** — no headless browser, talks to network layer directly
- **Structured JSON** — typed endpoints, clean return schemas
- **Self-healing** — auto-updates when target site changes
- **Actions** — can book flights, fill forms, reserve tables via API

## When to use
- Agent needs to scrape a site → search marketplace first, then build custom
- Agent stuck on "can't do that" with browser_use → build a Parse API, call it directly
- Need structured data from complex JS sites → Parse handles it
- Need to perform actions on web (book, reserve, submit) → build POST endpoints

## Existing marketplace APIs relevant to user
- **torob.com** (11 endpoints, 1 credit each) — search products, get details/sellers/specs
- **zillow.com** (8 endpoints) — search homes, property details, agents
- **hotels.com** / flightconnections / flightradar24
- **hackernews**, **amazon reviews**, **indeed jobs**, **divar.ir**, **irantalent.com**
- Many more: real estate, travel, ecommerce, finance, jobs, crypto, maps, etc.

Set PARSE_API_KEY before using. Free tier has 200 credits + 5 req/min.