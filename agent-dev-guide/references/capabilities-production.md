# Production Capabilities

What an agent needs before it touches real users, real money, or real data.

## Contents

- [Evaluation Framework](#evaluation-framework) ★★★★★
- [Guardrails](#guardrails) ★★★★★
- [Human-in-the-loop](#human-in-the-loop) ★★★★☆
- [Observability](#observability) ★★★★★
- [Cost Optimization](#cost-optimization) ★★★★☆

---

## Evaluation Framework

**What it is:** Systematically testing whether the agent is actually good. Many teams underestimate this.

**Metrics:**

```
Task Success Rate
Accuracy
Tool Selection Accuracy
Hallucination Rate
Latency
Cost
Retry Rate
Human Escalation Rate
```

**Example:** Prepare 1,000 real tasks, then compare:

| | Model A | Model B |
|---|---|---|
| Success rate | 89% | 94% |
| Cost per task | $0.04 | $0.08 |
| Latency | 4s | 7s |

Only with numbers like these can you make engineering decisions.

**When needed:** As soon as you plan to go to production.

**Guidance:**
- Start small: 20–50 real tasks with expected outcomes beat a large synthetic set you never run.
- Evaluate the trajectory too (did it call the right tools in a sane order?), not just the final answer.
- Run the eval on every prompt, model, or tool change. Treat it like a test suite.
- Grow the set from production failures.

---

## Guardrails

**What it is:** Limits on what the agent can do.

| Type | Purpose | Example |
|------|---------|---------|
| Input | Catch malicious or out-of-scope input | Detect prompt injection |
| Output | Stop harmful or leaking output | Block private data in replies |
| Tool | Gate tool calls by risk | See tiered example below |
| Permission | Restrict capabilities at the system level | Allow `SELECT`, deny `DROP TABLE` |

**Tiered tool guardrail example:**

```
Read order:             allow
Refund < £20:           auto-allow
Refund £20–£100:        rule-based check
Refund > £100:          human approval required
```

**When needed:** Whenever the agent can take real actions, especially involving:
- Money
- Email
- Deletion
- Modification
- Databases
- Shell
- Computer use

**Guidance:** Enforce permissions in code (DB user privileges, API scopes, sandboxing), not only in the prompt. A prompt instruction is a suggestion; a permission is a guarantee.

---

## Human-in-the-loop

**What it is:** Certain actions must be confirmed by a person.

```
Agent: I'm about to refund £249.

[Approve]  [Reject]
```

**Especially for:**
- Money
- Deletion
- Contracts
- Sending email
- Publishing content
- Modifying critical data
- Legal risk
- Large-value operations

**Guidance:**
- Approval means the agent pauses. That requires persisted state (see State Management) so the run can resume hours later.
- Show the human exactly what will happen (amount, recipient, diff), not the agent's reasoning.
- Most SDKs have built-in approval flows. Use them.

---

## Observability

**What it is:** Knowing what happened inside the agent.

```
Run #1029
Model:       <model id>
Steps:       7
Tool calls:  get_orders, get_inventory, send_email
Latency:     5.7s
Tokens:      19,232
Cost:        $0.041
Error:       inventory API timeout
```

Ideally with a full trace of every step, input, and output.

**When needed:** Required in production. Otherwise, when someone asks "Why did the agent refund that customer yesterday?", you have no answer.

**Guidance:** Turn on the SDK's built-in tracing first. Add a dedicated platform (Langfuse, LangSmith, Braintrust, Arize, OpenTelemetry-based tools, etc.) when you need retention, search, or eval integration. Redact PII in traces.

---

## Cost Optimization

**What it is:** Reducing the total cost of running the agent.

**Methods:**

| Method | How |
|--------|-----|
| Model tiering | Simple tasks → small model; complex reasoning → strong model |
| Reduce context | Don't resend full history every turn |
| Prompt caching | Cache repeated content (system prompt, tool definitions, documents) |
| Tool first | If the database can answer, don't make the model guess |
| Fewer loops | Prevent pointless iterations; set step limits |
| Control tool output | A 5,000-row SQL result doesn't need to go to the model in full |
| Small models for auxiliary tasks | Classification, extraction, format conversion, summarization |

**When needed:** Once traffic starts growing. Before that, optimize for correctness. Measure with the eval set so savings don't silently cost quality.
