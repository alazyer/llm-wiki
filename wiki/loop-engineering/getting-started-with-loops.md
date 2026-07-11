# Getting Started with Loops

## Summary
Loops are agents repeating cycles of work until a stop condition is met. The practical starting point is to choose the simplest loop type that fits the task: turn-based, goal-based, time-based, or proactive.

## Key Takeaways
- Turn-based loops are ordinary prompt-and-response work where the human still owns the check.
- Goal-based loops hand off the stop condition through `/goal` and work best when completion is measurable.
- Time-based loops use `/loop` or `/schedule` for recurring work or external systems that change over time.
- Proactive loops combine schedules, goals, skills, dynamic workflows, and auto mode for recurring well-defined streams of work.
- Verification skills improve quality by encoding how humans check work end-to-end.
- Token use is controlled through boundaries: choose the right primitive, cap attempts, pilot small, and use scripts for deterministic work.
- Start by finding one repeated task where you can hand off either the check, the stop condition, the trigger, or the prompt.

## Related
- [[loop-engineering/loops-explained]]
- [[loop-engineering/goal-as-agent-primitive]]
- [[agent-harness-engineering/dynamic-workflows-in-claude-code]]
- [[agent-harness-engineering/anatomy-of-agent-skills]]
