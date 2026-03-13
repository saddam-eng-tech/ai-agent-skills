---
name: rag-gap-auditor
description: Identifies knowledge gaps in RAG (Retrieval-Augmented Generation) setups by running structured test queries across categories, auditing documents for staleness or chunking issues, and producing a prioritised remediation plan. Use whenever a user says "my RAG isn't finding the right docs", "audit my knowledge base", "RAG is giving wrong answers", "improve my retrieval", or "why is my RAG missing context".
---

# RAG Gap Auditor

Systematically stress-tests a RAG pipeline by probing for gaps across query categories — then audits the document corpus for staleness, poor chunking, and missing coverage — and delivers a remediation plan ranked by impact.

**Input**: RAG system description + sample queries that fail
**Output**: Gap report by category + document audit findings + prioritised fixes

## Trigger phrases
- "audit my RAG"
- "knowledge base gaps"
- "RAG wrong answers"
- "improve retrieval"
- "RAG missing context"
- "why doesn't my RAG find this"

## Step-by-step workflow

### STEP 1 — Understand the RAG setup

Ask (or infer):
- What is the knowledge base? (docs, PDFs, wiki, database, etc.)
- What embedding model and vector store are in use?
- What retrieval strategy? (top-k, MMR, hybrid, re-ranking?)
- What LLM is doing generation?
- What are the failing queries? (get at least 5 examples)

---

### STEP 2 — Run structured gap probes

Generate 30 test queries across these gap categories:

| Category | # queries | What it probes |
|----------|-----------|----------------|
| Direct lookup | 5 | Single fact retrieval |
| Multi-hop | 5 | Answer requires combining 2+ chunks |
| Temporal | 5 | Questions about recent or time-sensitive info |
| Negation | 3 | "What does X NOT do" |
| Implicit | 5 | Answer is in docs but not keyword-matched |
| Cross-domain | 4 | Question spans multiple document categories |
| Edge / rare | 3 | Niche topics that may have sparse coverage |

For each query, record:
- Retrieved chunks (top-3)
- Whether the answer is actually present in any retrieved chunk
- Whether it's present in the corpus at all (search manually)
- Failure mode: NOT_RETRIEVED, NOT_IN_CORPUS, WRONG_CHUNK, STALE_DATA, CHUNK_BOUNDARY

---

### STEP 3 — Audit the document corpus

For each document source, check:

| Issue | How to detect |
|-------|---------------|
| Stale content | Check last-modified dates; flag docs > 6 months old |
| Poor chunking | Chunks that cut mid-sentence or mid-table |
| Duplicate content | Near-identical chunks inflating retrieval noise |
| Missing coverage | Gap categories from STEP 2 with no relevant docs |
| Metadata gaps | Chunks missing source, date, or section labels |
| Too-large chunks | Chunks > 1500 tokens burying specific facts |
| Too-small chunks | Chunks < 100 tokens losing context |

---

### STEP 4 — Classify failures by root cause

| Root cause | Fix |
|------------|-----|
| NOT_IN_CORPUS | Add missing documents |
| STALE_DATA | Refresh or remove outdated docs |
| CHUNK_BOUNDARY | Re-chunk with overlap or semantic boundaries |
| NOT_RETRIEVED | Improve embedding / add metadata filters / use hybrid search |
| WRONG_CHUNK | Re-rank results / add BM25 keyword layer |
| MULTI_HOP_FAIL | Add query decomposition step before retrieval |

---

### STEP 5 — Produce the remediation plan

Rank fixes by: (failures_affected × severity) / implementation_effort

```markdown
## RAG Gap Audit Report

### Summary
- Queries tested: 30
- Pass rate: N/30 (N%)
- Critical gaps: N categories

### Failure breakdown
| Failure mode | Count | % |
|--------------|-------|---|
| NOT_IN_CORPUS | N | N% |
| STALE_DATA | N | N% |
| CHUNK_BOUNDARY | N | N% |
| NOT_RETRIEVED | N | N% |

### Document corpus issues
| Issue | Affected docs | Severity |
|-------|---------------|----------|
| Stale (>6mo) | N docs | HIGH |
| Poor chunking | N docs | MEDIUM |

### Remediation plan (ranked by impact)
1. [HIGH] Add missing docs for [category] — fixes N failures
2. [HIGH] Re-chunk [source] with 200-token overlap — fixes N failures
3. [MEDIUM] Add hybrid BM25 + vector retrieval — fixes N failures
4. [MEDIUM] Refresh [N] stale documents — fixes N failures
5. [LOW] Add metadata date filters — fixes N failures
```

## Output format spec

- Gap report: markdown table with per-category results
- Corpus audit: table of issues per document source
- Remediation plan: ranked list with estimated impact
- Test query dataset: saved as `rag_audit_queries.jsonl` for regression testing

## Quality checklist

- [ ] At least 30 test queries generated across all 7 categories
- [ ] Each failing query has a classified root cause
- [ ] Corpus audit covers all document sources
- [ ] Remediation plan ranked by impact/effort ratio
- [ ] Queries saved for future regression testing

## Edge cases

- **No access to corpus directly**: Ask user to run sample queries and share retrieved chunks; infer gaps from the evidence.
- **Proprietary vector store**: Focus audit on query formulation and chunking strategy rather than index internals.
- **Small corpus (< 50 docs)**: Coverage gaps are likely the main issue; focus STEP 3 on what's missing, not chunking.
- **User has no failing query examples**: Generate synthetic "trap queries" that test common RAG failure modes instead.
