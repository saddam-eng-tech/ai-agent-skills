name: useeffect-auditor
description: Analyses React component files and audits every useEffect hook for dependency array problems, stale closures, missing deps, infinite loop risks, and cleanup gaps. Use this skill whenever a user pastes a React component and asks about useEffect issues, effect bugs, infinite re-renders, stale values in effects, or "why does my effect keep running". Also trigger on phrases like "audit my hooks", "check my effects", or "fix my useEffect".

# useEffect Auditor

Diagnoses every `useEffect` in a React component and produces a structured
report with per-effect verdicts, plain-English explanations, and corrected code.

**Triggers on**: "audit my effects", "useEffect keeps running", "stale closure",
"infinite loop in effect", "check my hooks", "dependency array", "effect cleanup"
**Input**: React component file (.tsx or .jsx) pasted or uploaded
**Output**: Annotated report per effect + corrected code snippets


## Step-by-step workflow

### STEP 1 — Receive and parse the component

Use the `view` tool if a file path is given. Otherwise read from the
conversation.

Identify all `useEffect` calls by scanning for the pattern:
```
useEffect(() => { ... }, [...])
useEffect(() => { ... })   // no dep array
```

Count them and announce: `Found N useEffect hooks. Auditing each one.`


### STEP 2 — Audit each effect

For each `useEffect`, evaluate all six criteria below. Record findings per
effect.

**Criteria checklist:**

1. **Missing dependency array** — Effect runs on every render (almost always wrong).
   Mark: `⚠️ MISSING DEP ARRAY`

2. **Incomplete dependencies** — State or props referenced inside the effect but
   absent from the dep array. This causes stale closures.
   Mark: `🔴 STALE CLOSURE RISK — missing: [list the identifiers]`

3. **Unnecessary dependencies** — Objects or functions created inline in the
   component body included as deps. They change reference every render →
   infinite loop.
   Mark: `🔴 INFINITE LOOP RISK — [identifier] is new each render`

4. **Missing cleanup** — Effect sets up subscriptions, timers, event listeners,
   or async operations without returning a cleanup function.
   Mark: `⚠️ MISSING CLEANUP — risk of memory leak or stale update`

5. **Should be an event handler** — Effect is only triggered by a specific user
   action; it should be an event handler, not an effect.
   Mark: `💡 REFACTOR CANDIDATE — move to event handler`

6. **Correct** — None of the above issues found.
   Mark: `✅ OK`


### STEP 3 — Generate fix suggestions

For each flagged effect, write:
- A 2-3 sentence explanation of _why_ this is a problem
- A corrected code snippet showing the fix

Use these standard fix patterns:

| Problem | Fix |
|---------|-----|
| Stale function dep | Wrap in `useCallback` OR use `useEffectEvent` (React 19+) |
| Inline object dep | Move object outside component or into `useMemo` |
| Missing cleanup | Add `return () => { cleanup() }` inside effect |
| Should be event handler | Remove `useEffect`, call logic directly in handler |
| Entire dep array empty but deps exist | Add missing identifiers to array |


### STEP 4 — Write the report

Produce a markdown report in this structure:

```markdown
# useEffect Audit — [ComponentName]

## Summary
- Total effects: N
- ✅ OK: N
- 🔴 Critical issues: N
- ⚠️ Warnings: N
- 💡 Refactor suggestions: N


## Effect 1 (line ~[N])

**Code:**
\```tsx
useEffect(() => { ... }, [deps])
\```

**Verdict**: 🔴 STALE CLOSURE RISK — `fetchData` missing from dependencies

**Explanation**: [2-3 sentences]

**Fix:**
\```tsx
// corrected code here
\```

[repeat for each effect]

## Quick Reference: dep array rules
- Functions used in effects: wrap in useCallback or list as dep
- Objects/arrays: move outside component or useMemo
- Always clean up: timers, subscriptions, listeners
```


### STEP 5 — Deliver report

If the component was provided as a file path, offer to write the corrected
component to a new file named `[ComponentName].fixed.tsx`.

Otherwise, present the full report inline.


## Output format spec

- File name: `[ComponentName]-useeffect-audit.md`
- Always includes: Summary table, per-effect analysis, corrected snippets
- Minimum one code block per flagged issue


## Quality checklist

- [ ] Every useEffect in the file has been audited (none skipped)
- [ ] Each flagged issue has a plain-English explanation
- [ ] Each flagged issue has a corrected code snippet
- [ ] Summary count matches total effects found
- [ ] Report uses consistent severity icons (✅ 🔴 ⚠️ 💡)
- [ ] No TypeScript `any` types introduced in fix suggestions


## Edge cases

- **No useEffect found**: Report "No useEffect hooks found. Component appears
  side-effect free."
- **React 19 project (detected by React version import or use() hook)**:
  Suggest `useEffectEvent` as an alternative to useCallback for stable
  function refs inside effects.
- **Custom hooks that call useEffect**: Audit them the same way; note they are
  inside a custom hook rather than a component.
- **Async effects**: Flag if `async` is used directly in the effect callback
  (anti-pattern) and show the correct inner-async-function pattern.
- **Conditional effects inside the dep array** (e.g. `[condition && fn]`):
  Flag as potential bug — undefined may enter the array.