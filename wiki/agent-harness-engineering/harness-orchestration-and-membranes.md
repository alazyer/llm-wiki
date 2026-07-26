# Harness Orchestration and Membranes

## Summary

The harness-orchestration sources treat a harness as the boundary around an autonomous work unit: workspace, tools, memory, verifiers, blast radius, and persistence. The key idea is that orchestration changes when the unit being orchestrated can decide, act, and keep state instead of merely returning a deterministic API response.

## Key Takeaways

- Harnesses are not ordinary functions: they combine agency, embodiment, and persistence inside a boundary that can drift while work is running.
- The membrane around a harness defines what it can read, what it can change, which verifiers it can see, and how much damage can be undone.
- Orchestrating harnesses means managing teams of autonomous systems such as Claude Code, Codex, Gemini, Cursor, or custom reasoners.
- Boundary design matters more than model choice when the harness has tools, filesystem access, and long-running context.
- Effective orchestration makes blast radius explicit through separate workspaces, checkpoints, approval gates, and verifier placement.
- Harness orchestration connects directly to graph engineering: nodes are no longer simple calls, but bounded autonomous workers with contracts.

## Source Coverage

- `raw/articles/2026-07-26-An Engineer's Guide to Harness OrchestrationWhy orchestration changes when the u.md`
- `raw/articles/2026-07-26-Part 2 · Engineering the MembraneThe Hidden Primitive Behind Claude Code, Codex.md`
- `raw/articles/2026-07-26-What is harness orchestration?What changes when the atomic unit of intelligence .md`

## Related

- [[agent-harness-engineering]]
- [[ai-backend-architecture/runtime-intelligence-and-ai-backends]]
- [[ai-backend-architecture/agentfield-control-plane-patterns]]
- [[graph-engineering/langgraph-runtime-and-graph-engineering]]
