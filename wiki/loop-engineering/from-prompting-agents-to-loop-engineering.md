# From Prompting Agents to Loop Engineering

## Summary
Loop engineering moves the work from hand-writing prompts to designing systems that prompt agents for you. A loop is a small program that sets a goal, acts, checks whether the goal is met, and repeats with the next step or error until it stops.

## Key Takeaways
- A loop is not just repeated prompting; it is a goal, an action, a check, and a repeat condition.
- The six core pieces of a loop are: trigger, isolation, written-down context, reach into your tools, a second agent checks, and state on disk.
- A concrete loop example is a PR babysitter: check CI, fix deterministic failures, rebase once, and ping a human when needed.
- Claude Code's `/goal` is a built-in loop: set a measurable end state, let the agent work, and have an evaluator check completion.
- Running many loops unattended requires auto-approval, workflows, `/goal` or `/loop`, cloud persistence, and self-verification.
- Orchestration tools like crabfleet package loops into boards, durable runs, and spawned agents.
- The cost of a loop is measured in iterations, not just tokens, so tight verifiers and fast failure matter.
- Loops are not for one-shot edits, unscoped work, or anything without a cheap automated check.
- Common failure modes are human verification burden, comprehension gaps, and silent drift on a loose check.
- Build loops by picking one repeatable task, scoping it tight, setting a budget and stop condition, adding an independent verifier, running it on a cadence, and keeping memory on disk.

## Related
- [[ai-agents/ai-agents-overview]]
- [[loop-engineering/goal-as-agent-primitive]]
- [[agent-harness-engineering/agent-harness-engineering]]
- [[ai-product-management/modern-ai-pm-in-age-of-agents]]
