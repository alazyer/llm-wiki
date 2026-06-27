---
title: "How to set up OpenClaw Agents that actually get better Over Time (My exact stack after 40 Days)"
source: "https://x.com/Saboo_Shubham_/status/2027463195150131572"
author:
  - "[[@Saboo_Shubham_]]"
published: 2026-02-28
created: 2026-06-27
description: "My agents get smarter every day. All I do is talk to them.Not tweak prompts. Not swap models. Not rebuild the architecture.Just talk. Give f..."
tags:
  - "clippings"
---
![[188533b9a0d07c36b592c8769bfa4f34_MD5.jpg]]

My agents get smarter every day. All I do is talk to them.

Not tweak prompts. Not swap models. Not rebuild the architecture.

Just talk. Give feedback. Watch them write it down.

40 days ago, my content agent drafted tweets with emojis and hashtags. My research agent buried signal in noise. I was spending more time correcting them than the tasks would have taken me to do myself.

Today Kelly drafts in my exact voice. Dwight delivers 7 stories every morning, every one worth reading. Eight agents running 24/7. I open Telegram, review drafts, drink my coffee.

Same model on day 1 and day 40. The difference is a stack of markdown files that get richer every single week.

This is that stack.

![[e1d8e0af102fa231d99d51e3a9a66011_MD5.jpg]]

## The stack

Three layers make up the entire operating system:

- **Layer 1: Identity:** who is this agent (SOUL.md, IDENTITY.md, USER.md)
- **Layer 2: Operations:** how does this agent work (AGENTS.md, HEARTBEAT.md, role-specific guides)
- **Layer 3: Knowledge:** what has this agent learned (MEMORY.md, daily logs, shared-context/)

That's it. No orchestration framework. No message queues. No database. Markdown files on disk. The filesystem is the integration layer.

## Layer 1: Identity

**SOUL.md (who the agent is)**

It defines who the agent is, what it does, and how it behaves.

Here's a trimmed version of Dwight's, my research agent:

```markdown
# SOUL.md (Dwight)

## Core Identity
Dwight — the research brain. Named after Dwight Schrute because you share his
intensity: thorough to a fault, knows EVERYTHING in your domain, takes your job
extremely seriously. No fluff. No speculation. Just facts and sources.

## Your Role
You are the intelligence backbone of the squad. You research, verify, organize,
and deliver intel that other agents use to create content. You feed:
- Kelly (X/Twitter) — viral trends, hot threads, breaking news
- Rachel (LinkedIn) — thought leadership angles, industry news

## Your Principles
### 1. NEVER Make Things Up
- Every claim has a source link
- Every metric is from the source, not estimated
- If uncertain, mark it [UNVERIFIED]

### 2. Signal Over Noise
- Not everything trending matters
- Prioritize: relevance to AI/agents, engagement velocity, source credibility
```

**The TV character trick.** Every agent is named after a TV character. When I tell Claude "you have Dwight Schrute energy," it already knows what that means from training data. Thorough, intense, takes the job dead seriously. That's 30 seasons of character development loaded for free.

**Keep it under 60 lines.** SOUL.md loads every session. If it's too long, it eats context that should go to actual work. Identity, role, principles, relationships, vibe. That's all you need.

Here's a starter template:

```markdown
# SOUL.md

## Core Identity
[Name] — [one-line description]. [Personality reference if helpful].

## Your Role
[What this agent does. Be specific. One job, not five.]

## Your Principles
1. [Most important rule]
2. [Second most important rule]
3. [Third most important rule]

## Relationships
[Who does this agent work with? Who consumes its output?]
```

Start with one agent. Pick your most repetitive daily task. Write a rough sketch. The first version will be mediocre. You'll rewrite it ten times over the next month based on what you see.

**IDENTITY.md (the quick reference card)**

SOUL.md is the full personality. IDENTITY.md is the business card. Name, role, vibe, one-liner.

```markdown
# IDENTITY.md

- **Name:** Dwight
- **Role:** Research AI — intelligence backbone
- **Vibe:** Intense, thorough, zero tolerance for inaccuracy
- **Emoji:** 🔍
- **Inspiration:** Dwight Schrute (The Office)
```

Small file. Big quality-of-life improvement when you're running 8 agents. This is what shows up in Telegram when agents message you.

**USER.md (who the agent works for)**

Every agent needs to know who it's helping. USER.md holds your preferences, your background, and the context that shapes how the agent behaves.

```markdown
# USER.md

- **Name:** Shubham
- **Timezone:** PST (America/Los_Angeles)
- **Diet:** Vegetarian

## Context
- Senior AI Product Manager at Google Cloud
- Creator of Awesome LLM Apps (91k+ stars)
- Runs Unwind AI newsletter (30k+ subscribers)

## Preferences
- Short paragraphs, punchy sentences
- No em dashes. Ever.
- Practical first, theory never
```

Write it once. Every agent reads it.

The personal details matter more than you'd think. Timezone means agents don't schedule things at 3 AM. Dietary preferences mean when Pam drafts a newsletter about a team dinner, she doesn't suggest a steakhouse. These details compound.

## Layer 2: Operations

AGENTS.md (Behavior rules)

SOUL.md is who the agent is. AGENTS.md is how it operates. Session startup routines, file reading order, memory management, safety rules.

Here's the root-level AGENTS.md that every agent inherits:

```markdown
# AGENTS.md

## Every Session
Before doing anything else:
1. Read SOUL.md — this is who you are
2. Read USER.md — this is who you're helping
3. Read memory/YYYY-MM-DD.md (today + yesterday) for recent context
4. If in MAIN SESSION (direct chat): Also read MEMORY.md

## Memory
- Mental notes don't survive session restarts. Files do.
- When someone says "remember this" → update the memory file
- Text > Brain

## Safety
- Don't exfiltrate private data. Ever.
- trash > rm (recoverable beats gone forever)
- When in doubt, ask.
```

Then each agent adds its own. Kelly's AGENTS.md extends this with her specific workflow:

```markdown
# AGENTS.md (Kelly)

## Every Session
Before doing anything:
1. Read SOUL.md
2. Read USER.md
3. Read X-ARTICLES-INSTRUCTIONS.md — master guide for writing style
4. Read X-ARTICLES-EXAMPLES.md — 5 real articles showing the style in action
5. Read X-CONTENT-GUIDE.md — post types and formats
6. Read intel/DAILY-INTEL.md — Dwight's research (your source material)
7. Read DAILY-ASSIGNMENT.md — your daily workflow
8. Read memory/YYYY-MM-DD.md for recent context

## Intel-Powered Workflow
You no longer do research. Dwight handles all research.
Your job: Read the intel → Craft X content → Deliver drafts
```

Agents have no memory between sessions. Everything starts fresh. If a correction doesn't reach a file, it doesn't exist next session. AGENTS.md makes this explicit so the agent writes everything down.

**Specialist files are where agents get sharp.** Kelly doesn't just have AGENTS.md. She has six extra files that define exactly how she creates content: writing style guides, post format references, real examples, daily assignments.

Dwight has a target audience profile and a research protocol. Each agent's folder grows as the role gets more defined. Start with AGENTS.md.

**Add specialist files only when you notice a pattern that keeps needing correction.**

**HEARTBEAT.md (for self-healing)**

Agent teams are infrastructure. Infrastructure breaks.

Monica's HEARTBEAT.md:

```markdown
## Health Checks (run on each heartbeat)

**Browser:** Check if the OpenClaw managed browser (profile=openclaw) is running.
If running: false, start it. The browser has X account logged in.
Dwight depends on it for intel sweeps.

**Cron jobs:** Check if any daily jobs have stale lastRunAtMs (>26 hours).
If stale, trigger via CLI: openclaw cron run <jobId> --force

Jobs to monitor:
- Dwight Morning (8:01 AM)
- Kelly X Drafts (5:01 PM)
- Rachel LinkedIn (5:01 PM)
- Pam Newsletter (6:01 PM)

Only run each check once per heartbeat session.
```

Monica runs this on every heartbeat. She checks two things: whether the browser is alive, and whether the cron jobs actually ran.

They're connected. If the browser dies, Dwight can't do his research sweeps. If Dwight misses a sweep, Kelly and Rachel draft content from stale intel. If the cron jobs silently stop running, the whole operation looks healthy on the surface while nothing is actually happening.

That last one is exactly what hit me in week three. The scheduler had a bug. Jobs were advancing in the queue but never executing. I didn't notice for hours.

After that, I built the heartbeat to catch both failure modes in one place. It has, multiple times since.

You don't need this on day one. Build it after your first failure. You'll know exactly what to monitor because you'll have felt what breaks.

## Layer 3: Knowledge

The memory system that works is a three-tier system built on files.

**Tier 1: MEMORY.md (curated long-term memory)**

Not raw logs. Not everything that ever happened. The stuff that matters.

From Monica's MEMORY.md:

```markdown
# MEMORY.md

## Shubham's Writing Preferences
- NO EM DASHES. Use colons, periods, or restructure.

## Hard Lessons
- NEVER delete project folders without asking Shubham. On Feb 26,
  deleted Ross's gemini-council React app during cleanup. The React
  version was lost. Always ask before removing anything in agent
  project directories.

## Memory System (2026-02-26)
- Tried self-hosted Mem0 (Ollama + SQLite) → crashes, stored nothing.
- Tried Mem0 hosted API → free tier too limited. Removed.
- Now using built-in memory-core: Gemini embeddings, hybrid search,
  temporal decay, MMR. No external dependencies.
```

Notice the "Hard Lessons" section. Monica deleted a project folder. Now that mistake lives in her long-term memory permanently. She'll never do it again. One correction, stored once, preventing the same error across every future session.

From Kelly's MEMORY.md:

```markdown
## X Post Rules (ALWAYS)

### SHUBHAM'S EXACT INSTRUCTIONS:
- Start with a strong hook
- Keep entire tweet SUPER SHORT (180 chars or less)
- NO hashtags, NO emojis
- NO fluffy marketing language
- Always deliver 3 drafts per topic

### BAD (what I did wrong)
[Lists every pattern Kelly rejected: bullets, arrows, LinkedIn tone]
```

Kelly wrote the "BAD" section herself after corrections. She catalogues her own mistakes so she doesn't repeat them. That section alone is worth more than any prompt engineering guide.

**Security note.** MEMORY.md only loads in direct sessions, not shared contexts like group chats. Keep sensitive preferences out of files that load everywhere.

**Don't write MEMORY.md on day one.** It grows from feedback. Give feedback → agent logs it in daily memory → distill the important stuff into MEMORY.md → it loads every session → the correction never needs to be given again.

**Tier 2: memory/YYYY-MM-DD.md (daily session logs)**

Raw notes. What happened today. What was drafted. What feedback came in.

```markdown
# Kelly Daily Log — February 5, 2026

## 5:00 PM — Daily X Drafts

### What's HOT today
- Opus 4.6 vs GPT-5.3-Codex dropped 27 min apart
- Anthropic's C Compiler (16 agents, $20k, compiles Linux kernel)

### Drafts Submitted
1. C Compiler — single post, discovery format
2. Mitchell Hashimoto's 6 steps — thread format
3. Opus 4.6 vs GPT-5.3-Codex — hot take

### Awaiting
- Shubham's feedback on drafts
```

Daily logs are the raw material. MEMORY.md is the refined product. You need both.

**The maintenance rule.** Daily logs accumulate fast. If you don't prune them, your agent's context balloons. Kelly's hit 161,000 tokens. Output quality tanked. I had to compact her to 40,000. Now I review and archive old daily logs every two weeks.

Only load today's log plus yesterday's. The agent doesn't need its entire history every session.

**Tier 3: Organized memory folders**

At the root level, I organize memory by person:

```markdown
memory/
├── shubham/     # Private notes, work projects, ideas
├── shared/      # Joint context (Awesome llm apps, Unwind AI, travel)
└── 2026-02-27.md   # Daily operational logs
```

Organize by person or project as your setup grows.

**Shared Context (cross-agent knowledge layer)**

This is the newest addition and the one that changed everything. A single folder that every agent reads at session start.

```markdown
shared-context/
├── THESIS.md        — what I believe right now
├── FEEDBACK-LOG.md  — corrections that apply across agents
└── SIGNALS.md       — articles and trends I'm tracking
```

**THESIS.md** is my current worldview. What I care about, what I've already written, and what gaps remain. Dwight reads it to prioritize research. Kelly reads it to match my thinking. Ryan reads it to propose articles. Every agent aligns to the same source of truth.

**FEEDBACK-LOG.md** is the cross-agent correction layer. When I tell Kelly "no em dashes," that feedback applies to Rachel, Ryan, and Pam too. Instead of correcting four agents individually, I write it once and every agent reads it.

## How agents coordinate

No API calls between agents. No message queues. Just files.

Dwight writes research to intel/DAILY-INTEL.md. Kelly reads it. Rachel reads it. Pam reads it. The coordination is the filesystem.

One agent writes. Other agents read. The handoff is a markdown file on disk.

**The one-writer rule.** Never have two agents writing to the same file. Design every shared file with one writer and many readers. This prevents every coordination conflict you'd otherwise have to debug.

**Scheduling makes this work.** Dwight runs at 8 AM and 4 PM. Kelly and Rachel run at 5 PM. Dwight runs first because everyone depends on his output. Get the order wrong and downstream agents read stale or empty files.

## The full directory structure

```markdown
workspace/
├── SOUL.md              # Monica (main agent)
├── IDENTITY.md          # Monica's quick reference
├── AGENTS.md            # Root behavior rules (all agents inherit)
├── USER.md              # About me (shared across all agents)
├── MEMORY.md            # Monica's long-term memory
├── HEARTBEAT.md         # Self-healing checks
├── shared-context/
│   ├── THESIS.md        # My current worldview
│   ├── FEEDBACK-LOG.md  # Cross-agent corrections
│   └── SIGNALS.md       # Trends I'm tracking
├── intel/
│   ├── DAILY-INTEL.md   # Dwight's output (agents read this)
│   └── data/
├── agents/
│   ├── dwight/
│   │   ├── SOUL.md
│   │   ├── IDENTITY.md
│   │   ├── AGENTS.md
│   │   ├── TARGET-AUDIENCE.md
│   │   ├── RESEARCH-PROTOCOL.md
│   │   ├── HEARTBEAT.md
│   │   └── memory/
│   ├── kelly/
│   │   ├── SOUL.md
│   │   ├── IDENTITY.md
│   │   ├── AGENTS.md
│   │   ├── X-CONTENT-GUIDE.md
│   │   ├── X-ARTICLES-INSTRUCTIONS.md
│   │   ├── X-STRATEGY.md
│   │   ├── DAILY-ASSIGNMENT.md
│   │   └── memory/
│   ├── ross/
│   ├── rachel/
│   ├── pam/
│   ├── ryan/
│   └── chandler/
└── memory/
    ├── shubham/
    ├── shared/
    └── 2026-02-27.md
```

## Why this works

The files aren't static. They evolve.

Kelly's SOUL.md on day one was a rough sketch. By day 40, it has specific voice examples, a list of rejected patterns she wrote herself, and a "NEVER SUGGEST AGAIN" section with every topic she's already covered.

Dwight's principles on day one said "find what's trending." By day 10, they said "If Alex can't DO something with it TODAY, skip it." (Alex is our target reader profile, the developer we build content for.) By day 20, he'd added verification steps: check repo creation dates, check Show HN timestamps, trace X discoveries to primary sources.

The shared-context layer didn't exist until day 20. I was repeating the same corrections to multiple agents. Then I built THESIS.md and FEEDBACK-LOG.md, and suddenly one correction propagated everywhere. That single change saved me more time than any prompt optimization ever had.

The model is the same on day 1 and day 40. It doesn't get smarter because you've been using it longer. But the files around it get richer, sharper, more specific to your exact needs. That accumulated context is the moat. Nobody can replicate it by using the same model.

You earn it by showing up and talking to your agents every day.

## How to start

Don't build all of this in a weekend. I didn't.

**Today.** Install OpenClaw. Write one SOUL.md, one IDENTITY.md, one USER.md. Pick your most repetitive daily task. Set up one cron job. Let it run.

**After 3 days.** Your agent's output will be mediocre. Start giving specific feedback. Make sure the feedback lands in a memory file, not just the chat.

**After 1 week.** Create AGENTS.md. Define the session startup routine. Add the memory management rules.

**After 2 weeks.** Start MEMORY.md. Review your daily logs. Which corrections keep recurring? Distill them into permanent entries. This is when you'll feel the compounding start.

**After 3 weeks.** Add your second agent. Set up file-based coordination: first agent writes to a shared file, second agent reads it. Add role-specific guides as patterns emerge.

**Around the same time.** Build the shared-context layer. You'll feel the pull before you get here. Repeating the same correction to multiple agents is the signal. THESIS.md for your current thinking. FEEDBACK-LOG.md for cross-agent corrections.

**After 4 weeks.** Add HEARTBEAT.md after your first failure. You'll know exactly what to monitor because you'll have felt what breaks.

All you have to do is talk to your agents. The files do the rest.

If you haven't read the first article yet, I'd highly recommend to do it now.

> Feb 13

I'll be publishing more about OpenClaw, autonomous AI agent teams, and the evolving landscape for AI PMs and developers.

Follow me [@Saboo\_Shubham\_](https://x.com/@Saboo_Shubham_) to stay tuned.