# Agent Engineering Discipline

## Summary

The agent-engineering sources define production agent work as a cross-functional discipline for turning non-deterministic LLM systems into reliable experiences. It combines product thinking, engineering, data science, framework selection, memory/context management, and repeated observation of real behavior.

## Key Takeaways

- Agent engineering is cyclical: build, test, ship, observe, refine, and repeat rather than treating launch as the finish line.
- Product teams define scope, jobs-to-be-done, prompts, and evals; engineers build tools, runtimes, UI, and durable execution; data teams measure behavior and mine failures.
- Framework choice should follow the required control surface: simple chains, graph runtimes, deep agents, or custom harnesses depending on persistence and coordination needs.
- Memory and context management are product decisions as much as technical ones because they shape what the agent can remember, ignore, and retrieve.
- Multi-agent systems are useful when tasks require specialization, independent verification, or parallel work; they add coordination and observability costs.
- Deep-agent patterns include context management, dynamic subagents, RLM usage, prompt caching, and long-running task decomposition.
- The bridge between agents and applications is the surrounding architecture: tools, UI, persistence, interrupts, traces, and explicit ownership.

## Source Coverage

- `raw/articles/2026-07-26-Agent Engineering- A New Discipline.md`
- `raw/articles/2026-07-26-Context Management for Deep Agents.md`
- `raw/articles/2026-07-26-How and when to build multi-agent systems.md`
- `raw/articles/2026-07-26-How to Build Memory into AI Agents.md`
- `raw/articles/2026-07-26-How to Use RLMs in Deep Agents.md`
- `raw/articles/2026-07-26-How to think about agent frameworks.md`
- `raw/articles/2026-07-26-Introducing Dynamic Subagents in Deep Agents.md`
- `raw/articles/2026-07-26-Prompt Caching with Deep Agents.md`
- `raw/articles/2026-07-26-The Missing Link Between Agents and Applications.md`
- `raw/articles/2026-07-26-What is an AI agent?.md`
- `raw/articles/2026-07-26-Why Fleet Has General Purpose Chat and Specialized Agents.md`

## Related

- [[ai-agents-overview]]
- [[agent-evaluation-observability/agent-evaluation-foundations]]
- [[agent-platform-operations/agent-runtime-sandboxing-governance]]
- [[ai-backend-architecture/runtime-intelligence-and-ai-backends]]
