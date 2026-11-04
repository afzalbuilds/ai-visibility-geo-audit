# Prompt set design

The rank tracker and the citation monitor measure whatever prompts you send. A good prompt set is the difference between a number the user can act on and noise. Build it once per brand, show it to the user, and reuse it unchanged on every later run.

## 1. Collect the inputs

From the conversation, the user's site, or a quick question, establish:

- **Brand name** as buyers write it (`brandName`). If the name is a common word (Notion, Monday, Square, Slack), note that mention matching is plain text matching and plan to read `mention_context`.
- **Brand domain** (`brandDomain`), registrable domain only: `notion.so`, not `https://www.notion.so/product`.
- **Category** in buyer language (`industry`): "note-taking apps for teams", not "productivity SaaS". This string also drives the brand reach checker, so keep it stable.
- **Competitors**: 2 to 5 brands the user actually loses deals to, with their domains. If the user names none, propose the three most obvious ones and ask them to confirm; wrong competitors make `position` and `share_of_voice` meaningless.
- **Market and language**: where the buyers are. Prompts are written in the buyers' language.

## 2. Write 10 to 20 prompts across five intents

Spread the prompts so that each intent has at least two. Write them the way a person types into ChatGPT, as full questions, 8 to 20 words, no brand name unless the intent calls for it.

| Intent | Share of the set | Pattern | Example for a CRM |
|---|---|---|---|
| Category discovery | 30% | "What is the best <category> for <audience>?" | What is the best CRM for a five-person sales team? |
| Problem-led | 20% | "How do I <job to be done>?" with no product words | How do I stop losing track of follow-ups with leads? |
| Comparison | 20% | "<Competitor A> vs <Competitor B>: which should I pick?" and "alternatives to <competitor>" | What are good alternatives to Salesforce for startups? |
| Feature or use case | 20% | "<category> that <specific need>" | Which CRM has the best Gmail integration? |
| Brand-named | 10% | "Is <brand> good for <use case>?" | Is HubSpot worth it for a solo consultant? |

Rules:
- Never include the brand name in discovery, problem-led or feature prompts; a mention there is earned visibility, which is what the user wants to measure.
- Include at most two brand-named prompts; they test accuracy of what the engines say, not reach.
- Keep prompts specific. "Best software?" returns lists where no brand ranks meaningfully.
- Use the market's own spelling and units (en-GB vs en-US, "estate agent" vs "realtor").
- 10 prompts on 4 engines is the standard set (40 checks). Offer 20 prompts only when the user has a budget in mind; the cost table in actor-contracts.md scales linearly.

## 3. Show the set before running

Present the prompts as a numbered list grouped by intent, with the competitor list and the `trackingId` you will use (`<brand-slug>-<category-slug>`, for example `hubspot-small-business-crm`). Ask the user to edit or approve. Save the approved set in your answer so it can be copied into the next run.

## 4. Keep runs comparable

- Reuse the exact same `prompts`, `platforms`, `competitors`, `industry` and `trackingId` on every later run. The Actors store history per `trackingId`; changing a prompt breaks its history line.
- Add new prompts at the end rather than replacing old ones, and note the date they were added.
- Run on the same weekday and roughly the same time; engines update their indexes and models on their own schedules.
- A single run is a snapshot. Call a change a trend only after the same direction appears in two consecutive runs or the `position_change` is 2 or more.
- Compare rates per engine. Perplexity cites 5 to 10 sources per answer and ChatGPT often 1 to 3, so a citation rate of 40 percent means different things on each.

## 5. Reading the results

- **Visibility score** (rank tracker `SUMMARY.visibility_score`): percent of checks that mentioned the brand. Under 20 means invisible, 20 to 50 means present but losing, over 50 means established, in this prompt set.
- **Average position** 1 to 2 means the engines lead with the brand; 3 or more means it is listed after competitors.
- **Share of voice** is the brand's share of all tracked-brand mentions in the answer, so it only compares against the competitors sent.
- **Citation rate** (citation monitor `SUMMARY.brand_citation_rate`) is the percent of answers that linked to the brand's domain. Mentions without citations usually mean the engines know the brand from third-party pages, not from the brand's own site; that points to the GEO site audit.
- **Citation gap domains** are the sites the engines trust for this category instead of the brand's site. Getting mentioned or published on those is the fastest route to citations.
