# How Building Software Is Changing at Anthropic

## Summary
This Pragmatic Engineer piece examines Anthropic's internal transformation of software development practices as they incorporate agentic systems into their engineering workflows. The article discusses the shift from traditional code-centric development toward model-augmented and agent-assisted development patterns—where agents are not just end products but active participants in the build process itself. It covers emerging architectural decisions around agent-assisted testing, automated code review, CI/CD integration, and the organizational implications of introducing agents that can modify code and infrastructure autonomously. While specific technical details aren't provided in the summary, the piece represents a first-hand account from one of the leading AI companies navigating this transition.

## Key Takeaways
- Anthropic is moving toward agent-assisted software engineering, where agents participate in coding, testing, and CI/CD rather than merely being the output of development teams.
- Traditional development workflows (review → test → deploy) are being augmented with agent-driven iterations: agents propose changes, run tests, suggest fixes, and even initiate deployments under guardrails.
- The change requires new operational patterns: code must be written with verifiability in mind, tests must be agent-executable, and deployment pipelines must include agent-specific safety checkpoints.
- Organizational adoption of agentic workflows introduces new governance questions—what parts of the codebase agents can touch, what budget limits they operate under, and who retains final sign-off authority.
- Early adopter experiences suggest the biggest friction point isn't technical capability but establishing shared mental models between human engineers and autonomous development agents about when to act versus when to seek approval.

## Related
- [[ai-assisted-software-engineering/loop-engineering-14-step-roadmap]]
- [[graph-engineering/graph-engineering-basics]]
- [Source: Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/inside-anthropic)