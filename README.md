# n8n Automations

Workflow automation projects built with self-hosted [n8n](https://n8n.io) — covering API integration, data transformation, webhooks, and AI-powered workflows.

Each folder is a standalone project with its own README, an importable `workflow.json`, and notes on the concepts it demonstrates.

## Projects

| # | Project | Concepts | Status |
|---|---|---|---|
| 01 | [Daily Tech Digest](./01-daily-tech-digest) | Schedule triggers, HTTP requests, item model, Code node | ✅ |
| 02 | [Webhook Lead Logger](./02-webhook-sheets-logger) | Webhooks, OAuth credentials, data mapping, deferred responses | ✅ |
| 03 | Gmail Auto-Triage | Conditional logic, branching, Merge | 🔜 |
| 04 | Paginated API Scraper | Loops, pagination, rate limiting, deduplication | 🔜 |
| 05 | AI Email Summarizer | LLM nodes, structured output parsing | 🔜 |
| 06 | AI Agent with Tools | Agent architecture, tool calling, memory | 🔜 |
| 07 | RAG Document Chatbot | Embeddings, vector stores, retrieval | 🔜 |
| 08 | Production Hardening | Error workflows, retries, sub-workflows, deployment | 🔜 |

## Running these workflows

You'll need n8n running locally:

```bash
npm install -g n8n
n8n start
```

Then open `http://localhost:5678`, and in any workflow use the ⋯ menu → **Import from File** to load a `workflow.json`.

Requires Node.js 20.19–24.x.

## A note on credentials

Exported workflow JSON references credentials by name only — no secrets, tokens, or keys are stored in these files. After importing a workflow that uses an external service, you'll need to connect your own credentials in n8n.

## Tech

n8n · JavaScript · REST APIs · Webhooks · Google Workspace APIs · LLM APIs

---

**Urooj Fatima** — Full-Stack Developer (MERN), Karachi
[Portfolio](https://portfolio-lime-two-58.vercel.app/) · [LinkedIn](https://www.linkedin.com/in/urooj-fatima-588ba2296/) · [GitHub](https://github.com/UroojFatim)
