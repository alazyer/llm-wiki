---
title: "Build a Taste Index with Hermes and GBrain"
source: "https://x.com/Saboo_Shubham_/status/2074526361159626959"
author:
  - "[[@Saboo_Shubham_]]"
published: 2026-07-08
created: 2026-07-09
description: "Your AI agent doesn't need more memory. It needs taste.I used to think the goal was to give my AI agents more memory.That was the obvious an..."
tags:
  - "clippings"
---
![[cb23ee126e290feb658fd7c22e68dac1_MD5.jpg]]

Your AI agent doesn't need more memory. It needs taste.

I used to think the goal was to give my AI agents more memory.

That was the obvious answer. If the agent forgot context, give it a place to store things. If it kept asking the same questions, make the memory bigger.

So I gave my setup a second brain.

Hermes Agent became the operating layer: the thing that coordinates tools, files, agents, memory, cron jobs, and workflows.

GBrain became the knowledge layer: the place where durable ideas, references, theses, and examples live as connected markdown pages.

That part worked.

But it also revealed the real problem. **The hard part wasn't storing more. The hard part was deciding what deserved to be stored.**

A memory system can remember what happened. That doesn't mean it knows what mattered. It can save a link. That doesn't mean it knows why I cared. It can retrieve an old note. That doesn't mean it knows whether that note should shape the next article, product idea, or agent workflow.

Memory tells an agent what happened. Taste tells it what matters.

That's what I now call a Taste Index. Not a bigger memory. A smaller, sharper one.

## The map is not the taste

A lot of agent memory systems confuse the map for the territory.

The map is easy to build: notes, links, transcripts, embeddings, search.

All of that is useful. But none of it is taste.

**Taste is the judgment behind the thing you saved.**

Why did this article work for you? Why did that demo feel real instead of generic?

Those are judgment questions, not storage questions.

And if you don't capture the judgment, your agent only gets the artifact. It can find the article again, but not the reason it mattered. It can summarize the demo, but not the part you wanted repeated. It can search your notes, but not know which ones represent your actual taste.

That's why you need a Taste Index.

![[a00c4e6b12aead9a76a69e5e10da73fe_MD5.jpg]]

## What a Taste Index is

A Taste Index is a structured record of your judgment.

It's not a place to save everything. It's a place to save the things you explicitly like, reject, find useful, want to revisit, or want future agents to learn from.

That word explicitly matters.

The agent shouldn't quietly decide that every useful-looking link belongs in permanent memory. It waits for a signal.

In my setup, the trigger phrases are simple:

```text
gbrain this
save this
I like this
this is useful
```

Those aren't just commands. They're taste events. They tell Hermes: this crossed the bar.

The point isn't to store the link. A bookmark already does that. A Taste Index stores the link plus what you liked, why it should shape future work, and what not to copy blindly.

**The link is the receipt. The why is the asset.**

## The rule that keeps it useful

The rule is simple: **No signal, no storage.**

If I send Hermes a random link, it doesn't go into GBrain by default.

Maybe I wanted a summary. Maybe I wanted a reply angle. Maybe I wanted to check if it was real. Maybe it was useful once, but not worth carrying forever.

**Useful once is not the same thing as durable.**

This is the part that feels counterintuitive with agents. Most AI systems are built with a "more context is better" instinct. More files. More transcripts. More embeddings.

But personal taste works in the opposite direction. Taste is selective. The selectivity is the point.

If everything goes in, nothing has signal. The agent may remember more, but it will understand less.

This is especially true because agents don't get tired. A human eventually stops saving things because organizing is annoying. An agent can ingest forever.

So the system needs a gate. That gate is explicit capture.

![[ce20194f18524f8ecc83f57af7793477_MD5.jpg]]

## The capture shape

Every Taste Index capture needs to answer a few questions.

```markdown
# Short title

## Source
The link, file, quote, screenshot, conversation, or artifact.

## What I liked
The specific part that crossed the bar.

## Why it's useful
Why this should influence future work.

## Where this should influence future work
Writing, product, design, research, investing, agent behavior, or another area.

## Related ideas
Connected notes, examples, projects, posts, or prior work.

## Do not reuse
What shouldn't be copied blindly.

## Status
Taste reference, anti-pattern, thesis input, draft input, or revisit later.
```

The template isn't complicated on purpose. The point isn't to create a perfect taxonomy. **The point is to force the agent to preserve the judgment.**

The two fields that matter most: "What I liked" and "Why it's useful."

If those are missing, the capture isn't ready. That one constraint prevents a lot of future garbage.

## A real example

One of the patterns in my Taste Index is about AI app demos.

I don't want generic demos that only swap the noun. You know the pattern:

```text
Find X.
Summarize X.
Rank X.
Show a table of X.
```

Different topic. Same app.

That's not a taste preference you want to explain every time. It should become reusable context. So the capture looks like this:

```markdown
# Product demos should show the workflow, not just the output

## Source
Feedback from reviewing AI app ideas and demos.

## What I liked
Demos that feel like real vertical workflows: structured state,
domain-specific steps, handoffs, evidence, and a clear user outcome.

## Why it's useful
Future agents should reject generic demos that only summarize, rank,
or display a table. A good app should teach a distinct agent pattern
and feel like a product.

## Where this should influence future work
Awesome LLM Apps ideas, launch posts, demo scripts, product specs,
and agent app reviews.

## Do not reuse
Don't create five versions of the same app with different nouns.
Don't package a basic wrapper as a product.

## Status
Taste reference.
```

That's a useful memory because an agent can act on it.

The next time Hermes proposes app ideas, it can reject the generic ones before I have to. The next time a coding agent builds a demo, it knows a table view isn't enough.

**That's the difference between retrieving information and retrieving judgment.**

## How Hermes and GBrain work together

Hermes is where the work happens. GBrain is where durable knowledge lives.

The Taste Index sits inside GBrain, but Hermes is the one that uses it.

The loop looks like this:

![[6bf50bb9e4d5806c6680452a02e7eab5_MD5.jpg]]

**The Taste Index isn't useful because it exists. It's useful because the agent reads it at the right time.**

Before writing a post, check writing taste. Before proposing product ideas, check product taste. Before saving a new note, check the capture rules.

That's how the system stops being a folder and starts becoming a working layer.

## What goes where

This was the part I had to learn the hard way. Not every kind of context belongs in the same place.

Here's the split that works for me:

![[8c1c17f7fb29273170e3705da3805442_MD5.jpg]]

This keeps the system clean. If GBrain stores task status, it becomes noisy. If Hermes memory stores full essays, it becomes bloated. If session context stores durable taste, it disappears.

GBrain is the only layer that needs a judgment call. Before anything goes in, ask one question:

**Will this help a future agent make a better choice?**

A writing pattern you want reused: yes. A design reference that shows what good looks like: yes. An anti-pattern you never want repeated: yes. Each of these changes what the agent does next time.

A meeting note: no. A debugging log: no. An article you skimmed once: no. They were useful today, but they say nothing about your judgment.

Yes means capture it in GBrain. No means it stays in the other layers.

## The maintenance loop

A Taste Index can still rot.

The gate stops junk from getting in. It doesn't stop good captures from going bad. A capture from three months ago might be missing its why. Two pages might say the same thing. A taste rule you wrote early on might not survive what you've learned since.

So Hermes has a second job: the curator.

Once a week, it reads every page in the Taste Index and checks each one against the capture rules. Is the "what I liked" there? Is the why still true? Is the page linked from the index? Does it still look like taste, or is it task status that snuck in?

![[8aa9f954293967a215d78dbe5eaea9cb_MD5.jpg]]

If everything passes, I hear nothing. No report. No summary. Silence means healthy.

If something fails, I get one message:

```markdown
2 captures need attention:
- "workflow demos" is missing its why
- "agent naming note" looks like task status, not taste
Fix or delete?
```

I reply in one line. Fix this, delete that. The index stays sharp.

One constraint makes this safe: the curator never ingests. It only audits what's already there. A maintenance job that adds material is just another ingestion path around the gate.

**A memory system should reduce maintenance, not become another inbox.**

## How to build your own

Two steps. Neither of them is writing code.

The old way to set up a system like this: create folders, write templates, wire a database. The new way: brief your agent and let it build its own memory system.

**Step 1: Get GBrain running.**

GBrain is open source, built by Garry Tan to run his own agents. It runs locally, no server needed. If you run Hermes, you don't even install it yourself. One message:

```text
Install GBrain from github.com/garrytan/gbrain and connect it.
Start me a fresh brain.
```

**Step 2: Send it this article.**

Literally. Paste the link with this briefing:

```markdown
Read this article. Build me a Taste Index in GBrain based on it.

Rules:
- No signal, no storage. Only capture when I say "save this,"
  "gbrain this," "this is useful," or "this is my taste."
- Every capture needs: source, what I liked, why it's useful,
  where it should influence future work, what not to reuse.
- If my reason is unclear, ask: "Why should this go in the
  Taste Index?"
- Check the Taste Index before writing, product, and research work.
- Run a weekly curator pass. Stay quiet unless something needs
  attention.

Start the index empty. I'll send the first capture.
```

That's the whole setup. **The article you're reading is the spec. Your agent is the builder.**

No Hermes? Same two steps work with any agent that can run commands and manage files. Claude Code, Codex, OpenClaw. GBrain speaks MCP, so it plugs into all of them.

But don't overbuild this on day one.

**The hard part isn't the database. The hard part is refusing to save things.**

## Start small

A good Taste Index should feel almost annoyingly selective.

**Twenty strong captures beat two thousand weak notes.**

Because generic agent behavior isn't neutral. It has a shape. It over-explains. It hedges. It rounds off sharp opinions. It writes like the average of the internet. It stores too much because storing feels safer than choosing.

The fix isn't more memory. The fix is a record of what you mean by good.

Pick one thing you genuinely liked this week. Tell your agent exactly why.

That's capture number one.