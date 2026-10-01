# 6. Structured Outputs

*Part II. Core Techniques · [Reading list](../README.md)*

- Use native structured outputs / constrained decoding for anything a
  machine will read: tool calls, extraction, classification, routing.
  Provider-native structured outputs enforce a JSON schema by masking
  invalid tokens; self-hosted serving gets the same guarantee from a
  constrained-decoding backend (XGrammar, Outlines, llguidance).
- Constrain the output envelope, not the thinking. Forcing a schema onto the
  model's reasoning field degrades quality; let it reason freely, then emit
  the structured object.
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

---

**Prev:** [5. Prompt Engineering That Survives Production](05-prompt-engineering.md) · [Reading list](../README.md) · **Next:** [7. Context Engineering](07-context-engineering.md)
