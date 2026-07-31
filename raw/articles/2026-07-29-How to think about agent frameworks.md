---
title: "How to think about agent frameworks"
source_url: "https://www.langchain.com/blog/how-to-think-about-agent-frameworks"
ingested: 2026-07-29
blog: "LangChain Blog"
published: "2026-06-15"
---
## Source Metadata
- **Blog:** LangChain Blog
- **Published:** 2026-06-15
- **URL:** https://www.langchain.com/blog/how-to-think-about-agent-frameworks

## Article Content

How to think about agent frameworks

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

Harrison's In the Loop

LangGraph

LangSmith

How to think about agent frameworks

Harrison Chase

April 20, 2025

26

min

Create agents

TL;DR:

The hard part of building reliable agentic systems is making sure the LLM has the appropriate context at each step. This includes both controlling the exact content that goes into the LLM, as well as running the appropriate steps to generate relevant content.

Agentic systems consist of both workflows and agents (and everything in between).

Most agentic frameworks are neither declarative or imperative orchestration frameworks, but rather just a set of agent abstractions.

Agent abstractions can make it easy to get started, but they can often obfuscate and make it hard to make sure the LLM has the appropriate context at each step.

Agentic systems of all shapes and sizes (agents or workflows) all benefit from the same set of helpful features, which can be provided by a framework, or built from scratch.

LangGraph is best thought of as a orchestration framework (with both declarative and imperative APIs), with a series of agent abstractions built on top.

OpenAI recently released a guide on building agents which contains some misguided takes like the below:

This callout initially angered me, but after starting to write a response I realized: thinking about agent frameworks is complicated! There are probably 100 different agent frameworks, there are a lot of different axes to compare them on, sometimes they get conflated (like in this quote). There is a lot of hype, posturing, and noise out there. There is very little precise analysis or thinking being done about agent frameworks. This blog is our attempt to do so. We will cover:

Background Info

What is an agent?

What is hard about building agents?

What is LangGraph?

Flavors of agentic frameworks

“Agents” vs “workflows”

Declarative vs non-declarative

Agent abstractions

Multi agent

Common Questions

What is the value of a framework?

As the models get better, will everything become agents instead of workflows?

What did OpenAI get wrong in their take?

How do all the agent frameworks compare?

Throughout this blog I will make repeated references to a few materials:

OpenAI’s guide on building agents (which I don’t think is particularly good)

Anthropic’s guide on building effective agents (which I like a lot)

LangGraph (our framework for building reliable agents)

Background info

Helpful context to set the stage for the rest of the blog.

What is an agent

There is no consistent definition of an agent, and they are often offered through different lenses.

OpenAI takes a higher level, more thought-leadery approach to defining an agent.

Agents are systems that independently accomplish tasks on your behalf.

I am personally not a fan of this. This is a vague statement that doesn’t really help me understand what an agent is. It’s just thought-leadership and not practical at all.

Compare this to Anthropic’s definition:

"Agent" can be defined in several ways. Some customers define agents as fully autonomous systems that operate independently over extended periods, using various tools to accomplish complex tasks. Others use the term to describe more prescriptive implementations that follow predefined workflows. At Anthropic, we categorize all these variations as agentic systems, but draw an important architectural distinction between workflows and agents:

Workflows are systems where LLMs and tools are orchestrated through predefined code paths.

Agents, on the other hand, are systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks.

I like Anthropic’s definition better for a few reasons:

Their definition of an agent is much more precise and technical.

They also make reference to the concept of “agentic systems”, and categorize both workflows and agents as variants of this. I love this.

Nearly all of the “agentic systems” we see in production are a combination of “workflows” and “agents”.

Later in the blog post, Anthropic defines agents as “… typically just LLMs using tools based on environmental feedback in a loop.”

Despite their grandiose definition of an agent at the start, this is basically what OpenAI means as well.

These types of agents are parameterized by:

The model to use

The instructions (system prompt) to use

The tools to use

You call the model in a loop. If/when it decides to call a tool, you run that tool, get some observation/feedback, and then pass that back into the LLM. You run until the LLM decides to not call a tool (or it calls a tool that triggers a stopping criteria).

Both OpenAI and Anthropic call out workflows as being a different design pattern than agents. The LLM is less in control there, the flow is more deterministic. This is a helpful distinction!

Both OpenAI and Anthropic explicitly call out that you do not always need agents. In many cases, workflows are simpler, more reliable, cheaper, faster, and more performant. A great quote from the Anthropic post:

When building applications with LLMs, we recommend finding the simplest solution possible, and only increasing complexity when needed. This might mean not building agentic systems at all. Agentic systems often trade latency and cost for better task performance, and you should consider when this tradeoff makes sense.

When more complexity is warranted, workflows offer predictability and consistency for well-defined tasks, whereas agents are the better option when flexibility and model-driven decision-making are needed at scale.

OpenAI says something similar:

Before committing to building an agent, validate that your use case can meet these criteria clearly. Otherwise, a deterministic solution may suffice.

In practice, we see that most “agentic systems” are a combination of workflows and agents. This is why I actually hate talking about whether something is an agent, but prefer talking about how agentic a system is. h/t the great Andrew Ng for this way of thinking about things:

Rather than having to choose whether or not something is an agent in a binary way, I thought, it would be more useful to think of systems as being agent-like to different degrees. Unlike the noun “agent,” the adjective “agentic” allows us to contemplate such systems and include all of them in this growing movement.

What is hard about building agents?

I think most people would agree that building agents is hard. Or rather - building an agent as a prototype is easy, but a reliable one, that can power business-critical applications? That is hard.

The tricky part is exactly that - making it reliable. You can make a demo that looks good on Twitter easily. But can you run it to power a business critical application? Not without a lot of work.

We did a survey of agent builders a few months ago and asked them: “What is your biggest limitation of putting more agents in production?” The number one response by far was “performance quality” - it’s still really hard to make these agents work.

What causes agents to perform poorly sometimes? The LLM messes up.

Why does the LLM mess up? Two reasons: (a) the model is not good enough, (b) the wrong (or incomplete) context is being passed to the model.

From our experience, it is very frequently the second use case. What causes this?

Incomplete or short system messages

Vague user input

Not having access to the right tools

Poor tool descriptions

Not passing in the right context

Poorly formatted tool responses

The hard part of building reliable agentic systems is making sure the LLM has the appropriate context at each step. This includes both controlling the exact content that goes into the LLM, as well as running the appropriate steps to generate relevant content.

As we discuss agent frameworks, it’s helpful to keep this in mind. Any framework that makes it harder to control exactly what is being passed to the LLM is just getting in your way. It’s already hard enough to pass the correct context to the LLM - why would you make it harder on yourself?

What is LangGraph

LangGraph is best thought of as a orchestration framework (with both declarative and imperative APIs), with a series of agent abstractions built on top.

LangGraph is an event-driven framework for building agentic systems. The two most common ways of using it are through:

a declarative, graph-based syntax

agent abstractions (built on top of the lower level framework)

LangGraph also supports a functional API, as well as the underlying event-driven API. There exist both Python and Typescript variants.

Agentic systems can be represented as nodes and edges. Nodes represent units of work, while edges represent transitions. Nodes and edges are nothing more than normal Python or TypeScript code - so while the structure of the graph is represented in a declarative manner, the inner functioning of the graph’s logic is normal, imperative code. Edges can be either fixed or conditional. So while the structure of the graph is declarative, the path through the graph can be completely dynamic.

LangGraph comes with a built-in persistence layer. This enables fault tolerance, short-term memory, and long-term memory.

This persistence layer also enables “human-in-the-loop” and “human-on-the-loop” patterns, such as interrupt, approve, resume, and time travel.

LangGraph has built-in support for streaming: of tokens, node updates, and arbitrary events.

LangGraph integrates seamlessly with LangSmith for debugging, evaluation, and observability.

Flavors of agentic frameworks

Agentic frameworks are different across a few dimensions. Understanding - and not conflating - these dimensions is key to being able to properly compare agentic frameworks.

Workflows vs Agents

Most frameworks contain higher level agent abstractions. Some frameworks include some abstraction for common workflows. LangGraph is a low level orchestration framework for building agentic systems. LangGraph supports workflows, agents, and anything in-between. We think this is crucial. As mentioned, most agentic systems in production are a combination of workflows and agents. A production-ready framework needs to support both.

Let’s remember what is hard about building reliable agents - making sure the LLM has the right context. Part of why workflows are useful is that they make it easy to pass the right context to LLMs. You decide exactly how the data flows.

As you think about where on spectrum of “workflow” to “agent” you want to build your application, there are two things to think about:

Predictability vs agency

Low floor, high ceiling

Predictability vs agency

As your system becomes more agentic, it will become less predictable.

Sometimes you want or need your system to be predictable - for user trust, regulatory reasons, or other.

Reliability does not track 100% with predictability, but in practice they can be closely related.

Where you want to be on this curve is pretty specific to your application. LangGraph can be used to build applications anywhere on this curve, allowing you to move to the point on the curve that you want to be.

High floor, low ceiling

When thinking about frameworks, it can be helpful to think about their floors and ceilings:

Low floor: A low floor framework is beginner-friendly and easy to get started with

High floor: A framework with a high floor means it has a steep learning curve and requires significant knowledge or expertise to begin using it effectively.

Low ceiling: A framework with a low ceiling means it has limitations on what can be accomplished with it (you will quickly outgrow it).

High ceiling: A high ceiling framework offers extensive capabilities and flexibility for advanced use cases (it grows with you?).

Workflow frameworks offer a high ceiling, but come with a high floor - you have to write lot of the agent logic yourself.

Agent frameworks are low floor, but low ceiling - easy to get started with, but not enough for non-trivial use cases.

LangGraph aims to have aspects that are low floor (built-in agent abstractions that make it easy to get started) but also high ceiling (low-level functionality to achieve advanced use cases).

Declarative vs non-declarative

There are benefits to declarative frameworks. There are also downsides. This is a seemingly endless debate among programmers, and everyone has their own preferences.

When people say non-declarative, they are usually implying imperative as the alternative.

Most people would describe LangGraph as a declarative framework. This is only partially true.

First - while the connections between the nodes and edges are done in a declarative manner, the actual nodes and edges are nothing more than Python or TypeScript functions. Therefore, LangGraph is kind of a blend between declarative and imperative.
