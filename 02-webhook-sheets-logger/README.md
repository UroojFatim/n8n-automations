# 02 — Webhook Lead Logger

An HTTP endpoint that accepts POST requests, normalizes the payload, appends a row to Google Sheets, and returns a JSON response to the caller.

This is the pattern behind contact forms, payment webhooks, CRM integrations, and any "system A notifies system B" flow — the receiving half of an integration rather than the polling half.

## Workflow

![Webhook Lead Logger workflow in n8n](./screenshot.png)

```
Webhook  →  Normalize Lead  →  Google Sheets  →  Respond to Webhook
 (POST)        (Set)            (Append Row)          (JSON)
```

## What it does

1. Listens for `POST /webhook/lead-capture`
2. Extracts fields from the request body, applying defaults for optional ones
3. Stamps an ISO timestamp onto the record
4. Appends the record as a row in a Google Sheet
5. Responds `{ "status": "ok", "receivedAt": "..." }` to the caller

## Nodes used

| Node | Type | Purpose |
|---|---|---|
| Webhook | Webhook | Exposes a POST endpoint; response deferred to the Respond node |
| Normalize Lead | Set / Edit Fields | Maps `$json.body.*` into a flat record and adds `receivedAt` |
| Google Sheets | Google Sheets | Appends one row, auto-mapping fields to column headers |
| Respond to Webhook | Respond to Webhook | Returns a JSON acknowledgement |

## Concepts demonstrated

**Receiving vs. calling.** Project 01 called an external API on a schedule. This one is the inverse — n8n is the server, and an external system initiates. Most real integration work lives on this side.

**Request shape.** Incoming data is not at the root of the item. The body is at `$json.body`, headers at `$json.headers`, query parameters at `$json.query`. Reading `$json.name` instead of `$json.body.name` silently yields empty values.

**Test vs. production URLs.** n8n exposes two paths for every webhook: `/webhook-test/<path>`, which accepts a single request and only while the editor is listening, and `/webhook/<path>`, which is live whenever the workflow is published. They are different endpoints, not different modes of one endpoint.

**Deferred responses.** Setting the Webhook node's *Respond* option to `Using 'Respond to Webhook' Node` holds the HTTP connection open until that node runs. This lets the response include data produced mid-workflow — but it also means the request fails if the workflow errors before reaching it.

**Serializing safely.** The response body is built with `JSON.stringify()` rather than hand-written JSON. Concatenating values into a JSON string breaks as soon as one of them contains a tab, newline, or quote; `JSON.stringify` escapes them correctly.

**Cross-node references.** By the time execution reaches the Respond node, `$json` holds the Google Sheets API response. The original record is retrieved with `$('Normalize Lead').first().json`.

## Setup

Requires a Google Cloud project with the **Google Sheets API** and **Google Drive API** enabled, an OAuth 2.0 Web application client whose authorized redirect URI matches n8n's callback URL, and your own account listed under test users while the consent screen is in Testing mode.

1. Create a spreadsheet with these headers in row 1:

   | receivedAt | name | email | company | message | source |
   |---|---|---|---|---|---|

2. Import `workflow.json` into n8n
3. Open the **Google Sheets** node, connect your credential, and select your own document and sheet
4. Publish the workflow

## Usage

```bash
curl -X POST http://localhost:5678/webhook/lead-capture \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Ali Raza",
    "email": "ali@example.com",
    "company": "Acme Pvt Ltd",
    "message": "Interested in a demo",
    "source": "website-form"
  }'
```

Response:

```json
{ "status": "ok", "receivedAt": "2026-08-11T19:44:01.776+05:00" }
```

## Debugging notes

Three failures worth recording, since each produced a misleading symptom:

**Wrong operation.** The Sheets node defaulted to *Get Rows* rather than *Append Row*. It succeeded, returned zero items, and the downstream node simply never ran — no error, just a connection with no item count on it. Item counts on connections are the fastest way to spot where a pipeline goes empty.

**Fixed vs. Expression mode.** An expression written into a field left in *Fixed* mode is returned verbatim as text. The syntax looks correct and nothing is flagged; the value is just never evaluated.

**An invisible tab character.** A stray tab pasted ahead of an expression ended up inside the timestamp value. A raw tab inside a JSON string literal is invalid JSON, so the response failed to serialize while the field itself looked perfectly normal. Switching to `JSON.stringify` would have escaped it and avoided the failure entirely.

## Possible extensions

- Validate required fields with an IF node and return HTTP 400 for bad payloads
- Add header-based authentication so the endpoint isn't open to anyone
- Deduplicate on email before appending
- Send a Slack or email notification when a lead arrives
- Enrich the record with an LLM node that scores or categorizes the message
