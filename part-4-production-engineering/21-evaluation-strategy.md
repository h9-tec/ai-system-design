# 21. Evaluation Strategy

*Part IV. Production Engineering · [Reading list](../README.md)*

## Start small, start real

Do not wait for a perfect benchmark. Anthropic's research team started with
about 20 representative queries and hand inspection; small-n evals with real
cases catch most regressions. Grow the set from production: successful
requests, user corrections, retrieval misses, ambiguous questions,
adversarial prompts, permission-boundary tests, long-context cases, tool
failures, escalations.

Every example defines expected behavior:

```json
{
  "input": "What is the refund window?",
  "expected_behavior": "Answer from current policy with citation",
  "allowed_sources": ["refund-policy-2026"],
  "must_not_do": ["Use deprecated policy", "Invent exceptions"],
  "success_criteria": {
    "grounded": true,
    "citation_required": true,
    "max_latency_ms": 3000
  }
}
```

## Layer the graders

1. **Deterministic checks**: schema validity, citation presence, banned
   content, latency, cost. Cheap, run everywhere.
2. **LLM-as-judge** with an explicit rubric for fuzzy qualities
   (groundedness, helpfulness, tone). Known biases you must control for:
   position bias (swap answer order), verbosity bias (control for length),
   and self-preference (use a different model as judge where possible).
   Calibrate judges against human labels on a sample before trusting them.
3. **Human review** for high-stakes flows and for periodically re-anchoring
   the automated graders.

Do not build graders from scratch: promptfoo, OpenAI Evals, and DeepEval
cover deterministic and model-graded checks; Ragas scores retrieval on
faithfulness, answer relevance, and context precision and recall; Braintrust,
LangSmith, and Phoenix add managed datasets, dashboards, and trace-linked
scoring.

## Agent-specific evaluation

Judge the **end state** (did the task actually get done in the environment)
and the **trajectory** (tool choices, budget use, safety of intermediate
actions), while allowing multiple valid paths to the goal. Turn-by-turn
similarity to a golden transcript is the wrong metric for agents.

## Red teaming

Adversarial testing is part of evaluation, not a one-time audit: injection
attempts through every untrusted channel (documents, web content, tool
output, memory writes), permission-boundary probes, jailbreak suites, and
domain-specific abuse cases. Automate replay of every known attack as a
regression test and re-run on every model or prompt change.

## Run evals on every meaningful change

Prompts, model or provider version, retrieval/index config, [embedding](../part-2-core-techniques/09-embeddings-and-vectors.md)
model, chunking logic, tool schemas or permissions, workflow transitions,
guardrails, [fine-tuned](../part-2-core-techniques/10-fine-tuning.md) checkpoints. Gate deploys in CI on the regression
suite, then verify with shadow or canary traffic. Convert every production
incident into a regression case. This is eval-driven development: the
regression suite is the release gate, not an afterthought.

---

**Prev:** [20. Security (mapped to the OWASP Top 10 for LLM Applications)](20-security.md) · [Reading list](../README.md) · **Next:** [22. Observability](22-observability.md)
