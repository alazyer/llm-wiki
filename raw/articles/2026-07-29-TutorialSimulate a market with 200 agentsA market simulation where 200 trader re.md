---
title: "TutorialSimulate a market with 200 agentsA market simulation where 200 trader reasoners read a shared order book from global memory, decide with a cheap model, and post orders back every round, while an orchestrator fans them out and code clears the price, all watchable live in the dashboard DAG.Jul 2, 2026 · 33 min"
source_url: "https://agentfield.ai/blog/simulate-a-market-with-200-agents"
ingested: 2026-07-29
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/simulate-a-market-with-200-agents

## Article Content

Simulate a market with 200 agents — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Simulate a market with 200 agents

A market simulation where 200 trader reasoners read a shared order book from global memory, decide with a cheap model, and post orders back every round, while an orchestrator fans them out and code clears the price, all watchable live in the dashboard DAG.

Santosh Kumar Radha·Co-founder & CTO·

33 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

This is the fifth build in the personal stack, the running system you started in your personal control plane. Point 200 trader agents at one shared order book, run them for ten rounds, and watch the price find its level in the dashboard. A market simulation on AgentField is N trader reasoners reading and writing one piece of shared memory, an orchestrator that fans all of them out each round, and deterministic clearing in plain Python between rounds. The traders are the intelligence. The price is arithmetic.

Here is the artifact. Each trader is an async def reasoner that takes a strategy as an input, reads the current price and its own position from global memory, asks a cheap model whether to buy, sell, or hold, and posts an order back. An orchestrator runs the rounds: each round it fans out all 200 traders with asyncio.gather, then clears the book in code and writes the new price. Ten rounds of 200 traders is 2,000 agent calls from one curl. On a cheap model that lands around 30 to 60 cents for the whole run. The fan-out and the shared state are the lesson. The clearing math is six lines.

What you are building

Three moving parts, one shared surface.

A trader reasoner. One cognitive job: given the price and my position, decide buy, sell, or hold, and how much. It reads the market from app.memory.global_scope, decides with app.ai, and posts an order back to memory. Strategy comes in as an input parameter, so the same reasoner is a momentum trader, a contrarian, or a market maker depending on what the orchestrator passes.

An orchestrator reasoner. It seeds the book, then loops R rounds. Each round it fans out all N traders at once, collects their orders, and hands them to the clearing function.

A clearing function, plain Python. It reads the round's orders, nets supply against demand, moves the price, and writes the new state back to global memory. No model call. Price discovery is arithmetic, and arithmetic belongs in code.

Build it

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

1. Seed the market in global memory

Global scope is the shared surface every trader reads and writes. The four real memory scopes are global, session, actor, and workflow, and this simulation only needs global. In Python the accessor is app.memory.global_scope; in TypeScript you pass 'global' positionally; in Go it is mem.GlobalScope().

PythonTypeScriptGo

import asyncio

import os

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="market", agentfield_server=os.getenv("AGENTFIELD_SERVER"))

STARTING_PRICE = 100.0

async def seed_market() -> None:

await app.memory.global_scope.set("price", STARTING_PRICE)

await app.memory.global_scope.set("round", 0)

await app.memory.global_scope.set("orders", []) # this round's orders

price, round, and orders are the whole world state. Every trader reads price, every trader appends to orders, and the clearing function rewrites price and empties orders between rounds.

2. The trader reasoner reads, decides, and posts back

One trader, one decision. It must be async def. A plain def reasoner serializes inline and you lose all the parallelism, which is the entire point of the exercise. Strategy arrives as a parameter, so the orchestrator can spawn 200 traders across a handful of strategies from one reasoner.

PythonTypeScriptGo

class Order(BaseModel):

action: str # "buy", "sell", or "hold"

size: int # shares, 0 for hold

confident: bool

@app.reasoner()

async def trader(trader_id: str, strategy: str, model: str | None = None) -> dict:

price = await app.memory.global_scope.get("price")

round_no = await app.memory.global_scope.get("round")

order = await app.ai(

system=(

f"You are trader {trader_id} using a {strategy} strategy. "

"Given the current price, decide buy, sell, or hold and a size "

"between 1 and 10 shares. A momentum trader buys when price rose "

"last round; a contrarian does the opposite; a market maker fades "

"extremes. Return JSON."

user=f"Round {round_no}. Current price: {price}.",

schema=Order,

model=model,

if not order.confident or order.action == "hold":

return {"trader_id": trader_id, "action": "hold", "size": 0}

# Post the order back to the shared book.

orders = await app.memory.global_scope.get("orders", [])

orders.append({"trader_id": trader_id, "action": order.action, "size": order.size})

await app.memory.global_scope.set("orders", orders)

return {"trader_id": trader_id, "action": order.action, "size": order.size}

The trader reads shared state, reasons over it with a cheap model, and writes shared state. That read-decide-write cycle over global_scope is the shared-state pattern. Two hundred of them run it at once every round.

3. Clear the price in code, not with a model

Clearing is deterministic. Do not ask a model to compute a price from supply and demand; a model would be slower, more expensive, and wrong in ways you cannot audit. Net the buy volume against the sell volume and nudge the price by the imbalance.

def clear(orders: list[dict], price: float) -> float:

buys = sum(o["size"] for o in orders if o["action"] == "buy")

sells = sum(o["size"] for o in orders if o["action"] == "sell")

total = buys + sells

if total == 0:

return price

imbalance = (buys - sells) / total # -1.0 to 1.0

return round(price * (1 + 0.05 * imbalance), 2) # 5% max move per round

Excess buyers push the price up, excess sellers push it down, and a balanced book leaves it flat. This is the one place in the system where an LLM would be the wrong tool. The traders supply judgment; the clearing supplies the rule.

4. The orchestrator runs the rounds

The orchestrator seeds the book, then runs R rounds. Each round it empties the order book, fans out all N traders with asyncio.gather, reads the orders they posted, clears the price, and advances the round counter.

@app.reasoner(tags=["entry"])

async def run_market(

n_traders: int = 200,

rounds: int = 10,

model: str | None = None,

) -> dict:

await seed_market()

strategies = ["momentum", "contrarian", "market_maker"]

history = [await app.memory.global_scope.get("price")]

for r in range(1, rounds + 1):

await app.memory.global_scope.set("round", r)

await app.memory.global_scope.set("orders", []) # fresh book each round

# Fan out every trader at once.

await asyncio.gather(*[

app.call(

f"{app.node_id}.trader",

trader_id=f"t{i}",

strategy=strategies[i % len(strategies)],

model=model,

for i in range(n_traders)

# Clear the book in code, write the new price.

orders = await app.memory.global_scope.get("orders", [])

price = await app.memory.global_scope.get("price")

new_price = clear(orders, price)

await app.memory.global_scope.set("price", new_price)

history.append(new_price)

return {

"rounds": rounds,

"traders": n_traders,

"final_price": history[-1],

"price_history": history,

if __name__ == "__main__":

app.run()

The asyncio.gather over app.call is the fan-out. Every trader is a separate tracked execution through the control plane, not a coroutine in this process, so 200 of them spread across the worker pool. strategies[i % len(strategies)] gives you roughly 67 momentum traders, 67 contrarians, and 66 market makers from one trader reasoner. Change the list and you change the population without touching the reasoner.

What the control plane does underneath

The orchestrator is plain Python. The control plane turns 2,000 app.call invocations into a running, observable system.

Fan-out and queueing. Each round dispatches 200 calls. Those beyond the concurrency ceiling queue and drain as workers free up, so 200 concurrent traders do not open 200 sockets from one process. Backpressure is automatic.

Shared state. app.memory.global_scope is one surface every trader reads and writes through the control plane, so the order book is consistent across all 200 executions rather than living in one process's local variables.

Tracing. Every round is a fan of 200 edges in the workflow DAG. Open the run in the dashboard and watch the tree fill in round by round, one layer of 200 per round, ten layers deep.

Retries. A transient failure in one trader is retried before the orchestrator sees it, so one flaky call does not stall the round.

Run it

Fire the entry reasoner over the async endpoint, since a 2,000-call run will exceed the 90-second sync cap:

curl -s -X POST http://localhost:8080/api/v1/execute/async/market.run_market \

-H "Content-Type: application/json" \

-d '{"input": {"n_traders": 200, "rounds": 10, "model": "gpt-4o-mini"}}'

# => {"execution_id": "exec_7c2d...", "status": "queued"}

Poll for the result:

curl -s http://localhost:8080/api/v1/executions/exec_7c2d...

# => {"status": "succeeded",

# "result": {"rounds": 10, "traders": 200, "final_price": 108.4,

# "price_history": [100.0, 102.1, 99.8, ...]}}

Open http://localhost:8080/ui/ while it runs and watch the DAG. Each round is a visible fan of 200 executions, and you can click any trader to see what it decided and why. Watching 200 agents move one number is the point of building it.

Before you scale it

Two honest limits, both about the local SQLite mode you get by default.

A single SQLite writer serializes writes. Two hundred traders all calling global_scope.set("orders", ...) in the same round hit one writer, one at a time. In local mode this is a latency ceiling, not a failure: the run completes, it just paces itself. Point the control plane at PostgreSQL and that ceiling lifts. Read your personal control plane for how to switch the storage mode.

Traders must be async def. A plain def reasoner runs inline and serializes the whole round, so your fan-out collapses into a sequential loop and the 200-agent spectacle disappears. Keep every reasoner in this build async def.

There is also a modeling caveat worth naming: appending to a shared orders list from 200 concurrent traders is a read-modify-write race, which is fine for a simulation where a dropped order or two changes nothing you care about. If you ever want exact accounting, have each trader write to its own key (orders:t{i}) and let the clearing function read them all, so no two traders touch the same key.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Next step: clone the two reasoners, drop n_traders to 20 and rounds to 3 for a fast first run, then open the dashboard, fire it, and watch the fan fill in before you scale back up to 200.

Related

Fan out 1,000 parallel agents, the recursive form of the same fan-out, when the width comes from decomposition instead of a fixed trader count.

Your personal control plane, the foundation this simulation runs on, and where to switch from SQLite to PostgreSQL.

Thinking in reasoners, the mental model behind why the traders reason and the clearing is code.

250 agents for 90 cents, the cost math for running fleets of agents on a cheap model.
