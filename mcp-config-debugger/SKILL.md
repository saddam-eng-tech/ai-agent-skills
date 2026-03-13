---
name: mcp-config-debugger
description: Diagnoses and fixes MCP (Model Context Protocol) configuration errors including JSON syntax mistakes, incorrect server paths, missing dependencies, port conflicts, and auth failures. Use whenever a user says "my MCP server isn't working", "Claude can't connect to MCP", "MCP server not found", "my claude_desktop_config.json is broken", or "MCP tool not showing up in Claude".
---

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

## Step-by-step workflow

### STEP 1 — Collect the config and error

Ask the user to provide:
1. The full contents of `claude_desktop_config.json` (or equivalent)
2. The exact error message from Claude / the MCP server logs
3. Operating system (macOS / Windows / Linux)
4. How they installed the MCP server (npx / global npm / uvx / binary)

If they can't find the config file, provide the default path:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

---

### STEP 2 — Run the diagnostic checklist

Check each category in order:

#### 2a — JSON syntax
- Is the JSON valid? (no trailing commas, no unquoted keys, no missing brackets)
- Common mistake: trailing comma after the last server entry

```json
// BROKEN
{
  "mcpServers": {
    "github": { ... },   // <-- trailing comma
  }
}

// FIXED
{
  "mcpServers": {
    "github": { ... }
  }
}
```

#### 2b — Command path
- Is `command` pointing to an executable that exists?
- For `npx`: is Node.js installed? Run `node --version`
- For `uvx`: is `uv` installed? Run `uv --version`
- For a binary path: does the file exist at that path?

Common mistakes:
- `"command": "npx"` fails on some systems where npx is not in PATH when launched by Claude; fix with full path: `/usr/local/bin/npx`
- Windows path separators: use `\\` not `\` in JSON strings

#### 2c — Arguments
- Are `args` in the correct array format?
- For npx-based servers: must include `-y` flag to avoid interactive prompts

```json
// BROKEN — missing -y
"args": ["@modelcontextprotocol/server-github"]

// FIXED
"args": ["-y", "@modelcontextprotocol/server-github"]
```

#### 2d — Environment variables
- Are required env vars set in the `env` block?
- Are they referencing actual values (not empty strings or placeholder text)?

```json
// BROKEN — placeholder not replaced
"env": { "GITHUB_TOKEN": "your-token-here" }

// FIXED
"env": { "GITHUB_TOKEN": "ghp_actualtoken123" }
```

#### 2e — Port conflicts (for HTTP-based MCP servers)
- Is the server trying to bind a port already in use?
- Check with: `lsof -i :[port]` (macOS/Linux) or `netstat -ano | findstr :[port]` (Windows)

#### 2f — Package not installed
- For npx servers: the package may need a one-time install
- Run: `npx -y [package-name]` manually in terminal to verify it works

---

### STEP 3 — Identify root cause

Based on the checklist, state the root cause clearly:

```
❌ Root cause: [category] — [specific problem]
Example: JSON syntax — trailing comma after "filesystem" server entry
Example: Command path — npx not found in PATH when Claude launches
Example: Env vars — GITHUB_TOKEN is set to placeholder value
```

---

### STEP 4 — Produce the corrected config

Output the full corrected `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "[server-name]": {
      "command": "[corrected-command]",
      "args": ["corrected", "args"],
      "env": {
        "REQUIRED_VAR": "[value]"
      }
    }
  }
}
```

Annotate each fix with a comment (in the explanation, not in the JSON) explaining what was changed and why.

---

### STEP 5 — Verify steps

Give the user steps to verify the fix:

1. Save the corrected config
2. Fully quit Claude (not just close window) — `Cmd+Q` / `Alt+F4`
3. Relaunch Claude
4. Check: does the tool appear in Claude's tool picker?
5. Test with a simple command: "List my GitHub repos" (for github MCP) etc.

If still failing, ask for the MCP server logs:
- macOS: `~/Library/Logs/Claude/mcp-server-[name].log`
- Windows: `%APPDATA%\Claude\logs\mcp-server-[name].log`

---

### STEP 6 — Output summary

```
✅ MCP Config Debugger — Fix Summary

  Server: [server-name]
  Root cause: [category] — [specific issue]
  Fix applied: [what was changed]

  Next: Save config → Quit Claude fully → Relaunch → Test
```

---

## Output format spec

- Root cause statement (1 line, specific)
- Full corrected JSON config block
- Annotated change explanation
- Verify steps (numbered)

---

## Quality checklist

- [ ] JSON is valid (validate before outputting)
- [ ] Root cause is specific (not "something is wrong")
- [ ] Full corrected config provided (not just the changed line)
- [ ] Verify steps include "fully quit and relaunch Claude"
- [ ] Log file location provided for further debugging

---

## Common error reference

| Error message | Likely cause | Fix |
|---------------|-------------|-----|
| `spawn npx ENOENT` | npx not in PATH | Use full path to npx |
| `SyntaxError: Unexpected token` | Invalid JSON | Fix trailing comma / bracket |
| `Error: connect ECONNREFUSED` | Server not starting | Check args + dependencies |
| `Tool not appearing in Claude` | Config not reloaded | Fully quit + relaunch Claude |
| `GITHUB_TOKEN: invalid` | Placeholder value | Replace with real token |
| `MODULE_NOT_FOUND` | Package not installed | Run `npx -y [package]` manually |

---

## Edge cases

- **Multiple broken servers**: Fix the JSON structure first (one broken server can prevent all others from loading), then address individual server issues.
- **Works on terminal but not in Claude**: PATH issue — Claude has a different environment. Use absolute paths for commands.
- **Corporate/proxy network**: MCP servers using npx may fail to download. Suggest pre-installing packages globally: `npm install -g [package]`.
- **MCP server crashes after connecting**: This is a runtime error, not a config error — redirect to checking the MCP server's own logs and documentation.
