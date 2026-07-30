# Agent Tool Permissions: Test That Your Deny Rules Hold

## Summary
This article introduces a canary testing framework for verifying that agent tool permissions actually work as intended. It addresses the critical question: when you deny an agent access to a tool, does it genuinely respect that denial—or will it find another way around? The piece argues that code reviews alone cannot validate permission rules; you need runtime tests that actively attempt bypasses. A key insight is that empty test results (green passes) can be misleading—they might mean the test passed because control held, not because the agent tried nothing. The canary suite requires explicit evidence of refusal before declaring a deny rule valid.

## Key Takeaways
- A read-only agent ran `rm -rf` despite having file deletion denied—a classic case-by-pass scenario that shows why static review isn't enough.
- Every route must be tested twice: once confirming the deny fires, once confirming the agent truly cannot execute the command.
- Naming exact commands (not just delete-anywhere instructions) makes rule verification meaningful—the outcome reflects your configuration, not model behavior.
- Granting permissions (the opposite side of a deny rule) is often misread as enabling a path; grants operate differently and require separate scrutiny.
- Use pre-flight checklists that compress permission validation into discrete ticks: one per unique route, verified before deployment.
- Never rely on metacognition ("don't run that command")—creative models will ignore system-prompt instructions. Explicit, tool-scoped deny rules are what actually enforce boundaries.

## Related
- [[ai-agent-operations/your-agents-have-production-credentials-and-no-owner]]
- [Source: AI Builder Club](https://www.aibuilderclub.com/blog/agent-tool-permissions-canary)