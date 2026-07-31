---
title: "Andrew Ng's Agentic Patterns, Mapped to GraphsNEW"
source_url: "https://www.aibuilderclub.com/blog/andrew-ng-loop-to-graph-engineering"
ingested: 2026-07-30
blog: "AI Builder Club"
published: "2026-07-29"
---
## Source Metadata
- **Blog:** AI Builder Club
- **Published:** 2026-07-29
- **URL:** https://www.aibuilderclub.com/blog/andrew-ng-loop-to-graph-engineering
- **Fetched:** 2026-07-31

## Article Content

Andrew Ng's Agentic Design Patterns, Mapped to Graphs

Andrew Ng published reflection, tool use, planning, and multi-agent collaboration as design patterns for agentic workflows. His March 2024 post does not present them as an ordered progression from loop engineering to graph engineering. This article keeps his source claim intact and adds a separate implementation view: how we at AI Builder Club might place those patterns inside an agent graph.
The distinction matters because two widely shared X posts describe an Andrew Ng PDF that neither post links. Their loop-to-graph packaging can still prompt a useful architecture discussion, as long as it is labeled as someone else's synthesis rather than Ng's method.
What Ng actually published
In his March 2024 post, Ng called reflection, tool use, planning, and multi-agent collaboration "four design patterns for AI agentic workflows." He described agentic work as prompting an LLM multiple times so it can build toward a better result.
His reflection example also rules out a narrow one-node interpretation. Ng wrote that tools such as unit tests or web search can evaluate an output. He then described another implementation with a generator agent and a separate critic agent. Reflection can be a self-review cycle, a tool-backed evaluation cycle, or a collaboration between agents.
The current Agentic AI course teaches the same patterns alongside evaluation and error analysis. Its access language is specific: free auditors can watch course videos and participate in the community forum, while assessments, certificates, and project tools require Pro membership. The Pro description also names quizzes and interactive labs.
None of those sources calls the patterns steps, orders them as a maturity ladder, or identifies multi-agent collaboration with a graph. The loop-to-graph order below is our synthesis.
What we could verify about the PDF claims
The @0xCodila post says Ng released an 8-page PDF, while the @0xMovez post claims 12 pages and uses a different title. Neither post links a document hosted by Ng or DeepLearning.AI.
Our check was bounded. As of 2026-07-29, we found no primary-source graph engineering PDF in the sources listed for this article. That finding supports caution about these two claims; it does not prove that no document can exist anywhere.
The useful material is already available directly through Ng's design-pattern post, Agentic AI course, and later feedback-loop letter. Readers do not need an unlinked PDF claim to study them.
