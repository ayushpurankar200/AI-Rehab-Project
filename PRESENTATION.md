# ContextLens — Presentation Speaking Notes

---

## 1. Opening / Hook

"Every time you send a message to an AI, it doesn't actually remember your previous messages the way a human does. It re-reads the entire conversation from scratch, every single time. That sounds fine for a short chat — but what happens when the conversation gets long? You hit a wall. The model runs out of space, and older information starts getting cut off. This project is about what happens at that wall, and the five main techniques engineers use to get around it."

---

## 2. The Core Problem — Context Windows

- Every LLM has a **context window** — a hard limit on how many tokens (pieces of text) it can process in one request.
- Claude's context window is **200,000 tokens**. GPT-4o is **128,000**. These sound large, but they fill up fast in real applications.
- The naive approach is to send the entire conversation history on every request. This works fine at first, but the cost grows **quadratically** — every new turn makes every future turn more expensive, because you're re-sending all previous turns each time.
- After 8 turns of a normal conversation, naive costs roughly **10,200 tokens**. After 50 turns, you're looking at hundreds of thousands. After 100 turns, you exceed the window entirely and the model starts forgetting the beginning.
- This isn't a theoretical problem. Any AI assistant meant to hold extended conversations — therapy bots, medical documentation tools, long-term coaching systems — runs into this.

---

## 3. The Five Strategies

### ① Naive Baseline
"This is the control group. No memory management at all — just dump the entire history into every request."
- Cost per turn grows linearly with conversation length
- Input tokens after 8 turns: **~2,450 per request**
- Session total: **~10,200 tokens**
- ✓ Perfect recall — nothing is ever lost
- ✗ Unsustainable — cost and context size both grow without bound

---

### ② Sliding Window Summarization
"Keep only the N most recent messages verbatim. When the window overflows, compress the oldest messages into a short summary using a separate API call, then discard the originals."

- Think of it like a conveyor belt — recent messages ride on top, older ones get squeezed into a summary block and fall off the end
- Compression fires at turns 4 and 7 in the demo — you can see the token cost drop after each compression
- Input tokens after 8 turns: **~480 per request**
- Session total: **~4,520 tokens** — **56% less than naive**
- ✓ Good recall for recent context; summaries preserve key facts
- ✗ Fine-grained details in compressed turns are lost; summary quality depends on the LLM doing the compression

---

### ③ Entity Extraction
"After every turn, run a second API call that extracts structured facts — budget numbers, place names, dates, plans — and stores them as typed key-value pairs. Instead of sending prose history, inject a compact fact block."

- Example: instead of sending a 300-token paragraph about budget, inject `[BUDGET] total: $4,000 | est_spend: $2,420–$3,120 | accommodation: $90–130/night`
- The fact block stays nearly the same size regardless of how many turns have passed
- Input tokens after 8 turns: **~280 per request**
- Session total: **~2,490 tokens** — **76% less than naive**
- ✓ Extremely token-efficient; structured facts are easy to query and update
- ✗ Only captures facts that fit a schema — nuance, tone, and reasoning are lost; works best for data-heavy conversations

---

### ④ RAG — Retrieval-Augmented Generation
"Archive every message permanently. On each new query, score the archive using keyword similarity to find the 2–3 most relevant past messages, and inject only those into the context."

- Nothing is ever deleted — everything goes into an archive
- The retrieval step finds what's *relevant to this specific question*, not just what's recent
- In the demo, when the user asks about the budget in turn 8, RAG retrieves the budget message from turn 2 — even though it's been 6 turns since it was mentioned
- Input tokens after 8 turns: **~240 per request**
- Session total: **~2,200 tokens** — **78% less than naive**
- ✓ Best for large archives; nothing is permanently forgotten; very token-efficient
- ✗ This demo uses simple keyword overlap — production RAG uses vector embeddings which require a separate embedding model; relevance scoring can miss things if the query wording differs from the archived text

---

### ⑤ Hierarchical Summarization
"Create a fine-grained L1 summary after every turn. After every 3 L1s, compress them into a higher-level L2 meta-summary. Send [L2s] + [recent L1s] + [live messages] instead of full history."

- Two-tier compression: L1 keeps turn-level detail, L2 gives a bird's-eye view of topics covered
- Analogous to how a textbook has chapter summaries and a book summary — different levels of detail at different granularities
- Input tokens after 8 turns: **~360 per request**
- Session total: **~3,360 tokens** — **67% less than naive**
- ✓ Better recall than flat summarization; the hierarchy preserves both detail and overview
- ✗ Most complex to implement; requires multiple compression passes; L2 quality degrades if L1s are poor

---

## 4. Results Comparison

After the same 8-turn conversation, here's how each strategy performed:

| Strategy | Session Total | Saved vs. Naive |
|---|---|---|
| Naive Baseline | 10,200 tokens | — |
| Sliding Window | 4,520 tokens | **56% saved** |
| Hierarchical | 3,360 tokens | **67% saved** |
| Entity Extraction | 2,490 tokens | **76% saved** |
| RAG | 2,200 tokens | **78% saved** |

"At 8 turns the savings are already significant. At 100 turns, the gap becomes enormous — naive becomes impossible (exceeds the context window), while strategies maintain a bounded cost per request."

---

## 5. When to Use Each Strategy

"There's no single best strategy — the right choice depends on the application."

| Use Case | Best Strategy | Why |
|---|---|---|
| Short to medium conversations | Sliding Window | Simple, easy to implement, good recall |
| Highly structured data (forms, records, stats) | Entity Extraction | Data fits naturally into key-value schema |
| Large knowledge base / document retrieval | RAG | Archive grows indefinitely; retrieval scales well |
| Long-form reasoning with layered context | Hierarchical | Preserves both macro and micro-level information |
| Prototype / baseline comparison | Naive | Always implement first to benchmark against |

---

## 6. Limitations of This Demo

"Being honest about what this demo doesn't do is part of good engineering."

- **RAG uses keyword overlap, not embeddings.** Real production RAG uses vector embeddings (e.g., OpenAI's `text-embedding-3-small`) to compute semantic similarity. Keyword matching breaks when the query uses different words than the source.
- **Demo mode uses scripted responses.** The 8-turn conversation is pre-written to guarantee meaningful results. A live API call can vary.
- **Summaries are only as good as the model doing the summarization.** If Claude summarizes poorly, the strategy loses important information. Prompt engineering for compression is its own sub-problem.
- **No evaluation metric.** A rigorous implementation would score recall — for example, can the system correctly answer questions about information that was compressed or archived? This demo shows token efficiency but not information fidelity.
- **Single conversation.** Real systems need to handle branching topics, contradictions, and updates to previously extracted facts.

---

## 7. Closing

"The takeaway is that context management is not optional for any serious long-form AI application. You either pay the exponentially growing naive cost, or you engineer around it. Each strategy trades something — simplicity, detail, recall, or computation — for efficiency. The best systems combine multiple strategies: RAG for long-term memory, sliding window for recent context, and entity extraction for structured facts, all layered together.

This project builds each one from scratch to make those trade-offs visible and measurable."

---

## Quick Reference — Token Math

- **1 token ≈ ¾ of a word, or ~4 characters**
- **This Request** = input tokens sent + output tokens received this turn
- **Session Total** = running sum of all actual tokens used
- **Naive Would Cost** = what Session Total would be if no strategy were applied (grows as a triangle number: turn 1 cost + turn 2 cost + turn 3 cost + ...)
- **Tokens Saved** = Naive Would Cost − Session Total (floored at 0)
