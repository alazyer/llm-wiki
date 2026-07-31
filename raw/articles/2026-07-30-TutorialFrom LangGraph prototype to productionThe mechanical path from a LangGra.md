---
title: "TutorialFrom LangGraph prototype to productionThe mechanical path from a LangGraph StateGraph that works in a notebook to reasoners other services can call, retry, and run for 40 minutes.Jul 2, 2026 · 24 min"
source_url: "https://agentfield.ai/blog/langgraph-to-production"
ingested: 2026-07-30
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/langgraph-to-production

## Article Content

From LangGraph prototype to production — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

From LangGraph prototype to production

The mechanical path from a LangGraph StateGraph that works in a notebook to reasoners other services can call, retry, and run for 40 minutes.

Santosh Kumar Radha·Co-founder & CTO·

24 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

The graph works. You built it in a notebook with LangGraph, wired two nodes together, and it does the thing. Then three requests land on the same afternoon.

The recommendations service wants to call it. The billing service wants to call it. Someone on the data team wants to run it over a backlog of 8,000 records overnight, and one of those records triggers a chain that takes 40 minutes. Now you need a queue, retries when the model 500s, a way for those three callers to reach the graph without importing your Python, and a trace when a run fails at minute 38.

None of that is what LangGraph is for. LangGraph is good at the part you already finished: authoring how the agent reasons, the state it carries, the branches it takes. The work that just showed up is deployment work. This post is the mechanical translation from the first to the second.

The LangGraph version

Here is a small graph. Two nodes: draft a summary, then critique it. State flows between them.

from typing import TypedDict

from langgraph.graph import StateGraph, START, END

from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")

class State(TypedDict):

document: str

draft: str

critique: str

def summarize(state: State) -> dict:

resp = llm.invoke(f"Summarize in 3 bullets:\n\n{state['document']}")

return {"draft": resp.content}

def critique(state: State) -> dict:

resp = llm.invoke(f"List gaps in this summary:\n\n{state['draft']}")

return {"critique": resp.content}

builder = StateGraph(State)

builder.add_node("summarize", summarize)

builder.add_node("critique", critique)

builder.add_edge(START, "summarize")

builder.add_edge("summarize", "critique")

builder.add_edge("critique", END)

graph = builder.compile()

result = graph.invoke({"document": open("report.txt").read()})

print(result["critique"])

This runs in one process. You call graph.invoke() from Python, in the same interpreter, and read the return value. That is the assumption the next three callers break.

The same logic as reasoners

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

Each node becomes a reasoner. Edges become app.call(). The State TypedDict becomes explicit inputs and outputs, or app.memory when a value needs to outlive a single call.

PythonTypeScriptGo

import os

from pydantic import BaseModel

from agentfield import Agent, AIConfig

app = Agent(

node_id="doc-summarizer",

agentfield_server=os.getenv("AGENTFIELD_SERVER", "http://localhost:8080"),

ai_config=AIConfig(model=os.getenv("AI_MODEL", "openrouter/openai/gpt-4o-mini")),

class Draft(BaseModel):

draft: str

class Critique(BaseModel):

critique: str

@app.reasoner(tags=["entry"])

async def summarize(document: str, model: str | None = None) -> Draft:

return await app.ai(user=f"Summarize in 3 bullets:\n\n{document}", schema=Draft, model=model)

@app.reasoner()

async def critique(draft: str, model: str | None = None) -> Critique:

return await app.ai(user=f"List gaps in this summary:\n\n{draft}", schema=Critique, model=model)

@app.reasoner(tags=["entry"])

async def review_document(document: str, model: str | None = None) -> Critique:

summary = await app.call(f"{app.node_id}.summarize", document=document, model=model)

result = await app.call(f"{app.node_id}.critique", draft=summary["draft"], model=model)

return Critique(**result)

if __name__ == "__main__":

app.run(host="0.0.0.0", port=int(os.getenv("PORT", "8001")))

The mapping is one reasoner per node. add_edge("summarize", "critique") becomes an app.call() line inside review_document. The state dict that LangGraph passed between nodes becomes named arguments on the wire.

One detail that bites people: app.call() crosses a JSON boundary and returns a plain dict, even when the target reasoner is annotated to return a Pydantic model. That is why the code reads summary["draft"], not summary.draft, and reconstructs with Critique(**result) when it needs the typed object back. The type hint documents the shape on the wire, not the type in memory.

What you gain without writing it

The two files above are close in length. The reasoner version is doing more, and the extra work lives in the control plane rather than in your code.

Each reasoner is a REST endpoint the moment it registers. The billing service does not import your Python. It POSTs to the control plane and gets a result back, the same way it calls any other internal service.

curl -X POST http://localhost:8080/api/v1/execute/doc-summarizer.review_document \

-H "Content-Type: application/json" \

-d '{"input": {"document": "Q3 revenue came in at..."}}'

You get async execution, retries on transient model failures, and webhook delivery for long runs without writing a queue. A 40-minute run does not sit on an open HTTP connection hoping the load balancer does not reap it. The control plane runs it in the background and calls you back. Bind a reasoner to an external event with a trigger and it fires on its own (triggers are available in Python and Go):

PythonGo

from agentfield import EventTrigger

@app.reasoner(

triggers=[EventTrigger(source="stripe", types=["invoice.finalized"], secret_env="STRIPE_SECRET")],

async def on_invoice(payload: dict, model: str | None = None) -> Critique:

return await review_document(document=payload["invoice"]["memo"], model=model)

Every app.call() is recorded in the workflow DAG. When review_document fails, you see which hop failed, its inputs, and its timing, without stitching together five log streams. The State you were threading by hand becomes memory scopes (workflow, session, actor, global) that the control plane syncs for you. New callers find the reasoner through service discovery on heartbeat, so there are no hardcoded URLs and no service mesh to configure.

What you keep

The migration does not touch your reasoning. Your prompts move over as strings. Your branching logic stays Python. Your model choices stay yours, and app.ai(model=...) is per call, so you can route the summary to a cheap model and the critique to a stronger one on the same request. If your LangGraph node ran a tool loop, app.ai(tools=[...]) is the multi-turn tool-using form of the same call.

The graph shape survives too. A conditional edge in LangGraph is an if in the entry reasoner deciding which app.call() to make next. A fan-out is asyncio.gather over several app.call() lines. You are moving where the graph runs, not redrawing it.

When to stay on LangGraph

Do not migrate for its own sake. Stay in LangGraph when:

It is still a prototype and no second service calls it. If graph.invoke() from one process is the whole requirement, a control plane is weight you do not need yet.

You run in a single process and want to keep it that way. The value here is cross-service and cross-team. One notebook, one caller, one owner does not trigger it.

You have hard sub-second latency budgets. Each app.call() hop goes through the control plane, which adds roughly 100 to 200ms per hop. For a two-node graph on a user-facing request with a 300ms SLA, that overhead is real. Batch and background work absorbs it without noticing; a hot synchronous path may not.

The line is whether the prototype has to be callable by something other than you. Until it does, LangGraph alone is the right amount of machinery.

Paste this into /agentfield

Hand your coding agent the spec, not the keystrokes. Install the CLI with curl -fsSL https://agentfield.ai/install.sh | bash; /agentfield runs in Claude Code, Codex, Gemini CLI, and other coding agents.

Migrate my 2-node LangGraph StateGraph (summarize -> critique) to AgentField

reasoners. One reasoner per node. Edges become app.call(). State becomes named

inputs/outputs. Add an entry reasoner review_document that chains the two.

Thread `model` through every reasoner. Reconstruct Pydantic from the dict that

app.call() returns. Add a Dockerfile and a curl verify command.

Expected tree:

doc-summarizer/

main.py # Agent(node_id="doc-summarizer"), 3 reasoners

Dockerfile

requirements.txt

Verify it registered and answers:

curl -X POST http://localhost:8080/api/v1/execute/doc-summarizer.review_document \

-H "Content-Type: application/json" \

-d '{"input": {"document": "Q3 revenue came in at $4.2M, up 18% QoQ..."}}'

Receipts

A 2-node graph moved in about 20 minutes: from 32 lines of LangGraph to 41 lines of reasoners, plus a 6-line Dockerfile. The line count barely moved because the queue, the retry logic, the REST layer, the trace, and the service discovery are not in the file. They are in the control plane.

A 5-node graph is the same mechanical loop five times: five reasoners, four or five app.call() edges in the entry reasoner, roughly 45 minutes end to end including the Dockerfile and one curl check. There is no rewrite, because the reasoning never changed. You moved where it runs.

Next: point one of your other services at the new endpoint and delete the import of the graph. When the second caller works through the control plane, the migration paid for itself.

Related

Fan out 1,000 parallel agents from one request, for turning a conditional edge into a real parallel fan-out.

Add an agent mesh to your existing FastAPI or Next.js app, for calling the migrated reasoners from the services you already run.
