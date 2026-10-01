# 20. Security (mapped to the OWASP Top 10 for LLM Applications)

*Part IV. Production Engineering · [Reading list](../README.md)*

| Risk | Core mitigation |
|---|---|
| Prompt injection | Assume it succeeds; containment architecture, least privilege, human gates on consequential actions |
| Sensitive information disclosure | Data minimization in context, output filtering, PII redaction in logs |
| Supply chain | Pin and verify models, adapters, datasets, [MCP](../part-3-agentic-systems/13-mcp.md) servers, and libraries; scan third-party artifacts |
| Data and model poisoning | Provenance and validation for training/[fine-tuning](../part-2-core-techniques/10-fine-tuning.md) data and [RAG](../part-2-core-techniques/08-rag-system-design.md) sources |
| Improper output handling | Treat model output as untrusted input to downstream systems; sanitize before render/execute |
| Excessive agency | Minimal tools, minimal permissions, confirmation for high-impact actions |
| System prompt leakage / hidden context exposure | Assume the system prompt and everything else placed before the model (retrieved docs, tool schemas, policy logic) is visible; never put secrets or auth logic there |
| Vector and [embedding](../part-2-core-techniques/09-embeddings-and-vectors.md) weaknesses | ACLs on vector stores, tenant isolation, index poisoning detection |
| Misinformation | Grounding, citations, abstention, human review for high-stakes output |
| Unbounded consumption | Budgets, quotas, per-run circuit breakers, anomaly alerts |

## Prompt injection: design for containment

"Separate instructions from data" is necessary but NOT sufficient; models
cannot reliably distinguish injected instructions inside data, and
classifier-based filters have been bypassed in shipped products (e.g., the
EchoLeak zero-click exfiltration in Microsoft 365 Copilot, CVE-2025-32711).
So:

- Apply Simon Willison's **lethal trifecta** rule: an agent that combines
  (1) access to private data, (2) exposure to untrusted content, and
  (3) the ability to communicate externally can be tricked into exfiltrating
  data. Break at least one leg by architecture: e.g., the agent that reads
  untrusted web/email content does not hold privileged tools or open egress.
- Everything influenced by untrusted input is itself untrusted, including
  the model's own output (chains into Improper Output Handling).
- Layer defenses: input classifiers, allowlisted egress, output validation,
  scoped credentials, human approval for consequential actions. Watch the
  research on capability-based designs (e.g., CaMeL) but do not bet the
  system on any single filter.
- Security controls live in deterministic, auditable code outside the
  model. A system prompt is not a security boundary.

Also standard hygiene: authenticate every request, authorize every
retrieval, encrypt in transit and at rest, redact secrets and PII in logs
and traces, audit privileged tool actions, define retention for prompts and
transcripts, keep credentials out of anything model-visible.

Map controls to a shared threat taxonomy: the OWASP Top 10 for LLM
Applications for the risk classes above, MITRE ATLAS for adversary tactics
and techniques against AI systems, and the NIST AI RMF Generative AI Profile
for governance. Agentic systems need threat modeling beyond that list (tool
abuse, sub-agent chaining, context and memory poisoning); treat excessive
agency, unbounded consumption, and hidden context exposure as the dominant
runtime risks and build the containment above around them.

---

**Prev:** [19. Reliability Patterns](19-reliability-patterns.md) · [Reading list](../README.md) · **Next:** [21. Evaluation Strategy](21-evaluation-strategy.md)
