# OpenClaw 24/7 Autonomous Agent Team Setup

## Summary
OpenClaw-style agent teams run as role-specialized agents on an always-on machine, coordinated through files, schedules, and a chat interface. The practical lesson is to start with one agent, one job, and one schedule before scaling to a team.

## Key Takeaways
- A multi-agent team needs roles, handoffs, memory, schedules, and health checks.
- `SOUL.md`-style identity files define agent role, tone, boundaries, and operating principles.
- The filesystem can coordinate agents through task files, logs, shared context, and memory.
- Cron plus heartbeat checks keeps long-running agents alive and observable.
- Telegram or chat interfaces are useful control planes, but files remain the durable source of truth.
- Scale gradually; six agents without discipline creates coordination noise.

## Related
- [[ai-agent-operations/openclaw-40-day-stack]]
- [[ai-agent-operations/openclaw-memory-architecture]]
- [[ai-agent-operations/thirty-day-agent-team-lessons]]
- [[agent-harness-engineering/agent-harness-engineering]]
