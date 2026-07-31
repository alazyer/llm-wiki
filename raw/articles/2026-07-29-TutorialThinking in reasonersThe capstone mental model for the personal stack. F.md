---
title: "TutorialThinking in reasonersThe capstone mental model for the personal stack. Five habits that turn a pile of prompts into a system, each shown as a small before and after, with the one table you need for deciding between app.ai and app.harness.Jul 2, 2026 · 24 min"
source_url: "https://agentfield.ai/blog/thinking-in-reasoners"
ingested: 2026-07-29
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/thinking-in-reasoners

## Article Content

Thinking in reasoners — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Thinking in reasoners

The capstone mental model for the personal stack. Five habits that turn a pile of prompts into a system, each shown as a small before and after, with the one table you need for deciding between app.ai and app.harness.

Santosh Kumar Radha·Co-founder & CTO·

24 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

This is the last post in the personal stack, the running system you built from your personal control plane forward. Everything you shipped so far, the assistant, the nightly fleet, the approval gate, the market simulation, follows the same five habits. Thinking in reasoners means decomposing a task into small reasoners with one job each, giving each one a goal instead of a script, deciding deliberately where a cheap classification suffices and where a tool-using agent is required, capping every loop with a number, and passing prose between models but structured data to code. Learn these five and you stop writing prompts and start designing systems.

Five principles. Twelve minutes to read. No new code to run: this post is the frame that makes the code you already wrote make sense.

1. One cognitive job per reasoner

A reasoner should do one thing and return two to four fields. When you find yourself writing a single prompt that classifies, then analyzes, then summarizes, then formats, you have written four reasoners crammed into one, and the model does all four worse than four focused calls would.

Before, one prompt carrying the whole job:

result = await app.ai(

system=(

"Read this repo. Find the failing tests, figure out why they fail, "

"propose a fix, estimate the risk, and write a PR description. "

"Return everything."

user=repo_dump,

After, the job split into reasoners that each hold one idea:

diagnosis = await app.call(f"{app.node_id}.diagnose", failures=failures) # {cause, confidence}

fix = await app.call(f"{app.node_id}.propose_fix", cause=diagnosis["cause"]) # {patch, risk}

if fix["risk"] == "low":

await app.call(f"{app.node_id}.open_pr", patch=fix["patch"])

Now each reasoner is small enough to name, test, and reuse. diagnose can be called from three places. propose_fix can be swapped without touching diagnosis. The split is what makes the system inspectable, and it is why the runtime-topology pipeline can decide which reasoners to run after it sees the input.

2. Set the goal, verify the outcome, skip the steps

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

A reasoner has freedom in how it answers and none in what it answers. You are a CEO handing off a task, not a manager dictating keystrokes. Tell it the outcome you need and how you will check it, then let it navigate.

Before, micromanaging every step:

step1 = await app.ai(user="List the files in the repo.")

step2 = await app.ai(user=f"For each file in {step1}, read it and note the imports.")

step3 = await app.ai(user=f"Given {step2}, find circular dependencies.")

# ...you are now the orchestrator, the model is a calculator

After, one guided-autonomy call with a verifiable outcome:

result = await app.harness(

prompt="Find the circular imports in this repo. Return each cycle as a list of files.",

schema=CycleReport, # {cycles: list[list[str]], confident: bool}

max_turns=8,

# You verify the shape and the confident flag; the harness chose how to explore.

The harness reads, greps, and traces on its own. You verify the result, not the route. The nightly repo fleet runs on exactly this: each harness gets a goal and a scratch clone, and you check the diff it produced rather than the commands it ran. Verification is the contract; the steps are the agent's business.

3. .ai() for a fast decision, .harness() for real work

The single most consequential choice you make per reasoner is which primitive it uses. app.ai is a single-shot classification: input in, a flat schema out, no tools, no navigation. app.harness is a stateful, tool-using agent that reads files, runs shell commands, and adapts as it goes. Pick the wrong one and you either pay 20x for a job a classifier could do, or you hand a document to a primitive that cannot read past the first page.

Use app.ai() whenUse app.harness() when

Classifying, routing, or gatingReading or navigating a document or repo

Input fits in one context window (under ~2k tokens)Input is large and must be explored, not read in one pass

Output is 2 to 4 flat fieldsOutput is narrative, a patch, or many fields

One shot, no tools neededMulti-turn: read X, then decide to read Y

A cheap model is enoughThe agent needs a shell, files, or sub-agents

The rule when you are unsure: reach for app.ai first, and give it a confident: bool field. When it comes back confident=False, escalate that one case to app.harness. That is the fallback pattern the confidence cascade is built on, and it is why a cheap classifier can front an expensive agent without ever crashing the pipeline. A wrong classification that propagates costs far more than the harness call you avoided.

4. Every loop gets a number

An adaptive system finds a reason to go one level deeper on every edge case. Left alone, it will. The difference between a system that learns and a system that empties your account is a hard integer cap on every loop and every spawn.

Before, a loop that ends when the model feels done:

while not result.confident: # no ceiling, no bill limit

result = await app.ai(user=f"Refine this further: {result}")

After, a loop that ends when the counter runs out:

for attempt in range(MAX_REFINEMENTS): # 3, and it means 3

result = await app.ai(user=f"Refine this: {result}", schema=Refined)

if result.confident:

break

The same discipline applies to spawns and to money. A parent that can dispatch sub-agents caps the count (budget=1, and the child is dispatched with budget=0 so it cannot recurse, exactly as the self-building pipeline does it). A harness carries a token cap: max_budget_usd is a hard ceiling for the claude-code provider, and for the others you cap with max_turns and read result.cost_usd after. Be precise about what each bounds: max_turns bounds turns, not dollars, and only claude-code enforces max_budget_usd as a hard cost ceiling, so on any other provider you read result.cost_usd after the run rather than trusting a dollar cap to have stopped it. "Keep going until it is good" is not a stopping condition. A number is.

5. Prose between models, structured data to code

Data changes shape depending on who reads it next. If the next reader is Python that branches on a value, pass structured JSON. If the next reader is another model that reasons over context, pass a string. Getting this backwards is the most common way a working pipeline turns brittle.

Before, JSON shoved into a model that only reads it as text:

findings = await app.call(f"{app.node_id}.analyze", doc=doc) # returns a dict

summary = await app.ai(

system="Summarize these findings.",

user=json.dumps(findings), # the model now parses braces instead of reading

After, prose to the model and structure only where code branches:

findings = await app.call(f"{app.node_id}.analyze", doc=doc) # {items: [...], severity: "high"}

if findings["severity"] == "high": # code branches on structure

summary = await app.ai(

system="Summarize these findings for an engineer.",

user="\n".join(f["description"] for f in findings["items"]), # prose to the model

The severity field is structured because an if reads it. The descriptions go to the model as plain sentences because a model reasons over language, not serialized dicts. The format follows the consumer. When you see if "critical" in output.text, that is a structured field trying to be born; give it an enum. When you see json.dumps(...) feeding an app.ai call, that is prose trying to escape a dict; render it to sentences.

Why this is a skill worth having

The five habits are not AgentField trivia. Decompose the work, set goals instead of steps, match the tool to the job, bound every loop, and route data by its consumer, and you have described how to build any reliable system out of unreliable parts. The reason it matters now is that the parts are LLM calls, and a single LLM call reasons at maybe a third of what the task needs. The intelligence is in the composition. Orchestrating many small, verified, bounded reasoners into a system that reasons at the level you need is becoming a core systems skill, the way concurrency and distributed state became core skills a generation ago. The developers who can do it are the ones who stopped chasing a better prompt and started designing a better graph.

You have already built five systems that do this. This post named the moves.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents. Paste this to have your coding agent review one of your own agent designs against the five principles:

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Next step: run that review against the market simulation or the nightly fleet you already built, fix the first thing it flags, and re-run it until every principle reads PASS.

Related

Your personal control plane, the foundation the whole personal stack registers into.

Pipelines that build themselves, principles 1, 3, and 5 shown in one runtime-topology build.

A fleet of coding agents with budget caps, principle 4 as a hard-capped harness fleet.

One control plane for XGBoost and agents, the confidence cascade that principle 3's fallback pattern comes from.

Human approval in 20 lines, where guided autonomy meets a human verifier before an irreversible step.

What is harness orchestration, the essay on why the harness, not the LLM call, is the unit you compose.
