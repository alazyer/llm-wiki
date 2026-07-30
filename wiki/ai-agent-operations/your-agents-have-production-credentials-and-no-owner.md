# Your Agents Have Production Credentials and No Owner

## Summary
This article explores a critical but often overlooked problem in agent deployments: autonomous agents running with production-level credentials while having no clear human owner or accountability mechanism. The piece examines a real-world case where an internal reconciliation agent ran for months with broad database access—read and write—yet no one had audited its actual behavior, because there was no systematic way to track what the agent was doing beyond application logs. It introduces a schema (a 10-field checklist) for auditing individual agents: who gets asked by name what it did last night, what it could reach if things went wrong, and what happens when you turn it off off. This isn't just about compliance—it's about closing the visibility gap between "it has probably been fine" versus "I don't know that it has been fine."

## Key Takeaways
- An agent with broad credentials can operate for months without anyone checking its actual behavior; absence of incidents is not evidence of safety.
- The gap between "probably been fine" and "don't know that it's been fine" is the core operational problem—and naming it is the first step toward fixing it.
- Use a schema-based audit for each running agent: identify its owner, review its reachable scope, define its failure mode impact, and document its shutdown procedure.
- Ten fields capture the essential audit dimensions; each field has a non-guesswork method of completion. A row that cannot be filled in signals incomplete governance.
- The schema records claims only—it does not enforce anything. Its value is making wrong claims visible so they become questions rather than blind spots.
- Start small: pick one agent you already run, apply the three-question test (who owned it last night? what could it reach worst-case? how would you turn it off?), then expand fleet-wide.

## Related
- [[ai-agent-operations/agent-tool-permissions-canary-test-your-deny-rules]]
- [Source: AI Builder Club](https://www.aibuilderclub.com/blog/who-owns-your-ai-agents)