name: mcp-tool-pruner
description: Audits an MCP server configuration to identify tool overload — too many tools flooding the context window — and produces a pruned, scoped configuration that groups tools by workflow and removes unused ones. Use whenever a user says "my Claude is slow with MCP", "too many MCP tools", "MCP is eating my context window", or "Claude picks the wrong tool".

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

## Source
`SKILL-39.md`