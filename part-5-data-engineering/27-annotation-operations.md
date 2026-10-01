# 27. Annotation Operations

*Part V. Data Engineering · [Reading list](../README.md)*

Labeling is an operations discipline, not a task you throw over a wall.

- **Guidelines with worked examples**, including hard negatives and edge
  cases; a rule without an example will be interpreted N different ways.
- **Calibration rounds before scale**: have all annotators label the same
  sample, measure inter-annotator agreement (e.g., Cohen's kappa; target
  substantial agreement, roughly 0.6-0.7+, before scaling), discuss
  disagreements, revise the guidelines, repeat.
- **Continuous QA**: seed gold tasks with known answers, sample-review each
  annotator, track per-annotator quality over time.
- Disagreement is signal: recurring disagreement usually means the taxonomy
  is wrong, not the annotators.
- For dialect and domain work, annotator selection is part of the design:
  native speakers of the target dialect, domain background where the
  content requires it, and a feedback channel from annotators back to the
  guideline owners.
- **Machine-assist the loop**: pre-label with a model and have annotators
  verify and correct (blind gold tasks guard against automation bias), and
  use active learning to spend human effort on the uncertain, high-value
  examples rather than the easy majority.
- Vendors: run a paid pilot with your own QA before committing volume;
  contract for quality metrics and rework, not just throughput; keep
  ownership of guidelines, gold sets, and the delivered data.

---

**Prev:** [26. Pipelines and Dataset Management](26-pipelines-and-datasets.md) · [Reading list](../README.md) · **Next:** [28. Synthetic Data](28-synthetic-data.md)
