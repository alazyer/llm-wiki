---
title: "TutorialAn agent that ships new versions of itselfA self-improvement loop where a production agent records its own failures to memory, an improver agent reads them and writes a revised prompt, and that prompt ships as a new version the self-tuning rollout then canaries behind a human gate.Jul 2, 2026 · 33 min"
source_url: "https://agentfield.ai/blog/agents-that-ship-themselves"
ingested: 2026-07-27
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/agents-that-ship-themselves

## Article Content

An agent that ships new versions of itself — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

An agent that ships new versions of itself

A self-improvement loop where a production agent records its own failures to memory, an improver agent reads them and writes a revised prompt, and that prompt ships as a new version the self-tuning rollout then canaries behind a human gate.

Santosh Kumar Radha·Co-founder & CTO·

33 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

A self-improvement loop is a system where an agent reads its own production failures, proposes a fixed version of itself, and ships that version behind a canary and a human gate. The production reasoner writes every failure case to memory. A second reasoner reads the accumulated failures, reasons about what they have in common, and produces a revised system prompt. That prompt becomes a new agent version, and the self-tuning rollout from the previous post carries it into production one traffic step at a time, pausing for a person before the new version takes the majority of requests.

By the end you have the closed loop: a classify reasoner that logs its misses, an improver reasoner that turns those misses into a better prompt, and the canary machinery that promotes the result only if it does better and only after a human says go. The improver is about 50 lines of Python. This is the loop people describe when they say agents will improve themselves, with the two things those descriptions skip: a bounded metric and a human in the path. It runs for a few cents per cycle plus production traffic cost.

The pattern

The naive version of self-improvement is an agent that edits its own prompt and immediately runs on the new one. That has no floor. A bad edit takes effect on the next request, and the thing that decides whether the edit was good is the same thing that made the edit. There is no independent check and no way back.

A self-improvement loop splits proposal from disposal. The agent proposes: it reads its failures and writes a candidate prompt. Infrastructure and one human dispose: the candidate ships as a separate version at a small weight, the self-tuning rollout measures it against the incumbent on real traffic, and a person approves before it wins. The agent's autonomy is bounded to writing a proposal. It cannot promote itself. That boundary is what makes the loop safe to run in production instead of a demo.

Build it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

The Go and TypeScript SDKs expose the same surface; the orchestration below is Python for length.

1. Record failures to memory

The production reasoner writes each failure case to memory as it happens. app.memory.set(key, data) and app.memory.get(key, default=...) store and read with hierarchical scoping: a write lands in the most specific available scope, and a read walks workflow to session to actor to global and returns the first hit. That default is enough for an accumulating failure log the improver reads back later. A failure here is any case where the classification was wrong, caught by a validator or a downstream correction.

PythonTypeScriptGo

# classify_v2_0_0.py

import os

from agentfield import Agent

app = Agent(node_id="classifier", version="2.0.0",

agentfield_server=os.getenv("AGENTFIELD_SERVER"))

SYSTEM = "Classify the ticket as billing, bug, or other. One word."

@app.reasoner(tags=["entry"])

async def classify(text: str, correct_label: str | None = None) -> dict:

label = (await app.ai(system=SYSTEM, user=text)).strip().lower()

if correct_label and label != correct_label:

# A miss. Append it to the failure log the improver reads.

failures = await app.memory.get("failures", default=[])

failures.append({"text": text, "predicted": label, "expected": correct_label})

await app.memory.set("failures", failures[-500:])

# Also record this run's miss for the rollout operator's per-version metric.

await app.memory.set(f"last_result.{app.version}", {"status": "failed"})

return {"label": label, "version": app.version}

if __name__ == "__main__":

app.run()

The failure log persists across executions under one key, so the improver reads the full accumulated set rather than one run's. The -500: slice bounds it, because unbounded failure data is a slow leak.

2. Read failures and propose a revised prompt

The improver reasoner reads the failure log and reasons about the failure modes. It is a harness, not a one-shot call, because it needs to look at a set of cases, find the pattern across them, and write a prompt that addresses it. Its output is a new system prompt and a one-line rationale.

PythonTypeScriptGo

# improver.py

import os

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="prompt-improver", agentfield_server=os.getenv("AGENTFIELD_SERVER"))

CURRENT_SYSTEM = "Classify the ticket as billing, bug, or other. One word."

class Proposal(BaseModel):

revised_system_prompt: str

rationale: str

@app.reasoner(tags=["entry"])

async def propose() -> dict:

failures = await app.memory.get("failures", default=[])

if len(failures) < 20:

return {"action": "hold", "reason": "not enough failures to learn from"}

result = await app.harness(

prompt=(

"You maintain a ticket classifier. Here is its current system prompt:\n\n"

f"{CURRENT_SYSTEM}\n\n"

"Here are recent cases it got wrong, with the label it should have "

f"produced:\n\n{failures[-100:]}\n\n"

"Find what these misses have in common. Write a revised system prompt "

"that would fix them without breaking the cases it already gets right. "

"Return the revised prompt and a one-line rationale."

schema=Proposal,

max_turns=6,

max_budget_usd=0.50,

proposal = result.parsed

await app.memory.set("pending_proposal", proposal.model_dump())

return {

"action": "proposed",

"rationale": proposal.rationale,

"cost_usd": result.cost_usd,

if __name__ == "__main__":

app.run()

The harness reads the whole failure set, not one case at a time, which is why it is a harness and not an ai call. It writes the proposal to memory and stops there. It does not deploy anything. One thing to check against the docs for your build: if you want the improver to pull failures by similarity rather than reading the last N, look for a vector search on the memory API; the accumulating-list approach above needs only get and set, which are confirmed.

3. Ship the proposal as a new version

Be plain about what deploying means. There is no hot-swap API that rewrites a running agent's prompt in place. A new version is a new process that registers under the same node_id with a new version string and the new prompt baked in. In practice, the improver's proposal is written into a fresh copy of the agent file (or an environment variable the agent reads at startup), and that process is started, exactly as you would redeploy a container with a new image.

# classify_v2_1_0.py: the challenger, carrying the improver's revised prompt

import os

from agentfield import Agent

app = Agent(node_id="classifier", version="2.1.0",

agentfield_server=os.getenv("AGENTFIELD_SERVER"))

# The revised system prompt the improver proposed, e.g. read from an env var it wrote.

SYSTEM = os.getenv("CLASSIFIER_SYSTEM_PROMPT",

"Classify the ticket as billing, bug, or other. Treat refund and "

"chargeback questions as billing. One word.")

@app.reasoner(tags=["entry"])

async def classify(text: str, correct_label: str | None = None) -> dict:

label = (await app.ai(system=SYSTEM, user=text)).strip().lower()

# ... same failure logging as 2.0.0, keyed by app.version ...

return {"label": label, "version": app.version}

if __name__ == "__main__":

app.run()

Start this process and the control plane holds classifier at two versions, 2.0.0 and 2.1.0. The version registry, the routing, and the weight split are the same machinery from the previous post.

4. Canary it with the self-tuning rollout

This is where the two posts join. The rollout operator seeds 2.1.0 at a small weight, polls each version's failure rate, and steps traffic up while the challenger wins. Seed it and start the loop:

# Start the challenger at 5% of classifier traffic.

curl -s -X PUT \

http://localhost:8080/api/v1/connector/reasoners/classifier/versions/2.1.0/weight \

-H "X-Connector-Token: $AF_CONNECTOR_TOKEN" \

-H "Content-Type: application/json" \

-d '{"weight": 5}'

The operator's floor, ceiling, and max step size bound the whole thing. A revised prompt that turns out worse gets rolled back to the floor before it touches more than a slice of traffic. See the self-tuning rollout post for the operator loop in full.

5. Gate the majority-traffic step on a human

The improver proposed the prompt. The rollout measured it. The last decision, letting the self-written version take the majority of production, belongs to a person. Put an app.pause in the operator right before the ceiling step.

if next_weight >= CEILING and challenger_rate < PROMOTE_BELOW:

decision = await app.pause(

approval_request_id=f"promote-{CHALLENGER}",

approval_request_url="https://your-console/approvals",

if not decision.approved:

return {"action": "hold", "approved": False}

await set_weight(CHALLENGER, CEILING)

return {"action": "promoted", "approved": True}

app.pause blocks the execution until the approval webhook resolves it. Until then the challenger sits at the ceiling minus one step, serving a minority of traffic while a human reviews the improver's rationale and the measured failure rates side by side. The agent wrote the fix and proved it on real traffic. It still does not get to promote itself.

What the control plane does underneath

Memory. The failure log lives under one key with hierarchical scoping, so it survives across executions and the improver reads the full set, while each version's own result is keyed separately for the rollout metric.

Version registry. The self-written prompt ships as a distinct version under the same agent, sitting next to the incumbent, not overwriting it.

Weighted routing plus rollback. The router splits traffic by weight, and an unhealthy or rolled-back version drops to zero without a redeploy.

Approval gating. app.pause turns the human step into a control-plane primitive that parks the execution and resumes on the webhook, so nothing promotes itself while you are asleep.

Run it

Feed the classifier some labeled misses, then run the improver:

curl -s -X POST http://localhost:8080/api/v1/execute/prompt-improver.propose \

-H "Content-Type: application/json" -d '{"input": {}}'

# => {"result": {"action": "proposed",

# "rationale": "misses were refund questions labeled 'other' not 'billing'",

# "cost_usd": 0.09}}

Ship the proposal as classifier 2.1.0, seed it at 5 percent, and run the rollout operator on a schedule. Watch the improved version climb only if it wins, and stop for your approval before it wins outright:

curl -s http://localhost:8080/api/v1/connector/reasoners/classifier/versions \

-H "X-Connector-Token: $AF_CONNECTOR_TOKEN"

# 2.1.0 weight rises tick by tick, then parks below the ceiling awaiting approval.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Related

The deployment that promotes itself, the prerequisite: the traffic-weight rollout operator this loop hands its new version to.

Human approval in 20 lines, for the app.pause gate that keeps the agent from promoting its own rewrite.
