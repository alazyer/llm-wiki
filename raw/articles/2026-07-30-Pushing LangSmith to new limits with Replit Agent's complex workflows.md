---
title: "Pushing LangSmith to new limits with Replit Agent's complex workflows"
source_url: "https://www.langchain.com/blog/customers-replit"
ingested: 2026-07-30
blog: "LangChain Blog"
published: "2026-06-15"
---
## Source Metadata
- **Blog:** LangChain Blog
- **Published:** 2026-06-15
- **URL:** https://www.langchain.com/blog/customers-replit

## Article Content

Pushing LangSmith to new limits with Replit Agent's complex workflows

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

LangSmith

Pushing LangSmith to new limits with Replit Agent's complex workflows

The LangChain Team

September 26, 2024

4

min

Create agents

Replit is at the forefront of AI innovation with its platform that simplifies writing, running, and collaborating on code for over 30+ million developers. They recently released Replit Agent, which immediately went viral due to the incredible applications people could easily create with this tool.

Behind the scenes, Replit Agent has a complex workflow which enables a highly custom agentic workflow with a high-degree of control and parallel execution. By using LangSmith, Replit gained deep visibility into their agent interactions to debug tricky issues.

The level of complexity required for Replit Agent also pushed the boundaries of LangSmith. The LangChain and Replit teams worked closely together to add functionality to LangSmith that would satisfy their LLM observability needs. Specifically, there were three main areas that we innovated on:

Improved performance and scale on large traces

Ability to search and filter within traces

Thread view to enable human-in-the loop workflows

Improved performance and scale on large traces

Most other LLMOps solutions monitor individual API requests to LLM providers, offering a limited view of single LLM calls. In contrast, LangSmith from day one has focused on tracing the entire execution flow of an LLM application to provide a more holistic context.

Tracing is important for agents due to their complex nature. It captures multiple LLM calls as well as other steps (retrieval, running code, etc). This gives you granular visibility into what’s happening, including at the inputs and outputs of each step, in order to understand the agent’s decision-making.

Replit Agent was a ripe example for advanced tracing needs. Their agentic tool goes beyond simply reviewing and writing code, but also performs a wider range of functions – including planning, creating dev environments, installing dependencies, and deploying applications for users.

As a result, Replit’s traces were very large - involving hundreds of steps. This posed significant challenges for ingesting data and displaying it in a visually meaningful way.

To address this, the LangChain team improved their ingestion to efficiently process and store large volumes of trace data. They also improved LangSmith’s frontend rendering to display long-running agent traces seamlessly.

Search and filter within traces to pinpoint issues

LangSmith has always supported search between traces, which allows users to find a single trace among hundreds of thousands based on events or full text search. But as Replit Agent’s traces got longer and longer, the Replit team needed to search within traces for specific events (oftentimes issues reported by alpha testers). This required augmenting existing search capabilities.

In response, a new search pattern – searching within traces – was added to LangSmith. Instead of sifting and scrolling call-by-call within a large trace, users could now filter directly on a criteria they cared about (e.g. keywords in the inputs or outputs of a run). This greatly reduced Replit’s time needed to debug agent steps within a trace.

Thread view to enable human-in-the-loop workflows

A key differentiator of Replit Agent was its emphasis on human-in-the-loop workflows. Replit Agent intends to be a tool where AI agents can collaborate effectively with human developers, who can come in and edit and correct agent trajectories as needed.

With separate agents to perform roles like managing, editing, and verifying generated code,  Replit’s agents interacted with users continuously - often over long periods with multiple turns of conversation. However, monitoring these conversational flows was often difficult, as each user session would generate disjoint traces.

To solve this, LangSmith’s thread view helped collate traces from multiple threads together that were related (i.e. from one conversation). This provided a logical view of all agent-user interactions across a multi-turn conversation, helping Replit better 1) find bottlenecks where users got stuck and 2) pinpoint areas where human intervention could be beneficial.

Conclusion

Replit is pushing the frontier of AI agent monitoring using LangSmith’s powerful observability features. By reducing the effort of loading long, heavy traces, the Replit team has greatly sped up the process of building and scaling complex agents. With faster debugging, improved trace visibility, and better handling of parallel tasks, Replit is setting the standard for AI-driven development.

Related content

Case Studies

How Apollo Rebuilt Its AI Assistant on Deep Agents to Power the Full GTM Loop

Sofia Sulikowski

July 21, 2026

6

min

Conceptual Guide

LangSmith

Building Governed Agents: A Framework for Cost, Control, and Compliance

Martha Janicki

July 20, 2026

15

min

Conceptual Guide

LangSmith

Agents need their own computer. Here's how to give them one safely.

Amy Ru

July 15, 2026

12

min
