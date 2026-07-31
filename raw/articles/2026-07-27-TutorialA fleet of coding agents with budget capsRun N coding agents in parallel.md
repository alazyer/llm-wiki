---
title: "TutorialA fleet of coding agents with budget capsRun N coding agents in parallel as one governed fleet. Each gets its own checkout, a dollar cap, a turn limit, and a tool allowlist, and you get back one cost table.Jul 2, 2026 · 39 min"
source_url: "https://agentfield.ai/blog/coding-agent-fleet"
ingested: 2026-07-27
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/coding-agent-fleet

## Article Content

A fleet of coding agents with budget caps — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

A fleet of coding agents with budget caps

Run N coding agents in parallel as one governed fleet. Each gets its own checkout, a dollar cap, a turn limit, and a tool allowlist, and you get back one cost table.

Santosh Kumar Radha·Co-founder & CTO·

39 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

You already made one Claude Code call safe to run from a backend: a dollar cap, a turn limit, restricted tools, typed output. The next thing you want is eight of them at once, each on a different checkout, doing the same mechanical migration, and one report at the end telling you what each cost and which ones failed. A harness fleet is a dispatcher that runs N capped app.harness() calls in parallel, each in its own working directory, and aggregates their cost, diffs, and pass/fail into one result. It is about 40 lines. A fleet of 8 mechanical fixes on cheap providers lands in the low single dollars total, and one runaway costs its own cap and nothing more.

The difference between the wave-1 post and this one is the difference between a tool and infrastructure. A tool fixes one bug when you ask it to. Infrastructure fans out across a backlog, enforces a per-worker budget, and hands you a receipt.

The pattern

A harness fleet runs many coding agents concurrently under one governor. Each worker is a fully independent app.harness() call: its own prompt, its own cwd, its own max_budget_usd, its own tool allowlist. The dispatcher gives each a slice of the work and never touches its steps.

The static alternative is a shell loop that runs one coding-agent CLI after another. It works until worker three hangs, worker five silently burns your whole budget, and you have no idea which of the eight touched which files. There is no per-worker cap, no isolation between checkouts, no typed result, and no aggregate cost. You find out the total on your provider invoice.

The fleet fixes all four: isolation is the cwd, the cap is per call, the result is a schema, and the aggregate is one pass over the results.

Build it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

1. Define the work and the per-worker result

The work is a list of identical mechanical tasks. Here it is eight modules that all need the same import rewrite. The result schema is flat, because you want to sort and tabulate it, not read prose.

PythonTypeScriptGo

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="fleet")

class WorkerResult(BaseModel):

module: str

files_changed: list[str]

tests_pass: bool

summary: str

MODULES = [

"billing", "auth", "search", "inbox",

"exports", "webhooks", "reports", "admin",

2. One capped worker per task

Each worker is a single app.harness() call. The important arguments are cwd (its own checkout, so two workers never edit the same file), max_budget_usd (a per-worker cap, not a shared pool), tools (an allowlist), and schema (the flat result). The provider is claude-code here; step 5 mixes providers.

PythonTypeScriptGo

async def migrate_one(module: str) -> WorkerResult:

result = await app.harness(

prompt=(

f"Rewrite the deprecated `oldlib` imports in the {module} module "

f"to the new `corelib` API. Run the module's tests before you finish."

provider="claude-code",

cwd=f"/work/checkouts/{module}",

max_budget_usd=1.50,

max_turns=25,

tools=["Read", "Write", "Bash"],

schema=WorkerResult,

if result.is_error:

# A worker that hit its cap or crashed returns is_error=True.

# Record it as a failure instead of raising, so the fleet survives it.

return WorkerResult(

module=module, files_changed=[],

tests_pass=False, summary=result.error_message or "worker failed",

return result.parsed

3. Dispatch the fleet and aggregate

asyncio.gather runs all workers at once. return_exceptions=True means one worker crashing does not cancel the other seven. Cost aggregation reads result.cost_usd off each harness, which is the real spend, not an estimate. Bound concurrency with a semaphore if you do not want all N in flight at the same time. Cost aggregation is Python and TypeScript only, because the Go harness result has no cost field.

PythonTypeScript

import asyncio

@app.reasoner(tags=["entry"])

async def migrate_fleet(concurrency: int = 4) -> dict:

sem = asyncio.Semaphore(concurrency)

async def bounded(module: str):

async with sem:

return await app.harness(

prompt=(

f"Rewrite the deprecated `oldlib` imports in the {module} "

f"module to `corelib`. Run the module's tests before finishing."

provider="claude-code",

cwd=f"/work/checkouts/{module}",

max_budget_usd=1.50,

max_turns=25,

tools=["Read", "Write", "Bash"],

schema=WorkerResult,

results = await asyncio.gather(

*[bounded(m) for m in MODULES], return_exceptions=True

total_cost = 0.0

rows, failures = [], []

for module, r in zip(MODULES, results):

if isinstance(r, Exception):

failures.append({"module": module, "error": str(r)})

continue

total_cost += r.cost_usd or 0.0

if r.is_error or (r.parsed and not r.parsed.tests_pass):

failures.append({"module": module, "cost_usd": r.cost_usd})

rows.append({

"module": module,

"cost_usd": round(r.cost_usd or 0.0, 4),

"turns": r.num_turns,

"files": r.parsed.files_changed if r.parsed else [],

"tests_pass": bool(r.parsed and r.parsed.tests_pass),

return {

"total_cost_usd": round(total_cost, 4),

"succeeded": len(rows) - len(failures),

"failed": len(failures),

"rows": rows,

"failures": failures,

4. Governance: what happens when one worker overruns

The max_budget_usd=1.50 is enforced per call. When a worker crosses it, that harness stops and returns is_error=True. The other seven keep running. The fleet total is bounded by sum of per-worker caps in the worst case, so eight workers at $1.50 each can never cost more than $12, no matter how badly one of them loops.

Failures are data, not exceptions. A worker that hit its cap, crashed, or came back with tests_pass=False lands in the failures list with its cost attached. You decide the retry policy in code: re-dispatch only the failed modules, optionally with a higher cap or a stronger model, and cap the number of retry rounds so a genuinely stuck module does not retry forever.

async def retry_failures(failures: list[dict], rounds: int = 1) -> list[WorkerResult]:

pending = [f["module"] for f in failures]

for _ in range(rounds):

if not pending:

break

retried = await asyncio.gather(

*[migrate_one(m) for m in pending], return_exceptions=True

pending = [

m for m, r in zip(pending, retried)

if isinstance(r, Exception) or not r.tests_pass

return [] # remaining `pending` are the modules that need a human

5. Mixed providers in one fleet

provider and model are per-call arguments, so one fleet can route hard tasks to claude-code and mechanical ones to a cheaper harness on a cheaper model. Tag each module with a tier and let the dispatcher pick.

PythonTypeScriptGo

TIERS = {

"billing": "hard", "auth": "hard", # gnarly, route to claude-code

"search": "easy", "inbox": "easy", # mechanical, route cheap

"exports": "easy", "webhooks": "easy",

"reports": "easy", "admin": "easy",

def provider_for(module: str) -> dict:

if TIERS[module] == "hard":

return {"provider": "claude-code", "max_budget_usd": 3.0}

return {"provider": "opencode", "model": "minimax/minimax-m2.5",

"max_budget_usd": 0.75}

The prompt, the schema, the cwd, and the tool allowlist stay identical across providers. Only the routing dict changes. The provider CLI must be installed in the environment that runs the reasoner; run af doctor and check recommendation.harness_usable before you ship a fleet that depends on more than one.

What the control plane does underneath

Each app.harness() call is a tracked execution with its own parent/child lineage, so the fleet shows up as one workflow with eight child nodes in the DAG.

Cost, turn count, and duration are recorded per worker, so the aggregate you compute in code matches what the control plane observed.

Concurrency is yours to bound with the semaphore; the control plane queues and routes each call without you managing worker pools.

A worker that hits its budget cap or crashes is recorded as a failed execution, not a lost one, so the failure survives into the DAG and your report.

Run it

curl -X POST http://localhost:8080/api/v1/execute/async/fleet.migrate_fleet \

-H "Content-Type: application/json" \

-d '{"input": {"concurrency": 4}}'

That returns an execution_id. Poll for the aggregate:

curl http://localhost:8080/api/v1/executions/<execution_id>

The result carries total_cost_usd, the per-module rows, and the failures list.

The receipt

A plausible run of the eight-module fleet, four workers concurrent, mechanical modules on a cheap provider and two hard modules on Claude Code:

module provider cost_usd turns tests_pass

billing claude-code 1.42 14 true

auth claude-code 2.10 19 true

search opencode 0.09 6 true

inbox opencode 0.07 5 true

exports opencode 0.11 7 true

webhooks opencode 0.18 9 false ← hit cap, retried

reports opencode 0.08 6 true

admin opencode 0.10 6 true

------------------------------------------------------

total 4.15 72 7/8 passed

Worst case is bounded before the run starts: eight caps at their tier ceilings, $3 + $3 for the hard pair and $0.75 each for the six easy ones, is $10.50. The actual $4.15 is what small, correctly-routed tasks cost. The one failure cost its own cap and nothing more, and the retry policy is one line away.

Next: swap MODULES for your real backlog, set TIERS from what you know about each one, and read total_cost_usd off the first run to calibrate your caps.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Go and TypeScript

The tabs above carry the Go and TypeScript surface inline. Two honest differences to keep in mind: in Go you fan out with goroutines and an errgroup rather than asyncio.gather or Promise.all, and the Go harness.Result exposes NumTurns and duration but not a cost field, so aggregate the fleet cost in Python or TypeScript where result.cost_usd / result.costUsd is populated. See the SDK docs.

Related

Claude Code as a function, the single capped harness call this fleet runs N of. Read it first.

How we ran a 250-agent security audit for 90 cents, the same fan-out-under-a-cap shape applied to security analysis.

What is harness orchestration, why the harness, not the LLM call, is the unit you govern.
