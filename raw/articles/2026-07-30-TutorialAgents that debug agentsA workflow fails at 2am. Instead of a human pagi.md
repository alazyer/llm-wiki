---
title: "TutorialAgents that debug agentsA workflow fails at 2am. Instead of a human paging through logs, an operator agent fetches the failed DAG, diagnoses the root cause, and drafts a fix behind an approval gate while you sleep.Jul 2, 2026 · 36 min"
source_url: "https://agentfield.ai/blog/agents-that-debug-agents"
ingested: 2026-07-30
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/agents-that-debug-agents

## Article Content

Agents that debug agents — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Agents that debug agents

A workflow fails at 2am. Instead of a human paging through logs, an operator agent fetches the failed DAG, diagnoses the root cause, and drafts a fix behind an approval gate while you sleep.

Santosh Kumar Radha·Co-founder & CTO·

36 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

A workflow fails at 2am. The old move is a pager, a human, and twenty minutes of scrolling logs to find which of forty agent hops broke and why. The operator pattern replaces the first responder: an operator reasoner detects the failed run, walks its execution DAG to the broken hop, reasons over the trace to produce a structured diagnosis, and for high-confidence code causes drafts a fix in a branch behind an approval gate. The operator pattern is an agent that investigates other agents' failures using the same control-plane execution history a human would read, then acts within a hard budget and a human gate. It is one reasoner plus one app.ai() diagnosis plus one optional capped app.harness(), around 90 lines, and every run is bounded to one investigation and at most one fix attempt.

An observability tool shows you the failure. An operator agent explains it and drafts the fix. The difference is whether you wake up to a red dot or to a diagnosis with a branch attached.

The pattern

The operator pattern is a reasoner that treats your fleet's execution history as its input. When a run fails, it fetches that run's DAG from the control plane, finds the failed node, pulls the node's error and input, and asks an LLM for a root-cause category, a confidence, and a suggested fix. Low confidence stops at a written diagnosis and a notification. High confidence on a code-level cause optionally commissions one budget-capped harness to attempt a fix in a branch, and nothing merges without a human approval.

The static alternative is an alerting rule that fires a webhook and a human who reads logs. It scales with headcount and it forgets: the same failure mode gets re-diagnosed from scratch every time. The operator writes each diagnosis to memory, so a recurring fingerprint is recognized instead of re-investigated, and the investigation itself is bounded so it can never become the incident.

Build it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

This is an advanced pattern, so the walkthrough is Python. The Go and TypeScript SDKs expose the same app.ai() and app.harness() surface; the failure detection has a Go tab below, since event triggers exist in Go but not TypeScript. The one Python-only piece is app.pause(): in Go or TypeScript you drive the same durable approval wait through the control plane webhook rather than a one-call blocking pause. See the SDK docs.

1. Detection: learn about the failure

There are two honest ways for the operator to learn a run failed. The direct one, verified in the control plane, is a webhook: the control plane emits an execution.failed event, and a triggered reasoner receives the failed execution's id. That is a push, so there is no polling and no lag. Event triggers exist in Python and Go; TypeScript has no trigger surface, so this block is a two-tab.

PythonGo

from agentfield import Agent, EventTrigger

app = Agent(node_id="operator")

@app.reasoner(triggers=[EventTrigger(source="generic_hmac", types=["execution.failed"])])

async def observe_failure(execution_id: str, workflow_id: str) -> dict:

return await investigate(workflow_id, failed_execution_id=execution_id)

If you cannot wire a webhook, the fallback is a scheduled sweep: a ScheduleTrigger reasoner that periodically reads the executions summary and picks up anything with a failed status since the last sweep. The webhook is lower latency; the sweep needs no inbound endpoint. Pick based on whether the control plane can reach your operator node.

2. Investigation: fetch the DAG and find the broken hop

The control plane already recorded the run as a tree of executions. Fetch it with the workflow DAG endpoint, which returns the run status, a nested dag of nodes, and a flat timeline. Each node carries its reasoner_id, agent_node_id, status, and a status_reason, plus its parent link. Walk the timeline for the first node whose status is failed, then pull that node's input and error from its execution detail and logs.

import httpx

SERVER = "http://localhost:8080"

async def investigate(workflow_id: str, failed_execution_id: str = "") -> dict:

async with httpx.AsyncClient(base_url=SERVER, timeout=30) as http:

dag = (await http.get(f"/api/ui/v1/workflows/{workflow_id}/dag")).json()

# The flat timeline is the whole run in order. Find the failed hop.

failed = next(

(n for n in dag["timeline"] if n["status"] == "failed"),

None,

if failed is None:

return {"workflow_id": workflow_id, "note": "no failed node found"}

exec_id = failed["execution_id"]

detail = (await http.get(f"/api/v1/executions/{exec_id}")).json()

logs = (await http.get(f"/api/ui/v1/executions/{exec_id}/logs")).json()

trace = {

"workflow_status": dag["workflow_status"],

"failed_reasoner": failed["reasoner_id"],

"failed_node": failed["agent_node_id"],

"status_reason": failed.get("status_reason"),

"input": detail.get("input"),

"error": detail.get("error") or detail.get("result"),

"logs_tail": logs.get("logs", [])[-40:],

return await diagnose(workflow_id, exec_id, trace)

3. Diagnosis: reason over the trace

The trace is small and bounded, so this is an app.ai() call, not a harness. It reads the failed hop's error, input, and log tail and returns a flat diagnosis: a root-cause category, a confidence, and a suggested fix. A flat schema is the point, because the next step routes on it in code. This part of the operator is the same in all three SDKs: a single structured call with a flat schema and a cheap model.

PythonTypeScriptGo

from pydantic import BaseModel

class Diagnosis(BaseModel):

root_cause: str # e.g. "schema_mismatch", "timeout", "bad_input", "code_bug"

confidence: float # 0.0 to 1.0

summary: str

suggested_fix: str

code_level: bool # is this fixable by editing the failed reasoner's code?

async def diagnose(workflow_id: str, exec_id: str, trace: dict) -> dict:

dx = await app.ai(

system=(

"You are an SRE for an agent fleet. Given one failed execution's "

"error, input, and logs, classify the root cause. Be conservative: "

"if the trace does not clearly show a code bug, set code_level=false "

"and lower your confidence."

user=str(trace),

schema=Diagnosis,

model="minimax/minimax-m2.5",

return await act(workflow_id, exec_id, trace, dx)

4. Action tiers: route on confidence

Low confidence writes the diagnosis to memory and notifies. That is the whole action for the common case, and it is deliberately cheap. High confidence on a code-level cause escalates to one capped harness that attempts a fix in a branch, gated by an approval before anything merges. The tiers are a plain if.

CONFIDENCE_FLOOR = 0.75

async def act(workflow_id: str, exec_id: str, trace: dict, dx: Diagnosis) -> dict:

# Every diagnosis is remembered, so recurrence is recognized, not re-investigated.

fingerprint = f"{trace['failed_reasoner']}:{dx.root_cause}"

await app.memory.set(f"diagnoses.{fingerprint}", {

"summary": dx.summary, "confidence": dx.confidence,

"last_seen_execution": exec_id,

if dx.confidence < CONFIDENCE_FLOOR or not dx.code_level:

# Tier 1: write and notify. No code is touched.

return {"tier": "diagnosis_only", "diagnosis": dx.model_dump()}

# Tier 2: one budget-capped fix attempt in a branch, behind an approval gate.

return await draft_fix(workflow_id, exec_id, trace, dx)

The fix attempt is a single app.harness() on a branch, capped hard, and the harness runs before the approval so a reviewer sees a real diff, not a promise. app.pause() moves the execution to a durable waiting state in the control plane, so the branch sits open until a human decides. Only an approval merges.

class FixAttempt(BaseModel):

branch: str

files_changed: list[str]

summary: str

tests_pass: bool

async def draft_fix(workflow_id: str, exec_id: str, trace: dict, dx: Diagnosis) -> dict:

fix = await app.harness(

prompt=(

f"A production reasoner `{trace['failed_reasoner']}` failed.\n"

f"Diagnosis: {dx.summary}\nSuggested fix: {dx.suggested_fix}\n"

f"Create a branch, apply the smallest fix, add a regression test, "

f"and run the suite. Do not merge."

provider="claude-code",

cwd="/work/repo",

max_budget_usd=3.0,

max_turns=30,

tools=["Read", "Write", "Bash"],

schema=FixAttempt,

if fix.is_error or fix.parsed is None:

return {"tier": "fix_failed", "error": fix.error_message,

"cost_usd": fix.cost_usd}

request_id = f"fix-{exec_id}"

approval = await app.pause(

approval_request_id=request_id,

approval_request_url=f"https://review.acme.com/fixes/{request_id}",

expires_in_hours=24,

return {

"tier": "fix_drafted",

"branch": fix.parsed.branch,

"cost_usd": fix.cost_usd,

"approved": approval.approved,

"diagnosis": dx.model_dump(),

5. The loop is bounded

Every part of this has a ceiling. One investigation per failure. One app.ai() diagnosis. At most one app.harness() fix attempt, capped at three dollars. One approval gate that nothing crosses without a human. The operator can never turn a single failure into a runaway spend or a merge you did not see, because there is no loop that adapts its own depth. If the fix is rejected, the operator records that and stops. It does not try again on its own.

What the control plane does underneath

The failed run is already a persisted DAG of executions with per-hop status, so the operator reads history instead of reconstructing it from log lines.

execution.failed is a real control-plane event you can trigger a reasoner on, so detection is a push, not a polling loop you maintain.

app.pause() parks the fix behind a durable waiting row, so the branch survives an operator restart and the reviewer's clock is the control plane's, not a coroutine in memory.

The harness fix attempt is itself a tracked, capped execution, so the operator's own action shows up in the DAG with its cost and turn count.

Run it

Fetch the DAG for a failed run to see what the operator reads:

curl http://localhost:8080/api/ui/v1/workflows/<workflow_id>/dag

Trigger an investigation manually:

curl -X POST http://localhost:8080/api/v1/execute/async/operator.observe_failure \

-H "Content-Type: application/json" \

-d '{"input": {"execution_id": "exec_abc123", "workflow_id": "wf_abc123"}}'

Poll the result for the tier and diagnosis:

curl http://localhost:8080/api/v1/executions/<execution_id>

The receipt

A tier-1 run, the common case, is one DAG fetch, one execution detail, one log pull, and one app.ai() call over a bounded trace: cents, seconds, no code touched. A tier-2 run adds one capped harness attempt at up to three dollars and one approval gate. The forkable production version of this pattern is agent-observability-af, which classifies failures against a taxonomy, verifies every harness claim against control-plane evidence before writing memory, and keeps a per-reasoner improvement backlog. It is the shape teams start from when they want an operator node they can trust with real incidents.

Next: point the operator at one recurring failure in your own fleet, set CONFIDENCE_FLOOR where you are comfortable letting it draft a branch, and read the first few tier-1 diagnoses before you enable tier 2.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Related

The deployment that promotes itself, the sibling operator that watches per-version metrics and shifts traffic itself.

Durable human approval in 20 lines, the app.pause() gate this operator puts in front of every merge.

A fleet of coding agents with budget caps, the capped harness dispatch the fix tier reuses.
