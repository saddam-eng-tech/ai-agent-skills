---
name: spec-md-generator
description: Generates a complete spec.md (project specification document) by having a structured conversation with the user about their idea, then producing a thorough written spec covering requirements, architecture, data models, and implementation plan. Use whenever a user says "help me spec out a feature", "write a spec for my project", "I want to plan before coding", or "create a spec.md".
---

# spec.md Generator

Turns a rough idea into a precise, AI-ready specification document that anchors the whole coding session. No more vague prompts, no more midway pivots.

**Input**: User's feature idea or problem description
**Output**: Complete spec.md ready to feed into Claude Code

## Trigger phrases
- "help me spec out"
- "create a spec.md"
- "plan before coding"
- "write design doc"
- "clarify requirements"
- "architect first"

## Step-by-step workflow

### STEP 1 — Gather the idea

Ask the user a series of structured questions to understand their project. Ask in batches of 3, not all at once:

**Batch 1 — What are we building?**
1. What is this project/feature in one sentence?
2. Who is the end user? What problem does it solve for them?
3. What is the single most important thing it must do?

Wait for answers before continuing.

**Batch 2 — Scope and constraints:**
4. What is explicitly OUT of scope for the first version?
5. What tech stack / language / framework are you using?
6. Are there any hard constraints? (performance targets, budget, deadline, existing systems to integrate with)

**Batch 3 — Data and flow:**
7. What are the main entities/objects in this system? (e.g. User, Order, Product)
8. What are the key actions a user takes? (e.g. create, search, approve)
9. How does data flow through the system? (input → processing → output)

If the user has already provided most of this, skip the questions and proceed.

---

### STEP 2 — Confirm understanding

Before writing the spec, present a one-paragraph summary:

> "Here's what I understand we're building: [summary]. The key entities are [list]. The core flow is [flow]. Out of scope: [list]. Does this match your intention?"

Wait for confirmation or corrections.

---

### STEP 3 — Write the spec.md

Produce a complete `spec.md` with all sections:

```markdown
# [Project Name] — Specification

## Overview
[2–3 sentences: what this is and why it's being built]

## Goals
- [Goal 1 — concrete and measurable]
- [Goal 2]

## Non-Goals (explicitly out of scope)
- [Non-goal 1]
- [Non-goal 2]

## User Stories
### [User type]
- As a [user], I want to [action] so that [outcome]
- As a [user], I want to [action] so that [outcome]

## Data Models
### [EntityName]
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| id | string (UUID) | yes | |
| [field] | [type] | [yes/no] | [notes] |

### Relationships
- [EntityA] has many [EntityB]
- [EntityB] belongs to [EntityA]

## API / Interface Design
### [Endpoint or method name]
- **Input**: [description]
- **Output**: [description]
- **Errors**: [list error cases]

## Architecture
[Description of layers and how they connect]

## Tech Stack
| Layer | Choice | Reason |
|-------|--------|--------|
| Language | [choice] | [why] |
| Database | [choice] | [why] |

## Implementation Plan
### Phase 1 — [name] (MVP)
- [ ] [Task 1]
- [ ] [Task 2]

### Phase 2 — [name]
- [ ] [Task 1]

## Open Questions
- [Question 1 that needs a decision before implementation]
- [Question 2]

## Success Criteria
- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]
```

---

### STEP 4 — Output

Write to `spec.md` in the project root (or current directory). Present the full file.

Print summary:
```
✅ spec.md generated: [project name]
  User stories: N
  Entities: N
  API methods: N
  Implementation phases: N
  Open questions: N (resolve before coding)
```

---

## Output format spec

- Single `spec.md` file
- All sections present (even if short)
- Implementation plan uses checkboxes for tasks
- Open Questions section always included
- No code in the spec — design only

---

## Quality checklist

- [ ] Overview is 2–3 sentences, jargon-free
- [ ] Non-Goals section explicitly listed
- [ ] Every entity has a field table
- [ ] Every API method has input + output + errors
- [ ] Implementation plan has at least 2 phases
- [ ] Open Questions captured (nothing swept under the rug)
- [ ] Success Criteria are measurable

---

## Edge cases

- **User says "just start coding"**: Explain that 10 minutes on spec.md saves hours of mid-session pivots. Ask the 3 most important questions only.
- **Very simple feature** (1 entity, 1 action): Use a shorter spec format — skip Phase 2, keep only essential sections.
- **User changes requirements mid-spec**: Update the spec in place; don't produce a second document.
- **Conflicting requirements**: Surface the conflict explicitly in Open Questions before writing the spec.
- **User doesn't know the tech stack**: Leave Tech Stack section blank with a note; suggest options based on the use case.
