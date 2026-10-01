# B. Production Readiness Checklist

*Appendices · [Reading list](../README.md)*

## Approach and models
- [ ] [Intervention ladder](../part-1-decisions/01-intervention-ladder.md) walked: cheaper fixes tried and [eval](../part-4-production-engineering/21-evaluation-strategy.md)-compared before expensive ones
- [ ] Model per route selected on own evals; gateway makes switching a config change
- [ ] API vs self-host decision documented (residency, economics, ops capacity)

## Architecture
- [ ] Simplest viable pattern chosen and justified by evals (ladder level documented)
- [ ] Workflows have explicit states, checkpoints, and terminal outcomes
- [ ] Every request has an end-to-end deadline
- [ ] Step, tool, token, cost, and retry budgets enforced (incl. per-run circuit breakers)
- [ ] External writes idempotent; two-phase commit for high-risk actions
- [ ] Pause/resume + human approval implemented as first-class states
- [ ] Business rules implemented deterministically

## RAG and data
- [ ] Parsing/[OCR](../part-5-data-engineering/29-document-processing-and-ocr.md) quality measured before blaming retrieval or the model
- [ ] Source, version, timestamp, and ACL metadata on every chunk
- [ ] Authorization filtering happens before retrieval
- [ ] Chunking appropriate per document type; contextual augmentation evaluated
- [ ] Hybrid retrieval + reranking measured on own corpus (not assumed)
- [ ] Answers carry verifiable citations; low-confidence requests abstain
- [ ] Retrieval metrics tracked separately from answer metrics
- [ ] Index/[embedding](../part-2-core-techniques/09-embeddings-and-vectors.md) migrations have dual-write and rollback plans
- [ ] Deletion propagates to indexes, caches, and derived datasets
- [ ] Checked whether [RAG](../part-2-core-techniques/08-rag-system-design.md) is even needed (small corpus + prompt caching?)

## Fine-tuning (if used)
- [ ] Prompt+RAG baseline documented; eval gap justifies tuning
- [ ] Training data versioned, deduplicated, decontaminated; held-out test set
- [ ] Before/after evals on target task AND general + safety suites
- [ ] Checkpoint promotion is eval-gated with rollback

## Security
- [ ] Threat model covers the [OWASP](../part-4-production-engineering/20-security.md) LLM Top 10
- [ ] Lethal trifecta broken by architecture for every agent
- [ ] Model output treated as untrusted downstream; sanitized before render/execute
- [ ] Tools least-privilege, risk-rated, high-risk gated by confirmation
- [ ] Third-party [MCP](../part-3-agentic-systems/13-mcp.md) servers pinned, reviewed, credential-scoped
- [ ] No secrets in prompts; system prompt assumed public
- [ ] Code execution sandboxed with egress allowlist
- [ ] Vector store ACLs and tenant isolation verified
- [ ] Red-team suite exists and replays on every change

## Quality
- [ ] Representative eval set exists and grows from production failures
- [ ] LLM judges rubric-based and calibrated against human labels (bias controls in place)
- [ ] Agent evals score end state + trajectory, not transcript similarity
- [ ] Regression suite gates every prompt/model/index/tool change
- [ ] Shadow/canary path exists before full rollout

## Operations
- [ ] Traces follow OTel GenAI conventions; IDs propagate end to end
- [ ] P95/P99 latency, error-rate, and cost-anomaly alerts configured
- [ ] Cost per successful task tracked by route and tenant
- [ ] Prompt caching implemented; cache hit rate monitored
- [ ] Provider outage and degraded modes tested (game days)
- [ ] Rainbow/canary deployment for stateful agents
- [ ] Secrets and PII redacted from logs; retention rules defined
- [ ] Incident runbooks for model, retrieval, tool, and cost failures
- [ ] Regulatory mapping done (EU AI Act category, local requirements, residency)

## Voice (if applicable)
- [ ] P95 voice-to-voice latency within budget; all stages streaming
- [ ] Barge-in and endpointing tested with real interruption patterns
- [ ] STT evaluated per dialect and channel on own recordings
- [ ] AI disclosure and human handoff implemented

---

**Prev:** [A. Failure Handling Table](a-failure-handling.md) · [Reading list](../README.md) · **Next:** [C. Anti-Patterns](c-anti-patterns.md)
