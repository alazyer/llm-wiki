# KV Caching in LLMs

## Summary
KV caching speeds up autoregressive LLM generation by storing each token’s attention keys and values after they are computed once. New tokens still attend over the full prior sequence, but the model avoids recomputing unchanged K/V projections at every generation step.

## Key Takeaways
- During generation, only the latest token’s logits are needed to sample the next token, but attention for that token still needs prior keys and values.
- Without caching, each step recomputes K and V vectors for all previous tokens, causing redundant $O(n^2)$ work across a full generation.
- KV caching stores prior K/V tensors per layer; each step computes Q/K/V only for the newest token and attends against the cached K/V history.
- Time-to-first-token is slower because the prefill phase processes the whole prompt and builds the initial cache before streaming can begin.
- Longer prompts increase prefill cost, while longer context windows increase KV-cache memory per request.
- The tradeoff is compute for memory: high concurrency can make KV cache memory exceed model weight memory.
- GQA, MQA, paged attention, prompt caching, chunked prefill, and speculative decoding are serving optimizations around this bottleneck.
- Production serving stacks such as vLLM, TGI, and TensorRT-LLM build around KV-cache management.

## Related
- [[agent-harness-engineering/self-learning-agent-layers]]
- [[ai-agent-operations/local-coding-agents-with-hermes]]
- [[loop-engineering/getting-started-with-loops]]
