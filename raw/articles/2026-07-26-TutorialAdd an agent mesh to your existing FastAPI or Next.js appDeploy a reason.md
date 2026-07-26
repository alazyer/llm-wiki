---
title: "TutorialAdd an agent mesh to your existing FastAPI or Next.js appDeploy a reasoning agent as a sidecar service and call it over REST from the app you already have. No framework adoption, no rewrite.Jul 2, 2026 · 27 min"
source_url: "https://agentfield.ai/blog/add-agents-to-your-existing-app"
ingested: 2026-07-26
blog: "AgentField"
published: "2026-07-24"
---

## Source Metadata

- **Blog:** AgentField
- **URL:** https://agentfield.ai/blog/add-agents-to-your-existing-app
- **Fetched:** 2026-07-26
- **Extracted title:** Add an agent mesh to your existing FastAPI or Next.js app

## Article Content

Add an agent mesh to your existing FastAPI or Next.js app — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Add an agent mesh to your existing FastAPI or Next.js app

Deploy a reasoning agent as a sidecar service and call it over REST from the app you already have. No framework adoption, no rewrite.

Santosh Kumar Radha·Co-founder & CTO·

27 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

You have a FastAPI backend or a Next.js app in production. You want one route to do something that needs judgment: triage a support ticket, classify an inbound lead, decide whether a refund is worth escalating. You do not want to rewrite the app around an agent framework to get there.

So don't. Run the agent as a separate service and call it over HTTP from the code you already have.

Here is the whole thing: a 25-line agent that triages support tickets, and the three ways to call it from an app that knows nothing about AgentField.

The agent (25 lines)

A reasoner is a registered function. AgentField registers it with the control plane and exposes it as a REST endpoint. The structured .ai() call returns output validated against a schema, so the shape is fixed at the call site.

PythonTypeScriptGo

import os

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="support-triage")

class Triage(BaseModel):

category: str # "billing" | "bug" | "how-to" | "abuse"

priority: str # "low" | "normal" | "high" | "urgent"

summary: str

confident: bool

@app.reasoner(tags=["entry"])

async def triage(ticket: str, model: str | None = None) -> Triage:

return await app.ai(

system="You triage support tickets. Return category, priority, one-line summary.",

user=ticket,

schema=Triage,

model=model,

if __name__ == "__main__":

app.run(host="0.0.0.0", port=int(os.getenv("PORT", "8001")), auto_port=False)

Run it. It registers with the control plane at AGENTFIELD_SERVER and starts listening. The reasoner is now live at POST /api/v1/execute/support-triage.triage. No route wiring, no OpenAPI hand-authoring. The type hint on triage and the Triage schema define the contract.

The confident flag is not decoration. When app.ai() gets a ticket it cannot read confidently (a wall of pasted logs, a language it did not expect), it says so, and your calling code decides what to do: queue for a human, retry with a stronger model, or fall back to a default. Every .ai() call should have that answer.

Call it from FastAPI

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

Your existing FastAPI route stays a FastAPI route. It gains one httpx POST.

import httpx

from fastapi import FastAPI

from pydantic import BaseModel

api = FastAPI()

AGENTFIELD = "http://localhost:8080"

class TicketIn(BaseModel):

body: str

@api.post("/tickets")

async def create_ticket(t: TicketIn):

async with httpx.AsyncClient(timeout=60) as http:

r = await http.post(

f"{AGENTFIELD}/api/v1/execute/support-triage.triage",

json={"input": {"ticket": t.body}},

r.raise_for_status()

triage = r.json()["result"]

# triage is a plain dict: {"category", "priority", "summary", "confident"}

if triage["priority"] == "urgent":

page_on_call(triage["summary"])

return {"routed_to": triage["category"], "triage": triage}

The body is always {"input": {...}}, where the inner keys are the reasoner's arguments. The response carries the reasoner's output under result. That is the entire integration surface. Your app does not import AgentField. It speaks HTTP.

The synchronous endpoint has a 90-second ceiling. Triage finishes in one LLM call, so it fits. For anything that fans out or runs long, use the async endpoint below.

Call it from a Next.js route handler

Same endpoint, TypeScript fetch, inside an App Router route handler.

// app/api/tickets/route.ts

export async function POST(req: Request) {

const { body } = await req.json();

const res = await fetch(

"http://localhost:8080/api/v1/execute/support-triage.triage",

method: "POST",

headers: { "Content-Type": "application/json" },

body: JSON.stringify({ input: { ticket: body } }),

const { result } = await res.json();

// result: { category, priority, summary, confident }

return Response.json({ routedTo: result.category, triage: result });

The React app calls /api/tickets like any other route. The reasoning lives in a service the frontend never sees.

Call it from Go

One HTTP call, standard library only.

body, _ := json.Marshal(map[string]any{"input": map[string]any{"ticket": ticket}})

resp, _ := http.Post(

"http://localhost:8080/api/v1/execute/support-triage.triage",

"application/json", bytes.NewReader(body),

// decode resp.Body -> { "result": { "category", "priority", "summary", "confident" } }

If you are building the calling service in Go with the AgentField SDK instead of raw HTTP, it is agent.Call(ctx, "support-triage.triage", map[string]any{"ticket": ticket}), which returns a map[string]any and records the call in the workflow DAG.

Slow jobs: use the async endpoint

Some reasoning takes longer than 90 seconds: a ticket that triggers a document lookup, a lead that needs enrichment across three sources. Switch one path segment and you get a job id back immediately.

async with httpx.AsyncClient() as http:

start = await http.post(

f"{AGENTFIELD}/api/v1/execute/async/support-triage.triage",

json={"input": {"ticket": t.body}},

exec_id = start.json()["execution_id"]

while True:

s = (await http.get(f"{AGENTFIELD}/api/v1/executions/{exec_id}")).json()

if s["status"] in ("succeeded", "failed"):

break

await asyncio.sleep(2)

POST /api/v1/execute/async/{agent}.{func} returns an execution_id. Poll GET /api/v1/executions/{execution_id} until status is succeeded or failed. Your web request returns fast; the agent works in the background. Same reasoner, no code change on the agent side.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash, then drop this into an AgentField-aware coding agent (Claude Code, Codex, Gemini CLI, Cursor) and let it scaffold the service:

Build an AgentField agent named "support-triage" with one entry reasoner

`triage(ticket: str, model: str | None = None) -> Triage`. Triage returns a

Pydantic model with fields: category (billing|bug|how-to|abuse), priority

(low|normal|high|urgent), summary (str), confident (bool). Use app.ai with

system+user prompts and schema=Triage, thread `model` through. Expose it via

app.run on PORT 8001. Then add an httpx POST from an existing FastAPI /tickets

route to /api/v1/execute/support-triage.triage with body {"input": {"ticket": ...}}.

Expected file tree:

support-triage/

main.py # Agent + triage reasoner

requirements.txt # agentfield, pydantic

Dockerfile

README.md

Verify it

With the control plane on localhost:8080 and the agent running, confirm registration and fire one real ticket:

curl -fsS http://localhost:8080/api/v1/discovery/capabilities \

| jq '.capabilities[] | select(.agent_id=="support-triage") | .reasoners[].id'

curl -sS -X POST http://localhost:8080/api/v1/execute/support-triage.triage \

-H 'Content-Type: application/json' \

-d '{"input": {"ticket": "Charged twice this month, need a refund before Friday."}}' \

| jq '.result'

Expected result:

"category": "billing",

"priority": "high",

"summary": "Customer double-charged, requests refund by Friday.",

"confident": true

Receipts

Agent code: 25 lines. Caller change per app: one HTTP POST. Frameworks adopted in the calling app: zero. The reasoner became a REST endpoint the moment it registered, and every call it makes to another reasoner is tracked in the workflow DAG, so you can audit what ran and in what order.

Next step: add a second reasoner (enrich_lead) to the same agent and call it from a different route. The pattern does not change: decorate, register, POST.

Related

Fan out 1,000 parallel agents from one request, once one route needs to trigger many agents at once.

Agents that run for three days, for the reasoning that outlasts a single web request.
