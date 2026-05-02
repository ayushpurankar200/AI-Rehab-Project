# ContextLens — AI Memory Strategy Demo

An interactive single-page demo built for an AI in Rehabilitation Engineering course project. It visualizes and compares five implemented context management strategies for LLMs — plus a reference panel covering nine advanced techniques used in production systems — showing in real time how each approach affects token usage, cost, and memory recall.

## What It Demonstrates

Large language models have a finite context window. As conversations grow, naively sending the full history on every request becomes expensive and eventually impossible. This app demonstrates five strategies that solve this problem:

| # | Strategy | How It Works |
|---|---|---|
| ① | **Naive Baseline** | Sends the entire conversation history every request — cost grows without bound |
| ② | **Sliding Window** | Keeps only the N most recent messages verbatim; compresses older messages into summaries |
| ③ | **Entity Extraction** | Extracts structured facts (budget, plans, places) after each turn and injects them as a compact key-value block |
| ④ | **Hierarchical Summarization** | Creates fine-grained L1 summaries per turn, then compresses groups of L1s into L2 meta-summaries |
| ⑤ | **RAG** | Archives all messages and retrieves only the most relevant ones per query using keyword similarity |

The **⊕ And Even More** tab covers nine additional techniques not implemented in the demo — each with a description, use case, and pros/cons:

Graph RAG · Prompt Caching · MemGPT/Letta · State Space Models (Mamba) · Neural Compression (LLMLingua) · Streaming LLM / Attention Sinks · Contextual Retrieval · Re-ranking · Episodic + Semantic Memory

## Features

- **Live token counters** — tracks This Request, Session Total, Naive Would Cost, and Tokens Saved
- **Honest overhead accounting** — side calls for compression, entity extraction, and summarization are counted in the session total in live mode, so strategy comparisons reflect true end-to-end cost
- **Memory panel** — shows exactly what context is being sent to the model for each strategy
- **Educational event bubbles** — appear in the chat thread after each turn explaining what happened behind the scenes
- **Turn 8 verdict bubbles** — at the memory recall test, each strategy gets an explicit summary of its strength and the specific failure case that would have broken it
- **Demo mode** — fully scripted 8-turn Switzerland trip-planning scenario, no API key required
- **Live mode** — connect your own Anthropic API key to run real conversations
- **And Even More tab** — reference panel of nine advanced production techniques with use cases and tradeoffs

## Usage

Just open `index.html` in any browser — no build step, no dependencies, no server required.

To use **Live Mode**, click **Settings** and enter your [Anthropic API key](https://console.anthropic.com/).

To explore without a key, enable **Demo Mode** in Settings and press Enter to step through the scenario. Switch strategies mid-demo to compare how each one handles the same conversation — the token counters, memory panel, and event bubbles all update to reflect the active strategy.

Click **⊕ And Even More** in the strategy bar for a reference grid of advanced techniques beyond the five implemented strategies.

## Token Accounting

| Stat | What it counts |
|---|---|
| This Request | Input + output tokens from the main conversation call this turn |
| Session Total | Running sum of all tokens — includes side calls (compression, extraction, summarization) |
| Naive Would Cost | What the session total would be with no strategy applied — grows quadratically |
| Tokens Saved | `max(0, Naive Would Cost − Session Total)` |

In demo mode, all token values are pre-calculated per strategy per turn. In live mode, the main call reports actual usage from the API; side calls also report actual usage and are added to the session total.

## Tech

- Vanilla HTML/CSS/JS — single file, zero frameworks
- Claude API (`claude-sonnet-4-20250514`) via direct browser fetch
- Token estimation: `chars / 4` for naive baseline; actual `usage` field from API responses in live mode
