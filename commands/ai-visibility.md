---
description: Check whether ChatGPT, Gemini, Perplexity and Claude mention, rank and cite a brand, versus competitors
argument-hint: <brand> [category] [competitor1, competitor2...]
---

Run an AI visibility check using the `ai-visibility-check` skill from this plugin for: $ARGUMENTS

Treat the first word or quoted phrase as the brand, the next phrase as the buyer-language category, and anything after it as competitors. If only a brand was given, ask for the category and competitors or infer obvious ones and show them. Build the prompt set as the skill's `references/prompt-set-design.md` describes, show it with the estimated Apify cost, wait for confirmation when the estimate is above USD 0.50, run the rank tracker (and the citation monitor if citations were asked for) through the ai-visibility MCP server, wait for every run to finish, read the dataset and the SUMMARY record, reconcile every prompt and engine, and present the headline, per-engine table, per-prompt table, citation leaderboard if available and the Apify cost line. If no brand was given, ask for one.
