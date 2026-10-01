# 15. Multi-Agent Systems (rarely justified)

*Part III. Agentic Systems · [Reading list](../README.md)*

Anthropic's multi-agent research system (orchestrator + parallel subagents)
outperformed a single-agent setup by 90.2% on their internal research [eval](../part-4-production-engineering/21-evaluation-strategy.md),
but at roughly 15x the tokens of a normal chat (single agents already run
about 4x). In Anthropic's BrowseComp analysis, token usage alone explained about 80% of the performance variance, a separate measurement from the 90.2% research-eval result above.

Use multi-agent only when:

- The task is breadth-first and decomposes into independent parallel
  directions (research, broad comparisons, large-scale review).
- The value per run justifies the token multiplier.
- Tasks with tight shared context or many inter-step dependencies are a
  poor fit; keep those single-agent.

A strong counter-argument cuts against multi-agent for most work: when subagents
write to shared state, their separate contexts drift and conflict, and
reconciling their output costs more than the parallelism saves. The dividing
line is read-heavy versus write-heavy. Parallel subagents pay off on read-heavy,
decomposable work like research and broad review; write-heavy tasks with one
continuously changing state (coding especially) are better kept single-agent.

Engineering lessons from production multi-agent systems:

- The orchestrator must give subagents detailed task descriptions
  (objective, output format, tool guidance, boundaries); vague delegation
  duplicates work and leaves gaps.
- Let subagents write outputs to shared artifacts (filesystem) instead of
  funneling everything through the lead agent's context.
- Add per-run cost circuit breakers; the 15x baseline compounds when a
  subagent misbehaves.
- Checkpoint state and resume from failure points; restarts are expensive.
- Deploy with rainbow deployments so in-flight runs finish on the old
  version (§23).

---

**Prev:** [14. Agent Memory](14-agent-memory.md) · [Reading list](../README.md) · **Next:** [16. Human-in-the-Loop](16-human-in-the-loop.md)
