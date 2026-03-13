---
name: agent-env-syncer
description: Generates a single source-of-truth workspace config that syncs skills and MCP server definitions across multiple AI coding agents (Claude Code, Cursor, Copilot, Windsurf, Cline) so they all use the same setup without drift. Use whenever a user says "my Claude Code and Cursor have different MCP setups", "keep my agents in sync", or "single source of truth for my AI tools".
---

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

## Step-by-step workflow

### STEP 1 — Audit current agent configs

Ask (or read from disk) which agents are in use:
- Claude Code (`~/.claude/` or `CLAUDE.md`)
- Cursor (`.cursor/mcp.json`, `.cursorrules`)
- GitHub Copilot (`.github/copilot-instructions.md`)
- Windsurf (`.windsurf/`)
- Cline (`.cline/`)

For each agent in use, identify:
- MCP servers configured
- Skills/instructions installed
- Any agent-specific settings

Flag discrepancies: "Agent A has MCP server X but Agent B does not."

---

### STEP 2 — Build the master config

Create `workspace.agents.json`:

```json
{
  "version": "1.0",
  "workspace": "[project-name]",
  "mcp_servers": [
    {
      "name": "github",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" },
      "agents": ["claude", "cursor", "cline"]
    },
    {
      "name": "filesystem",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${WORKSPACE_ROOT}"],
      "agents": ["claude", "cursor"]
    }
  ],
  "skills": [
    {
      "name": "react-pr-reviewer",
      "path": "./skills/react-pr-reviewer/SKILL.md",
      "agents": ["claude", "cursor", "copilot"]
    }
  ],
  "instructions": [
    {
      "name": "project-conventions",
      "path": "./CLAUDE.md",
      "agents": ["claude", "cursor", "copilot", "windsurf", "cline"]
    }
  ]
}
```

---

### STEP 3 — Generate per-agent configs

From `workspace.agents.json`, generate:

**Claude Code** (`~/.claude/claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```

**Cursor** (`.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "github": { ... },
    "filesystem": { ... }
  }
}
```

**Copilot** (`.github/copilot-instructions.md`):
```markdown
[Contents of CLAUDE.md adapted for Copilot format]
```

**Windsurf / Cline**: Generate their respective config formats.

---

### STEP 4 — Write a sync script

Generate `scripts/sync-agents.sh`:

```bash
#!/bin/bash
# Sync agent configs from workspace.agents.json
# Run this after adding a new MCP server or skill

set -e
echo "Syncing agent configs..."

# Claude Code
cp .agent-configs/claude_desktop_config.json ~/.claude/claude_desktop_config.json
echo "✅ Claude Code updated"

# Cursor
mkdir -p .cursor
cp .agent-configs/cursor_mcp.json .cursor/mcp.json
echo "✅ Cursor updated"

# Copilot
mkdir -p .github
cp .agent-configs/copilot-instructions.md .github/copilot-instructions.md
echo "✅ Copilot updated"

echo "✅ All agents synced from workspace.agents.json"
```

---

### STEP 5 — Output summary

```
✅ workspace.agents.json created
  MCP servers defined: N
  Skills configured: N
  Agents covered: [list]

Generated configs:
  → .agent-configs/claude_desktop_config.json
  → .agent-configs/cursor_mcp.json
  → .github/copilot-instructions.md
  → scripts/sync-agents.sh

To apply: bash scripts/sync-agents.sh
To add a new MCP server: edit workspace.agents.json, then re-run sync.
```

---

## Output format spec

- `workspace.agents.json` — master config at project root
- `.agent-configs/` — folder of generated per-agent configs
- `scripts/sync-agents.sh` — apply script

---

## Quality checklist

- [ ] All agents in use are covered
- [ ] MCP servers have correct command/args format per agent
- [ ] Environment variables use `${VAR}` syntax (not hardcoded values)
- [ ] Sync script is idempotent (safe to run multiple times)
- [ ] Instructions file adapted to each agent's expected format

---

## Edge cases

- **Agent not supported**: Note it as unsupported in the master config; skip generation for that agent.
- **MCP server only available on some agents**: Use the `"agents"` array in master config to restrict scope.
- **User has custom per-agent settings that shouldn't be overwritten**: Add a `"merge": true` flag per server entry; generate a diff instead of full replacement.
- **Config already exists**: Show a diff before overwriting; prompt for confirmation.
