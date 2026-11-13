# Actor contracts

Verified against live runs on 2026-11-04 (all seven Actors ran once with a one-item input; every dataset row and run record below was read from those runs). Prices are Apify Store pay-per-event prices on the FREE tier on 2026-11-04; paid Apify plans pay 2.5 to 20 percent less. Only results written to the dataset are charged, plus one start event per started GB of run memory.

All Actors belong to Apify user `muhammadafzal`. On the ai-visibility MCP server, the tool name is the Actor ID with `/` replaced by `--`, for example `muhammadafzal--ai-citation-monitor`. Some apps add a server prefix, turn hyphens into underscores, or shorten a long name and add a suffix; match on the start of the Actor slug. Each tool returns the run (`runId`, `status`, `storages.datasets.default.id`, `storages.keyValueStores.default.id`), not the rows. If a tool response already includes dataset items, use them; otherwise fetch them with `get-dataset-items`.

Run options (`memory`, `timeout`, `maxTotalChargeUsd`) go in the tool's `callOptions`, never inside `input`.

## Shared rules

- Send only the fields listed below in `input`. Every Actor handles its own network routing and its own AI model access internally, so no other fields are needed and the user never supplies an AI account.
- Free Apify accounts get at most 20 result rows per run from the rank tracker, the citation monitor and the content auditor. Paid accounts use the limits in each section.
- The four engines are always spelled `perplexity`, `chatgpt`, `claude`, `gemini` (lowercase) in input and in dataset rows.
- AI answers change between runs and days. Report one run as a snapshot and only call something a trend after two or more runs with the same `trackingId` and the same prompt list.
- A row whose `error` field is filled was not charged. Count it as "check failed", never as "not mentioned".
- Always compute a cost from the prices in this file. A run response can include usage or cost fields (for example `usageTotalUsd`); those are Apify's internal platform usage and are far lower than the billed price, so never report them as the cost.

---

## Rank tracker: `ai-visibility-rank-tracker`

What it does: asks each engine each prompt, records whether the brand is mentioned, its position among the tracked brands, share of voice, citations, and a light sentiment label. Keeps history per `trackingId` so the next run reports gained, lost, improved and declined.

Price: USD 0.060 per successful prompt-engine check (GOLD 0.048). Add-ons per successful check: USD 0.050 for citation extraction when `extractCitations` is true, and USD 0.050 for sentiment when `analyzeSentiment` is true and the brand was mentioned. Start USD 0.010 (default 512 MB, one event). A full check with both add-ons costs up to USD 0.16; a mention-only check costs USD 0.06. Up to 200 prompts per run, each prompt on every selected engine. About 5 seconds per check, checks run one at a time; a 10-prompt, 4-engine run takes 3 to 5 minutes. Run timeout default 3600 s.

Input:
```json
{
  "brandName": "Notion",
  "brandDomain": "notion.so",
  "industry": "productivity software",
  "prompts": ["What is the best note-taking app for teams?"],
  "platforms": ["perplexity", "chatgpt", "claude", "gemini"],
  "competitors": ["Evernote", "Obsidian", "Coda"],
  "extractCitations": false,
  "analyzeSentiment": false,
  "trackingId": "notion-productivity",
  "useWebSearch": true,
  "webSearchResults": 3,
  "maxPrompts": 25
}
```
- `brandName` is required. Put the domain in `brandDomain`, never in `brandName`.
- `prompts` empty means the Actor generates prompts from `industry`; always send explicit prompts so runs are comparable (see prompt-set-design.md).
- `maxPrompts` caps checks for cost control (1 to 200); set it to the length of `prompts`.
- `trackingId` must stay identical across runs for the gained/lost comparison to work. Use `<brand-slug>-<category-slug>`.
- Set `extractCitations: true` only when the user wants citations from this Actor and you are not also running the citation monitor. Set `analyzeSentiment: true` only when the user asks about sentiment.

Dataset row (snake_case), one per prompt and engine: `brand`, `brand_domain`, `engine`, `model_used` (neutral label such as `fixed-ai-model`), `prompt`, `mentioned` (bool), `position` (int or null: rank by first appearance among brand plus competitors), `share_of_voice` (0 to 100 or null), `cited` (bool), `citations[]` (URLs), `brand_cited_url`, `mention_count`, `mention_context` (first sentence with the brand), `sentiment` (`positive` | `neutral` | `negative` | null), `competitors_mentioned[]`, `previous_mentioned`, `previous_position`, `position_change` (positive means moved up), `visibility_status` (`new` | `gained` | `lost` | `improved` | `declined` | `unchanged` | `absent` | `error`), `error`, `checked_at`.

Key-value record `SUMMARY`: `brand`, `tracking_id`, `visibility_score` (percent of successful checks that mentioned the brand), `checks_total`, `checks_successful`, `checks_failed`, `checks_mentioned`, `avg_position`, `avg_share_of_voice`, `cited_count`, `gained`, `lost`, `improved`, `declined`, `per_engine` (object: engine to `{checks, mentioned}`), `checked_at`, `stop_reason` (null, or the AI access error that stopped the run early; the checks already saved stay valid and the rest were not charged).

Data-quality rules: brand matching is plain case-insensitive text matching, so a short or generic brand name (for example "Notion" matches the word "notion") can produce false mentions; when the brand name is a common word, read `mention_context` before reporting a mention. Sentiment comes from word lists, not a model; present it as a light signal. `position` is relative to the competitor list you sent, not to every brand in the answer.

---

## Citation monitor: `ai-citation-monitor`

What it does: for each prompt and engine, records every source URL the engine cited, whether your domain is among them, which competitor domains were cited, and run-over-run changes. Its `SUMMARY` has the citation leaderboard (which domains get cited most) and the citation gap (domains cited more often than yours).

Price: USD 0.050 per successful prompt-engine check (GOLD 0.040), start USD 0.010 (default 256 MB, one event). Up to 200 prompts per run on four engines. Each check times out after 45 s with no retry; a failed check is saved with `citation_status: "error"` and not charged. Web search is always on. About 5 to 10 seconds per check.

Input:
```json
{
  "brandDomain": "notion.so",
  "competitorDomains": ["evernote.com", "obsidian.md"],
  "prompts": ["What is the best note-taking app for teams?"],
  "industry": "productivity software",
  "platforms": ["perplexity", "chatgpt", "claude", "gemini"],
  "maxPrompts": 25,
  "trackingId": "notion-productivity"
}
```
- `brandDomain` is the registrable domain without `https://` or `www.`.
- Send the same `prompts` and `trackingId` as the rank tracker so the two datasets join on `prompt` and `engine`.

Dataset row (snake_case): `brand_domain`, `engine`, `model_used`, `prompt`, `brand_cited` (bool), `brand_cited_urls[]`, `citation_urls[]`, `citation_domains[]` (unique, in order of first appearance), `competitor_domains_cited[]`, `total_citations`, `new_domains[]`, `lost_domains[]` (versus the previous run), `previous_brand_cited` (null on the first run), `citation_status` (`new` | `gained` | `lost` | `unchanged` | `absent` | `error`), `error`, `checked_at`.

Key-value record `SUMMARY`: `brand_domain`, `tracking_id`, `prompts_checked`, `engines[]`, `checks_total` (successful checks), `brand_citation_count`, `brand_citation_rate` (percent), `top_cited_domains[]` of `{domain, count, share_pct}` (up to 20), `citation_gap_domains[]` (same shape, up to 10: domains cited more often than the brand), `gained`, `lost`, `stop_reason`, `checked_at`.

Data-quality rule: an answer with zero citations is a valid charged check (`total_citations: 0`), not an error. ChatGPT returned 2 citations and Perplexity usually returns 5 to 10, so compare citation rates per engine, not across engines.

---

## Brand reach checker: `ai-brand-reach-checker`

What it does: one web-grounded question per engine, "which are the top brands in <industry>", without naming the brand. A mention means the engine surfaced the brand on its own. Cheapest possible first look: at most four charged events per run.

Price: USD 0.050 per successful platform check (GOLD 0.045), start USD 0.005. One question per platform per run, up to four platforms. Each request times out after 120 s. A platform that fails or returns nothing is listed with an `error` and not charged; if no platform answers, no row is written and only the start fee applies. A run takes 20 to 60 seconds.

Input:
```json
{
  "brandName": "Notion",
  "brandDomain": "notion.so",
  "industry": "note-taking and workspace apps",
  "platforms": ["perplexity", "chatgpt", "claude", "gemini"],
  "competitors": ["Evernote", "Obsidian"]
}
```
- `brandName` and a non-empty `industry` are required (an empty `industry` is rejected). The wording of `industry` changes the answers, so reuse the exact same string for comparable snapshots.

Dataset row: exactly one row per run: `brand`, `brand_domain`, `overall_visible` (bool), `visibility_score` (0 to 100 across the checked platforms), `platforms[]` of `{name` (display name such as `ChatGPT`), `mentioned`, `cited`, `mention_count`, `mention_context`, `model_used`, `error`, `competitor_mentions[]}`, `competitor_mentions[]` (union across platforms), `recommendations[]` (generic text; do not present these as analysis), `checked_at`.

No key-value record. Read the single dataset row.

Data-quality rule: `visibility_score` is computed over platforms that answered, so with one platform it is 0 or 100. Report the per-platform `mentioned` flags, and quote `mention_context` so the user sees the sentence that counted as a mention.

---

## GEO content auditor: `geo-content-auditor`

What it does: fetches each listed page once and scores AI answerability 0 to 100 on a rubric with five sub-scores (structure max 30, facts 25, entity 20, readability 15, authority 10), with issues, quick wins, missing schema types and suggested Q&A pairs. Optional prompt-fit score against the user's target prompts.

Price: USD 0.050 per audited page (GOLD 0.040), start USD 0.006 (default 512 MB, one event). Up to 50 pages per run (`maxPages`), 20 on free accounts. About 10 seconds per normal page, up to a minute for very large pages; pages are audited one at a time. A page that returns HTTP 400 or higher, fails to load, or fails the AI step is not charged.

Input:
```json
{
  "startUrls": [{ "url": "https://example.com/pricing" }],
  "brandName": "Example",
  "targetPrompts": ["What is the best X for Y?"],
  "maxPages": 10
}
```
- `startUrls` is required and uses the `{url}` object form. It audits only the URLs listed; it does not crawl.
- Set `maxPages` to the number of URLs sent.

Dataset row (snake_case): `url`, `page_title`, `ai_answerability_score` (null when the audit failed), `structure_score`, `facts_score`, `entity_score`, `readability_score`, `authority_score`, `grade` (A 90 to 100, B 75 to 89, C 60 to 74, D 40 to 59, F below 40), `strengths[]`, `issues[]` (prioritized), `quick_wins[]` (top 3), `missing_schema_types[]`, `suggested_qa_pairs[]` of `{question, answer_hint}`, `llms_txt_present`, `has_json_ld`, `word_count`, `prompt_fit_score` (null without target prompts), `model` (neutral label), `error`, `audited_at`.

Key-value record `SUMMARY`: `brand_name`, `pages_audited`, `successful_audits`, `site_ai_answerability_score` (average), `site_grade`, `top_issues[]` (up to 10, by frequency), `quick_wins[]` (up to 10), `priority_pages[]` of `{url, ai_answerability_score, grade}` (lowest five), `pages_with_json_ld`, `pages_with_llms_txt`, `missing_schema_summary[]` of `{schemaType, count}`, `failed_urls[]` of `{url, error}`, `model`, `checked_at`.

Data-quality rule: scores are rubric judgments by a model and can move a few points between runs; treat a difference under 5 points as noise. The audit reads the first 3,000 characters of body text and up to 30 headings, so very long pages are scored on their opening.

---

## AI readiness auditor: `ai-readiness-auditor`

What it does: a technical audit of how discoverable pages are to AI crawlers: robots.txt rules for GPTBot, ChatGPT-User, Google-Extended, ClaudeBot and PerplexityBot, `/llms.txt`, XML sitemap, JSON-LD types, metadata, canonical, indexability, readable word count, and response time, with coded findings and a 0 to 100 score across seven categories. Can follow same-origin links to audit more of the site.

Price: USD 0.050 per audited page (GOLD 0.040), start USD 0.006 (default 512 MB, one event). `startUrls` holds 1 to 20 URLs; `maxPages` 1 to 100 caps the charged pages across the whole run (default 5); `maxDepth` 0 to 3 (default 1) follows same-origin links; `includeSitemapUrls` (default true) fills the queue from the sitemap. Run timeout default 300 s, so for more than 20 pages set `callOptions.timeout` to 900. A default single page took about 4 seconds. Non-HTML pages, HTTP errors, robots-blocked pages and fetch failures produce a warning and no charge; two URLs that redirect to the same page are charged once. A run with zero delivered audits ends FAILED; still read `OUTPUT`.

Input:
```json
{
  "startUrls": [{ "url": "https://example.com/" }],
  "maxPages": 5,
  "maxDepth": 1,
  "respectRobotsTxt": true,
  "includeSitemapUrls": true,
  "requestTimeoutSecs": 20
}
```
- For "audit these exact pages" send the URLs with `maxDepth: 0`, `includeSitemapUrls: false` and `maxPages` equal to the URL count. For "audit my site" send the homepage with `maxDepth: 1` and `maxPages` 10 to 25.

Dataset row (camelCase): `url`, `finalUrl`, `outcome` (`data` | `blocked` | `failed`), `statusCode`, `contentType`, `auditedAt`, `responseTimeMs`, `overallScore` (0 to 100), `grade` (A to F), `categoryScores` `{crawlability, aiDiscoverability, metadata, structuredData, content, accessibility, performance}` each 0 to 100, `title`, `metaDescription`, `canonicalUrl`, `language`, `wordCount`, `indexable`, `robotsAllowed`, `llmsTxtStatus` and `sitemapStatus` (`present` | `missing` | `unreachable`), `aiBotAccess` `{GPTBot, ChatGPT-User, Google-Extended, ClaudeBot, PerplexityBot}` each `allowed` | `blocked` | `unspecified`, `structuredDataTypes[]`, `findings[]` of `{code, category, severity (critical | high | medium | low | info), message, recommendation, evidence}`, `discoveredUrls[]`.

Key-value record `OUTPUT`: `outcome` (`data` | `blocked` | `empty`), `summary` (text), `deliveredAudits`, `datasetItemCount`, `chargedPageAuditEvents`, `startEventCount`, `warningCount`, `warnings[]` of `{url, kind (blocked | robots-blocked | fetch-failed | rejected-url | duplicate-redirect | input-rejected), message}`, `sites[]` of `{origin, robotsStatus, sitemapStatus, sitemapUrl, llmsTxtStatus, llmsTxtUrl, aiBotAccess}`, `stoppedByChargeLimit`, `completedAt`.

Data-quality rules: `aiBotAccess` reports `unspecified` when robots.txt has only wildcard rules, which usually means the AI crawlers are allowed by the wildcard; say "no explicit rule" rather than "blocked". Content rendered by JavaScript after load is not seen, so a low `wordCount` on a JavaScript-heavy site is a measurement limit, and you should say so. Finding codes are stable (for example `BREADCRUMB_SCHEMA_MISSING`, `SCRIPT_COUNT_HIGH`, `FRESHNESS_SIGNAL_MISSING`); group findings by `severity` then `category`.

---

## Google AI Overview extractor: `google-ai-overviews-scraper`

What it does: loads the rendered Google results page for each query and records whether an AI Overview is shown, its text and bullets, and the supporting source links with domains.

Price: USD 0.005 per processed query (GOLD 0.004), charged for every query that completes, including queries with no AI Overview and queries where Google showed a challenge page. Start USD 0.006 per started GB; the default run uses 2 GB, so the start fee is USD 0.0125. Up to 25 queries per run (`queries` 1 to 25, each 2 to 300 characters). Run timeout default 300 s. About 10 to 20 seconds per query. If every query fails, the run ends FAILED; still read the dataset.

Input:
```json
{
  "queries": ["best crm for small business"],
  "countryCode": "us",
  "languageCode": "en"
}
```
- `countryCode` and `languageCode` are two-letter codes; they change Google's request parameters only.

Dataset row (camelCase), one per query: `query`, `searchUrl`, `status` (`success` | `warning` | `error`), `overviewPresent` (bool), `overviewText` (null when none), `overviewBullets[]`, `sources[]` of `{title, url, domain, snippet}`, `googleTitle`, `countryCode`, `languageCode`, `extractor` (`dom` | `none`), `warnings[]`, `error`, `scrapedAt`.

Key-value record `SUMMARY`: `queriesRequested`, `requestsFinished`, `requestsFailed`, `queryPriceUsd`, `queryFailures`, `countryCode`, `languageCode`, `warnings[]`, `scrapedAt`.

Data-quality rule (important, observed on 2026-11-04): when `warnings[]` contains "Google returned a challenge or unusual-traffic page", Google blocked that request and `overviewPresent: false` means nothing. Report that query as **blocked** ("Google showed a verification page; not a result"), never as "no AI Overview", and offer to retry later. Only a row with `status: "success"` or a `warning` row without the challenge text is evidence about whether an AI Overview exists. The query is still charged.

---

## Reddit buyer-intent monitor: `reddit-buyer-intent-brand-monitor`

What it does: searches public Reddit posts for the given phrases (and for each brand name as its own search), returns matching posts with a 0 to 100 buyer-intent score and the intent signals found, and flags which queries and brands each post matched.

Price: USD 0.005 per delivered post (GOLD 0.004), start USD 0.006 (default 1 GB, one event). `maxResults` 1 to 500 unique posts across all searches (default 50). If you set a maximum total charge in `callOptions`, it must be at least USD 0.25 or the run is rejected. Run timeout default 300 s. A run of 50 posts takes 30 to 90 seconds.

Input:
```json
{
  "queries": ["looking for a crm for a small team", "best crm for startups"],
  "brandNames": ["HubSpot"],
  "subreddits": ["smallbusiness", "startups"],
  "sort": "relevance",
  "timeframe": "month",
  "maxResults": 50
}
```
- `sort`: `relevance` | `new` | `hot` | `top` | `comments`. Use `relevance` for buyer-intent listening; `new` returns the newest posts on Reddit regardless of fit (observed on 2026-11-04: with `new`, none of the returned posts matched the query).
- `subreddits` are names without `r/`; empty searches all of Reddit.
- Each entry in `brandNames` is also run as a search query.

Dataset row (camelCase): `id`, `title`, `text` (title plus body, truncated), `author`, `subreddit`, `url`, `createdAt`, `score` (upvotes), `numComments`, `matchedQueries[]`, `matchedBrands[]`, `buyerIntentScore` (0 to 100), `intentSignals[]` (for example `active research`, `solution category`, `comparison`, `recommendation request`), `source` (`reddit-direct` | `scrapecreators`, the second being the fallback data route when Reddit's own search refuses the request), `scrapedAt`.

Key-value record `OUTPUT`: `delivered`, `routes` `{direct, provider}` (candidate counts per route), `warnings[]` (strings; "Reddit direct search returned HTTP 403; using the fallback data route" is normal and not an error), `queries[]` (including brand names), `brandNames[]`, `maxResults`, `scrapedAt`.

Data-quality rules: only rows with a non-empty `matchedQueries` or `matchedBrands`, or `buyerIntentScore` of 40 or more, are worth showing; list the rest only as a count. Posts are public Reddit content with the author's username; show `author` only when the user asks who posted. `createdAt` is the post time, not the scrape time.

---

## Cost formula

cost = runs × start fee + charged results × per-result price.

Reference figures on the FREE tier:
- Brand reach checker, four engines: USD 0.205.
- Rank tracker, 10 prompts on 4 engines, mention-only: 40 × 0.060 + 0.010 = USD 2.41. With citation extraction: 40 × 0.110 + 0.010 = USD 4.41. With sentiment too, worst case: 40 × 0.160 + 0.010 = USD 6.41.
- Citation monitor, 10 prompts on 4 engines: 40 × 0.050 + 0.010 = USD 2.01.
- Standard visibility check (rank tracker mention-only plus citation monitor, 10 prompts, 4 engines): about USD 4.42.
- Content audit of 10 pages: USD 0.506. Readiness audit of 10 pages: USD 0.506.
- 25 Google AI Overview queries: USD 0.1375.
- 50 Reddit posts: USD 0.256.

Quote costs as "about", because failed checks are not charged and the user's Apify plan tier may be discounted.
