---
title: "The Ultimate Guide to running Free Local Coding Agents with Hermes"
source: "https://x.com/Saboo_Shubham_/status/2057880306804527566"
author:
  - "[[@Saboo_Shubham_]]"
published: 2026-05-23
created: 2026-06-27
description: "Local coding agents finally work for me with Hermes as the orchestrator. Not because the models got better. Because I stopped asking them to..."
tags:
  - "clippings"
---
![[7f435b70b822d59d0d6818cd790be40d_MD5.jpg]]

Local coding agents finally work for me with Hermes as the orchestrator. Not because the models got better. Because I stopped asking them to be Claude Code.

Now a free local model handles the boring work, and I only spend Claude and Codex tokens on the parts that actually need them.

If you want to run free local coding agents with Hermes, this is the setup.

Hermes Agent routes the work. Kanban tracks the state. SmallCode runs locally on a Mac Mini through Ollama. Claude Code and Codex review the hard parts.

**The local coding agent has one and only one job. Finish the card.**

![[cbe477ad956015215a3bffc8f1149ade_MD5.jpg]]

## The wrong question

The default question people ask is "can this local agent replace Claude Code or Codex for me?"

That question is the trap. It forces a yes or no answer about a thing that does not have a yes or no answer. A local agent running on a 7B model is not trying to be Claude Code. It cannot be. It does not have the model behind it, the reasoning depth, or the context window.

But it can do a surprising amount of real work if you stop asking it to act like a senior engineer. The better question is what work can a local agent do reliably if the task is small, the context is tight, and a stronger agent reviews the result?

That question has a real answer. And the answer is more than you think.

## One job per agent

This is the same principle I run my OpenClaw squad on. Each agent does one thing well, and Hermes Agent now sits above that squad and routes coding work the way Monica routes content work.

A local agent is not a senior engineer. It is a junior dev who is fast, cheap, private, and good at narrow tasks. Once I started treating it that way, everything else fell into place.

## The stack

**Hermes Agent** is the orchestrator. It owns the conversation, understands intent, and decomposes work into cards.

is the task bus. Every piece of work lives on a board with status, scope, acceptance criteria, and evidence.[Hermes Kanban](https://hermes-agent.nousresearch.com/docs/user-guide/features/kanban-tutorial)

is the local coding agent. It runs on the Mac Mini against any model you can serve through Ollama, a 7B coder, a 14B, whatever your machine handles. It does scoped patches, test fixes, lint cleanup, docs, and mechanical refactors.[SmallCode](https://github.com/Doorman11991/smallcode)

**Claude Code and Codex** handle the hard parts. Architecture decisions, ambiguous bugs, cross-file reasoning, final review.

The image at the top shows what this looks like in practice. A Telegram prompt comes in. Hermes decomposes it. Easy cards route to SmallCode on the Mac Mini. Hard cards route to Claude Code or Codex. The board tracks everything. Nothing disappears into terminal history.

**The board is what makes it useful.** Not the local agent. The board.

> May 13
> 
> Codex /goal builds it. Claude Code /goal review and refines it. Hermes /goal manages the orchestration and handoff. All tracked on a single Kanban Board and agents keep running in the loop.

## Why local agents usually fail

The default way to use a local coding agent is to open a terminal, run it, give it a prompt, and see what happens.

Sometimes it works. Often it does not. The agent loses context on long sessions. Edits files outside the intended scope. Generates plausible-looking patches that break tests. Forgets constraints from earlier in the conversation. Confuses "I ran the command" with "the command succeeded."

Some of this is the model. A 7B coding model has real limits compared to Claude or GPT. Less reasoning depth. Smaller context window. Weaker tool calling.

But a lot of it is the job description.

Ask a small model to own an ambiguous feature end to end and it will fail. Ask the same model to fix one failing test in one file with one clear test command, and it will probably finish. The model did not get smarter. The job got smaller.

Small work. Clear acceptance criteria. Easy verification.

That is the lane.

## I don't run the workflow. Hermes does.

This is the part most people miss when they hear "agent orchestration." They picture themselves at a keyboard, dispatching tasks, checking outputs, moving cards between columns. That is not the setup.

I send Hermes a prompt on Telegram. Hermes decides what to do.

If the prompt is something like "clean up the email helpers in this repo," Hermes creates the cards. Assigns them. Watches SmallCode run on the Mac Mini. Reads the result. Asks Claude Code to review. Creates a follow-up card if review fails. Logs every run on the board with full history.

If SmallCode hard-fails three times in a row, the circuit breaker trips and the card moves to blocked. Hermes pings me on Telegram. I do not have to be watching for it to happen.

If the work is too ambiguous for SmallCode in the first place, Hermes routes it to Claude Code or Codex directly. The decision is made at intake, not after a failed attempt.

I am not the dispatcher. I am the person who says what should exist. Hermes is the person who makes sure it does.

## Put the routing rules in a file

The routing logic does not live in my head. It lives in an AGENTS.md file at the root of every repo Hermes works on. Do not treat SmallCode output as final until tests and review pass.

```text
# Agent routing rules

Hermes is the orchestrator.
Use Kanban to track coding work.

Use SmallCode for:
- small patches
- test fixes
- lint cleanup
- docs near code
- mechanical refactors
- applying review comments

Use Claude Code or Codex for:
- architecture decisions
- ambiguous bugs
- cross-file reasoning
- high-risk changes
- final review of SmallCode output

Every coding card must include:
- goal
- files or module scope
- constraints
- test command
- acceptance criteria
- expected evidence

Do not treat SmallCode output as final until tests and review pass.
```

Hermes reads this on every session. The routing happens automatically. I do not have to re-explain my operating model every time I open Telegram.

Same pattern I use across my OpenClaw squad. SOUL.md for identity. AGENTS.md for rules. Memory files for continuity. The agent wakes up already knowing how the work should move.

## The Kanban board is non-negotiable

Without a board, agent runs disappear into terminal history. With one, every task has an owner, a status, scope, acceptance criteria, a test command, run history, review notes, and final evidence.

The board gives Hermes a durable queue. Tasks survive restarts. Run history survives context resets. When SmallCode retries a card after a review block, the second attempt sees the first attempt's findings and addresses them directly. No starting from scratch.

This is what makes smaller agents useful. They do not need to understand the entire company roadmap. They need to finish one card.

## Smaller cards win

Hermes decomposes my prompts into cards. The quality of that decomposition determines whether SmallCode can finish the work.

A bad card looks like "clean up the repo."

A good card looks like "remove unused helper functions from src/utils/email.ts, do not change public behavior, run npm test after the edit, return the diff summary and test result."

I do not write the second version. Hermes does. But Hermes only writes good cards if I prompt it with enough context. "Clean up the email helpers in the auth module" gives Hermes enough to produce four or five well-scoped cards. "Make the codebase better" gives Hermes nothing useful and produces vague cards that SmallCode will fail.

**The card is the spec.** Writing better prompts to Hermes is the same discipline as writing better prompts to any agent. The work upstream is what makes the work downstream possible.

## Route by what the task looks like

The question is not which agent is smartest. The question is which worker is right for this card.

SmallCode gets the card when the task is narrow, repetitive, low risk, easy to verify, mostly mechanical, scoped to a few files.

Claude Code or Codex get the card when the task is ambiguous, architectural, high risk, cross-repo, dependent on deep reasoning, hard to verify automatically.

Hermes makes this call based on the AGENTS.md rules and the shape of the card. If a card sits in the middle, Hermes asks me on Telegram. The decision happens before the work starts, not after a failed attempt.

## Review is mandatory

SmallCode output is never final by default. The loop is:

![[eaf9d0e3b528191eae3145cb440b4099_MD5.jpg]]

Hermes runs this loop without me. The review card gets created the moment SmallCode marks its card complete. If review passes, the card moves to done. If review blocks, Hermes creates a follow-up with the specific feedback and routes it back to SmallCode. The retry sees the block reason in its context.

This is what turns a local agent from a risky autonomous worker into a supervised one.

A local agent that completes 20 to 30 percent of boring repo tasks under supervision is already useful. Especially if the alternative is those tasks never getting done. Test coverage that nobody writes. Dead code that nobody removes. Docs that nobody updates.

SmallCode does not need to beat Claude Code. It needs to make the team cheaper, faster, and less blocked on the work nobody wants to do.

## The mistakes I made first

Asking SmallCode to own the whole feature. It cannot. Give it one file, one patch, one test command.

Skipping review. SmallCode output is a draft, not a merge candidate. Hermes enforces this now.

Running agents outside the board. If the work is not tracked on Kanban, it does not exist. The board is the source of truth.

Starting with risky production work. Start with tests, docs, cleanup, low-risk fixes. Build intuition before raising stakes.

Treating SmallCode like a worse Claude Code. It is not Claude Code. It is a worker. Different job. Different expectations.

## The shift

The future of local coding agents is not one local agent replacing Claude Code.

It is orchestration. Hermes routes the work. Kanban tracks the state. SmallCode executes scoped local tasks. Claude Code and Codex review the hard parts. I stay out of the loop until a decision needs to be made.

Local agents become useful the moment you stop expecting them to be senior engineers and start treating them like team members with a narrow job description.

Not a better local agent. A better job for the local agent.

Start with one repo. One board. One small card. See what a local agent can actually do when you give it work it can finish.