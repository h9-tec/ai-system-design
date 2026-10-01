# 3. Model Selection

*Part I. Decisions Before Code · [Reading list](../README.md)*

- **Your [evals](../part-4-production-engineering/21-evaluation-strategy.md) decide, not leaderboards.** Public benchmarks suffer
  contamination and rarely match your task, language mix, or latency
  budget. Build a 50-200 case eval from real traffic and run every
  candidate through it.
- Pick per route: capability, latency, and cost form a triangle; different
  routes in the same product should use different models. Reasoning effort
  (the thinking budget) is part of that triangle; dial it per route, not
  globally.
- Weigh open-weight against proprietary as a real axis: open weights buy
  self-hosting, a frozen version, and no training on your data; proprietary
  often leads on frontier capability. Decide per route, not per company.
- Test what marketing numbers hide: effective context recall at depth (not
  the advertised window), structured-output reliability, tool-calling
  quality under your real schemas, refusal behavior on your domain, and
  quality on YOUR languages and dialects.
- Check the boring parts: license and usage terms (including whether
  outputs can train competitors' models), deprecation policy, rate limits,
  regional availability, [fine-tuning](../part-2-core-techniques/10-fine-tuning.md) access.
- Re-run selection periodically; the model landscape shifts quarterly, and
  a gateway makes re-selection cheap.

---

**Prev:** [2. API vs Self-Hosting](02-api-vs-self-hosting.md) · [Reading list](../README.md) · **Next:** [4. Build vs Buy](04-build-vs-buy.md)
