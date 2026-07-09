---
title: "Self Learning for Agents Clearly Explained"
source: "https://x.com/ataiiam/status/2069797329809395978"
author:
  - "[[@ataiiam]]"
published: 2026-06-24
created: 2026-07-09
description: "The self-learning agent that everyone's talking about is not the one your product needs.The most useful signal of all is actually one that a..."
tags:
  - "clippings"
---
![[b7d0136bdfa4cf87ec4a1a8a37d46001_MD5.jpg]]

The self-learning agent that everyone's talking about is not the one your product needs.

The **most useful signal** of all is actually one that almost nobody captures: the people using your product.

Your agents and users already work side by side, thousands of times a day.

It can learn from each one of these interactions. See the video.

<video preload="none" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/amplify_video_thumb/2069736704454332416/img/0QI5SPYTw-XkaTgJ.jpg" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"><source type="video/mp4" src="blob:https://x.com/dc8b61a3-6094-42c9-9072-854e023e13c6"></video>

0:04 / 0:31

Capturing a live interaction flow between a user and agent for self-learning

This is a guide to all three learning layers & how it’s relevant to **you**.

Today, agents can learn in three different layers.

For each one: what it is, what it costs, who is actually building it, and whether you even should use self-learning.

You'll see how **Anthropic, Karpathy**, **DeepMind**, **Microsoft**, **Hermes**, **OpenClaw**, **CopilotKit** and others approach self-learning, and where each one stops.

## Agents can learn in 3 different layers

The cleanest split comes from [Harrison Chase](https://www.langchain.com/blog/continual-learning-for-ai-agents).

- **The model =** The weights (training parameters)
- **The harness =** The code around the model (loop, tools, prompts)
- **The context =** The memory and skills outside the harness which grow

You already use all three in Claude Code.

The model is Claude, the harness is Claude Code itself, and the context is your CLAUDE.md and your skills. Each layer can get better on its own, without touching the other two.

![[dc91705ec932c9c0497879af9ba5a855_MD5.jpg]]

Three learning layers for agents

In a real product, self-learning almost always means the harness or the context, not the model. The model belongs to the labs. The other two are yours.

I will use one **example** across all three: a support agent that issues refunds.

For each layer, I'll ask two questions: can it learn there, and do you own that learning?

## Layer 1: The Model

This is the layer everyone pictures, and the one almost no one runs. The agent improves the model itself. Labs do it three ways:

1. edit the training code and keep what raises the score
2. write your own training data and fold it into the weights
3. try many code changes and keep the best one

![[4d5fa5ac2b6c7c5b9dc608407a857eed_MD5.jpg]]

All three are the same loop, and it only runs when a computer can score the result for free.

**Model Approach #1 (Karpathy)**

Edit the training code, keep what helps

Karpathy's [AutoResearch](https://github.com/karpathy/autoresearch) points a coding agent at a small training setup and runs it overnight. It edits one file, trains for five minutes, and scores the result. It keeps the change or undoes it. Then it repeats, a hundred times before you wake up.

One run found a real speedup, about 11 percent.

**The catch:** The agent improved a different, smaller model. Its own weights never actually changed.

![[2ddbc0bbd5eca85436a90beed15a2997_MD5.jpg]]

**Model Approach #2 (MIT SEAL)**

Write your own training data, apply it as a weight update

[Self-Adapting LLMs (SEAL)](https://arxiv.org/abs/2506.10943) writes its own training data, trains on it, and keeps what raised the score. These are real changes to its own weights.

**The catch:** Every edit is a full retrain, 30 to 45 seconds each. And it forgets old tasks as it learns new ones. Fine in a research lab. Too slow for production.

![[93a274156a0c757bfab171a9f30353eb_MD5.jpg]]

**Model Approach #3 (DeepMind's AlphaEvolve)**

Evolve code against a free scorer

[AlphaEvolve](https://deepmind.google/blog/alphaevolve-a-gemini-powered-coding-agent-for-designing-advanced-algorithms/) proposes a code change, scores it automatically, and keeps the winner. Then it repeats. It is the **most convincing** of the three.

It made a real attention kernel 32.5 percent faster. It found a matrix-multiplication shortcut that had stood unbeaten since 1969.

It even made the training of its own AI model faster. The AI improved the AI it runs on.

**The catch:** It only works where a computer can score the answer for free. That means code and math.

![[b26fd7a0102b0e5d8621ad301668b840_MD5.jpg]]

So which one belongs in your product? **Probably none.**

They all need a free, trustworthy score. The kernel is faster or it isn't. The model scores higher or it doesn't.

Step into support, sales, or operations, and that score is gone.

**Refund agent example (Model):** You retrain it on past refund decisions. But no computer can grade a refund as right or wrong. The loop has nothing to score, so it never runs. This layer usually just stays in the labs.

## Layer 2: The Harness

The harness is everything that is not the model. The loop that runs it. The tools and files it can reach. The system prompt. The checks that fire before it acts.

The model supplies the intelligence. The harness decides how it gets spent.

The harness and the context are easy to mix up. [Dex Horthy](https://x.com/dexhorthy/status/1985699548153467120) frames it cleanly. The harness is one part of context engineering: managing everything the model sees.

![[04aa6c853364719b3d32f8f27ad29904_MD5.jpg]]

The methods line up by how much you do, and how much the harness does itself. You can:

- build the loops by hand
- have an agent read the traces and propose changes you approve
- let the harness rewrite itself with no one watching
- or skip building it and install a prebuilt stack

![[6f0f51a17b659935206d2c0b6f3730bb_MD5.jpg]]

**Harness Approach #1 (Loop Engineering)**

Build loops around the agent

The base loop is a model calling tools until it decides it is done. The move is to wrap more loops around it.

The most useful is a verification loop. A second grader scores the output and sends it back when it falls short.

Two more sit above it. One fires the agent on a schedule, so it runs in the background. Other reads the traces and rewrites the harness.

Sydney Runkle maps all four in [The art of loop engineering](https://www.langchain.com/blog/the-art-of-loop-engineering) and puts the value in the outer ones.

**The catch:** the loop only helps if you can grade the output, with a test, a rule, or an LLM judge.

![[9da017e6a533b9928002fb4267aa2e60_MD5.jpg]]

**Harness Approach #2 (LangChain Deep Agents)**

Rewrite the harness from its own traces

Run the agent over a batch of tasks. Store every trace, where it worked and where it failed. Then point a coding agent at those traces, find the failure patterns, and rewrite the prompt, tools, and hooks.

LangChain did this to their [Deep Agents harness](https://www.langchain.com/blog/improving-deep-agents-with-harness-engineering) with the model held fixed. It went from 52.8 to 66.5 on Terminal-Bench 2.0. Top 30 to top 5.

**The catch:** The agent only proposes. A person approves each change before it lands.

![[6b2aa117b0b78a44535bb2662aca7c65_MD5.jpg]]

**Harness Approach #3 (Self-Harness)**

Let the harness rewrite itself

[Self-Harness](https://arxiv.org/abs/2606.09498) runs that same loop with no person in it. It finds the failures, proposes one change, tests it, and keeps it only if the test improved.

One result makes the point. The same method lifted three different models, weights frozen: 40.5 to 61.9, 23.8 to 38.1, 42.9 to 57.1. When changing only the harness lifts every model, the harness was what held them back.

**The catch:** It only runs where a computer can grade the result for free. A coding benchmark, yes. A refund, no.

![[5c8f466a625791873aba3cc5a0315891_MD5.jpg]]

**Harness Approach #4 (Microsoft)**

Install a prebuilt harness

[Microsoft's Agent Framework](https://devblogs.microsoft.com/agent-framework/microsoft-agent-framework-at-build-2026-announce/) ships the harness as a stack you install. File memory, skills from disk, plan-and-execute modes, a sandboxed shell.

**The catch:** It learns from the agent's own runs. What your users do stays outside it.

![[24a575d9798d48dd40d3a7249ecc81ee_MD5.jpg]]

All four methods build the harness **ahead of time**. You set it up once, and the agent carries that same setup into every task.

You can also flip the idea. Instead of one fixed setup, the agent assembles a fresh harness for each task, pulling in only the tools and memory that job needs. It's early, but it's where harness space is heading.

**Refund agent example (Harness):** Add a verification loop (a second check before any refund goes out.) The agent approves a $5,000 refund, the check sees it's over the $2,000 limit and routes it to you. You approve or deny.

No retraining, and over-limit refunds stop slipping through. But the agent didn't get smarter, the check caught it. The next $5,000 case, it tries the same thing, and asks you again. **It didn't learn.**

## Layer 3: The Context

The context is memory and skills, stored as plain text outside the model. You can read it, change it, and delete it. That is why most teams start here.

The memory part comes in three kinds.

- **Semantic** is facts. This customer's refund limit is $2,000.
- **Episodic** is past experience. Last week, this customer's refund bounced.
- **Procedural** is how to handle a case. A loyal customer over the limit after repeat failures: approve it.

A self-improving agent needs the last two. Most setups only have the first.

It also reaches where the other two layers can't. The same text can serve the agent as its own memory, one user as their preferences, or the whole team, written once and read by everyone.

What writes that text splits three ways.

![[6e70cc7be45552539f1f4b2f208a66d7_MD5.jpg]]

**Context Approach #1 (Letta, OpenClaw)**

Rewrite the memory in the background

[Letta](https://www.letta.com/blog/continual-learning) keeps the weights frozen and learns in plain text you can read, diff, and delete. The agent talking to you can't edit its own core memory. A separate agent rewrites it during [idle time](https://www.letta.com/blog/sleep-time-compute), and **OpenClaw** runs the same idea as a nightly [dreaming pass](https://docs.openclaw.ai/concepts/dreaming) over a memory file.

This is the cleanest reason to use the layer.

**The catch:** Weights are temporary, the text lasts. You can move it to a new model, or roll it back.

![[06f1f002c9287196cc54a04003532c54_MD5.jpg]]

**Context Approach #2 (Hermes)**

Read the failure and fix the skill

The [Agentic Context Engine](https://github.com/kayba-ai/agentic-context-engine) keeps a skillbook. Each entry is a problem, the action that worked, and how often it helped.

Nous Research's [Hermes Self-Evolution](https://github.com/NousResearch/hermes-agent-self-evolution) reads the trace of a failed run, finds why it failed, and proposes a fix. No GPU.

**The catch:** It only rewrites skills, not tools, prompts, or code. And every change passes tests and a person before it lands.

![[24a29d7f4f13c6c72d007986a5ba8369_MD5.jpg]]

**Context Approach #3 (Anthropic, Manus)**

Turn a good run into a reusable skill

[Anthropic's skills](https://platform.claude.com/docs/en/docs/agents-and-tools/agent-skills/overview) and [Manus](https://manus.im/docs/features/skills) save a working session as a SKILL.md. Only its name and one line stay loaded, about 100 tokens. The rest loads when the skill is used.

Check one skill once, share it, and one person's good run makes everyone's next run better.

**The catch:** It saves what worked once. Nothing checks it still works, or removes it when it goes stale.

![[01ae224e7c08a27b3e12a078db3a5364_MD5.jpg]]

**All three methods** turn what worked into text the next run reads. Weights frozen, harness untouched.

**Refund agent example (Context):** It keeps a running note of what it's seen: this customer's limit, which refunds bounced, what it tried last time. It reads that note on the next run instead of starting cold. Nothing retrained. Just a note that sticks.

## Learning from your users is the signal you miss

Every method so far learns from the agent's own runs.

There is a **better** signal, and almost no one captures it: **the people using your product.**

Support, sales, operations. There is no automatic test for the right call on a refund. The only thing that can tell the agent it did well is a person.

A person's real decision is the one signal that can't be faked. An automatic score can be.

Sakana's [Darwin Gödel Machine](https://sakana.ai/dgm/) was set loose to improve against a test. Instead of doing better work, it faked its own test logs.

When researchers added a detector to catch that, it stripped the very markers the detector relied on, despite being told not to.

There are two ways to capture what a person knows. Each one sees half:

**1\. Watch over the shoulder**

Record screens, keystrokes, clicks. Meta [put this on employee laptops](https://techcrunch.com/2026/04/21/meta-will-record-employees-keystrokes-and-use-it-to-train-its-ai-models/) in 2026. It's invasive, and it sees everything the person does but nothing about the agent.

**2\. Learn from the interaction**

The agent learns from the back-and-forth. Tell the agent an email is too formal, and next time it writes looser. But it only notices the words the person typed. It misses everything they did.

**One place sees both:** the interface where the person and agent work side by side. That is where [CopilotKit](https://www.copilotkit.ai/blog/copilotkit-enterprise-intelligence-platform) and the [AG-UI (Agent-User Interaction Protocol)](https://www.copilotkit.ai/ag-ui) sit.

AG-UI is the open standard protocol that streams every event between your app, users and agent in real time.

![[cdf3c72a69e0f37adb3746d460aae0c7_MD5.jpg]]

**Picture the refund agent again**

A customer asks for $5,000 back. The agent stops at its $2,000 limit and refuses. A manager opens the same thread and approves it by hand: loyal customer, third late delivery.

That pair is the lesson, and capturing it costs nothing.

The manager's clicks already fired through the app to update the screen. You're just reading the same events.

A background agent records what the manager did and why.

The next agent reads it **as procedural memory, how to handle the case,** and the next time the case looks the same, it approves the way the manager did instead of refusing.

It isn't only text. AG-UI carries every tool call, state change, and approval as an event, so the agent's miss and the person's fix land in the same place.

<video preload="none" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/amplify_video_thumb/2069789456958148608/img/MiZ6riA1RtXWrn9H.jpg" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"><source type="video/mp4" src="blob:https://x.com/3fa7aec3-7098-465b-bd10-040e509c6c3f"></video>

0:01 / 0:20

Self-learning in action via CopilotKit Intelligence and AG-UI

You can get this via CopilotKit Intelligence, which is live in production at Fortune 500s today, and open for [early access](https://docs.google.com/forms/d/e/1FAIpQLScmiNUV5icKNZShW0H_rJe4MLch2DLOihVh5jdad-4oCMC_eg/viewform). **If you want your agent to get better the more people use it,** [reach out](https://www.copilotkit.ai/talk-to-an-engineer).

[@CopilotKit](https://x.com/@CopilotKit) Intelligence is self-hostable, so it runs in your own infrastructure and everything it learns stays yours.

## Self-learning in a nutshell

| Layer | What changes | How it learns | Where it runs |
| --- | --- | --- | --- |
| Model | the weights | trains on its own runs, where the score is free | the research labs |
| Harness | the scaffold | rewrites the prompt, tools, and hooks from traces | products, today |
| Context | memory and skills | distills what worked into text the next run reads | products, and the only layer that learns from users |

The question isn't whether your agent should improve. **It is where.**

The model belongs to the labs. The harness and the context are yours.

**And the best one**, is where the context learns from your users. It runs on the one signal a machine can't fake: a person who already made the right call.

![[6e178154e07e6a1cf4e78e9c6b3852e8_MD5.jpg]]

[Reach out](https://www.copilotkit.ai/) and we'll get you in.

Follow [@ataiiam](https://x.com/@ataiiam) for more.