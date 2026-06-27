# Self-Verifying Agent Swarms

## Summary
Large agent swarms do not become trustworthy merely by adding more workers. The useful pattern is a planner/verifier plus executor swarm: workers produce candidates, a stricter verifier checks live evidence, rejects failures, and requeues the work until it clears the checklist.

## Key Takeaways
- Raw swarm speed multiplies both output and error unless verification is explicit.
- Split roles: one model plans and verifies while many cheaper agents execute bounded tasks.
- Failed outputs should be rejected, diagnosed, and requeued rather than polished into reports.
- Live-source validation is stronger than confidence, style, or citation-looking text.
- Quality comes from the checklist and rejection loop, not the number of agents.
- This is the same loop pattern as coding agents: make, check, repair, repeat.

## Related
- [[loop-engineering/loops-explained]]
- [[loop-engineering/loop-engineering-clearly-explained]]
- [[agent-harness-engineering/dynamic-workflows-in-claude-code]]
- [[ai-assisted-software-engineering/agentic-code-review-verification-bottleneck]]
