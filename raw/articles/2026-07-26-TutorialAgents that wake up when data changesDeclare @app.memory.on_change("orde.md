---
title: "TutorialAgents that wake up when data changesDeclare @app.memory.on_change("order_*") and a handler fires whenever a matching key changes, written by any agent, in any scope. No cron, no broker, no consumer groups.Jul 2, 2026 · 33 min"
source_url: "https://agentfield.ai/blog/reactive-agents"
ingested: 2026-07-26
blog: "AgentField"
published: "2026-07-24"
---

## Source Metadata

- **Blog:** AgentField
- **URL:** https://agentfield.ai/blog/reactive-agents
- **Fetched:** 2026-07-26
- **Extracted title:** Agents that wake up when data changes

## Article Content

Agents that wake up when data changes — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Agents that wake up when data changes

Declare @app.memory.on_change("order_*") and a handler fires whenever a matching key changes, written by any agent, in any scope. No cron, no broker, no consumer groups.

Santosh Kumar Radha·Co-founder & CTO·

33 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

You want a second agent to run the moment a first agent finishes writing its result, without polling for it and without standing up a message broker to carry the signal. By the end of this post you have one agent writing enriched state to shared memory and another agent that wakes up on the write, reacts, and writes its own key, which wakes a third. The chain shows up in the DAG as a single causal trace.

A reactive agent is one that declares which memory keys it cares about and runs a handler whenever a matching key changes, no matter which agent wrote it or which scope it landed in. The write is the trigger. There is no queue in the middle.

Receipts: about 40 lines across three agents, one memory write per hop, no infrastructure to provision beyond the agents themselves. The reaction latency is a websocket round-trip, not a poll interval.

The pattern

The usual way to make agent B react to agent A is to put a broker between them. A publishes to a topic, B joins a consumer group, and now you own partitions, offsets, redelivery, and a dead-letter queue for the messages B choked on. The broker is not the feature. It is the tax you pay to get the feature.

Reactive memory removes the middle. Agent A writes a key to shared memory as it already does. Agent B declares @app.memory.on_change("order_*") and receives an event the instant any key matching that pattern changes, whoever wrote it. Patterns are wildcards: order_* matches order_1042, order_1043, and every future order key without B knowing them in advance. B does not subscribe to A. B subscribes to a shape of key, and the plane routes the change to it.

Build it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

1. One agent writes enriched state

The enricher writes a key into shared memory. In Python app.memory.set(key, data) and app.memory.get(key, default=...) use hierarchical scoping by default; to pin a write to a named scope you go through a scoped accessor, like app.memory.global_scope.set(key, data) for the shared scope every agent can read. Everything a downstream reactor should see, including any metadata, goes inside the data payload, because that payload is what the change event carries.

PythonTypeScriptGo

from agentfield import Agent

app = Agent(node_id="order-enricher")

@app.reasoner(tags=["entry"])

async def enrich(order_id: str, raw: dict) -> dict:

enriched = {

"order_id": order_id,

"total": raw["total"],

"region": lookup_region(raw["ship_to"]),

"risk_tier": "high" if raw["total"] > 5000 else "normal",

"source": "order-enricher", # metadata travels inside the value

# Written to global scope so any agent can react to it.

await app.memory.global_scope.set(f"order_{order_id}", enriched)

return enriched

2. A second agent reacts

The reactor declares the pattern and defines a handler. The handler receives one change event carrying the key that changed, the scope it changed in, and the new data. No polling loop, no get on a timer. The subscription is a Python and TypeScript primitive; a Go agent joins the chain by writing keys the others react to (covered below).

PythonTypeScript

from agentfield import Agent, MemoryChangeEvent

app = Agent(node_id="fulfillment")

@app.memory.on_change("order_*")

async def on_order(event: MemoryChangeEvent):

order = event.data # the value the enricher wrote

print(f"{event.key} changed in {event.scope} via {event.action}")

if order["risk_tier"] == "high":

# React, then write our own key so the next agent can wake up.

hold = {"order_id": order["order_id"], "state": "held_for_review"}

await app.memory.global_scope.set(f"hold_{order['order_id']}", hold)

else:

await app.call("warehouse.reserve_stock", order_id=order["order_id"])

The pattern order_* becomes the anchored regex ^order_.*$, so it matches every order_ key and nothing else. Match on order_1042 exactly, on order_*, or on a list of patterns; the plane checks each change against every active subscription and delivers the ones that match.

3. Chain the reactions

Because the fulfillment handler wrote hold_<id>, a third agent that watches hold_* wakes up on it. No agent names any other agent. Each one declares a pattern and writes a key, and the chain assembles itself from the writes.

PythonTypeScript

from agentfield import Agent, MemoryChangeEvent

app = Agent(node_id="review-queue")

@app.memory.on_change("hold_*")

async def on_hold(event: MemoryChangeEvent):

held = event.data

# A held order was written; queue it for a human reviewer.

await app.call("hitl.request_review", order_id=held["order_id"])

Three agents, zero direct references between them. enrich writes order_1042, which wakes fulfillment, which writes hold_1042, which wakes review-queue. Each hop is one memory write and one handler. The DAG stitches the writes into one causal trace, so you can read the whole chain from the enrichment that started it to the review request that ended it.

Memory scopes

A change fires against the scope it was written into, and reactors can filter to a scope. The four scopes, widest to narrowest:

ScopeLives forReact here when

globaluntil deletedany agent, any session should see the change (cross-agent state)

sessionone conversationthe reaction belongs to a single user session

actoracross sessions, per actorthe change is specific to one actor's state

workflowone workflow executionthe reaction is scoped to intermediate results of one run

The examples above write through global_scope so every agent can react. To confine a reaction to one session or one run, write through the matching scoped accessor (app.memory.session(id), app.memory.workflow(id)) and declare on_change on that same scoped accessor, for example app.memory.session(id).on_change("order_*").

The API as a building block

Memory read and write are the primitives. Every SDK exposes them; the reactive on_change subscription is Python and TypeScript.

PythonTypeScriptGo

# Write, read, and react.

await app.memory.global_scope.set("order_1042", enriched)

order = await app.memory.global_scope.get("order_1042")

@app.memory.on_change("order_*")

async def handler(event: MemoryChangeEvent):

... # event.key, event.scope, event.action, event.data

Go writes and reads the same shared memory, so a Go agent can be a link in a reactive chain by writing the key that a Python or TypeScript reactor is watching. It just cannot be the one that declares the subscription yet.

What the control plane does underneath

Change fan-out. Every memory.set and memory.delete becomes a change event; the plane matches it against active subscriptions and delivers only the matches.

Pattern routing. Wildcards compile to anchored regexes, so a reactor names a shape of key, not a fixed list, and new keys route to it automatically.

Scope filtering. Subscriptions can pin to a scope and scope id, so a workflow-scoped reaction never fires on a global write.

Delivery over websocket. Reactors hold a websocket to the change stream that reconnects with backoff, so a dropped connection resubscribes instead of losing the reactor.

Causal tracing. The writes that trigger reactions land in the workflow DAG, so a chain of reacting agents reads as one trace instead of a scatter of unlinked calls.

Run it

With the control plane on localhost:8080 and the three agents running, write an order and watch the chain fire. Writing the key is what triggers everything downstream.

# Kick the enricher; it writes order_1042 to global memory.

curl -sS -X POST http://localhost:8080/api/v1/execute/order-enricher.enrich \

-H 'Content-Type: application/json' \

-d '{"input": {"order_id": "1042", "raw": {"total": 9000, "ship_to": "SG"}}}' \

| jq '.result'

# The write reaches fulfillment, which writes hold_1042.

# Read it back to confirm the reaction ran.

curl -sS -X POST http://localhost:8080/api/v1/memory/get \

-H 'Content-Type: application/json' \

-d '{"key": "hold_1042", "scope": "global"}' \

| jq

A high-risk order returns from enrich, and a moment later hold_1042 exists in memory because fulfillment woke up and wrote it, which in turn woke review-queue. You called one endpoint. The chain ran itself.

Give this to your coding agent

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash, then paste this into an AgentField-aware coding agent (Claude Code, Codex, Cursor) and let it scaffold the reactive chain.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Count what you did not deploy: no message broker, no consumer groups, no offset commits, no dead-letter queue for the reactions that failed. The trigger is a memory write, and the plane carries it. The next step is to add a scope filter, declare the handler on app.memory.workflow(id).on_change("order_*"), and watch the same reactor fire only for orders written inside a single workflow run.

Related

Agents that run for three days, for the long-running work a reaction can kick off.

The agent that finds its own tools, once reactors need to discover which agent to call at runtime.
