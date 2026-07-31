---
title: "The Move from Monolithic AI to Modular SystemsWhat a documentation chatbot taught us about building AI features that scale. When web applications hit complexity, we extracted microservices. The same evolution is happening with AI.Dec 11, 2024 · 36 min"
source_url: "https://agentfield.ai/blog/microservices-moment-for-ai"
ingested: 2026-07-29
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/microservices-moment-for-ai

## Article Content

The Move from Monolithic AI to Modular Systems — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · December 11, 2024

The Move from Monolithic AI to Modular Systems

What a documentation chatbot taught us about building AI features that scale. When web applications hit complexity, we extracted microservices. The same evolution is happening with AI.

Santosh Kumar Radha·Co-founder & CTO·

36 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

When web applications hit complexity, we didn't keep everything in one process. We extracted microservices.

AI is hitting that same inflection point. Most teams haven't noticed yet.

AI Sprawl

A documentation chatbot gets built. It works. Customers love it.

Six months later, reasoning has spread through the organization. Support wants sentiment analysis on tickets. Marketing wants content moderation. Growth wants intelligent onboarding. The security team wants fraud signals. Everyone wants what the chatbot proved was possible.

The chatbot was never the destination. It was the existence proof.

Each capability becomes its own project—because there's no shared substrate for reasoning to live on:

The chatbot: Pinecone, FastAPI, Railway

The sentiment analyzer: Qdrant, separate codebase, separate CI/CD

The content moderator: yet another vector store, another queue, another set of secrets to rotate

Three independent AI systems. Separate infrastructure, separate deployment pipelines, separate on-call rotations. Separate ways to fail at 3am.

The sprawl looks like an organizational failure, but it's structural. Each AI capability carries its own assumptions about execution: the chatbot expects synchronous question-answer flows, the sentiment analyzer expects batch ingestion, the moderator expects streaming input. They can't share infrastructure because they don't share a model for how reasoning runs.

Without a common substrate, every AI feature reinvents the stack.

There's a harder question underneath the infrastructure problem, one that becomes urgent once reasoning spreads beyond documentation search:

Who made this decision? Can you explain it? Replay it? Bound it?

Scattered systems can't answer these questions. When reasoning lives in silos, accountability becomes forensics—digging through five different logging systems to reconstruct what happened. You can't audit what you can't trace.

The better question—the one worth asking before the first chatbot ships: What substrate should reasoning live on?

A Documentation Chatbot (The Case Study)

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

We built a documentation chatbot for Agentfield. What emerged was a set of insights about AI architecture.

The bootstrap is small:

main.py

SourceCopy

from agentfield import Agent, AIConfig

app = Agent(

node_id="documentation-chatbot",

agentfield_server=f"{os.getenv('AGENTFIELD_SERVER')}",

api_key=os.getenv("AGENTFIELD_API_KEY"),

ai_config=AIConfig(

model=os.getenv("AI_MODEL", "openrouter/openai/gpt-4o-mini"),

for router in (

query_router,

ingestion_router,

retrieval_router,

qa_router,

app.include_router(router)

No vector database setup, no queue configuration, no custom deployment scripts.

The chatbot connects to a control plane that handles orchestration, memory, logging, and deployment infrastructure.

This chatbot is an agent node, not a standalone application. Agent nodes are how AI capabilities should be built.

Four Lessons in AI Architecture

Building this chatbot surfaced four insights that most teams discover too late.

Async-First Execution

AI calls are non-deterministic. A simple question might take 200ms. A complex one might trigger multiple retrievals, self-assessment, and refinement, easily stretching to 30 seconds or more.

Traditional request-response patterns break under this variance. HTTP timeouts kill long-running chains, users stare at spinners, and systems fail without warning.

Our QA orchestrator handles this naturally:

qa_router.py

SourceCopy

@qa_router.reasoner()

async def qa_answer(question: str, namespace: str = "website-docs") -> DocAnswer:

# Step 1: Plan diverse search queries

plan = await plan_queries(question)

# Step 2: Parallel retrieval (concurrent, not sequential)

results = await parallel_retrieve(queries=plan.queries, namespace=namespace)

# Step 3: Synthesize with self-assessment

answer = await synthesize_answer(question, results)

# Step 4: Automatic refinement if needed

if answer.needs_more and answer.missing_topics:

additional = await parallel_retrieve(queries=refinement_queries, ...)

answer = await synthesize_answer(question, merged_results, is_refinement=True)

return answer

Three AI calls minimum, five when refinement is needed. If step 3 fails, you know exactly what happened in steps 1 and 2: the infrastructure logs everything automatically.

Automatic Observability

Every execution is tracked, every AI call logged, every response stored. The control plane handles instrumentation automatically:

Terminal

Copy

# Query execution history for one agent + workflow

curl "http://localhost:8080/api/ui/v1/agents/documentation-chatbot/executions?workflowId=run_qa_session_001"

Response

Copy

"executions": [

"execution_id": "exec_abc",

"workflow_id": "run_qa_session_001",

"agent_node_id": "documentation-chatbot",

"reasoner_id": "plan_queries",

"status": "succeeded",

"duration_ms": 847,

"input_size": 312,

"output_size": 640,

"created_at": "2026-03-23T10:00:00Z"

"execution_id": "exec_def",

"workflow_id": "run_qa_session_001",

"agent_node_id": "documentation-chatbot",

"reasoner_id": "parallel_retrieve",

"status": "succeeded",

"duration_ms": 1203,

"input_size": 512,

"output_size": 1480,

"created_at": "2026-03-23T10:00:01Z"

"execution_id": "exec_ghi",

"workflow_id": "run_qa_session_001",

"agent_node_id": "documentation-chatbot",

"reasoner_id": "synthesize_answer",

"status": "succeeded",

"duration_ms": 2156,

"input_size": 1480,

"output_size": 920,

"created_at": "2026-03-23T10:00:02Z"

"total": 3,

"page": 1,

"page_size": 10,

"total_pages": 1

Six months from now: query executions where synthesis took longer than 3 seconds. Find patterns in failed retrievals. Build regression tests from real questions.

No custom analytics pipeline, no third-party observability tool. Every API call is logged and queryable via REST.

For a documentation chatbot, logs are sufficient.

For agents handling refunds, approving transactions, or making decisions with consequences—the question we raised earlier becomes urgent: who authorized this? Logs record what someone claims happened. They can be edited, incomplete, reconstructed after the fact. They're a record, not proof.

Agentfield provides proof. Every execution can carry verifiable credentials, cryptographically signed and traceable to authorization. See Identity & trust for how the architecture works.

Memory Fabric (Not Separate Vector DB)

The chatbot stores embeddings and documents in the control plane's built-in memory:

ingestion.py

SourceCopy

# Store document (no Pinecone setup, no Qdrant API keys)

await global_memory.set(key=document_key, data={"full_text": full_text, ...})

await global_memory.set_vector(key=vector_key, embedding=embedding, metadata=metadata)

# Retrieve via similarity search

raw_hits = await global_memory.similarity_search(query_embedding=embedding, top_k=top_k)

Zero infrastructure. Memory is scoped: workflow, session, global. A support triage agent can read the website-docs namespace. It can't touch another agent's workflow state. Isolation where you need it, sharing where you want it.

Memory Scoping

Copy

# Documentation chatbot stores in global scope with namespace

await global_memory.set(key="website-docs:doc:getting-started", data={...})

# Support triage agent can read from the same namespace

docs = await app.memory.global_scope.get("website-docs:doc:getting-started")

Deployment Configuration

The entire deployment configuration:

railway.json

SourceCopy

"$schema": "https://railway.com/railway.schema.json",

"build": {

"builder": "NIXPACKS",

"nixpacksConfigPath": "nixpacks.toml"

"deploy": { "startCommand": "python main.py" }

Same code runs locally, in Docker, on Railway, on Kubernetes. The only thing that changes is the control plane URL in an environment variable.

No deployment-specific rewrites, no "works on my machine" debugging, no separate staging and production configurations beyond the infrastructure endpoint.

The Microservices Evolution

Architecture matters more than the model here.

When web applications hit complexity, we didn't keep everything in one process. We extracted microservices: same engineers, same company, same product, but independent services that could be deployed, scaled, and owned separately.

AI is reaching that point now.

When you have a documentation chatbot, a sentiment analyzer, a content moderator, and a support triage system all in one codebase, you hit the same walls:

Can't deploy independently (chatbot update requires redeploying everything)

Can't scale independently (sentiment analysis is overloaded but you can't add capacity without scaling everything)

Teams block each other (marketing waits for engineering's deployment window)

Testing becomes integration hell

Agent nodes are microservices for AI. Extract them when complexity hits, just like extracting a service from a monolith.

When to Extract an Agent

The same criteria you use for microservices:

Extract when a reasoner is reused across multiple places.

Our documentation chatbot has a query planner that generates diverse search queries from a user's question:

query_planning.py

SourceCopy

@query_router.reasoner()

async def plan_queries(question: str) -> QueryPlan:

"""Generate 3-5 diverse search queries from the user's question."""

Query planning is useful beyond documentation search. Support ticket search needs it. Knowledge base queries need it. Research automation needs it.

If it gets reused, extract it as its own agent node:

Extracted Agent

Copy

# Now it's a separate agent

app = Agent(node_id="query-planner", agentfield_server=...)

@app.reasoner()

async def plan_queries(question: str) -> QueryPlan:

# Now callable from ANY agent via app.call()

Extract when different teams need ownership.

Marketing's content analyzer runs on their infrastructure. Engineering's payment processor runs on theirs. Neither knows where the other lives. They don't need to.

Team Ownership

Copy

# Marketing's agent (deployed on marketing-infra.company.com)

app = Agent(node_id="content-analyzer", agentfield_server="https://control-plane.company.com")

# Engineering's agent (deployed on eng-infra.company.com)

app = Agent(node_id="payment-processor", agentfield_server="https://control-plane.company.com")

Both connect to the same control plane. The control plane auto-discovers them via heartbeat. When marketing's agent needs to call engineering's agent:

Cross-Agent Call

Copy

# No hardcoded URLs. No service mesh. No DNS configuration.

result = await app.call("payment-processor.validate_card", card_data=card)

The control plane routes the request. Load balances if there are multiple instances. Retries if one fails. Logs every call automatically.

Extract when you need independent scaling.

Your documentation chatbot handles 10 requests per minute. Your sentiment analyzer handles 1000. Traditional approach: scale the whole cluster together. Waste resources.

The fix: scale each agent independently. The control plane routes to available instances. Documentation chatbot at 1 instance. Sentiment analyzer at 50. Neither knows or cares about the other's capacity.

Extract when you want independent deployment schedules.

Support ships Friday afternoon. Marketing tests new features in staging. Neither waits for the other. The control plane discovers new agent versions automatically.

The Infrastructure You'd Otherwise Build

Without a control plane, multi-agent coordination requires:

Service registry (Consul, etcd) for discovery

API gateway for routing

Context propagation (manual)

Distributed tracing (Jaeger, Zipkin)

State synchronization (Redis, shared database)

Retry and circuit breaker logic

Workflow tracking system

Observability instrumentation

Then you write business logic.

With Agentfield:

Agentfield

Copy

# This is all you write

result = await app.call("query-planner.plan_queries", question=ticket["message"])

The control plane handles service discovery, routing, context propagation, logging, workflow tracking, and observability.

You write reasoning logic. Infrastructure handles itself.

What This Enables

Today, we have a documentation chatbot that connects to a control plane.

Tomorrow, we add a support triage agent. It connects to the same control plane. It can call the documentation chatbot's query planner if it needs to search knowledge bases. It shares memory. It shares logging. It shares deployment infrastructure.

Support Triage Agent

Copy

app = Agent(node_id="support-triage", agentfield_server=...)

@app.reasoner()

async def triage_ticket(ticket: dict) -> Decision:

# Call the query planner agent

queries = await app.call("query-planner.plan_queries", question=ticket["message"])

# Search knowledge base

context = await app.call("documentation-chatbot.retrieve", queries=queries)

# Make triage decision

return await decide_priority(ticket, context)

Next month, content moderation. Same pattern. Independent deployment.

All three agents:

Connect to the same control plane

Share the memory fabric

Call each other via app.call() (no hardcoded URLs, no service mesh configuration)

Have execution logs in the same place

Scale independently (query planner can have 10 instances while doc chatbot has 2)

Deploy independently (teams own their agents)

Looking Forward

The chatbot answers questions. What matters is what comes next.

RSS watchers for auto-reindex. The infrastructure already supports scheduled tasks. When documentation changes on GitHub, trigger re-ingestion: a scheduled execution through the control plane, not a separate cron job.

Evaluators running against execution logs. Every question and answer is stored. Build evaluation agents that run nightly, scoring answer quality against canonical Q&A pairs. No new infrastructure, just another agent reading from the same memory fabric.

More agents, same control plane. Support triage. Content moderation. Intelligent onboarding. Each is an independent agent node. Each deploys separately. All share infrastructure.

The AI backend grows with your needs through agent nodes connecting to a shared control plane, not through accumulating independent services with independent infrastructure.

We didn't build a documentation chatbot.

We built the first node in an AI backend.

Our Production Documentation Chatbot Code · Architecture overview

Twitter | LinkedIn
