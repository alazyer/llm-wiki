# Dynamic Workflows in Claude Code

## Summary
Dynamic workflows are task-specific harnesses generated for a particular job. Instead of trusting one long agent run, they classify work, fan out subtasks, synthesize results, run adversarial verification, filter candidates, or loop until a concrete condition is met.

## Key Takeaways
- Workflows counter agentic laziness, self-preference bias, and long-context goal drift.
- Useful patterns include classify-and-act, fan-out-and-synthesize, adversarial verification, generate-and-filter, tournament, routing, and loop-until-done.
- They are strongest for migrations, research, verification, triage, evals, and routing tasks.
- They cost more tokens and complexity, so avoid them for simple edits.
- Pair workflows with `/goal`, `/loop`, tests, and external verifiers for durable results.
- A workflow is a disposable harness: design the process, not just the prompt.

## Related
- [[agent-harness-engineering/agent-harness-engineering]]
- [[loop-engineering/goal-as-agent-primitive]]
- [[loop-engineering/self-verifying-agent-swarms]]
- [[generative-ui/generative-ui-new-frontend-patterns]]
