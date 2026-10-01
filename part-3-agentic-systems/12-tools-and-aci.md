# 12. Tools and the Agent-Computer Interface (ACI)

*Part III. Agentic Systems · [Reading list](../README.md)*

Tool design deserves the same care as prompt design. Anthropic's guidance
from building tools for Claude:

- **Few, focused, non-overlapping tools** beat exhaustive API mirrors.
  Build a handful of thoughtful tools for high-impact workflows that match
  your [evals](../part-4-production-engineering/21-evaluation-strategy.md), then expand. Every exposed tool definition costs tokens and
  attention.
- **Namespace related tools** with consistent prefixes (asana_search,
  asana_create) so the model picks the right one among overlapping options.
- **Consolidate multi-step operations** into single tools where the steps
  always go together (schedule_event that also checks availability, not
  three chained primitives).
- **Return token-efficient, high-signal responses.** A search_contacts tool
  beats list_all_contacts; do not make the agent brute-force through
  irrelevant output. Support concise vs detailed response formats, paginate or
  truncate large results with a clear signal when output was cut, and return
  semantic names rather than opaque IDs.
- **Make errors instructive.** A good error tells the model how to correct
  the call, then gets compacted into context (not a raw stack trace).
- **Poka-yoke the arguments.** Design parameters so misuse is hard (require
  absolute paths, enums instead of free strings, explicit idempotency_key).
- Iterate on tool descriptions using real transcripts; test tools with the
  agent in the loop, not just unit tests.

## Scaling to many tools

- **Expose tools as code APIs and let the agent write code to call them**
  ("code mode"): the model imports only the definitions it needs and passes
  data between calls in code, keeping large catalogs and intermediate results
  out of context for order-of-magnitude token savings.
- **Load tool definitions on demand.** Let the agent search a catalog and pull
  in only the schemas a task needs instead of putting every definition in the
  prompt.
- **Design for parallel tool calls.** The model may invoke several tools in one
  turn, so keep concurrently-run tools independent and idempotent.

## Tool contract

```json
{
  "name": "create_support_ticket",
  "description": "Create a customer-support ticket after user confirmation.",
  "input_schema": {
    "type": "object",
    "required": ["customer_id", "summary", "priority", "idempotency_key"],
    "properties": {
      "customer_id": { "type": "string" },
      "summary": { "type": "string", "maxLength": 1000 },
      "priority": { "enum": ["low", "normal", "high"] },
      "idempotency_key": { "type": "string" }
    }
  }
}
```

## Tool rules

- Minimum required permissions per tool; separate read tools from write
  tools; rate each tool's risk (low/medium/high) and gate high-risk tools
  behind confirmation.
- Validate all model-produced parameters server-side against the schema AND
  against business rules; validation next to the side effect is more
  reliable than agent-level checks.
- Per-tool timeout, retry, and cost limits; log inputs, outputs, duration,
  status, error category.
- Never expose credentials, raw database access, or unrestricted shell.
  Sandbox code execution with an egress allowlist.
- Use [structured outputs](../part-2-core-techniques/06-structured-outputs.md) / constrained decoding for every tool call; still
  validate, because schema conformance is not semantic correctness.

---

**Prev:** [11. The Architecture Ladder: Workflows vs Agents](11-workflows-vs-agents.md) · [Reading list](../README.md) · **Next:** [13. MCP (Model Context Protocol)](13-mcp.md)
