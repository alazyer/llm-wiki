---
title: "Loop Engineering for Product Managers"
source: "https://x.com/Saboo_Shubham_/status/2068730090457006588"
author:
  - "[[@Saboo_Shubham_]]"
published: 2026-01-07
created: 2026-06-27
description: "The next PM skill is not prompt engineering. It is Loop Engineering.For the last two years, most PMs have been trying to get better at promp..."
tags:
  - "clippings"
---
![[8d2a1554801b78185790699401557c58_MD5.jpg]]

The next PM skill is not prompt engineering. It is Loop Engineering.

For the last two years, most PMs have been trying to get better at prompting. Better instructions, better context, better examples, better ways to tell Claude, Codex, Cursor, or whatever agent they use what to do.

The end state is not a PM writing a perfect prompt every time they need something. It is a PM designing a system that improves every time it runs.

A loop is the cycle your AI system runs through repeatedly. You change something that shapes the agent's behavior. You run the agent. You evaluate the output. You keep the change if quality improves and revert it if quality drops. Then you commit the learning so the next version starts ahead of the last one.

For engineers, that cycle starts with code.

For PMs, it starts somewhere else. It starts with the artifacts that shape product work: a PRD-review skill, a customer-call summarizer, an eval rubric, a launch checklist, a research workflow, a CLAUDE.md instruction, a prompt template, or a prioritization framework.

These are durable. They get reused. They encode your judgment. They shape how an agent behaves across dozens of runs, not one.

And because they get reused, they compound in both directions. A good one makes your work sharper every week. A bad one makes it slower every week, quietly, until you notice the output feels off and can't say why.

That is why they need loops. A one-off prompt you can afford to get wrong. A rubric ten people depend on, you cannot.

![[92b69f111ed84d2dde4ac6da3e21073c_MD5.jpg]]

## The thing prompting does not solve

The first shift was obvious. Agents made it easier to build.

You describe a dashboard and get a prototype. You write a rough idea and get working code. You hand over ten customer transcripts and get a structured synthesis in minutes.

That changed the PM workflow. You no longer only write a spec, hand it to engineering, wait, review, and repeat. You can now shape the first version directly.

But once you work this way for a while, you run into a different problem.

Your AI workspace starts to drift.

Your CLAUDE.md gets longer. Your PRD-review skill gets stricter. Your research workflow picks up instructions from a past project. Your launch checklist expands until the agent ignores half of it. Your eval criteria change, but you forget when or why.

A month later, the agent feels worse. It writes worse PRDs. It gives generic research summaries. It misses the product nuance it used to catch.

The model probably did not get worse. Your artifacts drifted, and nothing was watching them.

That is the real problem loop engineering solves for PMs. Not making one prompt better. Making sure the artifacts shaping your product work are improving instead of slowly decaying.

## What a loop is actually made of

A useful loop has five parts: **a trigger, an action, proof, memory, and a stop condition.**

The trigger says when the loop starts. The action says what the agent should do. The proof says how you know the output is better. The memory says where the learning gets saved. The stop condition says when the loop should end.

That last part matters most with recursive loops.

A lot of AI systems do not fail because the model is bad. They fail because the loop has no clean exit. The agent keeps generating. It keeps expanding scope. It keeps inventing work. Or it gives you a confident summary without enough evidence.

A good PM loop should be allowed to say: nothing meaningful changed, the input is too thin, this is blocked, the quality bar was not met, or this needs a human decision.

A loop that can say "stop" is a loop you can leave running. One that can't is one you have to babysit.

If you want to see loops like this in the wild, Matthew Berman's [Loop Library](https://signals.forwardfuture.ai/loop-library/) collects real ones across engineering, research, and operations. They are built for engineers, but the five parts are the same for PM work.

![[e23a555d29e084051f64401d2864f39e_MD5.jpg]]

## What this looks like in practice

Take customer research.

The simple version is asking an agent, "Summarize these calls." That is useful once, but it does not improve the next time you summarize calls.

The better version is a customer-call summarizer that knows what your team cares about: pain points, current workaround, urgency, objections, product gaps, exact quotes, and follow-ups.

The loop version runs every week. It uses the summarizer, compares new calls against previous themes, flags what is repeated, what is new, and where evidence is thin. You review the output. If the summary missed nuance, you improve the artifact. The next run gets better.

Same with PRD review.

The simple version is asking, "Can you review this PRD?" The better version is a PRD-review artifact that knows your team's quality bar. The loop version tests whether that artifact is actually giving better feedback.

Does it catch vague problems? Does it push for real evidence? Does it call out missing metrics? Does it avoid useless nitpicks? Does it preserve the product intent?

When the answer slips, you fix the artifact, not the prompt.

Prompting helps you do the task once. Loop engineering improves the artifact that shapes how the task gets done every time.

## Your first loop: a weekly product signal

![[519c2797bd0eae32615bd4c954d00b7b_MD5.jpg]]

I would not start with a loop that tries to decide product strategy.

That is too broad, too subjective, and too easy to get wrong. I would start with product ops. That is where the repeated work is. That is where the evidence is. That is where consistency matters.

A good first loop is a weekly product signal loop.

Every Friday, the loop reads customer calls, support tickets, sales notes, experiment updates, analytics summaries, shipped changes, and open escalations. It produces one product signal memo.

The memo should not be a generic summary. It should separate repeated signal from isolated noise.

It should include exact customer language. It should show what changed since last week. It should call out where evidence is thin. It should identify which roadmap assumptions got stronger, weaker, or stayed the same.

You review it. If the memo missed something important, you update the artifact. If the agent over-weighted weak evidence, you tighten the rubric. If a new product question emerged, you add it to next week's lens.

After a few weeks, this becomes useful.

After a few months, it becomes the thing you check before any roadmap call.

## Taste still matters, it just needs proof now

PMs have always relied on taste.

A good PM can read a PRD and feel when the problem is vague. They can tell when a customer quote is being stretched too far. They can sense when a launch plan is pretending a dependency is not real.

That judgment still matters. Loop engineering does not remove it. If anything, it puts judgment at the center.

> Jun 20
> 
> Generation is solved with AI Agents. Loop Engineering can produce infinitely. Verification and judgment are all that's left. Boris, who created Claude Code recently said that he doesn't prompt anymore. He writes loops, and the loops do the prompting for him. In Jan 2026, I

The post's last line is the whole job now: define what correct means, and tell when it has been met. The question is how you do that without guessing.

But when you put your judgment into reusable artifacts, taste needs proof.

If you change a PRD-review artifact, how do you know it got better? If you update a research summarizer, how do you know it catches more signal and not more noise? If you tighten a launch checklist, how do you know it improves readiness instead of adding ceremony?

This is where evals become PM work.

A PM eval does not need to be a giant benchmark. It can start with known examples.

Take three strong PRDs and three weak PRDs. Run your PRD-review artifact against them. Did it catch the real gaps? Did it avoid nitpicking? Did it preserve the product intent?

Take five customer calls you already understand. Run your research summarizer. Did it capture the actual pain? Did it quote accurately? Did it distinguish strong signal from one-off noise?

Take two past launches: one clean, one messy. Run your launch-readiness loop. Would it have caught the thing that actually broke?

That is loop engineering. You are not asking, "Did the agent sound smart?" You are asking, "Did this artifact improve against known product judgment?"

## The loop needs a memory layer

Once you start evaluating artifacts, you need somewhere for that learning to live. Otherwise the work disappears.

You change a prompt in a chat. You paste a new instruction into a file. You tweak a rubric. You add an example. You remove a constraint. The output changes, but the history is gone.

A few weeks later, nobody knows why the agent behaves that way.

This is where GitHub becomes useful. Not because PMs need to become engineers, but because PMs need version history for the artifacts that increasingly shape their work.

The artifact lives there. The changes live there. The eval results live there. The decision log lives there. The rollback path lives there.

If an artifact improves, you want the commit. If it gets worse, you want the diff. If the rubric changed, you want to know why. If a launch checklist caught a real blocker, you want that learning saved for the next launch.

The repo becomes product memory.

![[0f9cf3c3a25bf4be8faa84186ba7ac9e_MD5.jpg]]

## The skeleton for any loop

You do not need to make this complicated.

Pick one repeated product task. Write down when it starts, what inputs it uses, what artifact guides it, what the agent should do, what good output looks like, where the learning gets saved, and when the loop should stop.

That is enough to start.

The key is to keep the loop small enough that you can actually tell whether it is improving. And every serious PM loop should say what the human still owns.

A loop can summarize customer evidence. It should not decide strategy alone. A loop can review a PRD. It should not become the product leader. A loop can flag a risky launch. It should not make the tradeoff without context.

**Build the loop, but stay the PM.**

## Where this breaks

Loops usually fail in simple ways.

The trigger is vague. The inputs are messy. The artifact is too long. The proof is subjective. The learning has nowhere to live. The stop condition is weak.

Or you give the loop too much authority too early.

That last one is the trap. Do not start with a loop that can change strategy, message customers, update the roadmap, or make product commitments without review.

Start with loops that inform a decision you still make. Let them earn trust. Then increase autonomy slowly.

That is probably the healthiest way to work with agents in product.

## What the job becomes

The PM job used to be translation. Customer pain into requirements. Business goals into roadmap. Engineering constraints into tradeoffs.

That job still exists. But there is a new layer on top of it.

You now design the systems that make product judgment repeat. You build the artifacts, version them, evaluate them, and notice when they drift. The agent runs the task. You own whether the thing running it keeps getting better.

The best PMs will not be the ones with the longest prompt library. They will be the ones who know which parts of product work should become durable loops, which artifacts should guide them, and which decisions should stay human.

That is the future of PM work.

Not better prompting. Reusable artifacts, versioned and evaluated, running inside loops that get sharper every time they run.