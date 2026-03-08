---

---

name: react-pr-reviewer
description: Performs a structured code review of React TypeScript pull requests or
  changed files, categorised by Security, Type Safety, Performance,
  Architecture, Accessibility, and Testing — with BLOCK/WARN/APPROVE verdicts.
  Trigger when a user pastes a PR diff, asks "review this code", "can you
  check this PR", "give feedback on my React code", "review this component",
  or asks about code quality of React/TypeScript files. Also trigger when
  a user says "what should I look for in this PR" or "is this code ok to merge
---

# Your Skill Title

---

name: react-pr-reviewer
description: >
  Performs a structured code review of React TypeScript pull requests or
  changed files, categorised by Security, Type Safety, Performance,
  Architecture, Accessibility, and Testing — with BLOCK/WARN/APPROVE verdicts.
  Trigger when a user pastes a PR diff, asks "review this code", "can you
  check this PR", "give feedback on my React code", "review this component",
  or asks about code quality of React/TypeScript files. Also trigger when
  a user says "what should I look for in this PR" or "is this code ok to merge".
---

# React PR Reviewer

Produces a fast, prioritised code review of React/TypeScript changes —
catching security holes, type safety issues, and performance bugs before
they ship.

**Triggers on**: "review this PR", "check this code", "give feedback on",
"is this ok to merge", "code review", "review my component", pasted diff or
component files asking for feedback
**Input**: PR diff text OR one or more React/TypeScript component files
**Output**: Structured review with BLOCK/WARN/APPROVE per category

---

## Step-by-step workflow

### STEP 1 — Receive the code

Accept input in any of these forms:
- Git diff format (lines prefixed with `+` / `-`)
- Raw file contents of changed components
- Multiple files pasted sequentially

If the input is a diff, focus only on added (`+`) lines for new issues.
Note removed (`-`) lines if they reveal context (e.g. removing a cleanup).

---

### STEP 2 — Run the review checklist by category

Evaluate the code against all items below. Assign a verdict to each category.

**Verdict scale:**
- 🔴 `BLOCK` — Must be fixed before merge. Bug, security issue, or breaking change.
- 🟡 `WARN` — Should be fixed. Technical debt or likely future bug.
- 🟢 `APPROVE` — No issues found in this category.

---

#### Category 1: Security

| Check | Look for |
|-------|---------|
| API key / secret in frontend code | `const API_KEY = 'sk-...'` or hardcoded tokens |
| Sensitive data logged | `console.log(password)`, `console.log(user.creditCard)` |
| Raw HTML injection | `dangerouslySetInnerHTML` without sanitisation |
| User input in URLs | `fetch(\`/api?q=${query}\`)` without encoding |

→ Any found: `🔴 BLOCK`

---

#### Category 2: Type Safety

| Check | Look for |
|-------|---------|
| `any` types | `data: any`, `: any =>`, `as any` |
| Non-null assertion `!` without comment | `user!.email` |
| Missing return type on exported functions | `export function getData()` without `: Promise<Data>` |
| Unhandled union cases | switch/if on a union without exhaustive check |
| `@ts-ignore` without comment explaining why | bare `// @ts-ignore` |

→ `any` or `@ts-ignore`: `🔴 BLOCK`; others: `🟡 WARN`

---

#### Category 3: Performance

| Check | Look for |
|-------|---------|
| Inline object/array as prop to memoised child | `<Child style={{ color: 'red' }} />` every render |
| Missing keys in lists | `items.map(item => <Item />)` without `key` |
| Expensive computation without useMemo | Heavy filter/sort in render body |
| State that causes unnecessary re-renders | State placed too high in tree |
| Missing `useCallback` on handlers passed to memoised children | `<MemoChild onClick={() => ...} />` |

→ Missing keys: `🔴 BLOCK`; others: `🟡 WARN`

---

#### Category 4: Architecture

| Check | Look for |
|-------|---------|
| Component > ~150 lines | Count lines in component body |
| Logic mixed with JSX | Data transformation inside return statement |
| Direct DOM manipulation | `document.getElementById` instead of ref |
| Side effects outside useEffect | Async calls directly in render |
| Prop drilling > 2 levels | Prop passed through 3+ components unused |

→ All: `🟡 WARN`

---

#### Category 5: Accessibility

| Check | Look for |
|-------|---------|
| Interactive element without accessible name | `<div onClick={...}>` without `role` or `aria-label` |
| Image without alt | `<img src={...}>` without `alt` attribute |
| Form input without label | `<input>` not associated with `<label>` |
| Focus management missing | Modal/dialog opened without focus moving to it |
| Colour-only information | Any design that relies solely on colour |

→ All: `🟡 WARN` (BLOCK if violates WCAG 2.1 AA for regulated industries)

---

#### Category 6: Testing

| Check | Look for |
|-------|---------|
| New component with no test file | Added `.tsx` with no accompanying `.test.tsx` |
| Existing tests not updated | New props/behaviour not covered |

→ `🟡 WARN`

---

### STEP 3 — Write the review

```markdown
# PR Review: [filename or "PR diff"]

## Overall verdict: [🔴 BLOCK / 🟡 WARN / 🟢 APPROVE]

---

### 🔴 Security — BLOCK
[Specific issue with line reference and suggested fix]

### 🔴 Type Safety — BLOCK
[Issue]

### 🟡 Performance — WARN
[Issue]

### 🟢 Architecture — APPROVE

### 🟡 Accessibility — WARN
[Issue]

### 🟡 Testing — WARN
[Issue]

---

## Summary
- BLOCK issues: N (must fix before merge)
- WARN issues: N (should address soon)
- APPROVE categories: N

## Top priority fix
[One specific, actionable instruction for the most critical issue]
```

---

### STEP 4 — Offer to fix

After the review, offer:
> "Want me to fix the BLOCK issues directly? Share the full component
> and I'll produce a corrected version."

---

## Output format spec

- Markdown with category headers using severity emoji
- Overall verdict at top
- Line references where possible
- Summary count at bottom
- Concise (aim for <600 words for a typical component review)

---

## Quality checklist

- [ ] All 6 categories evaluated (even if APPROVE)
- [ ] Line references included for all flagged issues
- [ ] BLOCK issues listed before WARN in output
- [ ] No personal style preferences flagged (only objective issues)
- [ ] Summary count matches issues in body
- [ ] Offer to fix BLOCK issues made at end

---

## Edge cases

- **Only a snippet provided (< 20 lines)**: Complete the review on what's
  available; note "Review based on partial code — full context may reveal
  additional issues."
- **Perfect code**: Give full APPROVE across all categories with a brief
  positive note. Do not invent issues.
- **Code from a test file**: Skip Architecture and some Performance checks;
  focus on test quality (coverage, query types, assertion completeness).
- **Legacy codebase patterns**: Note when something is a known legacy
  pattern (class components, PropTypes) without blocking — flag as WARN
  with migration suggestion.
- **User shares only a diff with no context**: Review only what's visible;
  note any assumptions made about the surrounding code.