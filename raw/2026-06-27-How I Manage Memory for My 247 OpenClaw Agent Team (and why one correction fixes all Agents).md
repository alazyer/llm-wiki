---
title: "How I Manage Memory for My 24/7 OpenClaw Agent Team (and why one correction fixes all Agents)"
source: "https://x.com/Saboo_Shubham_/status/2033026472856952849"
author:
  - "[[@Saboo_Shubham_]]"
published: 2026-02-13
created: 2026-06-27
description: "Most AI agents forget everything the moment the session ends.I run six agents 24/7 on a Mac Mini. Research, content, engineering, newsletter..."
tags:
  - "clippings"
---
![[c1def692613d2ce6fd48ff8177d67762_MD5.jpg]]

Most AI agents forget everything the moment the session ends.

I run six agents 24/7 on a Mac Mini. Research, content, engineering, newsletters, LinkedIn, coordination. They run on cron schedules. They wake up fresh every session with zero memory of what happened before.

That should be a disaster. It's not.

Because the memory isn't in the agent. It's in the files around it.

I wrote recently about how I built this autonomous agent team. The number one follow-up question: "How do you get them to actually remember anything?"

This is the answer. The memory architecture, the failures, and what actually worked.

## The day two problem

Every agent framework sells you on capabilities. Tool use. Multi-agent coordination. Streaming. Fancy orchestration patterns.

Nobody talks about what happens on day two.

Your agent nails the first session. Great output. You're excited. You close the terminal. You come back tomorrow. The agent has no idea who you are, what you told it yesterday, or what mistakes you already corrected.

You re-explain everything. Again.

This is the fundamental problem of autonomous agents. Every session starts from zero. The corrections you gave yesterday? Gone. The preferences you explained? Gone. Unless you make memory explicit, your agents wake up with amnesia every single time.

I hit this wall in week one. I told Kelly, my X/Twitter agent, to stop using emojis. She fixed it. Then I saw Rachel's LinkedIn draft. Emojis. I corrected Rachel. Next day, Pam's newsletter draft landed. Emojis.

Six agents. Same correction. Six separate conversations. Every single time.

I was spending more time repeating myself than doing actual work. A preference I told Kelly didn't reach Rachel. A rule I set for Pam didn't exist for Ross.

OpenClaw already handles the first two layers. SOUL.md files, daily logs, session structure. But I needed a third layer on top, one where corrections propagate across all six agents automatically.

## Three layers of Agent Memory

Agent memory isn't one thing. It's three layers, each solving a different problem.

![[bf729fd554bbef5e483da56564252379_MD5.jpg]]

**Layer 1: Working memory**

Markdown files loaded at boot. Every agent reads these before doing anything else.

SOUL.md tells the agent who it is. USER.md tells it who I am. MEMORY.md holds the curated lessons it's learned over time. Daily logs (memory/YYYY-MM-DD.md) hold yesterday's raw session notes.

This is the layer that makes agents feel like they remember.

Kelly has a MEMORY.md section called "BAD (what I did wrong)" where she lists every content pattern I've rejected. Emojis. Hashtags. LinkedIn tone. Bullet-point threads. She wrote that list herself, unprompted, after I gave corrections. It loads every session. She never repeats those mistakes.

Dwight has one rule in his memory that changed everything: "If Alex (our target persona) can't DO something with it TODAY, skip it." He went from flagging 47 stories to delivering 7. Same model. Different memory file.

**Layer 2: Session memory.**

Live conversations. Cron job outputs. Cross-agent messaging. Everything that happens during a single session.

This layer is ephemeral by design. When Kelly runs at 5 PM, she reads Dwight's intel, drafts posts, logs what she produced in her daily file, and the session ends. The conversation itself disappears. What mattered got written to a file. What didn't matter is gone.

That's not a limitation. That's garbage collection.

More memory isn't always better. Kelly's daily logs hit 161,000 tokens once and her output quality collapsed. She was reading so much history that she had no room left to do actual work. Now she loads only today plus yesterday. The important stuff lives in MEMORY.md anyway.

**Layer 3: Long-term memory**

Google Vertex AI Memory Bank. This is the layer that connects all six agents. It works through three channels.

- Auto-capture extracts facts from every conversation and stores them.
- File sync watches 21 workspace files and syncs changes when they update.
- Auto-recall pulls the 10 most relevant memories before every agent turn using similarity search.

This is the layer where one correction fixes all of them.

> Mar 14
> 
> Just open-sourced an OpenClaw Plugin for Google Vertex AI Memory Bank. Auto capture, auto recall with cross-agent persistent memory. Send this repo link to your agent and watch it install itself.

## How the memory layers work together

Here's how a single correction flows through all three layers.

I tell Monica "never use em dashes in any content." That's session memory. Monica logs it in her daily file. That's working memory for tomorrow. She distills it into her MEMORY.md. That's permanent working memory. The Memory Bank captures the fact from our conversation. That's long-term memory, shared across every agent.

![[2c876c9fc70c7f529c946240bdf6c80b_MD5.jpg]]

Now when any agent starts a session, that preference surfaces automatically. Kelly avoids them. So does Rachel. So does Pam. So does Ross. One conversation. Six agents updated. I never said it twice.

The same correction exists in three places, serving three purposes. Daily log for recent context. MEMORY.md for Monica's session startup. Memory Bank for cross-agent propagation.

Redundancy is the point. If one layer fails, the others catch it.

## What I tried before this worked

I did not land on this architecture on the first attempt. Let me save you the failures.

**Attempt 1:** Self-hosted Mem0 with Ollama and SQLite. Crashed constantly. Stored nothing useful. The embeddings were unreliable and the retrieval was worse. Abandoned after a week.

**Attempt 2:** OpenClaw's built-in memory-core with Gemini embeddings and hybrid search. Worked technically. But it indexed raw session transcripts. That meant agents were recalling operational noise like "checking cron status... all healthy" instead of actual preferences and decisions.

The noise problem was the killer. Memory systems that store everything are memory systems that recall garbage.

The current system works because it separates capture from recall. The Memory Bank uses an LLM to extract facts from conversations, not raw transcripts. What gets stored are actual preferences, decisions, and lessons.

## Start here

You don't need all three layers on day one. Start with one file.

Write a MEMORY.md with your three most important preferences. Make your agent load it at session start.

Give feedback for a week. When you find yourself correcting the same thing twice, add it to the file.

That alone will cut your repeat corrections in half within a week.

When you hit the wall I hit, the one where you're correcting six agents for the same thing, that's when you need Layer 3. I open-sourced the plugin that solved it for me: [openclaw-vertex-memorybank](https://github.com/Shubhamsaboo/openclaw-vertexai-memorybank)

Start with one file. One correction. The rest will follow.