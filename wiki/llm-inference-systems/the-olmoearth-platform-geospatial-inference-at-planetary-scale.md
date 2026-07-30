# The OlmoEarth Platform: Geospatial Inference at Planetary Scale

## Summary
Allen AI (which powers Hugging Face's models) has built the OlmoEarth platform to handle geospatial inference workloads at planetary scale. This article describes the infrastructure designed to run large language models on Earth observation and geospatial data—a workload that differs significantly from traditional NLP due to the high-dimensional, grid-like nature of satellite imagery and the massive volume of global datasets. OlmoEarth addresses key challenges in scaling LLM inference for geospatial applications: model parallelism across thousands of nodes, efficient handling of multi-modal inputs (text + raster data), and latency-sensitive deployment for near-real-time environmental monitoring use cases. The piece covers architectural decisions around cluster orchestration, data locality optimizations, and cost-aware routing patterns that make it feasible to run large foundation models over petabytes of Earth observation data.

## Key Takeaways
- Geospatial inference requires fundamentally different architecture than standard LLM serving due to multi-modal input structure (text prompts combined with 2D/3D raster data grids).
- Data locality is critical—the pipeline brings computation to the data by colocating inference workers near where geospatial datasets are stored, minimizing network egress costs.
- Model parallelism strategies have been adapted to handle both textual reasoning layers and spatial feature extraction modules that operate differently than standard transformer layers.
- Cost-aware routing directs different types of geospatial queries through appropriate model tiers: lightweight models for routine land-cover classification and heavier multimodal models for complex cross-modal analysis tasks.
- The platform integrates tightly with Hugging Face's model ecosystem, allowing users to fine-tune and deploy geospatial-specific variants of foundational models directly through the HF Hub.

## Related
- [[llm-inference-systems/kv-caching-in-llms]]
- [Source: Hugging Face Blog](https://huggingface.co/blog/allenai/olmoearth-infrastructure)