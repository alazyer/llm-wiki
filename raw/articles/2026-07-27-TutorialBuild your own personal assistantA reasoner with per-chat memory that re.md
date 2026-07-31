---
title: "TutorialBuild your own personal assistantA reasoner with per-chat memory that reaches your other agents, plus a 70-line Telegram bridge you own. Text your personal stack from your phone.Jul 2, 2026 · 27 min"
source_url: "https://agentfield.ai/blog/build-your-own-personal-assistant"
ingested: 2026-07-27
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/build-your-own-personal-assistant

## Article Content

Build your own personal assistant — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Build your own personal assistant

A reasoner with per-chat memory that reaches your other agents, plus a 70-line Telegram bridge you own. Text your personal stack from your phone.

Santosh Kumar Radha·Co-founder & CTO·

27 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

This is the second post in the personal stack. It assumes you already have the control plane running from Your personal control plane; everything here registers into that same local process.

A personal assistant here is one reasoner with per-chat memory that can reach every other agent you have registered, fronted by a small bridge you own that carries messages from a chat app to the control plane and back. It remembers your last few messages per chat, it can call your other reasoners when a request needs them, and you talk to it from your phone. By the end you have a bot you can text.

Receipts: about 20 lines of reasoner, about 70 lines of bridge, one HTTP call per message. No chat integration ships with AgentField; the bridge is plain code you run, shown in full below. Time to a working conversation: about fifteen minutes.

The pattern: a stateless reasoner plus a session key

A single LLM call has no memory. To make an assistant feel continuous, you need two things the control plane already gives you: a session id that stays stable across a conversation, and memory scoped to that id.

The control plane reads an X-Session-ID header on every execute call and exposes it to the reasoner as app.ctx.session_id. Memory has a session scope keyed on exactly that id. So the reasoner reads the running history for this chat, adds the new turn, calls the model, and writes the history back. The bridge sets one session id per chat (tg-<chat_id>), and each chat gets its own memory automatically.

Build the assistant reasoner

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

The reasoner loads history for the current session, appends the user's message, asks the model, stores the exchange, and returns a reply. History lives in session-scoped memory, so two different chats never see each other's context.

PythonTypeScriptGo

# main.py

import os

from agentfield import Agent

app = Agent(node_id="assistant")

MAX_TURNS = 12 # how many prior messages to carry as context

@app.reasoner(tags=["entry"])

async def chat(message: str) -> dict:

sid = app.ctx.session_id or "default"

mem = app.memory.session(sid)

history = await mem.get("history", default=[])

history.append({"role": "user", "content": message})

reply = await app.ai(

system=(

"You are a personal assistant. Be brief and direct. "

"You can call other tools when a request needs them."

user=str(history[-MAX_TURNS:]),

tools="discover",

history.append({"role": "assistant", "content": reply.text})

await mem.set("history", history[-MAX_TURNS:])

return {"reply": reply.text}

if __name__ == "__main__":

app.run(host="0.0.0.0", port=int(os.getenv("PORT", "8002")), auto_port=False)

Two things are doing the work here. The session-scoped memory client is bound to one session, so a read and write of history never leaks across chats. And discovery lets the model reach the other agents you have registered: if you have a reasoner that reads your calendar or triages a repo, the assistant can call it without you wiring anything. Every memory call is awaited (or returns an error in Go). One naming note: Python's app.memory.session(sid) takes the id explicitly, while Go's SessionScope() reads it from the request context, and TypeScript passes the id as a positional argument.

The bridge: getUpdates to execute to sendMessage

AgentField ships no Telegram integration. The bridge is user code, and here it is in full: it long-polls Telegram for new messages, POSTs each one to the assistant reasoner with a per-chat session id, and sends the reply back. It is a plain HTTP client (no SDK) hitting the documented execute endpoint, so it reads the same in any language.

Create a bot with @BotFather, take the token it gives you, and set TG_TOKEN. The control plane is on localhost, so no API key is needed.

PythonTypeScriptGo

# bridge.py: plain httpx, no SDK

import asyncio

import os

import httpx

TG_TOKEN = os.environ["TG_TOKEN"]

TG_API = f"https://api.telegram.org/bot{TG_TOKEN}"

AGENTFIELD = os.getenv("AGENTFIELD_SERVER", "http://localhost:8080")

REASONER = "assistant.chat"

async def call_assistant(http: httpx.AsyncClient, chat_id: int, text: str) -> str:

"""Send one message to the assistant reasoner, keyed by chat."""

r = await http.post(

f"{AGENTFIELD}/api/v1/execute/{REASONER}",

headers={"X-Session-ID": f"tg-{chat_id}"},

json={"input": {"message": text}},

timeout=90,

r.raise_for_status()

return r.json()["result"]["reply"]

async def send_message(http: httpx.AsyncClient, chat_id: int, text: str) -> None:

await http.post(

f"{TG_API}/sendMessage",

json={"chat_id": chat_id, "text": text},

timeout=30,

async def main() -> None:

offset = None

async with httpx.AsyncClient() as http:

print("bridge up; text your bot")

while True:

# Long-poll Telegram for new messages (up to 25s per wait).

params = {"timeout": 25}

if offset is not None:

params["offset"] = offset

resp = await http.get(f"{TG_API}/getUpdates", params=params, timeout=40)

updates = resp.json().get("result", [])

for update in updates:

offset = update["update_id"] + 1 # ack this update

msg = update.get("message")

if not msg or "text" not in msg:

continue

chat_id = msg["chat"]["id"]

try:

reply = await call_assistant(http, chat_id, msg["text"])

except Exception as exc: # keep the bridge alive on any single failure

reply = f"assistant error: {exc}"

await send_message(http, chat_id, reply)

if __name__ == "__main__":

asyncio.run(main())

The whole integration is one POST per message. The X-Session-ID: tg-<chat_id> header is what makes the assistant remember: two people (or two chats) each get their own history because the session id differs. Advancing offset past each update_id is Telegram's acknowledgement, so you never reprocess a message.

Run the three pieces in three terminals: af server, the assistant (python main.py), and the bridge (TG_TOKEN=... python bridge.py). Then text your bot.

Per-chat memory, demonstrated

Send two messages in a row and the second one sees the first:

you: my name is Sam and I run a homelab

bot: Noted, Sam. What do you want to do with the homelab?

you: what's my name?

bot: Sam.

The second message carried no name. The reasoner read it back from app.memory.session("tg-<your_chat_id>"). Open a different chat and the bot will not know your name there, because that is a different session.

What the control plane does underneath

The bridge is thin because the control plane carries the state:

Session routing: it threads X-Session-ID from the request through to app.ctx.session_id, so the reasoner keys memory without parsing anything.

Memory: session-scoped history persists in SQLite; it survives a reasoner restart, so redeploying the assistant does not wipe conversations.

Discovery: tools="discover" resolves against the live registry, so the assistant reaches whatever agents are up right now, not a hardcoded list.

Tracing: each message becomes a workflow DAG, including any downstream agent the model chose to call, so you can see what the assistant did.

Long-poll now, webhook later

Long-polling is the right default: it works from a laptop behind NAT with no public URL, and the bridge above is the whole thing. When you want the assistant always-on from a server, switch to Telegram's webhook mode. Telegram will POST each update to a public URL, and AgentField ingests that through its generic bearer source: Telegram's X-Telegram-Bot-Api-Secret-Token header maps to the generic_bearer source with config={"header": "X-Telegram-Bot-Api-Secret-Token", "scheme": ""}, which verifies the secret and rejects anything unsigned with a 401. Every inbound source on AgentField requires a secret, so the webhook path is authenticated by construction. Start with long-poll; reach for the webhook when you outgrow the laptop.

Run it

Confirm the assistant registered, then prove memory across two calls with the same session id:

curl -fsS http://localhost:8080/api/v1/discovery/capabilities \

| jq '.capabilities[] | select(.agent_id=="assistant") | .reasoners[].id'

curl -sS -X POST http://localhost:8080/api/v1/execute/assistant.chat \

-H 'Content-Type: application/json' \

-H 'X-Session-ID: tg-demo' \

-d '{"input": {"message": "my name is Sam"}}' | jq '.result.reply'

curl -sS -X POST http://localhost:8080/api/v1/execute/assistant.chat \

-H 'Content-Type: application/json' \

-H 'X-Session-ID: tg-demo' \

-d '{"input": {"message": "what is my name?"}}' | jq '.result.reply'

The second call, same X-Session-ID, answers "Sam". Change the header to tg-other and it will not.

Paste this into a coding agent

Drop this into an AgentField-aware coding agent (Claude Code, Codex, Gemini CLI, Cursor) and let it build both halves:

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Next step: your assistant can talk, but it should also do work and stay safe doing it. Add hands with the nightly repo fleet, and add brakes with phone approvals, below.

Related

Your personal control plane, the prerequisite this assistant registers into.

A nightly maintenance fleet for your repos, to give your assistant hands that do real work.

Approve your agents from your phone, to give it brakes before anything risky ships.

The agent that finds its own tools, for how tools="discover" assembles a mesh at runtime.

Add an agent mesh to your existing FastAPI or Next.js app, if you would rather call these reasoners from an app you already run.
