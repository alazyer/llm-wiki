# `/goal` as an Agent Primitive

## Summary
`/goal` is not just a prompt label. It is an assignment with a measurable done state, evidence requirements, constraints, and a verifier. In a multi-agent setup, `/goal` becomes a handoff contract between orchestrator, builder, reviewer, and human owner.

## Key Takeaways
- A good goal defines success evidence before work starts.
- The builder can act freely only inside the goal’s constraints, budget, and blast radius.
- The verifier should independently run checks instead of trusting the builder’s summary.
- Kanban cards or durable task records make goals composable across tools.
- Parallel goals need one-writer/worktree discipline to avoid collisions.
- `/goal` is the bridge from chat prompting to durable agent pipelines.

## Related
- [[loop-engineering/from-prompting-agents-to-loop-engineering]]
- [[loop-engineering/loop-engineering-14-step-roadmap]]
- [[ai-agent-operations/local-coding-agents-with-hermes]]
- [[ai-assisted-software-engineering/agentic-code-review-verification-bottleneck]]
