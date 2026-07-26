# Karpathy Loop and Bilevel Autoresearch

## Summary
The Karpathy Loop frames autonomous AI research as a measurable experiment cycle: an agent proposes changes, runs training, evaluates against a protected metric, commits improvements, rolls back failures, and repeats; bilevel autoresearch adds an outer loop that improves the search process itself.

## Key Takeaways
- A loop is not just repetition; it needs a verifier, state, and a stop condition so each pass can make measurable progress without running forever.
- Use loops only when the task repeats, verification is automated, the token budget can absorb retries, and the agent has tools such as logs, builds, tests, or training runs.
- The AutoResearch setup protected the evaluator from modification while allowing the agent to edit the training script and follow instructions in `program.md`.
- The key pattern is: read code, propose a change, train briefly, evaluate, keep improvements, roll back regressions, and repeat.
- The source cites Karpathy's run as finding additional improvements after hundreds of experiments, illustrating how agents can explore longer than tired humans.
- A working loop is assembled from automation, skills, sub-agents, connectors, and a verifier; the verifier is the part that makes the loop real.
- Bilevel autoresearch adds an outer loop that inspects the inner loop's traces, notices search stagnation, and modifies how the inner loop explores.
- The meta-loop improvement comes from architecture rather than a smarter model: the outer loop forces exploration beyond the inner loop's stale priors.
- Loops can create comprehension debt and cognitive surrender if humans stop understanding the work; loop design should move humans toward judgment, not abdication.

## Related
- [[loop-engineering/loops-explained]]
- [[loop-engineering/build-an-ai-that-codes-while-you-sleep]]
- [[agent-harness-engineering/self-improving-agent-system-fable-5]]
- [[graph-engineering/graph-engineering-basics]]