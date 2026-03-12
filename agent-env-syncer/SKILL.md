name: agent-env-syncer
description: Generates a single source-of-truth workspace config that syncs skills and MCP server definitions across multiple AI coding agents (Claude Code, Cursor, Copilot, Windsurf, Cline) so they all use the same setup without drift. Use whenever a user says "my Claude Code and Cursor have different MCP setups", "keep my agents in sync", or "single source of truth for my AI tools".

# Agent Environment Syncer

Creates a `workspace.agents.json` master config and generates per-agent config files from it, so adding an MCP server or skill once propagates everywhere.

**Input**: List of agents in use + current configs
**Output**: `workspace.agents.json` master + per-agent generated configs

## Trigger phrases
- "agents out of sync"
- "MCP setup drifting"
- "installed skill in one agent but not others"
- "single source of truth for AI tools"
- "sync Claude Code and Cursor"

## Source
`SKILL-41.md`