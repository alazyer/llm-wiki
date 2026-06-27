# Agent Harness Engineering

## Summary
An agent is not just a model. It is a model wrapped in a harness: prompts, tools, state, memory, hooks, sandboxes, sub-agents, observability, and escalation rules. Harness engineering designs that wrapper so agent behavior becomes repeatable, inspectable, and improvable.

## Key Takeaways
- Model quality matters, but the harness often determines operational reliability.
- Design backward from behavior: what must the agent do, avoid, remember, and prove?
- Every repeated failure should ratchet into a permanent rule, hook, test, or memory update.
- Context rot requires progressive disclosure, summaries, state files, and retrieval boundaries.
- Hooks enforce policy at runtime instead of hoping the prompt is remembered.
- Observability turns agent work from vibes into traces, budgets, events, and failure modes.
- Harness engineering is adjacent to [[loop-engineering/loop-engineering-addy-osmani]]: the harness is the agent body; the loop is the recurring operating system.

## Related
- [[agent-harness-engineering/dynamic-workflows-in-claude-code]]
- [[agent-harness-engineering/anatomy-of-agent-skills]]
- [[loop-engineering/loop-engineering-addy-osmani]]
- [[ai-agent-operations/openclaw-40-day-stack]]
