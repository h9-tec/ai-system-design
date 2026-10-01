# 31. Multilingual and Arabic-Specific Notes

*Part VI. Specialized Systems · [Reading list](../README.md)*

If your production traffic is Arabic (or any lower-resource language),
generic best practices need adjustments:

- **Token economics**: Arabic text frequently costs 1.5-3x more tokens than
  equivalent English on many tokenizers. Budget context, cost, and latency
  against your real language mix, not English benchmarks.
- **Lexical retrieval needs normalization**: alef/hamza variants, ta
  marbuta, diacritics, and rich morphology break naive BM25. Use
  Arabic-aware analyzers and normalization in the lexical leg of hybrid
  retrieval.
- **Choose [embeddings](../part-2-core-techniques/09-embeddings-and-vectors.md)/rerankers proven on your language**, and validate on
  your own [eval](../part-4-production-engineering/21-evaluation-strategy.md) set rather than English-centric leaderboards.
- **Dialect gap**: MSA-trained retrieval, STT, and judges often miss
  dialectal input. Add query rewriting (dialect to MSA) where it helps,
  build per-dialect eval slices, and test code-switching (Arabic + English
  technical terms), which is the norm in real Gulf traffic.
- **Generation checks**: RTL and mixed-direction rendering, Arabic-Indic vs
  Western numerals, citation formatting.
- **Regional models exist** (Arabic-centric families such as Fanar, ALLaM,
  Jais, Falcon-Arabic, AceGPT, and dialect-focused ones like Atlas-Chat)
  alongside multilingual frontier models; evaluate current versions on your
  own data, using Arabic leaderboards and benchmarks (OALL, AraLingBench,
  DialectalArabicMMLU) as a starting point rather than assuming either
  direction wins.
- **LLM-as-judge calibration is weaker outside English**: spot-check judges
  with native-speaker review before trusting automated scores.
- **Quantization and distillation hit low-resource languages harder** than
  English on average; re-run language-specific evals after any compression
  step (§25).

---

**Prev:** [30. Real-Time Voice AI](30-real-time-voice.md) · [Reading list](../README.md) · **Next:** [32. UX Patterns for AI Products](../part-7-product-and-organization/32-ux-patterns.md)
