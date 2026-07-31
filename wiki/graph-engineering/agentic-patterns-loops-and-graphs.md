# Agentic Patterns, Loops, and Graphs

## Summary

The fetched AI Builder Club graph articles map Andrew Ng-style agentic patterns and the “loops or graphs” debate into a practical design distinction. Loops are repeated feedback cycles; graphs are explicit dependency structures. Durable systems often use both: loops decide when to continue improving, while graphs make multi-step work, routing, verification, and parallelism inspectable.

## Key Takeaways

- Reflection, tool use, planning, and multi-agent collaboration can be represented as graph nodes and edges when the dependencies need to be visible.
- A loop is best for repeated improvement against a signal; a graph is best for decomposed work with named contracts between steps.
- Graphs help when you need fan-out/fan-in, independent verifiers, rollback points, parallel specialists, or evidence that each step ran.
- Loops remain simpler when the work is linear, low-risk, and does not need explicit multi-node topology.
- The “loops or graphs” debate is mostly a seam decision: use the smallest structure that makes failure modes observable.
- Graph engineering should preserve loop discipline: budgets, stop conditions, verifiers, and no-op paths still matter.

## Source Coverage

- `raw/articles/2026-07-30-Andrew Ng's Agentic Patterns, Mapped to GraphsNEW.md`
- `raw/articles/2026-07-30-Peter Steinberger's "Loops or Graphs?" TweetNEW.md`

## Related

- [[graph-engineering/graph-engineering-basics]]
- [[graph-engineering/langgraph-runtime-and-graph-engineering]]
- [[loop-engineering/seo-and-social-agent-loops]]
- [[agent-harness-engineering/harness-orchestration-and-membranes]]
