# Claude ARA Agents

> Make any website understandable by AI agents — in minutes, with Claude Code.

This repository provides **three Claude Code agents** and a `/ara` skill that implement the **ARA (Agent-Ready Architecture)** standard for any website.

```bash
/ara transform https://yoursite.com   # Generate ARA files
/ara enforce https://yoursite.com     # Force AI bots to use ARA
```

---

## The Problem Nobody Is Talking About

ChatGPT, Perplexity, Gemini, and Claude visit your website every day.

They parse **50,000+ tokens of raw HTML** to understand what your site does — your products, your services, your prices. They don't read your `llms.txt`. They don't know ARA exists. They just scrape.

This costs AI companies millions in compute. It means they index your site slowly, inaccurately, and expensively. And you have no control over what they find.

**ARA changes the equation — but only if you force it.**

The `ara-enforcer` agent detects your tech stack and injects signals at 4 layers of your server so that known AI bots receive structured ARA content **proactively** — without needing to know ARA exists.

```
Before ARA enforcement:
  GPTBot → fetches 4 pages → parses 100,000 tokens → understands 60% of your site

After ARA enforcement:
  GPTBot → hits your homepage → server detects User-Agent → 302 redirect
         → reads digest.md (300 tokens) → understands 100% of your site
```

**300x fewer tokens. 1 HTTP request. Complete, structured context.**

This is not just technical optimization. It's the mechanism by which ARA becomes a standard:
when enough sites redirect AI bots to structured ARA content, every major LLM company has a direct financial incentive to adopt ARA natively. You create the adoption pressure.

---

## The Four Agents

| Agent | What it does | When to use |
|-------|-------------|-------------|
| `ara-auditor` | Scores ARA compliance A-F, detects llms.txt | Before transforming |
| `ara-transformer` | Generates manifest.json, schemas/, actions.json, digest.md | Making a site ARA-ready |
| `ara-enforcer` | Injects signals that force AI bots to use ARA | After ARA files exist |
| `ara-monitor` | Measures GEO impact — citations, accuracy, bot traffic | After 7+ days live |

---

## Why ARA instead of llms.txt

`llms.txt` is a good first step. ARA is where it leads.

| | llms.txt | ARA |
|--|---------|-----|
| **Format** | Plain text, markdown links | Structured JSON |
| **What it tells an agent** | "Here are some links" | "Here's what I am, what I contain, and what you can do" |
| **Data schemas** | None | JSON Schema + Schema.org annotations |
| **Agent actions** | None | Full query/mutation definitions |
| **Protocol support** | None | REST, MCP, A2A, GraphQL |
| **Access policies** | None | Rate limits, auth scopes, data usage rights |
| **Agent token cost** | ~800 tokens to parse | ~150 tokens for manifest |
| **GEO layer** | None | `digest.md` — 300-token AI-optimized summary |
| **Machine-readable** | Partially | Fully |
| **Intent mapping** | None | Natural language → action routing |
| **Validation** | None | `npx ara-validate` — score A-F |

### The core difference

`llms.txt` tells a language model where to look.  
ARA tells an AI agent **what the site is, how it's structured, and what it can do** — without loading a single HTML page.

```
llms.txt                          ARA manifest.json
─────────────────────────         ─────────────────────────────────────────
# My Site                         {
                                    "$ara": "1.0",
> A SaaS platform                   "identity": {
                                      "name": "MySite",
## Docs                               "type": "saas",
- [Getting Started](...)              "description": "Project management..."
- [API Reference](...)              },
                                    "content_map": {
## Features                           "resources": [
- [Pricing](...)                        { "id": "projects", "count": 1200,
- [Changelog](...)                        "schema_ref": "schemas/project.json" }
                                        ]
                                    },
                                    "capabilities": {
                                      "actions_ref": "actions.json"
                                    }
                                  }
```

An agent reading `llms.txt` must still fetch and parse every linked page to understand what the site does.  
An agent reading `manifest.json` **already knows** — before making a single additional request.

### Token cost comparison

| Task | llms.txt | ARA |
|------|---------|-----|
| Understand what the site is | ~400 tokens (parse prose) | ~80 tokens (read `identity`) |
| Find what data is available | ~1,200 tokens (fetch + parse pages) | ~150 tokens (read `content_map.resources`) |
| Know what actions are possible | Not possible | ~300 tokens (read `actions.json`) |
| Total to understand + act | **~2,000–5,000 tokens** | **~150–400 tokens** |

**ARA reduces agent token consumption by 10-20x.**

### Migration path

Have a `llms.txt`? Keep it. ARA is additive.

```bash
/ara migrate https://yoursite.com
```

This reads your existing `llms.txt` and converts every section into proper ARA structure — no data lost, everything promoted to machine-readable format.

---

## What gets generated

```
.well-known/ara/
├── manifest.json         Layer 1 — Identity, content map, capabilities, policies
├── schemas/
│   ├── products.json     Layer 2 — Semantic schemas with Schema.org annotations
│   └── articles.json
├── actions.json          Layer 3 — Queries and mutations an agent can execute
└── digest.md             GEO layer — ~300-token LLM-optimized summary
```

### Layer 1 — Discovery (`manifest.json`)

The single entry point. An agent reads this one file and understands the entire site.

```json
{
  "$ara": "1.0",
  "identity": {
    "name": "ByteWire",
    "type": "news_media",
    "description": "Technology news with 2,400+ articles across AI, Web Dev, Security, Open Source, Hardware."
  },
  "content_map": {
    "summary": "Tech news site, 2,400 articles, 5 categories",
    "resources": [
      { "id": "articles", "type": "content", "count": 2400, "schema_ref": "schemas/article.json" }
    ]
  },
  "capabilities": { "actions_ref": "actions.json" },
  "policies": { "agent_access": "public", "rate_limit": { "requests_per_minute": 30 } }
}
```

### Layer 2 — Understanding (`schemas/`)

Every resource gets a semantic schema. Each field maps to a Schema.org property — this is the GEO layer that makes content citable by AI engines.

```json
{
  "$ara_schema": "1.0",
  "extends": "schema:Article",
  "properties": {
    "title":    { "type": "string", "semantic": "schema:headline" },
    "author":   { "type": "string", "semantic": "schema:author" },
    "category": { "type": "string", "semantic": "schema:articleSection" }
  },
  "search_hints": {
    "filterable_by": ["category", "author"],
    "text_searchable": ["title", "description"]
  }
}
```

### Layer 3 — Interaction (`actions.json`)

Everything an agent can do, with natural language intent mappings.

```json
{
  "actions": [
    {
      "id": "search_articles",
      "type": "query",
      "intents": ["Find articles about AI agents", "Latest posts on LLMs", "Security news this week"],
      "input": { "query": "string", "category": "string" },
      "protocols": {
        "rest":     { "method": "GET", "path": "/api/articles" },
        "mcp_tool": { "name": "search_articles" }
      }
    }
  ]
}
```

### GEO layer (`digest.md`)

A 200–400 token plain-text summary optimized for AI engines (ChatGPT, Perplexity, Claude, Gemini). Dense in facts, no marketing language.

```markdown
# ByteWire — Agent Digest

## Identity
Technology news site. 2,400+ articles published since 2019. 5 categories: AI/LLMs (840 articles),
Web Dev (510), Security (380), Open Source (420), Hardware (250).

## Key Capabilities
- Search articles by keyword, category, author, or date range
- Fetch full article content with author bio
- Filter by publication date or tag

## Policies
- Access: public, no authentication
- Rate limit: 30 req/min, caching allowed (30-min TTL)
- License: CC BY-NC 4.0
```

---

## Installation

### Requirements

- [Claude Code](https://claude.ai/code) CLI installed
- `~/.claude/agents/` directory (auto-created by Claude Code)

### Install (one command)

```bash
curl -fsSL https://raw.githubusercontent.com/ARA-Standard/claude-ara-agents/main/install.sh | bash
```

### Install manually

```bash
# Clone the repo
git clone https://github.com/ARA-Standard/claude-ara-agents.git
cd claude-ara-agents

# Run the installer
./install.sh
```

### What gets installed

```
~/.claude/agents/ara-auditor.md      Audit agent (read-only)
~/.claude/agents/ara-transformer.md  Generator agent (writes files)
~/.claude/skills/ara/SKILL.md        /ara slash command
```

---

## Usage

### Audit any site

```bash
/ara audit https://yoursite.com
```

Returns a scored report (A–F) with breakdown, issues, and next steps.

### Transform any site to ARA

```bash
/ara transform https://yoursite.com
```

Generates the full ARA file set in `.well-known/ara/`. Works with live URLs and local codebases.

### Migrate from llms.txt

```bash
/ara migrate https://yoursite.com
```

Detects your `llms.txt`, extracts all content, and converts it into proper ARA structure.

### Optimize for AI search (GEO)

```bash
/ara geo https://yoursite.com
```

Regenerates `digest.md` and adds Schema.org `semantic` annotations to all schemas. Improves citability in ChatGPT, Perplexity, Google AIO, and Claude.

### Quick compliance check

```bash
/ara quick https://yoursite.com
# → Level: 1 | Grade: C (72/100) | Top issue: actions.json absent
```

### Validate with the official CLI

```bash
/ara validate https://yoursite.com
# Runs: npx ara-validate https://yoursite.com
```

---

## Command reference

| Command | Description | Agents |
|---------|-------------|--------|
| `/ara audit <url>` | Full compliance report, A-F score | ara-auditor |
| `/ara transform <url>` | Generate all ARA files | ara-auditor → ara-transformer |
| `/ara enforce <url>` | Inject signals that force AI bots to use ARA | ara-enforcer |
| `/ara migrate <url>` | Convert llms.txt → ARA | ara-auditor → ara-transformer |
| `/ara validate <url>` | Run npx ara-validate | Bash |
| `/ara geo <url>` | Optimize digest.md + Schema.org | ara-transformer |
| `/ara quick <url>` | 30-second snapshot | ara-auditor |

All commands also work on local paths:

```bash
/ara transform ./my-project
/ara enforce ./my-project
/ara audit ./my-project/.well-known/ara
```

---

## Forcing AI Agents to Use ARA (the key to GEO)

Generating ARA files is step one. Step two is making sure AI agents actually use them.

The `ara-enforcer` agent implements **4 signal layers** in your server:

### Layer 1 — HTTP headers (every response)
```
Link: </.well-known/ara/manifest.json>; rel="ara-manifest"
X-ARA-Manifest: /.well-known/ara/manifest.json
X-ARA-Version: 1.0
```
Agents that check response headers find the manifest without knowing ARA.

### Layer 2 — HTML `<head>` hints
```html
<link rel="ara-manifest" href="/.well-known/ara/manifest.json">
<meta name="ara:digest" content="/.well-known/ara/digest.md">
```
Agents that parse HTML find the reference before reading any page content.

### Layer 3 — JSON-LD `potentialAction`
```json
{
  "@type": "WebSite",
  "potentialAction": {
    "@type": "ReadAction",
    "target": "/.well-known/ara/manifest.json",
    "additionalType": "https://ara-standard.org/schema/manifest/v1"
  }
}
```
Agents that extract Schema.org data find the ARA manifest embedded in structured data they already process.

### Layer 4 — Content negotiation (the most powerful)
When a known AI bot visits, the server **redirects to `digest.md` before the bot parses any HTML**:

```
User-Agent: GPTBot/1.0  →  302  →  /.well-known/ara/digest.md
```

The bot reads 300 tokens of structured context instead of 50,000 tokens of HTML. **It doesn't need to know ARA exists.**

AI bots detected and redirected:
`GPTBot` · `ClaudeBot` · `PerplexityBot` · `Google-Extended` · `anthropic-ai` · `ChatGPT-User` · `Bytespider` · `YouBot` · `FacebookBot` · `Applebot` · `cohere-ai` · `AI2Bot`

### Supported stacks

```bash
/ara enforce ./my-nextjs-app     → generates middleware.ts
/ara enforce ./my-wp-site        → generates WordPress plugin + functions.php additions
/ara enforce ./nginx-config      → generates nginx include block
/ara enforce ./apache-site       → generates .htaccess additions
/ara enforce ./cloudflare        → generates Cloudflare Worker
/ara enforce ./django-app        → generates Python middleware + settings.py snippet
/ara enforce ./laravel-app       → generates PHP middleware + Kernel.php registration
/ara enforce https://yoursite.com → detects stack from live site, generates all applicable configs
```

### Why this creates adoption pressure for the standard

When 10,000 sites redirect `GPTBot` to ARA digests instead of HTML, OpenAI sees:
- 300x cheaper crawl cost for ARA-ready sites
- More accurate indexing (structured data vs HTML scraping)
- A clear financial incentive to check `/.well-known/ara/manifest.json` natively

**Your enforcement of ARA today is the lobby for ARA becoming a web standard tomorrow.**

See [`middleware/README.md`](middleware/README.md) for the full technical explanation and verification commands.

---

## How ara-monitor Works

> "How do I know ARA is actually helping my GEO?"
>
> That question has no good answer with `llms.txt`. With ARA, it does — because ARA gives you a verifiable ground truth.

### The core insight: ARA as a test suite

Every ARA-ready site contains three types of **declared facts**:

- `digest.md` — facts the AI *should* know: "3,200 products, ships to 45 countries, founded 2019"
- `schemas/*.json` — the exact structure of every data type
- `actions.json` — the exact questions users ask: "find laptops under $1000", "show me wireless headphones"

`ara-monitor` uses these declared facts as a **test suite**. It submits the intents as real queries to AI search engines, then compares the answers against the ARA ground truth. The gap between what ARA declares and what AI engines actually say is your GEO score.

This is measurement that `llms.txt` can never provide — because it has no ground truth.

---

### The 7-step pipeline

```
                    ┌─────────────────────────────────────────┐
                    │           ara-monitor pipeline           │
                    └─────────────────────────────────────────┘

  Step 1            Step 2            Step 3            Step 4
  ────────          ────────          ────────          ────────
  Load ARA     →   Parse server  →   Score ARA    →   Run AI
  ground truth      access logs       file health       citation probes

  "What should      "Are AI bots      "Are your ARA     "Are AI engines
  AI know?"         finding ARA?"     files fresh?"     citing you?"

  Step 5            Step 6            Step 7
  ────────          ────────          ────────
  Semantic     →   Compare vs   →   Save report +
  accuracy          previous          recommendations
  check             run

  "What do they     "Is it           .ara-monitoring/
  actually say?"    improving?"      report-YYYY-MM-DD.json
```

---

### Step 1 — Build the ground truth

The agent reads your ARA files to build the expected facts:

```
manifest.json  → site identity, resource counts, declared protocols
digest.md      → key facts: every sentence containing a number, price, date, or proper noun
actions.json   → intents[].examples → these become AI search queries
schemas/*.json → field definitions with Schema.org mappings
```

Example — what gets extracted from a fashion ecommerce `digest.md`:

```markdown
# UrbanStyle — Agent Digest
## Identity
3,200+ products. Founded 2019, New York. Ships to 45 countries.
Prices: $45 (entry) to $380 (premium). Average rating 4.6/5.
```

Extracted facts (the test checklist):
```
fact_1: "3,200+ products"
fact_2: "founded 2019"
fact_3: "ships to 45 countries"
fact_4: "price from $45"
fact_5: "rating 4.6/5"
```

---

### Step 2 — Server log analysis

The agent parses your web server access logs to answer: **are AI bots finding and using your ARA files?**

```bash
# What the agent runs (nginx example)
grep -iE "GPTBot|ClaudeBot|PerplexityBot|Google-Extended" access.log \
  | grep -oiE "GPTBot|ClaudeBot|PerplexityBot|Google-Extended" \
  | sort | uniq -c | sort -rn

# Did the 302 redirect work?
grep -iE "GPTBot|ClaudeBot" access.log | grep " 302 " | wc -l

# Did bots actually read digest.md?
grep "/.well-known/ara/digest.md" access.log | grep " 200 " | wc -l

# Did any bot go directly to manifest.json? (knows ARA natively)
grep "/.well-known/ara/manifest.json" access.log | grep " 200 " | wc -l
```

**What the numbers mean:**

| Metric | What it tells you |
|--------|-------------------|
| `bot_visits_total` | How often AI bots index your site |
| `redirect_rate` | % of bot visits caught by `ara-enforcer` — should be >80% |
| `digest_reads` | How many bots successfully read the ARA digest |
| `manifest_reads` | Bots that found ARA *without* the redirect — native ARA support |

**The signal you want to see grow over time:**
`manifest_reads` increasing means AI companies are starting to check `/.well-known/ara/manifest.json` natively. That's ARA becoming a standard.

---

### Step 3 — ARA file health score

Before running the citation probes, the agent checks whether your ARA files are fresh and complete enough to produce accurate results. A stale `digest.md` will produce misleading accuracy scores.

```
manifest.json health (40 pts):
  - Is meta.generated_at recent? (< 30 days = fresh, > 90 days = stale)
  - Do resource counts match reality?
  - Is identity.description factual? (no adjectives like "powerful", "innovative")
  - Are declared protocols actually implemented?

digest.md health (20 pts):
  - Token count: 200–400 tokens (under 150 = too sparse, over 500 = too verbose)
  - Fact density: target 1 specific fact per sentence (number, name, price, date)
  - Does it mention the most important things for your site type?
    (ecommerce: prices + counts; SaaS: pricing plans + features; local: hours + location)

actions.json health (20 pts):
  - Every intent has 3+ natural language examples
  - Actions cover the main user intents for your site type
  - Protocols declared match what's actually available

schemas/ health (20 pts):
  - Every field has a Schema.org `semantic` annotation
  - search_hints defined for all catalog resources
  - Relationships between resources declared
```

A low health score before running citation probes means: fix your ARA files first, then measure.

---

### Step 4 — AI citation probes (the core measurement)

This is where the magic happens. The agent takes the `intents[].examples` from `actions.json` and submits them as **real queries to AI search engines**.

**Why intents are perfect test probes:**
The intent examples were written to represent what real users ask. If a user says "find laptops under $1000" to ChatGPT, and ChatGPT browses the web to answer — is your site in the results? That's exactly what the probe tests.

**The probe flow:**

```
actions.json intent:
  "examples": ["find laptops under $1000", "wireless headphones comparison", "gaming monitor 4K"]
                         ↓
Agent selects the most site-specific example:
  → "find laptops under $1000 at TechStore"  (too generic without site name)
  → "TechStore laptop recommendations"        (site-specific = traceable)
                         ↓
WebFetch → Perplexity search results page
WebFetch → Google search (check for AI Overview)
                         ↓
Parse response:
  - Is "techstore.com" in sources? → site_cited = true/false
  - Does the answer mention "3,200 products"? → fact accuracy check
  - Does the answer match the price range from digest.md?
```

**Probe selection strategy:**

| Probe type | Example | When to use |
|-----------|---------|-------------|
| Brand + category | "UrbanStyle winter coats" | Always — traceable to your site |
| Unique fact | "which fashion store ships to 45 countries" | When you have a distinctive differentiator |
| Price probe | "fashion store under $100 dresses New York" | Ecommerce |
| Feature probe | "project management tool with free tier and API" | SaaS |
| Location probe | "boulangerie artisanale Lyon centre" | Local business |

**What gets recorded per probe:**

```json
{
  "query": "UrbanStyle winter coats 2026",
  "engine": "perplexity",
  "site_cited": true,
  "citation_url": "https://urbanstyle.fashion/outerwear",
  "answer_excerpt": "UrbanStyle offers 290 outerwear products from $89 to $650...",
  "accuracy_checks": [
    { "fact": "290 outerwear products", "found": true,  "accurate": true  },
    { "fact": "ships to 45 countries",  "found": false, "accurate": false }
  ]
}
```

---

### Step 5 — Semantic accuracy analysis

The agent compares every specific fact in `digest.md` against what AI engines actually said in the probe answers.

```
ARA declares              AI answer says              Classification
──────────────────────────────────────────────────────────────────────
"3,200+ products"    →   "over 3,000 products"   →  APPROXIMATE (+1 pt)
"ships to 45 cntrs"  →   "international shipping" →  VAGUE      ( 0 pt)
"founded 2019"       →   "established 2021"       →  INACCURATE (-1 pt) ⚠
"price from $45"     →   not mentioned            →  MISSING    ( 0 pt)
"rating 4.6/5"       →   "highly rated brand"     →  VAGUE      ( 0 pt)
```

**Scoring:**
- `exact` +2 pts — AI answer matches ARA data precisely
- `approximate` +1 pt — close but not exact ("over 3,000" vs "3,200+")
- `vague` 0 pts — mentioned in general terms without specific data
- `missing` 0 pts — not covered in AI answer at all
- `inaccurate` -1 pt — AI contradicts ARA data ← **action required**

**What each result means:**

**INACCURATE** is the most important signal. It means one of two things:
1. Your `digest.md` is outdated — the AI scraped more recent data than your ARA files contain. → Run `/ara geo` to refresh.
2. The AI engine has wrong data in its index and hasn't picked up your ARA content yet. → Check that enforcement is working, wait 14 days, re-run.

**VAGUE** means the AI engine is summarizing rather than citing specific ARA data. Usually fixed by making `digest.md` denser: replace "affordable prices" with "$45–$380".

**MISSING** means the AI engine answered the query without mentioning that fact. Either the probe query doesn't surface that fact, or the fact isn't prominent enough in `digest.md`. Move it to the first paragraph.

---

### Step 6 — Trend analysis

Every run saves a structured JSON report to `.ara-monitoring/report-YYYY-MM-DD.json`. On each subsequent run, the agent loads the most recent previous report and computes deltas:

```
Report 1 (baseline, 2026-04-16):
  citation_rate: 14%  |  accuracy: 62/100  |  digest_reads: 203

Report 2 (14 days later, 2026-04-30):
  citation_rate: 29%  |  accuracy: 74/100  |  digest_reads: 441

Delta:
  citation_rate: ▲ +107%   ← ARA enforcement is working
  accuracy:      ▲ +19%    ← digest.md update improved accuracy
  digest_reads:  ▲ +117%   ← more bots reading ARA content
```

The trend is what matters. A single report is a snapshot. The trend shows whether ARA is creating a sustainable GEO advantage.

**The expected trajectory after ARA + enforcement deployment:**

```
Week 1–2:   Bots discover the redirect → digest_reads increases
Week 3–4:   ARA content appears in AI index → citation_rate starts increasing
Week 5–8:   Accuracy improves as AI engines use ARA data → accuracy_score increases
Week 9+:    manifest_reads increases → bots start checking ARA natively
```

---

### Step 7 — Report and recommendations

The agent generates two outputs:

**1. Human-readable terminal report:**
```
━━━ ARA GEO Impact Report — UrbanStyle ━━━━━━━━━━━━
2026-04-30 | Period: 30 days | ▲ +12% vs 2026-04-16

  ARA Health:        87/100   ▲ +4
  AI Citation Rate:  43%      ▲ +15%   (3/7 queries)
  Semantic Accuracy: 71/100   ▲ +9

── Server Signals ──────────────────────────────────
  Bot visits: 847  |  digest.md reads: 621 (73%)
  GPTBot: 312  ClaudeBot: 189  PerplexityBot: 156
  Manifest direct reads: 12 ← bots that know ARA

── Citation Probes ──────────────────────────────────
  Perplexity: 3/5 queries cite site  ✓
  Google AIO: 1/5 queries            △
  Claude:     2/5 queries            △

── Semantic Accuracy ────────────────────────────────
  ✓ "3,200+ products"    → AI says "over 3,000"   APPROXIMATE
  ✓ "ships to 45 cntrs"  → AI says "45 countries" EXACT
  ✗ "founded 2019"       → AI says "est. 2021"    INACCURATE ← fix

── Top 3 Recommendations ────────────────────────────
  1. Update digest.md: "Founded 2019" (AI says 2021 — stale data)
  2. Google AIO underperforming: add FAQ schema to homepage
  3. 4 bots never detected: check robots.txt for Facebook/Apple
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**2. Structured JSON report** saved to `.ara-monitoring/report-YYYY-MM-DD.json` for historical tracking and programmatic analysis.

---

### The improvement loop

`ara-monitor` is designed to feed back into the other agents:

```
/ara monitor → "digest.md fact 'founded 2019' is INACCURATE — AI says 2021"
                    ↓
/ara geo     → refreshes digest.md with current data
                    ↓
# wait 14 days
/ara monitor → accuracy_score improves
```

```
/ara monitor → "citation_rate 14% — digest.md too vague, no specific prices"
                    ↓
/ara geo     → adds "$45–$380 price range, 290 outerwear products"
                    ↓
# wait 14 days
/ara monitor → Perplexity starts citing specific prices → accuracy ▲
```

This creates a **closed feedback loop** between what ARA declares, what AI engines index, and what users see in AI search results.

---

## Architecture

Three specialized agents, one skill:

```
/ara <command>
    │
    ├─ ara-auditor       Read-only. Scores compliance (A-F). Detects llms.txt.
    │                    Never writes files.
    │
    ├─ ara-transformer   Generates ARA files. 7-phase pipeline:
    │                    1. Intelligence gathering (fetch + crawl)
    │                    2. llms.txt migration (if present)
    │                    3. manifest.json (Layer 1)
    │                    4. schemas/ (Layer 2 + Schema.org)
    │                    5. actions.json (Layer 3 + intent mapping)
    │                    6. digest.md (GEO optimization)
    │                    7. Validation (npx ara-validate)
    │
    └─ ara-enforcer      Forces AI bots to use ARA. 7-phase pipeline:
                         1. Verify ARA files exist
                         2. Detect tech stack
                         3. Generate middleware (Next.js/WP/nginx/Apache/CF/Django/Laravel)
                         4. Update robots.txt
                         5. Inject HTML <head> hints
                         6. Inject JSON-LD reference
                         7. Verify signals with curl simulations
```

The recommended full workflow:
```
/ara transform <url>  →  generates ARA files
/ara enforce <url>    →  forces AI bots to use them
/ara audit <url>      →  verifies everything is working
# — wait 7-14 days —
/ara monitor <url>    →  measures GEO impact (citations, accuracy, bot traffic)
```

---

## Examples

The `examples/` directory contains real ARA output for 4 site types:

- `examples/ecommerce/` — fashion boutique, REST + MCP
- `examples/saas/` — project management platform, MCP + A2A
- `examples/media/` — tech news site, 2,400 articles
- `examples/restaurant/` — local business, reservations

---

## ARA Adoption Levels

| Level | Files | Agent token cost | Capability |
|-------|-------|-----------------|------------|
| 0 | Nothing | ~2,000 (HTML parse) | Agent must scrape |
| 1 | manifest.json | ~150 | Identity + content map |
| 2 | + schemas/ | ~250 | Data structure + Schema.org |
| 3 | + actions.json | ~350 | Execute queries + mutations |
| 4 | + MCP/A2A | ~350 | Native LLM tool calls |

Start at Level 1 (one file, 30 minutes). The agents take you to Level 3 in a single command.

---

## ARA Specification

The full ARA v1.0 specification:
- [Layer 1 — manifest.json](https://ara-standard.org/spec/v1.0/manifest)
- [Layer 2 — schemas/](https://ara-standard.org/spec/v1.0/schemas)
- [Layer 3 — actions.json](https://ara-standard.org/spec/v1.0/actions)

Validate your implementation:
```bash
npx ara-validate https://yoursite.com
```

---

## Contributing

These agents are open-source under CC BY 4.0.

- Found a bug? [Open an issue](https://github.com/ARA-Standard/claude-ara-agents/issues)
- Want to add support for a new site type? [Submit a PR](https://github.com/ARA-Standard/claude-ara-agents/pulls)
- Questions? [Discussions](https://github.com/ARA-Standard/claude-ara-agents/discussions)

---

## License

CC BY 4.0 — Free to use, modify, and redistribute with attribution.

Built on the [ARA Standard](https://ara-standard.org) — the open web standard for the agentic web.
