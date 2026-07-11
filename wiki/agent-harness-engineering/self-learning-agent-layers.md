# Self-Learning Agent Layers

## Summary
Most practical “self-learning” in products happens outside model weights. The useful layers are the harness and context: loops, tools, prompts, hooks, memory, skills, and user-interaction traces that can be inspected, changed, and rolled back.

## Key Takeaways
- Agent learning can happen in three layers: model weights, harness, and context.
- Model-level learning belongs mostly in labs because it needs reliable automated scoring and expensive retraining.
- Harness learning improves prompts, tools, hooks, and loops from traces, often with human approval before changes land.
- Context learning stores memory and skills as plain text that the next run can read.
- Semantic memory records facts, episodic memory records experiences, and procedural memory records how to handle cases.
- The best product signal often comes from users correcting or approving agent work in the actual interface.
- AG-UI-style event streams can capture both the agent’s miss and the human’s fix as procedural memory.
- The practical question is not whether agents improve, but which layer you own and can safely update.

## Related
- [[agent-harness-engineering/agent-harness-engineering]]
- [[agent-harness-engineering/self-improving-agent-system-fable-5]]
- [[ai-agent-operations/openclaw-memory-architecture]]
- [[generative-ui/generative-ui-new-frontend-patterns]]
