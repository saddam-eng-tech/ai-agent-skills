name: rag-gap-auditor
description: Identifies knowledge gaps in RAG (Retrieval-Augmented Generation) setups by running structured test queries across categories, auditing documents for staleness or chunking issues, and producing a prioritised remediation plan. Use whenever a user says "my RAG isn't finding the right docs", "audit my knowledge base", "RAG is giving wrong answers", "improve my retrieval", or "why is my RAG missing context".

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

## Source
`SKILL-47.md`