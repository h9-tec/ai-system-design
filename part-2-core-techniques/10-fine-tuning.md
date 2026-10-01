# 10. Fine-Tuning and Model Adaptation

*Part II. Core Techniques · [Reading list](../README.md)*

## When it helps

- Consistent format, style, tone, or domain phrasing that prompting cannot
  hold reliably.
- Narrow structured tasks (classification, extraction, routing) where a
  tuned small model matches a frontier model at a fraction of the cost and
  latency.
- Teaching interaction patterns: your tool-calling conventions, your
  schema, your refusal policy.
- Distillation: generate high-quality traces with a strong teacher model,
  train a small student for the narrow route (check the teacher's terms of
  use first).

## When it does not

- Fresh or frequently changing facts: that is retrieval's job, and a tuned
  model with stale knowledge fails confidently.
- A moving product: every prompt or policy change may mean a retrain.
- As the first resort: exhaust prompting + retrieval and keep the [eval](../part-4-production-engineering/21-evaluation-strategy.md)
  gap as your justification.

## Methods, roughly in order of cost

- **SFT with LoRA/QLoRA**: parameter-efficient adapters (train ~1% of
  weights, 4-bit base for QLoRA); the default for most teams. Full-parameter
  SFT only when adapters demonstrably cap quality.
- **Preference tuning (DPO, and successors KTO, ORPO, SimPO)**: aligns
  behavior to chosen-vs-rejected pairs, or to binary keep/reject labels,
  without a full RLHF pipeline; use for tone, safety posture, and judgment
  calls that are easier to rank than to write. When writing preference labels
  is the bottleneck, RLAIF has a judge model generate them.
- **Reinforcement fine-tuning (RL with verifiable rewards, e.g. GRPO)**: for
  narrow tasks where correctness is programmatically checkable (code, math,
  extraction accuracy, policy compliance), train against a grader instead of
  labeled outputs. Available as a managed offering; the right rung when a
  gradeable reasoning task still misses after SFT and preference tuning.
- **Continued pretraining**: for genuine language/domain gaps (e.g., an
  underrepresented dialect). Needs orders of magnitude more data and
  compute, and risks degrading general ability; a last resort.

## Process discipline

- Data quality beats volume: a few thousand excellent, deduplicated,
  decontaminated examples routinely beat tens of thousands of scraped ones.
  Keep a held-out test set that never touches training.
- Always eval before/after on both the target task AND general + safety
  suites; catastrophic forgetting and safety regressions are real and
  silent.
- Version datasets like code (hashes, lineage, changelogs). You will need
  to answer "what was this model trained on" later, possibly to a
  regulator.
- Even after tuning, keep facts in retrieval and business rules in code.

---

**Prev:** [9. Embeddings and Vector Infrastructure](09-embeddings-and-vectors.md) · [Reading list](../README.md) · **Next:** [11. The Architecture Ladder: Workflows vs Agents](../part-3-agentic-systems/11-workflows-vs-agents.md)
