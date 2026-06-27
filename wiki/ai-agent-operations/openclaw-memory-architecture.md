# OpenClaw Agent Memory Architecture

## Summary
OpenClaw memory works because it is stored in files rather than ephemeral chat sessions. Corrections, facts, and operating rules are extracted into shared memory so one correction can improve every agent that reads it.

## Key Takeaways
- Memory has layers: working memory, session memory, and long-term Memory Bank.
- Raw transcripts are useful for audit but too noisy for direct recall.
- Extracted facts, corrections, and preferences should live in compact shared files.
- One correction can propagate across agents when all roles read a common memory source.
- Memory hygiene matters: stale, contradictory, or bloated memory degrades behavior.
- Start with one `MEMORY.md`; add cross-agent shared context only after repeated corrections emerge.

## Related
- [[ai-agent-operations/openclaw-40-day-stack]]
- [[ai-agent-operations/thirty-day-agent-team-lessons]]
- [[agent-harness-engineering/anatomy-of-agent-skills]]
- [[claude-usage-systems/claude-md-setup-21-things]]
