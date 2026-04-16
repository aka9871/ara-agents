# ARA Agents for Claude Code

4 agents covering the full ARA lifecycle, available as a `/ara` skill.

## Install

```bash
cp -r agents/* ~/.claude/agents/
cp -r skills/ara ~/.claude/skills/
```

Or run: `bash install.sh`

## Commands

```bash
/ara audit https://yoursite.com       # Score A–F, detect gaps
/ara transform https://yoursite.com   # Generate all 4 ARA files
/ara enforce https://yoursite.com     # Inject AI bot middleware
/ara monitor https://yoursite.com     # Measure GEO impact
```

## Structure

```
claude-code/
├── agents/
│   ├── ara-auditor.md      Score A–F across 13 criteria
│   ├── ara-transformer.md  Generate manifest + schemas + actions + digest.md
│   ├── ara-enforcer.md     Inject middleware for 8+ frameworks
│   └── ara-monitor.md      GEO impact measurement
├── skills/
│   └── ara                 The /ara skill that routes commands to agents
├── middleware/             Reference middleware (Next.js, nginx, WordPress…)
├── examples/               Example ARA files (ecommerce, saas)
└── install.sh              One-command install
```

Full documentation → [../README.md](../README.md)
