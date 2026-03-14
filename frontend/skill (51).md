---
name: frontend-agent-team
description: >
  Runs a full autonomous frontend implementation pipeline — orchestrator plans,
  coder builds, logic verifier reviews code, visual verifier checks UI in browser,
  then orchestrator summarizes with a PR-ready report. Trigger whenever a user says
  "implement this ticket", "build this frontend feature", "here's a Figma + story",
  "use the agent team", "run the frontend agents", "implement this with the full
  pipeline", or provides a task + Figma URL/image and wants end-to-end delivery.
  Also trigger when the user says "do the whole thing" or "handle this ticket
  autonomously". This is the entry point for the entire multi-agent workflow.
---

# Frontend Agent Team

Entry point for the full autonomous frontend implementation pipeline.
Coordinates four sub-agents: orchestrator, coder, logic verifier, visual verifier.

**Triggers on**: "implement this ticket", "run the full pipeline", "use the agent
team", task + Figma URL combo, "do this autonomously", "handle this end to end"
**Input**: Feature ticket / story + Figma URL or design image + local repo path
**Output**: Implemented code, two verification reports, PR description

---

## Pipeline overview

```
[ticket + figma/image + repo] 
        ↓
  ORCHESTRATOR  — reads standards, builds plan
        ↓  delegate
    CODER       — implements React/TS/Redux
        ↓  report
  ORCHESTRATOR  — routes to both verifiers
      ↙           ↘
LOGIC VERIFIER   VISUAL VERIFIER
      ↘           ↙
  ORCHESTRATOR  — all pass? summarize : fix loop
        ↓
  PR SUMMARY
```

Max 2 fix cycles. On third failure → surface issues to human.

---

## Step-by-step workflow

### STEP 1 — Gather inputs

Require before starting:
- **Ticket**: user story, acceptance criteria, or task description
- **Design**: Figma URL (uses Figma MCP) OR attached image
- **Repo path**: local path or assume CWD if not given
- **Standards doc**: look for `STANDARDS.md`, `CONTRIBUTING.md`,
  `.github/CONTRIBUTING.md`, or `docs/conventions.md` in repo root

If standards doc is missing, ask the user to point to it or confirm defaults.
Do not proceed without at least one of: standards doc, or user confirmation to
use inferred conventions from reading the codebase.

---

### STEP 2 — Load the appropriate sub-agent skill

Read and follow:

1. **`frontend-agent-orchestrator/SKILL.md`** → executes the planning phase
2. **`frontend-coder-agent/SKILL.md`** → executes the implementation phase
3. **`frontend-logic-verifier/SKILL.md`** → executes code review
4. **`frontend-visual-verifier/SKILL.md`** → executes visual review
5. Return here to run the fix loop and final summary

---

### STEP 3 — Fix loop

After both verifiers report:
- If both PASS → proceed to STEP 4
- If either FAIL:
  - Increment `fix_cycle` counter (start at 0)
  - If `fix_cycle < 2`: feed failure findings back to orchestrator, re-delegate
    to coder with explicit `fix_instructions` payload, re-run both verifiers
  - If `fix_cycle >= 2`: stop. Surface remaining failures to the human with
    a clear summary of what needs manual resolution. Do not loop again.

---

### STEP 4 — Final summary

Produce a PR-ready report:
```
## What was built
[2–3 sentence description of the feature]

## Files changed
[list of new/modified files with one-line purpose]

## Design fidelity
[Pass / Partial / manual-review-needed — from visual verifier]

## Code quality
[Pass / issues-resolved / outstanding — from logic verifier]

## Acceptance criteria
[check each criterion from the ticket: ✅ met / ⚠️ partial / ❌ not met]

## Known deviations
[anything that differs from the plan, with reason]

## PR description (copy-paste ready)
[conventional commit style title + body]
```

---

## Reference files

- `references/payload-schema.md` — JSON structures passed between agents
- `references/standards-defaults.md` — fallback conventions if no standards doc