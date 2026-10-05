# Knowledge Capabilities

Giving the agent access to documents and long-term memory.

## Contents

- [RAG](#rag) ★★★☆☆
- [RAG 2.0](#rag-20) ★★☆☆☆
- [Vector DB](#vector-db) ★★☆☆☆
- [Memory Layers](#memory-layers) ★★★☆☆

---

## RAG

**What it is:** Retrieval-Augmented Generation. Before answering, the model retrieves relevant material.

```
Question → Search → Relevant documents → LLM → Answer
```

**Good fit:**
- Company knowledge bases
- Product documentation
- Legal material
- FAQs
- Internal docs
- Large volumes of mostly static text

**Bad fit:** Live business data. Order status should come from a database/API call, not from RAG.

**When needed:** Depends on the project. Start with the simplest retrieval that works. Keyword search, SQL full-text, or an agent with a `search_docs` tool often beats a vector pipeline for small or well-structured corpora.

---

## RAG 2.0

Not a strict standard term. Usually means a more advanced retrieval pipeline:

```
User question
→ Query rewrite
→ Multi-source search
   ├─ Vector DB
   ├─ SQL
   ├─ Web
   ├─ Graph
   └─ Documents
→ Rerank
→ Is the material sufficient?
→ If not, search again
→ Generate
```

**Good fit:** Complex research agents.

**Guidance:** Ordinary products don't need this at the start. Note that an agent loop with good search tools already gives you "search again if not enough" for free; you may not need a dedicated pipeline.

---

## Vector DB

**What it is:** Stores embeddings and searches by semantic similarity.

User asks: "Did any customers say the product is too sweet?"
The system finds: "sweetness is a bit high", "tastes a little sweet", "wish it were less sweet". It works even without matching keywords.

**Common options:** pgvector, Qdrant, Pinecone, Weaviate, Milvus.

**When needed:** Semantic search over large amounts of unstructured text.

**Not needed:** Ordinary database queries. If the data is clearly structured (orders, inventory, prices, users), use SQL/API first.

**Guidance:** If you already run Postgres, pgvector avoids adding a new service. One of the four most over-engineered capabilities. Prove you need semantic search before adding infrastructure.

---

## Memory Layers

**What it is:** Letting the agent remember things across tasks. Split memory into layers:

| Layer | Holds | Example |
|-------|-------|---------|
| Working | Current task info | "Currently handling order #123" |
| Short-term | Recent conversation turns | "User just said: no refund, just reship" |
| Episodic | Past events | "This customer complained about shipping two months ago" |
| Semantic | Long-term facts and preferences | "User prefers short replies in English" |
| Procedural | How to do things | "Always verify order status before refunding" |

**When needed:** The agent serves the same user over a long period:
- Personal assistant
- CRM agent
- Long-term learning agent
- Customer support agent
- Project agent

**Not needed:** One-off tools.

**Guidance:**
- Working and short-term memory usually come free with the SDK's session/conversation handling.
- Procedural memory often belongs in instructions or skills, not a database.
- Retrieve memory selectively into context (see Context Engineering); don't inject everything every turn.
- Give users a way to see and correct what's remembered about them.
