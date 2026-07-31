---
title: "TutorialYour personal control planeRun AgentField in local mode on your own laptop or homelab, register one agent, and call it over HTTP. This is the substrate the rest of the personal stack builds on.Jul 2, 2026 · 24 min"
source_url: "https://agentfield.ai/blog/your-personal-control-plane"
ingested: 2026-07-29
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/your-personal-control-plane

## Article Content

Your personal control plane — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Your personal control plane

Run AgentField in local mode on your own laptop or homelab, register one agent, and call it over HTTP. This is the substrate the rest of the personal stack builds on.

Santosh Kumar Radha·Co-founder & CTO·

24 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

This is the first post in the personal stack: a series where you build a running set of agents for yourself and never tear it down. Everything later (an assistant you text, a nightly repo fleet, approvals on your phone) registers into the one thing you start here.

A personal control plane is a single local process that agents register into, that exposes every agent as an HTTP endpoint, and that records every call so you can see what ran. You run it on your laptop or a homelab box, it uses SQLite and needs no Docker and no cloud account, and by the end of this post you will have one agent live and answering a curl.

Receipts: one binary, one 12-line agent, zero external services. Time to a working call: about five minutes.

What "control plane" means here

Kubernetes does not run your containers by hand; it is the thing containers register into, and it gives you one place to schedule, route, and observe them. AgentField is that, for agents. You do not wire routes or hand-author an OpenAPI spec. You write a function, decorate it, and the control plane turns it into POST /api/v1/execute/<node>.<function>, tracks every call in a workflow DAG, and stores memory the agent can read back later.

Run it locally and you get the same surface a cloud deployment has: a dashboard, the execute API, memory scopes, and DAG tracing. Nothing about the code changes when you later point it at Postgres. The local process is the real thing, sized for one person.

Start the control plane

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

Install the CLI, then start the server in local mode. Local mode uses SQLite plus BoltDB and starts with no external dependencies.

curl -fsSL https://agentfield.ai/install.sh | bash

af server

af server boots the control plane on http://localhost:8080. The dashboard is at http://localhost:8080/ui/. Leave it running in its own terminal; the rest of the personal stack talks to it.

Local auth is off until you configure a key, so localhost calls need no header. When you later expose this box to the internet, set an API key and the control plane starts requiring it. For now, on your own machine, there is nothing to configure.

Scaffold and register one agent

Open a second terminal. Scaffold an agent, then run it. The agent is a separate process that registers itself with the control plane over HTTP.

PythonTypeScriptGo

# main.py

import os

from agentfield import Agent

app = Agent(node_id="hello")

@app.reasoner(tags=["entry"])

async def greet(name: str) -> dict:

return {"message": f"hello, {name}"}

if __name__ == "__main__":

app.run(host="0.0.0.0", port=int(os.getenv("PORT", "8001")), auto_port=False)

The Python scaffold comes from af init hello --language python (use -l typescript or -l go for the others). Run it with af run from the project directory, or just run the file directly.

The moment the process starts, it registers with the control plane and the reasoner is live at POST /api/v1/execute/hello.greet. There is no route to wire and no schema to publish by hand. The function signature is the contract.

What the control plane does underneath

The agent process is small; the control plane behind it is doing the infrastructure work:

Routing: it resolves hello.greet to the running process and forwards the call.

Tracing: every call becomes a node in a workflow DAG you can open in the dashboard, so agent-to-agent calls are auditable in order.

Memory: it holds four scopes (global, session, actor, workflow) that agents read and write through accessors, persisted in SQLite.

Registration: it tracks which agents are up and what reasoners they expose, so other agents can discover them.

You call one HTTP endpoint. The control plane handles the rest.

Run it

With af server up and the agent running, confirm the reasoner registered and fire one real call:

curl -fsS http://localhost:8080/api/v1/discovery/capabilities \

| jq '.capabilities[] | select(.agent_id=="hello") | .reasoners[].id'

curl -sS -X POST http://localhost:8080/api/v1/execute/hello.greet \

-H 'Content-Type: application/json' \

-d '{"input": {"name": "Sam"}}' \

| jq '.result'

Expected result:

{ "message": "hello, Sam" }

The body is always {"input": {...}}, where the inner keys are the reasoner's arguments. The response carries the reasoner's return value under result. That envelope is the same for every agent you add to the personal stack.

Keep it running

You want the control plane up whenever your laptop or homelab box is up, not just when a terminal is open. Register it with your init system once.

macOS (launchd)

Linux (systemd user service)

Paste this into a coding agent

Drop this into an AgentField-aware coding agent (Claude Code, Codex, Gemini CLI, Cursor) and let it scaffold the first node:

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Next step: give this control plane a front door you can text. The assistant post below adds a reasoner with per-chat memory and a Telegram bridge, so you can talk to your stack from your phone.

Related

Build your own personal assistant, the front door to everything you register here.

A nightly maintenance fleet for your repos, the first agent that does real work while you sleep.

Add an agent mesh to your existing FastAPI or Next.js app, if you want to call these reasoners from an app you already run.

What harness orchestration is, for the model behind the control plane.
