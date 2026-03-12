name: claude-md-auditor
description: Audits an existing CLAUDE.md file against the actual codebase to find stale commands, outdated conventions, missing sections, and wrong architecture descriptions. Use whenever a user says "update my CLAUDE.md", "my CLAUDE.md is outdated", "Claude keeps making mistakes despite my CLAUDE.md", "audit my project context file", or "my conventions changed and Claude doesn't know".

# CLAUDE.md Auditor

Compares an existing CLAUDE.md against the live codebase and produces a prioritized diff: what's stale, what's missing, what's wrong — with corrected replacements ready to apply.

**Input**: Existing CLAUDE.md + project codebase
**Output**: Audit report + corrected CLAUDE.md sections

## Trigger phrases
- "update CLAUDE.md"
- "CLAUDE.md is outdated"
- "Claude ignores my rules"
- "fix my project context file"
- "conventions changed"

## Source
`SKILL-37.md`