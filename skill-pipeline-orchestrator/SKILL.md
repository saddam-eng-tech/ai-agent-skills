---
name: skill-pipeline-orchestrator
description: >
  Runs the complete end-to-end pipeline: discover real developer issues online,
  filter them for skill-solvability, and auto-generate a ready-to-install
  SKILL.md for each one — all in a single command. Use this skill when the user
  says "run the full skill pipeline", "find issues and generate skills", "do the
  whole thing end to end", "build me a skill library from scratch", "automate
  skill discovery and creation", or "run issue-finder then skill-generator".
  Also trigger when the user wants a batch of new skills generated with no
  manual steps in between.
---

# Skill Pipeline Orchestrator

Chains `issue-finder` → `issue-skill-generator` into one automated run.
Zero manual steps between discovery and skill creation.

```
[ Web Search ]
     ↓
[ Issue Finder ]          ← searches forums, GitHub, HN, SO
     ↓
[ issue-list.md ]         ← structured, scored, ranked
     ↓
[ Issue Skill Generator ] ← one SKILL.md per issue
     ↓
[ /outputs/generated-skills/ ]
```

---

## When to use this vs individual skills

| Use case | Which skill |
|----------|-------------|
| Just explore what issues exist | `issue-finder` alone |
| Already have issue list | `issue-skill-generator` alone |
| Full automated pipeline | **this skill** ✅ |
| Want to review before generating | Run `issue-finder`, review, then `issue-skill-generator` |

---

## Pre-flight checklist

Confirm with user before starting:
1. **Scope** — specific domain? (default: general web dev)
2. **Volume** — how many skills? (default: top 10)
3. **Complexity** — include High-complexity? (default: Low + Medium only)

If user says "go" — use all defaults.

---

## Full pipeline steps

### STAGE 1 — Issue Discovery
Follow `issue-finder` workflow. Save to `/home/claude/pipeline/issue-list.md`.
Print: `🔍 Issue discovery complete — N issues found`

### STAGE 2 — Review gate
Print summary table. Ask user to confirm or pick issues.
Skip if user said "do the whole thing".

### STAGE 3 — Skill Generation
Follow `issue-skill-generator` workflow.
Print: `✅ [N/total] skill-name` per skill.

### STAGE 4 — Package output
```
/mnt/user-data/outputs/generated-skills/pipeline-run-YYYY-MM-DD/
├── issue-list.md
├── pipeline-summary.md
└── [skill-name]/SKILL.md
```

### STAGE 5 — Summary report
Write `pipeline-summary.md` with: issues searched, discovered, qualifying, skills generated, top 3 picks.

### STAGE 6 — Present
Use `present_files` to share pipeline-summary.md and all skill files.

---

## Error handling

| Error | Action |
|-------|--------|
| Web search returns nothing | Try alternative queries |
| < 5 qualifying issues | Lower score threshold to 2+ |
| Skill generation fails | Note as FAILED, continue rest |
| Output write fails | Write to `/home/claude/pipeline/` |

---

## Quality checklist

- [ ] At least 4 web sources used in Stage 1
- [ ] Final issue list has ≥ 10 entries
- [ ] Every skill has valid YAML frontmatter
- [ ] Every skill has numbered workflow
- [ ] pipeline-summary.md complete with stats
- [ ] All files in `/mnt/user-data/outputs/`
- [ ] Files presented via `present_files`