# 17. Reference Architecture

*Part IV. Production Engineering · [Reading list](../README.md)*

```text
                        ┌───────────────────────────┐
                        │        Client / API       │
                        └─────────────┬─────────────┘
                                      │
                        ┌─────────────▼─────────────┐
                        │ Edge / API Gateway        │
                        │ Auth · Rate limits · ACL  │
                        │ Request IDs · Validation  │
                        │ Input guardrails          │
                        └─────────────┬─────────────┘
                                      │
                  ┌───────────────────▼───────────────────┐
                  │ Application / Workflow Orchestrator   │
                  │ State machine · Budgets · Retries     │
                  │ Checkpoints · Pause/Resume            │
                  │ Idempotency · Human handoff           │
                  └───────┬─────────────────────┬─────────┘
                          │                     │
              ┌───────────▼──────────┐  ┌──────▼──────────────┐
              │ Retrieval Service    │  │ Tool Execution Layer │
              │ ACL filter (pre)     │  │ Least privilege      │
              │ Hybrid BM25+vector   │  │ Server-side validate │
              │ Reranker             │  │ Idempotency keys     │
              └───────────┬──────────┘  └──────┬──────────────┘
                          │                     │
                    ┌─────▼─────────────────────▼─────┐
                    │ Context Builder                  │
                    │ Token budget · Citations         │
                    │ Compaction · Prompt version      │
                    └──────────────┬───────────────────┘
                                   │
                    ┌──────────────▼───────────────────┐
                    │ Model Gateway                     │
                    │ Routing · Fallback · Prompt cache │
                    │ Timeouts · Circuit breakers       │
                    └──────────────┬───────────────────┘
                                   │
                    ┌──────────────▼───────────────────┐
                    │ Output Verification               │
                    │ Schema · Safety · Grounding       │
                    │ Output guardrails · Abstain       │
                    └──────────────┬───────────────────┘
                                   │
                    ┌──────────────▼───────────────────┐
                    │ Observability + Eval Pipeline     │
                    │ Traces (OTel GenAI) · Feedback    │
                    │ Eval sets ← production failures   │
                    └──────────────────────────────────┘
```

---

**Prev:** [16. Human-in-the-Loop](../part-3-agentic-systems/16-human-in-the-loop.md) · [Reading list](../README.md) · **Next:** [18. Core Design Rules](18-core-design-rules.md)
