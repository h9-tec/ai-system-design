# 24. Cost and Latency Engineering

*Part IV. Production Engineering · [Reading list](../README.md)*

- **Prompt caching first.** Structure prompts as stable prefix (system,
  tools, reference docs) + variable suffix. Providers discount cached
  tokens heavily: Anthropic documents up to 90% cost and up to 85% latency
  reduction on long cached prompts (cache reads about 10% of input price);
  OpenAI also applies a substantial automatic cached-input discount. This is
  often the single largest lever in [RAG](../part-2-core-techniques/08-rag-system-design.md) and agent systems.
- **Model routing / cascades**: default to a small model, escalate to a
  large one on low confidence or hard routes; use batch APIs (typically
  about 50% cheaper) for offline work.
- **Context discipline is cost discipline**: compaction, tool result
  clearing, and token-efficient tool responses directly cut spend.
- Stream tokens for perceived latency; parallelize independent tool calls;
  cap max output tokens per route.
- **Semantic caching**: fine for public FAQs; dangerous for personalized or
  authorized content. When the response varies by user, include the tenant,
  the auth scope, and the user principal in the cache key, or skip it. Tenant
  and scope alone can still let two users share an entry.
- Track spend per route, per tenant, per feature, and alert on anomalies
  (see [Unbounded Consumption](20-security.md)).

---

**Prev:** [23. Deployment and Change Management](23-deployment-and-change-management.md) · [Reading list](../README.md) · **Next:** [25. Inference and Serving (Self-Hosted)](25-inference-and-serving.md)
