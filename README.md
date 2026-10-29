# UniBot — AI Academic Assistant for the University of Dunaújváros

A retrieval-augmented (RAG) chatbot that answers student questions about the University of Dunaújváros using content scraped from the official university website. The front-end is a single static page; the backend is an n8n workflow that retrieves relevant website content from a vector database and generates an answer with an LLM.

> **Status: backend temporarily offline.** The chat page still loads at https://se-xy.github.io/UniBot/, but the n8n backend it talks to is being migrated to a self-hosted n8n instance (Docker), so questions will currently return a connection error. This README documents the architecture; a working demo and recorded walkthrough will be added when the migration is done.

<!-- Add a screenshot: docs/screenshot.png -->

## The problem

International students ask the same questions over and over — admissions, scholarships, Neptun, campus locations, contacts — and the answers are spread across many pages of the university website, some only in Hungarian. UniBot gives one place to ask in plain English (or Hungarian) and get an answer grounded in the official site.

## How it works

```
                 ┌─────────────── Ingestion ───────────────┐
 uniduna.hu ──► Firecrawl (crawl + clean) ──► chunk + embed ──► Pinecone (vector index)
                                                                    │
                 ┌─────────────── Query ──────────────────┐        │
 Student ──► UniBot web UI ──POST──► n8n webhook ──► retrieve top matches
                                          │                  from Pinecone
                                          ▼
                                    LLM answer grounded
                                    in retrieved content
                                          │
 Student ◄──────── JSON reply ◄───────────┘
```

1. **Ingestion:** Firecrawl crawls the university website and returns clean text. The text is split into chunks, embedded, and stored in a Pinecone index.
2. **Query:** the web UI sends the question to an n8n webhook with a session ID and the selected language.
3. **Retrieval + generation:** the n8n workflow retrieves the most relevant chunks from Pinecone and passes them to the LLM, which answers using only that context.
4. **Response:** the workflow returns `{ success, reply, sessionId }` and the UI renders it.

### Webhook contract

Request:

```json
{ "message": "How do I apply for the Stipendium Hungaricum?", "sessionId": "session-...", "studentName": "Student", "lang": "en" }
```

Response:

```json
{ "success": true, "reply": "...", "sessionId": "session-..." }
```

## Features

- Chat UI with typing indicator, connection status and error handling
- English / Hungarian interface toggle
- Light and dark theme
- Per-session conversation memory, with the last 5 sessions saved in the browser
- Quick links to key university resources (Stipendium Hungaricum, campus map, Neptun, contacts)
- No build step: one static HTML file, deployable on GitHub Pages

## Tech stack

| Layer | Tools |
|---|---|
| Front-end | HTML, Tailwind CSS, vanilla JavaScript |
| Orchestration | n8n (webhook-triggered workflow) |
| Scraping | Firecrawl |
| Vector store | Pinecone |
| LLM | OpenAI |
| Hosting | GitHub Pages (UI), n8n Cloud (backend) |

## Repository contents

| File | Purpose |
|---|---|
| `unibot-wired.html` | The chat application (UI + webhook client) |
| `index.html` | Redirects to the chat page for GitHub Pages |

## Run it yourself

1. Build an n8n workflow with a **Webhook** trigger (POST) that accepts the request above and returns the response above.
2. Point `WEBHOOK_URL` in `unibot-wired.html` at your webhook.
3. Open `unibot-wired.html` in a browser, or enable GitHub Pages on the repo.

## Limitations and next steps

- Answers are only as current as the last crawl; scheduled re-indexing is planned.
- The webhook is public; production use would add rate limiting and an origin check.
- In progress: moving the backend from n8n Cloud to self-hosted n8n (Docker) and restoring the live demo.
- Planned: source links in each answer, and publishing the n8n workflow export in this repo.

## Author

Built by Peter Robia — Computer Science student at the University of Dunaújváros, building n8n and Python automations.
