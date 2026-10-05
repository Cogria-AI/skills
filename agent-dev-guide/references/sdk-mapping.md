# SDK Mapping

Which built-in feature implements each capability, so you use the SDK's version instead of building your own.

> **Last verified: 2026-10.** Agent SDKs change fast. Names below are pointers, not API docs. Before writing code, check the current official documentation (or the installed package's docs) for exact signatures.

## Choosing an SDK

| SDK | Pick it when | Docs |
|-----|--------------|------|
| **OpenAI Agents SDK** (Python / TS) | Building on OpenAI models; want handoffs, guardrails, sessions, tracing out of the box | https://openai.github.io/openai-agents-python/ |
| **Claude Agent SDK** (`claude-agent-sdk` / `@anthropic-ai/claude-agent-sdk`) | Building on Claude; want the Claude Code harness (file/shell/web tools, permissions, hooks, subagents, skills) as a library | https://code.claude.com/docs/en/agent-sdk/overview |
| **Claude API + tool runner / Managed Agents** | Want a lighter loop over your own tools, or an Anthropic-hosted harness and sandbox | https://platform.claude.com/docs |
| **AI SDK** (Vercel, TypeScript) | Provider-agnostic TS apps, streaming UI, Next.js; easy multi-model via AI Gateway | https://ai-sdk.dev/docs/agents |
| **LangGraph** | You genuinely need an explicit graph, checkpointed state, and interrupts (Level 4–5) | https://docs.langchain.com/oss/python/langgraph |
| **Raw model API** | Level 0–1, or you need full control and the loop is trivial | Provider docs |

Default: pick the SDK that matches your primary model provider. Reach for LangGraph only when question 4 (complex branching) is a clear yes.

## Capability → SDK feature

| Capability | OpenAI Agents SDK | Claude Agent SDK | AI SDK | LangGraph |
|------------|-------------------|------------------|--------|-----------|
| Agent loop | `Agent` + `Runner` | `query()` / client agent loop | `ToolLoopAgent`; `generateText`/`streamText` with tools | Graph with tool node |
| Loop limit | `max_turns` | `maxTurns` / `max_turns` option | `stopWhen` (defaults to 20 steps) | `recursion_limit` |
| Tools | Function tools, hosted tools | Built-in tools (Read/Write/Bash/WebSearch…) + custom tools | `tool()` | Tools bound to model |
| MCP | MCP server tool calling | MCP servers config | MCP client | MCP adapters |
| Guardrails | `@input_guardrail`, `@output_guardrail`, tool input/output guardrails | Permissions + hooks (e.g. pre-tool-use) | Tool approval policies, middleware | Custom nodes |
| Human-in-the-loop | Human-in-the-loop / tool approval | Permission mode + `canUseTool`-style callback | `toolApproval` on agent or call (AI SDK 7) | `interrupt()` + `Command(resume=…)` |
| Sessions / state | Sessions | Sessions (resume / fork) | Your own store; `WorkflowAgent` for durable runs | Checkpointers (`thread_id`) |
| Multi-agent | Handoffs, agents-as-tools | Subagents | Agents as tools; subagents | Multi-node graphs / supervisor |
| Tracing | Built-in tracing | Hooks + OpenTelemetry / your logger | Telemetry (OpenTelemetry) | LangSmith |
| Sandbox | Sandbox agents | Runs in your process/container; Managed Agents for hosted sandbox | Vercel Sandbox | — |

## Durable / long-running execution

If a run must survive restarts or wait days for approval, use a durable runtime rather than hand-rolled persistence: LangGraph checkpointers, Temporal, Vercel Workflow (`WorkflowAgent` from `@ai-sdk/workflow`), or the SDK's own session store backed by a real database.
