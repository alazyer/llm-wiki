---
title: "TutorialPipelines that build themselvesAn intake reasoner reads the input, decides which analysis dimensions it actually needs, then the orchestrator spawns exactly those specialists with prompts written at runtime, so the pipeline shape is a trace of what happened rather than a graph declared upfront.Jul 2, 2026 · 30 min"
source_url: "https://agentfield.ai/blog/pipelines-that-build-themselves"
ingested: 2026-07-26
blog: "AgentField"
published: "2026-07-24"
---

## Source Metadata

- **Blog:** AgentField
- **URL:** https://agentfield.ai/blog/pipelines-that-build-themselves
- **Fetched:** 2026-07-26
- **Extracted title:** Pipelines that build themselves

## Article Content

Pipelines that build themselves — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Pipelines that build themselves

An intake reasoner reads the input, decides which analysis dimensions it actually needs, then the orchestrator spawns exactly those specialists with prompts written at runtime, so the pipeline shape is a trace of what happened rather than a graph declared upfront.

Santosh Kumar Radha·Co-founder & CTO·

30 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

You send in one document. A reasoner reads it, decides this particular input needs four analyses and not the other six, writes a specific instruction for each, and dispatches four workers that run at once. A runtime-topology pipeline is one whose shape is chosen by a reasoner after it sees the input, then built by spawning exactly the specialists needed with prompts composed at call time. The graph is not declared before the run. It is a record of the run.

About 90 lines of orchestration Python. A run that fans to four analysts and synthesizes costs a few cents on a cheap model. Working in fifteen minutes. This is the pattern behind the contract analysis and security auditing examples, where the path through the system depends on what the document turns out to contain.

The pattern

A static-graph framework asks you to declare every node and edge before the first input arrives. That works when the path is fixed. It breaks the moment the path depends on the input. A short vendor contract needs a liability pass and nothing else; a fifty-page employment agreement needs IP, non-compete, and termination analysis, plus a cross-reference trace between them. Encode both in one pre-declared DAG and you get a graph that runs every branch for every input, gated by a forest of conditionals, most of them false on any given run.

Runtime topology decides the shape after reading the input. One reasoner classifies the input and returns a plan naming the dimensions this input actually needs. The orchestrator then writes a specific prompt for each planned dimension and dispatches a generic analyst per dimension, in parallel. The DAG that comes out is what happened, not a cage you built in advance.

Build it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

1. Intake reasoner returns a plan, not an answer

Intake is a classifier. It reads the input, decides which analysis dimensions apply, and returns them as structured data the orchestrator can branch on. This is app.ai with a schema, no tools, one shot.

import asyncio

import os

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="pipeline", agentfield_server=os.getenv("AGENTFIELD_SERVER"))

class Dimension(BaseModel):

name: str # e.g. "liability", "ip_assignment", "termination"

why: str # what in the input triggered this dimension

sections: list[str] # where in the input to look

class Plan(BaseModel):

doc_type: str

dimensions: list[Dimension]

confident: bool

@app.reasoner(tags=["entry"])

async def analyze(document: str) -> dict:

plan = await app.ai(

system=(

"Classify this document. Return only the analysis dimensions it "

"actually needs, each with why it applies and which sections to read. "

"Do not include dimensions that do not apply."

user=document[:6000],

schema=Plan,

# ... orchestration continues below

The output is structured JSON on purpose. The orchestrator makes decisions on it (for dim in plan.dimensions), so it flows as data, not prose. A short contract comes back with one dimension; a dense one comes back with five. The pipeline width is now a function of the input.

2. Write each child's prompt at runtime, then fan out

Here is the meta-prompting step. For every dimension the plan named, the orchestrator composes a specific instruction from what intake found and passes it as input to a single generic analyst reasoner. The parent writes the child's job description; the child is one reusable worker.

async def run_dimension(dim: Dimension) -> dict:

prompt = (

f"You are analyzing the '{dim.name}' dimension of a {plan.doc_type}. "

f"This dimension applies because: {dim.why}. "

f"Read these sections: {', '.join(dim.sections)}. "

f"Report concrete findings with a severity of low, medium, high, or critical, "

f"and flag anything that needs a deeper look."

return await app.call(

"pipeline.analyst",

prompt=prompt,

document=document,

dimension=dim.name,

budget=1, # follow-up passes allowed, see step 3

results = await asyncio.gather(*[run_dimension(d) for d in plan.dimensions])

app.call("pipeline.analyst", ...) dispatches through the control plane, so each dimension runs as its own tracked execution. asyncio.gather launches them together. There is one analyst reasoner in the codebase; the orchestrator makes it four different specialists by giving each a different prompt. That is meta-prompting: the parent uses what it discovered to craft what the child does.

The analyst itself is a harness. It navigates the sections it was told to read and returns findings plus any deep-dive requests:

@app.reasoner()

async def analyst(prompt: str, document: str, dimension: str, budget: int = 0) -> dict:

result = await app.harness(

prompt=f"{prompt}\n\n--- DOCUMENT ---\n{document}",

schema=AnalystOutput, # {findings: [...], deep_dives: [...]}

max_turns=8,

max_budget_usd=0.20,

return result.parsed.model_dump() | {"dimension": dimension}

3. Depth-capped follow-ups on a budget counter

An analyst that finds something critical can ask for one deeper pass. The request is bounded by an explicit counter, not by the model's willingness to keep going. This is the inner control loop, and the cap is the whole point.

async def run_dimension(dim: Dimension, budget: int = 1) -> dict:

prompt = _prompt_for(dim, plan)

res = await app.call("pipeline.analyst", prompt=prompt,

document=document, dimension=dim.name, budget=budget)

# The analyst flagged a deeper thread. Spend one unit of budget on it.

for dive in res.get("deep_dives", [])[:budget]:

deeper = await app.call(

"pipeline.analyst",

prompt=(

f"Deeper pass on {dim.name}. The first pass found: {dive['reason']}. "

f"Investigate specifically: {dive['question']}."

document=document,

dimension=dim.name,

budget=0, # no further recursion; budget is spent

res["findings"].extend(deeper.get("findings", []))

return res

budget=1 allows exactly one follow-up. The child pass is dispatched with budget=0, so it cannot spawn another. In contract analysis this is MAX_REF_FOLLOWS and MAX_SUB_AGENTS per cluster: the analyst can follow a reference or spawn a definition-impact sub-agent, but only up to a hard count. Without the counter, an adaptive pipeline finds a reason to go one level deeper on every edge case and the bill has no ceiling.

4. Synthesis merges what actually ran

The final reasoner takes the results from whichever dimensions ran and writes one output. It does not know or care how many there were.

synthesis = await app.ai(

system="Merge these dimension analyses into one risk summary. "

"Rank findings by severity. Cite the dimension each came from.",

user="\n\n".join(

f"## {r['dimension']}\n" + "\n".join(

f["description"] for f in r.get("findings", [])

for r in results

return {

"doc_type": plan.doc_type,

"dimensions_run": [d.name for d in plan.dimensions],

"summary": synthesis,

"findings": [f for r in results for f in r.get("findings", [])],

Note what flows where. Intake to orchestrator is structured JSON, because code branches on it. Orchestrator to analyst is a string prompt, because an LLM reasons over it. Findings to synthesis are strings, because another LLM reads them. The format follows the consumer.

What the control plane does underneath

The orchestrator is plain Python. The control plane turns it into a distributed pipeline:

Routing. Each app.call("pipeline.analyst", ...) is a control-plane request, not a function pointer, so the four analysts can run on four workers, not four coroutines on one process.

Queueing. When the plan names more dimensions than there are free workers, the extras queue and drain as workers free up. Backpressure is automatic.

Tracing. Every dispatched analyst and every follow-up pass is a DAG edge. The run for a five-dimension document and the run for a one-dimension document are different-shaped trees, and both are queryable. The DAG is the trace of the runtime topology.

Retries. A transient failure in one analyst is retried before the orchestrator sees an error, so one flaky pass does not sink the run.

Run it

Fire the entry reasoner over the async endpoint, since a multi-dimension run can exceed the 90-second sync cap:

curl -s -X POST http://localhost:8080/api/v1/execute/async/pipeline.analyze \

-H "Content-Type: application/json" \

-d '{"input": {"document": "MASTER SERVICES AGREEMENT ... (full text) ..."}}'

# => {"execution_id": "exec_a12b...", "status": "queued"}

Poll for the result:

curl -s http://localhost:8080/api/v1/executions/exec_a12b...

# => {"status": "succeeded",

# "result": {"doc_type": "msa", "dimensions_run": ["liability","ip","termination"],

# "summary": "...", "findings": [...]}}

Send a short one-clause document and dimensions_run comes back with one entry. Send a dense one and it comes back with five. Same code, different graph, because the reasoner chose the shape after reading the input.

The Go and TypeScript SDKs expose the same app.call, app.ai, and harness surface, so the orchestrator ports directly. This is written Python-only because the pattern, not the syntax, is the lesson; the contract-af and sec-af examples run it in production.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Next step: clone the two reasoners, run the same document twice with two different classification prompts, and diff the dimensions_run in each result to see the topology change.

Related

Fan out 1,000 parallel agents, the recursive fan-out that each dispatched analyst runs on.

The agent that finds its own tools, when the pipeline should also discover which capabilities exist alongside choosing which dimensions to run.

What is harness orchestration, the essay on why the harness, not the LLM call, is the unit you compose.
