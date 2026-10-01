# 7. Context Engineering

*Part II. Core Techniques · [Reading list](../README.md)*

Context is a finite resource with diminishing returns. As token count grows,
recall and reasoning degrade ("context rot", the "lost in the middle"
effect). 12-Factor Agents practitioners report degradation well before the
window is full (past roughly 40% utilization). Bigger windows raise the
ceiling; they do not remove the need for curation.

Rules:

- Send the **smallest set of high-signal tokens** that makes the task
  solvable: system instructions, the user request, approved retrieved
  chunks with metadata, required output schema, abstention policy.
- Prefer **just-in-time retrieval** (the model fetches what it needs via
  lightweight tools) over pre-loading everything, and hybrid approaches
  where a small stable core is pre-loaded.
- Do not dump full conversation history or raw tool logs into the prompt.

For long-horizon tasks (Anthropic's context engineering guidance):

1. **Compaction**: summarize older turns into a compressed state and reset.
   Tune the compaction prompt on real traces: maximize recall first, then
   trim. The lightest-touch form is **tool result clearing** (drop old raw
   tool outputs, keep the record that the call happened). This context
   editing cut token use 84% in a 100-turn agent [eval](../part-4-production-engineering/21-evaluation-strategy.md) while letting the
   task finish.
2. **Structured note-taking (agentic memory)**: the agent writes durable
   notes to external storage (a NOTES.md, a memory tool) and reads them
   back. Store only what constrains future reasoning: decisions,
   preferences, failed approaches. Too much stored state becomes persistent
   context pollution.
3. **Sub-agents with clean contexts**: delegate focused subtasks to agents
   with fresh windows that return condensed summaries, instead of one agent
   dragging everything along.

---

**Prev:** [6. Structured Outputs](06-structured-outputs.md) · [Reading list](../README.md) · **Next:** [8. RAG System Design](08-rag-system-design.md)
