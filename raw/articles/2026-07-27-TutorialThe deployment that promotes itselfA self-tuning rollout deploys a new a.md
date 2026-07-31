---
title: "TutorialThe deployment that promotes itselfA self-tuning rollout deploys a new agent version at a small traffic weight, then an operator agent polls per-version failure rates and shifts weight itself, promoting on wins and rolling back on regressions.Jul 2, 2026 · 30 min"
source_url: "https://agentfield.ai/blog/self-tuning-rollouts"
ingested: 2026-07-27
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/self-tuning-rollouts

## Article Content

The deployment that promotes itself — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

The deployment that promotes itself

A self-tuning rollout deploys a new agent version at a small traffic weight, then an operator agent polls per-version failure rates and shifts weight itself, promoting on wins and rolling back on regressions.

Santosh Kumar Radha·Co-founder & CTO·

30 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

A self-tuning rollout is a deployment where an agent watches per-version metrics and shifts traffic itself. You deploy version 2.1.0 next to 2.0.0 at a 5 percent traffic weight, and an operator reasoner takes over from there: it polls how each version is doing, raises the new version's weight when it wins, drops it to zero when it regresses, and pauses for a human before the last step to 100 percent.

By the end you have three moving parts running against a live control plane: two versions of one agent registered under different version strings, the traffic-weight API that splits requests between them, and an operator reasoner that reads outcomes and calls that API on a loop. The operator is about 60 lines of Python. Weighted routing, the version registry, and every execution record it reads are not your code. A full rollout loop costs a few cents in operator LLM calls plus whatever the production traffic itself costs, and it works the first time you point it at two running versions.

The pattern

Container canaries shift traffic between two images and watch CPU, latency, and error codes. That works when the thing you are rolling out is a binary and the signal is an HTTP status. Agent versions break both assumptions. The change is a prompt, a model, a tool set, or a control loop, and the regression shows up as worse judgment, not a 500. A version that returns 200 on every request can still be quietly wrong.

A self-tuning rollout is canary for behavior. The two versions are the same agent with different version strings. The metric is an outcome the agent itself produces, read back from execution history. The controller that moves traffic is not a YAML file in a CI pipeline, it is an agent that reasons about the numbers and can hold before the risky step. Static weights in a config file cannot do that. They ship on a human's schedule, not on the data's.

Build it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

The Go and TypeScript SDKs expose the same surface; the orchestration below is Python for length.

1. Register two versions of the same agent

Version is a constructor argument. Run the same node_id twice with different version strings and the control plane keeps both as separate versioned records under one logical agent.

PythonTypeScriptGo

# classify_v2_0_0.py (the incumbent)

import os

from agentfield import Agent

app = Agent(

node_id="classifier",

version="2.0.0",

agentfield_server=os.getenv("AGENTFIELD_SERVER"),

@app.reasoner(tags=["entry"])

async def classify(text: str) -> dict:

label = await app.ai(

system="Classify the ticket as billing, bug, or other. One word.",

user=text,

return {"label": label.strip().lower(), "version": app.version}

if __name__ == "__main__":

app.run()

The challenger is the same file with version="2.1.0" and a revised prompt. Start both processes. Each registers under classifier, and the control plane now holds two versions of it. Requests to classifier.classify route to one version or the other by weight.

2. Split traffic with the weight API

Traffic weight per version is set over REST through the connector surface. It takes an integer from 0 to 10000, and the router does a weighted round-robin across healthy versions of the agent. Start the challenger low.

# Send 5% to 2.1.0. Leave 2.0.0 at its default weight of 100.

curl -s -X PUT \

http://localhost:8080/api/v1/connector/reasoners/classifier/versions/2.1.0/weight \

-H "X-Connector-Token: $AF_CONNECTOR_TOKEN" \

-H "Content-Type: application/json" \

-d '{"weight": 5}'

# => {"success": true, "id": "classifier", "version": "2.1.0",

# "previous_weight": 100, "new_weight": 5}

The connector surface is gated: it is only mounted when the connector feature is enabled and a token is set, and every call carries X-Connector-Token. That token is what lets an agent, rather than a human at a terminal, drive the rollout.

Read current weights and health back the same way:

curl -s http://localhost:8080/api/v1/connector/reasoners/classifier/versions \

-H "X-Connector-Token: $AF_CONNECTOR_TOKEN"

# => {"versions": [

# {"version": "2.0.0", "traffic_weight": 100, "health_status": "active", ...},

# {"version": "2.1.0", "traffic_weight": 5, "health_status": "active", ...}]}

3. Write the operator reasoner

The operator does four things in a loop: pull recent executions, compute a failure rate per version, decide the next weight, and call the weight API. Keep the metric honest and simple. Here it is the fraction of executions that ended in a failed status.

One thing to check against the docs for your build: execution queries filter by status and by agent, so the operator below tags each version's own outcomes into memory as they happen and reads that back per version. Do not assume an execution list filters by version string unless you confirm it; the memory tag is what makes the per-version split reliable.

# operator.py

import os

import httpx

from agentfield import Agent

app = Agent(node_id="rollout-operator", agentfield_server=os.getenv("AGENTFIELD_SERVER"))

CP = os.getenv("AGENTFIELD_SERVER", "http://localhost:8080")

TOKEN = os.getenv("AF_CONNECTOR_TOKEN")

AGENT = "classifier"

CHALLENGER = "2.1.0"

INCUMBENT = "2.0.0"

FLOOR, CEILING = 0, 100 # weight bounds for the challenger

STEP = 20 # max change per tick

PROMOTE_BELOW = 0.05 # promote if challenger failure rate under 5%

ROLLBACK_ABOVE = 0.15 # roll back if it climbs over 15%

async def set_weight(version: str, weight: int) -> None:

weight = max(FLOOR, min(CEILING, weight))

async with httpx.AsyncClient() as http:

await http.put(

f"{CP}/api/v1/connector/reasoners/{AGENT}/versions/{version}/weight",

headers={"X-Connector-Token": TOKEN},

json={"weight": weight},

def failure_rate(outcomes: list[dict]) -> float:

if not outcomes:

return 0.0

failed = sum(1 for o in outcomes if o.get("status") == "failed")

return failed / len(outcomes)

@app.reasoner(tags=["entry"])

async def tick(current_weight: int) -> dict:

# Per-version outcomes the production reasoner tagged into memory.

challenger = (await app.memory.get(f"outcomes.{CHALLENGER}", []))[-100:]

if len(challenger) < 20:

return {"action": "hold", "reason": "not enough traffic yet",

"weight": current_weight}

rate = failure_rate(challenger)

if rate > ROLLBACK_ABOVE:

await set_weight(CHALLENGER, FLOOR)

return {"action": "rollback", "failure_rate": rate, "weight": FLOOR}

if rate < PROMOTE_BELOW:

next_weight = min(CEILING, current_weight + STEP)

await set_weight(CHALLENGER, next_weight)

return {"action": "promote", "failure_rate": rate, "weight": next_weight}

return {"action": "hold", "failure_rate": rate, "weight": current_weight}

if __name__ == "__main__":

app.run()

For this to have per-version numbers to read, the production reasoner appends its own result to memory keyed by version. Add a few lines inside classify, where await is available:

PythonTypeScriptGo

@app.reasoner(tags=["entry"])

async def classify(text: str) -> dict:

label = await app.ai(

system="Classify the ticket as billing, bug, or other. One word.",

user=text,

result = {"label": label.strip().lower(), "version": app.version}

outcomes = await app.memory.get(f"outcomes.{app.version}", default=[])

outcomes.append({"status": "succeeded", "label": result["label"]})

await app.memory.set(f"outcomes.{app.version}", outcomes[-500:])

return result

4. Add the safety rails

The rails are already in the operator, and they are the point.

Floor and ceiling. set_weight clamps to [FLOOR, CEILING]. The challenger cannot exceed the ceiling on its own, so a runaway loop cannot silently promote to 100.

Max step size. STEP caps how far one tick can move traffic. Twenty points at a time means a bad version bleeds into at most 20 percent of requests before the next tick catches it.

Rollback beats promotion. The rollback check runs first and jumps straight to the floor. A regression does not step down gently, it stops.

A human gate before full traffic. The ceiling is 100, but the operator never sets 100 on its own. The last step needs a person.

Wire the last one with app.pause. Once the challenger reaches the ceiling with a clean rate, ask for approval before removing the incumbent entirely.

if next_weight >= CEILING and rate < PROMOTE_BELOW:

decision = await app.pause(

approval_request_id=f"promote-{CHALLENGER}",

approval_request_url="https://your-console/approvals",

if decision.approved:

await set_weight(CHALLENGER, CEILING) # incumbent drained separately

return {"action": "promoted", "approved": True}

return {"action": "hold", "approved": False}

app.pause blocks the execution until someone resolves it through the approval webhook. The agent proposes full promotion; a human disposes. Everything below the ceiling runs unattended.

What the control plane does underneath

Version registry. Two processes with the same node_id and different version strings become two versioned records under one agent. You did not build a registry.

Weighted routing. classifier.classify resolves to a version by weighted round-robin over the healthy versions. A version marked unhealthy drops out of selection automatically.

Execution history. Every call is a recorded execution with a status, which is the raw material the operator reads to judge each version.

Approval gating. app.pause parks an execution and resumes it when the approval webhook fires, so the human gate is a control-plane primitive, not a Slack message you have to poll.

Run it

Start the incumbent, the challenger, and the operator. Seed the challenger at 5 percent (step 2 above), then drive one operator tick:

curl -s -X POST http://localhost:8080/api/v1/execute/rollout-operator.tick \

-H "Content-Type: application/json" \

-d '{"input": {"current_weight": 5}}'

# => {"result": {"action": "promote", "failure_rate": 0.02, "weight": 25}}

Read the weights back and watch 2.1.0 climb as its failure rate stays low:

curl -s http://localhost:8080/api/v1/connector/reasoners/classifier/versions \

-H "X-Connector-Token: $AF_CONNECTOR_TOKEN"

Run tick on a schedule and the rollout drives itself between the floor and the ceiling, stopping for you at the top.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Related

An agent that ships new versions of itself, the sequel: the agent that writes the challenger version this rollout then canaries.

Human approval in 20 lines, for the app.pause gate that guards the last step to full traffic.
