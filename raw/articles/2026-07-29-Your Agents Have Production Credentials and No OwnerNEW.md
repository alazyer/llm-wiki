---
title: "Your Agents Have Production Credentials and No OwnerNEW"
source_url: "https://www.aibuilderclub.com/blog/who-owns-your-ai-agents"
ingested: 2026-07-29
blog: "AI Builder Club"
published: "2026-07-28"
---
## Source Metadata
- **Blog:** AI Builder Club
- **Published:** 2026-07-28
- **URL:** https://www.aibuilderclub.com/blog/who-owns-your-ai-agents
- **Fetched:** 2026-07-31

## Article Content

Who Owns Your AI Agents? A Registry, a Runbook and an Honest Score (2026)

Start with one agent you already run, and one question about it: when was the last time a human looked at what its credential can reach?
Here's one, described by the person who built it. He'd stood up a few internal agents over a quarter: ticket triage, report assembly, and a nightly reconciliation job that used to belong to a person. To get them working he gave them service credentials, broad ones, because scoping them properly was going to take a week and the demo needed to land. Everyone said they'd tighten it after. They didn't tighten it after.
So the reconciliation agent has read and write on a production database. It has run every night since March. There's no record of what it does in there beyond application logs he'd have to correlate by hand.
Then he writes the sentence this whole page is built on:

It has probably been fine. I do not know that it has been fine, which is a different sentence.

Sit with the gap between those two clauses, because everything worth doing about this lives inside it. He isn't reporting an incident. Nothing has gone wrong. He's reporting that he has no way to find out, and that the absence of bad news has been doing the work of evidence for four months.
You may not have a word for this. The people who have this problem describe it as a specific agent with a specific credential, while the words that would let you look it up are governance vocabulary borrowed from identity management. That mismatch is worth naming rather than papering over, because it means the fix arrives dressed as compliance work for a problem that feels like a Tuesday. The test that skips the vocabulary is three questions about one agent you already run.
Who gets asked, by name, what it did last night. What it could reach if it went wrong in a way you haven't imagined. And what happens the day you turn it off.
The files are here to be copied. agents/REGISTRY.md is a schema whose fields are chosen so that a blank one is a finding. agents/OFFBOARDING.md is the retirement runbook, which is the step our own fleet has no version of. Under those sits an action-log contract that gives "what did it do in there" an answer, and the autonomy ladder rewritten as a table with promotion evidence per rung.
It also ships our own score against that schema, which is 1 field out of 10 answered for all 29 of our agents, and a second field that answers for 28 of them. That number is further down and it isn't flattering, which is the reason it's here.
One scoping note before the artifacts. Whether the boundary you configured actually holds is a different question with its own page, testing that your agent guardrails hold. This page assumes your controls work exactly as written and asks who is accountable for them, what they cost you when the agent is retired, and how you'd reconstruct a night's work six months later.

The Evidence Here Is Thin, and That Shapes the Page
The account above is one person's post. When we read it at the source on 2026-07-28 it carried a score of 3, and the replies carrying the fix carried 1 and 2. Compare that to the review-bottleneck problem, where spoke one had a 1,904-upvote thread, five published policy files and an 8.1M-PR dataset to work from.
That difference is itself information, and here's the reading I'd offer for it rather than a finding. Anger is what fills a thread, and there's nothing to be angry about yet in an agent that has been fine. Quiet failure mode, quiet thread.
So this page doesn't lean on external volume it doesn't have. It leans on one very specific account, on a second operator's independent conclusion from the org side, and on the thing we can actually measure, which is our own fleet held against the schema and scored where it fails.

Artifact 1: agents/REGISTRY.md
The schema is the artifact. The file is a markdown table, and if you want it in a CMDB or a spreadsheet instead, the fields are what transfers.
Two things this file does not do, stated at the top of the file itself so that a reader who finds it later is not misled by it. It records, and it doesn't enforce. Nothing in it revokes a token, stops a run or blocks a write, and every field is a claim that has to be made true somewhere else in your infrastructure. The value is that a wrong claim becomes visible and a blank one becomes a question.
markdown# Agent registry

One row per agent that runs without a human watching it. An agent you trigger by hand
belongs here too, if it holds a credential you would not hand to a contractor on day one.

This file records. It does not enforce. Nothing here revokes a token, stops a run, or
blocks a write. Every field below is a claim that has to be true somewhere else in your
infrastructure. A blank field is a finding rather than a formatting problem.

Rows are generated from whatever runs your agents, then filled in by hand. If an agent
can be absent from this list, the list is decoration.

Last full review: <date> by <name>          Next review due: <date>

## <agent name>

| Field | Value |
|---|---|
| Owner | <one person. first name and last name.> |
| Purpose | <one sentence: what stops being true if you delete it> |
| Reaches | <every system it can touch, one per line> |
| Credential | <the identity it authenticates as, what that identity can do, where the secret lives> |
| Expiry | <the date this row stops being valid> |
| Reviewed | <date> by <name> |
| Escalation | <who gets woken when it goes wrong, out of hours> |
| Kill switch | <the exact command or click that stops it, and who can run it> |
| Autonomy rung | <1-4, and the date it reached that rung> |
| Action log | <where its writes are recorded, and how far back that record goes> |

## Retired

<rows move here, they never get deleted. a deleted row is indistinguishable from an
agent that was never registered.>

Ten fields. Each has a reason to exist and a way to fill it that doesn't collapse into guesswork, because a row that can't be filled is worse than no row.

FieldWhy it's thereHow to fill itOwnerA team alias means the question "what did it do last night" has no addressee, and a rota lets each person assume another one read itWhoever would be embarrassed if it went wrong, which in practice is whoever built it. Then go and tell themPurposeWritten as what stops being true when you delete the agent, it doubles as the retirement testOne sentence. An agent whose purpose sentence you can't write is an agent to turn off, and that's the cheapest outcome this file producesReachesThe spec says what an agent is supposed to touch and the credential says what it can touch. The gap between those two is the entire subjectRead the credential rather than the spec. Include what it gets to through another tool, since a shell is a route to everything the machine can reachCredentialThree parts, because a single value collapses into a secret-manager path that answers none of themThe identity it authenticates as, what that identity is permitted to do, and where the secret livesExpiryA token expiry makes the agent break. A row expiry makes a named person look at it and decideA date somebody has to look, rather than a date something stops workingReviewedTurns the file into a process rather than a documentA date and a name. It also produces the number that matters at audit time, which is how many rows have a review date older than their own expiryEscalationOwner and escalation diverge under load: the owner answers for it, escalation can do something at 3amA name for out of hours. If it's the owner and the owner sleeps, the row is telling you somethingKill switch"We'd pause it" is a plan rather than a controlThe exact command or click, plus who can run it. The test is whether somebody who didn't build the agent could stop it from this row alone, at 2am, without waking the ownerAutonomy rungA rung that lives in a spec file's prose can't be queried, so the one question worth asking across a fleet has nowhere to be asked from: how many things are running at rung fourA number from the ladder further down, and the date it got thereAction log"It logs" is not an answer, because the question that sends you to the log arrives weeks after the writeA location and a retention window. How far back you can read is the answer, and name who can revise it, since a log the agent can edit answers a different question
Two of those fields carry an argument rather than an instruction, and both of them failed a first pass at being fillable.
Credential is why Reaches can't be filled from the spec. If the row names a credential shared with a human or with other agents, writing that down is the point of the field. GitHub's own documentation is blunt about the common case: a classic personal access token grants access to all repositories within the organizations you have access to, as well as all personal repositories in your personal account. Where that's the credential, Reaches covers rather more than one repo.
Expiry gets confused with the token's expiry, and they're different controls. A token expiry makes the agent break, and a broken agent gets fixed at speed by whoever is on, without anyone being prompted to ask whether it should still exist. GitHub's fine-grained tokens default to 30 days and accept anything between 1 and 366 days or none at all, with infinite lifetimes allowed unless an organization or enterprise policy blocks them, so the token side of this is a choice somebody made rather than a constraint. The row's date is the one that makes a person decide.

What Our Own Fleet Scores Against It
We run 29 loops across three repos, 24 of them enabled, counted on 2026-07-28. The fleet view that lists them is generated rather than hand-maintained, on a rule we hold to hard: a loop missing from the view is a bug in the generator, never something to hand-prune.
So we solved one half of coverage. An agent of ours cannot be absent from the list, and that took a script. The other half, whether the file each row points at is still there, is not solved and I found that out while scoring this, which is further down.
Then we ran the fleet against the ten fields above, and here's what came back.

FieldWhat our fleet hasScoreOwnerNo such field. Of the 27 spec files that exist, 4 contain the word owner at all, and none of those names the loop's own owner: one names the owner of a downstream tracker, one is a template placeholder, one is a table header, one is a hand-off instruction0 of 29PurposeThe loop's name plus its task-file README. Not a field, and one loop's taskFile points at a README that is no longer on disk, so 28 are readable in one hop and one is not28 of 29, in proseReachesNot a field. Discoverable only by reading the spec's prose, which describes intent rather than reach0 of 29CredentialNot a field. Every loop runs as this machine's user and inherits what that user has, including one GitHub token on account JayZeeDesign whose scopes are gist, read:org and repo0 of 29ExpiryNot a field. 2 loops carry a stated finish line and both of those are paused0 of the 24 runningReviewedNot a field, and no review has been scheduled for any of them0 of 29EscalationNot a field. Each loop has a notify policy that reaches the machine's owner, which is one person for the whole fleetpartial, one contact for 29Kill switchYes. enabled is per loop, loopany edit <id> --json '{"enabled":false}' stops one, and 5 are paused right now29 of 29Autonomy rungNot a field. 11 of the 27 spec files that exist state a posting rule about themselves in prose, from "never post" to "auto-post". The other 16 state none0 of 29Action logPer run, not per write. Each record carries an id, timestamp, role, outcome, duration, cost, message, session id and declared statepartial, wrong granularity
One clean pass out of ten, and purpose missed being the second by a single row. Two partials. Six fields with no place in the system to put them.
That last part is the finding rather than the score. This isn't a case of fields left blank through neglect. The whitelist of every key a loop's config accepts has fourteen entries, and I read all fourteen: name, cron, timezone, notify, model, agent, allowControl, enabled, runAt, taskFile, goal, workflow, ui, stateSchema. There's no owner, no credential, no expiry, no review date, no escalation contact and no autonomy level, so a person who wanted to fill those fields properly has nowhere to write them and would end up keeping a second file by hand, which is the file that goes stale.
Purpose is the row I got wrong first time, and how I got it wrong is worth a paragraph. I scored it 29 of 29 because every loop has a taskFile and every taskFile is a README, so purpose is one hop away. Then I checked whether those files exist. Our 29 loops resolve to 28 unique paths, 27 of which are on disk. One loop points at a README that has been deleted, and a second path is registered twice under the same loop name. The CLI prints the path either way, because printing a string is not the same as resolving it, so a fleet view generated from that output shows 29 healthy rows. Coverage was the half I said we had solved. What we had actually solved was getting every loop into the list, which is a different thing from the item the list points at still being there.
Some of the other rows want a sentence more, starting with the ones I'd act on first.
Credential is the worst of them and it's structural rather than accidental. Every loop authenticates as this machine's user, so the blast radius of any one of them is the blast radius of all of them together. One token, repo scope, shared by 29 agents that do unrelated jobs, and per GitHub's own documentation that scope reaches every repository in every organization the account belongs to. A registry row for any single loop would have to name that token, and the row would immediately read as wrong, which is exactly the use I claimed for the file.
Expiry made me most uncomfortable. Two loops carry a finish line, both are paused, and so nothing currently running has a date on which someone is required to reconsider it. The agent in the thread had run every night since March. Ours have the same property and the same reason, which is that nothing in the system asks.
Then the action log, which is close but at the wrong granularity, and the distance is instructive. We get a run record with a session id, which tells you a run happened and lets you go read its transcript. It doesn't tell you which rows in which system that run changed, and reconstructing that is the hand-correlation of application logs the operator in the thread was already stuck doing. There's also a query ceiling: asking for 1,000 records on a loop with 128 returns 20, and the CLI says count: 20 of 128 total while it does it. On a loop that runs sixteen times a day, twenty records is about a day of readable history. Spoke two documented that cap and the null-cost fields on the same log, so I won't re-litigate them here beyond noting that a log with a twenty-record query window is not an audit trail.
One thing I want to be precise about, since it's the difference between a useful score and a self-flagellating one. None of the above means our loops are running wild. Their guardrails are real and they're hand-built per loop in that loop's spec file and its runner config. What the score says is that the guardrails aren't legible from outside the loop, so checking them means reading each spec file, which in practice means asking the person who already knows the loop.

Artifact 2: agents/OFFBOARDING.md
Retirement is where the whole thing gets tested, and it's the step people skip because the agent has already stopped mattering to them by the time they get to it.
The order below is chosen so that step 3 catches the failure in step 2.
markdown# Offboarding: <agent name>

Registry row: <link>            Owner at retirement: <name>
Reason for retirement: <one line: superseded, purpose gone, never worked, cost>

- [ ] 1. Stop it.
      <the kill switch from its registry row, run verbatim>
      Stopped at: <timestamp>

- [ ] 2. Revoke the credential. Revoke, not rotate.
      <where, and which credential. if it is shared, say so here and stop:
       a shared credential cannot be revoked and this agent is not offboarded,
       it is only switched off.>
      Revoked at: <timestamp>

- [ ] 3. Verify the revocation from outside.
      Run the agent's own first call with the old credential and watch it fail.
      Paste what you ran and what came back:
      <command>
      <output>
      A revocation you have not watched fail is a revocation you have assumed.

- [ ] 4. Archive the action log.
      <where it goes, who can read it, how long you keep it>
      The log outlives the agent. The questions about what it did arrive after
      it is gone, and that is the whole reason this step exists.
      Archived at: <path or URL>   Retained until: <date>

- [ ] 5. Close the registry row.
      Move it to the Retired section with today's date. Do not delete it.
      A deleted row is indistinguishable from an agent that was never registered.

## Left behind

<Anything you cannot answer about what this agent did, written down now, in plain
words, while you still remember. Reconstructing it later costs more and is worse.>

Retired: <date> by <name>

Step 2 has the trap in it and I've put the trap in the file. If the credential is shared with a human or with other agents, you cannot revoke it, and the honest thing the runbook can do at that point is refuse to call the agent offboarded. It's switched off. The credential is still live and still reaches everything it reached, and the only difference is that nothing is currently using it. Our fleet fails this step for all 29 loops, and it fails for the same reason the credential row scored zero.
Step 3 is the one to actually run. Watching the old credential fail converts an assumption into an observation, and the assumption is the kind that survives indefinitely because nothing contradicts it.
The "left behind" section is there because of a point the operator in that thread made himself, in a reply to somebody suggesting he add an approval gate. A gate added now, he said, does nothing for the four months his agent had already been writing, and that history stays opaque no matter what he adds. He's right, and it generalises past his case. Every control on this page is prospective. None of them reaches backwards, so the only thing available for the period before you started is writing down what you don't know, before the person who remembers moves on.

Artifact 3: The Action-Log Contract
The registry says what an agent may reach. This says what it actually did, and it's the difference between "probably fine" and "fine."
Append-only is a property of the writer, not a word you put on a heading, and the first draft of this contract failed its own label. It had one row per write, minted before the call with an outcome column filled in afterwards. That's a mutable row. A row the agent updates is a history the agent holds the pen on, which is the one property you need this file not to have.
So: two events per write, and a sink the agent can add to and nothing else.
codeINTENT   appended before the call. never updated, never deleted.
  op_id       uuid, minted before the call. not derived from the payload.
  ts          when the intent was recorded
  agent       the registry row this belongs to
  run_id      the run that decided on this write
  caused_by   the ticket, job, message or schedule slot that started the run
  target      the system, and the specific object
  before      what was there. a hash if it is large or sensitive.
  after       what it is being set to
  reason      the agent's own stated reason, in its words

OUTCOME  appended after the call, as a second event. never updated, never deleted.
  op_id       the same uuid. this is the only join between the two.
  ts          when the call returned
  result      applied | rejected | failed
  detail      the error, the affected row count, whatever came back

An intent with no outcome beside it is a finding rather than a gap in your data. It says a write was started and nothing came back, which is what a crash, a timeout or a killed process leaves behind. The mutable version records that same event as a row that simply never got updated, which is indistinguishable from a bug in the logger.
The writer is the other half, and without it the two-event shape is a convention rather than a control. Both events go somewhere the agent's own credential can append to and can't revise. On Postgres that's an INSERT grant on the log table with no UPDATE, no DELETE, no TRUNCATE and no ownership, and no ON CONFLICT DO UPDATE path in the code the agent runs. On a file it means the file isn't the agent's: it goes to a collector running as a different user, or to the platform's own append-only log service, and what the agent holds is a handle to a socket rather than to the history.
That has to be tested by attacking it, and the obvious version of the test does not test it. Revoking UPDATE and DELETE and then watching an insert succeed proves the insert works. It proves nothing about whether the history can be rewritten, because UPDATE and DELETE are not the only ways to rewrite it. TRUNCATE empties the table under a privilege of its own. The table's owner holds every privilege whether or not it was granted, so a writing role that owns the log is unconstrained no matter what the grants say. ON CONFLICT DO UPDATE revises a row through an INSERT statement. And a grant to PUBLIC, or to some other role, hands the same power back through a different login.
So tests/append-only-log-test.sql attempts every one of those as the writing role and fails, loudly, if any of them succeeds. Eleven checks, and a check it could not attempt counts as a failure rather than a skip:
text 1  INSERT intent event                            must be accepted
 2  INSERT outcome event                           must be accepted
 3  UPDATE a written event                         must be refused
 4  DELETE a written event                         must be refused
 5  TRUNCATE the log                               must be refused
 6  INSERT ... ON CONFLICT DO UPDATE               must be refused
 7  ALTER the log table                            must be refused
 8  writing role does not own the table            and is not a member of the owner
 9  writing role is not superuser or BYPASSRLS
10  no other role holds UPDATE, DELETE or TRUNCATE
11  both probe events still present afterwards

There are two ways to run that file. Use the first when you already have PostgreSQL:
bashpsql -v ON_ERROR_STOP=1 -f tests/append-only-log-test.sql

Use the bundled Node runner when you do not. Its PostgreSQL runtime is an explicit, test-only install, so a fresh checkout prints this same install command instead of an ERR_MODULE_NOT_FOUND stack trace:
bashnpm install --no-save --package-lock=false @electric-sql/pglite@0.5.4
node tests/run-append-only-test.mjs
node tests/run-append-only-test.mjs --overgranted

The first Node run requires all eleven checks to pass. The second is the negative control: the runner deliberately over-grants the writing role, requires six named checks to fail, and exits 0 only when that failure is observed. @electric-sql/pglite is kept out of this site's lockfile because the SQL remains the portable artifact and the package is needed only by the no-server runner.
Run against a log built to the contract, all eleven pass and the refusals are quoted from the server rather than assumed. Run against a role that kept the mutations, six fail and the message names each one, which is the half of the evidence that makes the other half worth anything: a test never seen to fail is not known to be a test. Both runs are in the receipts.
caused_by is what makes it an audit trail rather than a pile of events. Without it you can see that a row changed and you're back to correlating timestamps by hand, which is the exact position the operator in the thread described as his current state. With it, "why did this customer's balance change on the 14th" resolves to a ticket in one hop.
op_id is doing a second job, and it comes from a point raised on a different thread in this research: an approval layer stops one bad write from running unreviewed, and does nothing about the same write firing repeatedly because the agent looped or a retry landed after a timeout. That's an idempotency problem wearing an approval problem's clothes. If the applier checks op_id against already-executed ops before running anything, a duplicate becomes a no-op, and you get that property from a field you were writing anyway.
Some limits belong on the file itself, because a log that overpromises is its own hazard.
The agent's stated reason is a lead and not evidence. It's the agent's account of itself, produced by the same process that produced the write, and it's genuinely useful for finding the run you want to go read. It is not a finding about what happened.
Append-only stops the agent revising its history and does nothing about what the agent chose to write down in the first place. An agent that never appends an intent event leaves no trace at all, and no grant on the log table changes that. What the contract buys you is that the events you do have can't be edited afterwards, which is a narrower claim than "the log is complete" and worth not confusing with it.
And the retroactive gap doesn't close. Turning this on today gives you an answer from today. For everything before, the honest artifact is the "left behind" section of the offboarding file.

Artifact 4: The Autonomy Ladder
The pillar states this as a principle in three sentences: autonomy is granted per function, never to the system, and a function climbs on evidence and gets walked back down when the evidence stops holding. As a principle it's easy to agree with and impossible to check. Here it is as a table you can put a date against.

RungWhat it meansPromotion evidence, written before you promoteDemotion trigger1. Human onlyThe agent doesn't touch this function. It may read and suggestn/a. Nothing is promoted into rung 1n/a2. Drafts, a human sendsEvery output passes through a person who can edit it before it leavesA named person has read N consecutive drafts and states what a bad one looks like. Write the number and the person in the rowAny draft ships that the reviewer would not have sent, and the reviewer says so3. Executes, a human reads every actionThe agent acts. A human reads every action within a stated window, and the window is part of the rungThe reviewer at rung 2 stopped editing. Record how many consecutive drafts went out unedited, and over what periodTwo actions inside one window that needed correcting, or one that reached a system outside its registry row4. Autonomous inside stated limitsThe agent acts and nobody reads every action. The limits are the control, so they have to be written before you get hereRung 3 ran for a stated period with the reads happening and nothing corrected, and the limits are in config rather than proseAny action outside the stated limits, or a limit nobody can point at in a config file
Two rules make the table work, and without them it's decoration.
Promotion evidence gets written down before the promotion, not after. Evidence assembled afterwards is a justification, and a justification starts from the answer it wants. The row above asks for numbers you fill in prospectively, so a promotion you can't justify in advance doesn't happen.
The demotion trigger has no natural moment at which it gets written, which is why the table asks for it in the same row as the promotion. A rung with no written trigger only goes up, and a ladder that only goes up is a ratchet. Write it while you can still imagine failing.
The failure this catches doesn't look like an agent doing something wrong. A marketing agency running fourteen agents in production described a spend-monitoring agent that detected a client overspending, flagged it, specified the escalation action, then reported "escalation overdue" every day for 17 days without executing it. Nothing was broken. The specification was being treated as documentation rather than as executable logic, and nobody had verified the execution path end to end. That agent was sitting at rung 3 on paper with nobody performing the reading that rung 3 is defined by, which is the rung that quietly becomes rung 4.
Their other story is the coordination version: two agents tracking project deadlines from different data sources, each correct in isolation, producing two different due dates for the same project in one morning briefing. Their conclusion from both was organizational rather than technical: "one seat, one owner." Worth taking with its provenance attached, since that post ends in a pitch for the author's own product, and the reason to trust the two war stories anyway is that they're specific, they're consistent with each other, and the conclusion lands in the same place the thread this page opens on did, from a completely different direction.
