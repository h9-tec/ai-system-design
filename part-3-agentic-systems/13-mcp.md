# 13. MCP (Model Context Protocol)

*Part III. Agentic Systems · [Reading list](../README.md)*

MCP is the open standard for connecting models to tools and data
(client-server), created by Anthropic, adopted across the major AI vendors and
developer tools, and governed vendor-neutrally by a foundation under the Linux
Foundation. Treat it as the default integration layer when you want tools
reusable across models and clients, instead of bespoke per-provider function
wiring.

What the protocol standardizes:

- **Three primitives:** tools (model-invoked actions), resources (readable
  context the client pulls in), and prompts (reusable templates the server
  offers).
- **Transport:** local integrations use a stdio transport; remote ones use a
  streamable HTTP transport that returns either a single JSON response or an
  SSE stream, and operates statelessly, so a server holds no per-connection
  state and scales horizontally, with any cross-call state passed as explicit
  arguments.
- **Auth:** authorization is optional and applies to the HTTP transport, where
  servers act as OAuth resource servers and access tokens are bound to the
  specific server they were issued for. A stdio server takes its credentials
  from the environment instead.

Production rules:

- **Third-party MCP servers are supply chain** ([OWASP](../part-4-production-engineering/20-security.md) Supply Chain). The
  protocol standardizes connectivity, not trust: a server can hide instructions
  in a tool description (tool poisoning), inject text that front-runs other
  tools in the prompt (line jumping), change a tool's behavior after you approve
  it (rug pull), or abuse its granted authority to act against another system or
  leak a token to an unintended audience (confused deputy, token passthrough).
  Pin versions, review the tool descriptions you import, and re-review on update.
- Scope credentials per server, allowlist which servers each agent may use,
  and prefer read-only servers wherever possible.
- Everything in §12 still applies: an MCP tool is still a tool. The
  protocol standardizes transport and discovery, not safety.
- Pin the protocol version and server versions you deploy against and track
  deprecations; version compatibility is part of the integration contract.

---

**Prev:** [12. Tools and the Agent-Computer Interface (ACI)](12-tools-and-aci.md) · [Reading list](../README.md) · **Next:** [14. Agent Memory](14-agent-memory.md)
