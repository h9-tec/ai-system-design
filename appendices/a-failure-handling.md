# A. Failure Handling Table

*Appendices · [Reading list](../README.md)*

| Failure | Safe behavior |
|---|---|
| No relevant retrieval result | Abstain and ask for a source, or escalate |
| Conflicting sources | Explain the uncertainty and cite the conflicting documents |
| Model timeout | Retry safe request, use fallback model, or return partial safe result |
| Tool timeout | Never claim success; return pending/failure state |
| Tool validation failure | Reject the operation; feed a compact error back for one correction attempt |
| Repeated tool failures | Stop the loop and escalate to a human with clean context |
| Budget exceeded (steps/tokens/cost/time) | Stop the workflow, return best partial result or escalate |
| Permission failure | Deny without revealing whether the unauthorized data exists |
| Output schema failure | One retry with the error included, then deterministic fallback |
| Guardrail violation detected | Block, log, and route per policy; never "fix and forward" silently |
| Provider outage | Gateway fallback ladder (§19); degraded mode with honest messaging |
| Injection suspected | Contain: freeze privileged tools for the session, log a redacted trace, review |
| Low STT confidence / unintelligible audio (voice) | Ask to repeat once, then offer human handoff |

---

**Prev:** [34. Governance and Compliance (brief, but no longer optional)](../part-7-product-and-organization/34-governance-and-compliance.md) · [Reading list](../README.md) · **Next:** [B. Production Readiness Checklist](b-production-readiness-checklist.md)
