---
title: "TutorialApprove your agents from your phoneNotify yourself when an agent pauses, then approve or reject it with one tap from a lock screen. A five-line notify plus a twenty-line signed handler turns any pause into a phone gate.Jul 2, 2026 · 36 min"
source_url: "https://agentfield.ai/blog/approve-from-your-phone"
ingested: 2026-07-26
blog: "AgentField"
published: "2026-07-24"
---

## Source Metadata

- **Blog:** AgentField
- **URL:** https://agentfield.ai/blog/approve-from-your-phone
- **Fetched:** 2026-07-26
- **Extracted title:** Approve your agents from your phone

## Article Content

Approve your agents from your phone — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Approve your agents from your phone

Notify yourself when an agent pauses, then approve or reject it with one tap from a lock screen. A five-line notify plus a twenty-line signed handler turns any pause into a phone gate.

Santosh Kumar Radha·Co-founder & CTO·

36 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

This is the safety pattern for the personal stack, the series where every agent registers into the control plane you run yourself. Approving from your phone means an agent that pauses on a risky decision pushes a notification to your lock screen, and you approve or reject it with one tap, from anywhere, without opening a laptop. It is one reusable shape: notify, then pause, then resolve over a signed webhook. The notify is five lines inside the reasoner. The tap target is a twenty-line handler you host that signs one POST. Wire it once and every agent in your stack inherits a phone gate.

The human approval gates post built the pause: app.pause() flips an execution to waiting in the control plane database and blocks until a human resolves it. This post extends that gate to your phone. Two honest gaps sit between "the execution is waiting" and "you tapped approve on the train", and this tutorial fills both with real glue instead of pretending they ship.

The pattern

The pattern is three steps in a fixed order: notify, pause, resolve.

Notify runs inside the reasoner, before the pause blocks. AgentField sends nothing outbound when an execution pauses; it registers the wait and stops. So the reasoner, which is holding the approval link at that moment, pushes the notification itself. Five lines of httpx to a push service does it.

Pause is app.pause(). It moves the execution to waiting and blocks the coroutine until the wait resolves.

Resolve is a signed POST to POST /api/v1/webhooks/approval-response. That endpoint requires an HMAC signature and rejects anything unsigned. There is no built-in approve page in the control plane UI for you to tap from a phone, so the tap target is a tiny handler you host that signs the POST on your behalf. That handler is the twenty lines at the center of this post.

The static alternative is checking a dashboard by hand whenever you remember to. It does not scale past one agent and it fails the moment you are away from your desk. The notify-pause-resolve shape puts the decision on your lock screen and closes the loop with one tap.

What the control plane stores when a reasoner pauses

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

When your reasoner calls app.pause(approval_request_id=..., approval_request_url=..., expires_in_hours=...), the SDK registers a waiter, then tells the control plane to move the execution to waiting. The control plane persists the approval_request_id on the execution row and starts the expires_in_hours clock. The wait is a database row, not a coroutine in memory, so it survives an agent restart. The reasoner blocks on the last line and does nothing else until the wait resolves.

Nothing about that step reaches out to you. That is deliberate, and it is why the notify has to be your code, running one line before the pause.

Build it

1. Notify, then pause (five lines before the block)

The reasoner creates a stable request id, pushes a notification carrying the approve link, then pauses. The notification uses ntfy.sh, a plain-HTTP push service: POST to https://ntfy.sh/<your-topic>, title in a header, message in the body, and a Click header that makes the whole notification tappable. Subscribe your phone to that topic once in the ntfy app.

import os

import httpx

from agentfield import Agent

app = Agent(node_id="assistant")

NTFY_TOPIC = os.environ["NTFY_TOPIC"] # a hard-to-guess string

APPROVE_BASE = os.environ["APPROVE_BASE"] # where your signer is hosted

async def notify_and_pause(request_id: str, what: str) -> "ApprovalResult":

approve_url = f"{APPROVE_BASE}/decide?id={request_id}"

# 1. Notify first, since the pause below sends nothing outbound.

async with httpx.AsyncClient() as http:

await http.post(

f"https://ntfy.sh/{NTFY_TOPIC}",

headers={"Title": "Approval needed", "Click": approve_url},

content=what.encode("utf-8"),

# 2. Now block until you tap approve or reject.

return await app.pause(

approval_request_id=request_id,

approval_request_url=approve_url,

expires_in_hours=24,

Drop that into any reasoner that needs a gate:

@app.reasoner(tags=["entry"])

async def send_money(to: str, amount_usd: float) -> dict:

if amount_usd < 50:

return await do_transfer(to, amount_usd) # small, auto

request_id = f"pay-{to}-{int(amount_usd)}"

outcome = await notify_and_pause(

request_id, f"Send ${amount_usd:.2f} to {to}?"

if outcome.approved:

return await do_transfer(to, amount_usd)

return {"status": outcome.decision, "feedback": outcome.feedback}

app.pause() returns an ApprovalResult. Read outcome.approved for the yes/no, and outcome.decision (approved, rejected, request_changes, or expired) and outcome.feedback for the detail. If the 24 hours lapse with no tap, pause() returns decision="expired" rather than raising, so the else branch already handles it.

2. The signed resolution POST (the part that needs a secret)

The reviewer, meaning you, resolves the wait by POSTing to the control plane. This is the exact wire format, verified against the handler:

curl -sS -X POST http://localhost:8080/api/v1/webhooks/approval-response \

-H 'Content-Type: application/json' \

-H 'X-Webhook-Signature: sha256=<hmac_sha256_of_the_raw_body>' \

-d '{"requestId": "pay-alice-200", "decision": "approved", "feedback": "ok"}'

Three things to get exactly right, because the handler is strict:

The body field is requestId, camelCase. This is the webhook wire name, distinct from the approval_request_id you pass to app.pause() in Python. They carry the same value; only the JSON key differs.

decision is one of approved, rejected, request_changes, expired.

The signature header is X-Webhook-Signature (the handler also accepts X-Hax-Signature or X-Hub-Signature-256). Its value is sha256= followed by the hex HMAC-SHA256 of the raw request body, keyed by your webhook secret.

The endpoint returns 503 if no secret is configured, so this gate only works once you set one:

export AGENTFIELD_APPROVAL_WEBHOOK_SECRET="a-long-random-string"

That is the same secret your signer uses to sign the body. An unsigned or wrongly-signed POST gets 401. There is no unsigned path, and that is the point: the only thing that can resolve an approval is something holding your secret.

3. The twenty-line handler you host

There is no approve page in the control plane UI, so the tappable link in your notification points at a small handler you run. It reads the id, signs the resolution body with your secret, and forwards it to the control plane. Host it anywhere that can reach your control plane: a serverless function, a tiny box on your network, or the same machine the control plane runs on.

import hashlib

import hmac

import json

import os

import httpx

from fastapi import FastAPI

CP = os.environ["AGENTFIELD_URL"] # e.g. http://localhost:8080

SECRET = os.environ["AGENTFIELD_APPROVAL_WEBHOOK_SECRET"].encode()

app = FastAPI()

async def resolve(request_id: str, decision: str) -> int:

body = json.dumps(

{"requestId": request_id, "decision": decision},

separators=(",", ":"),

).encode()

sig = hmac.new(SECRET, body, hashlib.sha256).hexdigest()

async with httpx.AsyncClient() as http:

r = await http.post(

f"{CP}/api/v1/webhooks/approval-response",

headers={

"Content-Type": "application/json",

"X-Webhook-Signature": f"sha256={sig}",

content=body,

return r.status_code

@app.get("/decide")

async def decide(id: str, decision: str = "approved"):

status = await resolve(id, decision)

return {"resolved": id, "decision": decision, "cp_status": status}

Tapping the notification opens …/decide?id=pay-alice-200, which signs {"requestId":"pay-alice-200","decision":"approved"} and forwards it. To make reject a separate tap, put two Actions in the ntfy notification pointing at ?decision=approved and ?decision=rejected. The signing is the whole job: the control plane trusts the POST only because it carries a valid signature over the exact bytes.

One honesty note on the signature. The handler computes the HMAC over the raw body bytes it receives. Your signer must sign the exact same bytes it sends, which is why the code above builds the body string once and both signs and posts that identical value. Do not re-serialize the JSON between signing and sending, or the signature will not match.

What the control plane does underneath

The pause is durable: waiting is a row in the control plane database, so a review that spans your commute survives an agent restart. On recovery the agent re-attaches to the same wait.

The expires_in_hours clock does not count against the reasoner's normal wall-clock budget, so a 24-hour phone gate does not trip a timeout meant for slow model calls.

Resolution is HMAC-gated end to end. The endpoint is on the auth skip-list for API keys but is not open: it verifies the signature on every call and rejects anything unsigned with 401, or 503 if you never set a secret.

When a valid POST lands, the control plane resolves the wait and the reasoner continues from the exact line after pause(). Nothing before the gate re-runs.

Run it

Trigger a decision that lands above the auto-approve line, confirm it is waiting, then resolve it as your phone would.

# 1. Start it async so it can park.

EXEC=$(curl -sS -X POST http://localhost:8080/api/v1/execute/async/assistant.send_money \

-H 'Content-Type: application/json' \

-d '{"input": {"to": "alice", "amount_usd": 200}}' | jq -r '.execution_id')

# 2. Confirm it is waiting on a human.

curl -sS http://localhost:8080/api/v1/executions/$EXEC | jq '.status'

# -> "waiting"

# 3. Resolve it the way your hosted signer does (sign the raw body).

BODY='{"requestId": "pay-alice-200", "decision": "approved", "feedback": "ok"}'

SIG=$(printf '%s' "$BODY" | openssl dgst -sha256 -hmac "$AGENTFIELD_APPROVAL_WEBHOOK_SECRET" | awk '{print $2}')

curl -sS -X POST http://localhost:8080/api/v1/webhooks/approval-response \

-H 'Content-Type: application/json' \

-H "X-Webhook-Signature: sha256=$SIG" \

-d "$BODY"

# 4. The execution resumes and finishes.

curl -sS http://localhost:8080/api/v1/executions/$EXEC | jq '.status'

# -> "succeeded"

The printf '%s' matters: it signs the body without a trailing newline, so the bytes you sign are the bytes you send.

The receipt

Notify glue inside the reasoner: five lines. The hosted signer: about twenty lines of FastAPI. Push services used: one, ntfy.sh, plain HTTP, no account. Secrets to configure: one, the webhook secret, shared by the control plane and your signer. Approve pages AgentField ships for you: none, which is why the twenty-line handler exists. Once it is up, every agent in your stack reuses the same notify_and_pause and the same signer; you write the gate once.

Next step: add two ntfy Actions so approve and reject are separate taps, and set a shorter expires_in_hours on high-stakes gates so an unanswered request routes to a safe default instead of hanging.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Go and TypeScript

This post is Python only because the pattern is glue, not SDK surface. app.pause() is a Python-only blocking primitive, but the durable part lives in the control plane, not the SDK: the POST /api/v1/webhooks/approval-response endpoint and its HMAC contract are identical no matter what language your reasoner is written in. In Go or TypeScript you create the approval request, return, and let the control plane resume the execution when the signed webhook lands, rather than blocking on a one-call pause. The signer is plain HMAC-SHA256 in any language: see the SDK docs.

Related

Your personal control plane, the foundation every gated agent registers into. Read it first.

Human approval gates in 20 lines, the pause and resume this post pushes to your phone. Read it first for the durable-wait mechanics.

A nightly maintenance fleet for your repos, the fleet whose every merge sits behind this exact gate.

Agents that run for three days, the durable async execution the approval wait rides on.
