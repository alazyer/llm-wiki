---
title: "KV Caching in LLMs, Clearly Explained"
source: "https://x.com/_avichawla/status/2034902650534187503"
author:
  - "[[@_avichawla]]"
published: 2026-03-20
created: 2026-07-13
description: "You must have seen it every time you use ChatGPT or Claude that the first token takes noticeably longer to appear. Then the rest stream out ..."
tags:
  - "clippings"
---
![[6aee818c350d285c34dfeb26ec76b899_MD5.jpg]]

You must have seen it every time you use ChatGPT or Claude that the first token takes noticeably longer to appear. Then the rest stream out almost instantly.

Behind the scenes, it's a deliberate engineering decision called KV caching, and the purpose is to make LLM inference faster.

Before we get into the technical details, here's a side-by-side comparison of LLM inference with and without KV caching:

<video preload="none" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/amplify_video_thumb/2034896425310724097/img/y4A-fhJoCZIj_63a.jpg" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"><source type="video/mp4" src="blob:https://x.com/eb23b28d-59f2-446f-8de1-a4a071d30c81"></video>

0:10 / 0:47

Now let's understand how it works, from first principles.

## Part 1: How LLMs generate tokens

The transformer processes all input tokens and produces a hidden state for each one. Those hidden states get projected into vocabulary space, producing logits (one score per word in the vocabulary).

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HD1nzZmboAE8879.jpg" src="https://video.twimg.com/tweet_video/HD1nzZmboAE8879.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

GIF

But only the logits from the last token matter. You sample from them, get the next token, append it to the input, and repeat.

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HD1n36gbAAASEmH.jpg" src="https://video.twimg.com/tweet_video/HD1n36gbAAASEmH.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

GIF

This is the key insight: to generate the next token, you only need the hidden state of the most recent token. Every other hidden state is an intermediate byproduct.

## Part 2: What Attention actually computes

Inside each transformer layer, every token gets three vectors: a query (Q), a key (K), and a value (V). Attention multiplies queries against keys for scores, then uses those scores to weight the values.

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HD1n9AMboAEkHaT.jpg" src="https://video.twimg.com/tweet_video/HD1n9AMboAEkHaT.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

GIF

Now focus on just the last token.

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HD1oErLa0AAp8FL.jpg" src="https://video.twimg.com/tweet_video/HD1oErLa0AAp8FL.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

![[4bbfdcdc555a453bc0641a1d5ed8c81f_MD5.jpg]]

GIF

The last row of QK^T uses:

- The query vector of the last token
- All key vectors in the sequence

The final attention output for that row uses:

- The same query vector
- All key and value vectors

So to compute the only hidden state we need, every attention layer requires Q from the latest token, and K and V from everything.

## Part 3: The redundancy involved

Generating token 50 requires K and V vectors for tokens 1 through 50. Generating token 51 requires K and V vectors for tokens 1 through 51.

The K and V vectors for tokens 1 through 49 were already computed. They haven't changed. Same inputs, same outputs. Yet the model recomputes them from scratch every step.

![[0a7fc035b04de5c86fb6ba3b1cc9010b_MD5.png]]

That's O(n) redundant work per step. Over an entire generation, O(n²) wasted compute.

## Part 4: The fix

Instead of recomputing all K and V vectors at every step, store them. For each new token:

1. Compute Q, K, and V for only the newest token.
2. Append the new K and V to the cache.
3. Retrieve all previous K and V vectors from the cache.
4. Run attention using the new Q against the full cached K and V.

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HD1oNGXaUAAtPkt.jpg" src="https://video.twimg.com/tweet_video/HD1oNGXaUAAtPkt.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

![[87aef1c369ed6d5fb4c05de67fa64858_MD5.jpg]]

GIF

That's KV caching. One new K and one new V per layer per step. Everything else comes from memory.

The attention computation still scales with sequence length (you're attending over all keys and values). But the expensive projections to produce K and V happen only once per token, not once per step.

## Part 5: Time-to-First-Token

Now you can see why the first token is slow.

When you send a prompt, the model processes the entire input in one forward pass, computing and caching K and V vectors for every token. This is the prefill phase, and it's the most compute-intensive part of the request.

Once the cache is warm, each subsequent token needs only a single forward pass with one token.

![[c33efffebfe0831cffa748546f584201_MD5.jpg]]

That initial delay is called time-to-first-token (TTFT). Longer prompts mean longer prefills, which mean longer waits. Optimizing TTFT (chunked prefill, speculative decoding, prompt caching) is its own deep topic, but the dynamic is always the same: building the cache is expensive, reading from it is cheap.

## Part 6: The Tradeoff

KV caching trades compute for memory. Every layer stores K and V vectors for every token. For Qwen 2.5 72B (80 layers, 32K context, hidden dim 8192), the KV cache for a single request can consume several gigabytes of GPU memory. At hundreds of concurrent requests, it often exceeds the model weights themselves.

This is why grouped-query attention (GQA) and multi-query attention (MQA) exist: share key/value heads across query heads, cut memory, and minimal quality loss.

It's also why doubling context length is hard. Double the window, double the KV cache per request, fewer concurrent users.

There is another idea called Paged attention, which solves this, and I talked about it here recently:

> Mar 11

## tl;dr

KV caching eliminates redundant computation during autoregressive generation. Previous tokens always produce the same K and V vectors, so you compute them once and store them. Each new token only needs its own Q, K, and V. Then, attention runs against the full cache.

5x speedup in practice. The cost is GPU memory, which becomes the binding constraint at scale. Every LLM serving stack (vLLM, TGI, TensorRT-LLM) builds on this idea.

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HD1oaiiboAAJwwE.jpg" src="https://video.twimg.com/tweet_video/HD1oaiiboAAJwwE.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

![[bd1421a8a9cce51cd4bb5db5e7c3b34d_MD5.jpg]]

GIF

That's a wrap!

If you enjoyed this tutorial:

Find me → [@\_avichawla](https://x.com/@_avichawla)

Every day, I share tutorials and insights on DS, ML, LLMs, and RAGs.