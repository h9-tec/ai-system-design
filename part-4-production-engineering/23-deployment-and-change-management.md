# 23. Deployment and Change Management

*Part IV. Production Engineering · [Reading list](../README.md)*

- **Prompts are versioned artifacts**: reviewed, tested, and rolled back
  like code. Same for tool schemas and index configs.
- **Shadow, then canary, then progressive rollout** for model, prompt, and
  index changes.
- **Rainbow deployments for stateful agents**: long-running agents break if
  you swap code mid-flight. Shift traffic gradually to the new version and
  keep old versions alive until their in-flight runs finish (the pattern
  Anthropic uses for its research agents).
- Pin model versions; evaluate provider "upgrades" like any other change.
- Index and [embedding](../part-2-core-techniques/09-embeddings-and-vectors.md) migrations: dual-write, backfill, verify retrieval
  metrics, cut over, keep rollback.
- [Fine-tuned](../part-2-core-techniques/10-fine-tuning.md) checkpoints follow the same lifecycle: [eval](21-evaluation-strategy.md)-gated promotion,
  versioned artifacts, rollback.

---

**Prev:** [22. Observability](22-observability.md) · [Reading list](../README.md) · **Next:** [24. Cost and Latency Engineering](24-cost-and-latency.md)
