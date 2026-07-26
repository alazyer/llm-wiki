# LLM Judge and Verifier Patterns

## Summary

The verifier-focused articles treat LLM judges as useful but fallible components that need calibration, human preference alignment, clear rubrics, cost controls, and task-specific evidence. The goal is not to replace human judgment blindly; it is to make verification cheaper, more repeatable, and easier to audit.

## Key Takeaways

- LLM-as-judge systems should be aligned against human preferences rather than trusted solely because they produce confident scores.
- Trace judges can make evaluation cheaper by judging structured execution traces instead of rerunning expensive end-to-end workflows.
- Rubrics are executable quality definitions: they help agents evaluate, correct, and explain their own work.
- Domain-specific verifiers, such as legal-agent verifiers, need evidence requirements and failure criteria tied to the domain’s risk.
- Cost-aware verifier design routes easy checks to cheap evaluators and reserves stronger models or humans for ambiguous cases.
- Judge outputs should become artifacts in the improvement loop, not final truth; use them to triage, sample, and focus human review.

## Source Coverage

- `raw/articles/2026-07-26-Aligning LLM-as-a-Judge with Human Preferences.md`
- `raw/articles/2026-07-26-Building a 100x Cheaper Trace Judge with Fireworks.md`
- `raw/articles/2026-07-26-Designing Efficient Verifiers for Legal Agents.md`
- `raw/articles/2026-07-26-Eval Engineering Skill- Build Evals From Repo Context and Traces.md`
- `raw/articles/2026-07-26-Introducing Rubrics- Build Agents that Evaluate and Correct Their Work.md`

## Related

- [[agent-evaluation-foundations]]
- [[loop-engineering/agent-improvement-loops-and-rubrics]]
- [[loop-engineering/self-verifying-agent-swarms]]
- [[ai-assisted-software-engineering/agentic-code-review-verification-bottleneck]]
