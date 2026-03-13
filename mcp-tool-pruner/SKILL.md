---
name: mcp-tool-pruner
description: Audits an MCP server configuration to identify tool overload — too many tools flooding the context window — and produces a pruned, scoped configuration that groups tools by workflow and removes unused ones. Use whenever a user says "my Claude is slow with MCP", "too many MCP tools", "MCP is eating my context window", or "Claude picks the wrong tool".
---

# MCP Tool Pruner

Identifies which MCP tools are consuming context budget unnecessarily and produces a lean, workflow-scoped config that keeps agents fast and accurate.

**Input**: Claude MCP config file + usage context
**Output**: Pruned config + tool-group recommendations

## Trigger phrases
- "MCP too slow"
- "too many tools"
- "context window bloat from MCP"
- "Claude picks wrong tool"
- "MCP cost too high"
- "reduce MCP tools"

## Step-by-step workflow

### STEP 1 — Read the current MCP config

Ask the user to paste their `claude_desktop_config.json` (or `.cursor/mcp.json`, `.cline/mcp.json`, etc.).

Count:
- Total number of MCP servers
- Total number of tools across all servers (estimate if not visible)
- Estimated context tokens consumed by tool definitions

Rule of thumb: each tool definition costs ~200–500 tokens in context. If total tools > 20, flag as overloaded.

```
⚠️ Tool audit:
  Servers: N
  Estimated total tools: N
  Estimated context cost: ~N tokens
  Status: [OK / OVERLOADED / CRITICAL]
```

---

### STEP 2 — Classify tools by usage

For each MCP server, ask or infer:
- Is this server used daily / occasionally / rarely?
- Is it needed for the current project or just "installed"?
- Does it overlap with another server (e.g. two filesystem servers)?

Classify each server:

| Server | Usage | Overlap | Recommendation |
|--------|-------|---------|----------------|
| github | daily | none | keep |
| filesystem | daily | none | keep |
| browser-automation | rarely | none | move to on-demand |
| postgres-old | never | duplicates postgres-new | remove |

---

### STEP 3 — Group tools by workflow

Instead of loading all tools always, group them by task:

```json
{
  "profiles": {
    "coding": ["github", "filesystem", "bash"],
    "research": ["brave-search", "fetch"],
    "database": ["postgres", "filesystem"],
    "full": ["github", "filesystem", "bash", "brave-search", "postgres"]
  }
}
```

Recommend the user activate only the profile needed per task.

---

### STEP 4 — Produce the pruned config

Generate the leaner `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${WORKSPACE}"],
      "env": {}
    }
  }
}
```

Also produce the "full" config as a separate `claude_desktop_config.full.json` for when all tools are needed.

---

### STEP 5 — Write tool-restriction hints for CLAUDE.md

Add to `CLAUDE.md`:

```markdown
## MCP Tool Usage

Only call MCP tools when necessary. Prefer built-in tools (Read, Write, Bash) over MCP equivalents unless the MCP tool provides a clear advantage.

Do NOT use the following tools unless explicitly asked:
- browser-automation (slow, use only for visual testing)
- full-web-search (use only when codebase search fails)
```

---

### STEP 6 — Output summary

```
✅ MCP Tool Pruner complete

  Before: N servers, ~N tools, ~N tokens context cost
  After:  N servers, ~N tools, ~N tokens context cost
  Savings: ~N tokens per request (~N%)

  Removed: [list of removed servers]
  Kept: [list of kept servers]
  Moved to on-demand profile: [list]

Files generated:
  → claude_desktop_config.json (pruned default)
  → claude_desktop_config.full.json (all tools)
  → CLAUDE.md addition (tool usage hints)
```

---

## Output format spec

- Pruned `claude_desktop_config.json`
- Optional `claude_desktop_config.full.json` for power use
- CLAUDE.md snippet for tool usage guidance
- Summary table showing before/after token savings

---

## Quality checklist

- [ ] Pruned config has fewer than 10 servers
- [ ] No duplicate/overlapping servers remain
- [ ] Estimated context savings calculated and shown
- [ ] Profile grouping provided if user has 5+ servers
- [ ] CLAUDE.md hint added for infrequent-use tools
- [ ] Full config preserved separately (nothing lost)

---

## Edge cases

- **User only has 2–3 servers**: No pruning needed; confirm this and suggest profile grouping for future scaling.
- **User doesn't know which tools they use**: Ask them to describe their typical Claude session; infer usage from that.
- **Custom/private MCP server**: Keep it; do not remove servers you cannot identify — ask the user.
- **Server has 50+ tools** (e.g. a large API wrapper): Suggest contacting the server provider for a scoped/filtered version, or wrapping it with an allow-list proxy.
- **User wants all tools always**: Explain the trade-off (slower, more context used, more wrong-tool picks) and let them decide — do not force pruning.
