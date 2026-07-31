---
title: "Tuning the harness, not the model: a Nemotron 3 Ultra playbook"
source_url: "https://www.langchain.com/blog/tuning-the-harness-not-the-model-a-nemotron-3-ultra-playbook"
ingested: 2026-07-30
blog: "LangChain Blog"
published: "2026-07-08"
---
## Source Metadata
- **Blog:** LangChain Blog
- **Published:** 2026-07-08
- **URL:** https://www.langchain.com/blog/tuning-the-harness-not-the-model-a-nemotron-3-ultra-playbook

## Article Content

Tuning the harness, not the model: a Nemotron 3 Ultra playbook

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

Case Studies

Partner

Tuning the harness, not the model: a Nemotron 3 Ultra playbook

Nick Hollon

Srimanth Tangedipalli

July 8, 2026

11

min

Create agents

Key Takeaways

Near-frontier agent quality at a fraction of the cost. Tuning the harness alone took Nemotron 3 Ultra to a best run of 0.86 on the Deep Agents suite, nearly matching Opus 4.8's best of 0.87, at roughly 10x lower cost per run (about $4.48 against $43.48 on the full suite) with latency at parity.‍

Evals are the training data for harness work. Every change ran through a trace-driven loop, screened cheaply first, and earned its place only if the win repeated across trials and regressed nothing else.‍

Fit decides how much capability reaches the task. A matched harness lets the model spend its capability on the work; a mismatched one makes it fight the scaffolding, and the gap between the two shows up in the score without touching the weights.‍

Harness tuning has a ceiling. It fixes failures that come from the scaffolding, but it can't add what isn't in the weights, so a result that stays flat through every harness change points to post-training rather than another hook.

An agent is a model plus a harness. The model does the thinking, and the harness (the system prompt, the tool descriptions, the middleware) is the scaffolding it works inside. We've tuned harnesses around frontier models before, but, this time, we wanted to see how far we could get with an open model.

Open models are where this gets interesting. They've gotten good enough to take seriously for real agent work, and they cost a fraction of a frontier API. You get the weights, so you can host and fine-tune the model yourself, or you can use an endpoint from a variety of Cloud providers without lock-in. The catch is that a capable model can still underperform in a harness that wasn't built for it, which is the part we set out to fix.

As a member of the Nemotron Coalition, we thought Nemotron 3 Ultra was the right model to tune inside Deep Agents. NVIDIA built Nemotron to work inside agent harnesses, and we wanted to see how far we could take it.

The harness is the part you control

Out of the box, a generic harness is not tuned to the model. Using a model without harness tuning is a reasonable default but not best you can do.

The harness is everything around the model, and the model is the engine inside it. When the two are matched, the model spends its capability on the task. When they are not, it spends capability fighting the scaffolding, re-asking for details it already has, stopping early, or looping.

The fit matters more than most people expect, and we've shown it before. On Terminal-Bench 2.0, we took gpt-5.2-codex from 52.8 to 66.5, roughly Top 30 to Top 5 at the time, without touching the model. When we shipped per-model harness profiles, we improved a curated subset of tau2-bench by 10 to 20 points by conforming to prompting guides. The same weights with different scaffolding lead to a different score.

We did that harness-side work using a data-driven approach, mining traces for failure patterns. The case study is Nemotron 3 Ultra, an open model that already comes a long way on its own, because NVIDIA post-trained it specifically to behave consistently across agent harnesses, not just single-turn chat, on a large suite of long-running, tool-using tasks (NVIDIA's launch post covers the agentic post-training and the architecture behind it).
