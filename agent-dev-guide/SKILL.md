---
name: agent-dev-guide
description: Right-size the architecture of an AI agent project before and while building it. Use when designing, scaffolding, planning, reviewing, or refactoring anything agent-shaped — tool-calling agents, agent loops, MCP servers, RAG pipelines, memory, multi-agent systems, guardrails, evals, tracing — on any SDK (OpenAI Agents SDK, Claude Agent SDK, AI SDK, LangGraph, or raw API). Also use when someone asks "do I need RAG / a vector DB / multi-agent / LangGraph / fine-tuning for this?", or when an agent design looks over-engineered. Not for single-shot prompt features with no tools, or for general app code unrelated to the agent.
---

# Agent Development Guide

A decision framework for building AI agents that are exactly as complex as the task requires, and no more.

## Core principle

Not every agent needs every capability. The most common failure is starting with Multi-Agent, Graph, RAG, Memory and a Vector DB, and ending up with a system far more complex than the problem.

Work in this order:

1. Identify the task type.
2. Decide which capabilities the task actually needs.
3. If you can do without a capability, leave it out.
4. If the SDK already provides it, use the SDK's version instead of building your own.

The test for every capability: **"If I leave this out, will the current task be done badly?"** If the answer is no, don't add it yet.

Four things are especially prone to over-engineering: **Multi-Agent, Graph, Vector DB, Fine-tuning.**

Most agent products that work well need only:

```
a good model
+ a good agent loop
+ a few well-designed tools
+ strong context engineering
+ a reliable eval
```

Get those right and you cover 80%+ of real agent requirements.

## Step 1: Answer the ten questions

Go through these before writing code. Each "yes" unlocks capabilities; each "no" means leave them out.

| # | Question | If yes, enable | Notes |
|---|----------|----------------|-------|
| 1 | Is it just chat / text in, text out? | Prompt + Model + Response | Stop here. No agent needed. |
| 2 | Does it call external systems? | Tool Use, Function Calling | Many tools, many systems, or reuse across agents → add MCP. |
| 3 | Does it run multi-step tasks? | Loop Engineering, State Management | |
| 4 | Does it have complex branching, parallelism, or resumption? | Graph | Otherwise no. A loop handles most cases. |
| 5 | Must it remember a user across sessions? | Memory, Context Engineering | |
| 6 | Must it search lots of documents? | RAG | Large corpus needing semantic search → Vector DB. |
| 7 | Are there truly distinct specialist roles? | Multi-Agent | Only with real boundaries (see below). Default: Single Agent + Tools. |
| 8 | Can it modify real data, spend money, or send things? | Guardrails, Permissions, Human-in-the-loop, Observability | Mandatory, not optional. |
| 9 | Will it run in production? | Evaluation, Observability, Error Handling, Tracing, Guardrails | Minimum set. |
| 10 | Is call volume high? | Cost Optimization, AI Gateway, small models, Prompt Caching, Distillation | Only after real traffic exists. |

Multi-Agent is justified only when agents differ in **permissions, prompts, context, models, expertise, or accountability boundaries**. "It looks more advanced" is not a reason. `1 agent + 10 tools` usually beats `10 agents`.

## Step 2: Place the project on the maturity ladder

| Level | Name | Shape | Needs |
|-------|------|-------|-------|
| 0 | LLM Feature | Prompt → Model → Output (translate, summarize, rewrite) | Prompt, Model API |
| 1 | Tool Agent | Agent → Tool (weather, DB lookup) | Function Calling, Tool Use, Context Engineering |
| 2 | Workflow Agent | Multi-step execution | Loop, State, Tool Use, Tracing |
| 3 | Production Agent | Runs live for real users | Evaluation, Guardrails, Observability, Error Handling, Permissions, Human Approval |
| 4 | Autonomous Agent | Completes complex tasks independently | Memory, Long-running State, Advanced Context, Dynamic Tool Selection, Recovery, Planning |
| 5 | Agent System | Multiple agents collaborating | Multi-Agent, Graph, Shared Memory, MCP, Agent Routing, Observability, Evaluation |

Build for the level the task needs today. Level 5 complexity is very high; reach it only when lower levels demonstrably fail.

## Step 3: Start from the default template

For a new agent, start with this minimum and add layers only when a question above says so:

```
Agent SDK (pick one, see references/sdk-mapping.md)
+ Agent
+ Tools / Function Calling
+ Context Engineering
+ Loop (with a max-step limit)
+ Tracing
+ Basic Guardrails
```

Add when needed:

| Trigger | Add |
|---------|-----|
| Connecting to external services | MCP |
| Long-running tasks | State Management |
| Remembering users | Memory |
| Large knowledge base | RAG |
| Going live | Evaluation, Observability, Permissions, Human Approval |
| Only if very complex | Graph, Multi-Agent |
| Only at scale | Gateway, Distillation, Fine-tuning, Synthetic Data, advanced Cost Optimization |

## Step 4: Write the design decision record

Before implementing, produce a short record and show it to the user (or put it in the project docs). This is the main output of the skill.

```markdown
## Agent Design Decision

**Task:** <one sentence: what the agent achieves>
**Level:** <0–5> — <why this level and not one higher>
**SDK / runtime:** <choice> — <why>

**Enabled capabilities**
- <capability>: <which question/trigger requires it>

**Deliberately excluded**
- <capability>: <why the task does not need it yet; what signal would change that>

**Risky actions and controls**
- <action, e.g. refund, send email, delete>: <guardrail / approval threshold>

**How we'll know it works**
- <eval set, key metrics, target numbers>
```

The "Deliberately excluded" section matters as much as the enabled list. It forces the question for each tempting capability and gives a clear signal for when to revisit.

## Rules that apply to every agent

- **Tool first.** If a database or API can answer, call it. Don't let the model guess, and don't use RAG for live data like order status.
- **Context is selected, not dumped.** Put only task-relevant information in context. See `references/capabilities-core.md`.
- **Every loop has an exit.** Max steps, a done condition, and a plan for tool failure.
- **Tools are stateless.** Pass all required parameters on every call; never rely on a previous call having set state.
- **Dynamic knowledge lives in tools, not weights.** Prices, inventory, and policies change; never fine-tune them in.
- **Real-world side effects need gates.** Money, email, deletion, publishing, schema changes, shell: tiered guardrails plus human approval above a threshold.
- **If you can't trace it, you can't ship it.** You must be able to answer "why did the agent do X yesterday?"
- **Measure before optimizing.** Prompt optimization, fine-tuning, and model swaps are decided by an eval set, not by feel.
- **Verify SDK APIs against current docs.** Agent SDKs change fast. Don't write code from memory.

## Reference files

Read the one relevant to the decision in front of you. Don't load all of them.

| File | Covers | Read when |
|------|--------|-----------|
| `references/capabilities-core.md` | Agent Harness, Context Engineering, Loop Engineering, Agentic pattern, Tool Use, Function Calling, State Management | Designing the basic agent |
| `references/capabilities-integration.md` | MCP, Stateless MCP, AI Gateway | Connecting external systems or multiple models |
| `references/capabilities-knowledge.md` | RAG, RAG 2.0, Vector DB, Memory Layers | Agent needs documents or long-term memory |
| `references/capabilities-production.md` | Evaluation, Guardrails, Human-in-the-loop, Observability, Cost Optimization | Preparing for production or real side effects |
| `references/capabilities-advanced.md` | Graph, Multi-Agent, Fine-tuning, Prompt Optimization, Synthetic Data, Distillation | Someone proposes one of these. Check the "when not to" first |
| `references/sdk-mapping.md` | Which SDK feature implements each capability | Choosing an SDK or wiring a capability |
