---
title: "Building LangGraph: Designing an Agent Runtime from first principles"
source_url: "https://www.langchain.com/blog/building-langgraph"
ingested: 2026-07-26
blog: "LangChain Blog"
published: "2026-06-15"
---

## Source Metadata

- **Blog:** LangChain Blog
- **URL:** https://www.langchain.com/blog/building-langgraph
- **Fetched:** 2026-07-26
- **Extracted title:** Building LangGraph: Designing an Agent Runtime from first principles

## Article Content

Building LangGraph: Designing an Agent Runtime from first principles

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

LangGraph

Building LangGraph: Designing an Agent Runtime from first principles

The LangChain Team

September 4, 2025

21

min

Create agents

By Nuno Campos

Summary: We launched LangGraph as a low level agent framework nearly two years ago, and have already seen companies like LinkedIn, Uber, and Klarna use it to build production ready agents. LangGraph builds upon feedback from the super popular LangChain framework, and rethinks how agent frameworks should work for production. We aimed to find the right abstraction for AI agents, and decided that was little to no abstraction at all. Instead, we focused on control and durability. This post shares our design principles and approach to designing LangGraph based on what we've learned about building reliable agents.

LangGraph ALPHA

We just launched an alpha version of LangGraph 1.0!

Try it out now

We started LangGraph as a reboot of LangChain's super popular chains and agents more than two years ago. We decided that starting fresh would give us the most leeway to address all the feedback we had received since the launch of the original langchain open source library (in countless GitHub issues, discussions, Discord, Slack and Twitter posts). The main thing we heard was that langchain was easy to get started but hard to customize and scale.

This time around, our top goal was to make LangGraph what you'd use to run your agents in production. When we had to make tradeoffs, we prioritized production-readiness over how easy it would be for people to get started.

In this post, we'll share our process for scoping and designing LangGraph.

First: we cover what's different about building agents compared to traditional software.

Next: we discuss how we turned these differences into required features.

Finally: we show how we designed and tested our framework for these requirements.

The result is a low-level, production ready agent framework that scales with both the size and throughput of your agents.

1. What do agents need?

The first two questions we asked were, “Do we actually need to build LangGraph?” And, “Why can’t we use an existing framework to put agents in production?” To answer these questions, we had to define what makes an agent different (or similar) to previous software. By building many agents ourselves and working with teams like Uber, LinkedIn, Klarna, and Elastic, we boiled these down to 3 key differences.

More latency makes it harder to keep users engaged

Retrying long-running tasks when they fail is expensive and time-consuming

The non-deterministic nature of AI requires checkpoints, approvals, and testing

Managing latency

The first defining quality and challenge of LLM-based agents is latency. We used to measure the latency of our backend endpoints in milliseconds. Now, we need to measure agent run times in seconds, minutes, or soon hours.

This is because LLMs themselves are slow and are becoming slower with test-time compute. It’s also because we often need multiple LLM calls to achieve our desired results, with looping agents, and chaining LLM prompts to fix earlier output. And, we usually need to add non-LLM steps before and after the LLM call. For instance, you might need to get database rows into the context or create guardrails and verifiers to check the LLM call for accuracy.

All this latency enables building new things, but you do still need to keep end users engaged throughout. So, we identified two features that would come in handy when building agents:

Parallelization. Whenever there were multiple steps to your agent, when the next step doesn’t need the output of the previous one, then you could run them in parallel. But to do this reliably in production you want to avoid data races between your parallel steps.

Streaming. When you can’t reduce the actual latency of your agent any further without making it produce worse results, then you turn to perceived latency. Here the key unlock comes from showing useful information to the user while the agent is running, which can go all the way from a progress bar, or key actions taken by the agent, all the way to streaming LLM messages token-by-token in real-time to the end-user.

Managing reliability

The slowness of LLM agents had other implications, too. As we all know all too well, inevitably all software bugs out. Long-running agents fail more often because, the longer they run, the more opportunity there is for something to go wrong.

When traditional software bugs out, you usually want to retry. With AI agents? That may not be the best approach. If your agent fails on minute 9 of 10, going back to the beginning is pretty time consuming and also expensive.

So we knew we had to add two more features to the list:

Task queue. Queues eliminate one common source of failure by disconnecting the running of the agent from the request that triggered it. They provide the primitives to retry the agent reliably and fairly when needed.

Checkpointing. This saves snapshots of the computation state at intermediate stages and makes it a lot cheaper to retry when it does fail.

Managing non-deterministic LLMs

Next, the non-deterministic nature of LLMs creates two more challenges. When you write traditional software, you at least know what the code is supposed to do and what should result if you built it as you hoped. Generative AI obviously changes this.

With LLMs, both input and their output is open-ended. You can imagine when you’ve used ChatGPT, and the same prompt you used a day before produces a different result, or, how easy it is to ask your question in many different ways and get back similar results.

This is a very big part of what makes LLM agents so powerful compared other previous software, but it also introduces challenges when taking them to production.

The testing you do while developing will almost certainly miss many surprising ways in which your users will use your agent. You truly can’t plan for all the ways users will interact with your agent or how the LLM will respond. And so, two more features then become very useful when going to production:

Human-in-the-loop. Having the tools to interrupt and resume the agent at any point, without having to redo work done until then, enables many essential UX patterns for AI agents. For example, you can approve or reject actions, edit the next action, ask clarifying questions, or even time travel to re-do things from a previous step.

Tracing. To build for scale, developers need clear visibility into what’s happening within the details of their agent loops. You need to see inputs, trajectories, and outputs of the agent, otherwise you won’t know what users are asking of it, how the agent is handling it, and if users are happy with the outcome.

What developers need to build agents

This is how we built our shortlist of the six features most developers need when taking agents to production.

Parallelization – to save actual latency

Streaming – to save perceived latency

Task queue – to reduce number of retries

Checkpointing – to reduce the cost of each retry

Human-in-the-loop - to collaborate with the user

Tracing - to learn how users use it

If the agent you’re building doesn’t need most of these features (eg. because it’s a very short agent without tools and a single prompt), then you might not need LangGraph, or any other framework.

As we thought about building for each of these features, we also realized that developers wouldn’t adopt a framework that that provided all those features at the cost of making their LLM app perceivably slower to end users. Especially for agents deployed as chatbots. That made low latency our final overarching requirement.

Next, we’ll cover how we built these capabilities into LangGraph.

2. Why build LangGraph at all?

Back to our existential question, should we build something new, or adopt one of the existing open source frameworks built before LLMs? Armed with our feature shortlist, it became pretty easy to make that decision.

Why was a new framework needed?

Existing frameworks were mostly split between two categories:

DAG frameworks (made popular by Apache Airflow and many others)

These we had to exclude just based on the name, as LLM agents really benefit from looping, ie. the computation graph for an LLM agent is cyclical, and thus cannot be handled by DAG algorithms.

Durable execution engines (made popular by Temporal and others)

These options were closer, but in the end, as they were designed before LLM agents, so they were lacking some of those specific features — namely streaming. In addition, these engines introduced latency between steps which would have been noticeable to chatbot developers. Lastly, due to their design, the performance degrades the more steps there are in the history, which sounded like a bad bet to make as LLM agents get longer and more complicated.

So in the end our answer was, yes LLMs are different enough that previous production infrastructure needed some new ideas injected into it to become relevant for the new era. And so we embarked on building LangGraph.

3. Our design philosophy

We designed LangGraph with two leading principles.

We don’t know what the future holds for AI. The fewer assumptions we make about the future the better. No one really knows what it will look like to build with LLMs one, two, three years from now, so the fewer assumptions we bake into the design of the framework, the more relevant it will be in the future. The only assumptions we wanted to bake into it were the realizations we talked about above, i.e. that LLMs are slow, flaky, and open-ended.

It should feel like writing code. The public API for the framework should be as close as possible to writing regular framework-less code. Every requirement we place on the developer’s code needs to be justified by enabling a really high-value feature. Otherwise the pull of skipping the framework altogether is just too strong. The biggest competitor to any code framework is always no framework.

These principles impacted a few key design decisions that have stayed with us ever since.

First, the runtime of the library is independent from the developer SDKs. The SDKs are the public interfaces (classes, functions, methods, constants, etc) that developers use when building their agents. We currently offer two – StateGraph and the imperative/functional API. The runtime (which we call PregelLoop) implements each of the features listed earlier, plans the computation graph for each agent invocation, and executes it. This design enables us to evolve the developer APIs and the runtime independently. For instance, on the SDK side, it has allowed us to introduce the imperative SDK, and deprecate the very first SDK we offered, a Graph interface without shared state. On the runtime side, it has enabled us to implement many performance improvements over the last 2 years without any impact to the public APIs, and enabled experimentation with more radical changes to the runtime – more about this later when we get to distributed execution.

Second, we wanted to provide each of the 6 features as building blocks, with the developer free to pick which to use in their agent at any point in time. For instance, the ability to interrupt/resume for human-in-the-loop scenarios doesn’t get in your way until you reach for it (which is as easy as adding a call to the interrupt() function in one of your nodes). So LangGraph ended up as a uniquely low-level framework in a sea of other frameworks trying to convince developers to bet everything on the high-level abstraction du jour. We have seen these come and go, and LangGraph remaining relevant, so we’re happy with our approach so far.

4. The LangGraph runtime

With all this in mind, let’s look at how LangGraph implements each of the 6 features we wanted to have (as a reminder, these are parallelization, streaming, checkpointing, human-in-the-loop, tracing and a task queue).

Structured agents with discrete steps

If there is one idea that informs every other architectural decision we’ve made, it is the idea of structured agents. There’s a long tradition of adding more structure to computer programs, trading some amount of flexibility for a host of new features. Once upon a time, basic constructs like if statements and while loops were the new kid on the block. Agents too can be written directly as a single function with one big while loop. But when you do that, you lose the ability to implement features like checkpointing or human-in-the-loop. (Note: While it may technically be possible to interrupt execution of some kinds of subroutines, like generators, that execution state can’t be saved in a portable format that can be resumed from a different machine at a different time.)

Execution algorithm

Once you make the choice to structure agents into multiple discrete steps, you need to choose some algorithm to organize its execution. Even if it’s a naive one that feels like "no algorithm," which is where LangGraph started before launch. The problem with using “no algorithm” is, while it may seem simpler, you’re making it up as you go along, and end up with unexpected results. (For instance, an early version of a precursor to LangGraph suffered from non-deterministic behavior with concurrent nodes). The usual DAG algorithms (topological sort and friends) are out of the picture, given we need loops. We ended up building on top of the BSP/ Pregel algorithm, because it provides deterministic concurrency, with full support for loops (cycles).

Our execution algorithm works like this:

Channels contain data (any Python/JS data type), and have a name and current version (a monotonically increasing string)
