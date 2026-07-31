---
title: "TutorialA nightly maintenance fleet for your reposOne scheduled reasoner that clones your repos at 3am, runs a budget-capped coding agent on each, and leaves a morning summary and a stack of ready PRs behind an approval gate.Jul 2, 2026 · 39 min"
source_url: "https://agentfield.ai/blog/nightly-repo-fleet"
ingested: 2026-07-29
blog: "AgentField"
published: "2026-07-24"
---
## Source Metadata
- **Blog:** AgentField
- **Published:** 2026-07-24
- **URL:** https://agentfield.ai/blog/nightly-repo-fleet

## Article Content

A nightly maintenance fleet for your repos — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · July 2, 2026

A nightly maintenance fleet for your repos

One scheduled reasoner that clones your repos at 3am, runs a budget-capped coding agent on each, and leaves a morning summary and a stack of ready PRs behind an approval gate.

Santosh Kumar Radha·Co-founder & CTO·

39 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

This is the flagship of the personal stack, the series where every tutorial adds one agent to the control plane you run on your own machine. A nightly maintenance fleet is a single scheduled reasoner that wakes at 3am, scratch-clones a list of your repos, runs one budget-capped coding agent on each in parallel, and writes you a morning summary with the cost of each worker and a pull request waiting behind an approval gate. It is one @on_schedule reasoner plus about 60 lines of orchestration. A run over eight small repos on a cheap-to-mid provider lands in the low single dollars, and no single repo can cost more than its own cap.

The coding-agent fleet post built the parallel dispatcher: N capped app.harness() calls, one cost table out. This is its personal sequel. The dispatcher is the same shape; what is new is that a cron trigger fires it while you sleep, each worker gets a real git checkout and a token, and nothing merges until you tap approve on your phone.

The pattern

A maintenance fleet turns "I should keep my side projects tidy" into a scheduled job. The control plane runs a real 5-field cron scheduler. A reasoner bound to 0 3 * * * fires every night at 3am in the timezone you pick. Inside it, you fan out one app.harness() per repo, each in its own scratch clone with its own dollar cap, and you collect a flat result from each. The output is a summary in memory and, for repos where the agent actually changed something, a branch pushed and a PR opened that sits unmerged until a human says yes.

The static alternative is a cron entry that runs a shell script calling a coding-agent CLI in a loop. It has no per-repo budget, no isolation between checkouts, no typed result to summarize, and no place for a human to stand between the diff and main. The fleet gives you all four.

Safety first, because the harness gets a shell and your token

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

Be plain about what this does. Each worker is a coding agent with Bash in its tool list and your GH_TOKEN in its environment. It can run git, gh, and anything else on the PATH inside its working directory. That is the point, and it is also the risk. Four rules keep it bounded:

Every worker operates on a fresh scratch clone under a throwaway directory, never your real working copy. If a worker corrupts a checkout, you delete the directory.

Give the fleet a token scoped to the specific repos it maintains, with only the permissions it needs (contents and pull-requests), not your personal all-repo token.

Every worker gets a max_turns limit and a dollar cap, so a looping agent stops on its own.

The worker may push a branch and open a PR, but the merge sits behind app.pause(). Nothing lands on main without you.

Build it

1. The scheduled entry reasoner

Bind a reasoner to the cron expression with @on_schedule. The expression is 5-field, minute-granularity: minute, hour, day-of-month, month, day-of-week. There are no @daily macros and no day names, so "3am every day" is 0 3 * * *. Pass an IANA timezone so the schedule follows your wall clock. Schedule triggers exist in Python and Go; TypeScript has no trigger surface, so this block is a two-tab.

PythonGo

import asyncio

import os

import shutil

import subprocess

import tempfile

import httpx

from pydantic import BaseModel

from agentfield import Agent, on_schedule

app = Agent(node_id="nightly-fleet")

REPOS = [

"you/dotfiles",

"you/side-project",

"you/blog",

"you/scripts",

class RepoResult(BaseModel):

repo: str

changed: bool

branch: str

summary: str

pr_url: str

@app.reasoner(tags=["entry"])

@on_schedule("0 3 * * *", timezone="America/New_York")

async def nightly_maintenance() -> dict:

results = await asyncio.gather(

*[maintain_one(repo) for repo in REPOS],

return_exceptions=True,

return await summarize(results)

@on_schedule is sugar for triggers=[ScheduleTrigger(cron="0 3 * * *", timezone=...)]; both forms register the same cron row when the agent starts up. In Go the equivalent is agent.WithScheduleTrigger("0 3 * * *") or the typed agent.ScheduleTrigger{Cron: ..., Timezone: ...} above. The timezone="America/New_York" matters, because ScheduleTrigger defaults to UTC, so if you skip it the fleet fires at 3am UTC, which is probably not 3am where you are.

2. One capped worker per repo

Each worker scratch-clones its repo, then hands the checkout to a single app.harness() call. The worker gets Bash so it can run the project's own tooling, GH_TOKEN in env so git and gh authenticate, a cwd pointing at the throwaway clone, and a hard budget cap. It returns a tuple of the parsed result plus the real cost and turn count, so the summary step can total them.

UPKEEP_PROMPT = (

"You are a nightly maintenance agent for this repository. Apply safe, "

"mechanical upkeep only: fix lint errors, update obviously-stale "

"dependencies, tidy formatting, and fix broken internal links. Run the "

"project's own tests or linters if they exist. Do NOT change behavior or "

"public APIs. If you make changes, create a branch named 'nightly/upkeep', "

"commit, and push it. Do NOT open a pull request and do NOT merge. Report "

"what you changed, the branch name, and whether you changed anything at all."

async def maintain_one(repo: str) -> tuple[RepoResult, float, int]:

workdir = tempfile.mkdtemp(prefix="nightly-")

token = os.environ["GH_TOKEN"] # scoped to these repos only

clone_url = f"https://x-access-token:{token}@github.com/{repo}.git"

try:

subprocess.run(

["git", "clone", "--depth", "50", clone_url, workdir],

check=True, capture_output=True,

result = await app.harness(

prompt=UPKEEP_PROMPT,

provider="claude-code", # hard $ cap only works on claude-code

cwd=workdir,

env={"GH_TOKEN": token},

tools=["Read", "Write", "Bash"],

permission_mode="auto", # allow file writes

max_turns=30,

max_budget_usd=0.75, # per-repo cap, enforced by the provider

schema=RepoResult,

if result.is_error:

failed = RepoResult(

repo=repo, changed=False, branch="",

summary=result.error_message or "worker failed", pr_url="",

return failed, result.cost_usd or 0.0, result.num_turns

parsed = result.parsed

parsed.repo = repo

return parsed, result.cost_usd or 0.0, result.num_turns

finally:

shutil.rmtree(workdir, ignore_errors=True)

Two things to say plainly about cost. First, max_budget_usd is a hard cap only when provider="claude-code", where the cap is enforced inside that provider. For codex, gemini, or opencode, bound the spend with max_turns and read result.cost_usd after the fact rather than trusting the dollar cap. Second, env merges over the process environment, so GH_TOKEN reaches the git and gh the harness shells out to.

3. Aggregate the receipts yourself

There is no per-workflow cost API in the control plane. The receipts are something you build: each worker returns result.cost_usd, and you sum them. Never point a reader at a cost dashboard that does not exist. The summary is what you build and what you wake up to.

async def summarize(results: list) -> dict:

total_cost = 0.0

rows, changed_repos, failures = [], [], []

for item in results:

if isinstance(item, Exception):

failures.append({"error": str(item)})

continue

parsed, cost, turns = item

total_cost += cost

rows.append({

"repo": parsed.repo,

"changed": parsed.changed,

"branch": parsed.branch,

"cost_usd": round(cost, 4),

"turns": turns,

if parsed.changed:

changed_repos.append(parsed)

summary = {

"date": os.popen("date -u +%Y-%m-%d").read().strip(),

"repos_scanned": len(rows),

"repos_changed": len(changed_repos),

"total_cost_usd": round(total_cost, 4),

"rows": rows,

"failures": failures,

# Write it to global memory so the morning read is one GET.

await app.memory.global_scope.set("nightly/last-run", summary)

# Optional: ping your phone that the run finished (5 lines, no secret needed).

topic = os.getenv("NTFY_TOPIC")

if topic:

async with httpx.AsyncClient() as http:

await http.post(

f"https://ntfy.sh/{topic}",

headers={"Title": "Nightly fleet done"},

content=(

f"{summary['repos_changed']}/{summary['repos_scanned']} repos "

f"changed, ${summary['total_cost_usd']}"

).encode("utf-8"),

return summary

Memory is scoped through accessors, not a scope= keyword. app.memory.global_scope is a property that returns a client scoped to global; its .set(key, data) takes no scope argument. Every memory call is awaited. The nightly summary lives at one global key, so your morning read is a single GET.

The ntfy ping is plain HTTP: a POST to https://ntfy.sh/<your-topic> with the title in a header and the message in the body. AgentField does not push anything to your phone on its own, so this five-line block is the glue that tells you the run is done.

4. The approval gate before a merge

The worker pushes a branch but is told never to merge. The merge is a separate step behind app.pause(). Open the PR, notify your phone with the approve link, then block until you answer. The full pause-notify-resolve pattern, including the tiny handler you host to sign the approval, is its own post: approve your agents from your phone. Here is where it plugs in.

async def open_and_gate(parsed: RepoResult) -> dict:

token = os.environ["GH_TOKEN"]

# Open the PR from the branch the worker pushed.

pr = subprocess.run(

["gh", "pr", "create", "--repo", parsed.repo,

"--head", parsed.branch, "--base", "main",

"--title", "Nightly upkeep", "--body", parsed.summary],

capture_output=True, text=True, env={**os.environ, "GH_TOKEN": token},

pr_url = pr.stdout.strip()

request_id = f"merge-{parsed.repo.replace('/', '-')}-{parsed.branch}"

approve_url = f"{os.environ['APPROVE_BASE']}/approve?id={request_id}"

# Notify first (pause sends nothing outbound on its own).

topic = os.getenv("NTFY_TOPIC")

if topic:

async with httpx.AsyncClient() as http:

await http.post(

f"https://ntfy.sh/{topic}",

headers={"Title": f"Merge {parsed.repo}?", "Click": approve_url},

content=f"{parsed.summary}\n{pr_url}".encode("utf-8"),

# Block until you tap approve.

outcome = await app.pause(

approval_request_id=request_id,

approval_request_url=pr_url,

expires_in_hours=24,

if outcome.approved:

subprocess.run(

["gh", "pr", "merge", "--repo", parsed.repo, "--squash", parsed.branch],

check=True, env={**os.environ, "GH_TOKEN": token},

return {"repo": parsed.repo, "merged": True, "pr_url": pr_url}

return {"repo": parsed.repo, "merged": False, "decision": outcome.decision}

app.pause() registers the wait on the control plane and blocks; it does not send you anything. That is why the ntfy POST runs first, while the reasoner still holds the approve link. When you tap approve, the control plane resolves the wait and the reasoner continues from the exact line after pause() to run gh pr merge.

What the control plane does underneath

The cron scheduler is real, 5-field, timezone-aware, and starts firing on agent registration. You do not run your own scheduler.

Each app.harness() call is a tracked child execution, so one nightly run shows up as one workflow with one child per repo in the DAG.

The app.pause() wait is a waiting row in the control plane database, not a coroutine in RAM, so a review that spans your commute survives an agent restart.

Cost, turns, and duration are recorded per worker, so the totals you compute in code match what the control plane observed. There is no aggregate cost endpoint; the summary is yours to build.

Run it

You do not have to wait until 3am to test it. Fire the reasoner directly and watch it fan out.

curl -sS -X POST http://localhost:8080/api/v1/execute/async/nightly-fleet.nightly_maintenance \

-H 'Content-Type: application/json' \

-d '{"input": {}}'

That returns an execution_id. Poll for the summary:

curl -sS http://localhost:8080/api/v1/executions/<execution_id> | jq '.status, .result'

Read the morning summary straight from memory once the run lands:

curl -sS -X POST http://localhost:8080/api/v1/memory/get \

-H 'Content-Type: application/json' \

-d '{"key": "nightly/last-run", "scope": "global"}' | jq

The receipt

A plausible nightly run over four small repos, all four workers concurrent on claude-code with a $0.75 cap each:

repo changed cost_usd turns

you/dotfiles true 0.31 11

you/side-project true 0.44 16

you/blog false 0.12 5

you/scripts false 0.09 4

-------------------------------------------------

total 0.96 36 2 PRs opened, 0 merged yet

Worst case was bounded before the run: four repos at a $0.75 cap is at most $3.00 no matter how badly one worker loops. The actual $0.96 is what a night of small mechanical upkeep costs. Two PRs are waiting behind the approval gate, and they merge when you tap approve at breakfast.

Next step: add your real repos to REPOS, scope a token to exactly those, and set NTFY_TOPIC so the morning ping reaches your phone.

Paste this into /agentfield

Get the CLI with curl -fsSL https://agentfield.ai/install.sh | bash. The /agentfield command works in Claude Code, Codex, Gemini CLI, and other coding agents.

Give this to your coding agent

Copies the full setup prompt: install AgentField, add the Python SDK, start the agent, and run a smoke test.

Copy prompt

Go and TypeScript

The schedule and harness setup shows a Go tab above; the rest of this post stays Python because the orchestration is the point. Two honest surface differences decide what ports cleanly. Schedule triggers exist in Python and Go but not in TypeScript, so the cron entry has no TypeScript form. The harness runs in all three SDKs, but the Go harness result exposes turns and duration and no cost field, so aggregate cost in Python or TypeScript where result.cost_usd / result.costUsd is populated. And app.pause() is a Python-only blocking primitive; in Go or TypeScript you drive the same durable wait through the approval webhook rather than a one-call pause. See the SDK docs.

Related

Your personal control plane, the foundation this fleet registers into. Read it first.

A fleet of coding agents with budget caps, the parallel dispatcher this schedules on a cron. This is its personal sequel.

Approve your agents from your phone, the pause-notify-resolve pattern that gates every merge here.

What is harness orchestration, why the harness, not the LLM call, is the unit you govern.
