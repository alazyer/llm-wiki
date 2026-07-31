---
title: "Why Your Agent Bill Is WrongNEW"
source_url: "https://www.aibuilderclub.com/blog/ai-agent-runaway-cost"
ingested: 2026-07-30
blog: "AI Builder Club"
published: "2026-07-27"
---
## Source Metadata
- **Blog:** AI Builder Club
- **Published:** 2026-07-27
- **URL:** https://www.aibuilderclub.com/blog/ai-agent-runaway-cost
- **Fetched:** 2026-07-31

## Article Content

AI Agent Runaway Cost: Why Your Bill Is Wrong (2026)

Someone on the anthropics/claude-code tracker went to dinner with his family. A batch classifier had been denied, and the agent started spawning monitors to get around it. He came back to a reported $900 in extra usage credits.
That story is easy to file under carelessness, so here's the one that isn't. A different operator asked Claude to plan the budget for the agent stack he was about to run. It projected $1.73 a month and told him a $2 hard cap was more than enough, with about 15% headroom. He hit the cap in four days and his crons stopped. When he went through the accounting himself, thinking tokens were 47% of his spend, the single largest line, and they had never come up in the budget planning at all.
Both of those are self-reported by users on a public tracker, so treat the dollar amounts as claims. The second one is the subject of this page. His agent did roughly what he asked it to. The number he planned against had three whole categories missing from it, and he only found that out from the invoice.
This page ships three things you can copy. A table of the cost categories that don't appear in session metrics, so you know what to ask your own runner about. scripts/cost-reconcile.sh, which counts priced runs against retained runs and refuses to print a total when they don't match. And an agent definition file plus the orchestration rule that together stop recursive fan-out, which is the failure with the largest numbers attached to it.
This is a measurement page. What to do about a bill you can already see is agent reliability and cost control, which covers the levers. This one is about whether the number you're looking at is real. It goes deeper than the cost section of how to become an AI-native company, which prices one well-behaved loop and stops there.

Artifact 1: The Invisible-Categories Table
Your session view reports on a session. The invoice covers everything billed to the account, including work no session ever saw, and nothing in the session view flags the difference.
The first artifact is a table, because there's nothing to install here. It's the list of questions to put to whatever you run.
Here are four categories sitting in that gap. They come from two different operators, so the table names which one reported each. The first three are from the issue 49745 write-up: his figures, for his own stack, over his 69 sessions. The runtime he was budgeting was a small model on a cheap VPS, with Claude Code as the planner that produced the estimate. The fourth belongs to a different operator, from a thread on r/AI_Agents, on his own stack, and it played no part in the $2 cap in the story above. None of these figures is a rate.

CategoryWho reported itWhat that operator foundWhy session metrics miss itThinking tokensIssue 4974547% of his total spend, the largest single lineBilled as output, not surfaced as its own line, and absent from the budget the model itself proposedSilent auxiliary callsIssue 49745249 across 69 sessions, for title generation, auto-detect and vision auto-detectThe tool makes them, not the session, so no session records themMCP tool bloatIssue 49745One 29-tool server took his per-session input from about 12K to about 40K tokensThe tool definitions are in the context window before you type anythingIdle re-wakesA different operator, on r/AI_AgentsA run that exits without recording a disposition gets treated as incomplete and re-wokenThe re-run is a new run, and looks like work

The last row travels furthest, and it came from somewhere else. A second operator, running six agents on his own stack, traced his bill to an orchestrator that woke, read the plan, queried assignments, extracted facts, concluded there was nothing to do, and exited without recording an explicit disposition of done, blocked or handoff. The system read the missing disposition as an incomplete run and woke it again. Nothing-to-do could re-fire and re-run. He reports an idle heartbeat going from about $0.40 to about $0.06, which is his figure for his six agents, and he attributes it to fixing the exit logic and right-sizing the model together rather than to the disposition alone. His numbers and the $2 cap above belong to two separate bills.
That's why the guardrail config in the pillar carries a disposition: required: true line. Whether your own runner does something similar with a run that exits saying nothing is a question worth putting to it before you plan a budget around its numbers.

Artifact 2: The Reconciliation Script
The categories above are what your vendor doesn't show you. This next part is what your own runner doesn't show you, and it's worse, because you built that part.
Our loop runner's log lies in three specific ways. All three are reproducible on this machine as of 2026-07-27, against loopany v0.16.0, and all three have the same shape: a query that returns less than you asked for and exits 0.
Runs can carry no price at all. The costUsd field is nullable, and a null is not a zero. On our fleet on 2026-07-27 the CLI returned 255 run records across 29 loops, and 85 of them had no cost recorded. 38 of those 85 had run for longer than five minutes. What those 38 cost is an open question, because a duration is not a price and the log declines to give one. The sharpest single case is a visibility-check run on 2026-07-27 that ran for 7,800,589 milliseconds, about two hours ten minutes, collected 257 records by its own report, and carries costUsd: null. Its own run message explains why. The server had already reclaimed the run by the time the terminal tried to report, so every CLI verb returned a conflict. Two hours of metered model work, and no price on the record for any of it.
Then the window caps itself. loopany log <id> --json --limit 1000 returns 20 rows, and so do --limit 100 and --limit 21. The JSON carries no total anywhere in it, so a 20-element array is what a complete history and a truncated one both look like. Only the human-readable output says count: 20 of 71 total. Sum the JSON for that loop and you've priced 20 runs out of 71 while your script reports a clean number.
The third one took me longest to pin down, because it refuses to hold still. loopany log <id> --json > file.json writes 74,017 bytes for one of our loops, 8 records. Piped into anything, that same command came back cut to 65,536 bytes on 28 of 40 attempts and came back whole on the other 12. Ask for more history and it gets worse. At --limit 1000 the file redirect writes 190,249 bytes and 20 records, while 40 piped attempts came back at either 65,536 or 131,072 bytes, and not one of them came back whole. Every attempt exited 0, cut or not.
Those two cut points are pipe-buffer multiples, 64KiB and 128KiB, so what's happening is a write that stops on a buffer boundary and reports success. Every truncated result across those 80 reads was invalid JSON rather than a shorter valid array, which is the one piece of luck in it: jq fails loudly instead of quietly pricing 3 runs out of 8. Put 2>/dev/null on the pipeline and you've taken the loud failure away and left yourself an empty variable that arithmetic treats as zero. The intermittency is what makes this worth a check rather than a workaround. A pipeline that always cut at 64KiB is one you'd notice in an afternoon and design around. One that hands you the whole history 12 times out of 40 is one you'll come to trust.
So the script never pipes the JSON, and it checks five things before it prints anything. The first is the plain one that's easiest to skip: both fetches have to have exited 0. A runner that dies partway can leave a complete, well-formed JSON array on stdout describing a fraction of the history, and every later check will wave that through.
bash#!/usr/bin/env bash
#
# cost-reconcile.sh - print a loop's true metered cost, or refuse.
#
# The point of this script is the refusal. Summing the cost column of a run log
# gives you a number whether or not the log is complete, and a partial total
# looks exactly like a complete one. This checks five things before it prints
# anything, and exits non-zero if any of them fail:
#
#   1. Both fetches exited 0. A producer that failed never yields a total, no
#      matter how much well-formed JSON it managed to write before it died.
#   2. The runner answered, and answered with a run log rather than an error.
#   3. The JSON parses completely. A stream cut off mid-record is rejected,
#      not silently re-parsed as fewer runs.
#   4. The number of runs returned equals the number of runs the runner says it
#      is retaining. Not "at least": equal. Fewer means a capped window, more
#      means the two answers disagree and neither is trustworthy.
#   5. Every run in that window carries a price. One unpriced run means the sum
#      is a floor, not a total.
#
# Written against loopany v0.16.0. Two behaviours of that CLI are the reason
# checks 3 and 4 exist, and both are reproducible on this machine:
#
#   - `loopany log <id> --json` truncates intermittently when stdout is a pipe,
#     and exits 0 when it does. On one loop whose file redirect writes 74,017
#     bytes, 28 of 40 piped reads came back cut to 65,536 bytes and the other 12
#     came back whole. At `--limit 1000`, where the file redirect writes 190,249
#     bytes, 40 piped reads came back at either 65,536 or 131,072 bytes and not
#     one came back whole. The cuts land on pipe-buffer boundaries, 64KiB and
#     128KiB, and every truncated result was invalid JSON rather than a shorter
#     valid array. So `loopany log <id> --json | jq ...` corrupts some of the
#     time, at a boundary that moves, with nothing in the exit status to say so.
#     This script never pipes the JSON.
#   - `--limit` is capped at 20 rows and the JSON carries no total, so a loop
#     with more retained runs than that returns a short array that looks whole.
#     Only the human-readable output declares "count: N of M total", so this
#     script reads both and compares them.
#
# Usage:  scripts/cost-reconcile.sh <loop-id> [<loop-id> ...]
#
# Exit:   0  every named loop reconciled, totals printed
#         1  at least one loop refused
#         2  bad usage or a missing dependency
#
# Porting it: LOOPANY_BIN swaps the runner binary. If your runner is not
# loopany, the two functions to rewrite are fetch_json and fetch_declared_total.
# Both must return the producer's own exit status. Keep the five checks.

set -euo pipefail

LOOPANY_BIN="${LOOPANY_BIN:-loopany}"
FETCH_LIMIT="${FETCH_LIMIT:-1000}"

die() {
  printf 'cost-reconcile: %s\n' "$1" >&2
  exit "${2:-2}"
}

require() {
  command -v "$1" >/dev/null 2>&1 || die "$1 is required and was not found on PATH"
}

# Fetch the run log as JSON into a file. Never into a pipe: see the header.
# Returns the runner's own exit status. Do not swallow it.
fetch_json() {
  local loop="$1" dest="$2" rc=0
  "$LOOPANY_BIN" log "$loop" --json --limit "$FETCH_LIMIT" >"$dest" 2>"$dest.err" || rc=$?
  return "$rc"
}

# Fetch the human-readable header, which is the only place the runner states
# how many runs it is retaining in total. Returns the runner's exit status too.
fetch_declared_total() {
  local loop="$1" dest="$2" rc=0
  "$LOOPANY_BIN" log "$loop" --limit "$FETCH_LIMIT" >"$dest" 2>"$dest.err" || rc=$?
  return "$rc"
}

# Print up to four lines of whatever the runner said, indented, from stdout if
# it wrote anything there and from stderr otherwise.
echo_runner_output() {
  local out="$1"
  if [ -s "$out" ]; then
    head -4 "$out" | sed 's/^/         > /'
  elif [ -s "$out.err" ]; then
    head -4 "$out.err" | sed 's/^/         > /'
  fi
}

reconcile_one() {
  local loop="$1"
  local workdir json text
  local json_rc=0 text_rc=0
  workdir="$(mktemp -d)"
  # shellcheck disable=SC2064
  trap "rm -rf '$workdir'" RETURN
  json="$workdir/runs.json"
  text="$workdir/runs.txt"

  fetch_json "$loop" "$json" || json_rc=$?
  fetch_declared_total "$loop" "$text" || text_rc=$?

  # Check 1: did both fetches exit 0?
  # This is first because it is the only check that cannot be fooled by
  # well-formed output. A runner that dies partway through can leave a complete
  # JSON array on stdout describing a fraction of the history, and every later
  # check would pass it. A non-zero exit means no total, whatever was written.
  if [ "$json_rc" -ne 0 ] || [ "$text_rc" -ne 0 ]; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         the runner exited non-zero (json %s, text %s). Whatever it wrote before\n' \
      "$json_rc" "$text_rc"
    printf '         exiting is not a run log. No total.\n'
    if [ "$json_rc" -ne 0 ]; then
      echo_runner_output "$json"
    else
      echo_runner_output "$text"
    fi
    return 1
  fi

  # Check 2: did we get anything, and is it a run log rather than an error?
  # On loopany v0.16.0 an unknown loop id exits 1 and prints `error: "..."` plus
  # `code: NOT_FOUND` on stdout, so check 1 already catches that case. The grep
  # stays for the case an exit code cannot catch: a runner that reports a
  # failure in its output while still exiting 0.
  if [ ! -s "$json" ]; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         the runner exited 0 and returned nothing. No total.\n'
    return 1
  fi
  if grep -qE '^(error|code):' "$json"; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         the runner exited 0 but reported an error instead of a run log:\n'
    echo_runner_output "$json"
    printf '         No total.\n'
    return 1
  fi

  # Check 3: does the whole document parse? A truncated array fails here, which
  # is the entire reason this is a separate check from the count.
  if ! jq -e 'type == "array"' "$json" >/dev/null 2>&1; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         the run log did not parse as a complete JSON array (%s bytes read).\n' \
      "$(wc -c <"$json" | tr -d ' ')"
    printf '         A truncated stream is not a short history. No total.\n'
    return 1
  fi

  local returned priced unpriced sum declared
  returned="$(jq 'length' "$json")"

  if [ "$returned" -eq 0 ]; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         the run log is an empty array. Either the loop has never run or the\n'
    printf '         query returned nothing. Both are indistinguishable from here. No total.\n'
    return 1
  fi

  # Reject records that are not shaped like runs before trusting the cost field.
  if ! jq -e 'all(.[]; type == "object" and has("costUsd"))' "$json" >/dev/null 2>&1; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         at least one record is not a run object with a costUsd field.\n'
    printf '         The log is not the shape this script knows how to price. No total.\n'
    return 1
  fi

  priced="$(jq '[.[] | select(.costUsd != null and (.costUsd | type) == "number")] | length' "$json")"
  unpriced=$((returned - priced))
  sum="$(jq -r '[.[] | select(.costUsd != null and (.costUsd | type) == "number") | .costUsd] | add // 0 | . * 100 | round / 100' "$json")"

  # Check 4: is the window we read exactly the whole retained window?
  declared="$(sed -n 's/^count: [0-9][0-9]* of \([0-9][0-9]*\) total$/\1/p' "$text" | head -1)"
  if [ -z "$declared" ]; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         could not read a "count: N of M total" line, so there is no way to tell\n'
    printf '         whether the %s runs returned are the whole history. No total.\n' "$returned"
    return 1
  fi
  if [ "$returned" -lt "$declared" ]; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         %s of %s retained runs returned. The runner caps the page below its own\n' \
      "$returned" "$declared"
    printf '         history, so the visible rows are a window, not the log.\n'
    printf '         Priced rows in the window sum to $%s. That is a floor for the window\n' "$sum"
    printf '         alone, and no bound at all on the loop. No total.\n'
    return 1
  fi
  if [ "$returned" -gt "$declared" ]; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         %s runs returned against %s the runner says it retains. The two answers\n' \
      "$returned" "$declared"
    printf '         disagree, so neither one describes the history. No total.\n'
    return 1
  fi

  # Check 5: is every run in that window priced?
  if [ "$priced" -eq 0 ]; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         %s runs, and the log records a price for none of them. A null is the runner\n' "$returned"
    printf '         declining to say, so there is nothing here to add up and nothing to bound it\n'
    printf '         with either. No total.\n'
    return 1
  fi
  if [ "$unpriced" -gt 0 ]; then
    printf 'REFUSED  %s\n' "$loop"
    printf '         %s of %s runs priced, %s with no cost recorded.\n' "$priced" "$returned" "$unpriced"
    printf '         Priced rows sum to $%s, which is a floor, not a total. A null price is the\n' "$sum"
    printf '         runner declining to say what a run cost, not a record of zero, so what sits\n'
    printf '         above that floor is unknown here rather than small. No total.\n'
    return 1
  fi

  printf 'OK       %s\n' "$loop"
  printf '         %s of %s runs priced. Metered total: $%s\n' "$priced" "$declared" "$sum"
  jq -r '.[] | "         \(.ts[0:16] | sub("T"; " "))  \(.role)  \(.outcome // "-")  $\(.costUsd)"' "$json"
  return 0
}

main() {
  [ "$#" -ge 1 ] || die "usage: cost-reconcile.sh <loop-id> [<loop-id> ...]"
  require jq
  require "$LOOPANY_BIN"

  local failures=0
  for loop in "$@"; do
    reconcile_one "$loop" || failures=$((failures + 1))
    printf '\n'
  done

  if [ "$failures" -gt 0 ]; then
    printf '%s of %s loops refused. Do not add the printed floors together.\n' "$failures" "$#" >&2
    exit 1
  fi
  exit 0
}

main "$@"

Save it as scripts/cost-reconcile.sh, chmod +x, and pass it loop ids. It needs jq and your runner on PATH, no key and no account anywhere. shellcheck is clean on it.
On a loop whose history is whole, it prints the total and the rows behind it:
textOK       loop-mry8g8f2-5daaf53d
         6 of 6 runs priced. Metered total: $19.06
         2026-07-27 00:29  evolve  evolve  $3.102419
         2026-07-27 00:21  exec  exec  $4.0711485
         2026-07-26 03:23  exec  error  $3.2364104
         2026-07-25 04:03  exec  error  $3.4116462
         2026-07-24 01:14  evolve  evolve  $0.953405
         2026-07-24 01:12  exec  exec  $4.28062

On one whose history isn't, it says what's missing and exits 1:
textREFUSED  loop-mr8hfir9-a47fa334
         3 of 6 runs priced, 3 with no cost recorded.
         Priced rows sum to $10.68, which is a floor, not a total. A null price is the
         runner declining to say what a run cost, not a record of zero, so what sits
         above that floor is unknown here rather than small. No total.

The refusal is the feature. The obvious one-liner, loopany log <id> --json | jq '[.[].costUsd // 0] | add', prints 10.683629 for that same loop, to seven decimal places, with nothing anywhere to tell you it's short by half the history.
Two design choices worth stating, because without them I'd have stopped running the thing.
The script prints a floor when it refuses, and labels it a floor in the same sentence. A refusal with no number at all is a script I'd have abandoned by the third loop. A floor you can't mistake for a total is still worth having, and the closing line tells you not to add several of them together, because a sum of floors isn't a floor on the sum in any way you'd want to rely on.
The second choice is that costUsd: 0 counts as priced and costUsd: null doesn't. Those look similar and mean opposite things. One of our runs failed on an expired OAuth session and recorded exactly 0, which is a real and correct price for a run that never reached the model. A null is the runner declining to say.
