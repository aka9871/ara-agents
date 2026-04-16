# ARA Agents for Cursor

> Status: **In development** — contributions welcome

This directory will contain ARA agents adapted for [Cursor](https://cursor.sh).

## Planned agents

| Agent | What it will do |
|-------|----------------|
| `ara-auditor` | Score any site A–F from inside Cursor |
| `ara-transformer` | Generate all 4 ARA files from a URL or local project |
| `ara-enforcer` | Inject content-negotiation middleware for 8+ frameworks |
| `ara-monitor` | Measure GEO impact after deployment |

## Want to contribute?

The Claude Code agents in [`../claude-code/`](../claude-code/) are the reference implementation.  
Cursor agents should expose the same commands via Cursor's agent API.

Open a PR or start a discussion at [github.com/aka9871/ara-agents](https://github.com/aka9871/ara-agents).
