name: mcp-config-debugger
description: Diagnoses and fixes MCP (Model Context Protocol) configuration errors including JSON syntax mistakes, incorrect server paths, missing dependencies, port conflicts, and auth failures. Use whenever a user says "my MCP server isn't working", "Claude can't connect to MCP", "MCP server not found", "my claude_desktop_config.json is broken", or "MCP tool not showing up in Claude".

# MCP Config Debugger

Systematically diagnoses why an MCP server isn't connecting — JSON errors, wrong paths, missing deps, env issues — and produces a working config fix.

**Input**: User's config file + error message
**Output**: Diagnosed root cause + corrected config

## Trigger phrases
- "MCP not working"
- "MCP server not found"
- "can't connect to MCP"
- "MCP tool missing"
- "claude_desktop_config.json error"

## Source
`SKILL-38.md`