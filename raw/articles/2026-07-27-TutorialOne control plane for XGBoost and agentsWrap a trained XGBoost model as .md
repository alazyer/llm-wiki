---
title: "TutorialOne control plane for XGBoost and agentsWrap a trained XGBoost model as a skill, serve 95% of traffic in milliseconds, and cascade the low-confidence remainder to a reasoner. Same tracing, same DAG, one ops story.Jul 2, 2026 · 27 min"
source_url: "https://agentfield.ai/blog/one-control-plane-for-ml-and-agents"
ingested: 2026-07-27
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/one-control-plane-for-ml-and-agents

## Article Content

One control plane for XGBoost and agents — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

One control plane for XGBoost and agents

Wrap a trained XGBoost model as a skill, serve 95% of traffic in milliseconds, and cascade the low-confidence remainder to a reasoner. Same tracing, same DAG, one ops story.

Santosh Kumar Radha·Co-founder & CTO·

27 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

You have a trained XGBoost or scikit-learn model that scores fraud in a millisecond for a fraction of a cent. It is right most of the time and wrong on the hard 5%, and the hard 5% is where the money is. By the end of this post you have that model serving traffic as a skill, an entry reasoner that trusts it when it is confident, and an LLM judgment call that only fires on the cases the model cannot read.

A confidence cascade is a routing pattern where a cheap deterministic model serves every request, and only predictions below a confidence threshold escalate to a reasoner that can read the surrounding context. The model handles volume. The reasoner handles doubt.

Receipts: about 60 lines across two functions, one XGBoost predict_proba per request plus an occasional app.ai() call, working in an afternoon. The model call costs near zero; the LLM fires on a single-digit percentage of traffic, so your blended cost per request stays close to the model's.

The pattern

A static setup forces one choice for every request. Ship only the model and you are fast and cheap but blind on the edge cases: a chargeback pattern the training data never saw, a transaction whose risk lives in the merchant note, not the numbers. Ship only the LLM and every routine $4.99 subscription renewal pays for a reasoning call it never needed.

The cascade refuses the choice. The model votes first. When its probability lands in a confident band, that vote is the answer and the request is done in milliseconds. When the probability sits in the murky middle, the request escalates to a reasoner that reads the full transaction, the customer history, the free-text fields the model dropped on the floor. Cost tracks difficulty. Easy requests stay cheap; only the genuinely ambiguous ones buy reasoning.

Build it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

1. Wrap the model as a skill

A skill is a deterministic function. No LLM, no reasoning, same input gives the same output. AgentField generates its input and output schema from the type hints, exposes it as a REST endpoint, and records every call in the workflow DAG, exactly like a reasoner, minus the model call.

Load the trained booster once at module level so it lives in memory for the process, then score inside the function.

PythonTypeScriptGo

import os

import joblib

import numpy as np

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="risk-scorer")

# Loaded once per process, not per request.

MODEL = joblib.load(os.getenv("MODEL_PATH", "fraud_xgb.joblib"))

class RiskScore(BaseModel):

score: float # P(fraud), 0.0 - 1.0

confidence: float # distance from the decision boundary, 0.0 - 1.0

model_version: str

@app.skill(tags=["ml", "fraud"])

def score_risk(features: list[float]) -> RiskScore:

"""Deterministic fraud probability from the trained booster."""

proba = float(MODEL.predict_proba([features])[0][1])

# Confident near 0 or 1, unsure near 0.5.

confidence = abs(proba - 0.5) * 2

return RiskScore(

score=proba,

confidence=confidence,

model_version=os.getenv("MODEL_VERSION", "xgb-2026-06"),

if __name__ == "__main__":

app.run(host="0.0.0.0", port=int(os.getenv("PORT", "8010")), auto_port=False)

The skill is now live at POST /api/v1/execute/risk-scorer.score_risk. It behaves like every other agent on the plane: same execute route, same tracing, same DAG node. Nothing about the calling side knows it is a gradient-boosted tree and not a language model.

2. Cascade from the entry reasoner

The entry point is a reasoner. It calls the skill first, reads the confidence, and only reaches for judgment when the model is unsure. The cross-agent call returns a plain object; the .ai() call returns a validated structured result.

PythonTypeScriptGo

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="fraud-gateway")

CONFIDENCE_FLOOR = 0.6 # below this, the model is guessing

class Decision(BaseModel):

verdict: str # "allow" | "review" | "block"

reason: str

decided_by: str # "model" | "reasoner"

@app.reasoner(tags=["entry"])

async def assess(transaction: dict) -> Decision:

# 1. Cheap path: the model votes on every request.

scored = await app.call(

"risk-scorer.score_risk",

features=transaction["features"],

if scored["confidence"] >= CONFIDENCE_FLOOR:

verdict = "block" if scored["score"] >= 0.5 else "allow"

return Decision(

verdict=verdict,

reason=f"Model score {scored['score']:.2f} at confidence {scored['confidence']:.2f}.",

decided_by="model",

# 2. Expensive path: the model is unsure, so read the context it dropped.

return await app.ai(

system=(

"You review transactions the fraud model could not score confidently. "

"Read the full record, including merchant notes and customer history. "

"Return allow, review, or block with a one-line reason."

user=str(transaction),

schema=Decision,

The model runs on 100% of traffic. The reasoner runs only when confidence falls under the floor, which on a well-trained model is the tail, not the body. You tune the floor to trade cost against coverage: raise it and more requests buy reasoning, lower it and the model owns more of the edge.

3. A/B the model against the reasoner

Here is what the shared plane buys you. Both the skill and the reasoner are versioned agents registered the same way, so you can point a slice of traffic at "reasoner decides everything" and compare it against "model decides, reasoner catches the tail" using the same execute routes and the same metrics. No second serving stack, no separate experiment harness. The two candidates are just two agents you route between, and the per-version traffic weight is a REST call on the connector API. The deployment that promotes itself walks that mechanism end to end, including an operator agent that shifts the weights for you.

The DAG makes the comparison legible. Every assess execution records whether it stopped at the model or escalated, so you can read straight off the trace which path each request took and what it cost, then decide whether the floor is set right.

What the control plane does underneath

One execute surface. The XGBoost skill and the LLM reasoner both answer POST /api/v1/execute/{node}.{func}. Callers do not branch on which one they are hitting.

Uniform tracing. Model calls and reasoning calls land in the same workflow DAG as sibling nodes, with the same timing and cost accounting.

Cost attribution per path. Because the reasoner call is a tracked child of assess, you can total what the escalated tail actually costs versus the model-only body.

Version routing. Skills and reasoners both carry a version, so promoting a retrained booster or swapping the reasoner's model is a routing change, not a redeploy of the caller.

Schema at the boundary. The skill's RiskScore and the reasoner's Decision are generated from type hints and validated on the way out, so a malformed prediction fails at the edge, not three hops downstream.

Run it

With the control plane on localhost:8080 and both agents running, confirm the skill registered, then fire one transaction through the gateway.

# The skill shows up under .skills, not .reasoners.

curl -fsS http://localhost:8080/api/v1/discovery/capabilities \

| jq '.capabilities[] | select(.agent_id=="risk-scorer") | .skills[].id'

# Score directly against the model skill.

curl -sS -X POST http://localhost:8080/api/v1/execute/risk-scorer.score_risk \

-H 'Content-Type: application/json' \

-d '{"input": {"features": [0.2, 1.0, 0.05, 3.0, 0.9]}}' \

| jq '.result'

# Run a full transaction through the cascade.

curl -sS -X POST http://localhost:8080/api/v1/execute/fraud-gateway.assess \

-H 'Content-Type: application/json' \

-d '{"input": {"transaction": {"features": [0.2, 1.0, 0.05, 3.0, 0.9], "merchant_note": "first order, gift address"}}}' \

| jq '.result'

A confident transaction comes back with "decided_by": "model" in milliseconds. A borderline one comes back with "decided_by": "reasoner" after an LLM call. Same endpoint, two paths, both on the trace.

Give this to your coding agent

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash, then paste this into an AgentField-aware coding agent (Claude Code, Codex, Cursor) and let it scaffold the cascade.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Your scikit-learn model and your agents now have the same deployment, the same tracing, and the same version routing. MLOps and agent infrastructure stop being two stacks bolted together and become one plane with two kinds of node on it. The next step is to attach a version to the retrained booster and route 10% of score_risk traffic to it, then read the DAG to see whether the new model changed how often the cascade escalates.

The Go and TypeScript SDKs register skills and reasoners through the same surface; see the SDK docs.

Related

250 agents for 90 cents, for routing cheap models under expensive ones by cost rather than confidence.

The deployment that promotes itself, once you want the retrained model to earn its own traffic.
