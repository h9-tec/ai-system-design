# 2. API vs Self-Hosting

*Part I. Decisions Before Code · [Reading list](../README.md)*

Decide per workload, not per company. The strongest pattern in practice is
hybrid: frontier models via API for hard, low-volume, high-value routes;
small self-hosted or cheap-API models for high-volume narrow tasks.

| Factor | Favors API | Favors self-hosting |
|---|---|---|
| Data residency / sovereignty | Provider has an in-region or sovereign offering | Regulator or client requires on-prem / in-country (common in Gulf, finance, government) |
| Volume economics | Low or spiky volume (you pay only for use) | Sustained high volume on a narrow task where a small model suffices |
| Capability needed | Frontier reasoning, long context, tool use | An open-weight model covers the task (gpt-oss, Qwen3, DeepSeek, Llama), from a small tuned model to a large mixture-of-experts |
| Latency control | Standard SLOs acceptable | Hard real-time budgets (voice), no network egress |
| Ops maturity | Small team, no GPU/on-call capacity | Existing infra team, GPU access, serving experience |
| Model lifecycle | You accept provider deprecations (mitigate via gateway) | You need a frozen model for years ([compliance](../part-7-product-and-organization/34-governance-and-compliance.md), reproducibility) |

Notes:

- Self-hosting has a fixed floor: GPUs, serving engineering, upgrades,
  on-call. Below a real utilization threshold, APIs are cheaper even at
  list price. Model the break-even with your actual traffic before buying
  hardware.
- Residency often decides first in regulated markets; check provider
  regional hosting and no-training/no-retention terms before assuming you
  must self-host.
- Whatever you choose, put a model gateway in front (§19) so switching is a
  config change, not a rewrite.

---

**Prev:** [1. The Intervention Ladder](01-intervention-ladder.md) · [Reading list](../README.md) · **Next:** [3. Model Selection](03-model-selection.md)
