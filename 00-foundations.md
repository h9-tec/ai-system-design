# Foundations

A comprehensive reference for engineers, tech leads, and teams who build AI
systems: from choosing an approach, through core techniques (prompting, RAG,
fine-tuning, agents), to production engineering, data operations,
specialized systems, and governance.

This guide started from practitioner discussions (Reddit engineering
threads) and was validated and expanded against primary sources: Anthropic
and OpenAI engineering guides, the OWASP Top 10 for LLM Applications,
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

## How this reference is organized

The full text is split into an ordered reading list. Start here, then work
through the parts in sequence, or jump straight to the section you need from
the [reading list](README.md). Prefer one page? See [full-reference.md](full-reference.md).

**Start reading:** [1. The Intervention Ladder](part-1-decisions/01-intervention-ladder.md)
