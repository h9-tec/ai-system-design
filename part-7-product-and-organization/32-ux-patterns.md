# 32. UX Patterns for AI Products

*Part VII. Product and Organization · [Reading list](../README.md)*

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
  all of it into the [eval](../part-4-production-engineering/21-evaluation-strategy.md) set (§21).

---

**Prev:** [31. Multilingual and Arabic-Specific Notes](../part-6-specialized-systems/31-multilingual-and-arabic.md) · [Reading list](../README.md) · **Next:** [33. Team Practices](33-team-practices.md)
