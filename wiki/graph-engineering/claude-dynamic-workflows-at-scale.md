# Claude Dynamic Workflows at Scale

## Summary
Claude Code dynamic workflows let a user describe a graph-shaped objective, review a generated orchestration script, and run coordinated subagents that fan out, verify findings, synthesize results, and can later be saved as reusable workflows.

## Key Takeaways
- A practical first workflow is a scoped code audit: one agent per route file, independent verifiers for each finding, and a cap such as 20 files to control cost.
- Dynamic workflows coordinate through code rather than chat handoffs, so intermediate results live in script variables instead of filling the main conversation context.
- Coordination may be token-cheap, but the spawned agents still consume usage; start small, observe cost, then widen only after the graph earns it.
- Saved workflows become reusable commands, turning a good graph shape into versioned operational infrastructure.
- Large runs can scale in waves, but scale introduces real supervision duties: usage caps, progress monitoring, worker isolation, merge rules, and disagreement handling.
- The source highlights Bun's Zig-to-Rust port as a ceiling case: massive parallelism can compress calendar time, but it can also create review, cost, and safety concerns.
- Useful starter graphs include security sweeps, cited research reports, module ports, adversarial diff reviews, scheduled ecosystem scans, and unknown-size discovery loops.
- The durable method is: find real edges, fan out independent work, verify on independent context, isolate workers, freeze truth anchors, and keep humans as architects.

## Related
- [[graph-engineering/graph-engineering-basics]]
- [[graph-engineering/graph-engineering-roadmap]]
- [[agent-harness-engineering/dynamic-workflows-in-claude-code]]
- [[loop-engineering/self-verifying-agent-swarms]]