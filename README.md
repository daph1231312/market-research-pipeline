# AI Market Research Pipeline (Make.com)

An automation that turns one topic into a short market-research report. It reads what
people complain about in online communities, uses an LLM to pull out the real pain points,
checks how crowded each one already is, and writes a **build / investigate / skip**
recommendation into a Google Sheet.

`blueprint.json` is the exported Make.com scenario. You can import it into your own Make
account (see below). All personal values (API key, spreadsheet ID, connection IDs, e-mail)
have been replaced with placeholders.

## How it works

```
Webhook (topic)
   │
   ▼
Reddit posts (Arctic Shift archive API)
   │
   ▼
Text aggregator ── all posts joined into one text
   │
   ▼
LLM #1 (Groq): "top 5 genuine pain points" → JSON
   [{ pain_point, evidence (a quote), frequency }]
   │
   ▼
Iterator ── one pass per pain point
   ├─► GitHub search: how many repositories already exist?
   └─► Hacker News (Algolia) search: how many "Show HN" posts?
   │
   ▼
LLM #2 (Groq): score the pain point as open / competitive / saturated
   and recommend build / investigate / skip, with a two-sentence reason → JSON
   │
   ▼
Google Sheets: one row per pain point
   (niche · pain point · score · recommendation · reasoning)
```

## What you need

| Requirement | Notes |
|-------------|-------|
| A [Make.com](https://www.make.com) account | the free plan is enough to try it |
| A **Groq API key** | free at <https://console.groq.com>; put it in both HTTP modules that call Groq (`Authorization: Bearer YOUR_GROQ_API_KEY`). The blueprint's model is `openai/gpt-oss-120b`; use any current Groq chat model |
| A Google account and a spreadsheet | add the Google Sheets connection in Make, replace `YOUR_SPREADSHEET_ID`, and give the sheet the columns `niche`, `pain_point`, `score`, `recommendation`, `reasoning` |

The Reddit archive, GitHub and Hacker News requests need no key (GitHub's unauthenticated
search has a low rate limit).

## Import and run

1. In Make, create a new scenario, open the **⋯** menu → **Import Blueprint**, and choose `blueprint.json`.
2. Re-connect the Google Sheets module and pick your spreadsheet.
3. Put your Groq key in the two Groq HTTP modules.
4. Run once, send a request to the webhook URL Make gives you with a JSON body such as `{"niche": "solopreneurs"}`.

## Known limitations

- The Reddit step uses a fixed search (`query=solopreneurs`, `subreddit=SaaS`, 25 posts) in the
  exported scenario; it does not yet use the topic from the webhook. Change the URL in that module for other topics.
- The scoring is a simple signal (repository and post counts) judged by an LLM, not a validated
  market model. Treat it as a quick first pass, not an answer.
- Groq retires models from time to time. This blueprint uses `openai/gpt-oss-120b` (with `max_tokens` set to 2000,
  because reasoning models spend tokens thinking). If a request fails with `model_not_found`, pick a current model from
  <https://console.groq.com/docs/models> and change it in both Groq HTTP modules.
- Importing a blueprint does not carry over connections or the webhook; you recreate those in your account.
