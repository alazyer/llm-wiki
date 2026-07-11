# Self-Improving Agent Systems with Fable 5

## Summary
A self-improving agent system does not change model weights; it makes the environment around the model compound through loops, workflows, routines, state files, skills, evals, and independent verifiers. Fable 5 is presented as useful when it acts as an orchestrator for long-running systems, not when used as a more expensive chat model.

## Key Takeaways
- Self-improving means the system around the model improves; self-learning would mean the model updates its own weights.
- The compound stack has primitives, orchestration, memory, and self-improvement layers.
- Route by task complexity: strongest models for orchestration and judgment, cheaper models for workers and graders.
- `/goal` and hosted Outcomes share the same pattern: independent grader checks a measurable stop condition.
- Verifier sub-agents outperform self-critique because the maker should not grade its own work.
- State files support a five-stage memory progression: fail, investigate, verify, distill, consult.
- Skills should accumulate confirmed procedural lessons, not just exist as static instructions.
- Long-running systems need safety-boundary handling, cost controls, retention review, and fallback paths.

## Related
- [[loop-engineering/goal-as-agent-primitive]]
- [[agent-harness-engineering/dynamic-workflows-in-claude-code]]
- [[loop-engineering/self-verifying-agent-swarms]]
- [[ai-agent-operations/hermes-vs-openclaw-learning-loop]]
