# Advanced Capabilities

Powerful, expensive, and frequently added too early. Read the "when not to" for each before adopting.

## Contents

- [Graph Engineering](#graph-engineering) ★★☆☆☆
- [Multi-Agent Systems](#multi-agent-systems) ★★☆☆☆
- [Prompt Optimization](#prompt-optimization) ★★☆☆☆
- [Synthetic Data](#synthetic-data) ★★☆☆☆
- [Fine-tuning](#fine-tuning) ★☆☆☆☆
- [Distillation](#distillation) ★☆☆☆☆

---

## Graph Engineering

**What it is:** Modeling the agent's flow as an explicit graph once it's no longer a simple loop, i.e. when it has branches, parallelism, backtracking, multiple roles, or conditional execution.

```
              ┌→ Sales Agent ─────┐
User → Router ├→ Ads Agent ───────┼→ Summary Agent
              └→ Inventory Agent ─┘
```

**Consider only if at least one applies:**
- Several clearly independent steps
- Lots of if/else logic
- Multiple agents
- Parallel tasks
- Tasks must be resumable
- Long-running execution

**When not to:** Simple agents. Don't start with a graph.

**Rule:** If a loop can solve it, don't use a graph yet. Many "graph" needs (parallel tool calls, resumability) are already handled by the SDK or a durable workflow runtime without an explicit graph framework.

---

## Multi-Agent Systems

**What it is:** Several agents dividing the work.

```
Manager Agent
├─ Research Agent
├─ Sales Agent
├─ Finance Agent
└─ Support Agent
```

**Good fit:** Roles that differ sharply, for example Developer Agent (writes code), Reviewer Agent (reviews), Security Agent (security).

**Justified only when agents differ in:**
- Permissions
- Prompts (completely different)
- Context (completely different)
- Models
- Expertise (very large gap)
- Accountability boundaries

**When not to:** Splitting agents to look sophisticated. Often `1 agent + 10 tools` beats `10 agents`. Every hand-off loses context and adds latency, cost, and failure points.

**Cheaper alternatives first:** a single agent with more tools; a subagent/agent-as-tool call for one isolated subtask (keeps the main context clean without a full multi-agent architecture).

---

## Prompt Optimization

**What it is:** Improving prompts using data, not by editing on intuition.

```
Prompt A, Prompt B, Prompt C
→ Eval dataset
→ Compare results
```

**Metrics:** success rate, accuracy, cost, latency. A model can also propose prompt improvements automatically, scored against the eval.

**When needed:** After the agent runs stably and you have an eval set. Without an eval, "optimization" is guessing.

---

## Synthetic Data

**What it is:** Using AI to generate training or test data.

Real data: 1,000 support conversations. Generated: 50,000 simulated questions covering refunds, fraud, shipping, malicious users, extreme inputs, edge cases.

**Uses:** evaluation, fine-tuning, testing, classification.

**Guidance:** Seed generation from real examples and spot-check outputs. Synthetic sets drift toward what the generator finds easy. Best used to fill coverage gaps (adversarial and edge cases) around a real core.

---

## Fine-tuning

**What it is:** Teaching a model a stable behavior pattern.

**Good fit:**
- Fixed output formats
- A specific tone
- Specialized classification
- Industry-specific behavior patterns
- Large volumes of repetitive tasks

**When not to:** Storing dynamic knowledge.

| Wrong | Right |
|-------|-------|
| Fine-tune product prices into the model | Prices come from a database/tool |
| | Fine-tune how to respond to customers |

**When needed:** After the agent is running and the eval shows a behavior gap that prompting can't close. Most agents never need it early on.

---

## Distillation

**What it is:** Using a strong model to produce high-quality outputs, then training a cheaper model to imitate it.

| | Strong model | Distilled small model |
|---|---|---|
| Success rate | 97% | 93% |
| Cost per call | $0.10 | $0.005 |

At a million calls a day, this is worth a lot.

**When needed:** Large-scale, high-frequency, narrow tasks. Small projects shouldn't consider it.
