---
title: "TutorialAgents that run for three daysFire an agent with one POST, close the connection, and get a signed webhook when it finishes hours or days later, backed by a durable PostgreSQL queue with retries.Jul 2, 2026 · 30 min"
source_url: "https://agentfield.ai/blog/agents-that-run-for-days"
ingested: 2026-07-29
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/agents-that-run-for-days

## Article Content

Agents that run for three days — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Agents that run for three days

Fire an agent with one POST, close the connection, and get a signed webhook when it finishes hours or days later, backed by a durable PostgreSQL queue with retries.

Santosh Kumar Radha·Co-founder & CTO·

30 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

You POST once, get a 202 back, and close the connection. Three days later a signed webhook lands on your server with the result. In between, the agent ran through worker restarts, retried the calls that failed, and never hit a request timeout, because there was no request to time out.

Here is the artifact. A long-running reasoner grinds through a large job, writing progress to memory and the trace as it goes. You fire it with curl, walk away, and a webhook receiver picks up the completion whenever it comes. The whole fire-and-forget contract is one HTTP call and one small handler.

The Python hero

A reasoner that processes a big list of items, checkpoints its progress, and returns when done. Nothing about it assumes a caller is still holding a socket.

import asyncio

import os

from agentfield import Agent

from agentfield.logger import log_info

app = Agent(node_id="batch", agentfield_server=os.getenv("AGENTFIELD_SERVER"))

@app.reasoner(tags=["entry"])

async def process_corpus(items: list[str], model: str | None = None) -> dict:

total = len(items)

done = await app.memory.get("done", default=0)

results: list[dict] = await app.memory.get("results", default=[])

# Resume from wherever the last run left off.

for i in range(done, total):

summary = await app.ai(

system="Summarize this document in three sentences.",

user=items[i],

model=model,

results.append({"item": i, "summary": summary})

# Checkpoint every step so a crash loses at most one item.

await app.memory.set("results", results)

await app.memory.set("done", i + 1)

log_info(f"processed {i + 1}/{total}")

await asyncio.sleep(0) # yield; real work goes here

return {"processed": total, "results": results}

if __name__ == "__main__":

app.run()

The progress signal is log_info plus a memory checkpoint. Every log_info line lands in the execution trace, and every app.memory.set(...) write is a durable checkpoint scoped to this run (memory auto-scopes to the workflow inside a reasoner). If a worker dies at item 4,000 of 10,000, the next run reads done from memory and starts at 4,000. At-most-one-item loss, no bespoke resume logic.

There is no timeout on this. A synchronous HTTP handler dies when a proxy cuts the connection at 30 or 60 seconds. Async execution has no such limit. The reasoner runs until it returns.

Fire and forget

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

Instead of POST /api/v1/execute, you POST to the async endpoint. It enqueues the work and returns immediately with a 202.

curl -s -X POST http://localhost:8080/api/v1/execute/async/batch.process_corpus \

-H "Content-Type: application/json" \

-d '{

"input": {"items": ["doc one text", "doc two text", "..."]},

"webhook": {

"url": "https://your-app.com/hooks/agentfield",

"secret": "whsec_your_signing_secret"

# => {"execution_id": "exec_7b21...", "status": "queued", "webhook_registered": true}

Three fields carry the whole contract. input is the reasoner's arguments. webhook.url is where the control plane POSTs the result. webhook.secret is the HMAC key it signs each delivery with. Once you have the 202, you can close the connection and shut down the client. The job lives in the queue now, not in your process.

The webhook receiver

When the reasoner returns, the control plane POSTs the result to your URL and signs the raw body with HMAC-SHA256. The signature arrives in the X-AgentField-Signature header, formatted as sha256=<hex digest>. You recompute it over the raw bytes and compare in constant time. This is plain Python, framework-agnostic.

import hashlib

import hmac

SECRET = b"whsec_your_signing_secret"

def verify(raw_body: bytes, signature_header: str) -> bool:

expected = "sha256=" + hmac.new(SECRET, raw_body, hashlib.sha256).hexdigest()

return hmac.compare_digest(expected, signature_header)

# In any handler that gives you the raw body and headers:

def handle(raw_body: bytes, headers: dict) -> tuple[int, str]:

sig = headers.get("X-AgentField-Signature", "")

if not verify(raw_body, sig):

return 401, "bad signature"

import json

payload = json.loads(raw_body)

if payload["event"] == "execution.completed":

result = payload["result"]

# ... store it, notify a user, kick off the next step

elif payload["event"] == "execution.failed":

err = payload.get("error_message")

# ... alert, requeue, whatever your ops need

return 200, "ok"

Two things make this safe. Verify against the raw body, not a re-serialized dict, because re-encoding changes the bytes and breaks the signature. Use hmac.compare_digest, not ==, so the comparison does not leak timing. The payload carries event (execution.completed or execution.failed), execution_id, status, result, and error_message when it failed.

Retries and durability

The webhook is not a single best-effort POST. If your receiver is down or returns a 5xx, the control plane retries with exponential backoff, up to 5 attempts:

attempt 1 immediate

attempt 2 +5s

attempt 3 +10s

attempt 4 +20s

attempt 5 +40s (backoff caps at 5 minutes)

The delivery queue is PostgreSQL-backed. Pending webhooks survive a control plane restart, because on warm start the dispatcher scans the database for deliveries that are due and picks them back up. The execution itself is durable the same way: work is leased to a worker, and if that worker dies the lease expires and the job becomes available again. A crash is a delay, not a lost job.

Because deliveries retry, your receiver must be idempotent. Key on execution_id and treat a second delivery of the same id as a no-op.

Watching progress with SSE
