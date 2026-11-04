---
name: geo-site-audit
description: Audit web pages and sites for AI answerability and AI crawler readiness, and produce a prioritized fix list. Use when the user asks for a "GEO audit", "AEO audit", "AI readiness audit", "is my site ready for AI search", "why doesn't ChatGPT cite my site", "llms.txt check", "is GPTBot or ClaudeBot blocked", "AI crawler access", "structured data for AI", "make this page citable", "AI answerability score", "content audit for AI search", or gives a URL or domain and asks how to improve its visibility in AI answers.
---

# GEO site audit

You audit pages with two Actors on the **ai-visibility** connector (tool names `muhammadafzal--geo-content-auditor` and `muhammadafzal--ai-readiness-auditor`), then merge their findings into one fix list. Read `../ai-visibility-check/references/actor-contracts.md` for the exact input and output of both before the first call in a conversation.

| Actor | Looks at | Produces |
|---|---|---|
| `geo-content-auditor` | The content of each listed page: question headings, facts, entity clarity, FAQ, Q&A fit to target prompts | AI answerability score 0 to 100 with five sub-scores, issues, quick wins, missing schema types, suggested Q&A pairs |
| `ai-readiness-auditor` | The technical side: robots.txt rules for AI crawlers, llms.txt, sitemap, JSON-LD, metadata, canonical, indexability, response time; can follow links across the site | AI readiness score 0 to 100 across seven categories, coded findings with severity |

Run both for a full audit. Run only the readiness auditor when the question is technical ("is ClaudeBot blocked", "do we have llms.txt"). Run only the content auditor when the user asks about one page's content or wants Q&A suggestions.

## Steps

1. **Decide the page list.**
   - One or more specific URLs: audit exactly those. Content auditor `startUrls` = the list, `maxPages` = count. Readiness auditor `startUrls` = the list, `maxDepth: 0`, `includeSitemapUrls: false`, `maxPages` = count.
   - A domain or homepage ("audit my site"): readiness auditor with the homepage, `maxDepth: 1`, `includeSitemapUrls: true`, `maxPages: 10` (offer 25 for a bigger site). Then pick the pages that matter for AI answers (homepage, product or service pages, pricing, top guides, FAQ) for the content auditor, 5 to 10 URLs; ask the user which pages matter if the site is large.
   - If the user has target prompts (from the `ai-visibility-check` skill or stated), pass them as `targetPrompts` to the content auditor so each page gets a `prompt_fit_score`. Pass `brandName` too.

2. **Estimate the cost**: USD 0.050 per page per Actor plus USD 0.006 start per run. A 10-page full audit is about USD 1.01. State it and, above USD 0.50, run only after the user confirms. Say it is billed to the user's own Apify account.

3. **Run both Actors in parallel** with the input shapes from the contract file. For readiness runs over 20 pages pass `callOptions.timeout: 900`.

4. **Wait for every run to finish**: `get-actor-run` with `waitSecs: 45`, repeat until the status is terminal, up to 8 times. Never report partial results as final without saying so.

5. **Fetch the results**: `get-dataset-items` for each run, then `get-key-value-store-record` with `SUMMARY` (content auditor) and `OUTPUT` (readiness auditor). The readiness run ends FAILED when no page could be audited; still read `OUTPUT.warnings` to explain why (blocked, robots-blocked, fetch-failed, rejected-url).

6. **Reconcile.** Every URL sent appears once in your answer. A content row with `error` filled, or a URL in `SUMMARY.failed_urls`, is "could not audit" and was not charged. Readiness pages that redirected to the same final page are one row.

7. **Build one fix list** by merging:
   - Readiness `findings` with severity `critical` or `high` first (blocked AI crawlers, noindex, missing sitemap, unreachable llms.txt, missing JSON-LD on important pages), then `medium`.
   - Content `quick_wins` and the top `issues` across pages, grouped by theme (definition near the top, FAQ section, labelled statistics, question-style headings, author and date, schema types).
   - `missing_schema_types` across pages and the site-level `missing_schema_summary`.
   - Keep each fix as: what to change, on which pages, why it matters for AI answers, and the expected effect on the score dimension.

## Answer format

1. Headline: site content score and grade (`SUMMARY.site_ai_answerability_score`, `site_grade`), site readiness (average `overallScore` and the worst grade), pages audited, pages that could not be audited.
2. Site-level signals table (from readiness `OUTPUT.sites` and content `SUMMARY`): robots.txt, sitemap, llms.txt, AI crawler access for GPTBot, ChatGPT-User, Google-Extended, ClaudeBot, PerplexityBot (say "no explicit rule" for `unspecified`), pages with JSON-LD.
3. Per-page table: URL (shortened), Content score and grade, Readiness score and grade, Words, Top issue.
4. The fix list, numbered, highest impact first, at most 10 items, each 1 to 3 sentences. Mark the three quick wins.
5. For a single-page audit, add the suggested Q&A pairs as a ready-to-paste FAQ draft and the five sub-scores with their maximums.
6. One line: "Apify cost: about USD X for N page audits in M runs, billed to your Apify account."
7. Offer next steps: re-audit after the fixes (scores move a few points between runs, so look for 10 or more), run the `ai-visibility-check` skill to see whether citations follow, or `ai-overview-check` for Google.

Do not paste raw JSON. Quote the page's own title when naming a page. Do not present the content auditor's score as a measurement of real AI citations; it is a rubric judgment.

## When things go wrong

- Authorization error or connector not connected: ask the user to connect Apify on the plugin's Connectors tab, then retry.
- Readiness `OUTPUT.outcome` is `blocked` or warnings say `robots-blocked`: the site's robots.txt disallows generic crawlers or the server refused the request; report it as a finding in itself (AI crawlers will see the same) and audit the content with the content auditor only if the page is reachable.
- A page is JavaScript-rendered and `wordCount` is tiny: say the auditor reads the server HTML only, and that AI crawlers mostly do the same, which is itself a finding.
- Very long page: the content auditor scores the first 3,000 characters and 30 headings; say so when a long page scores low on facts.
