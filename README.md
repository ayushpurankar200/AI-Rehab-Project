# ContextLens — AI Memory Strategy Demo

An interactive single-page demo built for an AI in Rehabilitation Engineering course project. It visualizes and compares five techniques for managing LLM context windows, showing in real time how each strategy affects token usage, cost, and memory recall.

## What It Demonstrates

Large language models have a finite context window. As conversations grow, naively sending the full history on every request becomes expensive and eventually impossible. This app demonstrates five strategies that solve this problem:

| # | Strategy | How It Works |
|---|---|---|
| ① | **Naive Baseline** | Sends the entire conversation history every request — cost grows without bound |
| ② | **Sliding Window** | Keeps only the N most recent messages verbatim; compresses older messages into summaries |
| ③ | **Entity Extraction** | Extracts structured facts (budget, plans, places) after each turn and injects them as a compact key-value block |
| ④ | **RAG** | Archives all messages and retrieves only the 2–3 most relevant ones per query using keyword similarity |
| ⑤ | **Hierarchical Summarization** | Creates fine-grained L1 summaries per turn, then compresses groups of L1s into L2 meta-summaries |

## Features

- **Live token counters** — tracks This Request, Session Total, Naive Would Cost, and Tokens Saved
- **Memory panel** — shows exactly what context is being sent to the model for each strategy
- **Educational event bubbles** — appear in the chat thread after each turn explaining what happened behind the scenes
- **Demo mode** — fully scripted 8-turn Switzerland trip-planning scenario, no API key required
- **Live mode** — connect your own Anthropic API key to run real conversations

## Usage

Just open `index.html` in any browser — no build step, no dependencies, no server required.

To use Live Mode, click **Settings** and enter your [Anthropic API key](https://console.anthropic.com/).

To explore the demo without a key, enable **Demo Mode** in Settings and press Enter to step through the scenario. Switch strategies mid-demo to compare how each one handles the same conversation differently.

## Tech

- Vanilla HTML/CSS/JS — single file, zero frameworks
- Claude API (`claude-sonnet-4-5`) via direct browser fetch
- Token estimation: `chars / 4` for naive baseline; actual `usage` field from API responses in live mode
