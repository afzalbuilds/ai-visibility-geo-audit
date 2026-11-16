# n8n workflow templates: AI Visibility Tracker & GEO Audit

Two importable n8n workflows that run the same Apify Actors as this plugin. Built with the official n8n skills plugin and n8n's instance-level MCP, on n8n Cloud with the Apify node (`@apify/n8n-nodes-apify` 0.9.0).

| File | What it does | Trigger | Needs |
|---|---|---|---|
| `weekly-ai-visibility-tracker.json` | Every Monday asks Perplexity, ChatGPT, Claude and Gemini a fixed list of prompts, logs whether your brand was mentioned to a Google Sheet and emails a summary with changes since the last run | Schedule, Monday 08:00 | Apify, Google Sheets, Gmail |
| `geo-page-audit-form.json` | A form takes a page URL and shows its AI answerability score (0 to 100), sub-scores, quick wins and issues on the form's end page | n8n Form | Apify |

Actors used: `muhammadafzal/ai-visibility-rank-tracker` and `muhammadafzal/geo-content-auditor`. The Actors handle their own AI model access, so no AI account is needed in n8n.

## Import

1. In n8n, install the verified community node **Apify** (Settings → Community nodes, or search "Apify" in the node panel).
2. Create a workflow, open the **⋯** menu and choose **Import from file**, then pick a JSON file from this folder.
3. Open each node marked with a warning and select your credential: an Apify account, plus Google Sheets and Gmail for the weekly tracker.
4. Publish the workflow.

## Setting up the weekly tracker

Open the **Check brand visibility in AI answers** node and edit the JSON input: `brandName`, `brandDomain`, `industry`, `prompts`, `competitors` and `trackingId`. The sample input tracks Notion as a placeholder. Keep `trackingId` and the prompt list the same from week to week, or the gained and lost comparison will not line up. Set `maxPrompts` to the number of prompts.

The history goes to a tab named `Checks` with this header row:

`Checked at` | `Brand` | `Engine` | `Prompt` | `Mentioned` | `Position` | `Share of voice` | `Change` | `Competitors mentioned` | `Context`

Set the recipient in the **Email weekly visibility summary** node. A check that failed is logged as `check failed` and is never counted as "not mentioned".

## Cost

Billed by Apify to your own account. Weekly tracker: about USD 0.06 per prompt per engine plus USD 0.01 per run, so 5 prompts on 4 engines is about USD 1.21 a week; the workflow caps one run at USD 3. Page audit: about USD 0.05 per page; a page that fails to load is not charged.

## Reading the results

AI answers change between runs, so treat one week as a snapshot and look for a trend over several weeks. Audit scores are rubric judgments by a model; a difference under 5 points is noise.

## What was tested

Each workflow passed n8n's validator and a test run with sample data in place of the Apify, Google Sheets and Gmail calls. The audit form's report page was opened and read. A live run with real accounts was not done, so check the first real run yourself.
