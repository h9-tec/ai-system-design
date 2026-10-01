# F. Sources and Further Reading

*Appendices · [Reading list](../README.md)*

## Architecture and agents

- Anthropic, Building Effective Agents: https://www.anthropic.com/engineering/building-effective-agents
- Anthropic, How we built our [multi-agent](../part-3-agentic-systems/15-multi-agent-systems.md) research system: https://www.anthropic.com/engineering/multi-agent-research-system
- Anthropic, Code execution with MCP: https://www.anthropic.com/engineering/code-execution-with-mcp
- OpenAI, A Practical Guide to Building Agents: https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf
- Cognition, Don't Build Multi-Agents: https://cognition.com/blog/dont-build-multi-agents
- 12-Factor Agents (HumanLayer): https://github.com/humanlayer/12-factor-agents
- [Model Context Protocol](../part-3-agentic-systems/13-mcp.md) spec and blog: https://modelcontextprotocol.io

## Agent memory

- Anthropic, Managing context (memory tool and context editing): https://claude.com/blog/context-management
- Mem0: https://github.com/mem0ai/mem0
- Letta: https://github.com/letta-ai/letta
- Zep (temporal knowledge-graph memory): https://www.getzep.com

## Context and retrieval

- Anthropic, Effective [Context Engineering](../part-2-core-techniques/07-context-engineering.md) for AI Agents: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Anthropic, Introducing Contextual Retrieval: https://www.anthropic.com/engineering/contextual-retrieval
- Jina AI, Late Chunking: https://jina.ai/news/late-chunking-in-long-context-embedding-models/
- Liu et al., Lost in the Middle (TACL 2024): https://arxiv.org/abs/2307.03172
- RAGAS [evaluation](../part-4-production-engineering/21-evaluation-strategy.md) framework: https://docs.ragas.io

## Tools and structured outputs

- Anthropic, Writing Effective Tools for Agents: https://www.anthropic.com/engineering/writing-tools-for-agents
- Outlines (constrained decoding): https://github.com/dottxt-ai/outlines
- XGrammar (constrained decoding): https://github.com/mlc-ai/xgrammar

## Security

- [OWASP](../part-4-production-engineering/20-security.md) Top 10 for LLM Applications: https://genai.owasp.org/llm-top-10/
- OWASP GenAI Security Project (including the Top 10 for Agentic Applications): https://genai.owasp.org/
- MITRE ATLAS (adversarial threat landscape for AI systems): https://atlas.mitre.org
- Simon Willison, The Lethal Trifecta: https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/
- Invariant Labs, MCP tool-poisoning attacks: https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks
- EchoLeak (CVE-2025-32711), zero-click exfiltration in Microsoft 365 Copilot: https://www.cve.org/CVERecord?id=CVE-2025-32711
- Beurer-Kellner et al., Design Patterns for Securing LLM Agents against Prompt Injections: https://arxiv.org/abs/2506.08837

## Evaluation and observability

- Zheng et al., Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena: https://arxiv.org/abs/2306.05685
- DeepEval: https://github.com/confident-ai/deepeval
- promptfoo: https://www.promptfoo.dev
- Langfuse: https://github.com/langfuse/langfuse
- Arize Phoenix: https://github.com/Arize-ai/phoenix
- OpenTelemetry GenAI Semantic Conventions: https://opentelemetry.io/docs/specs/semconv/registry/attributes/gen-ai/
- Hamel Husain, Your AI Product Needs Evals: https://hamel.dev/blog/posts/evals/

## Serving and inference

- Kwon et al., Efficient Memory Management for LLM Serving with PagedAttention (vLLM): https://arxiv.org/abs/2309.06180
- vLLM: https://github.com/vllm-project/vllm
- SGLang: https://github.com/sgl-project/sglang
- NVIDIA Dynamo (disaggregated prefill/decode serving): https://github.com/ai-dynamo/dynamo

## Fine-tuning and data

- Dettmers et al., QLoRA: https://arxiv.org/abs/2305.14314
- Rafailov et al., Direct Preference Optimization: https://arxiv.org/abs/2305.18290
- Ethayarajh et al., KTO: Model Alignment as Prospect Theoretic Optimization: https://arxiv.org/abs/2402.01306
- Hong et al., ORPO: Monolithic Preference Optimization without Reference Model: https://arxiv.org/abs/2403.07691
- Shao et al., DeepSeekMath (introduces GRPO): https://arxiv.org/abs/2402.03300
- OpenAI, Reinforcement Fine-Tuning cookbook: https://developers.openai.com/cookbook/examples/reinforcement_fine_tuning
- Qwen3 Embedding technical report: https://arxiv.org/abs/2506.05176
- Docling (document parsing): https://github.com/docling-project/docling
- Shumailov et al., AI models collapse when trained on recursively generated data (Nature 2024): https://www.nature.com/articles/s41586-024-07566-y

## Specialized systems

- OpenAI, Realtime API guide (speech-to-speech): https://developers.openai.com/api/docs/guides/realtime

## Broader references

- Chip Huyen, AI Engineering (O'Reilly, 2025)
- Eugene Yan, Patterns for Building LLM-based Systems and Products: https://eugeneyan.com/writing/llm-patterns/

## Governance

- EU AI Act (European Commission regulatory framework): https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai
- EU AI Act and Digital Omnibus timeline analysis: https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/
- NIST AI Risk Management Framework: https://www.nist.gov/itl/ai-risk-management-framework
- NIST AI 600-1, Generative AI Profile: https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf
- ISO/IEC 42001: https://www.iso.org/standard/42001

## Practitioner threads (original Reddit grounding)

- Production-grade AI tools development (r/LocalLLaMA); Building [RAG](../part-2-core-techniques/08-rag-system-design.md) for production (r/Rag); RAG platform for 10M queries/day (r/softwarearchitecture); Context engineering as information architecture (r/LocalLLaMA); Building LLM workflows, observations (r/LocalLLaMA); Enterprise-grade RAG beyond vector DB + LLM (r/Rag)

---

**Prev:** [E. Definition of Done](e-definition-of-done.md) · [Reading list](../README.md)
