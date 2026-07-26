---
title: "TutorialHuman approval gates in 20 linesSuspend a running agent on a low-confidence decision, notify a reviewer, and resume exactly where it stopped. The wait state lives in the control plane, so it survives restarts.Jul 2, 2026 · 30 min"
source_url: "https://agentfield.ai/blog/human-approval-20-lines"
ingested: 2026-07-26
blog: "AgentField"
published: "2026-07-24"
---

## Source Metadata

- **Blog:** AgentField
- **URL:** https://agentfield.ai/blog/human-approval-20-lines
- **Fetched:** 2026-07-26
- **Extracted title:** Human approval gates in 20 lines

## Article Content

Human approval gates in 20 lines — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Human approval gates in 20 lines

Suspend a running agent on a low-confidence decision, notify a reviewer, and resume exactly where it stopped. The wait state lives in the control plane, so it survives restarts.

Santosh Kumar Radha·Co-founder & CTO·

30 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

Your agent is about to issue a $4,000 refund. It is 82 percent sure that is the right call. You want a human to look before the money moves.

The naive version keeps a Python coroutine parked on an asyncio.Event while it waits for someone to click approve. If the agent process restarts, that wait is gone, and so is the in-flight decision.

app.pause() does not park in process memory. It flips the execution to a waiting state in the control plane database and blocks. If the agent restarts while a reviewer is still deciding, the execution is still waiting on the control plane, and the agent can pick the wait back up. That is what "durable" means here: the pending state is a row in a database, not a coroutine in RAM.

The reasoner

The full gate. A decision, a confidence check, and a suspend that only resumes when a human answers.

import os

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="refund-agent")

class RefundDecision(BaseModel):

approve: bool

amount_usd: float

reason: str

confidence: float # 0.0 - 1.0

@app.reasoner(tags=["entry"])

async def decide_refund(ticket: str, model: str | None = None) -> dict:

decision = await app.ai(

system="Decide whether to refund. Return approve, amount_usd, reason, confidence.",

user=ticket,

schema=RefundDecision,

model=model,

if decision.confidence >= 0.9:

return {"status": "auto_approved", "decision": decision.model_dump()}

# Low confidence: create the review on your reviewer service, then suspend.

request_id = await create_review(decision) # your code: writes to your DB/queue

result = await app.pause(

approval_request_id=request_id,

approval_request_url=f"https://review.acme.com/refunds/{request_id}",

expires_in_hours=48,

if result.approved:

return {"status": "approved", "feedback": result.feedback,

"decision": decision.model_dump()}

return {"status": result.decision, "feedback": result.feedback}

if __name__ == "__main__":

app.run(host="0.0.0.0", port=int(os.getenv("PORT", "8001")), auto_port=False)

That is the gate, around 20 lines of logic. The mechanics are worth walking through.

What each argument does

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

app.pause() takes three things you control:

approval_request_id: the id of the review record on your own service. You create the review first (a row, a queue message, a Slack thread), then hand its id to pause(). AgentField does not own the review UI; it owns the suspend and the resume. This is the Python SDK argument name; when you resolve the same wait over REST the JSON field is requestId, carrying the identical value.

approval_request_url: where a human goes to decide. AgentField stores it and surfaces it in the dashboard so a reviewer has one click to the review.

expires_in_hours: how long the execution may stay parked before the control plane gives up. Default is 72. When it lapses, pause() returns an ApprovalResult with decision="expired" instead of raising, so the expired branch is code you write, not an exception you catch.

The call blocks until one of two things happens: a reviewer resolves the request, or the timer runs out.

How suspend and resume actually work

When the reasoner hits await app.pause(...), three things happen in order.

First, the SDK registers a future keyed by approval_request_id before it tells the control plane anything, so a fast callback cannot race past an unregistered waiter.

Second, it calls the control plane to move the execution to waiting. From this point the wait is durable: the pending state is persisted, not held in the coroutine. The reviewer's clock is running against expires_in_hours, and that clock does not count against the reasoner's normal wall-clock budget, so a 48-hour review does not trip a timeout meant for slow LLM calls.

Third, the reasoner blocks on the future.

When the reviewer decides, your service (or the dashboard) sends the resolution to the control plane, which calls the agent's approval webhook. That resolves the future, and the reasoner continues from the exact line after pause(), with the human's decision and feedback in hand. Nothing re-runs. The refund logic that came before the gate does not execute twice.

If the agent restarted while the reviewer was still deciding, the execution row is still waiting. On recovery the agent re-registers the wait with wait_for_resume(approval_request_id=...), which listens for the same callback without asking the control plane to transition again. The decision survives the crash because it was never in the process to begin with.

Approving over REST

The reviewer service resolves the request by posting the decision to the control plane webhook. The endpoint is HMAC-gated, so the POST carries a signature header alongside the body.

BODY='{"requestId": "req-abc-123", "decision": "approved", "feedback": "Verified the double charge in Stripe. Refund is correct."}'

SIG=$(printf '%s' "$BODY" | openssl dgst -sha256 -hmac "$AGENTFIELD_APPROVAL_WEBHOOK_SECRET" | awk '{print $2}')

curl -sS -X POST http://localhost:8080/api/v1/webhooks/approval-response \

-H 'Content-Type: application/json' \

-H "X-Webhook-Signature: sha256=$SIG" \

-d "$BODY"

The body field is requestId, camelCase. That is the webhook wire name, distinct from the approval_request_id you pass to app.pause() in Python; they carry the same value, only the JSON key differs. decision is one of approved, rejected, or request_changes. As soon as the control plane processes it, the paused decide_refund execution unblocks and returns. The reviewer never touches the agent directly; they touch the control plane, which owns the resume.

The endpoint is HMAC-only. The signature is sha256= followed by the hex HMAC-SHA256 of the raw request body, keyed by the approval webhook secret in your control plane config, and it goes in the X-Webhook-Signature header. The control plane returns 503 if no secret is set, so this gate only works once you configure one, and 401 on a missing or wrong signature. Sign the exact bytes you send; the printf '%s' above signs the body without a trailing newline so the signed bytes match the sent bytes.

The dashboard approval UI

You do not have to build a review screen. Every execution in waiting shows up in the AgentField dashboard with its approval_request_url, so a reviewer can open the request, read the agent's reasoning, and approve or reject from there. The webhook above is what the dashboard button calls under the hood, so REST and UI resolve the same wait through the same path.

Go and TypeScript

The suspend/resume contract is the same across SDKs because the durable state lives in the control plane, not the SDK. The one Python-only piece is the blocking app.pause() call itself. In Go and TypeScript there is no one-call blocking pause; instead the reasoner creates the approval request, returns, and the control plane resumes the execution when the decision arrives. In every language, resolution lands at POST /api/v1/webhooks/approval-response with { requestId, decision, feedback } and an X-Webhook-Signature header, and the waiting execution continues from there. The REST surface is the durable part; the SDK is a thin waiter over it.

Paste this into /agentfield

Install the CLI with curl -fsSL https://agentfield.ai/install.sh | bash, then hand this to an AgentField-aware coding agent to scaffold the gate. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Build an AgentField agent "refund-agent" with an entry reasoner

`decide_refund(ticket: str, model: str | None = None) -> dict`. Use app.ai with

schema RefundDecision (approve: bool, amount_usd: float, reason: str,

confidence: float). If confidence >= 0.9 auto-approve. Otherwise create a review

record (stub `create_review`), then call

`await app.pause(approval_request_id=..., approval_request_url=..., expires_in_hours=48)`.

Return status based on result.approved / result.decision. Handle the expired branch.

Expected file tree:

refund-agent/

main.py # Agent + decide_refund reasoner with app.pause()

requirements.txt # agentfield, pydantic

Dockerfile

README.md

Verify it

Kick off a decision with a ticket that lands below the auto-approve line, watch it suspend, then resolve it.

# 1. Start the decision async so it can park.

EXEC=$(curl -sS -X POST http://localhost:8080/api/v1/execute/async/refund-agent.decide_refund \

-H 'Content-Type: application/json' \

-d '{"input": {"ticket": "Please refund my annual plan, I was billed twice."}}' \

| jq -r '.execution_id')

# 2. Confirm it is waiting on a human.

curl -sS http://localhost:8080/api/v1/executions/$EXEC | jq '.status'

# -> "waiting"

# 3. Approve it (requestId is the value your create_review returned; sign the raw body).

BODY='{"requestId": "req-abc-123", "decision": "approved", "feedback": "confirmed"}'

SIG=$(printf '%s' "$BODY" | openssl dgst -sha256 -hmac "$AGENTFIELD_APPROVAL_WEBHOOK_SECRET" | awk '{print $2}')

curl -sS -X POST http://localhost:8080/api/v1/webhooks/approval-response \

-H 'Content-Type: application/json' \

-H "X-Webhook-Signature: sha256=$SIG" \

-d "$BODY"

# 4. The execution resumes and finishes.

curl -sS http://localhost:8080/api/v1/executions/$EXEC | jq '.status, .result'

# -> "succeeded"

Receipts

Gate logic: about 20 lines. Storage the app has to manage for the wait: none, because the waiting state is a row in the control plane database. Executions lost to an agent restart mid-review: zero, because the wait was never in process memory. Resolution surface: one webhook, POST /api/v1/webhooks/approval-response, shared by the REST call and the dashboard button.

Next step: set a shorter expires_in_hours and handle the expired branch, so a review nobody answers routes to a safe default instead of hanging forever.

Related

Approve your agents from your phone, the full notify-pause-resolve flow that pushes this gate to your lock screen with the signed webhook and a hosted signer.

Agents that run for three days, the durable async execution the approval wait rides on.

Add an agent mesh to your existing FastAPI or Next.js app, for wiring the gated reasoner into an app you already run.
