name: reverse-spec-writer
description: Reads an existing codebase and reverse-engineers a spec.md — documenting what the system actually does, its architecture, data models, and key behaviours. Use whenever a user says "document what this code does", "generate a spec from my existing code", "write a design doc for this codebase", or "I inherited this codebase and need to understand it".

# Reverse Spec Writer

Reads a codebase and produces the spec.md that should have been written before it was built — documenting intent, architecture, data models, and API contracts as they actually exist.

**Input**: Project codebase
**Output**: spec.md that describes what was built

## Trigger phrases
- "document this codebase"
- "reverse engineer spec"
- "write architecture doc"
- "I inherited this code"
- "understand existing system"
- "generate spec from code"

## Source
`SKILL-43.md`