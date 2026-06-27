---
title: "What 30 days of running 24/7 AI Agent team taught me"
source: "https://x.com/Saboo_Shubham_/status/2023105475919884562"
author:
  - "[[@Saboo_Shubham_]]"
published: 2026-02-16
created: 2026-06-27
description: "I built a 24/7 AI agent team and almost killed it on day four.Every output was garbage. My content agent wrote like a LinkedIn influencer. M..."
tags:
  - "clippings"
---
![[581e6ffc8807535c171611b9bd128427_MD5.jpg]]

I built a 24/7 AI agent team and almost killed it on day four.

Every output was garbage. My content agent wrote like a LinkedIn influencer. My research agent flagged 47 stories and 40 were noise. I was spending more time correcting agents than the tasks would have taken me to do manually.

Then the corrections started compounding.

Week four, I'm shipping drafts with two-word edits. Same model. Same prompts. The only thing that changed was 30 days of accumulated context living in files the agents read every morning.

That's the lesson no tutorial teaches. And it's the reason most people quit before the compounding kicks in.

## The ugly first week

Kelly, my X/Twitter agent, joined the team on January 15th. First day was just onboarding. Setting up the cron jobs but no output yet.

First real drafts came the next day. Long. Bullet points. Arrows. LinkedIn-influencer tone. The kind of posts that open with "I'm excited to announce" and end with a string of hashtags.

Not my voice. Not even close.

I spent more time correcting agents in week one than the tasks would have taken me to do manually.

**The first week feels like a net negative.** You set up this system expecting leverage and instead you're editing garbage output while also maintaining the infrastructure that produces it.

I call this corrective prompt-engineering. You don't design the perfect agent personality upfront. You start with a rough sketch in SOUL.md, watch how the agent behaves, and course-correct over time. Exactly like managing real people. The first version is mediocre. The tenth is good. The thirtieth is great.

I almost scrapped the whole thing before the corrections started compounding.

## What corrective prompt-engineering actually looks like

The key is specificity. Not "make this better." Specific enough to be a rule.

This is what went into Kelly's memory file after her first drafts:

```markdown
Start with a strong hook. Keep entire tweet SUPER SHORT
(180 chars or less). NO hashtags, NO emojis. NO fluffy
marketing language (no 'game-changer', 'groundbreaking').
```

Kelly's memory file now has a section called "BAD (what I did wrong)." She wrote it herself after corrections. It lists every pattern I rejected: bullet points in tweets, arrow formatting, LinkedIn tone, marketing buzzwords.

She also has a "Shubham's example of GOOD tone/style" section with my most popular posts as templates. Not abstract guidelines. Real posts she reverse-engineers every session.

By day 10, emojis were gone. By day 15, she was matching my sentence structure. By day 20, drafts needed one or two word changes before publishing.

**Dwight**, my research agent, went through a parallel arc.

His first intel sweep: 47 stories. 40 of them noise. Every trending repo. Every minor model update. Every rumor from unverified sources.

So I gave him THE RUTHLESS RULE: "If Alex can't DO something with it TODAY, skip it." Alex is our target reader profile. A developer who builds with AI daily. If a story doesn't give Alex something actionable, it doesn't make the cut.

Day 10: 18 stories, mostly signal. Day 25: 7 stories, every one worth reading.

Not because I picked which 7 to keep. Because weeks of "this matters, this doesn't" built a filter that persists across sessions.

## The failure that taught me the most

A week in, Dwight flagged a tool as a new launch worth covering. Clean summary. Good framing. I told Kelly to draft a tweet.

Then I looked closer. The tool wasn't new. Someone had just tweeted about it that day. Dwight saw the tweet, assumed it was a launch, and flagged it.

This kept happening. Old projects resurfacing because someone mentioned them on X. Repos I'd covered weeks ago showing up as "new finds." Dwight was doing exactly what I asked: find what people are talking about. The problem was that talked about today doesn't mean launched today.

My feedback: "Just because someone tweets about a tool today doesn't mean the tool launched today."

Dwight added verification steps to his workflow. Check Show HN timestamps. Check repo creation dates. Check actual launch dates. If the project is older than a week and nothing meaningfully changed, skip it.

I also removed GitHub trending as a source entirely. Too much noise, too many resurfacing repos. Replaced it with [goodailist.com](https://goodailist.com/) which filters for genuinely new projects.

**The agent handles research. I handle editorial judgment. The boundary between those two needs to be explicit, and it only becomes clear through failure.**

## What broke (and when)

Week three, the cron scheduler had a bug. Jobs were advancing in the queue but never actually executing. Agents were "scheduled" but producing nothing. I didn't notice for hours because everything looked healthy on the surface.

That's the day I built the heartbeat self-healing pattern. Monica, my chief-of-staff agent, now checks every heartbeat whether cron jobs actually ran. If any job is more than 26 hours stale, she forces a re-run.

Then the context problem hit. Kelly's context ballooned to 161,000 tokens. Dwight's hit 156,000. Both were loading so much memory and history that they had almost no room left for actual work. Output quality degraded. Response times slowed.

I had to compact both agents. Kelly went from 161K to 40K tokens. Dwight from 156K to 43K. The process: review every memory file, keep what's actively useful, archive everything else.

**Memory files need maintenance the same way a codebase needs refactoring.** Left unchecked, they accumulate cruft that actively hurts performance. I now do a memory review every two weeks.

## What actually compounds

Three things get better over time. Everything else is static.

**Memory files** are preferences learned from feedback. "Shubham doesn't use emojis." "Our audience cares about what they can build, not model benchmarks." "When covering a new tool, lead with the problem it solves, not the company that built it."

Every correction that reaches a memory file is a correction you never give again.

**Skill files** are rules codified from failures. "Keep tweets under 180 characters." "Verify launch dates before flagging a project as new." "Hook must be under 15 words." These are prescriptive. They tell the agent how to do the job, not just what you prefer.

The distinction matters. Memory is "what I've learned about working with Shubham." Skill is "how to do this specific task correctly." Both compound. Skill files compound faster because they're prescriptive.

**Feedback loops** are the mechanism that feeds both. And this is the one most people skip.

They set up the agent, let it run, and wonder why it doesn't improve.

The loop has to close. When Kelly drafts something and I say "too long, cut the first paragraph," that feedback needs to land in a file. Not in the chat. In a file she loads next session. If feedback stays in the conversation and never reaches persistent storage, the agent makes the same mistake tomorrow.

My pattern: give feedback → agent updates memory or skill file → next session starts with that lesson loaded. Simple. But you have to be disciplined about it.

## The three phases every agent team goes through

![[db5f6c822ffe9ffc375a9e277176c1c4_MD5.jpg]]

Every agent team goes through the same arc.

**Phase 1: Mediocre everything (days 1-7).** The agent produces generic output. You spend more time correcting than you save. This is where most people quit.

**Phase 2: Specific competence (days 8-21).** Feedback accumulates. The agent stops making the obvious mistakes. Output goes from "technically correct but useless" to "pretty good with minor edits." You start saving real time.

**Phase 3: Compounding returns (day 22+).** The agent's context is rich enough that it produces output you'd actually ship with minimal changes. New tasks benefit from old lessons. The system gets better at getting better.

Phase 1 to Phase 2 is about volume. Correct everything. Be specific. "This is wrong because our audience doesn't care about model benchmarks, they care about what they can build with it" is feedback that compounds. "This is wrong" is not.

Phase 2 to Phase 3 is about pattern recognition. Kelly doesn't just follow rules anymore. She drafts in my voice naturally because thirty days of corrections built a model of what I sound like.

**Phase 1 is a tax. Phase 3 is the return. Most people pay the tax and quit before the return kicks in.**

## The mistakes I made so you don't have to

**I set up six agents in two weeks.** Too fast. I was debugging coordination between agents that weren't individually good yet. Like hiring six employees on the same day and wondering why onboarding is chaos. Start with one. Get it to Phase 2. Then add the second.

**I got the file structure wrong.** For the first two weeks, everything went into one file. Preferences mixed with rules mixed with lessons. The agent loaded contradictory context because a preference from week one conflicted with a rule from week two. Separate memory from skills early. And when context hits 150K+ tokens, compact ruthlessly.

**I gave vague feedback.** "Make this better" doesn't compound. "The hook needs to be shorter because my audience decides in 3 seconds whether to keep reading" compounds forever. Every piece of feedback should be specific enough that you could write it as a rule.

## Your first 30 days

**Week 1:** Pick your most repetitive daily task. Set up one agent. Write a SOUL.md. Create a single cron schedule. Let it run. It will produce mediocre output. That's expected. Your only job: correct everything with specific feedback. Not "this is bad." Specific. "This is bad because X. Next time, do Y instead."

**Week 2:** Check if lessons are landing. Run the same task twice. Did the agent make the same mistake? If yes, the feedback loop isn't closing. Make sure corrections reach a persistent file. Start a skill file. Codify the rules you keep repeating.

**Week 3:** Your agent should be in Phase 2 now. Drafts need edits, not rewrites. Start tracking how long each review takes. This number should be dropping week over week. If it's not, your feedback isn't specific enough.

**Week 4:** Consider the second agent. Only if the first one is reliably producing useful output. Set up file-based coordination: first agent writes to a shared file, second agent reads it. Keep the integration dead simple.

## This is what really matters

The model is the same on day 1 and day 30. Claude doesn't get smarter because you've been using it longer.

But the system around it does.

200 lines of context. 30 days of corrections. A memory file that knows your voice, your audience, your standards. A skill file that encodes every failure into a rule the agent follows automatically.

That's not something you can download. It's not something a competitor can copy by using the same model. It's earned through reps.

The people who push through the ugly first week are building systems that learn. Everyone else is restarting from zero every session.

One agent. One job. Thirty days. The compounding will surprise you.

I'll be publishing more about OpenClaw, autonomous AI agent teams, and the evolving landscape for AI PMs and developers.

Follow me [@Saboo\_Shubham\_](https://x.com/@Saboo_Shubham_) to stay tuned.