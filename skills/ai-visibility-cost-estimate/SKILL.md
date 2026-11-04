---
name: ai-visibility-cost-estimate
description: Estimate how much an AI visibility check, GEO audit, AI Overview check or Reddit listening run will cost on Apify before running it. Use when the user asks "how much does the AI visibility check cost", "what will this GEO audit cost", "price per prompt", "price per page", "is it free", "how is this billed", "which option is cheapest", or asks about Apify credits, plans or charges for this plugin.
---

# AI visibility cost estimate

Prices are in `../ai-visibility-check/references/actor-contracts.md` (FREE-tier Apify Store prices, verified 2026-11-04). Use that file as the single source; do not quote other numbers.

## How billing works (explain in two or three sentences when asked)

The plugin is free. Each Actor is a pay-per-event Actor on Apify Store, billed to the user's own Apify account: one event per result row (a prompt checked on one engine, a page audited, a Google query, a Reddit post) plus a small start event per run. The AI model usage behind the checks is included in the event price, so the user needs no AI accounts. Checks that fail are not charged. Free Apify accounts include monthly usage credit and are capped at 20 result rows per run on the rank tracker, citation monitor and content auditor; paid plans get tiered discounts of up to 20 percent.

## Estimate

1. Work out the units: prompts × engines for the rank tracker and citation monitor; platforms (max 4) for the brand reach checker; pages for each auditor; queries for Google; posts for Reddit.
2. Cost per Actor = runs × start fee + units × per-unit price. Rank tracker add-ons: USD 0.050 per check for citation extraction and USD 0.050 per mentioned check for sentiment; the default in this plugin leaves both off.
3. Total = sum. Present a small table (Actor, units, per unit, subtotal) and the total as "about USD X". Mention that failed checks and unreachable pages are not charged.
4. Reference figures: quick look USD 0.21; standard visibility check (10 prompts, 4 engines) USD 2.41; full check with citation monitor USD 4.42; 10-page content plus readiness audit USD 1.01; 25 Google queries USD 0.14; 50 Reddit posts USD 0.26.
5. Cheaper options when asked: fewer engines (ChatGPT and Perplexity only halves the cost), 5 prompts for a first pass, the brand reach checker instead of the rank tracker for a yes/no answer, readiness auditor alone for technical questions.
6. Add the rough time: about 5 seconds per prompt-engine check, 10 seconds per page, 15 seconds per Google query, under 2 minutes for a Reddit run; runs for different Actors happen in parallel.

Do not run any Actor from this skill. If the user then says to go ahead, hand over to `ai-visibility-check`, `geo-site-audit` or `ai-overview-check`.
