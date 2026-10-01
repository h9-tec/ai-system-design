# AI Engineering Reference: System Design and Real-World Practices

A comprehensive reference for engineers, tech leads, and teams who build AI
systems: from choosing an approach, through core techniques (prompting, RAG,
fine-tuning, agents), to production engineering, data operations,
specialized systems, and governance.

This guide started from practitioner discussions (Reddit engineering
threads) and was validated and expanded against primary sources: Anthropic
and OpenAI engineering guides, the OWASP Top 10 for LLM Applications (2025),
the 12-Factor Agents methodology, OpenTelemetry GenAI conventions, academic
work on retrieval and long context, and current regulation (EU AI Act as
amended by the 2026 Digital Omnibus). Full references at the end.

> Principle: treat AI systems as probabilistic distributed systems, not as
> ordinary API integrations. Design for containment and recovery, not for a
> model that never fails.

How to use this document: each section stands alone. Skim the contents,
read the sections relevant to your current problem, and use the appendices
(failure handling, checklist, anti-patterns, metrics) as working documents
during design reviews and launches.

---

## Contents

- **Part I. Decisions Before Code**
  1. The intervention ladder
  2. API vs self-hosting
  3. Model selection
  4. Build vs buy
- **Part II. Core Techniques**
  5. Prompt engineering that survives production
  6. Structured outputs
  7. Context engineering
  8. RAG system design
  9. Embeddings and vector infrastructure
  10. Fine-tuning and model adaptation
- **Part III. Agentic Systems**
  11. The architecture ladder: workflows vs agents
  12. Tools and the agent-computer interface
  13. MCP (Model Context Protocol)
  14. Agent memory
  15. Multi-agent systems
  16. Human-in-the-loop
- **Part IV. Production Engineering**
  17. Reference architecture
  18. Core design rules
  19. Reliability patterns
  20. Security
  21. Evaluation strategy
  22. Observability
  23. Deployment and change management
  24. Cost and latency engineering
  25. Inference and serving (self-hosted)
- **Part V. Data Engineering**
  26. Pipelines and dataset management
  27. Annotation operations
  28. Synthetic data
  29. Document processing and OCR
- **Part VI. Specialized Systems**
  30. Real-time voice AI
  31. Multilingual and Arabic-specific notes
- **Part VII. Product and Organization**
  32. UX patterns for AI products
  33. Team practices
  34. Governance and compliance
- **Appendices**
  A. Failure handling table · B. Production readiness checklist ·
  C. Anti-patterns · D. Metrics quick reference · E. Definition of done ·
  F. Sources

---

## First Principles (validated across sources)

1. **Find the simplest solution that passes evaluation.** Anthropic's core
   guidance: the most successful production implementations use simple,
   composable patterns, not complex frameworks. Often a single well-prompted
   LLM call with retrieval and examples is enough. Add complexity only when
   it measurably wins.
2. **Eval-driven development.** You cannot iterate on what you cannot
   measure. A small set of real cases (even 20) beats zero evals.
3. **Most "agents" in production are mostly software.** Reliable systems are
   deterministic code with LLM decision points placed at strategic moments,
   not "prompt + bag of tools + loop until done" (12-Factor Agents).
4. **Assume prompt injection succeeds.** There is no reliable prevention
   today. Security comes from architecture: least privilege, output
   handling, and human gates, not from instructions in the prompt.
5. **Data quality is the ceiling.** Retrieval cannot fix bad parsing,
   fine-tuning cannot fix bad labels, and no model fixes an OCR layer that
   destroyed the tables.
6. **Cost per successful task** is the efficiency metric that matters, not
   cost per request.

---

# Part I. Decisions Before Code

## 1. The Intervention Ladder

Most quality problems have a cheap fix and an expensive fix. Take the cheap,
reversible one first and keep the eval results that prove whether you need
the next rung.

| Symptom | First intervention | Escalate to |
|---|---|---|
| Wrong format, style, tone, or verbosity | Better prompt + few-shot examples + output schema | Supervised fine-tuning (SFT) |
| Missing private or fresh knowledge | RAG, or the full corpus in context if small (see §8) | Never fine-tuning; facts change, weights do not |
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

## 2. API vs Self-Hosting

Decide per workload, not per company. The strongest pattern in practice is
hybrid: frontier models via API for hard, low-volume, high-value routes;
small self-hosted or cheap-API models for high-volume narrow tasks.

| Factor | Favors API | Favors self-hosting |
|---|---|---|
| Data residency / sovereignty | Provider has an in-region or sovereign offering | Regulator or client requires on-prem / in-country (common in Gulf, finance, government) |
| Volume economics | Low or spiky volume (you pay only for use) | Sustained high volume on a narrow task where a small model suffices |
| Capability needed | Frontier reasoning, long context, tool use | Task is narrow enough for a tuned 7B-70B class model |
| Latency control | Standard SLOs acceptable | Hard real-time budgets (voice), no network egress |
| Ops maturity | Small team, no GPU/on-call capacity | Existing infra team, GPU access, serving experience |
| Model lifecycle | You accept provider deprecations (mitigate via gateway) | You need a frozen model for years (compliance, reproducibility) |

Notes:

- Self-hosting has a fixed floor: GPUs, serving engineering, upgrades,
  on-call. Below a real utilization threshold, APIs are cheaper even at
  list price. Model the break-even with your actual traffic before buying
  hardware.
- Residency often decides first in regulated markets; check provider
  regional hosting and no-training/no-retention terms before assuming you
  must self-host.
- Whatever you choose, put a model gateway in front (§19) so switching is a
  config change, not a rewrite.

## 3. Model Selection

- **Your evals decide, not leaderboards.** Public benchmarks suffer
  contamination and rarely match your task, language mix, or latency
  budget. Build a 50-200 case eval from real traffic and run every
  candidate through it.
- Pick per route: capability, latency, and cost form a triangle; different
  routes in the same product should use different models.
- Test what marketing numbers hide: effective context recall at depth (not
  the advertised window), structured-output reliability, tool-calling
  quality under your real schemas, refusal behavior on your domain, and
  quality on YOUR languages and dialects.
- Check the boring parts: license and usage terms (including whether
  outputs can train competitors' models), deprecation policy, rate limits,
  regional availability, fine-tuning access.
- Re-run selection periodically; the model landscape shifts quarterly, and
  a gateway makes re-selection cheap.

## 4. Build vs Buy

- Buy or configure when the capability is a commodity: generic chat over
  documents, transcription, translation, standard OCR. Your differentiation
  will not come from rebuilding these.
- Build when the moat is yours: the workflow, the domain data, the eval
  sets, the feedback flywheel, and the integrations. Thin wrappers die when
  the platform ships the feature; proprietary evals and data pipelines
  survive model swaps.
- For anything bought: demand exportable data, measurable quality (run your
  evals against the vendor), and an exit path.

---

# Part II. Core Techniques

## 5. Prompt Engineering That Survives Production

- **Structure every production prompt**: role and context, task, hard
  constraints, output schema, then examples. Explicit beats clever; a prompt
  another engineer cannot predict the behavior of is a liability.
- **Few-shot beats rule piles.** Two to five diverse, current examples
  usually outperform long lists of instructions. Keep examples in sync with
  the eval set; stale examples silently teach stale behavior.
- **Reasoning**: for hard tasks with non-reasoning models, ask for explicit
  steps before the answer. With reasoning models, do not force chain-of-
  thought in the prompt; control effort via the model's reasoning settings
  and spend the tokens where evals show they pay.
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

## 6. Structured Outputs

- Use native structured outputs / constrained decoding for anything a
  machine will read: tool calls, extraction, classification, routing.
- Schema conformance is not correctness. Validate business rules
  server-side: date ranges, ID existence, enum semantics, cross-field
  consistency.
- Design schemas the model can fill reliably: enums over free strings,
  bounded lengths, explicit null semantics, flat over deeply nested
  optional trees. Add a short field-by-field description; the schema is
  part of the prompt.
- On schema or validation failure: one retry that includes the concrete
  error, then a deterministic fallback or escalation. Do not loop.
- For extraction at scale, include an explicit "not_found" / abstain path
  in the schema so the model has a legal way to say the data is absent.

## 7. Context Engineering

Context is a finite resource with diminishing returns. As token count grows,
recall and reasoning degrade ("context rot", the "lost in the middle"
effect). 12-Factor Agents practitioners report degradation well before the
window is full (past roughly 40% utilization). Bigger windows raise the
ceiling; they do not remove the need for curation.

Rules:

- Send the **smallest set of high-signal tokens** that makes the task
  solvable: system instructions, the user request, approved retrieved
  chunks with metadata, required output schema, abstention policy.
- Prefer **just-in-time retrieval** (the model fetches what it needs via
  lightweight tools) over pre-loading everything, and hybrid approaches
  where a small stable core is pre-loaded.
- Do not dump full conversation history or raw tool logs into the prompt.

For long-horizon tasks (Anthropic's context engineering guidance):

1. **Compaction**: summarize older turns into a compressed state and reset.
   Tune the compaction prompt on real traces: maximize recall first, then
   trim. The lightest-touch form is **tool result clearing** (drop old raw
   tool outputs, keep the record that the call happened). Server-side
   compaction cut token use 84% in a 100-turn agent eval while letting the
   task finish.
2. **Structured note-taking (agentic memory)**: the agent writes durable
   notes to external storage (a NOTES.md, a memory tool) and reads them
   back. Store only what constrains future reasoning: decisions,
   preferences, failed approaches. Too much stored state becomes persistent
   context pollution.
3. **Sub-agents with clean contexts**: delegate focused subtasks to agents
   with fresh windows that return condensed summaries, instead of one agent
   dragging everything along.

## 8. RAG System Design

### Ingestion pipeline

```text
Source systems
    ↓ Parse and normalize (document processing quality decides everything, see §29)
    ↓ Extract metadata and ACLs
    ↓ Structure-aware chunking (per document type)
    ↓ Contextual augmentation (prepend chunk-situating context)
    ↓ Embed + build BM25 index
    ↓ Version documents, chunks, and indexes; track lineage
    ↓ Evaluate retrieval quality on your own data
```

Store metadata with every chunk:

```json
{
  "chunk_id": "policy-2026-012#section-4",
  "document_id": "policy-2026-012",
  "title": "Refund Policy",
  "source_url": "https://internal.example/policy",
  "updated_at": "2026-07-01T00:00:00Z",
  "tenant_id": "tenant_123",
  "allowed_roles": ["support", "manager"],
  "embedding_model": "model-version",
  "content_hash": "sha256..."
}
```

### Contextual Retrieval (measured, not folklore)

The practitioner consensus (hybrid retrieval + reranking) is confirmed by
Anthropic's published benchmarks. Baseline top-20 retrieval failure rate of
5.7% improved as follows:

| Technique | Failure rate | Reduction |
|---|---|---|
| Contextual embeddings (prepend ~50-100 token chunk context) | 3.7% | 35% |
| + Contextual BM25 (hybrid lexical + dense) | 2.9% | 49% |
| + Reranking (retrieve ~150, rerank, keep top ~20) | 1.9% | 67% |

Practical notes:

- Generate the chunk context with a cheap model + prompt caching; treat it
  as a one-time indexing cost.
- Reranking adds latency and cost; tune the candidate count for your SLO.
- These numbers came from codebases, papers, and fiction. Measure on your
  own corpus before trusting any published gains.

### Know when NOT to use RAG

If the whole knowledge base fits in roughly 200K tokens (about 500 pages),
put it in the prompt and use prompt caching instead of building a retrieval
pipeline. For code-like corpora, agentic just-in-time search (grep/glob
style tools) often beats embedding pipelines. Static RAG is also the wrong
tool for open-ended research tasks that need iterative exploration; that is
what agentic retrieval loops are for.

### Retrieval pipeline requirements

```text
Query → AuthZ/tenant filter → query classification and rewriting
      → dense + lexical retrieval → merge (e.g., RRF) → rerank
      → relevance/confidence gate → bounded context build
      → cited answer or abstain
```

- Filter by tenant, role, and document ACL **before** anything reaches the
  model. Vector stores with weak access control are their own OWASP
  category now (LLM08: vector and embedding weaknesses, including index
  poisoning and cross-tenant leakage).
- Chunk per document type; preserve headings, tables, source URLs,
  timestamps, ownership.
- Query understanding earns its cost: classify (does this need retrieval at
  all?), rewrite conversational queries into standalone ones, decompose
  multi-part questions.
- Re-embed only changed sections; version embeddings, chunking logic, and
  indexes; keep rollback plans for index migrations.
- Require citations for knowledge-backed answers; abstain when relevance is
  low or sources conflict.
- Evaluate retrieval separately from generation: recall@k, MRR/nDCG for
  retrieval; faithfulness/groundedness and answer relevance for generation
  (RAGAS-style metrics plus human review). A high cosine score is not
  evidence of correctness.
- Consider GraphRAG for multi-hop questions over entity-heavy corpora, and
  agentic RAG for iterative retrieval. Adopt only after the boring hybrid
  pipeline is measured and found insufficient.

## 9. Embeddings and Vector Infrastructure

- **Embedding model choice**: multilingual coverage on your real language
  mix, retrieval quality on your own eval set (not just MTEB rank),
  dimension (storage and latency scale with it), max input length, and
  license. Switching models later means re-embedding everything: version
  the model with the index and budget for migration.
- **Index type by scale**: exact/flat search is often fine and simplest
  below roughly a million vectors; HNSW for low-latency approximate search
  at scale; IVF/PQ variants when memory-bound. ANN is approximate: measure
  the recall of the index itself, not just end-to-end answer quality.
- **Metadata filtering**: pre-filtering vs post-filtering changes both
  correctness and speed; heavily filtered HNSW queries can degrade badly.
  Test with your real filter selectivity (tenant + role + date is the
  common hard case).
- **Do not over-buy.** Postgres + pgvector or your existing search engine
  (OpenSearch/Elasticsearch, which also gives you BM25 for hybrid) covers
  most workloads. A dedicated vector database is a scale decision, not a
  default.
- **Deletion is a feature.** Deleted or expired documents must leave the
  index (tombstones + reindex jobs). Stale embeddings of removed content
  are a compliance and correctness bug.

## 10. Fine-Tuning and Model Adaptation

### When it helps

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

### When it does not

- Fresh or frequently changing facts: that is retrieval's job, and a tuned
  model with stale knowledge fails confidently.
- A moving product: every prompt or policy change may mean a retrain.
- As the first resort: exhaust prompting + retrieval and keep the eval
  gap as your justification.

### Methods, roughly in order of cost

- **SFT with LoRA/QLoRA**: parameter-efficient adapters (train ~1% of
  weights, 4-bit base for QLoRA); the default for most teams. Full-parameter
  SFT only when adapters demonstrably cap quality.
- **Preference tuning (DPO and successors)**: aligns behavior to
  chosen-vs-rejected pairs without a full RLHF pipeline; use for tone,
  safety posture, and judgment calls that are easier to rank than to write.
- **Continued pretraining**: for genuine language/domain gaps (e.g., an
  underrepresented dialect). Needs orders of magnitude more data and
  compute, and risks degrading general ability; a last resort.

### Process discipline

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

# Part III. Agentic Systems

## 11. The Architecture Ladder: Workflows vs Agents

Move down this ladder only when the level above fails your evals:

```text
Level 1: Single LLM call
         + retrieval, in-context examples, structured output
Level 2: Workflow (predefined code path orchestrating multiple calls)
Level 3: Agent (model directs its own loop and tool use)
Level 4: Multi-agent (orchestrator + parallel subagents)
```

### Workflow patterns (Anthropic, "Building Effective Agents")

| Pattern | Use when |
|---|---|
| Prompt chaining | Task decomposes into fixed sequential steps, each verifiable |
| Routing | Distinct input categories need different handling or models |
| Parallelization | Independent subtasks (sectioning) or diverse attempts (voting) |
| Orchestrator-workers | Subtasks cannot be predicted upfront; a lead model delegates |
| Evaluator-optimizer | Clear evaluation criteria exist and iteration adds value |

### When agents are justified

- The path cannot be hardcoded, but progress can be verified
- The task is valuable enough to pay for exploration (tokens, latency)
- You can sandbox execution and define stopping conditions

### Framework caution

Frameworks (LangGraph, CrewAI, etc.) speed up the start but add abstraction
layers that obscure prompts and control flow, making debugging harder.
Start with direct API calls; if you adopt a framework, make sure you can
inspect every prompt and own the loop (retry, pause, terminate logic).

## 12. Tools and the Agent-Computer Interface (ACI)

Tool design deserves the same care as prompt design. Anthropic's guidance
from building tools for Claude:

- **Few, focused, non-overlapping tools** beat exhaustive API mirrors.
  Build a handful of thoughtful tools for high-impact workflows that match
  your evals, then expand. Every exposed tool definition costs tokens and
  attention.
- **Consolidate multi-step operations** into single tools where the steps
  always go together (schedule_event that also checks availability, not
  three chained primitives).
- **Return token-efficient, high-signal responses.** A search_contacts tool
  beats list_all_contacts; do not make the agent brute-force through
  irrelevant output. Support concise vs detailed response formats.
- **Make errors instructive.** A good error tells the model how to correct
  the call, then gets compacted into context (not a raw stack trace).
- **Poka-yoke the arguments.** Design parameters so misuse is hard (require
  absolute paths, enums instead of free strings, explicit idempotency_key).
- Iterate on tool descriptions using real transcripts; test tools with the
  agent in the loop, not just unit tests.

### Tool contract

```json
{
  "name": "create_support_ticket",
  "description": "Create a customer-support ticket after user confirmation.",
  "input_schema": {
    "type": "object",
    "required": ["customer_id", "summary", "priority", "idempotency_key"],
    "properties": {
      "customer_id": { "type": "string" },
      "summary": { "type": "string", "maxLength": 1000 },
      "priority": { "enum": ["low", "normal", "high"] },
      "idempotency_key": { "type": "string" }
    }
  }
}
```

### Tool rules

- Minimum required permissions per tool; separate read tools from write
  tools; rate each tool's risk (low/medium/high) and gate high-risk tools
  behind confirmation.
- Validate all model-produced parameters server-side against the schema AND
  against business rules; validation next to the side effect is more
  reliable than agent-level checks.
- Per-tool timeout, retry, and cost limits; log inputs, outputs, duration,
  status, error category.
- Never expose credentials, raw database access, or unrestricted shell.
  Sandbox code execution with an egress allowlist.
- Use structured outputs / constrained decoding for every tool call; still
  validate, because schema conformance is not semantic correctness.

## 13. MCP (Model Context Protocol)

MCP is the open standard for connecting models to tools and data
(client-server), launched by Anthropic in late 2024, adopted across the
major AI vendors and developer tools, and now governed vendor-neutrally
under a Linux Foundation body. Treat it as the default integration layer
when you want tools reusable across models and clients, instead of bespoke
per-provider function wiring.

Production rules:

- **Third-party MCP servers are supply chain** (OWASP LLM03). Pin versions,
  review the tool descriptions you import (tool-description injection is a
  real attack vector), and re-review on update.
- Scope credentials per server, allowlist which servers each agent may use,
  and prefer read-only servers wherever possible.
- Everything in §12 still applies: an MCP tool is still a tool. The
  protocol standardizes transport and discovery, not safety.
- The spec is evolving (auth hardening, long-running tasks, stateless
  transport); pin the protocol version you deploy against and track
  deprecations.

## 14. Agent Memory

- **Working memory is the context window.** Manage it with the techniques
  in §7 (compaction, tool result clearing, sub-agents).
- **Persistent memory** is external: structured notes/files the agent
  writes and re-reads, or retrieval over past interactions. Store
  decisions, stable preferences, constraints, and failed approaches, not
  raw transcripts; over-storage becomes permanent context pollution.
- **Memory is an attack and privacy surface.** Poisoned memories persist
  across sessions (a stored injected instruction fires forever), so
  validate and constrain what gets written; give users visibility and
  deletion controls; apply the same ACL discipline as retrieval (§8).
- Expire or summarize aggressively. Memory systems degrade into noise
  without a curation policy.

## 15. Multi-Agent Systems (rarely justified)

Anthropic's multi-agent research system (orchestrator + parallel subagents)
outperformed a single-agent setup by 90.2% on their internal research eval,
but at roughly 15x the tokens of a normal chat (single agents already run
about 4x). Token usage alone explained about 80% of performance variance.

Use multi-agent only when:

- The task is breadth-first and decomposes into independent parallel
  directions (research, broad comparisons, large-scale review).
- The value per run justifies the token multiplier.
- Tasks with tight shared context or many inter-step dependencies are a
  poor fit; keep those single-agent.

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

## 16. Human-in-the-Loop

Two triggers should always hand control to a human (OpenAI's agent guide):

1. **Failure thresholds exceeded**: repeated retries, repeated
   misunderstanding of intent, budget exhaustion.
2. **High-risk actions**: irreversible, sensitive, or high-value operations
   (refunds, payments, cancellations, external communications) until
   confidence is earned.

Implementation notes:

- Model "contact a human" as a tool call that pauses the workflow and
  resumes on response (async approvals via Slack/email work well when state
  is checkpointed).
- Escalations must carry clean context: what was tried, what failed, what
  is pending. No raw transcript dumps.
- Review queues + sampled human review for medium-risk output; 100% review
  early in a launch, then ratchet down as evals earn trust.
- Capture feedback (thumbs, edits, escalation reasons) and feed it into the
  eval set and retrieval fixes. This flywheel is the actual moat.

---

# Part IV. Production Engineering

## 17. Reference Architecture

```text
                        ┌───────────────────────────┐
                        │        Client / API       │
                        └─────────────┬─────────────┘
                                      │
                        ┌─────────────▼─────────────┐
                        │ Edge / API Gateway        │
                        │ Auth · Rate limits · ACL  │
                        │ Request IDs · Validation  │
                        │ Input guardrails          │
                        └─────────────┬─────────────┘
                                      │
                  ┌───────────────────▼───────────────────┐
                  │ Application / Workflow Orchestrator   │
                  │ State machine · Budgets · Retries     │
                  │ Checkpoints · Pause/Resume            │
                  │ Idempotency · Human handoff           │
                  └───────┬─────────────────────┬─────────┘
                          │                     │
              ┌───────────▼──────────┐  ┌──────▼──────────────┐
              │ Retrieval Service    │  │ Tool Execution Layer │
              │ ACL filter (pre)     │  │ Least privilege      │
              │ Hybrid BM25+vector   │  │ Server-side validate │
              │ Reranker             │  │ Idempotency keys     │
              └───────────┬──────────┘  └──────┬──────────────┘
                          │                     │
                    ┌─────▼─────────────────────▼─────┐
                    │ Context Builder                  │
                    │ Token budget · Citations         │
                    │ Compaction · Prompt version      │
                    └──────────────┬───────────────────┘
                                   │
                    ┌──────────────▼───────────────────┐
                    │ Model Gateway                     │
                    │ Routing · Fallback · Prompt cache │
                    │ Timeouts · Circuit breakers       │
                    └──────────────┬───────────────────┘
                                   │
                    ┌──────────────▼───────────────────┐
                    │ Output Verification               │
                    │ Schema · Safety · Grounding       │
                    │ Output guardrails · Abstain       │
                    └──────────────┬───────────────────┘
                                   │
                    ┌──────────────▼───────────────────┐
                    │ Observability + Eval Pipeline     │
                    │ Traces (OTel GenAI) · Feedback    │
                    │ Eval sets ← production failures   │
                    └──────────────────────────────────┘
```

## 18. Core Design Rules

### Model every workflow as an explicit state machine

Represent each request as state that lives outside the model:

```python
class AgentState:
    request_id: str
    user_id: str
    conversation_id: str

    input: str
    retrieved_context: list[dict]
    tool_results: list[dict]

    step_count: int
    token_count: int
    estimated_cost_usd: float

    status: str           # running | waiting_human | done | failed | escalated
    checkpoint: dict      # enough to resume after crash or deploy
    final_answer: str | None
    error: str | None
```

Every transition needs a precondition, a timeout, a retry policy, a maximum
attempt count, structured logs, and a terminal failure or escalation path.

Two production requirements confirmed by Anthropic's multi-agent write-up
and 12-Factor Agents:

- **Checkpoint and resume.** Long-running agents cannot restart from zero on
  every failure. Persist state so the system resumes from the failure point.
- **Pause for humans as a first-class state.** Treat "ask a human" or
  "request approval" as a tool call that pauses the loop (possibly for
  hours) and resumes when the answer arrives.

### Enforce hard execution budgets

Never let a loop run unbounded. Unbounded consumption is an OWASP Top 10
risk (LLM10), not just a cost problem: it is also a denial-of-wallet attack
surface.

```python
MAX_STEPS = 8
MAX_TOOL_CALLS = 5
MAX_TOKENS = 12_000
MAX_COST_USD = 0.10
MAX_WALL_TIME_SECONDS = 30
```

On budget exhaustion: stop safely, return the best partial result or a
deterministic fallback, or escalate to a human. Never continue silently.

### Make side effects idempotent

Any action that changes the outside world (email, tickets, CRM updates,
payments, bookings, webhooks) must be safe to retry:

```text
idempotency_key = hash(user_id + operation_type + business_object_id)
```

Use a two-phase pattern for high-risk actions:

```text
prepare -> validate -> confirm (human if consequential) -> commit -> record
```

Never let model output directly authorize an irreversible action. Model
output selects the action; deterministic, authenticated code executes it.

### Keep business rules in code

If a rule can be written as deterministic logic (pricing, eligibility,
routing thresholds, permission checks), implement it in code and let the
model call it. LLMs are for the parts that cannot be hardcoded.

## 19. Reliability Patterns

### Timeouts at every boundary

| Layer | Examples |
|---|---|
| Client | Request deadline, cancellation |
| Gateway | Rate limit, auth timeout |
| Retrieval | Search and reranker deadline |
| Model | Time-to-first-token and total-generation timeout |
| Tools | Per-tool deadline |
| Workflow | End-to-end wall-clock deadline |

### Retries

- Retry only transient, safe failures: network timeouts, provider 5xx,
  rate-limit responses. Exponential backoff with jitter, capped attempts,
  preserve the original request ID.
- Never blindly retry non-idempotent writes, validation failures, or
  permission errors.
- When a tool fails, feed a **compact** error summary back into context so
  the model can self-correct, with a counter that escalates to a human
  after N consecutive failures (12-Factor Agents, factor 9).

### Fallback ladder

```text
Primary model unavailable
    ↓ Fallback model (same gateway, pinned versions)
    ↓ Cached answer, if still valid and authorized
    ↓ Safe deterministic response
    ↓ Human escalation
```

Fallback output must still pass schema, safety, and grounding checks. Test
degraded modes deliberately (provider outage game days), do not discover
them in production.

### Model gateway

Centralize routing, provider fallback, key management, pinned model
versions, prompt caching, circuit breakers, and per-tenant quotas in one
gateway layer instead of scattering provider logic through the codebase.

## 20. Security (mapped to OWASP Top 10 for LLM Applications, 2025)

| Risk | Core mitigation |
|---|---|
| LLM01 Prompt injection | Assume it succeeds; containment architecture, least privilege, human gates on consequential actions |
| LLM02 Sensitive information disclosure | Data minimization in context, output filtering, PII redaction in logs |
| LLM03 Supply chain | Pin and verify models, adapters, datasets, MCP servers, and libraries; scan third-party artifacts |
| LLM04 Data and model poisoning | Provenance and validation for training/fine-tuning data and RAG sources |
| LLM05 Improper output handling | Treat model output as untrusted input to downstream systems; sanitize before render/execute |
| LLM06 Excessive agency | Minimal tools, minimal permissions, confirmation for high-impact actions |
| LLM07 System prompt leakage | Assume the system prompt is public; never put secrets or auth logic in it |
| LLM08 Vector and embedding weaknesses | ACLs on vector stores, tenant isolation, index poisoning detection |
| LLM09 Misinformation | Grounding, citations, abstention, human review for high-stakes output |
| LLM10 Unbounded consumption | Budgets, quotas, per-run circuit breakers, anomaly alerts |

### Prompt injection: design for containment

"Separate instructions from data" is necessary but NOT sufficient; models
cannot reliably distinguish injected instructions inside data, and
classifier-based filters have been bypassed in shipped products (e.g., the
EchoLeak zero-click exfiltration in Microsoft 365 Copilot, CVE-2025-32711).
So:

- Apply Simon Willison's **lethal trifecta** rule: an agent that combines
  (1) access to private data, (2) exposure to untrusted content, and
  (3) the ability to communicate externally can be tricked into exfiltrating
  data. Break at least one leg by architecture: e.g., the agent that reads
  untrusted web/email content does not hold privileged tools or open egress.
- Everything influenced by untrusted input is itself untrusted, including
  the model's own output (chains into LLM05).
- Layer defenses: input classifiers, allowlisted egress, output validation,
  scoped credentials, human approval for consequential actions. Watch the
  research on capability-based designs (e.g., CaMeL) but do not bet the
  system on any single filter.
- Security controls live in deterministic, auditable code outside the
  model. A system prompt is not a security boundary.

Also standard hygiene: authenticate every request, authorize every
retrieval, encrypt in transit and at rest, redact secrets and PII in logs
and traces, audit privileged tool actions, define retention for prompts and
transcripts, keep credentials out of anything model-visible.

## 21. Evaluation Strategy

### Start small, start real

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

### Layer the graders

1. **Deterministic checks**: schema validity, citation presence, banned
   content, latency, cost. Cheap, run everywhere.
2. **LLM-as-judge** with an explicit rubric for fuzzy qualities
   (groundedness, helpfulness, tone). Known biases you must control for:
   position bias (swap answer order), verbosity bias (control for length),
   and self-preference (use a different model as judge where possible).
   Calibrate judges against human labels on a sample before trusting them.
3. **Human review** for high-stakes flows and for periodically re-anchoring
   the automated graders.

### Agent-specific evaluation

Judge the **end state** (did the task actually get done in the environment)
and the **trajectory** (tool choices, budget use, safety of intermediate
actions), while allowing multiple valid paths to the goal. Turn-by-turn
similarity to a golden transcript is the wrong metric for agents.

### Red teaming

Adversarial testing is part of evaluation, not a one-time audit: injection
attempts through every untrusted channel (documents, web content, tool
output, memory writes), permission-boundary probes, jailbreak suites, and
domain-specific abuse cases. Automate replay of every known attack as a
regression test and re-run on every model or prompt change.

### Run evals on every meaningful change

Prompts, model or provider version, retrieval/index config, embedding
model, chunking logic, tool schemas or permissions, workflow transitions,
guardrails, fine-tuned checkpoints. Gate deploys in CI on the regression
suite, then verify with shadow or canary traffic. Convert every production
incident into a regression case.

## 22. Observability

### Standardize on OpenTelemetry GenAI conventions

The OTel GenAI semantic conventions define a vendor-neutral schema
(gen_ai.* attributes; inference, tool-execution, and agent spans) covering
model, token usage, finish reasons, tool calls, and retrieval sources, and
are supported across major platforms. Adopting them keeps your telemetry
portable instead of locked to one vendor. The spec is still evolving
quickly; pin the convention version you emit.

### Propagate identifiers end to end

```text
request_id · trace_id · conversation_id · user_id · tenant_id
prompt_version · model_version · retrieval_index_version · workflow_version
```

### Monitor four layers, plus unit economics

| Layer | Example metrics |
|---|---|
| Availability | Error rate, provider failures, circuit-breaker state |
| Performance | P50/P95/P99 latency, time to first token, queue depth |
| Cost | Tokens, tool calls, cache hit rate, **cost per successful task** |
| Quality | Retrieval precision, grounded-answer rate, task success, escalation rate, feedback |

Never rely on one end-to-end metric; a stable overall success rate can hide
a retrieval regression compensated by model behavior. Store full sampled
transcripts (with PII controls) so non-deterministic failures can be
replayed and diagnosed; monitor agent decision patterns even where privacy
prevents content inspection.

## 23. Deployment and Change Management

- **Prompts are versioned artifacts**: reviewed, tested, and rolled back
  like code. Same for tool schemas and index configs.
- **Shadow, then canary, then progressive rollout** for model, prompt, and
  index changes.
- **Rainbow deployments for stateful agents**: long-running agents break if
  you swap code mid-flight. Shift traffic gradually to the new version and
  keep old versions alive until their in-flight runs finish (the pattern
  Anthropic uses for its research agents).
- Pin model versions; evaluate provider "upgrades" like any other change.
- Index and embedding migrations: dual-write, backfill, verify retrieval
  metrics, cut over, keep rollback.
- Fine-tuned checkpoints follow the same lifecycle: eval-gated promotion,
  versioned artifacts, rollback.

## 24. Cost and Latency Engineering

- **Prompt caching first.** Structure prompts as stable prefix (system,
  tools, reference docs) + variable suffix. Providers discount cached
  tokens heavily: Anthropic documents up to 90% cost and up to 85% latency
  reduction on long cached prompts (cache reads about 10% of input price);
  OpenAI caches automatically at roughly 50% cost reduction. This is often
  the single largest lever in RAG and agent systems.
- **Model routing / cascades**: default to a small model, escalate to a
  large one on low confidence or hard routes; use batch APIs (typically
  about 50% cheaper) for offline work.
- **Context discipline is cost discipline**: compaction, tool result
  clearing, and token-efficient tool responses directly cut spend.
- Stream tokens for perceived latency; parallelize independent tool calls;
  cap max output tokens per route.
- **Semantic caching**: fine for public FAQs; dangerous for personalized or
  authorized content. Include auth scope and tenant in the cache key or
  skip it.
- Track spend per route, per tenant, per feature, and alert on anomalies
  (see LLM10).

## 25. Inference and Serving (Self-Hosted)

Only relevant if §2 pointed you at self-hosting; skip otherwise.

### Serving stack

- Use a production inference engine (vLLM, SGLang, TensorRT-LLM class), not
  raw transformers loops. The wins that matter: **continuous batching**
  (new requests join in-flight batches), **paged/managed KV cache**, and
  **prefix caching** (shared system prompts computed once).
- At high concurrency the KV cache, not the weights, is usually the memory
  bottleneck: it grows with context length times batch size. Long-context
  workloads need this modeled explicitly.
- Rough sizing: FP16/BF16 weights take about 2 bytes per parameter; 4-bit
  quantization roughly quarters that. Then add KV cache headroom for your
  target batch and context.

### Quantization

- Weight quantization (8-bit, 4-bit: AWQ/GPTQ class, FP8 on supported
  hardware) is the standard cost lever. Quality loss is usually small on
  English benchmarks and **larger on smaller models and lower-resource
  languages**; measure on your own evals per language before shipping.
- KV cache quantization buys concurrency; same rule: measure.

### Latency levers

- Separate the two metrics: **TTFT** (time to first token, dominated by
  prefill) and **TPOT/ITL** (per-token decode speed). Different levers move
  each.
- Speculative decoding (draft model or self-speculative methods) can give
  2-3x decode speedups in favorable cases with unchanged outputs.
- Throughput and latency trade against each other through batch size;
  define SLOs per route and tune deliberately.

### Ops realities

- GPU autoscaling is slow (model load takes minutes): plan capacity with
  headroom and queue-depth alerts instead of assuming elastic scale-out.
- Load-test with realistic prompt/output length distributions; synthetic
  short prompts flatter every benchmark.
- Version the full serving stack (engine version, model build, quant
  config); engine upgrades change numerics and occasionally behavior, so
  they go through evals like any model change.

---

# Part V. Data Engineering

## 26. Pipelines and Dataset Management

- **Data quality is the ceiling** for retrieval, fine-tuning, and evals
  alike. Budget engineering time for parsing, deduplication, filtering,
  and PII scrubbing before budgeting for model work.
- Version datasets like code: content hashes, lineage (source, transform,
  date), changelogs, and reproducible builds. "Which data produced this
  model/index" must be answerable in minutes.
- Decontaminate: keep eval sets strictly out of training and few-shot
  pools; leakage produces beautiful dashboards and broken products.
- Define retention and deletion flows across every copy: raw store,
  processed sets, indexes, caches, fine-tuned weights trained on deleted
  data. Deletion requests must propagate.
- Automate quality gates in the pipeline (schema checks, language ID,
  length and dedup filters, toxicity/PII scans) and alert on distribution
  drift in incoming data.

## 27. Annotation Operations

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
- Vendors: run a paid pilot with your own QA before committing volume;
  contract for quality metrics and rework, not just throughput; keep
  ownership of guidelines, gold sets, and the delivered data.

## 28. Synthetic Data

- Best uses: augmenting rare classes and edge cases, generating format
  examples for SFT, building eval variants, and distillation traces from a
  strong teacher (check the teacher's terms of use).
- **Never ship unverified synthetic data.** Filter with deterministic
  checks plus an LLM judge, then human-sample. Generation is cheap;
  verification is the actual work.
- Keep real data in the mix and track the synthetic ratio. Training
  recursively on model output degrades quality and diversity (the "model
  collapse" failure mode documented in the literature).
- Label provenance: every synthetic example tagged as such, with generator
  model and prompt version, so it can be excluded or reweighted later.

## 29. Document Processing and OCR

For document-heavy systems, parsing quality sets the ceiling on everything
downstream; no retrieval trick recovers a table the parser destroyed.

- Route by document type: digital-native PDFs (extract text + layout),
  scanned documents (OCR), forms and tables (structure-aware extraction),
  images/figures (vision models where they matter).
- Preserve structure: headings hierarchy, tables as tables (not
  concatenated cells), reading order in multi-column layouts, page numbers
  and bounding boxes for citation back to the source.
- Vision-LLM parsing is a real option for messy layouts; weigh cost and
  hallucination risk against classical OCR + layout models, and eval
  extraction accuracy separately from end-to-end answer quality.
- Right-to-left scripts add specific failure modes: reading order, mixed
  RTL/LTR lines (Arabic text with Latin product names and numbers), digit
  shaping, diacritics. Test your parser on your real documents, not vendor
  samples.
- Keep per-page extraction confidence and route low-confidence pages to
  review instead of silently indexing garbage.

---

# Part VI. Specialized Systems

## 30. Real-Time Voice AI

Voice agents are a systems-latency problem first and a model-quality
problem second.

- **Pipeline**: VAD (voice activity detection) → streaming STT → LLM →
  streaming TTS, or an end-to-end speech-to-speech model. Pipelines give
  control, observability, and component-level swaps; speech-to-speech cuts
  latency and preserves prosody but reduces control. Most production
  systems still run pipelines.
- **Latency budget**: target well under a second voice-to-voice; humans
  read silence beyond that as failure. Stream every stage: partial STT into
  the LLM, first LLM sentence into TTS while the rest generates. Measure
  p95 voice-to-voice, not averages.
- **Turn-taking**: endpointing (when has the caller finished?) and barge-in
  (caller interrupts playback: stop TTS, cancel generation, keep state)
  make or break the experience. Test with real interruption patterns.
- **Reality of audio**: telephony codecs, background noise, code-switching,
  and dialects destroy lab WER numbers. Evaluate STT per dialect and per
  channel (phone vs web) on your own recordings.
- Everything from Parts III-IV still applies: budgets per call, tool
  confirmation for consequential actions, fallback to a human as a
  first-class flow (and legally required disclosure that the caller is
  talking to an AI in a growing number of jurisdictions, see §34).
- Voice-specific evals: WER/CER per dialect, endpointing errors,
  interruption handling, task completion, latency percentiles, and TTS
  quality (MOS-style human scoring plus pronunciation checks on domain
  vocabulary and names).

## 31. Multilingual and Arabic-Specific Notes

If your production traffic is Arabic (or any lower-resource language),
generic best practices need adjustments:

- **Token economics**: Arabic text frequently costs 1.5-3x more tokens than
  equivalent English on many tokenizers. Budget context, cost, and latency
  against your real language mix, not English benchmarks.
- **Lexical retrieval needs normalization**: alef/hamza variants, ta
  marbuta, diacritics, and rich morphology break naive BM25. Use
  Arabic-aware analyzers and normalization in the lexical leg of hybrid
  retrieval.
- **Choose embeddings/rerankers proven on your language**, and validate on
  your own eval set rather than English-centric leaderboards.
- **Dialect gap**: MSA-trained retrieval, STT, and judges often miss
  dialectal input. Add query rewriting (dialect to MSA) where it helps,
  build per-dialect eval slices, and test code-switching (Arabic + English
  technical terms), which is the norm in real Gulf traffic.
- **Generation checks**: RTL and mixed-direction rendering, Arabic-Indic vs
  Western numerals, citation formatting.
- **Regional models exist** (Arabic-centric families from Gulf institutions
  and labs) alongside multilingual frontier models; evaluate current
  versions on your own data instead of assuming either direction wins.
- **LLM-as-judge calibration is weaker outside English**: spot-check judges
  with native-speaker review before trusting automated scores.
- **Quantization and distillation hit low-resource languages harder** than
  English on average; re-run language-specific evals after any compression
  step (§25).

---

# Part VII. Product and Organization

## 32. UX Patterns for AI Products

- **Set expectations**: say what the system can and cannot do; label AI
  output as AI output (increasingly a legal requirement, §34).
- **Manage perceived latency**: stream tokens, show progress for agent
  steps ("searching policies...", "checking availability...") instead of a
  spinner.
- **Show your work**: citations that link to the exact source passage;
  confidence signals; visible tool actions before they execute.
- **Make "I don't know" a designed state**, not an error: abstention with a
  path forward (rephrase, escalate, provide a source) beats a confident
  wrong answer every time.
- **Keep the human in control**: editable outputs, undo where possible,
  explicit confirmation for consequential actions, always-available
  escalation to a person.
- **Harvest feedback where it is cheap**: thumbs with reason codes, edit
  capture (the user's correction is a free label), escalation reasons. Wire
  all of it into the eval set (§21).

## 33. Team Practices

- **Small full-stack teams beat siloed handoffs.** Two to four people who
  own prompt, retrieval, evals, and deployment for a use case move faster
  than a chain of specialists; embed with the product team rather than
  operating as an internal service desk.
- **Clear ownership**: every prompt, eval set, index, and model route has
  an owner; changes go through review like code (because they are code).
- **Evals are a first-class deliverable**: a use case is not "done" when
  the demo works; it is done when the eval suite exists and passes (see
  Definition of Done).
- **On-call includes AI failure modes**: runbooks for provider outages,
  quality regressions, injection incidents, and cost anomalies; postmortems
  produce new eval cases, not just action items.
- **Document decisions**: why this model, this chunking, this threshold.
  Six months later, the eval numbers behind a decision are the only defense
  against re-litigating it.
- Grow people through calibration: reviewing transcripts and grading eval
  outputs together is the fastest way to build shared judgment on a team.

## 34. Governance and Compliance (brief, but no longer optional)

- **EU AI Act** (if you serve EU users): in force since Aug 2024. The 2026
  Digital Omnibus amendment (adopted June 2026) moved stand-alone Annex III
  high-risk obligations to 2 Dec 2027 and Annex I embedded systems to
  2 Aug 2028, BUT most Article 50 transparency duties still apply from
  2 Aug 2026: chatbot disclosure, deepfake labeling, and machine-readable
  marking of AI-generated content (legacy systems get until 2 Dec 2026 for
  marking). GPAI model obligations have applied since Aug 2025. Map your
  system's risk category early.
- **NIST AI RMF + Generative AI Profile**: govern/map/measure/manage; a
  practical checklist source even outside the US.
- **ISO/IEC 42001** for organizations that want a certifiable AI management
  system.
- **Gulf/MENA deployments**: Saudi Arabia's SDAIA publishes AI Ethics
  Principles and Generative AI guidelines, and PDPL governs personal data;
  UAE and other GCC states have their own frameworks. Data residency
  requirements often decide your hosting architecture before anything else.
- Keep decision logs, model/data documentation (model cards, dataset
  sheets), and an AI incident-response runbook alongside the ordinary ops
  runbooks.

---

# Appendices

## A. Failure Handling Table

| Failure | Safe behavior |
|---|---|
| No relevant retrieval result | Abstain and ask for a source, or escalate |
| Conflicting sources | Explain the uncertainty and cite the conflicting documents |
| Model timeout | Retry safe request, use fallback model, or return partial safe result |
| Tool timeout | Never claim success; return pending/failure state |
| Tool validation failure | Reject the operation; feed a compact error back for one correction attempt |
| Repeated tool failures | Stop the loop and escalate to a human with clean context |
| Budget exceeded (steps/tokens/cost/time) | Stop the workflow, return best partial result or escalate |
| Permission failure | Deny without revealing whether the unauthorized data exists |
| Output schema failure | One retry with the error included, then deterministic fallback |
| Guardrail violation detected | Block, log, and route per policy; never "fix and forward" silently |
| Provider outage | Gateway fallback ladder (§19); degraded mode with honest messaging |
| Injection suspected | Contain: freeze privileged tools for the session, log full trace, review |
| Low STT confidence / unintelligible audio (voice) | Ask to repeat once, then offer human handoff |

## B. Production Readiness Checklist

### Approach and models
- [ ] Intervention ladder walked: cheaper fixes tried and eval-compared before expensive ones
- [ ] Model per route selected on own evals; gateway makes switching a config change
- [ ] API vs self-host decision documented (residency, economics, ops capacity)

### Architecture
- [ ] Simplest viable pattern chosen and justified by evals (ladder level documented)
- [ ] Workflows have explicit states, checkpoints, and terminal outcomes
- [ ] Every request has an end-to-end deadline
- [ ] Step, tool, token, cost, and retry budgets enforced (incl. per-run circuit breakers)
- [ ] External writes idempotent; two-phase commit for high-risk actions
- [ ] Pause/resume + human approval implemented as first-class states
- [ ] Business rules implemented deterministically

### RAG and data
- [ ] Parsing/OCR quality measured before blaming retrieval or the model
- [ ] Source, version, timestamp, and ACL metadata on every chunk
- [ ] Authorization filtering happens before retrieval
- [ ] Chunking appropriate per document type; contextual augmentation evaluated
- [ ] Hybrid retrieval + reranking measured on own corpus (not assumed)
- [ ] Answers carry verifiable citations; low-confidence requests abstain
- [ ] Retrieval metrics tracked separately from answer metrics
- [ ] Index/embedding migrations have dual-write and rollback plans
- [ ] Deletion propagates to indexes, caches, and derived datasets
- [ ] Checked whether RAG is even needed (small corpus + prompt caching?)

### Fine-tuning (if used)
- [ ] Prompt+RAG baseline documented; eval gap justifies tuning
- [ ] Training data versioned, deduplicated, decontaminated; held-out test set
- [ ] Before/after evals on target task AND general + safety suites
- [ ] Checkpoint promotion is eval-gated with rollback

### Security
- [ ] Threat model covers OWASP LLM Top 10 (2025)
- [ ] Lethal trifecta broken by architecture for every agent
- [ ] Model output treated as untrusted downstream; sanitized before render/execute
- [ ] Tools least-privilege, risk-rated, high-risk gated by confirmation
- [ ] Third-party MCP servers pinned, reviewed, credential-scoped
- [ ] No secrets in prompts; system prompt assumed public
- [ ] Code execution sandboxed with egress allowlist
- [ ] Vector store ACLs and tenant isolation verified
- [ ] Red-team suite exists and replays on every change

### Quality
- [ ] Representative eval set exists and grows from production failures
- [ ] LLM judges rubric-based and calibrated against human labels (bias controls in place)
- [ ] Agent evals score end state + trajectory, not transcript similarity
- [ ] Regression suite gates every prompt/model/index/tool change
- [ ] Shadow/canary path exists before full rollout

### Operations
- [ ] Traces follow OTel GenAI conventions; IDs propagate end to end
- [ ] P95/P99 latency, error-rate, and cost-anomaly alerts configured
- [ ] Cost per successful task tracked by route and tenant
- [ ] Prompt caching implemented; cache hit rate monitored
- [ ] Provider outage and degraded modes tested (game days)
- [ ] Rainbow/canary deployment for stateful agents
- [ ] Secrets and PII redacted from logs; retention rules defined
- [ ] Incident runbooks for model, retrieval, tool, and cost failures
- [ ] Regulatory mapping done (EU AI Act category, local requirements, residency)

### Voice (if applicable)
- [ ] P95 voice-to-voice latency within budget; all stages streaming
- [ ] Barge-in and endpointing tested with real interruption patterns
- [ ] STT evaluated per dialect and channel on own recordings
- [ ] AI disclosure and human handoff implemented

## C. Anti-Patterns

- Fine-tuning before prompting + retrieval baselines are exhausted
- Choosing models from leaderboards instead of your own evals
- Self-hosting for prestige while volume says the API is cheaper
- A single unrestricted agent with broad tool permissions
- Multi-agent by default (pay 15x tokens without a parallelizable task)
- No step, token, time, or cost budget; no per-run circuit breaker
- Framework abstraction you cannot debug; prompts you do not own
- Vector search without ACL filtering; semantic cache shared across tenants
- One universal chunk size for every document type
- Treating a high cosine score as proof of correctness
- Indexing documents your parser mangled, then tuning retrieval to compensate
- Sending full conversation history and all retrieved chunks every turn
- Using model output as direct authorization for external actions
- Treating the system prompt as a security control
- Relying on a single injection classifier instead of containment
- Importing third-party MCP servers/tools without review or version pinning
- Training on unverified synthetic data, or letting eval data leak into training
- Scaling annotation before inter-annotator agreement is measured
- Evaluating only hand-picked happy-path prompts, only offline
- Shipping prompt/model/index changes without regression testing
- Measuring latency while ignoring task success, grounding, and cost per success
- Hiding abstention: forcing an answer where "I don't know + escalate" is correct

## D. Metrics Quick Reference

| Area | Metric | Notes |
|---|---|---|
| Retrieval | recall@k, precision@k, MRR, nDCG | Measure against labeled query-passage pairs; per language/dialect slices |
| Generation | Faithfulness/groundedness, answer relevance, citation accuracy | LLM-judged with calibration; human-anchor periodically |
| Task | Task success rate, escalation rate, abstention rate | Define success per route; abstention is not failure |
| Agents | End-state success, trajectory quality, steps/tool calls per task | Allow multiple valid paths |
| Latency | TTFT, TPOT/ITL, end-to-end P50/P95/P99; voice-to-voice for voice | Different levers move TTFT vs TPOT (§25) |
| Cost | Cost per successful task, tokens per task, cache hit rate | The denominator is successes, not requests |
| Speech | WER/CER per dialect and channel, endpointing error, TTS MOS | Lab numbers do not transfer; use own recordings |
| Fine-tuning | Target-task delta, general-suite regression, safety-suite regression | All three, before promotion |
| Data | Inter-annotator agreement (kappa), gold-task accuracy, dedup/PII rates | Gate the pipeline on these |
| Ops | Error rate, provider failovers, budget-kill rate, cost anomalies | Budget kills are a signal, not just a safeguard |

## E. Definition of Done

A feature is production-ready only when it is:

1. **Bounded**: explicit time, token, cost, and execution limits.
2. **Observable**: every request traceable through retrieval, model, and tools.
3. **Evaluated**: passes a representative regression suite, online and offline, including adversarial cases.
4. **Safe**: authorization, validation, layered guardrails, containment of injection, rollback behavior.
5. **Recoverable**: retries, idempotency, checkpoints, fallbacks, and human escalation defined.
6. **Grounded**: knowledge answers expose the sources used to produce them.
7. **Deployable**: staged rollout (shadow/canary/rainbow) with rollback tested.
8. **Accountable**: data lineage, decision log, and regulatory mapping exist.

## F. Sources and Further Reading

### Architecture and agents
- Anthropic, Building Effective Agents: https://www.anthropic.com/engineering/building-effective-agents
- Anthropic, How we built our multi-agent research system: https://www.anthropic.com/engineering/multi-agent-research-system
- OpenAI, A Practical Guide to Building Agents: https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf
- 12-Factor Agents (HumanLayer): https://github.com/humanlayer/12-factor-agents
- Model Context Protocol spec and blog: https://modelcontextprotocol.io

### Context and retrieval
- Anthropic, Effective Context Engineering for AI Agents: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Anthropic, Introducing Contextual Retrieval: https://www.anthropic.com/engineering/contextual-retrieval
- Liu et al., Lost in the Middle (TACL 2024): https://arxiv.org/abs/2307.03172
- RAGAS evaluation framework: https://docs.ragas.io

### Tools
- Anthropic, Writing Effective Tools for Agents: https://www.anthropic.com/engineering/writing-tools-for-agents

### Security
- OWASP Top 10 for LLM Applications 2025: https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/
- Simon Willison, The Lethal Trifecta: https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/
- Beurer-Kellner et al., Design Patterns for Securing LLM Agents against Prompt Injections: https://arxiv.org/abs/2506.08837

### Evaluation and observability
- Zheng et al., Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena: https://arxiv.org/abs/2306.05685
- OpenTelemetry GenAI Semantic Conventions: https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/
- Hamel Husain, Your AI Product Needs Evals: https://hamel.dev/blog/posts/evals/

### Serving, fine-tuning, and data
- Kwon et al., Efficient Memory Management for LLM Serving with PagedAttention (vLLM): https://arxiv.org/abs/2309.06180
- Dettmers et al., QLoRA: https://arxiv.org/abs/2305.14314
- Rafailov et al., Direct Preference Optimization: https://arxiv.org/abs/2305.18290
- Shumailov et al., AI models collapse when trained on recursively generated data (Nature 2024): https://www.nature.com/articles/s41586-024-07566-y

### Broader references
- Chip Huyen, AI Engineering (O'Reilly, 2025)
- Eugene Yan, Patterns for Building LLM-based Systems and Products: https://eugeneyan.com/writing/llm-patterns/

### Governance
- EU AI Act + Digital Omnibus 2026 timeline analyses: https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/
- NIST AI RMF and Generative AI Profile: https://www.nist.gov/itl/ai-risk-management-framework
- ISO/IEC 42001: https://www.iso.org/standard/42001

### Practitioner threads (original Reddit grounding)
- Production-grade AI tools development (r/LocalLLaMA); Building RAG for production (r/Rag); RAG platform for 10M queries/day (r/softwarearchitecture); Context engineering as information architecture (r/LocalLLaMA); Building LLM workflows, observations (r/LocalLLaMA); Enterprise-grade RAG beyond vector DB + LLM (r/Rag)
