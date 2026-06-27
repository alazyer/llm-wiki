# Agentic Code Review: The Verification Bottleneck

## Summary
AI made writing code cheaper, but understanding and review did not become cheap at the same rate. Agentic code review must scale scrutiny by risk, require evidence before human review, and use independent reviewers to catch failures the builder missed.

## Key Takeaways
- Generated code increases throughput, churn, and potential review load.
- Review depth should scale with blast radius, not lines changed.
- Pull requests should include the agent’s intent, assumptions, tests run, and known risks.
- Heterogeneous AI reviewers can catch different classes of issues before humans review.
- Humans move “on the loop”: owners, auditors, and escalation points rather than keystroke producers.
- Evidence gates should reject work before review if tests, lint, or requirements checks fail.

## Related
- [[loop-engineering/goal-as-agent-primitive]]
- [[loop-engineering/self-verifying-agent-swarms]]
- [[ai-assisted-software-engineering/dont-outsource-the-learning]]
- [[ai-agent-operations/local-coding-agents-with-hermes]]
