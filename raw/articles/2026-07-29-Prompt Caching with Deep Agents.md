---
title: "Prompt Caching with Deep Agents"
source_url: "https://www.langchain.com/blog/deep-agents-prompt-caching"
ingested: 2026-07-29
blog: "LangChain Blog"
published: "2026-06-26"
---
## Source Metadata
- **Blog:** LangChain Blog
- **Published:** 2026-06-26
- **URL:** https://www.langchain.com/blog/deep-agents-prompt-caching

## Article Content

Prompt Caching with Deep Agents

LangSmith Platform

Agent Improvement

Engine

Improve agents autonomously

Observability

See exactly what your agents are doing

Evaluation

Score and improve agent performance

Agent Infrastructure

Deployment

Ship and scale agents in production

Sandboxes

Run agent-generated code safely

No-Code Agents

Fleet

Agents for the whole company

Open Source Frameworks

deepagents

Build long-running agents for complex tasks

langgraph

Build reliable agents with low-level control

langchain

Quick start agents with any model provider

Resources

Customer Stories

Guides

Max Agency

How-To

LangChain Academy

YouTube

Documentation

Community

LangSmith for Startups

Meetups

Community

About

Careers

Partners

Deep Agents

Open Source

Prompt Caching with Deep Agents

Alex Olsen

June 26, 2026

5

min

Create agents

A powerful lever in running agents cost-efficiently at scale is Prompt Caching, a feature offered by model providers that can reduce the token cost of inference by 41-80%. As Manus AI puts it -

"If I had to choose just one metric, I'd argue that the KV-cache hit rate is the single most important metric for a production-stage AI agent."

However, model providers support varied strategies for controlling caching, making provider-agnostic caching a trickier solve.

Deep Agents is our general purpose, model-agnostic agent harness which supports prompt caching features across all major providers. We’re going to dig into how Deep Agents uses prompt caching to cut API costs, but first let’s look at how prompt caching reduces token costs in a chat model conversation.

TL;DR: prompt caching

The token cost of a chat model conversation grows quickly. For each new message, the model must reprocess every prior token in the conversation, including the:

System prompt

Tool descriptions

Loaded skills

Message history

New message

When we opt into prompt caching, the provider stores a snapshot of the model’s state after processing a prompt:

On the next request, the model picks up from that snapshot and only processes new text.

However, loading a new skill or tool can modify our prompt earlier in the conversation, potentially causing a cache bust. Some model providers enable us to add explicit cache breakpoints earlier in the prompt, resulting in a cache hit on a subset of the prompt rather than a full cache bust. However, not all model providers support explicit cache breakpoints:

Anthropic

OpenAI

Gemini

AWS Bedrock

Fireworks

Explicit Breakpoints

Per-provider

Explicit caching is also just one prompt caching feature with varied support among providers:

Anthropic

OpenAI

Gemini

AWS Bedrock

Fireworks

Explicit Breakpoints

Per-provider

Configurable TTL

Per-model

Per-provider

Cache Prewarm

Anthropic

Routing Key

OpenAI

The prompt caching feature support landscape changes quickly. Be sure to check model provider docs for reference on feature support.

Between differing prompt caching implementations and feature support among providers, it can be a challenge to achieve maximal cost savings across providers.

How we’re solving this in Deep Agents

The Deep Agents harness makes a best-effort attempt at utilizing prompt caching features by automatically:

Setting explicit cache breakpoints when supported

Opting in to provider-side implicit caching when explicit breakpoints aren’t supported

Structuring your prompt to maximize cache reads

These strategies are supported for all major providers, so you’re able to switch provider at any time and still reap maximal token savings. To take advantage of provider-specific features, the harness detects the current model provider and delegates caching to provider-specific middleware. You can also use the middleware in your own createAgent() to opt in to prompt caching savings:

// In Deep Agents you get prompt caching for free!

const agent = createDeepAgent({ model: 'gpt-5.5' });

// In LangChain, opt in via our middleware:

const agent = createAgent({

model: 'claude-haiku-4-5-20251001',

middleware: [anthropicPromptCachingMiddleware()],

‍The Deep Agents harness also structures your prompt and explicit cache points to minimize cache degradation. Optimally the static prefix (your tool descriptions, skills, system prompt) in a model invocation remains static. It can however change when doing things like updating a memory or compacting a conversation, leading to a cache bust. Deep Agents minimizes the blast radius by structuring your prompt and explicit cache points such that if e.g. a memory is updated, you still get a cache read on a subset of your prompt.

The real savings of prompt caching

Feature tables tell us what's possible. To see what prompt caching actually saves, we ran the Deep Agents eval suite across a mid-tier model from each of three providers: claude-haiku-4-5, gpt-5.4-mini, and gemini-3.5-flash. The result is the chart below. On real agent trajectories, prompt caching cut token cost by 49–80%.

claude-haiku-4-5: -77%. Using Anthropic's explicit breakpoints, we can keep a large portion of the prompt cached. This significantly reduced the token cost of each request.

gpt-5.4-mini: -80%. OpenAI's automatic longest-prefix caching gives us a sizable 80%  cost reduction

gemini-3.5-flash: -49%. Gemini's implicit caching makes no explicit savings guarantee, but we still see considerable savings

It's also worth noting that caching pays off more the longer a conversation runs: the cached prefix is reused across every turn, so the long-horizon tasks are the ones that benefit most.

Observability with LangSmith

Cost savings from prompt caching are only as good as your ability to measure them. LangSmith offers visibility into API cost, cache reads, and token usage at a per-invocation and per-trajectory level:

For each invocation you get time-to-first-token, total input tokens, cache-read tokens, and total output tokens rolled up to a per-trajectory aggregate. Because cache reads are itemized separately, you can see exactly how much of each prompt was served from cache rather than reprocessed.

This is also how we produced the numbers in this post:

Run the Deep Agents eval suite against each agent configuration

Inspect trace data in the LangSmith dashboard to verify run results

Pull the run data via the LangSmith Client SDK

Compute per-provider cost deltas by dropping the data into a Jupyter notebook (or have an agent use LangSmith Skills to help)

LangSmith lets us disentangle savings from caching, trajectory length, and cheaper turns, which can inform how we optimize our agent. More on how to read and act on data in LangSmith here.

Next in prompt caching

Model providers have yet to converge on a common feature set for prompt caching. Explicit breakpoints drove some savings above, but it’s only the start. A handful of other features - cache prewarm, routing keys, configurable TTL - stand to unlock further cost savings and  latency wins.

You can take advantage of the currently-supported features today by using createDeepAgent - no additional config needed. As model providers add additional feature support, we’ll continue to fold them into the existing harness.

Related content

Open Source

Observability & Evals

How We Benchmark Deep Agents

Nick Hollon

Harrison Chase

July 23, 2026

4

min

Open Source

LangGraph

Agent Architecture

3 Years of Graph Engineering with LangGraph

Sydney Runkle

Harrison Chase

July 22, 2026

7

min

Deep Agents

Agent Architecture

Open Source

How to Use RLMs in Deep Agents

Sydney Runkle

July 1, 2026

8

min
