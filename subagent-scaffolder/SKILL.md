---
name: subagent-scaffolder
description: Generates complete Claude Code sub-agent definitions — system prompts, tool restrictions, and CLAUDE.md entries — for specialized agents like code reviewer, test runner, security scanner, or documentation writer. Use whenever a user says "create a sub-agent for", "scaffold a Claude Code subagent", or "I want a specialized agent that only does X".
---

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

## Step-by-step workflow

### STEP 1 — Gather sub-agent spec

From the user's request, extract:
- **Role name**: e.g. `code-reviewer`, `test-runner`, `security-scanner`
- **Purpose**: What exactly does this agent do?
- **Constraints**: What is it NOT allowed to do? (e.g. read-only, no file writes)
- **Tools**: Which tools does it need? (e.g. Read, Bash, WebSearch)
- **Input/Output**: What does it receive and what does it produce?

If purpose is unclear, ask one question: "What should this sub-agent specialise in?"

---

### STEP 2 — Design the system prompt

Write a focused system prompt following this structure:

```
You are a [role] agent. Your ONLY job is to [specific task].

## What you DO
- [Task 1]
- [Task 2]
- [Task 3]

## What you NEVER do
- Do not write or modify any files
- Do not run tests or execute code
- Do not make decisions outside [domain]
- If asked to do something outside your role, respond: "That is outside my scope. Please use the [appropriate agent] for that."

## Input format
[What you receive]

## Output format
[What you produce — be specific about structure]

## Quality bar
[What constitutes a good output for this role]
```

Rules:
- System prompt must be under 500 tokens for efficiency
- Always include explicit "never do" list
- Always define exact output format
- Use imperative, direct language

---

### STEP 3 — Define tool restrictions

Map the role to the minimal required tools:

| Agent type | Allowed tools |
|-----------|---------------|
| Code reviewer | Read only |
| Test runner | Read, Bash (test commands only) |
| Security scanner | Read, Bash (audit tools only) |
| Documentation writer | Read, Write (docs folder only) |
| Dependency updater | Read, Write (package files only), Bash |
| Database migrator | Read, Write (migrations only), Bash |

Generate the tools config:
```json
{
  "tools": {
    "allowed": ["Read"],
    "denied": ["Write", "Edit", "Bash", "WebSearch"]
  }
}
```

---

### STEP 4 — Write the CLAUDE.md entry

Generate the entry to add to the project's CLAUDE.md:

```markdown
## Sub-agents

### [agent-name]
**Role**: [one-line description]
**Trigger**: Use this agent when [specific condition]
**Tools**: [list]
**Scope**: [what it touches]
**Output**: [what it produces]
```

---

### STEP 5 — Output all three artefacts

Present:
1. **System prompt** — ready to paste into Claude Code sub-agent config
2. **Tools config JSON** — minimal tool set for this role
3. **CLAUDE.md entry** — add to project root CLAUDE.md

Print summary:
```
✅ Sub-agent scaffolded: [agent-name]
  Role: [purpose]
  Tools: [list]
  Constraints: [key restrictions]
  Add to: CLAUDE.md → Sub-agents section
```

---

## Output format spec

- System prompt: plain text block, under 500 tokens
- Tools config: JSON block
- CLAUDE.md entry: markdown block
- Summary line at end

---

## Quality checklist

- [ ] System prompt has explicit "never do" list
- [ ] Tools list is minimal (only what's needed)
- [ ] Output format is precisely defined in prompt
- [ ] CLAUDE.md entry covers trigger condition
- [ ] Agent name is kebab-case
- [ ] Role is single-responsibility (does one thing well)

---

## Edge cases

- **User wants agent that does everything**: Push back — explain that broad agents defeat the purpose. Ask them to pick one primary responsibility.
- **Conflicting constraints** (e.g. "write docs but never write files"): Clarify scope — docs are files; suggest restricting to a specific folder path.
- **Agent needs to call another agent**: Note that sub-agent orchestration is handled by the planner/orchestrator; this agent should only do its own task and signal completion.
- **User wants to reuse existing agent for new purpose**: Generate a new specialised agent instead of modifying the existing one — keep roles clean.
