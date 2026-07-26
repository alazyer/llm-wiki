---
title: "TutorialClaude Code as a functionCall a multi-turn coding agent like Claude Code from your backend with a dollar budget cap, a turn limit, restricted tools, and typed output.Jul 2, 2026 · 21 min"
source_url: "https://agentfield.ai/blog/claude-code-as-a-function"
ingested: 2026-07-26
blog: "AgentField"
published: "2026-07-24"
---

## Source Metadata

- **Blog:** AgentField
- **URL:** https://agentfield.ai/blog/claude-code-as-a-function
- **Fetched:** 2026-07-26
- **Extracted title:** Claude Code as a function

## Article Content

Claude Code as a function — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

Claude Code as a function

Call a multi-turn coding agent like Claude Code from your backend with a dollar budget cap, a turn limit, restricted tools, and typed output.

Santosh Kumar Radha·Co-founder & CTO·

21 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

You already run Claude Code in your terminal. It reads files, edits them, runs shell, and iterates until the task is done. The problem starts when you want that same loop inside a backend: no interactive terminal, no human to approve each step, and a very real chance it burns twenty dollars fixing a typo.

app.harness() turns a coding-agent CLI into a callable function. You pass a prompt and a set of guardrails. You get back structured output and the turn count. Here is a reasoner that dispatches a bug-fix task to Claude Code with a three-dollar ceiling and gets a typed result:

PythonTypeScriptGo

from pydantic import BaseModel

from agentfield import Agent

app = Agent(node_id="fixer")

class FixResult(BaseModel):

files_changed: list[str]

summary: str

tests_pass: bool

@app.reasoner(tags=["entry"])

async def fix_bug(repo_path: str, issue: str) -> FixResult:

result = await app.harness(

prompt=(

f"Fix this bug: {issue}\n"

f"Work in {repo_path}. Run the test suite before you finish."

provider="claude-code",

max_budget_usd=3.0,

max_turns=40,

tools=["Read", "Write", "Bash"],

schema=FixResult,

return result.parsed

In Python, result.parsed is a validated FixResult. result.cost_usd is what it actually spent. result.num_turns is how many round trips it took. The agent had full autonomy to read, write, and run shell inside repo_path, but the shape of what came back was fixed before it started.

What each guardrail does

max_budget_usd is the one that matters. The agent stops when it crosses the cap, so a runaway loop costs three dollars, not thirty. Set it per call, not per process, because a recon task and a refactor task have different worst cases.

max_turns bounds iterations. A single-file edit rarely needs more than ten turns; a multi-file migration might need fifty. Setting it low is a second brake independent of the budget.

tools=["Read", "Write", "Bash"] is an allowlist. Pass ["Read"] for a read-only audit and the agent physically cannot write to disk. This is the difference between "please don't touch other files" in a prompt and the agent not having the capability at all.

schema=FixResult constrains the output to a Pydantic model. The harness writes the agent's answer to a JSON file inside the agent's working root, reads it back, and validates it. If the JSON is malformed, it tries one cheap reformat call before re-running the whole agent, so a trailing comma does not cost you another full session.

system_prompt overrides the agent's default persona. env injects environment variables (an API key, a NODE_ENV) into the subprocess without leaking them into your own process. permission_mode is "auto" (act without asking) or "plan" (plan first, act second).

When harness, when ai

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

Both app.harness() and app.ai(tools=...) can use tools. The split is about how many turns and how much navigation.

Use app.harness() when the task is a real coding job: read a file, decide what to read next, edit three files, run the tests, read the failure, fix it. That is a multi-turn agent navigating a repo it did not see in full up front.

Use app.ai(tools=...) when the task is a single reasoning step that happens to need a lookup or two. Classification, routing, a structured extraction over a bounded input. It is one LLM call with a short tool loop, not a coding session.

The heuristic: if you would open a terminal and let Claude Code churn for a few minutes, that is harness(). If you would write one prompt and read one answer, that is ai().

The four providers

provider accepts claude-code, codex, gemini, or opencode. The prompt, the schema, and the guardrails stay the same across all four. Switching provider is a one-line change:

PythonTypeScriptGo

result = await app.harness(

prompt="Refactor the auth module to use the new token API.",

provider="opencode",

model="minimax/minimax-m2.5",

max_budget_usd=2.0,

tools=["Read", "Write", "Bash"],

schema=FixResult,

The CLI must be installed in the environment that runs the reasoner. Do not assume it is there. Run af doctor and check recommendation.harness_usable: true; recommendation.harness_providers lists the CLIs it found. In a container, install the provider's CLI in the Dockerfile and add a shutil.which("<binary>") guard at startup so a missing binary fails loudly instead of at the first call.

Across the three SDKs

All three SDKs have the harness with the same four providers and the same guardrails. The syntax differs: Python and TypeScript take the prompt and an options object, while Go passes the schema and a destination pointer as positional args and the guardrails as a harness.Options struct. One difference to know: the Go harness result carries the turn count and duration but not a cost field, so a Go call reads hr.NumTurns but has no hr.CostUsd; Python (result.cost_usd) and TypeScript (result.costUsd) both return the spend.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Build a reasoner fix_bug(repo_path, issue) on an AgentField agent node named fixer. It calls app.harness() with provider="claude-code", max_budget_usd=3.0, max_turns=40, tools=["Read","Write","Bash"], and a Pydantic schema with fields files_changed: list[str], summary: str, tests_pass: bool. Return result.parsed. Add a startup shutil.which("claude") guard in main.py that exits with a clear error if the CLI is missing.

Expected file tree:

fixer/

main.py # Agent + fix_bug reasoner + shutil.which guard

Dockerfile # installs claude-code CLI

pyproject.toml

Verify it registered:

curl -X POST http://localhost:8080/api/v1/execute/async/fixer.fix_bug \

-H "Content-Type: application/json" \

-d '{"input": {"repo_path": "/work/repo", "issue": "null deref in parse_config"}}'

The receipt

The whole thing is one reasoner and one Pydantic model, about 25 lines of Python. A bug-fix run against a small repo with max_turns=40 typically lands in single-digit turns and well under the three-dollar cap, and you get the exact cost back on result.cost_usd every time. The guardrails are what make it safe to call from code that no human is watching.

Next: read af doctor output and confirm harness_usable is true before you ship a reasoner that depends on a provider CLI.

Related

How we ran a 250-agent security audit for 90 cents, where harness calls run under a hard budget cap at scale.

Fan out 1,000 parallel agents from one request, for calling many of these harnesses in parallel from one request.
