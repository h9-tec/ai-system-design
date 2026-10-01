# C. Anti-Patterns

*Appendices · [Reading list](../README.md)*

- [Fine-tuning](../part-2-core-techniques/10-fine-tuning.md) before prompting + retrieval baselines are exhausted
- Choosing models from leaderboards instead of your own [evals](../part-4-production-engineering/21-evaluation-strategy.md)
- [Self-hosting](../part-1-decisions/02-api-vs-self-hosting.md) for prestige while volume says the API is cheaper
- A single unrestricted agent with broad tool permissions
- [Multi-agent](../part-3-agentic-systems/15-multi-agent-systems.md) by default (pay 15x tokens without a parallelizable task)
- No step, token, time, or cost budget; no per-run circuit breaker
- Framework abstraction you cannot debug; prompts you do not own
- Vector search without ACL filtering; semantic cache shared across tenants
- One universal chunk size for every document type
- Treating a high cosine score as proof of correctness
- Indexing documents your parser mangled, then tuning retrieval to compensate
- Sending full conversation history and all retrieved chunks every turn
- Using model output as direct authorization for external actions
- Treating the system prompt as a security control
- Relying on a single injection classifier instead of containment
- Importing third-party [MCP](../part-3-agentic-systems/13-mcp.md) servers/tools without review or version pinning
- Training on unverified [synthetic data](../part-5-data-engineering/28-synthetic-data.md), or letting eval data leak into training
- Scaling [annotation](../part-5-data-engineering/27-annotation-operations.md) before inter-annotator agreement is measured
- Evaluating only hand-picked happy-path prompts, only offline
- Shipping prompt/model/index changes without regression testing
- Measuring latency while ignoring task success, grounding, and cost per success
- Hiding abstention: forcing an answer where "I don't know + escalate" is correct

---

**Prev:** [B. Production Readiness Checklist](b-production-readiness-checklist.md) · [Reading list](../README.md) · **Next:** [D. Metrics Quick Reference](d-metrics-quick-reference.md)
