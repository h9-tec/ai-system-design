# AI System Design Reference

A practitioner reference for engineers, tech leads, and teams who build AI
systems: from choosing an approach, through core techniques (prompting, RAG,
fine-tuning, agents), to production engineering, data operations, specialized
systems, and governance.

This is the index. The material is an ordered reading list: each Part builds on
the ones before it, and every section is its own file so you can read straight
through or jump to what you need. Sections link to each other where topics
connect.

> Principle: treat AI systems as probabilistic distributed systems, not as
> ordinary API integrations. Design for containment and recovery, not for a
> model that never fails.

## Start here

- **[Foundations](00-foundations.md)**: framing and the six first principles the
  rest of the reference is built on. Read this first.
- Prefer the whole thing on one page? **[full-reference.md](full-reference.md)** is
  the complete original, unsplit.

## Reading list

### Part I. Decisions Before Code

- [1. The Intervention Ladder](part-1-decisions/01-intervention-ladder.md)
- [2. API vs Self-Hosting](part-1-decisions/02-api-vs-self-hosting.md)
- [3. Model Selection](part-1-decisions/03-model-selection.md)
- [4. Build vs Buy](part-1-decisions/04-build-vs-buy.md)

### Part II. Core Techniques

- [5. Prompt Engineering That Survives Production](part-2-core-techniques/05-prompt-engineering.md)
- [6. Structured Outputs](part-2-core-techniques/06-structured-outputs.md)
- [7. Context Engineering](part-2-core-techniques/07-context-engineering.md)
- [8. RAG System Design](part-2-core-techniques/08-rag-system-design.md)
- [9. Embeddings and Vector Infrastructure](part-2-core-techniques/09-embeddings-and-vectors.md)
- [10. Fine-Tuning and Model Adaptation](part-2-core-techniques/10-fine-tuning.md)

### Part III. Agentic Systems

- [11. The Architecture Ladder: Workflows vs Agents](part-3-agentic-systems/11-workflows-vs-agents.md)
- [12. Tools and the Agent-Computer Interface (ACI)](part-3-agentic-systems/12-tools-and-aci.md)
- [13. MCP (Model Context Protocol)](part-3-agentic-systems/13-mcp.md)
- [14. Agent Memory](part-3-agentic-systems/14-agent-memory.md)
- [15. Multi-Agent Systems (rarely justified)](part-3-agentic-systems/15-multi-agent-systems.md)
- [16. Human-in-the-Loop](part-3-agentic-systems/16-human-in-the-loop.md)

### Part IV. Production Engineering

- [17. Reference Architecture](part-4-production-engineering/17-reference-architecture.md)
- [18. Core Design Rules](part-4-production-engineering/18-core-design-rules.md)
- [19. Reliability Patterns](part-4-production-engineering/19-reliability-patterns.md)
- [20. Security (mapped to the OWASP Top 10 for LLM Applications)](part-4-production-engineering/20-security.md)
- [21. Evaluation Strategy](part-4-production-engineering/21-evaluation-strategy.md)
- [22. Observability](part-4-production-engineering/22-observability.md)
- [23. Deployment and Change Management](part-4-production-engineering/23-deployment-and-change-management.md)
- [24. Cost and Latency Engineering](part-4-production-engineering/24-cost-and-latency.md)
- [25. Inference and Serving (Self-Hosted)](part-4-production-engineering/25-inference-and-serving.md)

### Part V. Data Engineering

- [26. Pipelines and Dataset Management](part-5-data-engineering/26-pipelines-and-datasets.md)
- [27. Annotation Operations](part-5-data-engineering/27-annotation-operations.md)
- [28. Synthetic Data](part-5-data-engineering/28-synthetic-data.md)
- [29. Document Processing and OCR](part-5-data-engineering/29-document-processing-and-ocr.md)

### Part VI. Specialized Systems

- [30. Real-Time Voice AI](part-6-specialized-systems/30-real-time-voice.md)
- [31. Multilingual and Arabic-Specific Notes](part-6-specialized-systems/31-multilingual-and-arabic.md)

### Part VII. Product and Organization

- [32. UX Patterns for AI Products](part-7-product-and-organization/32-ux-patterns.md)
- [33. Team Practices](part-7-product-and-organization/33-team-practices.md)
- [34. Governance and Compliance (brief, but no longer optional)](part-7-product-and-organization/34-governance-and-compliance.md)

### Appendices

- [A. Failure Handling Table](appendices/a-failure-handling.md)
- [B. Production Readiness Checklist](appendices/b-production-readiness-checklist.md)
- [C. Anti-Patterns](appendices/c-anti-patterns.md)
- [D. Metrics Quick Reference](appendices/d-metrics-quick-reference.md)
- [E. Definition of Done](appendices/e-definition-of-done.md)
- [F. Sources and Further Reading](appendices/f-sources.md)

---

*Sources and further reading live in [Appendix F](appendices/f-sources.md). The
appendices (failure handling, readiness checklist, anti-patterns, metrics,
definition of done) are working documents for design reviews and launches, not
part of the linear path.*
