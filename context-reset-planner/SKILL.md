---
name: context-reset-planner
description: Detects context rot in long LLM/Claude Code sessions and produces a structured context reset: a compact summary of what's been decided, what's in progress, and what comes next — ready to paste into a fresh session. Use whenever a user says "my Claude session is going off the rails", "Claude keeps forgetting earlier decisions", "context window almost full", or "should I start a new chat?".
---

# Context Reset Planner

Diagnoses context rot symptoms and generates a clean handoff document that preserves all decisions and state so a fresh session picks up exactly where the good work ended.

**Input**: Current task description + what's been done/decided
**Output**: Compact context handoff doc + fresh session starter prompt

## Trigger phrases
- "session going off rails"
- "context window full"
- "Claude forgetting decisions"
- "start fresh session"
- "LLM loop of doom"
- "should I reset chat"

## Step-by-step workflow

### STEP 1 — Diagnose context rot

Check for these symptoms in the current session:

| Symptom | Severity |
|---------|----------|
| Claude contradicts an earlier decision | HIGH |
| Claude asks for information already provided | HIGH |
| Responses are getting shorter/worse | MEDIUM |
| Claude is repeating the same approach that already failed | HIGH |
| Context window > 80% full | MEDIUM |
| Session has been going > 2 hours / 50+ messages | LOW |

If 2+ HIGH symptoms: **strongly recommend reset now**.
If 1 HIGH or 2+ MEDIUM: **recommend reset soon**.
If LOW only: **optional, session may continue**.

Tell the user what you found:
```
⚠️ Context rot detected: [symptoms list]
Recommendation: [Reset now / Soon / Optional]
```

---

### STEP 2 — Extract session state

Ask the user (or infer from conversation):

1. **Goal**: What is the overall task we're trying to complete?
2. **Decisions made**: What choices have been locked in? (architecture, naming, approach)
3. **Work completed**: What has actually been built/written/done?
4. **Work in progress**: What was being worked on when the session degraded?
5. **Next steps**: What is the immediate next thing to do?
6. **Constraints**: Any hard rules Claude must follow? (don't touch X, always use Y)
7. **Known failures**: What approaches have already been tried and failed?

---

### STEP 3 — Write the handoff document

Produce a compact `CONTEXT.md` handoff:

```markdown
# Session Handoff — [Task Name]

> Created: [date/time]. Paste this at the start of a fresh Claude session.

## Goal
[1–2 sentences: what we are building and why]

## Decisions Locked In
- [Decision 1 — brief, specific]
- [Decision 2]
- [Decision 3]

## What's Done
- [x] [Completed task 1]
- [x] [Completed task 2]
- [x] [Completed task 3]

## Currently In Progress
- [ ] [Task being worked on — describe exact state]

## Immediate Next Step
[One clear action for the next session to start with]

## Hard Constraints
- [Rule 1 — e.g. "Do not modify the auth middleware"]
- [Rule 2 — e.g. "Use sqlc for all DB queries, not raw SQL"]

## What NOT to Try Again
- [Failed approach 1 — brief reason why it didn't work]
- [Failed approach 2]

## Relevant Files
- `[path/to/file]` — [what it is]
- `[path/to/file]` — [what it is]

## Context
[Any other information needed to pick this up without re-reading the old session]
```

Keep total length under 500 tokens. Prioritise decisions and next step.

---

### STEP 4 — Write the fresh session starter prompt

Produce a prompt to paste at the start of the new session:

```
I am continuing work on [task name]. Here is the current state:

[paste CONTEXT.md content here]

Please confirm you have read and understood the context above before we continue.
Start by: [immediate next step from CONTEXT.md]
```

---

### STEP 5 — Output

Present both:
1. `CONTEXT.md` — the handoff document
2. Fresh session starter prompt — ready to copy/paste

Print summary:
```
✅ Context reset prepared for: [task name]
  Decisions captured: N
  Completed tasks: N
  In-progress items: N
  Failed approaches noted: N

Next: Open a new chat, paste the starter prompt, then paste CONTEXT.md.
```

---

## Output format spec

- `CONTEXT.md`: Compact, under 500 tokens, all sections present
- Starter prompt: Single block, ready to paste
- No code in the handoff doc — references to files only

---

## Quality checklist

- [ ] Handoff doc fits in one screen (compact)
- [ ] All locked decisions are captured
- [ ] Failed approaches documented to prevent re-trying
- [ ] Immediate next step is single, unambiguous action
- [ ] Hard constraints section present
- [ ] No sensitive data (secrets, tokens) in handoff doc

---

## Edge cases

- **User doesn't know what decisions were made**: Scan the conversation and extract them; present as draft for confirmation.
- **Very short session** (< 10 messages): No reset needed; suggest continuing and come back when symptoms appear.
- **User is mid-implementation of a complex function**: Capture the function signature and partial implementation state exactly.
- **Multiple parallel tasks in one session**: Create one CONTEXT.md per task thread, or recommend they work on one task per session going forward.
- **User wants to reset but not lose the old session**: Remind them the old chat window stays open — they can still reference it.
