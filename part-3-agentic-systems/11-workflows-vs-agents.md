# 11. The Architecture Ladder: Workflows vs Agents

*Part III. Agentic Systems · [Reading list](../README.md)*

Move down this ladder only when the level above fails your [evals](../part-4-production-engineering/21-evaluation-strategy.md):

```text
Level 1: Single LLM call
         + retrieval, in-context examples, structured output
Level 2: Workflow (predefined code path orchestrating multiple calls)
Level 3: Agent (model directs its own loop and tool use)
Level 4: Multi-agent (orchestrator + parallel subagents)
```

## Workflow patterns (Anthropic, "Building Effective Agents")

| Pattern | Use when |
|---|---|
| Prompt chaining | Task decomposes into fixed sequential steps, each verifiable |
| Routing | Distinct input categories need different handling or models |
| Parallelization | Independent subtasks (sectioning) or diverse attempts (voting) |
| Orchestrator-workers | Subtasks cannot be predicted upfront; a lead model delegates |
| Evaluator-optimizer | Clear evaluation criteria exist and iteration adds value |

## When agents are justified

- The path cannot be hardcoded, but progress can be verified
- The task is valuable enough to pay for exploration (tokens, latency)
- You can sandbox execution and define stopping conditions

## Code as the orchestration layer

For agents juggling many tools, having the model write and run code that calls
those tools in a sandbox often beats emitting one tool call per turn: loops,
branching, and data passing stay in code instead of the context window, which
cuts tokens and round-trips. Reach for it when tools compose and execution can
be sandboxed.

## Framework caution

Frameworks (LangGraph, CrewAI, etc.) speed up the start but add abstraction
layers that obscure prompts and control flow, making debugging harder.
Start with direct API calls; if you adopt a framework, make sure you can
inspect every prompt and own the loop (retry, pause, terminate logic).

The 12-Factor Agents methodology captures the same discipline: own your prompts,
context window, and control flow; keep tool calls as structured outputs; unify
execution and business state; launch, pause, and resume over simple APIs;
compact errors back into context; and prefer small, focused agents over one
sprawling loop.

---

**Prev:** [10. Fine-Tuning and Model Adaptation](../part-2-core-techniques/10-fine-tuning.md) · [Reading list](../README.md) · **Next:** [12. Tools and the Agent-Computer Interface (ACI)](12-tools-and-aci.md)
