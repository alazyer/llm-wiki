# Wiki Compilation Log

## 2026-07-31 — PDF Compile: Building an AI-Native Engineering Team

**Raw source processed:**
- `raw/2026-07-31-Building an AI-native engineering team.md` — MarkItDown extraction from `building-an-ai-native-engineering-team.pdf`.

**New article compiled:**
- [[ai-assisted-software-engineering/ai-native-engineering-team-sdlc|AI-Native Engineering Team SDLC]] — Coding-agent patterns across planning, design, build, test, review, documentation, and operations.

**Index updated:**
- ai-assisted-software-engineering/_index.md

## 2026-07-31 — Chrome Fetch for Non-Hugging Face Articles

**Raw article fetch status:**
- Filled 400 duplicate non-Hugging Face placeholders from existing fetched raw files with matching `source_url` values.
- Used Chrome-rendered pages to extract 25 unique non-Hugging Face article URLs into 42 raw files.
- Skipped Hugging Face because it is blocked/unreliable from the current Mainland China network path; 44 Hugging Face placeholders remain pending.

**New digest articles compiled:**
- [[ai-agent-operations/agent-cost-security-and-ownership-controls|Agent Cost, Security, and Ownership Controls]] — Permission canaries, owner registries, budget attribution, and AI security-incident lessons.
- [[ai-native-organizations/ai-native-enterprise-operating-environment|AI-Native Enterprise Operating Environment]] — Enterprise operating surfaces that make work legible and safe for agents.
- [[ai-assisted-software-engineering/ai-review-formal-methods-and-agentic-dev|AI Review, Formal Methods, and Agentic Development]] — Review evidence, formal-methods pressure, and verification practices for AI-generated code.
- [[loop-engineering/seo-and-social-agent-loops|SEO and Social Agent Loops]] — Growth-channel loops with external signals, no-op valves, and human gates.
- [[graph-engineering/agentic-patterns-loops-and-graphs|Agentic Patterns, Loops, and Graphs]] — Mapping agentic design patterns onto loops and graph seams.
- [[agent-platform-operations/deep-agents-v07-context-harness|Deep Agents v0.7 Context Harness]] — Leaner prompts, middleware, and filesystem runtime ergonomics.
- [[agent-evaluation-observability/similarweb-agent-report-evaluation|Similarweb Agent Report Evaluation]] — Rubrics, datasets, judges, and traces for long-form agent report evals.
- [[ai-technology-briefs/ai-compute-energy-and-market-briefs-2026-07-30|AI Compute, Energy, and Market Briefs: 2026-07-30]] — Adjacent AI, energy, compute, and market context.

**Indexes updated:**
- ai-agent-operations/_index.md
- ai-native-organizations/_index.md
- ai-assisted-software-engineering/_index.md
- loop-engineering/_index.md
- graph-engineering/_index.md
- agent-platform-operations/_index.md
- agent-evaluation-observability/_index.md
- ai-technology-briefs/_index.md
- ai-agents/_index.md
- pending-ingest/_index.md

## 2026-07-29 — Daily Compile

**New articles compiled (6 from AI Builder Club, LangChain, Pragmatic Engineer, MIT Technology Review):**

### ai-agent-operations/
- [[ai-agent-operations/agent-tool-permissions-canary-test-your-deny-rules|Agent Tool Permissions: Test That Your Deny Rules Hold]] — A canary testing framework for verifying actual runtime enforcement of agent tool permissions through active bypass attempts. (Source: AI Builder Club)
- [[ai-agent-operations/your-agents-have-production-credentials-and-no-owner|Your Agents Have Production Credentials and No Owner]] — An operational schema for auditing which human owns each running agent, what credentials it uses, what it could reach if things went wrong, and what happens on shutdown. (Source: AI Builder Club)

### ai-assisted-software-engineering/
- [[ai-assisted-software-engineering/how-building-software-is-changing-at-anthropic|How Building Software Is Changing at Anthropic]] — Shift from traditional code development toward model-augmented and agent-assisted workflows with autonomous code participation in CI/CD. (Source: Pragmatic Engineer)
- [[ai-assisted-software-engineering/how-langchain-built-an-agent-first-data-stack|How LangChain Built an Agent-First Data Stack]] — Architectural shift from dashboard-centric data stacks to agent-supporting infrastructure with explainable sources and unified human-agent experience. (Source: LangChain Blog)

### ai-agents/
- [[ai-agents/openai-predictable-hack-ai-stock-sell-off|The Download: OpenAI's Predictable Hack, and an AI Stock Sell-Off]] — Security incident patterns affecting investor confidence across the AI ecosystem; market response to repeated vulnerability disclosures. (Source: MIT Technology Review)
- [[ai-agents/samsung-chip-workers-jump-sku-hynix|Samsung's Chip Workers Are Jumping Ship to Rival SK Hynix]] — Talent migration in semiconductor industry impacting hardware continuity and AI accelerator supply chain stability. (Source: MIT Technology Review)

**Index files updated:**
- ai-agent-operations/_index.md
- ai-assisted-software-engineering/_index.md
- ai-agents/_index.md

**Correction on 2026-07-31:**
- Hugging Face-derived entries from the 2026-07-29 compile were moved back to pending ingest because the corresponding `raw/articles/` files are metadata-only placeholders, not fetched article bodies.
