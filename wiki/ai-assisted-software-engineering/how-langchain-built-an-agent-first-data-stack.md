# How LangChain Built an Agent-First Data Stack

## Summary
LangChain rearchitected its data infrastructure to serve both traditional BI users and autonomous agents, moving away from a dashboard-centric model toward one designed for self-serve analysis, shared context, and agent use. The key insight is that agents require different data layer capabilities than humans: they need clear definitions, trusted sources, business context, and access to the logic behind the data—not just query execution. Without this contextual scaffolding, agents can generate SQL but produce answers that are technically valid yet untrustworthy or unhelpful for business needs. The migration created a unified platform serving multiple user patterns (dashboards, notebooks, conversational interfaces, Slack integrations) while keeping everything in a single place to preserve context and trust across agent and human interactions.

## Key Takeaways
- Traditional data stacks built around dashboards and static reports don't support agent autonomy; agents need definitions, trusted sources, business context, and data logic accessibility.
- A successful agent-first data layer must answer two simultaneous questions: what does the data mean in business terms? And which specific sources did the agent use to derive this answer?
- Support multiple interaction surfaces simultaneously—dashboards for polished reporting, notebooks for technical exploration, conversational interfaces for casual queries, and Slack/CLI integrations where people already work—to drive adoption across roles.
- Keep the entire data experience in one unified system rather than splitting usage between old and new tools; fragmentation destroys context and erodes trust across agent-human handoffs.
- Design agent queries to be explainable by construction: every agent response should surface the source definitions and calculation logic, enabling humans to audit and validate without reverse-engineering the chain of thought.
- For non-technical users, provide guided question-asking that returns not just an answer but also traces which sources were used and how they contributed—the audit trail is as important as the result itself.

## Related
- [[ai-assisted-software-engineering/how-building-software-is-changing-at-anthropic]]
- [Source: LangChain Blog](https://www.langchain.com/blog/agent-data-stack)