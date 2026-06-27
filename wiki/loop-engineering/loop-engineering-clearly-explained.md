# Loop Engineering Clearly Explained

## Summary
The simple `while` loop is not the hard part of agent loops. The hard parts are defining a real stop condition, protecting context, designing focused tools, adding a critic that can say no, and preventing the agent from drifting into busywork.

## Key Takeaways
- A loop should stop on task completion, not on turn count or agent confidence.
- Context rot is a core failure mode; use compaction, offloading, and sub-agents to keep the active context clean.
- Tools should be focused, idempotent, safe to retry, and readable by agents.
- Self-verification is weak; a separate critic, test suite, or deterministic gate must be able to reject work.
- Loose done criteria create random walks where the agent keeps acting without converging.
- Start with caps, a narrow blast radius, automated done checks, and an explicit tool audit.

## Related
- [[loop-engineering/loops-explained]]
- [[loop-engineering/loop-engineering-14-step-roadmap]]
- [[agent-harness-engineering/dynamic-workflows-in-claude-code]]
- [[ai-assisted-software-engineering/agentic-code-review-verification-bottleneck]]
