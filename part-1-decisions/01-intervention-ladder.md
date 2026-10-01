# 1. The Intervention Ladder

*Part I. Decisions Before Code · [Reading list](../README.md)*

Most quality problems have a cheap fix and an expensive fix. Take the cheap,
reversible one first and keep the [eval](../part-4-production-engineering/21-evaluation-strategy.md) results that prove whether you need
the next rung.

| Symptom | First intervention | Escalate to |
|---|---|---|
| Wrong format, style, tone, or verbosity | Better prompt + few-shot examples + output schema | Supervised [fine-tuning](../part-2-core-techniques/10-fine-tuning.md) (SFT) |
| Missing private or fresh knowledge | [RAG](../part-2-core-techniques/08-rag-system-design.md), or the full corpus in context if small (see §8) | Never fine-tuning; facts change, weights do not |
| Inconsistent reasoning on hard tasks | Decomposition, a stronger model, evaluator-optimizer loop | Reasoning models on the hard routes only |
| Too slow or too expensive at target quality | Prompt caching, routing, trimmed context (§24) | Distill / fine-tune a small model for the narrow task |
| Weak on your language, dialect, or domain jargon | In-language prompts and examples, better model choice | SFT; continued pretraining only as a last resort |
| Unsafe or off-policy outputs | System policy + layered guardrails (§20) | Preference tuning (DPO/RLHF-style) |

Rules of thumb:

- Prompting and retrieval are cheap, fast to change, and reversible.
  Fine-tuning is slow, sticky, and couples you to a base model version.
- Fine-tune for **form and behavior**, retrieve for **facts**. A model can
  be tuned to your ticket format; it should not be tuned to memorize
  policies that change quarterly.
- Every step down the ladder needs a baseline eval from the step above.
  "We fine-tuned because prompting felt weak" is not evidence.

---

**Prev:** [Foundations](../00-foundations.md) · [Reading list](../README.md) · **Next:** [2. API vs Self-Hosting](02-api-vs-self-hosting.md)
