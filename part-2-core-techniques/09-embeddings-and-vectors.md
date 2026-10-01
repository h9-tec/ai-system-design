# 9. Embeddings and Vector Infrastructure

*Part II. Core Techniques · [Reading list](../README.md)*

- **Embedding model choice**: [multilingual](../part-6-specialized-systems/31-multilingual-and-arabic.md) coverage on your real language
  mix, retrieval quality on your own [eval](../part-4-production-engineering/21-evaluation-strategy.md) set (not just MTEB rank),
  dimension (storage and latency scale with it), max input length, and
  license. Strong families to benchmark include Qwen3-Embedding, Gemini
  Embedding, Voyage, and Cohere. Switching models later means re-embedding
  everything: version the model with the index and budget for migration.
- **Shrink vectors deliberately.** Matryoshka (MRL) models let you truncate
  dimensions for large storage and latency wins at small recall cost.
  Quantize to int8 (roughly 4x smaller, minor recall loss) or to binary
  (far smaller, recover recall by rescoring the top candidates at full
  precision). Measure recall at each setting.
- **Index type by scale**: exact/flat search is often fine and simplest
  below roughly a million vectors; HNSW for low-latency approximate search
  at scale; IVF/PQ variants when memory-bound. ANN is approximate: measure
  the recall of the index itself, not just end-to-end answer quality.
- **Metadata filtering**: pre-filtering vs post-filtering changes both
  correctness and speed; heavily filtered HNSW queries can degrade badly.
  Test with your real filter selectivity (tenant + role + date is the
  common hard case).
- **Rerank the shortlist.** A cross-encoder reranker over the top candidates
  lifts top-k precision far more than swapping embedding models; Cohere
  Rerank, Voyage, Qwen3-Reranker, and open BGE rerankers are the usual
  choices. Budget its latency against your SLO.
- **Do not over-buy.** Postgres + pgvector or your existing search engine
  (OpenSearch/Elasticsearch, which also gives you BM25 for hybrid) covers
  most workloads. A dedicated vector database is a scale decision, not a
  default.
- **Deletion is a feature.** Deleted or expired documents must leave the
  index (tombstones + reindex jobs). Stale embeddings of removed content
  are a [compliance](../part-7-product-and-organization/34-governance-and-compliance.md) and correctness bug.

---

**Prev:** [8. RAG System Design](08-rag-system-design.md) · [Reading list](../README.md) · **Next:** [10. Fine-Tuning and Model Adaptation](10-fine-tuning.md)
