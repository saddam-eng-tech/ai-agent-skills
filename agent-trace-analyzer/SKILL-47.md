---
name: agent-trace-analyzer
description: >
  Analyzes Claude Code agent session logs or conversation traces to explain
  why an agent made specific decisions, identify where it went wrong, and
  produce a root-cause report with corrective prompts or CLAUDE.md fixes.
  Use whenever a user says "why did Claude do that?", "debug my agent
  session", "Claude made a wrong decision and I don't know why", "analyze
  my Claude Code trace", "agent went off script", "understand agent
  behavior", or pastes a Claude Code session and asks what went wrong.
---

# Agent Trace Analyzer

Reads an agent session log or conversation and explains decision points,
identifies root causes of failures, and prescribes targeted fixes.

**Triggers on**: "why did Claude do that", "debug agent session", "agent
went off script", "analyze Claude trace", "wrong decision and I don't
know why", "understand agent behavior"
**Input**: Agent session log, conversation transcript, or problem description
**Output**: Decision-point analysis + root cause + corrective actions

---

## Step 1 — Ingest the Trace

Accept input in any of these formats:
- Pasted conversation text
- Claude Code session log file
- Description of what happened vs what was expected

If a file, read it:
```bash
cat [session-log-file] 2>/dev/null | head -500
```

---

## Step 2 — Extract Decision Points

Scan the trace for these key decision moments:

**Tool selections**: "Why did it call X instead of Y?"
**File choices**: "Why did it edit file A and not file B?"
**Approach decisions**: "Why did it choose implementation X?"
**Stopping points**: "Why did it stop or ask for help here?"
**Errors/retries**: "What triggered the retry loop?"

For each decision point, extract:
```
DECISION: [What the agent decided]
CONTEXT AT THAT POINT: [What was in context — visible instructions, prior turns]
AVAILABLE ALTERNATIVES: [What else it could have done]
```

---

## Step 3 — Classify Root Causes

For each problematic decision, classify by root cause type:

| Root Cause Type | Description | Fix Approach |
|-----------------|-------------|-------------|
| **Missing instruction** | Agent wasn't told what to do | Add to CLAUDE.md or prompt |
| **Ambiguous instruction** | Two valid interpretations existed | Clarify the instruction |
| **Context rot** | Earlier instruction forgotten | Reduce session length / inject reminders |
| **Conflicting instructions** | Two rules contradicted each other | Resolve conflict in CLAUDE.md |
| **Scope creep** | Agent did more than asked | Add explicit "do NOT" constraint |
| **Tool confusion** | Agent used wrong tool | Add tool selection guidance |
| **Hallucination** | Agent invented facts | Add verification instruction |
| **Model limit** | Task exceeded model capability | Break into smaller steps |

---

## Step 4 — Produce Root Cause Report

```markdown
## Agent Trace Analysis

**Session summary**: [1 sentence describing what the agent was trying to do]
**Problem observed**: [1 sentence describing what went wrong]

---

### Decision Point Analysis

#### Decision 1: [Short title]
- **What happened**: [Agent action]
- **Expected**: [What user wanted]
- **Root cause**: [Type from classification table]
- **Evidence**: [Quote from trace that shows why it happened]
- **Fix**: [Specific corrective action]

#### Decision 2: [Short title]
[same format]

---

### Summary of Fixes Required

| Fix | Where to Apply | Priority |
|-----|---------------|---------|
| [Fix 1] | CLAUDE.md | High |
| [Fix 2] | System prompt | Medium |
| [Fix 3] | Task description | Low |
```

---

## Step 5 — Generate Corrective Prompts/Rules

For each fix, produce the exact text to add:

### For CLAUDE.md additions:
```markdown
## [New Section or Existing Section to Update]

[Exact text to add that prevents the failure]

**Do NOT**: [Specific prohibition based on the trace failure]
```

### For better task prompts:
```markdown
Improved prompt for this task type:

---
[Rewritten prompt that prevents the observed failure mode]
---

Key changes:
- Added: [what was added and why]
- Clarified: [what was ambiguous]
- Constrained: [what new boundary was set]
```

---

## Step 6 — Prevention Checklist

Produce a checklist the user can apply before future similar sessions:

```markdown
## Before Running This Type of Task Next Time

- [ ] [Verification step 1 based on failure]
- [ ] [Verification step 2]
- [ ] CLAUDE.md updated with new constraints
- [ ] Session scoped to [recommended smaller scope]
```

---

## Quality Checklist

- [ ] Every problematic decision has a root cause classification
- [ ] Root causes are specific (not "the model made an error")
- [ ] Fixes are actionable (exact text provided, not vague suggestions)
- [ ] Both CLAUDE.md fixes AND prompt fixes addressed
- [ ] Prevention checklist tailored to the specific failure type

---

## Edge Cases

- **User describes problem from memory (no trace)**: Work from description; note low confidence in root cause
- **Multiple compounding failures**: Prioritize — fix the earliest decision point first
- **Failure was actually correct behavior**: Explain this diplomatically and clarify user's real intent
- **Trace is very long (100+ turns)**: Summarize mid-section, focus analysis on the failure window
