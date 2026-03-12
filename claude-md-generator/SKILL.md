name: claude-md-generator
description: Generates a production-ready CLAUDE.md file for any project by analyzing the codebase structure, tech stack, and conventions. Use this skill whenever a user says "create a CLAUDE.md", "set up Claude Code for my project", "write a CLAUDE.md from scratch", "help me configure Claude Code", or "my Claude Code doesn't know about my project conventions". Also trigger when a user sets up a new repo and asks how to get Claude Code working well.

# CLAUDE.md Generator

Analyzes a project and produces a complete, structured CLAUDE.md that gives Claude Code all the context it needs: stack, conventions, architecture, commands, and what to avoid.

**Input**: Project directory (Claude reads the files)
**Output**: A complete CLAUDE.md written to the project root

## Trigger phrases
- "create CLAUDE.md"
- "set up Claude Code"
- "write project context file"
- "configure Claude for my project"
- "Claude keeps forgetting my conventions"

## Source
`SKILL-36.md`