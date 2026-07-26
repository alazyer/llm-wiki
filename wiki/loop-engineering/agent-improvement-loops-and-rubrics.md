# Agent Improvement Loops and Rubrics

## Summary

The loop-improvement sources describe agent quality as a feedback system: observe traces, mine failures, update prompts/tools/rubrics, run evals, and repeat. Loop engineering appears as the operational discipline that keeps non-deterministic agents improving under human judgment instead of drifting invisibly.

## Key Takeaways

- Loop engineering is the art of repeatedly turning agent output into evidence, evidence into changes, and changes into measured behavior.
- Agent improvement is a data-mining problem because the highest-value fixes often come from clusters of real production failures.
- Human judgment remains central: humans define what matters, review ambiguous cases, and decide which evaluator or rubric failures are meaningful.
- Rubrics make quality criteria explicit enough for agents to evaluate and correct their own work, but rubrics still need calibration and review.
- Prompt optimization tools can help, but they should optimize against task-specific evals rather than generic preference signals.
- Prompt caching and context control improve deep-agent economics by reducing repeated setup cost and stabilizing long-running work.
- Self-improvement loops need deployment gates: canaries, rollback criteria, and human approval before automated changes affect production.

## Source Coverage

- `raw/articles/2026-07-26-Improving Agents is a Data Mining Problem.md`
- `raw/articles/2026-07-26-Human judgment in the agent improvement loop.md`
- `raw/articles/2026-07-26-Introducing Rubrics- Build Agents that Evaluate and Correct Their Work.md`
- `raw/articles/2026-07-26-Prompt Caching with Deep Agents.md`
- `raw/articles/2026-07-26-Promptim- an experimental library for prompt optimization.md`
- `raw/articles/2026-07-26-The Art of Loop Engineering.md`
- `raw/articles/2026-07-26-TutorialAn agent that ships new versions of itselfA self-improvement loop where .md`
- `raw/articles/2026-07-26-TutorialThe deployment that promotes itselfA self-tuning rollout deploys a new a.md`

## Related

- [[loop-engineering]]
- [[agent-evaluation-observability/langsmith-observability-feedback-loops]]
- [[agent-evaluation-observability/llm-judge-verifier-patterns]]
- [[ai-assisted-software-engineering/own-the-outer-loop]]
