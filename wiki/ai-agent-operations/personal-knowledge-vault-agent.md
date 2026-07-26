# Personal Knowledge Vault Agent

## Summary
A self-hosted agent can become a durable personal context system by using built-in memory only as an index card to a markdown vault, then filing life, work, project, people, source, and decision facts with dates, status, confidence, evidence, and regular health checks.

## Key Takeaways
- The central problem is scattered personal context across notes, repos, chat history, databases, and old exports; each new AI session otherwise makes the human re-integrate context manually.
- Built-in agent memory is too small for a life-scale knowledge base, so memory should point to the vault path and retrieval rules rather than store all facts directly.
- The proposed vault uses predictable numbered folders: inbox, identity, projects, areas, people, decisions, sources, and meta files such as indexes or changelogs.
- Every note should carry frontmatter such as id, type, `as_of` date, status, and confidence so facts can age, conflict, and be superseded cleanly.
- A vault root `AGENTS.md` should tell the agent never to answer from memory alone and to open files before making claims.
- A filing skill should search before creating notes, handle contradictions by linking current and superseded notes, avoid overwriting in place, and append changelog entries.
- The answer protocol should build evidence cards with status, date, confidence, file and line range, and known contradictions; missing vault evidence should be reported as missing instead of filled in from general knowledge.
- Braindumps and imported archives should enter as untrusted inbox/source material first, then be processed with user clarification before becoming durable facts.
- Weekly vault health checks should report stale summaries, missing metadata, duplicate ids, broken links, and long-current notes without silently fixing them.

## Related
- [[ai-agent-operations/openclaw-memory-architecture]]
- [[ai-agent-operations/taste-index-hermes-gbrain]]
- [[agent-harness-engineering/anatomy-of-agent-skills]]
- [[pending-ingest/ai-second-brain-claude-obsidian-empty]]