# How We Built Our Knowledge Base

**Source:** https://www.cerebras.ai/blog/how-we-built-our-knowledge-base
**Authors:** Isaac Tai, Daniel Kim, Mike Gao
**Date:** July 15, 2026

---

Employees ask Cerebras's internal knowledge base more than 15,000 questions every day. It's become one of the most widely adopted internal tools at the company since launching 3 months ago. Used by humans, automations and agents.

## Architecture Overview

```
SOURCES (Slack, Wiki, Code, Incidents)
    ↓
DISTILLATION (LLM Extractors)
    ↓
EMBEDDINGS (PGVector - 3072-dim - HNSW)
    ↓
RETRIEVAL (Six Lists in Parallel)
    ↓
FUSION + RERANK (RRF K=60 → LLM Rerank)
    ↓
SYNTHESIS (Answer + Citations)
```

## Key Design Principle: Meet Data Where It Lives

Rather than forcing all information into one platform, Cerebras built a system that extracts data from each tool directly — Slack, code repos, Confluence, netlists, custom databases. Every source lands in the same embeddings table and is immediately queryable through the same interface.

---

## Anatomy of the Knowledge Base

Three layers:

1. **Collection & Storage** — single Postgres table for embeddings, raw summaries, and metadata
2. **Query Platform** — unified interface across all sources
3. **Auth & Audit Layer** — authentication, authorization, auditing, analytics

### Uniform Data Interface

Every source defines: what the data is, how to connect, fetch frequency. Each embedding row follows the same schema:

```
DOCUMENT | EMBEDDING | METADATA | SOURCE + TIMESTAMPS
```

Queryable via: **MCP · Web UI · Agents**

---

## Slack Integration

Slack was the most important data source — where up-to-date engineering discussions happen.

### Architecture

```
Socket Event → Route → Sync Worker → Distill → Thread Vector + Burst Vectors
                         ↓
                   Bot Reply / Reingest Thread / Upsert Thread
```

- **Socket Mode:** Real-time event ingestion via persistent WebSocket — no polling, no rate-limit concerns
- **Deduplication** via stable event ID
- **Thread-level ingestion:** Each message arrival re-fetches the entire thread (parent + all replies), so stored content, participants, and timestamps always reflect the complete conversation
- **Per-channel tuning:** Each Slack channel is its own data source with independent freshness settings

### Four Complementary Search Signals

No single scorer is trusted on its own:

| # | Technique | What It Catches |
|---|-----------|----------------|
| 1 | **Full-text search** | Exact tokens: error strings, flag names, host names. Lexical match trumps semantic similarity for pasted errors |
| 2 | **Embedding search** | Paraphrase across vocabulary. "restore hangs after manifest load" → "checkpoint stalls on NFS mount" |
| 3 | **Inverse Document Frequency (IDF)** | Rare tokens beat filler. Obscure config flags rank high; "sounds good, thanks!" scores near zero |
| 4 | **Age decay** | Slack answers expire. Newer threads win ties (8-month-old answers may describe defunct infrastructure) |

### Distillation Pipeline

Raw Slack text → LLM extraction → Structured artifact per thread:

```json
{
  "question": "Why does restore stall after manifest load?",
  "summary": "Large restores stop before cache warmup.",
  "resolution": "Set CKPT_PREFETCH=4 for the NFS mount.",
  "systems": ["checkpoint restore", "NFS"],
  "code_refs": ["CKPT_PREFETCH"]
}
```

**Key finding:** Embedding the normalized thread artifact significantly outperforms embedding the raw transcript. Consistent format gives the semantic match more useful signal.

### Bursting

Individual messages within long threads can be missed by thread-level summaries.

- A **burst** = run of consecutive messages from the same author
- Bursts are embedded with the thread topic prepended as context
- **Quality threshold** before embedding:
  - IDF ≥ 4.0 (contains rare tokens)
  - Combined burst ≥ 200 characters
  - Reactions provide social boost
- Qualifying bursts are embedded alongside the thread-level record

---

## Code Repositories

Some repos are over 40 GB. Key concerns: keeping embeddings current efficiently.

### CocoIndex for Code Embeddings

Used CocoIndex — open-source document embedding framework specializing in vectorizing codebases.

- **Language-aware recursive chunking:** Class → method → smaller blocks. High-level regex boundaries first, then finer splits
- **Incremental updates:** Sync metadata in Postgres. On each commit, only changed chunks re-embed and re-export (not whole repo)
- **Team self-service:** Teams submit config files with allowlists/denylists at the file-path level

---

## Custom Data Sources

Teams with existing databases can participate without moving data:

- Open a PR with a small Python module that reads from their system
- Emits rows matching the embeddings table schema + a data source entry
- Becomes queryable alongside Slack, code, and documents — no special handling needed

---

## Query Pipeline

### Planning & Tool Fan-Out

For every query, an LLM planning pass decides which tools to use:

| Tool | Purpose |
|------|---------|
| `subsystem_index` | Per-file LLM summaries |
| `search` | Unified vector pipeline (Slack, wiki, code, etc.), merged + reranked |
| `search_slack` | Direct Slack retrieval |
| `search_specific` | Ripgrep over source repos |
| `recent_prs` | Recent pull requests |
| `who_knows` | People withdemonstrated expertise |

**Flow:** Planner → Parallel execution → Normalized evidence → Synthesis (Answer + Citations)

### Reranking

**RRF (Reciprocal Rank Fusion)**: score(d) = Σ 1 / (60 + rank)

- Smoothing constant K=60 → consensus across retrievers beats a single #1 ranking
- Duplicate chunks merged back to source
- Cap results per file for diversity
- Top 20 → small reranker model (0-10 score) → top 10 kept
- **Context expansion:** Wiki sections expanded to neighboring sections so preconditions and caveats aren't lost

---

## MCP Integration

Exposes retrieval building blocks as **direct tools** (not one monolithic "answer this question" endpoint):

- `search_slack`, `search_code`, `search`, `who_knows` — each corresponds to one underlying primitive
- Inputs/outputs are narrow, structured, stable
- Most tools are **LLM-free** — run one query pipeline + lightweight heuristics → raw evidence rows
- Claude Code (or any MCP-compatible agent) as the orchestration engine

### Web UI vs MCP

| Aspect | Web UI | MCP |
|--------|--------|-----|
| Orchestration | Server-side planner → executor → synthesizer | Client orchestrates |
| Flow | Ask question → get answer with citations | Direct tool calls → raw evidence rows |
| LLM dependency | Full pipeline with LLM at every step | Agent/client decides what to call |

---

## Projects & Scoped Search

- **Projects** = named bundles of data sources (Slack channels, repos, databases, document spaces)
- Same source can be referenced by multiple projects (no duplication)
- **Onboarding:** user picks a default project (ML training, Compiler, Data Center Ops, etc.) — queries auto-scoped
- Eliminates the "search everywhere" problem

---

## Key Takeaways

1. **Meet data where it is** — don't force migration to one platform
2. **Single embeddings table** simplifies everything: one schema, one query interface, one connector per source
3. **Hybrid search** is essential — FTS + embeddings + IDF + age decay each compensate for the others' weaknesses
4. **Distill first, embed later** — normalizing raw Slack threads into structured artifacts significantly improves accuracy
5. **Bursting** catches important sub-messages that thread-level summaries miss
6. **Incremental code indexing** via CocoIndex avoids recomputing entire repos on every commit
7. **MCP tools should be LLM-free primitives** — simple, deterministic retrieval that agents compose
8. **RRF + reranker** fusion → diverse top results from multiple retrievers
9. **Projects/scope** prevent the "search everything" decay as the corpus grows
10. **Per-channel freshness tuning** — busy incident channels get more frequent ingestion

---

## References

1. Malkov and Yashunin, *Efficient and Robust Approximate Nearest Neighbor Search Using HNSW*, arXiv:1603.09320
2. Anthropic, *Introducing Contextual Retrieval*, 2024
3. Cormack, Clarke, Büttcher, *RRF Outperforms Condorcet and Individual Rank Learning Methods*, SIGIR 2009
4. Li et al., *Search-o1: Agentic Search-Enhanced Large Reasoning Models*, arXiv:2501.05366
5. Anthropic, *Code Execution with MCP*, 2025
6. Liu et al., *Lost in the Middle: How Language Models Use Long Contexts*, arXiv:2307.03172
7. Anthropic, *Use XML Tags*
8. Salesforce/Slack Engineering, *How Slack AI Processes Billions of Messages*
9. Improving Agents, *Best Nested Data Format*
10. Cursor, *Improving Agent with Semantic Search*, 2025