# AI Visibility Tracker & GEO Audit

This extension checks how ChatGPT, Gemini, Perplexity and Claude mention and cite a brand, finds Google AI Overviews and audits pages for AI search, with Imzlab Technologies' Actors on Apify, through the `ai-visibility` MCP server. The full instructions live in the extension's skills. Activate the matching skill before you call any of the server's tools:

- `ai-visibility-check`: brand mentions, rank, share of voice and citations.
- `geo-site-audit`: page answerability and AI crawler readiness, as one fix list.
- `ai-overview-check`: Google AI Overview triggers and sources, plus Reddit buyer intent.
- `ai-visibility-cost-estimate`: the Apify cost of a check before running it.

Every tool call on `ai-visibility` starts a paid Actor run on the user's own Apify account. Follow the skill's cost rules: estimate from the price table in its references, ask before large batches, and never quote a cost from a run's usage figures. Wait for each run to finish, account for every requested item, and never invent a result.

If the visibility tools are missing, the server is not signed in yet: ask the user to run `/mcp auth ai-visibility` and approve Apify's sign-in page in the browser.

Commands: `/ai-visibility`, `/aio-check`, `/geo-audit`. Subagent for unattended runs that leave a report on disk: `geo-report-writer`.
