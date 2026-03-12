name: agent-trace-analyzer
description: Examines Claude Code agent session logs and traces to explain decisions made, identify root causes of failures or unexpected behaviour, and suggest targeted fixes including CLAUDE.md updates or tool config changes. Use whenever a user says "why did Claude do that", "analyse my agent session", "Claude made a wrong decision", "debug my agent run", or "explain this agent trace".

# Agent Trace Analyzer

Reads an agent session log and produces a structured post-mortem: what each decision was, why it was made, what went wrong, and what to change to fix it — classified by issue type (missing instructions, context rot, wrong tool, hallucination).

**Input**: Agent session log or trace output
**Output**: Decision-by-decision analysis + root cause classification + remediation steps

## Trigger phrases
- "why did Claude do that"
- "analyse agent session"
- "debug agent run"
- "explain agent trace"
- "Claude made wrong decision"
- "agent went off rails"

## Source
`SKILL-46.md`