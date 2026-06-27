# Anatomy of Agent Skills

## Summary
Agent skills package reusable behavior as a folder contract, usually centered on a `SKILL.md` file plus optional references, scripts, and assets. The skill metadata routes the agent; the body provides progressive instructions only when the skill is relevant.

## Key Takeaways
- A skill is a reusable capability, not a giant prompt pasted into every conversation.
- Name and description act as the routing index, so metadata quality controls invocation.
- Progressive disclosure keeps token use low: load metadata first, instructions second, references only when needed.
- Skills can include examples, assets, scripts, and test fixtures.
- Good skills encode constraints, decision rules, and common failure recoveries.
- Skills compose like packages, but governance matters because bad skills can spread bad behavior.

## Related
- [[agent-harness-engineering/agent-harness-engineering]]
- [[agent-harness-engineering/dynamic-workflows-in-claude-code]]
- [[claude-usage-systems/claude-md-setup-21-things]]
- [[ai-agent-operations/openclaw-memory-architecture]]
