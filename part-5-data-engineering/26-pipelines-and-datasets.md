# 26. Pipelines and Dataset Management

*Part V. Data Engineering · [Reading list](../README.md)*

- **Data quality is the ceiling** for retrieval, [fine-tuning](../part-2-core-techniques/10-fine-tuning.md), and [evals](../part-4-production-engineering/21-evaluation-strategy.md)
  alike. Budget engineering time for parsing, deduplication, filtering,
  and PII scrubbing before budgeting for model work.
- Version datasets like code: content hashes, lineage (source, transform,
  date), changelogs, and reproducible builds (DVC, lakeFS, OpenLineage).
  "Which data produced this model/index" must be answerable in minutes.
- Decontaminate: keep eval sets strictly out of training and few-shot
  pools; leakage produces beautiful dashboards and broken products.
- Define retention and deletion flows across every copy: raw store,
  processed sets, indexes, caches, fine-tuned weights trained on deleted
  data. Deletion requests must propagate.
- Classify each record's provenance at ingestion (human-authored,
  human-edited, AI-generated, unknown), automate quality gates in the
  pipeline (schema checks, language ID, length and dedup filters,
  toxicity/PII scans), and alert on distribution drift in incoming data.

---

**Prev:** [25. Inference and Serving (Self-Hosted)](../part-4-production-engineering/25-inference-and-serving.md) · [Reading list](../README.md) · **Next:** [27. Annotation Operations](27-annotation-operations.md)
