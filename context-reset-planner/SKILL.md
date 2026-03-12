name: context-reset-planner
description: Detects context rot in long LLM/Claude Code sessions and produces a structured context reset: a compact summary of what's been decided, what's in progress, and what comes next — ready to paste into a fresh session. Use whenever a user says "my Claude session is going off the rails", "Claude keeps forgetting earlier decisions", "context window almost full", or "should I start a new chat?".

# Context Reset Planner

Diagnoses context rot symptoms and generates a clean handoff document that preserves all decisions and state so a fresh session picks up exactly where the good work ended.

**Input**: Current task description + what's been done/decided
**Output**: Compact context handoff doc + fresh session starter prompt

## Trigger phrases
- "session going off rails"
- "context window full"
- "Claude forgetting decisions"
- "start fresh session"
- "LLM loop of doom"
- "should I reset chat"

## Source
`SKILL-40.md`