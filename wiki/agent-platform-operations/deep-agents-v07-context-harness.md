# Deep Agents v0.7 Context Harness

## Summary

LangChain's Deep Agents v0.7 release is a context-engineering and harness-simplification story. The release trims base prompts and tool descriptions, makes todos opt-in, adds more configurable middleware, and improves filesystem ergonomics. The broader lesson is that agent platforms should treat context as a maintained runtime interface rather than a pile of evergreen prompt prose.

## Key Takeaways

- Deep Agents v0.7 reduces base input tokens by removing legacy system-prompt guidance, trimming tool descriptions, and making todo middleware opt-in.
- Tool schemas and clear interfaces can teach behavior better than long examples that narrow model exploration.
- Harness defaults should change as model behavior changes; prompt bloat becomes a platform cost and quality risk.
- Middleware configuration lets teams tune summarization thresholds, prompt caching, and context management for specific workloads.
- Filesystem tools are part of the agent runtime contract: overwrite behavior, paginated reads, bounded grep/glob, and delete allowlists shape safety and usability.
- Breaking changes are acceptable when they simplify the base harness and make runtime behavior easier to reason about.

## Source Coverage

- `raw/articles/2026-07-30-Deep Agents v0.7.md`

## Related

- [[agent-platform-operations/agent-runtime-sandboxing-governance]]
- [[agent-harness-engineering/agent-harness-engineering]]
- [[loop-engineering/agent-improvement-loops-and-rubrics]]
- [[ai-assisted-software-engineering/coding-agent-fleets-and-cost-control]]
