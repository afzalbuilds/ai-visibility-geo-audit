---
name: ai-visibility-check
description: Check and track whether AI answer engines mention, rank and cite a brand. Use when the user asks "does ChatGPT mention my brand", "are we visible in AI search", "AI visibility", "AI search visibility", "brand mentions in ChatGPT, Gemini, Perplexity or Claude", "share of voice in AI answers", "which sources do AI engines cite for my category", "citation rate", "GEO tracking", "AEO tracking", "LLM visibility", "generative engine optimization report", "rank in AI answers vs competitors", or wants a weekly AI visibility trend for a brand, product, company or website.
---

# AI visibility check

You measure brand visibility in AI answers by running Imzlab Technologies' Actors on Apify through the **ai-visibility** connector. Each Actor is a tool on that connector whose name ends with the Actor slug, for example `muhammadafzal--ai-visibility-rank-tracker`. The connector also exposes `get-actor-run`, `get-dataset-items` and `get-key-value-store-record`, which you need to collect results.

Read `references/actor-contracts.md` for the exact input and output of every Actor before your first call in a conversation, and `references/prompt-set-design.md` for how to build the prompt set.

Three Actors do this job. Pick the depth the user needs:

| Depth | Actor(s) | Answers | Cost on the FREE tier |
|---|---|---|---|
| Quick look | `ai-brand-reach-checker` | Do the four engines name the brand among the top brands in its category? | about USD 0.21 |
| Standard | `ai-visibility-rank-tracker` (mention-only) | Mentioned, position and share of voice versus competitors per prompt and engine, with run-over-run changes | about USD 2.41 for 10 prompts on 4 engines |
| Full | rank tracker plus `ai-citation-monitor` | Standard, plus which domains each engine cites, the brand's citation rate and the citation gap | about USD 4.42 for 10 prompts on 4 engines |

Default to **Quick look** when the user asks a yes/no question or has given only a brand name. Default to **Standard** when they ask for a report, a rank, competitors or a trend. Add the citation monitor when they ask about citations, sources, links, "where do the engines get their information" or GEO strategy. Never run the rank tracker's own citation and sentiment add-ons at the same time as the citation monitor.

## Steps

1. **Collect the inputs**: brand name, brand domain, category in buyer language, 2 to 5 competitors with domains, market and language. For Standard and Full, build the prompt set as described in `references/prompt-set-design.md` and show it, with the `trackingId`, for approval. For Quick look, confirm the one-line `industry` wording.

2. **Estimate the cost** from `references/actor-contracts.md` and state it: number of prompts × engines × per-check price plus start fees. For any estimate above USD 0.50, show it and run only after the user confirms. Say that it is billed to the user's own Apify account.

3. **Run the Actors.** Use the input shapes in the contract file. Send the same `prompts`, `platforms`, `competitors` and `trackingId` to the rank tracker and the citation monitor so rows join on `prompt` and `engine`. Set `maxPrompts` to the number of prompts. For the rank tracker in Standard or Full depth send `extractCitations: false` and `analyzeSentiment: false`; enable sentiment only when the user asked for it. The two Actors can run in parallel. For more than 15 prompts on 4 engines pass `callOptions.timeout: 1800`.

4. **Wait for every run to finish.** A tool response with status `RUNNING` or `READY` is not a result. Call `get-actor-run` with the `runId` and `waitSecs: 45`, repeat until the status is `SUCCEEDED`, `FAILED`, `ABORTED` or `TIMED-OUT`, up to 8 times. Never report a run as done while it is running and never invent a result.

5. **Fetch the results**, whatever the run status: `get-dataset-items` with the run's default `datasetId` (`limit` at least prompts × engines, minimum 20), then `get-key-value-store-record` with the run's default `keyValueStoreId` and the record named in the contract file (`SUMMARY` for the rank tracker and the citation monitor; the brand reach checker has a single dataset row and no record). A `stop_reason` in `SUMMARY` means the AI access route stopped the run early; the saved checks are valid and the rest were not charged.

6. **Reconcile.** Every prompt × engine pair you sent must appear once in your answer. A row with `error` filled is "check failed" and was not charged; it is not "not mentioned". If a pair is missing (free-plan cap of 20 rows, charge limit, early stop), say so and offer to rerun the rest.

7. **Apply the data-quality rules** in the contract file: read `mention_context` for common-word brand names, treat sentiment as a light signal, compare citation rates per engine.

8. **Answer** in the format below and offer the follow-ups: the `geo-site-audit` skill when mentions are high but citations are low, the `ai-overview-check` skill for Google, and a repeat run in a week with the same prompt set for a trend.

## Answer format

**Quick look**: one line per engine: "ChatGPT: mentioned (2 times), not cited. Context: '<mention_context>'" then the competitors named and the visibility score. Do not repeat the Actor's `recommendations`.

**Standard and Full**: in this order:
1. Headline: visibility score, average position, share of voice, and (Full) citation rate, with the engines and prompt count. On a repeat run add gained, lost, improved and declined counts.
2. A per-engine table: Engine, Checks, Mentioned, Mention rate, Avg position, Cited (Full).
3. A per-prompt table: Prompt (shortened), then one column per engine showing `position` (or "no" when not mentioned, "err" when failed), and the competitors that appeared.
4. (Full) Citation leaderboard: top 10 `top_cited_domains` with count and share, with the brand's own domain highlighted, then the `citation_gap_domains` as "sites to get featured on".
5. One line: "Apify cost: about USD X for N checks in M runs, billed to your Apify account."
6. Three short observations and the next step, grounded in the numbers (for example "mentioned in 60 percent of answers but cited in 10 percent, so engines learn about you from third-party pages; audit your own pages next").

Keep the engines' own wording when you quote `mention_context`. Do not paste raw JSON. Give `checked_at` as the run date.

## When things go wrong

- The connector is not connected or a tool call returns an authorization error: tell the user to open the plugin's Connectors tab and connect Apify (a free account works), then retry.
- `SUMMARY.stop_reason` is set or many rows carry the same `error`: the Actor's AI access route refused requests; report the completed checks, say the rest were not charged, and offer to retry later.
- The run reports a charge limit: the user's Apify maximum charge per run was hit; report which prompts were skipped and suggest raising the limit in Apify Console or fewer engines.
- Only 20 rows came back from a larger request: the user's Apify account is on the free plan, which caps these Actors at 20 results per run; say so and split into runs of 20 or suggest a paid plan.
- The brand is a common word and mentions look wrong: rerun with the full product name (for example "Notion app") and tell the user why.
