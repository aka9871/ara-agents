# ARA Agents

<p align="left">
  <img src="https://img.shields.io/badge/ARA-v1.1-brightgreen?style=flat-square" alt="ARA v1.1" />
  <img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="MIT" />
  <img src="https://img.shields.io/badge/Claude_Code-available-00c896?style=flat-square" alt="Claude Code" />
  <img src="https://img.shields.io/badge/Cursor-coming_soon-f5a33a?style=flat-square" alt="Cursor" />
  <img src="https://img.shields.io/badge/Opencode-coming_soon-f5a33a?style=flat-square" alt="Opencode" />
</p>

**ARA agents for every AI editor** — audit, generate, enforce, and monitor the [ARA standard](https://github.com/aka9871/ara-standard) on any website, from inside your editor.

> ARA (Agent-Ready Architecture) is the open standard that gives every website a front door for AI agents.  
> One HTTP request to `manifest.json` gives an AI agent complete understanding of any site — in ~300 tokens instead of 50,000.  
> → [ara-standard.org](https://ara-standard.org) · [Full spec](https://github.com/aka9871/ara-standard)

---

## Editor support

| Editor | Status | Directory |
|--------|--------|-----------|
| **Claude Code** | ✅ Available | [`claude-code/`](./claude-code/) |
| **Cursor** | 🔨 In development | [`cursor/`](./cursor/) |
| **Opencode** | 🔨 In development | [`opencode/`](./opencode/) |
| Other editors | Open for contributions | Open a PR |

---

## Claude Code — Quick start

The fastest way to make any site ARA-ready.

### Install

```bash
# Copy agents and skill to your Claude Code config
cp -r claude-code/agents/* ~/.claude/agents/
cp -r claude-code/skills/ara ~/.claude/skills/

# Or run the install script
bash claude-code/install.sh
```

### Commands

```bash
/ara audit https://yoursite.com       # Score A–F across 13 criteria
/ara transform https://yoursite.com   # Generate manifest + schemas + actions + digest.md
/ara enforce https://yoursite.com     # Inject middleware — force AI bots to use ARA
/ara monitor https://yoursite.com     # Measure GEO impact (citation rate, semantic accuracy)
```

### Full workflow

```bash
# 1. Generate all 4 ARA files
/ara transform https://yoursite.com

# 2. Force AI bots (GPTBot, ClaudeBot, PerplexityBot…) to use ARA
/ara enforce https://yoursite.com

# 3. Verify your grade
/ara audit https://yoursite.com       # expect A (80+)

# 4. Measure GEO impact after 7–14 days
/ara monitor https://yoursite.com
```

### The 4 agents

| Agent | Role | Command |
|-------|------|---------|
| `ara-auditor` | Scores any site A–F across 13 criteria, detects llms.txt, checks enforcement signals | `/ara audit <url>` |
| `ara-transformer` | Generates all 4 ARA files: manifest.json, schemas/, actions.json, digest.md | `/ara transform <url>` |
| `ara-enforcer` | Injects content-negotiation middleware for 8+ frameworks (Next.js, nginx, WordPress, Laravel, Django, Cloudflare, Apache, Vercel) | `/ara enforce <url>` |
| `ara-monitor` | Measures GEO impact — citation rate and semantic accuracy across AI search engines | `/ara monitor <url>` |

---

## What ARA generates

```
.well-known/ara/
├── manifest.json       Layer 1 — identity, content map, capabilities, policies  (~150 tokens)
├── schemas/
│   ├── products.json   Layer 2 — semantic schemas with Schema.org annotations   (~250 tokens)
│   └── articles.json
├── actions.json        Layer 3 — agent actions with intent examples              (~350 tokens)
└── digest.md           GEO layer — 200–400 token AI-optimized summary
```

---

## Why ARA instead of llms.txt?

| | llms.txt | ARA |
|--|---------|-----|
| Format | Plain text / markdown links | Structured JSON |
| Data schemas | None | JSON Schema + Schema.org |
| Actions | None | Full query & mutation definitions |
| Protocol support | None | REST, MCP, A2A, GraphQL |
| Access policies | None | Rate limits, auth, data usage |
| Token cost for agents | ~800 tokens | ~150 tokens (manifest only) |
| Machine-readable | Partially | Fully |

ARA replaces llms.txt — or you can keep both. Run `/ara migrate` to convert your existing llms.txt.

---

## Enforcement: forcing AI bots to use ARA

AI bots (GPTBot, ClaudeBot, PerplexityBot, Google-Extended, and 10 others) don't know ARA exists yet.  
`ara-enforcer` solves this with server-side content negotiation:

```
GPTBot visits your site
→ server detects User-Agent
→ 302 redirect to /.well-known/ara/digest.md
→ bot reads 300 tokens of structured context
→ done — no ARA knowledge required on the bot's side
```

Supported stacks: **Next.js, Cloudflare Worker, nginx, Apache, WordPress, Laravel, Django, Vercel**

---

## Contributing

### Adding a new editor

1. Fork this repo
2. Create a directory: `your-editor/`
3. Implement the 4 agents (`ara-auditor`, `ara-transformer`, `ara-enforcer`, `ara-monitor`)
4. The Claude Code agents in `claude-code/` are the reference implementation
5. Open a PR

### Improving existing agents

- Bug reports and improvements → [Issues](https://github.com/aka9871/ara-agents/issues)
- Discussions → [Discussions](https://github.com/aka9871/ara-agents/discussions)

---

## Related

- **[ara-standard](https://github.com/aka9871/ara-standard)** — The ARA specification (v1.1) + npx CLI tools
- **[ara-standard.org](https://ara-standard.org)** — Documentation site

## License

MIT — ARA Standard Contributors
