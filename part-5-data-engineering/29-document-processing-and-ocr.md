# 29. Document Processing and OCR

*Part V. Data Engineering · [Reading list](../README.md)*

For document-heavy systems, parsing quality sets the ceiling on everything
downstream; no retrieval trick recovers a table the parser destroyed.

- Route by document type: digital-native PDFs (extract text + layout),
  scanned documents (OCR), forms and tables (structure-aware extraction),
  images/figures (vision models where they matter).
- Preserve structure: headings hierarchy, tables as tables (not
  concatenated cells), reading order in multi-column layouts, page numbers
  and bounding boxes for citation back to the source.
- Vision-LLM parsing is a mainstream default, not just a fallback for messy
  layouts: structure-aware toolkits (Docling and compact document VLMs like
  Granite-Docling, plus Marker and Unstructured) for self-hosting, or hosted
  parsers (LlamaParse, Mistral OCR) when you want less ops. Choose hosted
  (per-page cost, data leaves your boundary) versus self-hosted VLMs (no
  per-page fees, full control), weigh both against classical OCR + layout
  models on cost and hallucination risk, and [eval](../part-4-production-engineering/21-evaluation-strategy.md)
  extraction accuracy separately from end-to-end answer quality.
- Right-to-left scripts add specific failure modes: reading order, mixed
  RTL/LTR lines ([Arabic](../part-6-specialized-systems/31-multilingual-and-arabic.md) text with Latin product names and numbers), digit
  shaping, diacritics. Test your parser on your real documents, not vendor
  samples.
- Keep per-page extraction confidence and route low-confidence pages to
  review instead of silently indexing garbage.

---

**Prev:** [28. Synthetic Data](28-synthetic-data.md) · [Reading list](../README.md) · **Next:** [30. Real-Time Voice AI](../part-6-specialized-systems/30-real-time-voice.md)
