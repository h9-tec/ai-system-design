# 8. RAG System Design

*Part II. Core Techniques · [Reading list](../README.md)*

## Ingestion pipeline

```text
Source systems
    ↓ Parse and normalize (document processing quality decides everything, see §29)
    ↓ Extract metadata and ACLs
    ↓ Structure-aware chunking (per document type)
    ↓ Contextual augmentation (prepend chunk-situating context)
    ↓ Embed + build BM25 index
    ↓ Version documents, chunks, and indexes; track lineage
    ↓ Evaluate retrieval quality on your own data
```

Store metadata with every chunk:

```json
{
  "chunk_id": "policy-2026-012#section-4",
  "document_id": "policy-2026-012",
  "title": "Refund Policy",
  "source_url": "https://internal.example/policy",
  "updated_at": "2026-07-01T00:00:00Z",
  "tenant_id": "tenant_123",
  "allowed_roles": ["support", "manager"],
  "embedding_model": "model-version",
  "content_hash": "sha256..."
}
```

## Contextual Retrieval (measured, not folklore)

The practitioner consensus (hybrid retrieval + reranking) is confirmed by
Anthropic's published benchmarks. Baseline top-20 retrieval failure rate of
5.7% improved as follows:

| Technique | Failure rate | Reduction |
|---|---|---|
| Contextual [embeddings](09-embeddings-and-vectors.md) (prepend ~50-100 token chunk context) | 3.7% | 35% |
| + Contextual BM25 (hybrid lexical + dense) | 2.9% | 49% |
| + Reranking (retrieve ~150, rerank, keep top ~20) | 1.9% | 67% |

Practical notes:

- Generate the chunk context with a cheap model + prompt caching; treat it
  as a one-time indexing cost.
- Late chunking (embed the full document with a long-context model, then pool
  each chunk's vector from those token embeddings) is a cheaper complement
  that recovers cross-chunk context without a per-chunk LLM call; it composes
  with contextual embeddings.
- Reranking adds latency and cost; tune the candidate count for your SLO.
- These numbers came from codebases, papers, and fiction. Measure on your
  own corpus before trusting any published gains.

## Know when NOT to use RAG

If the whole knowledge base fits within your model's tested effective
context, put it in the prompt and use prompt caching instead of building a
retrieval pipeline; large context windows plus caching make this practical
well past what a fixed page count would suggest. For code-like corpora, agentic just-in-time search (grep/glob
style tools) often beats embedding pipelines. Static RAG is also the wrong
tool for open-ended research tasks that need iterative exploration; that is
what agentic retrieval loops are for.

## Retrieval pipeline requirements

```text
Query → AuthZ/tenant filter → query classification and rewriting
      → dense + lexical retrieval → merge (e.g., RRF) → rerank
      → relevance/confidence gate → bounded context build
      → cited answer or abstain
```

- Filter by tenant, role, and document ACL **before** anything reaches the
  model. Vector stores with weak access control are their own [OWASP](../part-4-production-engineering/20-security.md)
  risk (Vector and Embedding Weaknesses, including index poisoning and
  cross-tenant leakage).
- Chunk per document type; preserve headings, tables, source URLs,
  timestamps, ownership.
- Query understanding earns its cost: classify (does this need retrieval at
  all?), rewrite conversational queries into standalone ones, decompose
  multi-part questions.
- Re-embed only changed sections; version embeddings, chunking logic, and
  indexes; keep rollback plans for index migrations.
- Require citations for knowledge-backed answers; abstain when relevance is
  low or sources conflict.
- Evaluate retrieval separately from generation: recall@k, MRR/nDCG for
  retrieval; faithfulness/groundedness and answer relevance for generation
  (RAGAS-style metrics plus human review). A high cosine score is not
  evidence of correctness.
- Consider GraphRAG for multi-hop questions over entity-heavy corpora, and
  agentic RAG for iterative retrieval. Adopt only after the boring hybrid
  pipeline is measured and found insufficient.

---

**Prev:** [7. Context Engineering](07-context-engineering.md) · [Reading list](../README.md) · **Next:** [9. Embeddings and Vector Infrastructure](09-embeddings-and-vectors.md)
