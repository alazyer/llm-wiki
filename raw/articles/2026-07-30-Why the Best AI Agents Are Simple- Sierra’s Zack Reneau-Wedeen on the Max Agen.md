---
title: "Why the Best AI Agents Are Simple: Sierra’s Zack Reneau-Wedeen on the Max Agency Podcast"
source_url: "https://agentfield.ai/blog/fan-out-1000-agents"
ingested: 2026-07-30
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/fan-out-1000-agents

## Article Content

Fan out 1,000 parallel agents from one request — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Fan out 1,000 parallel agents from one request

A recursive reasoner that decomposes a question and calls itself in parallel through the control plane, depth-capped, with queueing and tracing handled for you.

Santosh Kumar Radha·Co-founder & CTO·

18 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

One HTTP request comes in. A thousand agent calls go out. The control plane queues them, retries the ones that fail, records every edge in a DAG, and hands you back a single answer.

Here is the artifact. A reasoner takes a question, breaks it into sub-questions, and calls itself once per sub-question. Each of those calls breaks its slice down again. Three levels deep with a branching factor of ten, you have crossed a thousand agent invocations, all from a single curl. The fan-out is 40 lines of Python. The queue, the retry logic, and the trace are not your code.

The hero

Same shape in every SDK. Python fans out with asyncio.gather, Go with goroutines and an errgroup, TypeScript with Promise.all. The recursive call routes through the control plane in all three.

PythonTypeScriptGo

import asyncio

import os

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="fanout", agentfield_server=os.getenv("AGENTFIELD_SERVER"))

class Split(BaseModel):

subquestions: list[str]

confident: bool

@app.reasoner(tags=["entry"])

async def research(question: str, depth: int = 0, model: str | None = None) -> dict:

# Leaf: stop recursing, answer directly.

if depth >= 3:

answer = await app.ai(

system="Answer the question in two sentences.",

user=question,

model=model,

return {"question": question, "answer": answer, "leaf": True}

# Decompose into up to 10 sub-questions.

split = await app.ai(

system="Break this into up to 10 independent sub-questions. Return JSON.",

user=question,

schema=Split,

model=model,

if not split.confident or not split.subquestions:

# .ai() had no clean split, so treat this node as a leaf.

answer = await app.ai(system="Answer directly.", user=question, model=model)

return {"question": question, "answer": answer, "leaf": True}

# Fan out: one call per sub-question, all in flight at once.

children = await asyncio.gather(*[

app.call(f"{app.node_id}.research", question=sub, depth=depth + 1, model=model)

for sub in split.subquestions

summary = await app.ai(

system="Synthesize these child answers into one paragraph.",

user="\n\n".join(c.get("answer", "") for c in children),

model=model,

return {"question": question, "answer": summary, "children": children}

if __name__ == "__main__":

app.run()

The recursion is the whole trick. app.call(f"{app.node_id}.research", ...) calls the same reasoner you are already inside, but the call goes through the control plane, not a Python function pointer. That routing is what turns recursion into fan-out. Every child runs as its own tracked execution, so a node at depth 2 with ten children spawns ten separate executions, each of which can spawn ten more.

asyncio.gather launches all children for a node at once and waits for the set. You do not manage a thread pool or a task queue. The SDK holds an internal semaphore on outbound calls (set by AGENTFIELD_AGENT_MAX_CONCURRENT_CALLS, default is the connection pool size capped at 256), so a single process will not open ten thousand sockets. Above that ceiling, the control plane queue absorbs the backlog and drains it as workers free up.

The depth >= 3 check is the only thing standing between you and an unbounded bill. Keep it. A recursive fan-out without a depth cap spends money fast.

agent.Call in Go and ctx.call in TypeScript are the same primitive as app.call: each returns the child's result and routes through the control plane. The Go errgroup cancels the whole level if one child errors, which is the behavior you want for a research tree; Promise.all and asyncio.gather fan out the level and wait for the set.

What the control plane does underneath

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

app.call() is not an in-process function call. It is a request to the control plane, which then does the work you would otherwise write yourself:

Queueing. Calls beyond the concurrency ceiling land in a durable queue and run when a worker is free. Backpressure is automatic.

Retries. A transient failure (a 5xx from a provider, a malformed response) is retried before the parent ever sees an error.

Tracing. Every app.call writes an edge into the workflow DAG. Parent to child, across all three levels, is one queryable graph. You can open the run in the dashboard and watch the tree fill in.

Model routing. model threads through every call, so you can point the whole tree at a cheap model per request and A/B test cost against quality without touching the code.

Routing through the control plane adds roughly 100 to 200 milliseconds per hop. For a fan-out that overlaps hundreds of calls, that overhead is paid once per level, not once per call, because the level runs concurrently.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Build a recursive research agent named "fanout" with one reasoner, "research".

It takes {question: str, depth: int = 0, model: str | None = None}. At depth >= 3

it answers directly with app.ai and returns {answer}. Below that depth it uses

app.ai with a Pydantic schema {subquestions: list[str], confident: bool} to split

the question into up to 10 sub-questions, then fans out with

asyncio.gather(*[app.call(f"{app.node_id}.research", question=sub, depth=depth+1,

model=model) for sub in subquestions]). Thread `model` through every call.

Synthesize child answers with a final app.ai. Cap depth at 3. Entry point app.run().

Expected files:

fanout/

main.py # the Agent + research reasoner

requirements.txt # agentfield, pydantic

Dockerfile

Verify: fire the async endpoint, poll for the result.

Verify with curl

Fire the entry reasoner and get an execution id back:

curl -s -X POST http://localhost:8080/api/v1/execute/async/fanout.research \

-H "Content-Type: application/json" \

-d '{"input": {"question": "What made the 2008 financial crisis possible?", "depth": 0}}'

# => {"execution_id": "exec_9f3a...", "status": "queued", ...}

Poll for the result:

curl -s http://localhost:8080/api/v1/executions/exec_9f3a...

# => {"status": "succeeded", "result": {"answer": "...", "children": [...]}, "duration_ms": 41200}

The DAG for that run is one tree in the dashboard. Every app.call is an edge, so you can see which sub-question was slow and which child failed and got retried.

Receipts

For a depth-3 tree with a branching factor of 10, from one request:

About 40 lines of Python for the reasoner. Zero lines for queueing, retries, or tracing.

Roughly 1,100 agent invocations (10 + 100 + 1,000 leaves, plus the splits and syntheses along the way).

A cheap model at a fraction of a cent per call puts a full tree in the low single-digit dollars per run. Point model at something cheaper for the leaves and it drops further.

Control plane routing adds about 100 to 200 ms per level, paid once per level because each level runs concurrently.

This is not a toy ceiling. The deep-research example runs 10,000 or more agent invocations per query using this exact recursion: it fans out searches, filters them, finds gaps, and recurses into the gaps. The pattern holds because the control plane, not your process, owns the queue.

Next step: clone the fanout example, change the split prompt to your domain, and fire one request.

Related

Agents that run for three days, for when the fan-out runs long enough to need async execution and webhooks.

From LangGraph prototype to production, if your fan-out started as a LangGraph graph.

AgentField on GitHub, the deep-research example that runs this recursion at 10,000 calls per query.
