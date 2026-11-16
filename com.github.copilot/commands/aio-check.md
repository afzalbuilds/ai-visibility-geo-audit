---
description: "Check which Google searches show an AI Overview and which sources it cites"
argument-hint: "<search terms, separated by commas or new lines> [--country us] [--lang en]"
---

Run a Google AI Overview check using the `ai-overview-check` skill from this plugin for: $ARGUMENTS

Treat the argument as a list of Google searches (up to 25 per run; split larger lists into several runs), with optional `--country` and `--lang` two-letter codes. Show the estimated Apify cost, run the Google AI Overview extractor through the ai-visibility MCP server, wait for the run to finish, read the dataset and the SUMMARY record, classify every query as AI Overview shown, no AI Overview, blocked or failed, and present the summary line, the per-query table, the source leaderboard and the Apify cost line. Do not run the Reddit monitor unless the user asks for Reddit. If no search terms were given, ask for them.
