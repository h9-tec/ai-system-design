# 33. Team Practices

*Part VII. Product and Organization · [Reading list](../README.md)*

- **Small full-stack teams beat siloed handoffs.** Two to four people who
  own prompt, retrieval, [evals](../part-4-production-engineering/21-evaluation-strategy.md), and deployment for a use case move faster
  than a chain of specialists; embed with the product team rather than
  operating as an internal service desk.
- **Clear ownership**: every prompt, eval set, index, and model route has
  an owner; changes go through review like code (because they are code).
- **Evals are a first-class deliverable**: a use case is not "done" when
  the demo works; it is done when the eval suite exists and passes (see
  [Definition of Done](../appendices/e-definition-of-done.md)).
- **On-call includes AI failure modes**: runbooks for provider outages,
  quality regressions, injection incidents, and cost anomalies; postmortems
  produce new eval cases, not just action items.
- **Document decisions**: why this model, this chunking, this threshold.
  Six months later, the eval numbers behind a decision are the only defense
  against re-litigating it.
- Grow people through calibration: reviewing transcripts and grading eval
  outputs together is the fastest way to build shared judgment on a team.

---

**Prev:** [32. UX Patterns for AI Products](32-ux-patterns.md) · [Reading list](../README.md) · **Next:** [34. Governance and Compliance (brief, but no longer optional)](34-governance-and-compliance.md)
