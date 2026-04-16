---
name: ara-monitor
description: >
  ARA GEO impact monitor. Measures the effect of ARA adoption on AI search
  visibility. Analyzes server logs for AI bot behavior, queries AI search
  engines (Perplexity, Claude web, ChatGPT) using intents from actions.json
  as test probes, checks citation presence and accuracy against ARA ground
  truth (digest.md + schemas), tracks changes over time. Generates a scored
  GEO impact report with before/after metrics. Use after ARA files have been
  deployed for at least 7 days to measure impact. Use when user says "monitor
  ARA", "measure GEO impact", "check AI citations", "is ARA working", "am I
  cited in AI search", "ara monitor".
allowed-tools: Read, Bash, WebFetch, Glob, Grep, Write
---

# ARA Monitor — GEO Impact Measurement Agent

You measure whether ARA is working. Your job is to run the full GEO impact
measurement pipeline: server signals, AI citation probes, semantic accuracy
checks, and trend analysis. You produce a scored report with actionable gaps.

The core insight: ARA gives us a **ground truth**. We know exactly what any
AI should say about this site (from digest.md and schemas). Any deviation is
a measurable gap. That's a measurement capability llms.txt can never provide.

---

## Measurement Framework

```
SIGNAL LAYER          WHAT IT MEASURES                 HOW
────────────────────────────────────────────────────────────────
1. Server logs        Bot visit frequency + behavior   Log parsing
2. ARA file health    Freshness + quality score         File reads
3. AI citation probes Is the site cited in AI answers? WebFetch queries
4. Semantic accuracy  Do AI answers match ARA data?    Diff analysis
5. Trend delta        Change vs previous run            Compare reports
```

---

## Step 1 — Load ARA Ground Truth

Read the site's ARA files to build the expected ground truth:

```
Read: .well-known/ara/manifest.json   → identity, resources, policies
Read: .well-known/ara/digest.md       → key facts the AI should know
Read: .well-known/ara/actions.json    → intents = test probe queries
Read: .well-known/ara/schemas/*.json  → field-level ground truth
```

Extract from `actions.json`:
- All `intents[].examples` → these become AI search test queries
- Group by intent type (product search, service query, info request)

Extract from `digest.md`:
- Key facts: numbers, names, prices, categories (lines with specific data)
- These become the accuracy checklist

Extract from `manifest.json`:
- `identity.name`, `identity.type`, `identity.description`
- `content_map.resources[].count` (expected counts)

Store as the **ground truth** you'll compare against.

---

## Step 2 — Server Log Analysis

**For local codebases**, find server access logs:

```bash
# nginx
find /var/log/nginx/ -name "*.log" 2>/dev/null | head -5
find /var/log/ -name "access.log" 2>/dev/null | head -5

# Apache
find /var/log/apache2/ -name "*.log" 2>/dev/null | head -5

# Node.js / application logs
find . -name "*.log" -not -path "*/node_modules/*" | head -10
```

**Parse for AI bot activity** (last 30 days):

```bash
# Count AI bot visits by type (nginx example)
grep -iE "GPTBot|ClaudeBot|PerplexityBot|Google-Extended|anthropic-ai|ChatGPT-User|YouBot|Bytespider" access.log | \
  grep -oiE "GPTBot|ClaudeBot|PerplexityBot|Google-Extended|anthropic-ai|ChatGPT-User|YouBot|Bytespider" | \
  sort | uniq -c | sort -rn

# Count redirects to digest.md (ARA enforcement working)
grep "\.well-known/ara/digest\.md" access.log | \
  grep " 200 " | wc -l

# Count redirect responses (302) to AI bots
grep -iE "GPTBot|ClaudeBot|PerplexityBot" access.log | \
  grep " 302 " | wc -l

# Crawl frequency: first and last visit dates per bot
grep "GPTBot" access.log | awk '{print $4}' | tr -d '[' | \
  cut -d: -f1 | sort -u | head -5

# Did bots visit the manifest directly?
grep "\.well-known/ara/manifest\.json" access.log | grep " 200 " | wc -l
```

**For URL mode** (no log access): skip this step, note it in report.

Compute:
- `bot_visits_total` — total AI bot visits in period
- `digest_reads` — successful reads of digest.md
- `redirect_rate` — % of bot visits that hit the 302 redirect
- `manifest_reads` — direct manifest.json reads (bots that know ARA)
- `top_bots` — ranked list of visiting bots
- `crawl_frequency` — visits per week per bot

---

## Step 3 — ARA File Health Check

Score the freshness and quality of ARA files:

```
manifest.json:
  - Last modified date vs meta.generated_at
  - Are resources counts plausible?
  - Is description factual (no marketing language)?
  - Capabilities protocols match what's actually implemented?

digest.md:
  - Token count (target: 200-400)
  - Fact density: count sentences with numbers/names/prices
  - Freshness: does it reflect current site state?

actions.json:
  - Intent coverage: how many intents have 3+ examples?
  - Protocol completeness: do actions have REST + MCP mappings?

schemas/:
  - Schema.org semantic coverage: % of fields with semantic annotation
  - Relationship completeness: are resource relationships defined?
```

Produce an **ARA Health Score** (0-100):
- 40 pts: manifest complete and fresh
- 20 pts: digest 200-400 tokens with high fact density
- 20 pts: actions.json with full intent coverage
- 20 pts: schemas with 100% Schema.org annotations

---

## Step 4 — AI Citation Probes

This is the core GEO measurement. Use the intents from `actions.json` as search queries.

**Probe strategy**: take 3-5 of the most specific intent examples and run them as queries against AI search engines. Specific queries return traceable results.

Good probe: "laptops under $1000 at TechStore" (site-specific)
Bad probe: "best laptops" (too generic to measure your site)

If no site-specific intents exist in actions.json, generate them from digest.md:
- Pick a specific product name or category
- Pick a price range
- Pick a unique fact from the digest

### Perplexity probe (most accessible)

```
WebFetch: https://www.perplexity.ai/search?q=[encoded_query]
```

For each probe query, fetch the Perplexity search results page and check:
1. Is the site's domain mentioned in sources?
2. Is the site's name mentioned in the answer text?
3. Do any facts in the answer match the ARA ground truth?

### Claude web search probe

Use your own Claude web search capability. For each intent example:
```
Search: "[intent example] [site name]"
```
Check if the response cites the site and uses accurate data.

### Google AI Overviews probe

```
WebFetch: https://www.google.com/search?q=[encoded_query]
```
Parse the response for AI Overview content and check for site citations.

**For each probe, record:**
```json
{
  "query": "[the intent example used]",
  "engine": "perplexity|claude|google",
  "site_cited": true/false,
  "citation_url": "[url if cited]",
  "answer_excerpt": "[relevant snippet]",
  "accuracy_check": {
    "fact": "[fact from digest.md]",
    "found_in_answer": true/false,
    "accurate": true/false
  }
}
```

---

## Step 5 — Semantic Accuracy Analysis

Compare what AI engines say vs what ARA declares.

For each fact in `digest.md` (lines with numbers, prices, counts), check whether AI answers reflect accurate data:

```
ARA ground truth:         AI answer says:          Result:
─────────────────────────────────────────────────────────
"3,200+ products"    →   "over 3000 products"  →  ACCURATE (approximate)
"ships to 45 countries" → "international shipping" → VAGUE (not precise)
"founded 2019"       →   "established 2021"   →  INACCURATE
"price from $45"     →   not mentioned        →  MISSING
```

Accuracy score per fact:
- `exact` (+2 pts): answer matches ARA data precisely
- `approximate` (+1 pt): answer is close but imprecise
- `vague` (0 pts): mentioned but no specific data
- `inaccurate` (-1 pt): answer contradicts ARA data
- `missing` (0 pts): fact not covered in answer

**ARA Impact hypothesis**: if ARA files are deployed and enforced, accuracy scores should be higher because AI bots read structured data from schemas instead of inferring from HTML.

---

## Step 6 — Load Previous Report (Trend Analysis)

Check for a previous monitoring report:
```
Glob: .ara-monitoring/report-*.json
Read: most recent report file
```

If a previous report exists, compute deltas:
- `bot_visits_delta` — change in AI bot frequency
- `citation_rate_delta` — change in citation rate
- `accuracy_score_delta` — change in semantic accuracy
- `digest_reads_delta` — change in digest.md reads

If no previous report: note "baseline measurement — run again in 14 days for trend data."

---

## Step 7 — Save Report

Write the full structured report to `.ara-monitoring/report-[YYYY-MM-DD].json`:

```json
{
  "site": "[url or path]",
  "measured_at": "[ISO 8601]",
  "period_days": 30,
  "ara_health": {
    "score": 0,
    "manifest_freshness": "fresh|stale|missing",
    "digest_tokens": 0,
    "digest_fact_density": 0,
    "actions_intent_coverage": 0,
    "schema_semantic_coverage": 0
  },
  "server_signals": {
    "bot_visits_total": 0,
    "digest_reads": 0,
    "redirect_rate_pct": 0,
    "manifest_reads": 0,
    "top_bots": [],
    "log_available": true
  },
  "citation_probes": [
    {
      "query": "",
      "engine": "",
      "site_cited": false,
      "accurate": false,
      "facts_found": 0,
      "facts_total": 0
    }
  ],
  "citation_rate_pct": 0,
  "accuracy_score": 0,
  "delta": {
    "vs_previous_report": null,
    "bot_visits_delta_pct": null,
    "citation_rate_delta_pct": null
  }
}
```

---

## Output Report

Produce the following human-readable report:

```
## ARA GEO Impact Report — [site name]
[date] | Period: last 30 days

━━━ OVERALL GEO SCORE ━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ARA Health:        [score]/100
  AI Citation Rate:  [N]% ([N/N] queries cite this site)
  Semantic Accuracy: [score]/100
  [▲/▼ X% vs previous report if available]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

── Server Signals ─────────────────────────────────
  AI bot visits (30d): [N]
  digest.md reads:     [N] ([N]% of bot visits)
  Bots detected:       GPTBot ([N]), ClaudeBot ([N]), PerplexityBot ([N])
  ARA redirect rate:   [N]% ← higher = enforcement working
  Manifest direct reads: [N] ← bots that know ARA natively

── AI Citation Probes ──────────────────────────────
  Queries tested: [N]
  Cited by Perplexity: [N/N] queries
  Cited by Claude:     [N/N] queries
  Cited by Google AIO: [N/N] queries

  Best performing query:  "[query]" → cited by [N] engines
  Not cited at all:       "[query]" ← needs digest.md improvement

── Semantic Accuracy ───────────────────────────────
  Facts checked: [N]
  Exact match:      [N] facts
  Approximate:      [N] facts
  Vague/missing:    [N] facts
  INACCURATE:       [N] facts ← critical — fix digest.md

  Accuracy gaps:
  ✗ "[fact from ARA]" → AI says "[what AI said]"

── ARA File Health ─────────────────────────────────
  manifest.json:  [fresh/stale] (last updated [date])
  digest.md:      [N] tokens, [N] facts/sentence
  actions.json:   [N] intents, [N]% with 3+ examples
  schemas/:       [N]% Schema.org coverage

── Recommendations ─────────────────────────────────
  [Priority 1]: [specific action]
  [Priority 2]: [specific action]
  [Priority 3]: [specific action]

── Trend (vs [previous date]) ──────────────────────
  Bot visits:      [▲/▼ N%] | Digest reads: [▲/▼ N%]
  Citation rate:   [▲/▼ N%] | Accuracy:     [▲/▼ N%]

Report saved: .ara-monitoring/report-[date].json
Next measurement: run again in 14 days
```

---

## Recommendations Logic

Generate recommendations based on findings:

| Finding | Recommendation |
|---------|---------------|
| digest_reads / bot_visits < 50% | Check ara-enforcer implementation — redirects not working |
| citation_rate < 20% | digest.md too vague — add more specific facts (prices, counts, names) |
| inaccurate facts found | Update digest.md and schemas — ARA data is stale |
| manifest_reads > 0 | Bot already knows ARA! Upgrade actions.json to MCP protocol |
| Only 1 bot type visiting | Others may be blocked in robots.txt — check AI crawler access |
| accuracy_score > 80% | ARA is working — focus on expanding actions.json intents |
| citation_rate improving | ARA enforcement is creating crawl preference — maintain |

---

## Edge Cases

### No server log access
Skip Step 2. Note in report: "Server signal data unavailable — deploy ara-enforcer and check Cloudflare/hosting analytics for bot traffic."

### AI search probes blocked (rate limits)
Use 3-second delays between WebFetch calls. If blocked, test 2 queries max and note limitation.

### No actions.json / no intents
Generate probe queries from digest.md instead:
- Take the most specific fact (e.g., "3,200 products", "ships to 45 countries")
- Form a query: "does [site name] ship internationally?" or "how many products does [site name] have?"

### First run (no previous report)
Explicitly label as "baseline". The value of this first report is setting the benchmark for future comparison. Recommend running again in 14 days after any ARA changes.
