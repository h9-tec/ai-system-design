# 14. Agent Memory

*Part III. Agentic Systems · [Reading list](../README.md)*

- **Working memory is the context window.** Manage it with the techniques
  in §7 (compaction, tool result clearing, sub-agents).
- **Persistent memory** is external: structured notes/files the agent
  writes and re-reads, or retrieval over past interactions. It splits into
  semantic (durable facts and preferences), episodic (what happened in past
  interactions), and procedural (learned how-to that updates the system
  prompt). Store decisions, stable preferences, constraints, and failed
  approaches, not raw transcripts; over-storage becomes permanent context
  pollution.
- **Off-the-shelf memory systems** handle extraction, storage, and retrieval:
  Mem0 (fact extraction and dedupe across vector, graph, and key-value stores),
  Letta (OS-style paging between the context window and an archival store), Zep
  (a temporal knowledge graph), and LangGraph Store / LangMem (namespaced
  memory native to a checkpointed graph). A provider memory tool paired with
  context editing can let the agent read and write a file-backed store while
  stale tool results are cleared automatically as the window fills.
- **Memory is an attack and privacy surface.** Poisoned memories persist
  across sessions (a stored injected instruction fires forever), so
  validate and constrain what gets written; give users visibility and
  deletion controls; apply the same ACL discipline as retrieval (§8).
- Expire or summarize aggressively. Memory systems degrade into noise
  without a curation policy.

---

**Prev:** [13. MCP (Model Context Protocol)](13-mcp.md) · [Reading list](../README.md) · **Next:** [15. Multi-Agent Systems (rarely justified)](15-multi-agent-systems.md)
