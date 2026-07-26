# Graph Engineering Roadmap

## Summary
The graph-engineering roadmap upgrades a linear agent into a fleet-ready workflow by defining node contracts, treating edges as data contracts, selecting the right fan-out/fan-in topology, routing at runtime, isolating workers, adding convergent cycles, and tiering models by judgment requirements.

## Key Takeaways
- Linear scripts are degenerate graphs; redraw them by asking which arrows carry data and which are only typed order.
- Every node needs a contract: one job, explicit input, validated structured output, and no dependence on a shared chat window.
- Edges should be deterministic plumbing where possible; flattening, deduping, and filtering belong in code, not extra model calls.
- Use `parallel()` for independent batches and a fan-in barrier only when a stage needs the whole set together.
- The diamond pattern is the workhorse: split, parallel work, deterministic reduce, optional verification, and final synthesis.
- Runtime routers can classify outputs and choose downstream paths while preserving deterministic control flow in the orchestration script.
- Verifier nodes can be adversarial, perspective-diverse, or judge panels, but they must inspect the finding rather than inherit the worker's full context.
- Isolation matters when nodes write: separate worktrees or sandboxes prevent parallel agents from overwriting each other.
- Cycles need convergence rules such as loop-until-dry, hard caps, and dedupe against everything seen, not only confirmed findings.
- Tier cheap models for repetitive extraction/classification nodes and reserve stronger models for synthesis, judgment, or adjudication.
- Topology determines latency and cost; default to streaming/pipeline shapes unless a true cross-set dependency requires a barrier.

## Related
- [[graph-engineering/graph-engineering-basics]]
- [[graph-engineering/claude-dynamic-workflows-at-scale]]
- [[agent-harness-engineering/dynamic-workflows-in-claude-code]]
- [[ai-assisted-software-engineering/agentic-code-review-verification-bottleneck]]