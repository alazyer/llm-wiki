# Loop Engineering: Build an AI That Codes While You Sleep

## Summary
Loop engineering is the practice of building a small system that finds work, hands it to an agent, checks the result, and decides the next move. The goal is not to replace the engineer, but to let an agent run bounded, repeatable work while the engineer stays responsible for judgment.

## Key Takeaways
- Boris Cherny's example shows the leverage shift: his job was to write the thing that writes the code.
- A working loop has six pieces: state, automations, worktrees, skills, connectors, and sub-agents.
- State is the only thing the next run inherits; use `STATUS.md` or a board to record done, in-progress, next, and never-touch items.
- Automations make a loop run on a schedule; in Claude Code, `/loop` is a skill, Routines are scheduled cloud jobs, and `/goal` runs until a condition is true.
- Worktrees isolate parallel agents so they cannot overwrite each other's files.
- Skills codify your intent once, so each run does not re-derive conventions and constraints from scratch.
- Connectors, often through MCP, let the loop act in real tools such as issue trackers, staging APIs, databases, and chat.
- Sub-agents split the maker from the checker; a model reviewing its own work is too generous.
- Install brakes before horsepower: step caps, budget ceilings, blast radius limits, circuit breakers, and dead-man heartbeats.
- Loops die through runaway recursion, silent death, random walks, and comprehension debt.
- The safest starting point is one small, dull, capped loop around a task you already understand.

## Related
- [[loop-engineering/from-prompting-agents-to-loop-engineering]]
- [[loop-engineering/loop-engineering-14-step-roadmap]]
- [[agent-harness-engineering/anatomy-of-agent-skills]]
- [[ai-agents/ai-agents-overview]]
