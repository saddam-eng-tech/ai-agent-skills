name: subagent-scaffolder
description: Generates complete Claude Code sub-agent definitions — system prompts, tool restrictions, and CLAUDE.md entries — for specialized agents like code reviewer, test runner, security scanner, or documentation writer. Use whenever a user says "create a sub-agent for", "scaffold a Claude Code subagent", or "I want a specialized agent that only does X".

# Subagent Scaffolder

Produces complete, ready-to-install Claude Code sub-agent definitions that keep specialized roles focused and prevent scope creep in agentic workflows.

**Input**: Description of what the sub-agent should do and its constraints
**Output**: Sub-agent system prompt + CLAUDE.md entry + tools config

## Trigger phrases
- "create sub-agent"
- "scaffold specialized agent"
- "read-only reviewer agent"
- "test runner agent"
- "agent that only does X"
- "worker agent for pipeline"

## Source
`SKILL-44.md`