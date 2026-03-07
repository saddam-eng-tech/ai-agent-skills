---
name: issue-skill-generator
description: >
  Takes a list of developer issues (typically produced by the `issue-finder`
  skill) and generates a complete, production-ready SKILL.md file for each
  issue. Use this skill when the user says "generate skills from this list",
  "turn these issues into skills", "create SKILL.md for each problem",
  "build skills from the issue list", or "automate skill creation from
  issues". Also trigger when the user pastes or uploads an issue list and
  asks for skills, or wants to batch-produce skills from a backlog of
  developer pain points.
---

# Issue → Skill Generator

Reads a structured issue list and produces one complete SKILL.md per
issue, ready to install and use.

---

## Inputs accepted

- A markdown file produced by the `issue-finder` skill
- A pasted list of issues in any reasonable format
- A single issue description (will produce one skill)

---

## Skill Writing Standard

Every generated skill MUST contain:

### 1. YAML Frontmatter
```yaml
---
name: [kebab-case-name]
description: >
  [What the skill does + when to trigger — be "pushy".]
---
```

**Naming rules:** kebab-case, lowercase, verb-noun preferred (e.g. `pdf-splitter`), max 30 chars.

### 2. Overview section
```markdown
# [Skill Name]
[1–2 sentences: what problem this solves]
**Triggers on**: ...
**Input**: ...
**Output**: ...
```

### 3. Step-by-step workflow
- Number every step
- Specify which tools to call
- Include decision branches
- Min 4 steps, max 12 steps

### 4. Output format spec
- File type and name pattern
- Sections the output must contain
- Example snippet

### 5. Quality checklist
4–8 items to verify before finishing.

### 6. Edge cases
At least 3 tricky situations and how to handle them.

---

## Batch generation workflow

### PHASE 1 — Plan
Print a plan table, ask confirmation if > 5 skills.

### PHASE 2 — Generate one at a time
Write each to `/home/claude/generated-skills/[skill-name]/SKILL.md`.
Print: `✅ Generated: [skill-name]`

### PHASE 3 — Verify
Run checklist for each skill. Fix failures before proceeding.

### PHASE 4 — Summarise
Copy all to `/mnt/user-data/outputs/generated-skills/` and present files.

---

## Quality rules

- No vague verbs — use concrete actions
- Every step invokes a tool
- No infinite loops — clear terminal state
- Tight scope — one thing per skill
- Under 500 lines — move excess to `references/`

---

## Edge cases

- **Duplicate issues**: merge if core action is identical
- **Ambiguous input**: ask one clarifying question
- **Too broad**: decompose into 2–3 sub-issues
- **Already covered**: propose enhancement instead
- **Needs external API**: add `## Prerequisites` section