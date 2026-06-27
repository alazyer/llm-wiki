---
title: "The Self-Verifying Loop: 300 agents, 4,000 steps, 5 live data feeds on autopilot with Kimi K2.6"
source: "https://x.com/0xRicker/status/2067584599509651652"
author:
  - "[[@0xRicker]]"
published: 2026-06-18
created: 2026-06-21
description: "Most agent swarms hand you confident garbage. This one checks its own work, throws out what fails, and runs again until every number traces ..."
tags:
  - "clippings"
---
![[bdf6c26ba36d550a9853cb34849fd475_MD5.jpg]]

Most agent swarms hand you confident garbage. This one checks its own work, throws out what fails, and runs again until every number traces to a source.

- **300** parallel agents
- **4,000** steps per run
- **5** live data feeds
- **3** verify passes to zero errors

The dirty secret of agent swarms is that more agents usually means more confident nonsense.

Point 300 agents at a research job and they will absolutely come back fast. They will also come back with stale numbers, half-invented citations, and three companies that don't exist. Speed was never the hard part. Trust was.

So I stopped treating the swarm as the finish line and made it one stage of a loop. **Opus 4.8** plans the work and, more importantly, checks it. The **Kimi K2.6** swarm executes. Then Opus verifies every output against its source, throws out whatever fails, and sends those tasks back to run again. The loop only stops when nothing fails.

To test it I gave the loop a job that punishes hallucination harder than anything: **analyze 100 companies in the EV market and produce a research-grade report with a comparison matrix, every figure traced to a live source.**

> **A swarm gives you speed. A loop gives you speed you can actually trust. The difference is the verify step, and it changes everything.**

<video preload="none" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/amplify_video_thumb/2067524890819702784/img/NuxwSQu344NYNnnX.jpg" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"><source type="video/mp4" src="blob:https://x.com/d7882f04-6c36-47e1-b09f-3fdd5dd7f884"></video>

0:04 / 0:06

The missing piece

## Why raw swarms can't be trusted

A swarm with no verifier has exactly one quality setting: whatever the worst agent produced. If 97 agents nail their company and 3 quietly hallucinate a revenue figure, your finished report contains three landmines and looks identical to a perfect one. You won't know which three until it blows up in a meeting.

This is why "just add more agents" plateaus. Volume scales the output and the error count at the same rate. More hands, more mistakes, same lack of anyone checking.

The loop fixes this by making verification a first-class stage with real teeth. **Opus 4.8** reads every agent's output back against the live source it claimed to use. A number that doesn't match gets rejected. A citation that doesn't resolve gets rejected. Anything rejected goes back into the queue and runs again. Nothing ships until it survives the check.

![[88234d3533a664f37f343c3afd5ceb52_MD5.png]]

The loop

## Four stages, running until clean

The whole system is a cycle, not a line. Each half does only what it is best at, and the cycle keeps turning until the verify stage has nothing left to reject.

![[60d075040f0c11d2568d2a187be5295a_MD5.jpg]]

> That fourth stage is the entire idea. A normal swarm runs steps 1 through 3 once and hands you the result, errors and all. The loop refuses to stop while anything is still wrong.

The run

## Watching the loop catch its own mistakes

Here is the prompt I gave Opus 4.8. Notice the checklist at the bottom. That checklist is what the verify stage uses to reject bad work later, so it is the most important part of the whole prompt.

```python
# Role: plan the work, then verify every result.

GOAL: research 100 EV-market companies.
OUTPUT: comparison matrix + research report, every
        figure traced to a live source.

PER-COMPANY CHECKLIST (verify against this):
- revenue + margin pulled from a live feed
- source URL attached and resolvable
- figure matches source within tolerance
- no field left empty

# after the swarm runs, check EVERY company.
# reject any that fail. send them back. repeat.
```

Opus planned 100 research tasks, one per company, and handed them to the **Kimi K2.6** swarm. The first pass came back in minutes. Then the interesting part started.

<video preload="none" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/amplify_video_thumb/2067526170455719937/img/K3Ya4IQ4qZTMB2nF.jpg" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"><source type="video/mp4" src="blob:https://x.com/f1b8b30d-3e03-4158-919f-3767012a1e6f"></video>

0:02 / 0:07

On the first verify pass, Opus rejected 12 of the 100 companies. Some had a revenue figure that didn't match the feed it cited. Two cited a source that didn't resolve. One left a margin field empty. None of these would have been obvious in the final report. All of them would have been wrong.

Those 12 went back into the queue with their rejection reason attached. Second pass: 3 still failed. Third pass: zero. The loop stopped on its own, because there was nothing left to reject.

![[b6072832629ca18770642eb2f1049437_MD5.jpg]]

> A raw swarm would have shipped those 12 errors and called it done. The loop caught all of them without me reading a single row.

<video preload="none" tabindex="-1" playsinline="" aria-label="Embedded video" poster="https://pbs.twimg.com/amplify_video_thumb/2067526553894797313/img/FcEksEwF57u_Ojz8.jpg" style="width: 100%; height: 100%; position: absolute; background-color: black; top: 0%; left: 0%; transform: rotate(0deg) scale(1.005);"><source type="video/mp4" src="blob:https://x.com/5f1b6c01-be26-4424-a855-1330a8932c53"></video>

0:03 / 0:07

The five live feeds are why verification can be strict instead of vague. Every figure in the report points to **Binance, Yahoo Finance, the World Bank, the IMF, or the live stock market**. When Opus verifies, it isn't asking the model if it feels confident. It is checking the claimed number against the actual feed. That is the difference between research-grade and confident-sounding.

![[e2751c64d5f68d9bff4974405291dd97_MD5.jpg]]

The bigger picture

## This is another DeepSeek moment

Step back from the run, because the strategic picture is the real story.

While the closed labs ship single-agent chatbots, an open Chinese lab valued at **$20B** shipped the swarm that makes a loop like this possible at all. Their open-weight model, **Kimi K2.6**, currently sits at #1 on the OpenRouter weekly leaderboard. By usage, it is the most-used LLM in the world right now.

And it is strongest exactly where verification matters most:

- **Finance and consulting.** Professional charts, heatmaps, multi-year report analysis, McKinsey-grade output by default.
- **Academic and research.** LaTeX formula rendering, literature reviews with comparison matrices, citations that trace to source.
- **Scale that breaks other tools.** 200,000+ words of context in a single pass, 100-company datasets, 100-slide decks.
- **Traceability.** Every data point links to a clickable source. Research-grade is the default, not a setting.

![[b16a3551579b333e02166161849fab49_MD5.png]]

Run it yourself

## The loop, start to finish

You do not need a lab. You need the two halves wired into a cycle, and a checklist strict enough to verify against.

![[1b67aea6468a10d7196272e4bc78509b_MD5.jpg]]

```python
{
  "pass": 1,
  "checked": 100,
  "passed": 88,
  "rejected": [
    { "company":"co_041", "reason":"revenue != source" },
    { "company":"co_067", "reason":"citation 404" },
    { "company":"co_092", "reason":"margin empty" }
  ],
  "action": "requeue rejected -> swarm"
}
```

## The difference in one frame

**Raw swarm**

❌ Runs once, hands you the result

❌ Hidden errors ship with the report

❌ Quality equals the worst agent

❌ You audit every row by hand

❌ Confident, unverifiable numbers

**Self-verifying loop**

✔️ Runs until the verify pass is clean

✔️ Failures caught and rerun automatically

✔️ Quality equals the checklist

✔️ You audit nothing, the loop did it

✔️ Every figure traced to a live source

> **A swarm gives you speed. A loop gives you speed you can trust.** The single-agent era is closing, but the swarm era has a catch nobody mentions: volume without verification is just faster mistakes. The people who win the next one aren't running the most agents. They're running the ones that check their own work.