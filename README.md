# n8n-automation-portfolio

A collection of n8n workflows built while learning AI agent development — covering triggers, AI Agents with tools, memory, multi-agent coordination, human-in-the-loop patterns, batch processing, RAG, error handling, and retry/fallback resilience.

Built as a practice track alongside a Udemy course on RAG agents and n8n automation, as preparation for AI Agent Developer roles.

## Tech stack

- **n8n** (self-hosted, Community Edition)
- **Google Gemini API** (Chat Model and embeddings — free tier, no credit card required)
- **Pinecone** (vector database, free tier)
- **wttr.in** — free weather API (no key required)
- **Open-Meteo** — free geocoding + weather API (no key required)

## Workflows

| # | Name | What it demonstrates |
|---|------|----------------------|
| 01 | Webhook Age Calculator | Webhook trigger + basic data processing |
| 02a | AI Agent Weather (Single Tool) | AI Agent calling one external API as a Tool |
| 02b | AI Agent Weather (Two Tools) | Chaining two independent APIs (geocoding → weather) via Agent tool calls |
| 03 | AI Agent Memory | Session-based memory (Simple Memory node) across multiple messages |
| 04 | Multi-Agent Coordinator (Weather + Age) | A coordinator agent delegating to two specialist sub-agents |
| 05 | Sequential Agent Human-in-the-Loop | Lead capture flow with explicit user confirmation before saving data |
| 06 | Batch Product Description Generator | Code node + Basic LLM Chain to generate marketing copy for multiple items in one run |
| 07 | Resilient Weather Agent (Error Handling) | HTTP-level error handling — branches on the HTTP Request node's error output (500 response) instead of relying on the LLM to detect and report failures |
| 08a | FODMAP Agent | RAG chat agent answering FODMAP-level questions via Pinecone retrieval, grounded to avoid hallucination |
| 08b | FODMAP Ingestion | Ingests a FODMAP knowledge base from Google Drive into Pinecone using markdown-aware chunking and Gemini embeddings |
| 09 | Weather Agent – Retry & Fallback | Primary agent (wttr.in) with Retry On Fail, error-output routing and a sentinel check that hands over to a fallback agent (Open-Meteo) when the primary fails |
| 10 | Modular Weather Agent (Sub-workflow) | Splitting API logic into a reusable sub-workflow called via the Call n8n Workflow Tool node |

Each workflow's exported JSON is in [`workflows/`](./workflows).

### Workflow 10 details

**Files:** [`10-weather-tool-(sub).json`](./workflows/10-weather-tool-(sub).json), [`10-weather-agent-(main).json`](./workflows/10-weather-agent-(main).json)

Two separate workflows: `Weather Tool (Sub)` (an `Execute Workflow Trigger` → HTTP Request to wttr.in → Edit Fields, returning just `temp` and `description`) and `Weather Agent (Main)` (Chat Trigger → AI Agent → Gemini, with the sub-workflow attached as its only Tool via `Call n8n Workflow Tool`). The main workflow contains no HTTP Request node at all — every API call lives in the sub-workflow.

Tested: a normal weather question answers correctly, an unrelated question ("What is 2+2?") does not trigger the tool, and an invalid city name is correctly reported as not found rather than hallucinated.

## Key learnings

- **Don't rely on the LLM to detect failures it wasn't designed to detect.** In task 07, an AI Agent correctly recognized an invalid city name and replied sensibly — but that was luck, not a guarantee. Moving the error handling to the HTTP Request node's error output makes the behavior deterministic instead of dependent on the model's judgment call.
- **HTTP-level vs. data-level failure.** Not every failed lookup comes back as a bad status code — some APIs return `200 OK` with an empty or missing field instead. Always check which kind of failure you're dealing with before deciding where to branch.
- **Explicit tool descriptions matter.** An AI Agent with tools attached won't necessarily call them without being told when and why — a clear System Prompt naming each tool and its purpose makes tool use reliable rather than optional.
- **Session memory needs an explicit Memory node.** Without one, each message to an AI Agent is stateless by default.
- **An LLM can swallow a tool failure.** In task 09, when the wttr.in tool failed, the agent simply apologized in plain text and finished "successfully", so the error output never fired and the fallback never started. Fix: the System Message tells the agent to return an exact sentinel string (`TOOL_FAILED`), and an IF node routes that value to the fallback agent.
- **Cover both failure paths.** Node-level failures (e.g. model errors) go through *Retry On Fail* + *On Error → Continue (using error output)*; failures the LLM hides go through the sentinel check. Both paths lead to the same fallback agent.
- **Rate limits are a failure mode too.** The Gemini free tier allows 5 requests/minute per model, and one chat message can trigger many calls (retries, tool calls). Running the two agents on different models gives each its own quota.
- **A fallback agent needs an explicit prompt.** Data arriving through an IF node does not carry the original `chatInput`, so the fallback agent's prompt is defined explicitly from the Chat Trigger.
- **Test every path, not just the happy one.** Normal run, tool failure and model failure each exercise a different branch. A small trap found along the way: in a Fixed-mode IF value, quotation marks become part of the compared string.
- **Sub-workflows must be published/active.** A workflow called via `Call n8n Workflow Tool` fails with "Workflow is not active and cannot be executed" until it's explicitly published, even though it's only ever triggered by another workflow, never directly.

## Setup

```bash
git clone https://github.com/lwa4tester-bot/n8n-automation-portfolio.git
```

Import any workflow JSON from `workflows/` into a local n8n instance (`npx n8n` or Docker). Each workflow needs its own credentials configured (Google Gemini API key at minimum; Pinecone for the RAG workflows). No other secrets are required since the external APIs used (wttr.in, Open-Meteo) are key-free.
