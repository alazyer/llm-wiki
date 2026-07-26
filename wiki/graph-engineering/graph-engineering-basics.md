# Graph Engineering Basics

## Summary
Graph engineering treats AI work as a dependency graph instead of a single prompt chain: nodes do bounded jobs, edges carry real outputs, independent nodes run in parallel, and verifier/anchor nodes keep the graph from merely agreeing with itself.

## Key Takeaways
- A node is one bounded job with explicit input and structured output; an edge exists only when one node's output is actually consumed by another node.
- The fake-edge test is the core speed lever: if the next step does not read the previous result, remove the dependency and run the jobs side by side.
- The most reusable topology is the diamond: fan out independent work, reduce with deterministic code, verify findings, then synthesize one answer.
- Verifiers should be separate nodes with fresh context; a worker checking its own output is still a single loop grading its own homework.
- Graphs break through context collapse, false independence, shared-resource collisions, silent node failures, and self-referential metrics.
- Graphs need anchors: tests that actually ran, real revenue, real customer behavior, frozen rules, or other evidence the optimizer cannot rewrite.
- Skip graphs for small tasks, exploratory steering, tight step-by-step approval, or genuinely sequential work with no independent branches.

## Related
- [[graph-engineering/graph-engineering-roadmap]]
- [[graph-engineering/claude-dynamic-workflows-at-scale]]
- [[loop-engineering/loops-explained]]
- [[ai-assisted-software-engineering/own-the-outer-loop]]