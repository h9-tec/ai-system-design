# 18. Core Design Rules

*Part IV. Production Engineering · [Reading list](../README.md)*

## Model every workflow as an explicit state machine

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

Two production requirements confirmed by Anthropic's [multi-agent](../part-3-agentic-systems/15-multi-agent-systems.md) write-up
and 12-Factor Agents:

- **Checkpoint and resume.** Long-running agents cannot restart from zero on
  every failure. Persist state so the system resumes from the failure point.
- **Pause for humans as a first-class state.** Treat "ask a human" or
  "request approval" as a tool call that pauses the loop (possibly for
  hours) and resumes when the answer arrives.

## Enforce hard execution budgets

Never let a loop run unbounded. Unbounded consumption is an [OWASP](20-security.md) Top 10
risk (Unbounded Consumption), not just a cost problem: it is also a denial-of-wallet attack
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

## Make side effects idempotent

Any action that changes the outside world (email, tickets, CRM updates,
payments, bookings, webhooks) must be safe to retry:

```text
idempotency_key = sha256("{user_id}|{operation_type}|{business_object_id}|{intent_id}")
```

Use a stable hash, not a language's per-process `hash()` (which is randomized
across runs, so replicas would disagree), and include a unique intent id so
distinct actions on the same object get distinct keys. Reuse a key only when
retrying that same action.

Use a two-phase pattern for high-risk actions:

```text
prepare -> validate -> confirm (human if consequential) -> commit -> record
```

Never let model output directly authorize an irreversible action. Model
output selects the action; deterministic, authenticated code executes it.

## Keep business rules in code

If a rule can be written as deterministic logic (pricing, eligibility,
routing thresholds, permission checks), implement it in code and let the
model call it. LLMs are for the parts that cannot be hardcoded.

---

**Prev:** [17. Reference Architecture](17-reference-architecture.md) · [Reading list](../README.md) · **Next:** [19. Reliability Patterns](19-reliability-patterns.md)
