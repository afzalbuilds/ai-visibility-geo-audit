# AI Visibility Tracker & GEO Audit inside Claude

See how your brand shows up in AI answers without leaving Claude. Give it a brand and a category, and the plugin tells you whether **ChatGPT, Gemini, Perplexity and Claude** mention you, where you rank against competitors, which sources they cite instead of you, which Google searches show an AI Overview, and what to fix on your pages so the engines can cite them. This is generative engine optimization (GEO, also called AEO) with real engine answers, not estimates.

It works in Claude chat, in Cowork and in Claude Code, and the same repository also installs in Grok Build, Grok Bot, Cursor and Hermes Agent (see [Other AI agents](#other-ai-agents-grok-and-hermes)). You need no accounts with the AI engines; the model usage is included in each check's price. Checks run as pay-per-event Apify Actors on your own Apify account, and you only pay for checks that complete.

## What you can ask

- "Does ChatGPT mention Notion when people ask about note-taking apps?"
- "Run an AI visibility report for HubSpot in small-business CRM against Salesforce, Pipedrive and Zoho."
- "Which sites do Perplexity and Gemini cite for 'best project management software'? Are we one of them?"
- "Audit acme.com for AI search. Is ClaudeBot blocked? Do we have llms.txt?"
- "Why doesn't ChatGPT cite our pricing page? What should we change?"
- "Which of these 20 searches trigger a Google AI Overview, and who does it cite?"
- "Where are people on Reddit asking for a tool like ours this month?"
- "What would a weekly AI visibility report cost for 10 prompts on four engines?"

Claude builds a realistic prompt set in your buyers' words, shows it with the cost, runs the checks, waits for the results and gives you tables with every prompt and engine accounted for. Checks that fail are listed as failed, never guessed.

## What is covered

| Question | Actor used | Per run | What you get |
|---|---|---|---|
| Do the engines name my brand in my category? | AI Brand Reach Checker | 4 engines | mentioned, cited, the sentence that mentioned you, competitors named, 0 to 100 score |
| How do I rank per prompt and engine versus competitors? | AI Visibility Rank Tracker | up to 200 prompts × 4 engines | mentioned, position, share of voice, citations, run-over-run gained, lost, improved, declined |
| Which sources do the engines cite, and is my site among them? | AI Citation Monitor | up to 200 prompts × 4 engines | cited URLs and domains per answer, your citation rate, citation leaderboard, citation gap |
| Is this page written so AI can answer from it? | GEO Content Auditor | up to 50 pages | 0 to 100 answerability score, five sub-scores, issues, quick wins, missing schema, suggested FAQ |
| Can AI crawlers reach and read my site? | AI Readiness Auditor | up to 100 pages | robots rules for GPTBot, ChatGPT-User, Google-Extended, ClaudeBot and PerplexityBot, llms.txt, sitemap, JSON-LD, metadata, coded findings |
| Which Google searches show an AI Overview, citing whom? | Google AI Overview Extractor | 25 queries | overview text, bullets, cited sources with domains |
| Where do buyers ask for recommendations? | Reddit Buyer Intent Monitor | up to 500 posts | posts with a 0 to 100 buyer-intent score, matched phrases and brands |

Engines checked: Perplexity, ChatGPT, Claude and Gemini, each through a fixed model with live web search. Not covered: Google AI Mode, Bing Copilot, Grok, paid-tier models, personalized answers, and historical data before your first run.

## How to set it up

1. Install the plugin from the Claude directory (claude.ai > Customize > Plugins) or in Claude Code with `claude plugin install ai-visibility-geo-audit@imzlab-ai-visibility` after adding the marketplace `afzalbuilds/ai-visibility-geo-audit`.
2. Open the plugin's **Connectors** tab and connect the **ai-visibility** server. It is Apify's official MCP server (`mcp.apify.com`) limited to the seven Actors above, so a browser window opens for you to sign in or create a free Apify account. Free accounts include monthly usage credit, so you can try a brand check without adding a card. Claude may note that the server is not a separately registered directory connector; that is expected, because it is Apify's hosted server rather than one Imzlab runs.
3. Ask Claude about your brand.

In Claude Code, `/mcp` shows the connection state, and `/ai-visibility-geo-audit:ai-visibility <brand> <category> <competitors>`, `/ai-visibility-geo-audit:geo-audit <url>` and `/ai-visibility-geo-audit:aio-check <search terms>` run the checks directly.

## Other AI agents: Grok and Hermes

The skills and the ai-visibility server are the same on every platform; only the manifest each app reads differs. All of them sign you in to Apify in the browser, and checks are billed to your own Apify account exactly as in Claude.

| App | Reads | What loads |
|---|---|---|
| Claude (chat, Cowork, Claude Code) | `.claude-plugin/plugin.json`, `.mcp.json` | skills, commands, geo-report-writer agent, ai-visibility server |
| Grok Build | `.grok-plugin/plugin.json`, `.mcp.json` | skills, commands, geo-report-writer agent, ai-visibility server |
| Cursor, and Grok Bot through the same plugin format | `.cursor-plugin/plugin.json`, `mcp.json` | skills, commands, geo-report-writer agent, ai-visibility server |
| Hermes Agent | `plugin.json`, `mcp.json` (Agent Plugins v1) | skills and ai-visibility server |

### Grok Build

```bash
grok plugin install afzalbuilds/ai-visibility-geo-audit
```

Grok asks you to confirm that you trust the plugin; its ai-visibility server stays off until you do. Then open `/mcps`, select **ai-visibility** and press `i` to sign in to Apify. Ask Grok about your brand.

### Grok Bot and Cursor

The plugin is not listed in their marketplaces yet. Once it is, add it from the marketplace and sign in to Apify when the ai-visibility server asks.

On grok.com you can use the Actors today without the skills: open Connectors, add a custom connector and paste the server URL below.

### Hermes Agent

Needs Hermes 0.21.6 or newer, which loads Agent Plugins packages (a root `plugin.json`).

```bash
hermes plugins install afzalbuilds/ai-visibility-geo-audit --no-enable
```

```bash
hermes plugins enable ai-visibility-geo-audit
```

The package declares the ai-visibility server, but Hermes signs in to a remote server only when your own configuration asks for it. Register the server under the same name with browser sign-in, so Hermes uses your Apify account for it. Run this in a terminal, not from a script, because Hermes starts the sign-in only when it can talk to you:

```bash
hermes mcp add ai-visibility --auth oauth --url "https://mcp.apify.com/?tools=muhammadafzal/ai-visibility-rank-tracker,muhammadafzal/ai-citation-monitor,muhammadafzal/ai-brand-reach-checker,muhammadafzal/geo-content-auditor,muhammadafzal/ai-readiness-auditor,muhammadafzal/google-ai-overviews-scraper,muhammadafzal/reddit-buyer-intent-brand-monitor"
```

Hermes opens Apify's consent page in your browser. After you allow access it lists the server's tools (the seven Actors plus the run, dataset and record tools); press Enter to enable them all. `hermes mcp test ai-visibility` confirms the connection, and `hermes mcp login ai-visibility` signs in again later if the access is ever revoked.

Every check is a paid Actor run on your Apify account, so the package marks the server `untrusted`: Hermes asks before each call and refuses them in unattended runs. Your own server entry replaces the package's entry, including that setting; add `trust: untrusted` under `mcp_servers.ai-visibility` in `~/.hermes/config.yaml` to keep the prompts.

To install only one skill instead of the whole package: `hermes skills install afzalbuilds/ai-visibility-geo-audit/skills/ai-visibility-check`. The ai-visibility server still has to be added as above.

### Server URL

Any MCP client can use the Actors directly by adding this URL as a remote (streamable HTTP) server with OAuth:

```
https://mcp.apify.com/?tools=muhammadafzal/ai-visibility-rank-tracker,muhammadafzal/ai-citation-monitor,muhammadafzal/ai-brand-reach-checker,muhammadafzal/geo-content-auditor,muhammadafzal/ai-readiness-auditor,muhammadafzal/google-ai-overviews-scraper,muhammadafzal/reddit-buyer-intent-brand-monitor
```

## Pricing

The plugin itself is free. Each check is billed by Apify on your account at the Actor's pay-per-event price. Prices below are the Apify Store prices on the FREE tier as of 2026-11-13; paid Apify plans get a discount of up to 20 percent, and failed checks are never charged.

| Actor | Per result | Per run start | Example |
|---|---|---|---|
| AI Brand Reach Checker | USD 0.050 per engine | USD 0.005 | 4 engines: USD 0.21 |
| AI Visibility Rank Tracker | USD 0.060 per prompt and engine (plus USD 0.050 each for optional citation extraction and sentiment) | USD 0.010 | 10 prompts × 4 engines: USD 2.41 |
| AI Citation Monitor | USD 0.050 per prompt and engine | USD 0.010 | 10 prompts × 4 engines: USD 2.01 |
| GEO Content Auditor | USD 0.050 per page | USD 0.006 | 10 pages: USD 0.51 |
| AI Readiness Auditor | USD 0.050 per page | USD 0.006 | 10 pages: USD 0.51 |
| Google AI Overview Extractor | USD 0.005 per query | USD 0.0125 | 25 queries: USD 0.14 |
| Reddit Buyer Intent Monitor | USD 0.005 per post | USD 0.006 | 50 posts: USD 0.26 |

A standard report (rank tracker plus citation monitor, 10 prompts on four engines) is about USD 4.42. Before anything over USD 0.50, Claude shows the estimate and asks. Ask "what will this cost" to get an estimate without running anything. Free Apify accounts are capped at 20 result rows per run on the rank tracker, citation monitor and content auditor.

## Data and privacy

- The brand names, domains, competitor names, prompts, URLs and search terms you give Claude are sent to Apify's hosted MCP server (`mcp.apify.com`), which runs the Actor on your Apify account. The Actors send the prompts to the AI engines through the Actor owner's model access, fetch your public pages, load Google results pages, or search public Reddit.
- Results (AI answers, cited URLs, audit scores, public Reddit posts) are stored in your Apify account's datasets for your plan's retention period, where you can download them as CSV, Excel or JSON. The plugin stores nothing itself and sends nothing to any other service.
- Reddit results contain public posts with the author's username. Claude shows usernames only when you ask.
- You authorize Apify through OAuth in the plugin's Connectors tab. The plugin itself holds no Apify account details.
- Full policy: https://imzlab.com/privacy#ai-visibility. Terms: https://imzlab.com/terms#ai-visibility. Support: https://imzlab.com/support#ai-visibility

## Frequently asked questions

**Do I need ChatGPT, Gemini, Perplexity or Claude accounts?** No. Each Actor queries the engines through fixed models that the Actor owner pays for; the cost is inside the per-check price.

**Are the answers the same as what a user sees in the ChatGPT app?** Close, but not identical. The Actors use the engines' developer models with live web search, without personal history or location. Treat results as a representative sample and compare trends across runs, not single answers.

**How many prompts should I track?** Ten prompts on four engines is the standard set and costs about USD 2.41 per run. Keep the same prompts, competitors and tracking id every week so the gained and lost counts mean something.

**Why does my brand show as mentioned when the answer was about something else?** Mention matching is plain text matching. For a brand whose name is a common word, Claude reads the mention context and can rerun with the full product name.

**Can I schedule a weekly report?** In Claude Code or Cowork, ask the geo-report-writer agent to run your saved prompt set and write a dated report with changes versus last time. For unattended schedules, create an Apify schedule for the Actor in Apify Console with the same input.

**What if Google shows a verification page?** The AI Overview extractor reports that query as blocked, not as "no AI Overview", and Claude offers to retry later. Blocked queries are still charged at USD 0.005.

**Is this an official tool of OpenAI, Google, Perplexity or Reddit?** No. This plugin and the Actors are independent and not affiliated with them. Engine answers and Google pages are quoted as returned.

## Skills and commands in this plugin

- **ai-visibility-check**: brand mentions, rank, share of voice and citations across the four engines, with a prompt-set design guide.
- **geo-site-audit**: content answerability and AI crawler readiness audits merged into one fix list.
- **ai-overview-check**: Google AI Overview triggers and sources, plus Reddit buyer-intent listening.
- **ai-visibility-cost-estimate**: estimate the Apify cost before running.
- **/ai-visibility-geo-audit:ai-visibility**, **/ai-visibility-geo-audit:geo-audit** and **/ai-visibility-geo-audit:aio-check** commands (Cowork, Claude Code, Grok Build).
- **geo-report-writer** agent (Cowork, Claude Code, Grok Build): runs a saved prompt set and writes a dated report with changes.

Hermes Agent loads the skills and the ai-visibility server; it has no equivalent for the commands and the geo-report-writer agent, so ask for a dated report in plain words there.

## Source and support

Source: https://github.com/afzalbuilds/ai-visibility-geo-audit. Issues and feature requests: https://github.com/afzalbuilds/ai-visibility-geo-audit/issues. The Actors are published on Apify Store by Imzlab Technologies under the `muhammadafzal` account.

Last verified: 2026-11-13. Checked with the plugin validators of Claude Code, Grok Build and Hermes Agent 0.23.1.
