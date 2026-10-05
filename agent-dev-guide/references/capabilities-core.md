# Core Capabilities

The foundation layer. Almost every agent beyond Level 0 needs most of these.

## Contents

- [Agent Harness](#agent-harness) ★★★★★
- [Context Engineering](#context-engineering) ★★★★★
- [Loop Engineering](#loop-engineering) ★★★★★
- [Agentic pattern](#agentic-pattern)
- [Tool Use](#tool-use) ★★★★★
- [Function Calling](#function-calling) ★★★★★
- [State Management](#state-management) ★★★★☆

---

## Agent Harness

**What it is:** The runtime shell around the model. The model is only the reasoning core; the harness gives it the environment it needs to actually do things:

```
Model
 ↓
Agent Harness
 ├─ Files
 ├─ Shell
 ├─ Browser
 ├─ Tools / MCP
 ├─ Database
 ├─ Permissions
 ├─ Session & state
 └─ Logs & tracing
```

**When needed:** Almost every agent that does real work. A pure text-in/text-out feature doesn't need a complex harness.

**Example (coding agent):** read code → edit files → run tests → read errors → edit again → submit.

**Guidance:** Don't build a harness from scratch if an SDK provides one. Agent SDKs (OpenAI Agents SDK, Claude Agent SDK, AI SDK agents) are harnesses. Your job is to configure tools, permissions, and sessions, not to reimplement the loop.

---

## Context Engineering

**What it is:** Deciding what the model sees in this turn. The question is not "how do I phrase the prompt?" but "which information belongs in context?"

A turn's context may contain:

```
System instructions
+ current user request
+ current task state
+ relevant memory
+ tool results
+ retrieved documents
+ business rules
```

**Wrong:** stuff everything in.
**Right:** include only what's relevant to the current task.

**Example:** User asks "Why did sales drop yesterday?"

Include: yesterday's sales, the day before's sales, ad spend, inventory changes, price changes, site incidents.

Don't include: three years of orders, every support chat, the full product catalog.

**When needed:** As soon as the agent is even slightly complex.

**Techniques:**
- Summarize or trim old turns instead of resending full history.
- Truncate or aggregate large tool outputs before they reach the model (5,000 SQL rows → summary + top N).
- Load instructions and reference docs on demand rather than up front.
- Keep stable content (system prompt, tool definitions) at the start so prompt caching works.

This is one of the most important parts of agent engineering.

---

## Loop Engineering

**What it is:** Designing how the agent repeats think → call tool → observe → decide → call tool → finish.

```
User → Agent → Tool Call → Tool Result → Agent → Tool Call → Tool Result → Final Answer
```

**Design decisions you must make:**
- Maximum iterations
- Termination condition (what counts as done)
- What happens when a tool fails
- Whether to retry, and how many times
- Whether to switch to a different tool
- When to ask the user
- When human confirmation is required

**Example:** "Find out why sales dropped yesterday."

```
check sales → dropped
→ check traffic → normal
→ check conversion → dropped
→ check inventory → out of stock
→ conclusion
```

**When needed:** Whenever the task can't be done in a single model call. This is the main difference between an agent and a chatbot.

**Guidance:** Always set a step limit. Unbounded loops are the most common source of runaway cost. Distinguish retryable errors (timeouts, rate limits) from non-retryable ones (bad input, permission denied) and return the latter to the model as information rather than retrying blindly.

---

## Agentic pattern

Not a feature, a system mode.

| Plain AI | Agentic AI |
|----------|------------|
| Question → answer | Goal → plan next step → call tool → observe → continue → reach goal |
| "Write me a refund email." | "Handle this refund." |

An agentic refund handler might: look up order → check shipping → decide fault → calculate refund amount → draft refund request → wait for approval → execute refund.

This is the definition of an agent. If the task is genuinely question → answer, you're at Level 0 and don't need the rest of this guide.

---

## Tool Use

**What it is:** The agent invoking external capabilities.

```
search_web()
send_email()
query_database()
create_image()
create_invoice()
```

**When needed:** Whenever the agent needs to do something. For real agents, tool use is the core.

**Tool design guidance:**
- Name tools by intent (`get_orders`, `refund_order`), not by implementation (`call_api_v2`).
- Write descriptions for the model: what it does, when to use it, what it returns.
- Keep parameters explicit and typed; make every call self-contained (see Stateless MCP in `capabilities-integration.md`).
- Return compact, structured results. Large raw payloads waste context.
- Return errors as readable messages the model can act on.
- Prefer a few well-designed tools over many overlapping ones.

---

## Function Calling

**What it is:** The model telling your program, in structured form, "call this function with these arguments."

User: "Will it rain in Maidenhead tomorrow?"
Model emits: `get_weather({"city": "Maidenhead"})`
Your program executes the API and returns the result.

**Relationship:**

```
Function Calling → Tool Use → Agent
```

Function calling is the main mechanism that implements tool use. Use structured/strict schemas where the SDK supports them so arguments always validate.

---

## State Management

**What it is:** Recording where a task is right now.

```
Task ID: 9823
status: waiting_for_user
completed_steps:
  - checked_order
  - checked_inventory
next_step: refund
```

With persisted state, the agent can continue after a restart, a timeout, a worker moving to another machine, or the user coming back the next day.

**When needed:** Long-running tasks, tasks that wait on humans, anything that can't finish in one request.

**Guidance:**
- State (where the task is) is different from memory (what the agent knows about the user). Keep them separate.
- Make steps idempotent so resuming doesn't double-execute side effects (e.g. refund twice).
- Use the SDK's sessions or a durable workflow runtime before building your own persistence.
