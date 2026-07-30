# LLM Inference Systems

LLM inference systems covers the runtime mechanics and serving tradeoffs that determine latency, throughput, memory use, and deployment scale for large language models.

## Articles
- [[kv-caching-in-llms|KV Caching in LLMs]] — Why autoregressive generation stores key/value tensors, how prefill creates TTFT, and why memory becomes the serving bottleneck.
- [[the-olmoearth-platform-geospatial-inference-at-planetary-scale|The OlmoEarth Platform: Geospatial Inference at Planetary Scale]] — Infrastructure for running large language models over Earth observation data at global scale, with adaptations for multi-modal input, data locality, and cost-aware routing patterns.
- [[lfm2-5-encoders-fast-long-context-inference-on-cpu|LFM2.5-Encoders for Fast Long-Context Inference on CPU]] — Efficient encoder-only models optimized for long-document understanding tasks on commodity CPUs, delivering 3.7x speedups over ModernBERT at 8K sequence length for always-on classification workloads.
