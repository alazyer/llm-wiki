---
title: "Improving Agents is a Data Mining Problem"
source_url: "https://www.langchain.com/blog/improving-agents-is-a-data-mining-problem"
ingested: 2026-07-26
blog: "LangChain Blog"
published: "2026-07-08"
---

## Source Metadata

- **Blog:** LangChain Blog
- **URL:** https://www.langchain.com/blog/improving-agents-is-a-data-mining-problem
- **Fetched:** 2026-07-26
- **Extracted title:** Improving Agents is a Data Mining Problem

## Article Content

Improving Agents is a Data Mining Problem

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

Observability & Evals

Improving Agents is a Data Mining Problem

Vivek Trivedy

July 7, 2026

7

min

Create agents

Key Takeaways

• Mining traces gives you signals to hill climb on

• Open model fine-tuning & compound agent systems help you process large scale trace data

• Continual Learning is about processing & integrating agent data back into agents over long time horizons

• Agents will produce more data than humans have in history. We need to update our tooling to process it.

Continual Learning, Harness Engineering, Post-Training all boil down to the same substrate: curating data at scale to run experiments & improve agents. I gave a talk about this at this year’s AI Engineer World Fair, shoutout @swyx for creating something amazing!

We talked about why data mining from Traces is one of the highest leverage muscles companies can build to understand their agents, curate data at scale, & then run improvement loops.

Here’s a brief walking through of the slides I shared with some commentary and (semi-)spicy takes from the talk.

Every Continual Learning Company is an Observability Company

Let’s warm up with a lukewarm take: Every Continual Learning Company is an Observability Company…and vice versa! When we look at the workflows of teams that do Continual Learning, the first thing we see is some sort of message to share traces.

Continual Learning is about agents taking actions in their environment and then integrating information produced from that experience back into the agent system. This roughly mirrors how humans learn by taking actions in their environment, triggering some sort of learning, memorization, or storage to use later.

This is why Traces are the currency of long horizon agent improvement.  They’re projections of agent experience in environments into a data format we can mine to understand.

Today it’s unclear exactly how to integrate back all of the information, but it likely will be a mix of:

collecting training data to integrate into back into model weights via SFT, RL, etc

harness engineering to add instructions, tools, skills, orchestration strategies, etc

integrating information into memory stores for contextual retrieval

The term Scaling Dreaming is a nice way to describe how to do this at large data scales over long time-horizons

A Practical Recipe for Agent Improvement

This talk was largely centered around motivating a practical recipe so teams can start improving their agents today.

Kickstarting the data collection flywheel is one of the most valuable things teams can do! And that means building some decent version of the agent and getting it out there. From there improvement comes from:

mining data to see what to improve

curating evals (ie. training data) to fit on

running experiments to improve your agent along some axis you care about

Understanding Agents via Traces at Scale

Agents behavior is more opaque than traditional code. We don’t know exactly what agent will do from just reading their agent definition. We trade determinism for autonomy and then use Traces to fill the understanding gap.

By running evals and reading traces at scale, we can develop a quantitative measure + intuition for how agents are likely to behave on tasks we give them.

Traces allow us to understand and then improve agent behavior. But modern agents are becoming much more complex, do much more work, and thus produce much more data than we’ve ever seen before. Reading millions of traces where many traces are millions of tokens long is:

A cost problem for processing that many tokens

A context problem for searching through traces to find important signals

This is why we create specialized agents and models to efficiently understand and curate data from Traces at Scale.

Building Systems to Understand Traces

Open models have crossed an intelligence threshold and are cost-efficient options for processing traces at scale. Every company looks for different signals in their traces. This includes nuances in user interactions, domain specific data, and an ability to disambiguate between which sub-sections of data matter. We fine-tuned a Trace judge model to mine signals across our tracing project and find that on narrow tasks, open, small models outperform closed frontier models while being orders of magnitude cheaper to run.

Another benefit of owning and deploying your own model intelligence is that it can be much cheaper to run at scale as you trade token costs for infrastructure costs. At high enough inference volumes this tradeoff makes sense for many teams.

We think that mining traces is important, so we built a product around it.  LangSmith Engine uses specialized agents to read every trace, look for particular signals your team cares about, finds issues, creates code fixes, generates evals, commits important pieces of information to memory+context stores, and works to improve every agent over time.

Here’s a deep dive into how we think about building and evaluating Engine.

Model-Task-Harness Fit, Evals, & How to Fit Agents

The outputs of Trace Mining become inputs become the inputs of running improvement loop experiments. Mining “good” traces gives us a signal to distill smaller models which gives greater cost efficiency. Every agent failure in production becomes a target we can create eval and environment for.

Evals are training data for agents.

The point of evals is to make them pass and thus the same behaviors we measure in traces get transferred into agent behavior as we hill-climb them.

Generally our job is to:

Find good data

Find good fit functions

Once we’ve collected data, our job is to fit our learning system to that data. In the same way “classical ML” had sci-kit-learn fit functions, modern day agents have fitting strategies like fine-tuning (SFT, RL, DPO) or Harness Engineering with strategies like auto-research on the harness using eval scores as the hill-climbing metric.

The idea here is to work backwards from the model as a source of intelligence and finding optimal alterations to the model and harness to optimize performance on a set of tasks.  Harnesses are amplifiers and extenders of native model intelligence and as models get smarter much of the harness will dissolve to allow models to freely use their intelligence.

For an example of a fit function, we founds loops and auto-research as a general strategy for improving agents grounded in environments and evals. On Terminal Bench 2.0, we found that simply adjusting the harness by hill-climbing correctness metrics & traces to understand behavior gave us a big 13.7% lift over the base harness.

Traces densify the feedback signal by giving agents rich behavior feedback to search across beyond simple scalar rewards.

We get this question a lot: “when should I do harness engineering vs. fine-tuning?” Like most questions in machine learning…it depends. But a general strategy we've seen be very successful is a funnel (or a sandwich) of Harness Engineering -> Fine-Tuning -> Harness Engineering.

Harness Engineering is often enough for most teams. Teams get immediate feedback and get access to a very high-bandwidth surface for transferring their knowledge and observations of errors into their agent. Models get smarter every generation and with that grows their ability to do in-context learning. Teams that take care in preparing good context, tool surfaces, and prompts can very often get good enough performance for their tasks.

But eventually harness engineering hits an intelligence ceiling where simply tweaking the prompt doesn’t create more gains. For both these cases or cases of high inference workloads where you want to distill information to smaller models, fine-tuning makes sense. It’s more involved, requires curating data and doing experiments over longer feedback loops. But reshaping model intelligence towards your tasks is an effective way to increase performance.

And finally once you’re happy with a fine-tuned model, further harness engineering is a good avenue to see how the new intelligence landscapes generalizes to related problems as you use the fine-tuned model across your tasks. If I had to recommend teams one recipe, it would be this one for getting the best tradeoff of rapid iteration and progressive exposure to more complicated fit functions.

Takeaways

It’s difficult to distill the vast landscape of agent improvement, Continual Learning, Environments/Evals, and more into a single talk and set of slides. But the goal of this talk and write-up was to give teams a practical jumping off point that motivates why data will be the key driver of agent improvement in the years to come.

Some brief takeaways:

Mining traces gives you signals to hill climb on

Open model fine-tuning & compound agent systems help you process large scale trace data

Continual Learning is about processing & integrating agent data back into agents over long time horizons

Agents will produce more data than humans have in history. We need to update our tooling to process it.

Our research team at LangChain Labs is focusing on these problems and helping every team use their data to build better agents. To better data, better understanding, and better agents.

Related content

Open Source

Observability & Evals

How We Benchmark Deep Agents

Nick Hollon

Harrison Chase

July 23, 2026

4

min

Observability & Evals

Towards Automating Eval Engineering

Vivek Trivedy

July 22, 2026

5

min

Observability & Evals

IssueBench - How We Evaluate Engine

Nick Bray

Arjun Nargolwala

July 20, 2026

6

min
