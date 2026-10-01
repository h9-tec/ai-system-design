# D. Metrics Quick Reference

*Appendices · [Reading list](../README.md)*

| Area | Metric | Notes |
|---|---|---|
| Retrieval | recall@k, precision@k, MRR, nDCG | Measure against labeled query-passage pairs; per language/dialect slices |
| Generation | Faithfulness/groundedness, answer relevance, citation accuracy | LLM-judged with calibration; human-anchor periodically |
| Task | Task success rate, escalation rate, abstention rate | Define success per route; abstention is not failure |
| Agents | End-state success, trajectory quality, steps/tool calls per task | Allow multiple valid paths |
| Latency | TTFT, TPOT/ITL, end-to-end P50/P95/P99; voice-to-voice for voice | Different levers move TTFT vs TPOT (§25) |
| Cost | Cost per successful task, tokens per task, cache hit rate | The denominator is successes, not requests |
| Speech | WER/CER per dialect and channel, endpointing error, TTS MOS | Lab numbers do not transfer; use own recordings |
| [Fine-tuning](../part-2-core-techniques/10-fine-tuning.md) | Target-task delta, general-suite regression, safety-suite regression | All three, before promotion |
| Data | Inter-annotator agreement (kappa), gold-task accuracy, dedup/PII rates | Gate the pipeline on these |
| Ops | Error rate, provider failovers, budget-kill rate, cost anomalies | Budget kills are a signal, not just a safeguard |

---

**Prev:** [C. Anti-Patterns](c-anti-patterns.md) · [Reading list](../README.md) · **Next:** [E. Definition of Done](e-definition-of-done.md)
