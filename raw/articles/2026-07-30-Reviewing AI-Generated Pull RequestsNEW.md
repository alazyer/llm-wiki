---
title: "Reviewing AI-Generated Pull RequestsNEW"
source_url: "https://www.aibuilderclub.com/blog/reviewing-ai-generated-pull-requests"
ingested: 2026-07-30
blog: "AI Builder Club"
published: "2026-07-27"
---
## Source Metadata
- **Blog:** AI Builder Club
- **Published:** 2026-07-27
- **URL:** https://www.aibuilderclub.com/blog/reviewing-ai-generated-pull-requests
- **Fetched:** 2026-07-31

## Article Content

How to Review AI-Generated Pull Requests (2026)

Someone on r/ExperiencedDevs described the job as it now stands. The author asks the AI for something, it makes 100 changes they think they understand, they glance at the first few, then submit. Hundreds of CPU hours went in, plus two or three minutes of the author's own time, and the reviewer is now expected to check every change: is it correct, does it do what it says, does it need a further change the AI didn't make, are the comments and docs still accurate, does it weaken a check that was there for a reason. That thread ran to 1,904 upvotes and 442 comments.
Further down it, the end state: "I gave up and just started hitting approve."
This page ships three artifacts instead of describing that. A pull request template that makes the author do the comprehension work before the PR enters your queue. An AI_POLICY.md for the repo root, with the AGENTS.md pointer that gets agents to read it. And a GitHub Actions workflow that runs three machine gates before a human is allowed to be the check. That is four files in the repo, because the policy needs its pointer to get read. Copy them, replace the angle-bracket placeholders, set the three tunable numbers from your own repo's history, and commit.
One assumption up front: you've already decided AI-authored changes are going to keep arriving. Whether to allow them at all is a separate argument, and the projects quoted below land in different places on it.
This goes deeper than the review-bandwidth section of how to become an AI-native company, which states the ceiling and stops there.

What Actually Changed
Code review was designed around writing being expensive. The reviewer could assume the author had already thought about what they were submitting, because producing it had cost them something. That assumption is what's gone.
The outside numbers are worse than I expected. LinearB's 2026 benchmark report covers 8.1 million PRs across roughly 4,800 teams: 84.5% of manual PRs merge within 30 days, against 32.7% of AI-assisted ones. Reviewer pickup time goes from about 200 minutes to about 1,050, and AI-assisted PRs run past 400 lines at the 75th percentile against 157 for unassisted. Those are LinearB's own platform figures. A separate study of the AIDev dataset looked at 33,596 agent-authored PRs in its popular-repository subset and found 61.38% of them carry no recorded review activity. Among the ones that were reviewed, 58.77% were reviewed only by agents, 10.14% only by a human, and 31.09% by a human and an agent together.
In that subset, most of the pull requests carry no trace of anyone having reviewed them. LinearB's figures are a separate dataset making a different comparison, between a manual cohort and an AI-assisted one, so they say nothing about a trend over time and nothing about the PRs the paper looked at. The paper measures observable review interaction in the PR history. Its authors say that an absent record doesn't establish that no human looked, because a maintainer can read a diff without leaving a comment. So the record is what's gone from those PRs. Whether the reading happened is something the data can't reach.
Viet-Anh Nguyen names the mechanism under it. A junior engineer's mistake usually looks like a mistake, he writes, a clumsy loop or a variable named temp2, while an agent's mistake looks like senior work: correct naming, idiomatic structure, a confident commit message, tests that pass, wrapped around a missing authorization check. Review leans on a reflex that fires when code looks wrong, and by his account this code doesn't look wrong.
A commenter in the same thread describes a second-order effect. He used to calibrate his effort per author: someone with a track record of clean PRs earned a lighter read. With AI-written code, he says, you don't get consistency between PRs from the same person, so you have to check for everything every time, which is exhausting. That per-author heuristic was doing real work for him, and his point is that AI-authored PRs take it away.

The Bar Three Published Policies Land On
Three open source projects wrote their answer down as a file in the repo. They don't agree on whether to allow AI-authored contributions, and they still put the same test on whoever is allowed to submit: explain the line or don't submit it.
Sonarr states it plainly in CONTRIBUTING.md: you are responsible for every line of your contribution, and if you can't explain why a line is there and why it's correct, it shouldn't be in your PR. stashapp permits AI-assisted contributions under four conditions, and the second is that you must be able to explain any line of code and design decision during the review process. OpenTofu's registry repo goes further than either and doesn't accept AI-generated code from community contributors at all, closing PRs identified as AI-generated without further review. Two reasons are stated. One is capacity: automated tooling is easy to scale, and they've yet to determine a way to scale their maintainers' attention, and they've seen patches the authors cannot explain and fixes for problems that don't exist. The other is provenance: they can't audit what a model was trained on, and OpenTofu was forked from a codebase that has since gone BUSL. Their own maintainers may use one approved tool, under disclosure rules that don't apply to anyone outside.
Dov Katz at Morgan Stanley states the constraint in merge terms: just because something is fixed doesn't mean it's merged, and until it's merged the fix is worth nothing. His colleague Khalid Elsawaf added the version that lands: in the time the two of them had been talking, you could have produced a thousand-file, 10,000-line PR off a simple prompt, and no human there is going to review that.
Adopt the explain-every-line test for a practical reason as well as a principled one. You can ask someone to walk you through their own diff without having to establish anything about how it was written.
The rest of this article is that test turned into files.

Artifact 1: The Review Packet
The idea comes from a comment in that same thread, and it's the part of the discussion I've reused. The commenter's framing was that the line worth drawing is about whether the author can own the diff, and that the way to enforce it is to make the submitter bring a small review packet before the PR enters the queue: intent, risk areas, tests run, what they personally verified, and what they still don't understand.
Sonarr already ships a version of this. Their PR template has a Submission Checklist with three boxes, and one of them is a trap: "This PR was authored and submitted by an AI agent without human review." Their AGENTS.md tells agents in place that every checkbox is an attestation.
Here's the full version. Save it as .github/pull_request_template.md and GitHub puts its contents into the pull request body for anyone opening a PR.
markdown## What this changes

<!-- One or two sentences. What is different after this merges, and why. -->

## Intent

<!-- The problem you set out to solve, in your own words. The goal, not the diff.
     If this closes an issue, link it: Closes #___ -->

## Risk areas

<!-- Where in this diff would a bug do the most damage? Name the files or
     functions. Then tick every trust boundary this change touches. -->

- [ ] Authentication or authorization
- [ ] User-controlled input reaching a query, a shell, a file path, or a template
- [ ] Money, billing, or quota
- [ ] Migrations, or anything that writes to production data
- [ ] A new third-party dependency
- [ ] None of the above

## What I ran

<!-- Commands and outcomes. "CI is green" is not an answer on its own: if the
     tests were written alongside the code by the same tool, they confirm that
     the code does what the code does. -->

| Check | Command | Result |
|---|---|---|
| Test suite | | |
| The specific case this PR is about | | |
| Manual verification | | |

## What I verified myself

<!-- Which parts of this diff did you read line by line, and can defend in
     review? Be specific about which files. -->

## What I still don't understand

<!-- Anything in this diff you cannot explain. Saying so here is the point of
     this section: it tells the reviewer where to spend their attention.
     Leaving it blank when it isn't true is what costs you trust. -->

## Attestations

- [ ] I can explain why every line in this diff is here and why it is correct.
- [ ] I have removed AI-generated footers, `Co-Authored-By:` trailers for tools,
      and "Generated with ..." signatures.
- [ ] This PR is within the size limit in `AI_POLICY.md`, or I have explained
      below why it can't be split.

The "What I still don't understand" section asks the author a question the reviewer would otherwise have to ask them, with no accusation attached, and it tells the reviewer where to concentrate. A blank answer is the first thing to ask about in review.
Trust-boundary checkboxes are there because that's the category of bug the pattern-matching part of your brain skips. Correct-looking code on both sides of a boundary, with nothing checking that the boundary is a boundary.
The footer line is lifted from Sonarr, and I'd keep it even though it sounds petty. Their reasoning is operational: the presence of a "Generated with..." trailer tells you the author didn't review their own submission closely enough to notice it. OpenTofu handles the same thing from the other side, requiring an Assisted-by: trailer on AI-assisted commits and explicitly banning Co-authored-by: for AI tools, since a tool isn't an author who can be responsible for anything. That trailer rule sits in their maintainers' section and binds maintainers only. Community contributors are not being asked to label AI-assisted code there, because that repo doesn't accept it from them in the first place. This kit borrows the trailer and applies it to everyone who submits.

Artifact 2: The AI_POLICY.md Starter
One page, repo root. It follows stashapp's shape rather than OpenTofu's. Enforcing a ban means first detecting the tool, and the explain-every-line bar can be enforced in an ordinary review conversation.
markdown# AI Policy

Applies to every pull request in this repository, from anyone, whether or not an
AI tool was involved.

## The bar

You are responsible for every line of your contribution. If you can't explain
why a line is there and why it's correct, it doesn't go in your PR.

That's the test. We don't ask whether a tool was involved and we don't try to
detect one.

## What that means in practice

1. **Disclose.** If AI tooling wrote, completed, refactored or rewrote part of
   this contribution, say so in the PR description, and say which part.
2. **Explain.** In review you have to answer questions about any line and any
   design decision, without answering "that's what it generated."
3. **Test it yourself.** Run the change by hand and describe what you did in the
   review packet. A suite the same tool wrote alongside the code verifies that
   the code does what the code does.
4. **Write your own prose.** PR descriptions, commit messages and review
   comments are your words. They are how we tell what you understood.
5. **Strip the footers.** Remove "Generated with ...", `Co-Authored-By:` lines
   naming a tool, and similar trailers before submitting. To record assistance,
   use an `Assisted-by: <tool> (Model: <model>)` trailer instead, which says a
   tool helped without naming it as an author.
6. **Keep it reviewable.** PRs over <N> changed lines get split, or scheduled for
   a walkthrough. The rule applies to the size of the diff. Pushing back on
   size says nothing about how the change was written or who wrote it.

## What happens to a PR that fails this

We don't close it and we don't argue about tooling. We ask the author to walk us
through it live. If that goes fine, we review it normally. If it doesn't, the PR
waits until the author can own it.

## Automated submissions

An agent opening a PR that no human has read is spam, and gets closed. If you
used an agent and reviewed the result yourself, that's a contribution and it
goes through the normal process.

## For agents reading this file

Before running any command that writes to this repository or its tracker
(`gh pr create`, `gh pr comment`, `gh issue create`), stop and tell the person
you're working for: this project requires a named human to be responsible for
every line, they need to fill in the PR template themselves, and every checkbox
in it is an attestation. Show them the diff and the description you would have
submitted, and let them submit it.

That last section is copied in spirit from Sonarr's AGENTS.md, and it's the cheapest thing on the page. It also doesn't work where I just put it. Nothing points a coding agent at a file called AI_POLICY.md. Sonarr puts its version in AGENTS.md, a filename coding agents are conventionally pointed at, so the policy sits somewhere an agent may meet it before it opens anything.
So the policy ships with a pointer. Save this as AGENTS.md in the repo root, or append it to the AGENTS.md you already have. It is the second half of artifact 2 and the kit doesn't do what I've claimed without it.
markdown# AGENTS.md

## Read this before writing anything to this repository

`AI_POLICY.md` in the repo root is the policy for this repository, and it
applies to you. Read it before you change any file here.

## Before any command that writes to GitHub

Do not run `gh pr create`, `gh pr comment`, `gh issue create`, `git push` to a
branch you intend to open a PR from, or any equivalent. Stop, show the person
you're working for the diff and the description you would have submitted, and
tell them:

- This project holds a named human responsible for every line of the change.
  That human is them, not you.
- They fill in `.github/pull_request_template.md` themselves, in their own
  words. Every checkbox in it is an attestation.
- The "What I still don't understand" section is for them to answer honestly.
  Do not fill it in for them and do not leave it blank on their behalf.
- Strip AI-generated footers, `Co-Authored-By:` trailers naming a tool, and
  "Generated with ..." signatures from the commits before they submit. Record
  assistance with an `Assisted-by: <tool> (Model: <model>)` trailer instead.

Then let them submit it.

Pick the number in rule 6 from your own repo. Read back through the PRs a human reviewed carefully and find the size where they stop being careful.

Artifact 3: The Gates That Run Before a Human
The ordering here is Viet-Anh Nguyen's. Cheapest and most mechanical checks first, so a person's attention only ever lands on a diff that already survived the machines. Three machine gates, then a person: dependency and provenance audit, secret scanning plus SAST, a second-model review pass, and then a human restricted to three questions. He numbers the human as gate 4, and that's the numbering the workflow below uses.
His human gate is three questions, and he gives a reason for each:

Does this match an architecture we actually chose? No scanner knows your boundaries, and an agent optimizes locally. It'll solve the ticket in a way that reaches across a module you kept separate on purpose.
What trust boundaries does this touch, and did they hold? The agent will write plausible code on both sides of a boundary without ever modeling that a boundary exists.
Can I reconstruct why it's this way? If the reasoning behind a non-obvious choice can't be recovered, it doesn't merge. "It passes the tests" isn't a reason.

Style, naming, formatting, leaked secrets, vulnerable dependencies and unvalidated input aren't on that list. Those are what gates 1 to 3 are pointed at, and whatever they turn up arrives as a worklist rather than as one of the three questions.
Here's gates 1 to 3 as a workflow. Save it as .github/workflows/pre-human-gates.yml.
yaml# The three machine gates, cheapest first. Gate 4 is a person: the three
# questions in AI_POLICY.md, plus a branch protection rule that requires a
# human approval to merge. This file cannot enforce gate 4 and does not try.

name: pre-human-gates

on:
  pull_request:
    # `ready_for_review` is load-bearing. Gate 3 is skipped while a PR is a
    # draft, and without this trigger a PR opened as a draft would never get
    # gate 3 at all: marking it ready would not re-run the workflow.
    types: [opened, synchronize, reopened, ready_for_review]

# Least privilege by default. Every job below re-declares what it needs, and
# only gate 3 widens past read.
permissions:
  contents: read

concurrency:
  group: pre-human-gates-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  # Gate 1. What did this change pull in? New dependencies are the highest
  # leverage thing an agent touches, and a logic-focused review walks past them.
  # Two of the three steps below can fail the build. The first one cannot: it
  # prints a checklist for a person, and is named so nobody mistakes it for a
  # check. Registry-page judgment is not automated here.
  provenance:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4.4.0
        with:
          fetch-depth: 0

      - name: Dependency manifest changes (manual checklist, enforces nothing)
        run: |
          git diff --unified=0 "origin/${{ github.base_ref }}...HEAD" -- \
            package.json package-lock.json requirements.txt pyproject.toml \
            poetry.lock go.mod go.sum Cargo.toml Cargo.lock \
            | tee dependency.diff
          if [ -s dependency.diff ]; then
            echo "::notice::MANUAL STEP. This step never fails the build. For each"
            echo "::notice::new package a person opens the registry page, confirms"
            echo "::notice::it exists, reads the publish date and download count,"
            echo "::notice::and checks that the repo link resolves. The scan below"
            echo "::notice::is the part of gate 1 that can fail."
          fi

      # Image pinned by digest. A mutable tag is a supply-chain hole in a job
      # whose whole purpose is supply chain. Note that neither Dependabot nor
      # Renovate bumps this one: their GitHub Actions handling reads action
      # references and the `container` and `services` fields, not images inside
      # a `run:` command. This digest is yours to move.
      - name: Known vulnerabilities in the lockfiles
        run: |
          docker run --rm -v "$PWD:/src" \
            ghcr.io/google/osv-scanner:v2.4.0@sha256:5116601dedc01c1c580eb92371883ec052fc4c13c3fbc109d621a63ac416d475 \
            scan source --recursive /src

      - name: Reject pickled model files
        run: |
          if git diff --name-only "origin/${{ github.base_ref }}...HEAD" \
             | grep -Ei '\.(pkl|pickle)$'; then
            echo "::error::Pickle executes arbitrary code on load. Not merging this."
            exit 1
          fi

  # Gate 2. Secrets and static analysis. There is no reason a person should be
  # the one who notices a hardcoded key.
  secrets-and-sast:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4.4.0
        with:
          fetch-depth: 0

      # The gitleaks CLI in a container, not gitleaks/gitleaks-action. The
      # action requires a GITLEAKS_LICENSE secret on any repository owned by an
      # organization, and a job triggered by a fork PR gets no repository
      # secrets at all. Both cases fail on a missing key, not on a finding, and
      # both are cases this file is written for. The CLI is
      # MIT-licensed and takes no key, so this step needs no secret and no
      # setup, and the job drops to `contents: read` because nothing here
      # writes a comment. Pinned by digest like the other two containers.
      - name: Secret scan
        run: |
          docker run --rm -v "$PWD:/repo" \
            ghcr.io/gitleaks/gitleaks:v8.30.1@sha256:c00b6bd0aeb3071cbcb79009cb16a60dd9e0a7c60e2be9ab65d25e6bc8abbb7f \
            git /repo \
              --log-opts="origin/${{ github.base_ref }}...HEAD" \
              --redact --no-banner --verbose

      - name: SAST
        run: |
          docker run --rm -v "$PWD:/src" -w /src \
            semgrep/semgrep:1.171.0@sha256:bdf7013b2c3634a487671158da77c554f531742326b543a9464d2adf6c433ac8 \
            semgrep scan \
              --config p/security-audit \
              --config p/owasp-top-ten \
              --error --skip-unknown-extensions

  # Gate 3. A second model reads the diff before a person does. Fresh context,
  # no stake in the code, prompted to break the change rather than bless it.
  # Its output is a worklist for the human. It never approves anything.
  #
  # Fork policy, stated rather than discovered: this gate CANNOT run on a pull
  # request from a fork, and it fails instead of skipping. See the first step.
  second-model-review:
    runs-on: ubuntu-latest
    needs: [provenance, secrets-and-sast]
    # Drafts are exempt on purpose. `ready_for_review` in the trigger list above
    # is what runs this the moment a draft is marked ready for review.
    if: ${{ github.event.pull_request.draft == false }}
    permissions:
      contents: read
      # Needed only by the last step, which posts the worklist as a comment.
      pull-requests: write
    steps:
      # Compare repo names rather than reading head.repo.fork, which is true
      # for every PR in a repository that is itself a fork of something.
      - name: Fork policy
        if: ${{ github.event.pull_request.head.repo.full_name != github.repository }}
        run: |
          echo "::error::Gate 3 cannot run on a pull request from a fork. Under"
          echo "::error::the pull_request event a fork gets no ANTHROPIC_API_KEY"
          echo "::error::and a read-only token, so this job can neither call the"
          echo "::error::API nor post its worklist. It fails rather than going"
          echo "::error::green on a gate that did not run. Run the adversarial"
          echo "::error::pass by hand from a maintainer checkout and say so in"
          echo "::error::the review before approving. Do not switch this file to"
          echo "::error::pull_request_target to make the check pass: that hands"
          echo "::error::your secrets to code nobody has read yet."
          exit 1

      - uses: actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4.4.0
        with:
          fetch-depth: 0

      - name: Build the diff
        env:
          # TUNABLE STARTING VALUE, not a measurement and not a receipt. Same
          # class of number as <N> in AI_POLICY.md: replace it from your own
          # repo's history with the diff size past which one review pass stops
          # being worth anything.
          MAX_DIFF_BYTES: "400000"
        run: |
          git diff "origin/${{ github.base_ref }}...HEAD" > pr.diff
          if [ "$(wc -c < pr.diff)" -gt "$MAX_DIFF_BYTES" ]; then
            echo "::error::Diff is over $MAX_DIFF_BYTES bytes, too large to review in one pass. Split the PR."
            exit 1
          fi

      - name: Adversarial review pass
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          # TUNABLE STARTING VALUE, not a measurement and not a receipt. Same
          # class of number as MAX_DIFF_BYTES above. It caps how long the
          # worklist is allowed to get. If the step below starts failing on
          # `max_tokens`, raise this or split the PR.
          MAX_REVIEW_TOKENS: "4000"
        run: |
          if [ -z "$ANTHROPIC_API_KEY" ]; then
            echo "::error::ANTHROPIC_API_KEY is not set, so gate 3 did not run."
            exit 1
          fi

          jq -n --rawfile diff pr.diff --argjson cap "$MAX_REVIEW_TOKENS" '{
            model: "claude-opus-5",
            max_tokens: $cap,
            system: "You are a paranoid staff engineer reviewing a diff you did not write. Your job is to break it. Look for the authorization gap, the unvalidated input crossing a trust boundary, the dependency that should not be there, the error path that swallows a failure, the test that asserts nothing. Report every finding with a file, a line, a one-sentence failure scenario, and your confidence. Do not filter by severity; a human does that next. Do not compliment the change. If you find nothing, say so in one line.",
            messages: [{role: "user", content: ("Review this diff:\n\n" + $diff)}]
          }' > request.json

          curl -sS --fail-with-body https://api.anthropic.com/v1/messages \
            -H "content-type: application/json" \
            -H "x-api-key: $ANTHROPIC_API_KEY" \
            -H "anthropic-version: 2023-06-01" \
            -d @request.json > response.json

          # A truncated worklist reads exactly like a clean one. Check the stop
          # reason before the content, because a response cut off at
          # MAX_REVIEW_TOKENS is well-formed and non-empty.
          stop_reason="$(jq -r '.stop_reason // "missing"' response.json)"
          if [ "$stop_reason" = "max_tokens" ]; then
            echo "::error::The review pass was cut off at MAX_REVIEW_TOKENS"
            echo "::error::($MAX_REVIEW_TOKENS), so the worklist is incomplete and"
            echo "::error::gate 3 did not finish. Raise MAX_REVIEW_TOKENS or split"
            echo "::error::the PR. Failing rather than posting a truncated worklist."
            exit 1
          fi
          if [ "$stop_reason" != "end_turn" ] && [ "$stop_reason" != "stop_sequence" ]; then
            echo "::error::The review pass stopped for an unexpected reason"
            echo "::error::($stop_reason), so gate 3 did not complete. Failing"
            echo "::error::rather than trusting a response we cannot account for."
            exit 1
          fi

          jq -r '[.content[]? | select(.type == "text") | .text] | join("\n")' \
            response.json > review.md

          # `jq -r` writes a trailing newline even when every text block is
          # empty, so review.md is one byte and `[ -s ]` would pass on nothing.
          # Strip whitespace and test what is actually left.
          if [ -z "$(tr -d '[:space:]' < review.md)" ]; then
            echo "::error::The review pass returned no text, so gate 3 did not"
            echo "::error::run. Failing rather than posting an empty worklist."
            exit 1
          fi

      - name: Post the worklist as a comment
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          {
            echo "### Second-model review pass"
            echo
            echo "Machine-generated. A worklist for the human reviewer, not an approval."
            echo
            cat review.md
          } | gh pr comment "${{ github.event.pull_request.number }}" --body-file -

Three numbers in that file are placeholders: <N> in AI_POLICY.md, and MAX_DIFF_BYTES and MAX_REVIEW_TOKENS in the workflow. All three are labelled in place as tunable starting values rather than findings. Replace them from your own repo's history before you rely on any of them.
Gate 3 fails on two shapes of half-finished output that would otherwise go green. A response truncated at MAX_REVIEW_TOKENS is well-formed and non-empty, so it reads like a completed worklist unless the step checks stop_reason, and the step checks it. And jq -r writes a trailing newline even when the model returned an empty text block, which makes the file one byte long, so the emptiness test strips whitespace instead of testing file size. The workflow never approves a PR, so what those two holes produced was a green check on gate 3 and a worklist that was empty or cut off mid-finding.
Everything the workflow pulls in is pinned: the one action to a commit SHA, the three containers to an image digest, with the readable version in a trailing comment. A mutable tag inside a supply-chain gate is the hole the gate exists to close.
Bumping them is only half automatic. Dependabot and Renovate both handle the actions/checkout line, because their GitHub Actions support reads action references. Neither sees the three container digests, because those sit inside run: commands, and Renovate's GitHub Actions manager extracts images from the uses, container and services fields only. So the three docker run digests are yours to bump by hand, or with a Renovate customManagers regex rule you write. A digest nothing is watching ages into the mutable-tag problem with extra steps.
Running the gitleaks CLI in a container rather than gitleaks/gitleaks-action is what keeps gate 2 copy-and-commit. The action asks for a GITLEAKS_LICENSE secret on any repository owned by an organization, and it is free but you have to go request it, add it as a secret, and remember it exists. A job triggered by a fork PR gets no repository secrets at all, so on a fork the action would fail on a missing key before the fork policy in gate 3 ever ran. The CLI is MIT-licensed, takes no key, and needs nothing set up, so the copy-and-commit path stays copy and commit. The one thing you give up is the review comment the action posts on each finding, which is why gate 2 is now contents: read with no write scope at all. The finding is in the job log and the check is red.
Gate 1's first step can't fail the build, and its name says so. It prints the manifest diff and a checklist, because deciding whether a freshly published package with almost no downloads belongs in your lockfile is a judgment call about your repo, and I can't write it as a rule from here. The osv-scanner pass and the pickle rejection in the same job are the parts that can fail the build.
The second-model pass is prompted adversarially and told not to filter by severity. Ask it for coverage and do the filtering yourself. A severity filter written into the prompt moves the judgment call into the machine, which is the thing gate 4 exists to keep.
Gate 3 posts a comment and never approves. Keep it that way, and keep the branch protection rule that requires a human approval. Otherwise the recorded reviewing on your repo is agent-on-agent, which is the shape the AIDev numbers found, and the human gate is left with nothing to show for itself.
A draft PR doesn't get gate 3 at all, which is why ready_for_review sits in the trigger list. Leave that trigger out and a PR opened as a draft sails past gate 3 for good, because marking it ready doesn't re-run the workflow. Gates 1 and 2 still run on drafts.
The fork case is worse, so decide it before you turn this on. Under the pull_request event a PR from a fork gets no repository secrets and a read-only token, so gate 3 can neither call the API nor post its comment. The workflow above fails the check instead of quietly going green on a gate that didn't run, which leaves a maintainer to run the adversarial pass by hand from their own checkout and say so in the review. The fix that suggests itself is pull_request_target, which does give the job your secrets, and gives them to code nobody has read yet. If you take outside contributions, pick one of those on purpose.
