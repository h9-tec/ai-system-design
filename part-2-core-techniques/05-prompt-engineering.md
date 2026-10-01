# 5. Prompt Engineering That Survives Production

*Part II. Core Techniques · [Reading list](../README.md)*

- **Structure every production prompt**: role and context, task, hard
  constraints, output schema, then examples. Explicit beats clever; a prompt
  another engineer cannot predict the behavior of is a liability.
- **Few-shot beats rule piles.** Two to five diverse, current examples
  usually outperform long lists of instructions. Keep examples in sync with
  the [eval](../part-4-production-engineering/21-evaluation-strategy.md) set; stale examples silently teach stale behavior.
- **Reasoning**: for hard tasks with non-reasoning models, ask for explicit
  steps before the answer. With reasoning models, do not force chain-of-
  thought in the prompt; control effort via the model's reasoning settings
  (Anthropic exposes these as extended, interleaved, and adaptive thinking;
  other providers have their own reasoning-effort controls) and spend the
  tokens where
  evals show they pay. For agentic tool loops, prefer adaptive thinking over a
  fixed budget, and interleaved thinking so the model can reconsider between
  tool calls.
- **Decompose.** Several small prompts with deterministic checks between
  them beat one mega-prompt. This is what the workflow patterns in §11
  formalize.
- **Delimit untrusted data** (XML-style tags around documents, user input,
  tool output). This improves model behavior and clarity, but it is a
  quality technique, not a security control (§20).
- **Write at the right altitude**: specific enough to guide behavior,
  general enough to cover unseen cases. A prompt that accumulates a
  hardcoded if-else clause per incident is a smell; fix the pattern, not
  the instance.
- **Prompts are versioned artifacts**: code review, changelog, eval run on
  every change, rollback path. Keep the stable parts (system, tools,
  reference material) first and identical across requests so prompt caching
  hits (§24).

---

**Prev:** [4. Build vs Buy](../part-1-decisions/04-build-vs-buy.md) · [Reading list](../README.md) · **Next:** [6. Structured Outputs](06-structured-outputs.md)
