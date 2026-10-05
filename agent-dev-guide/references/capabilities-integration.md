# Integration Capabilities

Connecting the agent to external systems and to multiple models.

## Contents

- [MCP](#mcp) ★★★★☆
- [Stateless MCP](#stateless-mcp) ★★★★☆
- [AI Gateway](#ai-gateway) ★★★☆☆

---

## MCP

**What it is:** Model Context Protocol, a standard interface protocol for AI tools.

```
Agent
 ↓
MCP
 ├─ Gmail
 ├─ Notion
 ├─ GitHub
 ├─ Database
 ├─ Shopify
 └─ Internal API
```

Your own business system can expose tools like:

```
get_orders
get_inventory
get_customer
create_coupon
refund_order
```

and any MCP-capable agent can call them the same way.

**Good fit when you:**
- Have multiple agents
- Have multiple systems
- Want standardized tools
- Plan to reuse tools across projects or clients

**Not necessary when:** You have two simple APIs used by one agent. Plain function tools are simpler.

---

## Stateless MCP

**What it is:** An MCP server where no call depends on state left by a previous call.

Not recommended:

```
select_store("ruffood")
get_orders()            // depends on the previous call
```

Recommended:

```
get_orders(store="ruffood", date="2026-09-07")
```

**Benefits:**
- Scales horizontally
- Serverless-friendly
- Load-balances cleanly
- More reliable with multiple agents calling concurrently
- Simple failure recovery

**When needed:** Strongly recommended for any production MCP server. The same rule applies to plain function tools.

---

## AI Gateway

**What it is:** A single layer that manages access to multiple models.

```
App
 ↓
AI Gateway
 ├─ OpenAI
 ├─ Anthropic
 ├─ Gemini
 ├─ DeepSeek
 └─ Local model
```

**The gateway handles:** routing, fallback, rate limiting, logging, cost tracking, authentication, caching.

Examples of this idea: OpenRouter, Vercel AI Gateway, LiteLLM, Cloudflare AI Gateway.

**Recommended split:** If the core agent relies on one vendor's agent SDK, keep the agent's reasoning on that vendor's official API (SDK features like tracing, hosted tools, and sessions often assume it). Route auxiliary tasks through a gateway to cheaper models:

```
Agent reasoning  → official provider API
Classification   → cheap model via gateway
Translation      → cheap model via gateway
Summarization    → cheap model via gateway
```

**When needed:** Depends on scale. Worth it once you use several models, need provider fallback, or want centralized cost tracking.
