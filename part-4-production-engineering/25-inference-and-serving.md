# 25. Inference and Serving (Self-Hosted)

*Part IV. Production Engineering · [Reading list](../README.md)*

Only relevant if §2 pointed you at [self-hosting](../part-1-decisions/02-api-vs-self-hosting.md); skip otherwise.

## Serving stack

- Use a production inference engine (vLLM, SGLang, TensorRT-LLM class), not
  raw transformers loops. The wins that matter: **continuous batching**
  (new requests join in-flight batches), **paged/managed KV cache**, and
  **prefix caching** (shared system prompts computed once).
- **Prefill/decode disaggregation** splits the compute-bound prefill phase
  and the memory-bound decode phase onto separate instance pools, streaming
  the KV cache between them, so each scales and tunes independently. This is
  the standard shape for high-throughput serving at scale.
- At high concurrency the KV cache, not the weights, is usually the memory
  bottleneck: it grows with context length times batch size. Long-context
  workloads need this modeled explicitly. Offloading or tiering the KV cache
  to CPU or a remote store, and sharing it across requests, extends prefix
  reuse beyond a single node.
- Rough sizing: FP16/BF16 weights take about 2 bytes per parameter; 4-bit
  quantization roughly quarters that. Then add KV cache headroom for your
  target batch and context.

## Quantization

- Weight quantization (8-bit, 4-bit: AWQ/GPTQ class, FP8 and FP4 on
  supported hardware) is the standard cost lever. Quality loss is usually small on
  English benchmarks and **larger on smaller models and lower-resource
  languages**; measure on your own [evals](21-evaluation-strategy.md) per language before shipping.
- KV cache quantization buys concurrency; same rule: measure.

## Latency levers

- Separate the two metrics: **TTFT** (time to first token, dominated by
  prefill) and **TPOT/ITL** (per-token decode speed). Different levers move
  each.
- Speculative decoding (draft model, self-speculative, or EAGLE-style
  methods where the model drafts its own next tokens) can give 2-4x decode
  speedups in favorable cases with unchanged outputs.
- Throughput and latency trade against each other through batch size;
  define SLOs per route and tune deliberately.

## Ops realities

- GPU autoscaling is slow (model load takes minutes): plan capacity with
  headroom and queue-depth alerts instead of assuming elastic scale-out.
- Load-test with realistic prompt/output length distributions; synthetic
  short prompts flatter every benchmark.
- Version the full serving stack (engine version, model build, quant
  config); engine upgrades change numerics and occasionally behavior, so
  they go through evals like any model change.

---

**Prev:** [24. Cost and Latency Engineering](24-cost-and-latency.md) · [Reading list](../README.md) · **Next:** [26. Pipelines and Dataset Management](../part-5-data-engineering/26-pipelines-and-datasets.md)
