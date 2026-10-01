# 19. Reliability Patterns

*Part IV. Production Engineering · [Reading list](../README.md)*

## Timeouts at every boundary

| Layer | Examples |
|---|---|
| Client | Request deadline, cancellation |
| Gateway | Rate limit, auth timeout |
| Retrieval | Search and reranker deadline |
| Model | Time-to-first-token and total-generation timeout |
| Tools | Per-tool deadline |
| Workflow | End-to-end wall-clock deadline |

## Retries

- Retry only transient, safe failures: network timeouts, provider 5xx,
  rate-limit responses. Exponential backoff with jitter, capped attempts,
  preserve the original request ID.
- Never blindly retry non-idempotent writes, validation failures, or
  permission errors.
- When a tool fails, feed a **compact** error summary back into context so
  the model can self-correct, with a counter that escalates to a human
  after N consecutive failures (12-Factor Agents, factor 9).
- Hedge tail latency on idempotent read-only calls: issue a backup request
  once the P95 deadline elapses and take the first response. Shed or queue
  load when budgets or capacity are exceeded instead of letting latency
  cascade.

## Fallback ladder

```text
Primary model unavailable
    ↓ Fallback model (same gateway, pinned versions)
    ↓ Cached answer, if still valid and authorized
    ↓ Safe deterministic response
    ↓ Human escalation
```

Fallback output must still pass schema, safety, and grounding checks. Test
degraded modes deliberately (provider outage game days), do not discover
them in production.

## Model gateway

Centralize routing, provider fallback, key management, pinned model
versions, prompt caching, circuit breakers, and per-tenant quotas in one
gateway layer instead of scattering provider logic through the codebase.

---

**Prev:** [18. Core Design Rules](18-core-design-rules.md) · [Reading list](../README.md) · **Next:** [20. Security (mapped to the OWASP Top 10 for LLM Applications)](20-security.md)
