---
title: "TutorialThe agent that finds its own toolsA reasoner that calls app.ai(tools="discover") sees every capability registered on the mesh at call time and invokes what it needs, so deploying a new agent tonight makes it callable by tomorrow with zero redeploys.Jul 2, 2026 · 30 min"
source_url: "https://agentfield.ai/blog/agents-that-find-their-own-tools"
ingested: 2026-07-30
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/agents-that-find-their-own-tools

## Article Content

The agent that finds its own tools — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

The agent that finds its own tools

A reasoner that calls app.ai(tools="discover") sees every capability registered on the mesh at call time and invokes what it needs, so deploying a new agent tonight makes it callable by tomorrow with zero redeploys.

Santosh Kumar Radha·Co-founder & CTO·

30 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

You will have three agents running. Two are single-purpose specialists. The third answers a question that needs both, and nobody told it they exist. A self-assembling mesh is a system where an agent discovers every capability registered on the control plane at call time and lets the LLM pick and invoke the ones it needs. Deploy a fourth specialist tonight, and the answering agent can call it tomorrow morning with no code change, no redeploy, and no config edit.

The whole thing is one keyword: tools="discover". About 60 lines of Python across three agents. A run that fans through two specialists costs a few cents. Working in ten minutes.

The pattern

Every other framework makes you hand the LLM a static tool list. You import the tools, you register them, you keep the list in sync as the system grows. Add a capability and you edit every caller that should be allowed to use it. The list is a second copy of your architecture, and it rots.

A self-assembling mesh inverts that. The control plane already knows every reasoner and skill registered against it, with input schemas and descriptions. app.ai(tools="discover") asks the control plane for that catalog at call time, hands it to the LLM as native tool schemas, and runs a discover, call, feed-back loop until the LLM has an answer. The caller never names a tool. Capability grows with the mesh, not with the caller's import block.

Build it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

1. Two small specialists with tagged reasoners

Each specialist registers one reasoner and tags it. Tags are how you scope discovery later.

PythonTypeScriptGo

# currency.py

import os

from agentfield import Agent

app = Agent(node_id="currency", agentfield_server=os.getenv("AGENTFIELD_SERVER"))

_RATES = {"USD": 1.0, "EUR": 0.92, "JPY": 157.0, "GBP": 0.79}

@app.reasoner(tags=["finance", "convert"])

async def convert(amount: float, from_ccy: str, to_ccy: str) -> dict:

"""Convert an amount between two ISO currency codes."""

usd = amount / _RATES[from_ccy]

return {"amount": round(usd * _RATES[to_ccy], 2), "currency": to_ccy}

if __name__ == "__main__":

app.run()

The second specialist is the same shape with a different job. A weather node registers forecast(city: str) tagged ["weather"] and returns a temperature. Keep it small. The point is not the specialist, it is that the specialist registers itself against the control plane and then forgets about everyone else.

2. A generalist that discovers instead of importing

The answering agent imports none of the specialists. In Python and TypeScript it passes tools="discover" to the model call; in Go, auto-discovery is a distinct AIWithTools method. Either way the question happens to need both specialists.

PythonTypeScriptGo

# concierge.py

import os

from agentfield import Agent

app = Agent(node_id="concierge", agentfield_server=os.getenv("AGENTFIELD_SERVER"))

@app.reasoner(tags=["entry"])

async def answer(question: str) -> dict:

result = await app.ai(

system="Answer the user. Use the available tools when you need real data.",

user=question,

tools="discover",

return {

"answer": result.text,

"tools_called": [c.tool_name for c in result.trace.calls],

"turns": result.trace.total_turns,

if __name__ == "__main__":

app.run()

Ask it "What is 200 EUR in JPY, and is it warm in Tokyo right now?" and the LLM sees currency:convert and weather:forecast in the discovered catalog, calls both, and writes one answer. The trace is the receipt: it names exactly which tools the LLM chose. You wrote no dispatch logic. The discovery request is the entire integration.

The return object is a ToolCallResponse. result.text is the final answer, and result.trace carries total_turns, total_tool_calls, and a ToolCallRecord per call with arguments, result, and latency. That trace is how you audit what the mesh did.

3. Scope discovery on a large mesh

Handing the LLM every tool on a hundred-node mesh wastes tokens and confuses the model. Swap the bare discovery request for a config you can scope. Every field below is real:

PythonTypeScriptGo

from agentfield.tool_calling import ToolCallConfig

result = await app.ai(

system="Answer the user. Use tools for real data.",

user=question,

tools=ToolCallConfig(

tags=["finance", "weather"], # only discover tools carrying these tags

agent_ids=["currency", "weather"], # or pin to specific nodes

max_candidate_tools=40, # cap how many tools the LLM sees

schema_hydration="lazy", # send names first, hydrate schemas on selection

max_hydrated_tools=8, # cap full-schema tools after selection

max_turns=6, # cap LLM round-trips

max_tool_calls=12, # cap total tool invocations

tags and agent_ids narrow the catalog before it ever reaches the LLM. In Python and TypeScript, schema_hydration="lazy" is the trick for wide meshes: the first pass sends only tool names and descriptions, the LLM picks the ones it wants, and only those get their full input schemas fetched on the next turn. max_turns and max_tool_calls are your budget caps. Set them. A discover loop without caps can spin.

In Python and TypeScript a plain object works too, if you would rather not import the class: tools={"tags": ["finance"], "max_turns": 6} is parsed into the same config.

4. Watch the auto-invoked hops in the DAG

Every tool call the LLM makes is an app.call under the hood, and every app.call writes an edge into the workflow DAG. The run for one question is a small tree: concierge.answer at the root, currency:convert and weather:forecast as children, invoked because the LLM chose them, not because you wired them. Open the run in the dashboard and the tree shows which tool the model reached for and how long each took.

What the control plane does underneath

tools="discover" is a thin client keyword. The control plane does the work:

Discovery. It holds the live catalog of every registered reasoner and skill, with input schemas, descriptions, tags, and health status. The client asks; the control plane answers with what is registered right now.

Routing. When the LLM emits a tool call, the SDK maps the tool name back to an invocation target and dispatches it through app.call. Same routing, retries, and queueing as any other call.

Tracing. Each auto-invoked call is a DAG edge, so a discovered tool call is as observable as a hand-written one.

Health filtering. health_status lets discovery skip nodes that are down, so the LLM is not offered a tool that cannot answer.

Run it

Start all three agents, then fire the entry reasoner:

curl -s -X POST http://localhost:8080/api/v1/execute/concierge.answer \

-H "Content-Type: application/json" \

-d '{"input": {"question": "What is 200 EUR in JPY, and is it warm in Tokyo?"}}'

# => {"result": {"answer": "200 EUR is about 34,130 JPY, and Tokyo is ...",

# "tools_called": ["currency__convert", "weather__forecast"],

# "turns": 3}}

Now deploy a third specialist, say a timezone node with local_time(city: str), and start it. Change nothing in concierge. Ask "What time is it in Tokyo and how warm is it?" and the same reasoner calls the new tool. That is the mesh assembling itself: the caller discovered a capability that did not exist when it was written.

The Go and TypeScript SDKs expose the same discover loop. The specialist registration above is shown in all three; the generalist's tools="discover" is ctx.ai(prompt, { tools: "discover", schema }) in TypeScript and the ExecuteToolCallLoop over agent.Discover in Go. Details are in the docs.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Next step: clone the three files, start the two specialists, then start a third one after the concierge is already running and watch it get called without a redeploy.

Related

Pipelines that build themselves, when the mesh should decide its own shape per input on top of discovering its own tools.

Add agents to your existing app, to expose an endpoint you already have as a discoverable capability.

Fan out 1,000 parallel agents, for the recursion pattern that these discovered calls run on.
