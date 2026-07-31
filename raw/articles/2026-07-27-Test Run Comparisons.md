---
title: "Test Run Comparisons"
source_url: "https://www.langchain.com/blog/test-run-comparisons"
ingested: 2026-07-27
blog: "LangChain Blog"
published: "2026-06-15"
---
## Source Metadata
- **Blog:** LangChain Blog
- **Published:** 2026-06-15
- **URL:** https://www.langchain.com/blog/test-run-comparisons

## Article Content

Test Run Comparisons

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

LangChain

Test Run Comparisons

The LangChain Team

October 17, 2023

4

min

Create agents

One pattern I noticed is that great AI researchers are willing to manually inspect lots of data. And more than that, they build infrastructure that allows them to manually inspect data quickly. Though not glamorous, manually examining data gives valuable intuitions about the problem.

- Jason Wei, OpenAI

Evaluations continue to be one of the hardest parts of building LLM applications. It's really tough to evaluate in a quantitative way the effect of changes to your prompt, chain, or agent. We're bullish on LLM-assisted evaluation, but, at the same time, we definitely recognize that it's hard to have complete trust in them.

Jason's tweet above sums up what we see a lot of the best researchers (and engineers) doing. They want to manually inspect data to gain intuition about the problem. At LangChain, we want to build the infrastructure to help do that - which is why we're excited to announce Test Run Comparisons today.

In the initial release of LangSmith we had support for running tests, including scoring them with LLM-assisted feedback. However, each test was run in isolation. We quickly saw two usage patterns emerge:

People are still hesitant to trust the LLM-assisted feedback directly

Users often wanted to not only score their test run in isolation, but also compare it to previous iterations

When building Test Run Comparisons, we kept both of these insights in mind. We wanted to create an easy UX to see multiple test runs side-by-side. We also wanted to create an easy UX where people could use LLM-assisted evals (or regex/other eval) to get an initial score, then manually explore those datapoints for further insights.

So how does it work?

First, you need to set up a dataset and run some tests. See documentation here for instructions on how to do that. Nothing new here, so if you've already done that for an existing project you're all good.

Inside a dataset, you can easily select two (or more) test runs, then click Compare.

From there, you will be brought into the Test Run Comparison view. This should look like the below

You can easily see the inputs, the reference output, and then the actual output for each datapoint - along with any eval metrics, time and latency for that run.

This view is designed to make it easy to quickly compare test runs across the same inputs. If you want a deeper look at a particular datapoint, you can click on that row and sidebar will pop up allowing you to drill down into the details of those runs.

On that sidebar, we've also added up and down carets (▲ and ▼) to easily flip between runs.

This view should hopefully make it easy to compare runs for a particular datapoint. But how do you know what datapoints to be looking at?

We've added filters for each column - similar to Excel. Using these filters, you can filter the rows according to any criteria.

The criteria we recommend using to start? Filter one test run to datapoints it got correct, and the other one to datapoints that it got incorrect. This allows you to quickly drill on places of significant difference between the two test runs, which should more easily allow you to discover what has changed.

Building an LLM application is hard. A big part of that is understanding how the LLM is working on a particular task. Setting up an evaluation dataset and then being able to easily compare runs on that dataset is crucial for developing the understanding needed to improve the application.  Test Run Comparison in LangSmith aimed at solving this problem. Please let us know any feedback you have!

LangSmith is in private beta - sign up here. We'll be rolling out more access over the next few weeks, as well as continuing to add features like this.

Related content

LangChain

LangSmith

Observability & Evals

Your coding agent bill doubled. Here’s how to fix it.

Amy Ru

July 2, 2026

6

min

Open Source

LangChain

Agent Architecture

Deep Agents

How to Build a Custom Agent Harness

Sydney Runkle

June 3, 2026

6

min

LangChain

Open Source

LangGraph

From Token Streams to Agent Streams

Christian Bromann

Nick Hollon

May 21, 2026

9

min
