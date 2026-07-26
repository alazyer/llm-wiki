---
title: "Recap of Interrupt 2025: The AI Agent Conference by LangChain"
source_url: "https://www.langchain.com/blog/interrupt-2025-recap"
ingested: 2026-07-26
blog: "LangChain Blog"
published: "2026-06-15"
---

## Source Metadata

- **Blog:** LangChain Blog
- **URL:** https://www.langchain.com/blog/interrupt-2025-recap
- **Fetched:** 2026-07-26
- **Extracted title:** Recap of Interrupt 2025: The AI Agent Conference by LangChain

## Article Content

Recap of Interrupt 2025: The AI Agent Conference by LangChain

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

Company Announcements

Recap of Interrupt 2025: The AI Agent Conference by LangChain

The LangChain Team

May 14, 2025

5

min

Create agents

That's a wrap on Interrupt 2025! This year, 800 folks from across the globe gathered in San Francisco for LangChain's first industry conference to hear stories of teams building agents – and we’re still riding the high! Cisco, Uber, Replit, LinkedIn, Blackrock, JPMorgan, Harvey, and more shared lessons on architectures, evals, observability, and prompting strategies – both their challenges and their wins.

The main thing we felt leaving the day was that agents are here, and we’ve never been more bullish on the future of the industry. If you weren’t with us in person, we’ll be sharing content over the next few weeks, including recordings of all the talks. Sign up here to get the content as soon as it drops!

Keep reading for big themes of the days and product launches!

In Case You Missed It ✨

Keynote Themes:

Harrison's opening keynote at Interrupt highlighted a few key beliefs:

Agent Engineering is a new discipline – Taking inspiration from the best of software engineering, prompting, product, and machine learning, we believe you need to code, engineer your prompts for the right context, understand the business workflows to turn them into agents, and understand likelihoods and distributions similar to in ML. Being good at all four disciplines is a tall task, and in pursuit of our mission to make agents ubiquitous, we want to make everyone an 100x agent engineer – no matter what your relative strengths are to start with.

LLM apps will rely on many different models. The LangChain package today is mostly about giving companies model optionality. LangChain has had 3 stable releases, and we’re laser focused on depth and breadth of integrations. Developers want the choice and flexibility that LangChain provides, and as a result, LangChain has been downloaded over 70M times in the last month – even more than the OpenAI SDK 🤯.

LangGraph is how you build reliable agents. One of the hardest parts about building agents is getting the right context to the LLM. LangGraph, our agent orchestration framework, gives you full authorship over the cognitive architecture so you can control the workflow and information flow. This low-level control makes LangGraph unique as an agent orchestration framework.

AI Observability is different. With GenAI apps, you’re dealing with dense, unstructured information – often text, audio, or image. The agent engineer needs to understand what’s happening with the application, and is a totally different user with different needs than SREs that traditional observability tools serve. If LangSmith's aggregate trace volume reflects broader industry trends, more agents are moving into production—making the need for an AI observability stack more critical than ever.

Launches!

We love to ship at LangChain, and we announced a LOT.

LangGraph Platform is Generally Available. LangGraph Platform is a deployment and management platform for long-running, stateful agents, and you can 1-click deploy your agent today – available with Cloud, Hybrid, and fully self-hosted deployments. See the docs for more information or check out our 4 min walk through.

Open Agent Platform – an open source, no code agent builder. You can now build agents without being a developer – select MCP tools, customize prompts, select models, connect to data source, and other agents all through the UI. Powered by LangGraph Platform. Sign up here.

LangGraph Studio v2. LangGraph Studio can now be run locally without a desktop app. It’s an agent IDE that lets you visualize and debug agent interactions. In v2, we're giving you the ability to pull down traces into the studio to investigate, add examples to a dataset for evals, and directly update prompts in a UI.

LangGraph Pre-Builts lowers the effort for building agents. There are common architectures that we see repeatedly used when building agents – Swarm, Supervisor, tool-calling agent – so we want to lower the burden for implementing these architectures in your app. LangGraph pre-builts lets you leverage common architectures with less config code.

LangSmith Observability now includes agent specific metrics. We’ve added support for tool calling and trajectory tracking so you can see the common paths your agent is taking and spot expensive, slow, or spotty calls.

Open Evals and Chat Simulations. Authoring evaluators is tedious. While some evals are very application / use case specific, some are not – and that’s good news, because we can write those for you. We now have an open source catalog of evals, useful for code, extraction, RAG, agent trajectory testing, and more. We’re also excited to release chat simulation and evals for multi-turn conversation. Check it out here.

LLM-as-Judge: alignment and calibration (in Private Preview). LLM-as-judge is a fantastic technique for evaluating performance when more discretion or judgement is required. However, even the judge is subject to being faulty. We’re excited to launch, in private preview, a way to bootstrap LLM-as-a-judge evaluators with human feedback scores and constantly calibrate and audit scores to make sure the judge is performing well. If you’re interested, sign up here for access!

We’re so excited to be building alongside you all, and aim to make this an annual event. We’ll see you the Community slack, at our meetups, and we’ll see you next year at Interrupt: The AI Agent Conference by LangChain.

Nothing beats the LangChain community in-person!

Related content

Company Announcements

LangSmith

Everything we shipped at Interrupt

Jacob Talbot

May 14, 2026

11

min

Company Announcements

Introducing LangChain Labs

Harrison Chase

May 14, 2026

3

min

Company Announcements

Interrupt Preview: Meet the MC

Becca Weng

April 28, 2026

7

min
