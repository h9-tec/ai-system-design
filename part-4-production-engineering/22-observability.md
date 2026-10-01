# 22. Observability

*Part IV. Production Engineering · [Reading list](../README.md)*

## Standardize on OpenTelemetry GenAI conventions

The OTel GenAI semantic conventions define a vendor-neutral schema
(gen_ai.* attributes; inference, tool-execution, and agent spans) covering
model, token usage, finish reasons, tool calls, and retrieval sources, and
are supported across major platforms (Langfuse, Arize Phoenix, LangSmith,
Helicone). Adopting them keeps your telemetry portable instead of locked to
one vendor. The conventions are still experimental and not yet stable:
attribute names and span shapes can change between releases, so pin the
convention version you emit.

## Propagate identifiers end to end

```text
request_id · trace_id · conversation_id · user_id · tenant_id
prompt_version · model_version · retrieval_index_version · workflow_version
```

## Monitor four layers, plus unit economics

| Layer | Example metrics |
|---|---|
| Availability | Error rate, provider failures, circuit-breaker state |
| Performance | P50/P95/P99 latency, time to first token, queue depth |
| Cost | Tokens, tool calls, cache hit rate, **cost per successful task** |
| Quality | Retrieval precision, grounded-answer rate, task success, escalation rate, feedback |

Never rely on one end-to-end metric; a stable overall success rate can hide
a retrieval regression compensated by model behavior. Store full sampled
transcripts (with PII controls) so non-deterministic failures can be
replayed and diagnosed; monitor agent decision patterns even where privacy
prevents content inspection.

---

**Prev:** [21. Evaluation Strategy](21-evaluation-strategy.md) · [Reading list](../README.md) · **Next:** [23. Deployment and Change Management](23-deployment-and-change-management.md)
