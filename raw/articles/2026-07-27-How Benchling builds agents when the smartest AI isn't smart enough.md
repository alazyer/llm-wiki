---
title: "How Benchling builds agents when the smartest AI isn't smart enough"
source_url: "https://www.langchain.com/blog/benchling-max-agency-podcast"
ingested: 2026-07-27
blog: "LangChain Blog"
published: "2026-06-12"
---
## Source Metadata
- **Blog:** LangChain Blog
- **Published:** 2026-06-12
- **URL:** https://www.langchain.com/blog/benchling-max-agency-podcast

## Article Content

How Benchling builds agents when the smartest AI isn't smart enough

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

Max Agency Podcast

How Benchling builds agents when the smartest AI isn't smart enough

James Donner

June 11, 2026

4

min

Create agents

Nicholas Larus-Stone is the Head of AI at Benchling , the R&D data platform that life science companies use to store and manage their experiments, samples, instruments, and analysis. Benchling has been around since 2012. In October 2025, it launched Benchling AI, an intelligence layer with a chat interface, backed by an agent, that helps scientists find data, design experiments, and write reports. Nick came to Benchling through its acquisition of Sphinx Bio (acquired), the analysis startup he founded.

In this conversation with LangChain Co-Founder & CEO Harrison Chase, Nick walks through what it takes to build agents for scientific work, and where the playbook from coding agents holds up and where it breaks down.

🎧 Watch the full conversation on YouTube, or listen & subscribe on Apple Podcasts or Spotify.

What we learned

Why Benchling runs multiple models on the same task

Instead of running the same model multiple times, Benchling runs across different providers. Different model families make different mistakes, so there is a stronger quality indicator for their team. If multiple models agree, it indicates good data quality. If multiple models disagree, there's usually an error.

"Each of them will make slightly different errors... being able to ask different model providers, we found gives us much better performance."

How Benchling approaches trace review

In the world of scientific research, evals can only get you so far. Benchling leans on a structured approach for looking at production traces. Every week, they have a rotating fire chief who addresses and flags issues that are addressed in their weekly tech operations meeting. For external signals, they look at thumbs up & thumbs down user feedback.

"People who are working on specific features are gonna go look at the traces — our product managers, our engineers who are building something will actually go and see how people are using that feature after releasing it."

Agents are having a big impact in scientific work

Nicholas points out that agents are compressing workflows and reducing the number of experiments needed to get an answer. By reducing dead time between steps, a day saved can often become a week saved. In addition, agents are also helping scientists design experiments more rigorously upfront, reducing the number of runs needed to get to a conclusion.

Other Topics Discussed

Why Benchling invests so heavily in getting clean data upfront

How they cross-check answers between models to get more out of each one

Why and how Benchling leans on production traces

Where AI actually helps science today, and where it still gets stuck

Why understanding LLMs is closer to biology than software engineering

Timestamps

00:00 Intro

01:22 What Benchling AI is, and the 14-year data platform underneath it

04:36 Why a decade of structured data is a core advantage

05:57 The architecture under the hood

08:28 Similarities and differences compared to a coding harness

11:14 Benchling’s multi-agent architectures

14:36 Dealing with verifiable vs non-verifiable tasks

16:19 Doing evals when clean benchmarks aren’t possible

18:13 Context engineering: SQL vs. file-based harnesses

22:11 Memory: agents that create and update their own skills

25:30 What user education for scientists looks like

30:33 Why understanding LLMs is closer to biology than software

33:28 When will agents discover a novel cure for disease?

44:58 The future of harnesses in science

48:13 Why fine-tuning on biology hasn't beaten frontier models

People & Tools Mentioned During This Episode

Agent Skills (Claude Docs)

Benchling’s Deep Research Agent

Claude (Anthropic)

Design of experiments (DOE)

FDA Investigational New Drug (IND) application

Gemini (Google)

Google AI co-scientist

LangSmith

Model Context Protocol (MCP)

The Ralph (Wiggum) Loop (Geoffrey Huntley)

Sphinx Bio

Get More Max Agency

Hosted by Harrison Chase, CEO of LangChain, each episode goes deep with the builders designing, deploying, and learning from real agent systems in the wild. From architecture decisions to evals, tooling, and failure modes, Max Agency is for people who want to understand what it really takes to build useful agents.
