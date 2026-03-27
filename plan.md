# Valuation Plugin — Docs Website Plan

## Goal

Build a docs website for the Valuation Plugin that serves two audiences:

1. **AI agents** — make the docs machine-readable so coding assistants (Cursor, Claude Code, Copilot) can discover, understand, and recommend the plugin
2. **Humans** — investors, finance professionals, and Damodaran enthusiasts who want to understand what the plugin does and how to use it

The site doubles as a **landing page** (sell the plugin) and a **documentation hub** (teach people and agents how to use it).

---

## Domain & Hosting Strategy

### Phase 1: Free (now)

| Component | Choice | URL |
|-----------|--------|-----|
| Landing page | Vercel | `valuation-plugin.vercel.app` |
| Docs site | Same Vercel project, `/docs` path | `valuation-plugin.vercel.app/docs` |

**Why Vercel over alternatives:**
- Free tier is generous (unlimited static requests, 100 GB bandwidth)
- Native Next.js/Docusaurus support with zero-config deploys
- Automatic preview deployments on every PR
- Free HTTPS on the `.vercel.app` subdomain
- When you buy a domain later, you just add it in the dashboard — no migration

> **Alternative:** Cloudflare Pages (`valuation-plugin.pages.dev`) is equally good and has no bandwidth limits. Pick whichever you prefer.

### Phase 2: Custom domain (when ready to pay ~$10/year)

| Component | Domain |
|-----------|--------|
| Landing page | `valuationplugin.com` (or `.dev`, `.ai`) |
| Docs | `docs.valuationplugin.com` |

The architecture below is designed so Phase 2 is a config change, not a rebuild.

**Domain name candidates to check availability for:**
- `valuationplugin.com` / `.dev` / `.ai`
- `damodaranplugin.com` (risky — trademark-adjacent)
- `intrinsicvalue.dev`
- `fcffvaluation.com`

---

## Framework: Docusaurus

### Why Docusaurus

| Criterion | Docusaurus | Mintlify | Nextra |
|-----------|-----------|----------|--------|
| Cost | Free (MIT) | $300/mo for Pro | Free |
| Agent-friendly (llms.txt) | Plugin available | Native | Manual |
| React components in docs | Yes (MDX) | Limited | Yes |
| Versioning | Built-in | Yes | Manual |
| Search | Algolia (free for OSS) | Built-in | Manual |
| Maintenance risk | Meta-backed, 62k stars | Commercial SaaS | Single maintainer, uncertain |
| Control | Full | Limited | Full |

**Decision:** Docusaurus. It's free, battle-tested, has a mature llms.txt plugin ecosystem, and gives full control. We can always migrate to Mintlify later if the $300/mo becomes worth it.

### Key Docusaurus plugins

1. **`docusaurus-plugin-llms`** — auto-generates `/llms.txt` and `/llms-full.txt` from all docs pages at build time ([GitHub](https://github.com/rachfop/docusaurus-plugin-llms))
2. **`@docusaurus/plugin-sitemap`** — built-in, helps with SEO/GEO
3. **`docusaurus-theme-search-algolia`** — free search for open-source docs

---

## Agent-Friendly Architecture

This is the core differentiator. Based on research into how Resend, Stripe, and other agent-optimized docs work:

### 1. `/llms.txt` (required)

A curated index following the [llmstxt.org](https://llmstxt.org/) spec:

```markdown
# Valuation Plugin

> A conversational FCFF intrinsic valuation tool built on Aswath Damodaran's
> fcffsimpleginzu spreadsheet. Talk to it; it does the math.

## Docs

- [Getting Started](https://docs.valuationplugin.com/docs/getting-started.md): Install the plugin and run your first valuation
- [How It Works](https://docs.valuationplugin.com/docs/how-it-works.md): The 7-phase FCFF pipeline explained
- [Skills Reference](https://docs.valuationplugin.com/docs/skills/overview.md): All 16 skills with inputs, outputs, and dependencies
- [Python Library](https://docs.valuationplugin.com/docs/lib/overview.md): The deterministic math engine behind the skills
- [Data Pipeline](https://docs.valuationplugin.com/docs/data-pipeline.md): How SEC EDGAR data flows from raw JSON to valuation inputs

## Optional

- [Damodaran Methodology](https://docs.valuationplugin.com/docs/methodology.md): Deep dive into the academic framework
- [FAQ](https://docs.valuationplugin.com/docs/faq.md): Common questions
```

### 2. `/llms-full.txt` (auto-generated)

The Docusaurus plugin concatenates all docs pages into a single markdown file. Agents that want to ingest everything in one shot (like Claude Code's context loading) use this.

### 3. Structured skill pages

Every skill gets a page with a **machine-parseable frontmatter block**:

```markdown
---
title: Cost of Capital
skill_id: cost-of-capital
phase: 4
inputs:
  - company ticker
  - risk-free rate (or auto-fetched)
  - equity risk premium
  - bond rating or default spread
outputs:
  - wacc_detailed (%)
  - wacc_industry_avg (%)
  - wacc_distribution (percentile)
depends_on:
  - company-identifier
  - last-twelve-months
feeds_into:
  - fcff-model
---
```

This lets agents programmatically understand the skill graph without reading prose.

### 4. Mermaid flow diagrams

The plugin already has `data-flow.mermaid` — we embed interactive Mermaid diagrams in the docs so both humans and agents can parse the pipeline visually/textually.

### 5. Code examples with copy-paste invocations

Every skill page includes a "Quick Start" block showing exactly how to invoke it:

```
User: "Value Apple using the detailed WACC method"
→ triggers: company-identifier → identify-required-statements → pull-raw-data
  → parse-raw-data-to-filings → last-twelve-months → last-10-k
  → cost-of-capital → growth-and-profitability → fcff-model
```

---

## Site Structure

```
valuation-plugin-docs/
├── docusaurus.config.js
├── package.json
├── src/
│   └── pages/
│       └── index.jsx          ← Landing page (not in /docs)
├── docs/
│   ├── getting-started.md
│   ├── how-it-works.md        ← 7-phase pipeline overview
│   ├── data-pipeline.md       ← SEC EDGAR → companyfacts → _raw.json → LTM
│   ├── methodology.md         ← Damodaran's FCFF framework explained
│   ├── faq.md
│   │
│   ├── skills/
│   │   ├── overview.md        ← Skill map + dependency graph
│   │   ├── company-identifier.md
│   │   ├── identify-required-statements.md
│   │   ├── pull-raw-data.md
│   │   ├── parse-raw-data-to-filings.md
│   │   ├── last-twelve-months.md
│   │   ├── last-10-k.md
│   │   ├── cost-of-capital.md
│   │   ├── r-and-d-converter.md
│   │   ├── lease-converter.md
│   │   ├── employee-options.md
│   │   ├── failure-rate.md
│   │   ├── growth-and-profitability.md
│   │   ├── fcff-model.md
│   │   ├── diagnostics.md
│   │   └── valuation-report.md
│   │
│   ├── lib/
│   │   ├── overview.md        ← Python library architecture
│   │   ├── dcf-engine.md
│   │   ├── cost-of-capital.md
│   │   ├── adjustments.md
│   │   ├── equity-bridge.md
│   │   └── data-loader.md
│   │
│   └── guides/
│       ├── expert-mode.md
│       ├── novice-mode.md
│       ├── feeling-lucky.md
│       └── interpreting-results.md
│
├── static/
│   ├── img/                   ← Screenshots, diagrams
│   └── llms.txt               ← (auto-generated at build, but can also be manual)
│
└── blog/                      ← Optional: tie into your Substack cross-posts
```

### Landing page (`/`) — what visitors see first

Sections:
1. **Hero** — "Value any company in a conversation" + CTA to docs
2. **How it works** — 3-step visual (Pick a company → AI fetches SEC data → Get intrinsic value)
3. **Built on Damodaran** — credibility section referencing the fcffsimpleginzu methodology
4. **For agents** — callout that the plugin is designed for Claude Code / Cowork, with link to `/llms.txt`
5. **Sample output** — embedded screenshot or interactive demo of a valuation run
6. **Substack CTA** — "Follow for weekly valuations" linking to your Substack

---

## Content Strategy: What to Write

### Priority 1 (launch)
- [ ] Landing page
- [ ] Getting Started (install + first valuation)
- [ ] How It Works (pipeline overview with Mermaid diagram)
- [ ] Skills Overview (the dependency graph + table of all 16 skills)
- [ ] 3-4 key skill pages (fcff, cost-of-capital, fcff-model, company-identifier)

### Priority 2 (week 2)
- [ ] Remaining 12 skill pages
- [ ] Python library docs
- [ ] Data pipeline deep dive
- [ ] Methodology page (Damodaran framework)

### Priority 3 (ongoing)
- [ ] Mode guides (Expert, Novice, Feeling Lucky)
- [ ] FAQ
- [ ] Blog cross-posts from Substack
- [ ] Sample valuation walkthroughs

---

## Agent Optimization Checklist

Drawing from [Resend's approach](https://resend.com/docs/introduction), [Mintlify's AI trends](https://www.mintlify.com/blog/ai-documentation-trends-whats-changing-in-2025), and the [llms.txt standard](https://llmstxt.org/):

- [ ] **`/llms.txt`** — curated index with descriptions (via docusaurus-plugin-llms)
- [ ] **`/llms-full.txt`** — full concatenated docs (auto-generated)
- [ ] **Structured frontmatter** on every skill page (skill_id, inputs, outputs, depends_on, feeds_into)
- [ ] **Consistent page structure** — every skill page follows the same template (Overview → Inputs → Outputs → Dependencies → Example → Edge Cases)
- [ ] **Token-efficient writing** — concise, no filler; agents pay per token in context window
- [ ] **Mermaid diagrams** — parseable by both humans and LLMs
- [ ] **No JavaScript-gated content** — all content is in the static HTML/markdown (agents can't run JS)
- [ ] **Semantic HTML** — proper heading hierarchy (H1 → H2 → H3, no skipping)
- [ ] **OpenAPI-style skill definitions** — the frontmatter acts like an API spec for each skill
- [ ] **Sitemap.xml** — for crawlers and traditional search engines
- [ ] **Meta tags** — og:title, og:description for link previews (Substack shares, social)

### Future: MCP Server for docs (stretch goal)

Expose the docs as an MCP server so agents can query skill definitions, look up inputs/outputs, and get methodology explanations programmatically — without needing to fetch and parse the website.

---

## Implementation Steps

### Step 1: Scaffold Docusaurus project
```bash
npx create-docusaurus@latest valuation-plugin-docs classic
```

### Step 2: Install agent plugins
```bash
npm install docusaurus-plugin-llms
```

### Step 3: Configure
- Set up `docusaurus.config.js` with site metadata, navbar, footer
- Add llms plugin config
- Create the landing page component in `src/pages/index.jsx`

### Step 4: Write Priority 1 content
- Pull from existing `docs/` folder in the plugin (there's already good content there)
- Standardize into the skill page template

### Step 5: Deploy to Vercel
- Connect GitHub repo → auto-deploy on push
- Verify `/llms.txt` and `/llms-full.txt` are generated correctly

### Step 6: Test with agents
- Feed `llms.txt` to Claude and ask it to explain the plugin
- Ask Claude Code to "use the valuation plugin to value Apple" and see if it can discover the right skill flow from the docs

---

## Cost Summary

| Item | Cost |
|------|------|
| Docusaurus | Free |
| Vercel hosting | Free |
| Algolia search | Free (open-source tier) |
| Domain (Phase 2) | ~$10-15/year |
| **Total at launch** | **$0** |

---

## Key References

- [llms.txt specification](https://llmstxt.org/)
- [Resend docs (agent-friendly example)](https://resend.com/docs/introduction)
- [docusaurus-plugin-llms](https://github.com/rachfop/docusaurus-plugin-llms)
- [Mintlify AI documentation trends](https://www.mintlify.com/blog/ai-documentation-trends-whats-changing-in-2025)
- [Fern: llms.txt guide for API docs](https://buildwithfern.com/post/optimizing-api-docs-ai-agents-llms-txt-guide)
