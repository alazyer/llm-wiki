# AI Review, Formal Methods, and Agentic Development

## Summary

The newly fetched software-engineering articles connect three verification pressures: AI-generated pull requests increase review load, Anthropic-style agentic development changes how software gets built, and formal methods may become more attractive as code generation accelerates. The durable lesson is that AI coding workflows must shift verification left and make correctness evidence easier for humans and agents to inspect.

## Key Takeaways

- Reviewing AI-generated pull requests requires evidence gates: tests, diffs, risk tiers, ownership notes, and explicit reviewer attention to generated-code failure modes.
- Agentic development changes CI/CD expectations because agents can propose, test, revise, and sometimes deploy changes under policy guardrails.
- Formal methods remain specialized, but AI may lower the cost of writing specs, exploring state spaces, and making verification workflows more approachable.
- Human reviewers need better artifacts, not just more code: intent summaries, changed invariants, traceable tests, and proof-like evidence where risk warrants it.
- Version control and executable tests are core coordination surfaces between humans and coding agents.
- The biggest risk is false confidence: generated code that looks plausible, passes shallow tests, but violates hidden invariants or distributed-system assumptions.

## Source Coverage

- `raw/articles/2026-07-29-Reviewing AI-Generated Pull RequestsNEW.md`
- `raw/articles/2026-07-29-How building software is changing at Anthropic.md`
- `raw/articles/2026-07-30-Formal methods with Hillel Wayne.md`

## Related

- [[ai-assisted-software-engineering/agentic-code-review-verification-bottleneck]]
- [[ai-assisted-software-engineering/how-building-software-is-changing-at-anthropic]]
- [[ai-assisted-software-engineering/how-langchain-built-an-agent-first-data-stack]]
- [[loop-engineering/agent-improvement-loops-and-rubrics]]
