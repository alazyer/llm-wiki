---
title: "TutorialHow we ran a 250-agent security audit for 90 centsA full security audit that runs 166 to 255 agent calls and confirms 28 exploitable findings costs between 18 and 90 cents, because each call is small, routed to the right model, and capped.Jul 2, 2026 · 24 min"
source_url: "https://agentfield.ai/blog/250-agents-90-cents"
ingested: 2026-07-27
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/250-agents-90-cents

## Article Content

How we ran a 250-agent security audit for 90 cents — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

How we ran a 250-agent security audit for 90 cents

A full security audit that runs 166 to 255 agent calls and confirms 28 exploitable findings costs between 18 and 90 cents, because each call is small, routed to the right model, and capped.

Santosh Kumar Radha·Co-founder & CTO·

24 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

The obvious way to audit a codebase with an LLM is one big call: dump the repo into context, ask "find the vulnerabilities," read the answer. It is cheap to write and expensive to trust. A single pass over a large context hallucinates, misses cross-file data flows, and gives you a wall of maybes with no way to tell signal from noise.

SEC-AF, our open-source security auditor, does the opposite. One audit of the Damn Vulnerable GraphQL Application runs 166 to 255 focused agent calls across 82 DAG edges. It discovers 106 raw findings, dedupes to 61, and after adversarial verification confirms 28 exploitable ones. That is 94 percent noise reduction. The whole run costs between $0.18 and $0.90.

Here is how the number stays that low.

Small calls beat one big call, on cost and on quality

The 250-call design is why the cost stays low, not a tax you pay for splitting the work up.

A monolithic prompt pays for the entire repo context on every reasoning step, and it reasons worse because the model drowns in irrelevant code. SEC-AF gives each agent a narrow task and only the context that task needs. The injection hunter sees data-flow maps and input entry points. The crypto hunter sees dependency trees and key management. A verifier sees a projected view of one finding with only the fields it needs to judge it. Nobody pays for context they never read, and nobody hallucinates over code they never saw.

The pipeline compresses signal at every stage: 106 raw findings, 61 after semantic dedup, 28 confirmed after the prove phase. Each stage is a different kind of reasoning acting as a filter. You are not paying 250 times for the same work. You are paying once each for 250 different small jobs, and the cheap jobs vastly outnumber the expensive ones.

Route each call to the model that fits it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

Not every agent needs the strongest model. Classification and routing gates can run on a cheap model; the calls that weigh conflicting evidence and decide exploitability get a stronger one. AgentField makes model a per-call parameter, so the routing lives in code, not in a global setting.

PythonTypeScriptGo

@app.reasoner(tags=["entry"])

async def audit(repo_url: str, model: str | None = None) -> dict:

# cheap model for the routing gate: which hunters to run

plan = await app.ai(

system="Select hunt strategies for this stack.",

user=recon_summary,

schema=HuntPlan,

model="minimax/minimax-m2.5",

# thread the strong model down into each hunter call

findings = await asyncio.gather(*[

app.call(f"{app.node_id}.hunt", strategy=s, repo_url=repo_url, model=model)

for s in plan.strategies

Two things to notice. First, app.ai(model=...) overrides the model for that single call, so the gate runs cheap while the judgment calls run strong. Second, app.call() has no native model override, so you thread model as a keyword argument and the receiving reasoner passes it into its own app.ai() and app.harness() calls. SEC-AF reads its defaults from HARNESS_MODEL and AI_MODEL (Kimi K2.5 via OpenRouter in the benchmark), and any per-request override flows through the same channel.

The benchmark ran on OpenRouter pricing of $0.22 per million input tokens and $0.88 per million output tokens. At those rates, 250 small calls with pruned context add up to cents, not dollars. Swap to a pricier model and the same architecture still holds; only the multiplier changes.

The ai vs harness cost gap

SEC-AF uses two primitives with very different price tags. app.ai() is a single-shot structured call: input in, flat schema out, no repo browsing. app.harness() runs a real coding agent that reads files and traces code over many turns. A harness call costs an order of magnitude more than an .ai() call, so the pipeline uses harness only where navigation is actually required (hunting through source, tracing a data flow) and uses .ai() for everything that is classification, dedup, or routing.

Getting this split right is most of the cost control. Every finding that a cheap .ai() gate can reject never reaches an expensive harness verification. The gates are the point.

Cap the budget at every loop

A pipeline that adapts at runtime can also spend at runtime, so every loop has a hard ceiling. SEC-AF splits its budget by phase and bounds concurrency so the worst case is knowable:

class BudgetConfig(BaseModel):

max_cost_usd: float | None = None # global ceiling per audit

recon_budget_pct: float = 0.10

hunt_budget_pct: float = 0.45

prove_budget_pct: float = 0.45

max_concurrent_hunters: int = 4

max_concurrent_provers: int = 3

hunter_early_stop_file_threshold: int = 30

The audit input carries a max_cost_usd that the orchestrator enforces across the whole run. Each phase gets a slice: recon is cheap (10 percent), hunt and prove split the rest evenly. Concurrency is capped so you never fan out into a thousand simultaneous calls by accident. And hunters stop early: if one inspects 30 files without a credible lead, it returns empty instead of grinding through the whole tree. Without those caps, an adaptive system quietly becomes an unbounded one.

Where the money actually went

The budget splits roughly in half between the two phases that do real work.

Hunt runs 11 strategy hunters, each a coding agent navigating the source with an early-stop rule. This is where harness calls happen, so it carries real per-call cost, but each hunter is bounded by its file threshold and the concurrency cap.

Prove is the adversarial phase and it is where the noise reduction is earned. Each surviving finding goes through a four-agent chain: a tracer reconstructs the data flow, a sanitization analyzer looks for mitigations the hunter missed, an exploit hypothesizer builds a concrete attack, and a verdict agent weighs the conflicting evidence. Four calls per finding sounds expensive, but they run only on the 61 findings that survived dedup, and most of them are .ai() calls over a pruned view, not full harness sessions. The chain is what turns 61 maybes into 28 confirmed and 2 correctly rejected.

Recon is the cheap 10 percent up front: map the architecture, dependencies, and config so every downstream agent gets pruned context instead of the raw repo.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Build an AgentField reasoner audit(repo_url, model) that (1) runs a recon .ai() gate on a cheap model to pick hunt strategies, (2) fans out hunters via app.call(..., model=model) with a semaphore cap of 4, (3) dedupes findings with an .ai() semantic pass, (4) runs a 4-agent prove chain per surviving finding with a cap of 3, and (5) enforces a max_cost_usd ceiling split 10/45/45 across recon, hunt, prove. Thread model through every app.call as a kwarg since app.call has no native model override.

Verify it registered and run it:

curl -X POST http://localhost:8080/api/v1/execute/async/sec-af.audit \

-H "Content-Type: application/json" \

-d '{"input": {"repo_url": "https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application"}}'

The receipt

From the SEC-AF benchmark against DVGA, standard depth:

Raw findings discovered 106

After AI deduplication 61

After adversarial verification 28 confirmed

Noise reduction 94%

Agent calls ~166 to 255

DAG edges 82

Strategies run 11

Wall-clock time ~78 min

Estimated cost (Kimi K2.5) ~$0.18 to $0.90

The dollar figure is honest because the architecture makes it honest: many small calls, cheap models on the gates, strong models only on judgment, harness only where navigation is required, and a hard cap on every loop.

Next: run it against your own repo with a max_cost_usd you are comfortable with, and read result.cost_usd to see where your run landed.

Related

Claude Code as a function, the single capped harness call this audit fans out hundreds of.

Fan out 1,000 parallel agents from one request, the parallel recursion behind the multi-agent shape.
