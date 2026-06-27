---
title: "Forward Deployed Engineering 101"
source: "https://x.com/vasuman/status/2057177266984226892"
author:
  - "[[@vasuman]]"
published: 2026-05-21
created: 2026-06-27
description: "By the end of this article, you should understand why Anthropic, OpenAI, Google, and other AI companies are looking for FDEs, and how you ca..."
tags:
  - "clippings"
---
![[dbdb31113a1049ec690f8effb8a89148_MD5.jpg]]

By the end of this article, you should understand why Anthropic, OpenAI, Google, and other AI companies are looking for FDEs, and how you can capitalize on this demand.

I've done the job myself, hired some of the best in the world to build out our FDE motion at [Varick](https://www.varickagents.com/), and noticed there's no real definitive guide to the hottest role in tech right now. This is that guide.

**Why do AI companies need FDEs**

To become an FDE, the first step is to internalize why AI companies are in desperate need of them.

If you believe intelligence is becoming commoditized, it then follows that the only competitive edge is how and where you use it. In fact, I will go so far as to say that there is no competitive advantage in intelligence alone. Therefore, determining how and where companies use it becomes the most important role and that is the role of a forward deployed engineer.

Businesses hire an Applied AI company (like Varick) that deploys FDEs to help them get the most out of the technology. Doing this gives them access to a team that has already done large-scale AI transformations that make clients move much faster than their competitors, and by doing so, yields massive efficiency gains.

The FDE is a highly skilled engineer who can understand the customer's problems very deeply, write code into a code base they've potentially never seen before, and communicate the business impact to a non-technical decision maker to close the deal. This is a million-dollar hire.

**What the role requires**

Being an FDE requires you to be on-site with a customer. Palantir's CTO says that you cannot build products for an environment without actually being in the environment itself. We've seen the same thing internally.

The term FDE actually originated from Palantir, and they took being on-site very seriously. In 2010, they worked with the Special Forces group in Afghanistan. The Special Forces would go on the mission in the day, get feedback, and route it to the FDEs who would ship code during the night.

Being in the environment is as necessary for deploying military software as it is for deploying AI. In order to see real efficiency gains, a company needs to be rebuilt around AI from the ground up. And that is only possible through sitting with the customer and building custom agents that are engineered on company-specific data, with company-specific context.

**About the role**

In our view, there are three main parts of an Applied AI FDE's job: Audit, Evals, and Deployment. Let's break down each one.

**Audit:** You're onsite with a client, mapping processes/workflows in different teams within the company. For example: two weeks with rev ops, one week with procurement, and a full month with finance.

With each team you sit with, you learn a few things: what their job looks like, where the bottlenecks are, and where you might create agents that deliver value.

Along with understanding the workflows of each team in the company, an important part of the audit phase is determining what should be automated vs what shouldn't. There's a point where agents can create more problems than they solve.

Here are three general principles you can follow to help you decide.

If a workflow can be distilled into rules but the inputs are different (one input is an email, the next could be a PDF, the next is a scanned image), and the work involves calling tools, put an agent in. If the rules and inputs are both predictable, code is faster and cheaper. If the decision needs pattern recognition and domain expertise, leave it manual.

Your clients aren't going to get good ROI if the agent runs five times a month. Look for lengthy, high-volume automations. There needs to be enough volume to matter.

Don't overuse AI when building your agents. Most automation tasks can be done with a series of tool calls and just one call to an LLM as an orchestrating layer. Too much AI leads to unneeded token costs (which compound at scale) and often a lower-quality output.

The final part of the Audit phase is prototyping. See [Agents 101](https://x.com/vasuman/status/2011923440769659132) to learn how to build an agent, and [Agents 102](https://x.com/vasuman/status/2013742184772550851) to take that agent from demo to production.

**Evals:** If a customer is spending millions on an AI deployment, they need to know it's working. To do that, an FDE builds detailed evals.

A good eval doesn't just check if the final answer an agent gives you is correct, but also verifies the AI is thinking like a human would. In order to do that, do two things:

Trace the human's steps and grade the AI on each one: A human doesn't solve problems in one move. It's a multi-step process. Map out those steps and see if the AI is hitting the same checkpoints along the way.

Start small with great examples of the intended outcome, then measure everything against them: If you're building a customer support agent, sit with a human and figure out what the best possible answer to a user's query is. Repeat that a few times over a few tasks. Now you know what "great" looks like and can hold the agents to that standard.

Evals prove value to the customer. While everyone says they want AI in their company, there are still many who are skeptical of whether it works. A good agent evaluation is what an executive needs to trust the agent will provide ROI.

**Deployment:** Avoid large-scale data migrations. Instead, build APIs over an existing data layer (SharePoint or databases) and place a model on top as an orchestrator to query through it. This saves time and money, and more importantly, saves you from the brutal nightmare of ripping out your existing systems. Our clients have spent millions of dollars and multiple years migrating to their latest ERP. The last thing they want to do is replace it again.

Once all of the above is completed, create an execution environment to test the agent safely. This is a sandbox directly in the company's infra so you can run, test, and debug the agent before hitting production.

When moving to production, start slow. Take a small workflow, get it to work, and then layer on additional capabilities. For example, start with an agent that catches bugs, investigates, and writes a ticket summarizing what it thinks went wrong. If that works, only then give it the capability of writing code and pushing a PR.

Start with the smallest unit of autonomy; only then give it the capability to take action.

That is how you go from an audit to deployment as an FDE. Learning these steps is the entire job in itself.

**How to become an FDE in 30 days**

There are typically three backgrounds that find the most success as an FDE: Consultants, Product Managers, and Software Engineers. Even if you don't fall into any of these, following the 30-day roadmap at the end of this section will exponentially increase your chances of getting the role. Do these in parallel with applying and interviewing.

**Consultants/PMs**

As a consultant or PM, you should already have the ability to translate data into ROI. That is half the job. But the biggest roadblock for someone with this background is a lack of engineering experience.

A high-quality portfolio can mitigate this. Pick two of these side projects and go all in:

- A production-ready AI agent that can execute an entire process that you used to do manually in your old job. It should be able to call APIs, log its thinking autonomously, and have a failure harness.
- A RAG pipeline built on top of a dataset (choose a custom dataset for whatever industry you're trying to break into: legal docs, medical records, financial filings, etc.).
- An eval framework you built yourself that scores an agent's outputs across multiple dimensions (correctness, format, cost, latency) for different business processes (procurement, accounts payable, etc.).
- An MCP where you can connect an LLM to legacy software that doesn't currently support AI integration.

Do not outsource your understanding to AI. If you take it step by step, these concepts should be quite understandable. There's a reason why this is titled 30 days, not 30 minutes.

**Software Engineers**

Arguably the most important part of being an FDE is communication. You need to translate what AI can and can't do into something that makes sense to a non-technical VP. If you can't do that, you can't be an FDE.

SWEs should build projects similar to those mentioned in the consulting/PM section, but explain every single component of what you just built. The tech stack, results, iterations you made, business outcomes. Most importantly, you need to have a reason for building those agents in the first place: what was the pain point you're solving for, and how might this go down in a real client interaction?

**30-day outline regardless of role**

For something more concrete, follow this 30-day plan that will prepare you for almost anything:

Checkpoint 1 (7 days in):

- What is an agent and how an agent loop works: read Anthropic's [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents), then write a script that runs the loop: prompt → model → response → next step.
- How to make an agent call a tool: add two tool calls (an API call and a web search) using the [Anthropic](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)/[OpenAI](https://developers.openai.com/api/docs/guides/tools) tool use tutorials.
- How to build proper guardrails: add input validation, a max-step limit, and output filtering before anything reaches the user.
- When to use context window vs external memory: default to context unless state needs to persist longer than the run.
- What is an audit trail and how to build one: log every prompt, tool call, and response with timestamps. This helps find and flag agent errors.

Checkpoint 2 (14 days in):

- How to enforce structured outputs: always return JSON. Read through OpenAI's [developer page](https://developers.openai.com/api/docs/guides/structured-outputs).
- How to take a demo to prod and what usually breaks: Read Agents 102.
- How to checkpoint: save agent state every n steps to a file so it can restart from the last checkpoint.

Checkpoint 3 (21 days in):

- How retry logic and exponential backoff work: every external call needs retries. On failure, wait 1s, 2s, 4s, 8s, cap at 16s.
- How to optimize cost when deploying agents: three things: cheaper models for cheap subtasks (Opus should be used only for reasoning), cache common prompts, cap max tokens. Track cost per query.
- How to build a golden dataset for evals: start with 20 real queries, label the perfect output yourself. Anthropic's "[Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)" covers everything.
- How multi-agent pipelines and parallel architectures work: split the job when one agent can't handle it. One plans, others execute, one synthesizes.

Checkpoint 4 (Final week):

Review the above and communicate everything out loud. Tie everything you can to business metrics.

**TLDR**

The FDE is the most in-demand role in tech right now. Every company needs AI, but nobody knows how to deploy it.

The job has three phases (audit, evals, deployment). Your job is to understand each phase and its purpose.

Your portfolio and your ability to speak about it are the deciding factors. Build agents, RAG pipelines, eval frameworks, MCPs, etc, and, most importantly, be able to confidently articulate the business use case behind everything you're building.

Lack of communication ability is a deal-breaker for the FDE role. If you can't explain what AI can and can't do to a non-technical decision maker, there won't be a deployment.

Know when AI isn't the answer; this builds trust with the client and more importantly, ROI for the agents in production.

Do these steps and your chances of breaking in will be exponentially higher.

If you want to make the jump to the fastest-growing Applied AI company in SF, Varick is hiring. We're building out the most elite FDE team in Silicon Valley, led by a former COO of Citadel Securities. Apply directly at [https://www.varickagents.com/careers](https://www.varickagents.com/careers).