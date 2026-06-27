---
title: "Anatomy of Agent SKILLS"
source: "https://x.com/Saboo_Shubham_/status/2048596364196761721"
author:
  - "[[@Saboo_Shubham_]]"
published: 2026-04-27
created: 2026-06-27
description: "Your agent has a 200k token context window. The instruction it actually needed was 400 tokens long. It ignored them anyway.Those 400 tokens ..."
tags:
  - "clippings"
---
![[87e14feeda0cd667e1e2e5724e730094_MD5.jpg]]

Your agent has a 200k token context window. The instruction it actually needed was 400 tokens long. It ignored them anyway.

Those 400 tokens were buried at position 142k under six tool definitions, four reference docs, and a brand guide nobody asked it to read.

This is the most common reason agents fail in production. It's not become of the model or the framework. **The prompt got too big and the right thing got buried.**

Skills are the cleanest fix for this problem. Not a bigger model. Not a bigger window. Not a smarter retriever. Just a small set of design decisions about where context lives and when it loads.

Five parts make the whole thing work. Here is how each one fits.

## 1\. A skill is a folder

A skill is not a Python class or a registered tool. It is a folder on disk with a markdown file inside it.

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HG23Jf2bwAA31Hq.jpg" src="https://video.twimg.com/tweet_video/HG23Jf2bwAA31Hq.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

![[e1574f640d58baf042986a7628bb2202_MD5.jpg]]

GIF

SKILL.md is the only required file. References hold docs the agent reads on demand. Assets hold templates and brand files. Scripts hold code the agent can execute. Everything except SKILL.md is opt-in.

Because a skill is just files, you version it in git. You diff it in pull requests. You copy it between projects. You publish it on GitHub. The format is the contract.

Same SKILL.md works across Claude Code, Codex, Gemini CLI, Cursor, Agent Development Kit, LangChain, and a growing list of agent tools and frameworks. One folder, many runtimes.

## 2\. The first two lines are the search index

Open any SKILL.md and the first thing you see is YAML frontmatter with two fields. These two fields are not just metadata. **They are the search index.**

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HG26TcXboAAC-uJ.jpg" src="https://video.twimg.com/tweet_video/HG26TcXboAAC-uJ.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

![[056ec017b06f5d8a27b3342ded73ac41_MD5.jpg]]

GIF

At session start, the agent loads the name and description of every installed skill. Roughly 100 tokens per skill. The body, the references, the scripts, all of it stays on disk.

When a request comes in, the model reads its own catalog and decides which skill to open. The description is what it matches against. Write a vague description and the skill never fires. Write a sharp one with concrete trigger words and the skill activates exactly when it should.

This single line is the most important piece of writing in the whole skill. People spend hours on the body and ten seconds on the description, then wonder why their skill never gets used.

Flip that ratio.

## 3\. Progressive disclosure is the whole trick

A single skill can hold tens of thousands of tokens of instructions and reference material. An agent with twenty skills could carry hundreds of thousands of tokens. Multiple full context windows of dead weight before the user has even typed.

Progressive disclosure prevents this with three loading tiers

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HG2698_aMAAfHad.jpg" src="https://video.twimg.com/tweet_video/HG2698_aMAAfHad.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

![[b213d35ef2dbc58bfa410e58a4e03c19_MD5.jpg]]

GIF

- **L1 metadata.** Name and description. Always loaded at session start. Roughly 100 tokens per skill.
- **L2 instructions.** The body of SKILL.md. Loaded only when the description matches a user task. Usually a few thousand tokens.
- **L3 references.** Files in references/, assets/, and scripts/. Loaded only when the L2 instructions explicitly point the agent there.

An agent with twenty skills installed pays the same upfront cost as an agent with one. Add a 21st skill tomorrow and yesterday's tasks cost what they cost yesterday.

The catch is that progressive disclosure only saves tokens if you actually use the tiers. Stuff every example into SKILL.md and the body balloons to 10K tokens. Now every task that triggers the skill pays that cost.

Keep SKILL.md short. Push edge cases, long examples, and reference tables into references/. The agent pulls them in only when it needs them.

## 4\. Agent routes the query

When a request comes in, the model does what you would do looking at a toolbox. Read the labels. Pick the right one. Open it.

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HG28RNNaUAADsVw.jpg" src="https://video.twimg.com/tweet_video/HG28RNNaUAADsVw.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

![[6accf8c020910aec7c4bad8481206937_MD5.jpg]]

GIF

The user says "clean up this messy CSV and dedupe rows." The model scans the catalog of descriptions. **pdf-forms**, low match. brand-voice, low match. **data-clean: CSV cleanup, dedup, nulls**, strong match. The body of **data-clean** loads. Work begins.

Two details matter here.

**The match is not vector retrieval.** The model decides directly from descriptions inside its own context. No embedding step. No similarity score. No separate routing layer. The LLM is the router.

**The match is also exclusive.** Only one skill activates per task. The others stay at L1. Their bodies never enter the context window. The cost of skills you do not need is essentially zero.

This is what makes skills feel different from MCP tools or function calls. Tools are always loaded, always visible, always paid for. Skills load only when relevant.

## 5\. Composition without context bloat

Scale this up. One agent, eight skills installed. Three different tasks come in over the course of a session.

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HG28crbbYAAk8IB.jpg" src="https://video.twimg.com/tweet_video/HG28crbbYAAk8IB.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

![[e59313212c8a252ca93f976e21ba0f8c_MD5.jpg]]

GIF

The skills the agent did not use stay at L1. Roughly 100 tokens each, no body, no references. The body cost is paid only on the tasks that need it.

The pattern matters beyond context economics.

**Teams can ship skills independently.** The data team owns **data-clean** and **sql-runner**. The design team owns brand-voice and deck-build. The platform team wires up the agent. Nobody coordinates. Nobody merges prompts. Nobody rebuilds the system prompt every time a new capability lands.

Skills are doing for agents what npm did for JavaScript. Small, focused, composable units behind a clear interface.

The package manager won JavaScript. The same shape is going to win agents.

## With Agent skills vs without

Put the five parts together and the gap between an agent built with skills and one built without is sharp enough to draw on a single page.

<video preload="auto" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/tweet_video_thumb/HG3IAeya0AAmjWu.jpg" src="https://video.twimg.com/tweet_video/HG3IAeya0AAmjWu.mp4" type="video/mp4" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"></video>

![[e9d4c53bf46aebc12fb341d8fd9faffb_MD5.jpg]]

GIF

If you build agents and you have not written a skill yet, pick one workflow you do every week. Write a skill for it. One folder, one SKILL.md, version it in git. Watch the agent activate it.

The format is just files. The leverage is enormous.

Start today. One skill. One workflow. See what changes.