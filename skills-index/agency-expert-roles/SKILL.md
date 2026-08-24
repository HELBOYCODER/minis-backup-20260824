---
name: agency-expert-roles
description: Index of 267 plug-and-play AI expert role prompts (engineering/design/marketing/finance/academic...) stored at /var/minis/shared/agency-agents/. Use when user asks for a specialist persona, expert review panel, or role-based agent team.
---

# Agency Expert Roles (267 roles)

Source: jnMetaCode/agency-agents-zh (19.8k★), cloned to **/var/minis/shared/agency-agents/** (2026-08-23).
Prompts are Chinese markdown; output language follows the user anyway.

## Layout
- `AGENT-LIST.md`, `CATALOG.md` — full catalogs (start here)
- Category dirs: academic/, design/, engineering/, marketing/, finance/, ... each containing `<role>.md` prompt files
- `.sh` helpers + sample outputs

## Usage pattern
1. `ls /var/minis/shared/agency-agents/<category>/`
2. Read the chosen role file → adopt that persona/system-prompt for the task, or spawn parallel reviewer roles with it.
3. For panels (e.g. code review): pick 2–4 relevant roles, run their lenses sequentially, merge verdicts.

## Best-of picks
Engineering: architect-reviewer, security-auditor, debugger, devops-troubleshooter.
Design: ui-polisher, typography-specialist. Marketing: seo-content-writer, growth-hacker.
