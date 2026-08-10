# 01 — Daily Tech Digest

A scheduled n8n workflow that pulls the Hacker News front page every morning, filters and ranks the stories, and formats them into a clean readable digest.

Built as the foundation project for learning n8n's execution model. No API keys or credentials required — it runs on a public endpoint.

## What it does

Every day at 9:00 AM the workflow:

1. Loads its own configuration (score threshold, story count)
2. Calls the Hacker News Algolia API for the current front page
3. Splits the returned array into individual n8n items
4. Filters out low-scoring stories, sorts by score, keeps the top N
5. Outputs a formatted digest string ready to be emailed or posted

## Workflow

![Daily Tech Digest workflow in n8n](./screenshot.png)

```
Every day at 9am  →  Config  →  Fetch Hacker News  →  Split Out Stories  →  Build Digest
 (Schedule Trigger)   (Set)      (HTTP Request)         (Split Out)            (Code)
```

Note the item count on the connection after **Split Out Stories** — one API response containing 20 stories becomes 20 individual items, and every downstream node runs once per item without an explicit loop.

## Nodes used

| Node | Type | Purpose |
|---|---|---|
| Every day at 9am | Schedule Trigger | Fires the workflow daily at 09:00 |
| Config | Set / Edit Fields | Holds `minPoints` and `topN` so thresholds aren't hardcoded |
| Fetch Hacker News | HTTP Request | `GET https://hn.algolia.com/api/v1/search?tags=front_page` |
| Split Out Stories | Split Out | Expands the `hits` array into one item per story |
| Build Digest | Code | Filters, sorts, formats — runs once over all items |

## Concepts demonstrated

**The item model.** n8n passes an array of items between nodes, not a single object. A node receiving 20 items executes 20 times automatically — no explicit loop. `Split Out` is what turns one item containing an array into many items.

**Cross-node references.** The Code node reads its thresholds with `$('Config').first().json`, rather than hardcoding them. This is how you keep configuration in one place and reference any earlier node in the workflow, not just the immediately preceding one.

**Code node modes.** `Run Once for All Items` receives the full set via `$input.all()` and is the correct mode for aggregation. `Run Once for Each Item` runs per item via `$json` and suits individual transformations.

## Running it

1. In n8n, open the workflow menu (⋯) → **Import from File**
2. Select `workflow.json`
3. Click **Test workflow** to run it immediately, or toggle **Active** to enable the daily schedule

No credentials to configure.

## Configuration

Edit the **Config** node to change behaviour:

| Field | Default | Effect |
|---|---|---|
| `minPoints` | 100 | Minimum score for a story to be included |
| `topN` | 10 | Maximum number of stories in the digest |

## Sample output

```
Tech Digest — 10/08/2026

1. Story title goes here
   412 points · 178 comments
   https://example.com/article

2. Another story
   295 points · 91 comments
   https://news.ycombinator.com/item?id=12345678
```

## Possible extensions

- Append a Gmail node to deliver the digest to your inbox
- Swap the Code node's formatting for HTML and send a styled email
- Add a Google Sheets node to archive each day's stories for trend analysis
- Feed the digest into an LLM node to generate a one-line summary per story
