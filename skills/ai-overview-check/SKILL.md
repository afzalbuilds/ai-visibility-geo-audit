---
name: ai-overview-check
description: Find which Google searches show an AI Overview, what it says and which sources it cites, and listen for buyer-intent conversations on Reddit. Use when the user asks "which searches trigger an AI Overview", "does Google show an AI Overview for X", "AI Overview sources", "SGE check", "AI Mode citations", "what does Google's AI say about my category", "where are buyers asking about <category> on Reddit", "Reddit mentions of my brand", "buyer intent posts", "social listening for AI visibility", or wants the Google and Reddit side of a GEO report.
---

# Google AI Overview and Reddit buyer-intent check

Two Actors on the **ai-visibility** connector cover this: `muhammadafzal--google-ai-overviews-scraper` (one row per Google query) and `muhammadafzal--reddit-buyer-intent-brand-monitor` (one row per Reddit post). Read `../ai-visibility-check/references/actor-contracts.md` for their exact input and output before the first call in a conversation.

Why they belong together: AI Overviews and AI answer engines cite the same kind of sources, and Reddit threads are among the most cited. Knowing which searches show an Overview, which domains it cites, and where buyers ask for recommendations tells the user where to earn a mention.

## Steps

1. **Build the query list.**
   - For Google: take the user's search terms as typed by searchers (2 to 300 characters each), up to 25 per run. If the user gives a category instead, draft 10 to 20 searches across informational ("what is", "how to"), commercial ("best", "top", "vs", "alternatives") and brand terms, show them, and ask to confirm. Set `countryCode` and `languageCode` to the user's market (two-letter codes, default `us` and `en`).
   - For Reddit: 3 to 8 phrases a buyer would write ("looking for a <category>", "recommend a <category> for <audience>", "<competitor> alternative"), the brand name in `brandNames`, optional subreddits without `r/`, `sort: "relevance"`, `timeframe` `month` (or `year` for niche categories), `maxResults` 50.
   Run only the part the user asked for; run both for a "Google and Reddit" or "social listening" request.

2. **Estimate the cost**: Google USD 0.005 per query plus USD 0.0125 start (25 queries about USD 0.14); Reddit USD 0.005 per post plus USD 0.006 start (50 posts about USD 0.26). State it; above USD 0.50 run only after the user confirms. Say it is billed to the user's own Apify account.

3. **Run** with the input shapes from the contract file, both Actors in parallel when both are wanted. Do not set a maximum total charge on the Reddit run below USD 0.25; the Actor rejects it.

4. **Wait for every run**: `get-actor-run` with `waitSecs: 45` until the status is terminal, up to 8 times.

5. **Fetch results**: `get-dataset-items` for each run (`limit` = queries, or `maxResults`), then `get-key-value-store-record` with `SUMMARY` (Google) and `OUTPUT` (Reddit). The Google run ends FAILED when every query failed; still read the dataset.

6. **Reconcile and classify.**
   - Google: every query appears once. Classify each row as **AI Overview shown** (`overviewPresent: true`), **no AI Overview** (`status` `success` or `warning` with `overviewPresent: false` and no challenge warning), **blocked** (any warning containing "challenge or unusual-traffic"; Google showed a verification page, so this is not evidence either way), or **failed** (`status: "error"`). Blocked and failed queries are still charged; offer to retry them later.
   - Reddit: keep rows with a non-empty `matchedQueries` or `matchedBrands`, or `buyerIntentScore` of 40 or more; count the rest as "not relevant". The `OUTPUT.warnings` line about Reddit returning HTTP 403 and using the fallback data route is normal.

7. **Answer** in the format below and offer follow-ups: audit the user's page for a query whose Overview cites competitors (`geo-site-audit`), or add the Reddit threads with the highest intent to an outreach or community plan.

## Answer format

**Google**:
1. Summary line: N queries, X show an AI Overview, Y do not, Z blocked, W failed, country and language.
2. Table: Query, AI Overview (yes / no / blocked), Cited domains (first 3), Brand cited (yes when any source domain is the user's), First line of the Overview (shortened).
3. Source leaderboard: domains counted across all Overviews, most cited first, with the user's domain highlighted or "not cited anywhere" stated.
4. One line: "Apify cost: about USD X for N queries, billed to your Apify account."

**Reddit**:
1. Summary line: posts returned, relevant posts, posts that mention the brand, subreddits seen.
2. Table of relevant posts sorted by `buyerIntentScore` then `score`: Title (shortened, linked to `url`), Subreddit, Intent score and signals, Brands matched, Upvotes and comments, Posted.
3. Three patterns you see: recurring questions, competitors people recommend, subreddits worth joining.
4. The cost line.

Do not paste raw JSON. Quote Overview text and post text in short excerpts only. Show Reddit usernames only when the user asks who posted.

## When things go wrong

- Authorization error or connector not connected: ask the user to connect Apify on the plugin's Connectors tab, then retry.
- Most Google rows are blocked: Google is challenging automated requests at the moment; report the queries that did return, say the blocked ones were charged at USD 0.005 each, and suggest retrying in an hour or splitting into smaller runs.
- Reddit returns few relevant posts: widen `timeframe` to `year`, drop the subreddit filter, or rewrite the phrases in the words buyers use; `sort: "new"` returns unrelated recent posts, so never use it for intent listening.
