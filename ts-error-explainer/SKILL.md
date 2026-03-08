name: ts-error-explainer
description: >
  Decodes cryptic TypeScript compiler errors into plain English and provides
  a minimal, correct fix. Trigger when a user pastes a TypeScript error
  message, asks "what does this TS error mean", "why is TypeScript complaining",
  "can you fix this type error", or "I don't understand this TypeScript error".
  Also trigger when error messages contain TS error codes like TS2345, TS2339,
  TS2304, TS7006, or when users paste red underline text from their IDE.

# TypeScript Error Explainer

Translates TypeScript's notoriously verbose compiler errors into clear
explanations and shows exactly how to fix them — without introducing `any`.

**Triggers on**: "what does this TS error mean", "TypeScript error", "TS2345",
"why is TypeScript complaining", "fix this type error", pasted TS error output
**Input**: TypeScript error message + surrounding code context (pasted text)
**Output**: Plain-English explanation + corrected code snippet


## Step-by-step workflow

### STEP 1 — Extract the error information

From what the user has shared, identify:
- **Error code**: e.g. `TS2345`, `TS2339`, `TS7006`
- **Error message text**: the compiler's description
- **Location**: file, line, column if given
- **Code context**: the surrounding code where the error appears

If error code is present, look it up in the reference table in STEP 2.
If no code is present, derive the category from the message text.


### STEP 2 — Categorise the error

Use this quick reference table:

| Code | Category | Common cause |
|------|----------|-------------|
| TS2345 | Argument type mismatch | Passing wrong type to function/prop |
| TS2339 | Property does not exist | Typo in prop name or missing type definition |
| TS2304 | Cannot find name | Import missing or variable not declared |
| TS2322 | Type not assignable | Assigning incompatible type to variable |
| TS7006 | Parameter implicitly any | Function param has no type annotation |
| TS2531 | Object possibly null | Not handling null/undefined before access |
| TS2769 | No overload matches | Wrong props passed to a component or function |
| TS2532 | Object possibly undefined | Optional value accessed without guard |
| TS1005 | Syntax error | Malformed TypeScript syntax |
| TS2693 | Type used as value | Importing a type with `import` instead of `import type` |


### STEP 3 — Write the plain-English explanation

Structure the explanation as:

```
**What TypeScript is saying:**
[Restate the error in 1-2 plain sentences. No jargon.]

**Why this happens:**
[Explain the underlying type mismatch or missing information. Use an analogy
if helpful. 2-4 sentences max.]

**The fix:**
[Describe what change will resolve it, before showing code.]
```

Do NOT:
- Paste the error back at the user
- Use TypeScript jargon without explaining it
- Suggest `// @ts-ignore` or `as any` as a solution


### STEP 4 — Produce the fix

Show the corrected code side-by-side with the original:

```tsx
// ❌ Before (causes TS2345)
function greet(name: string) { ... }
greet(42);

// ✅ After
greet(String(42));
// or: greet('42');
```

Rules for fixes:
- Prefer the narrowest, most specific type
- Prefer type narrowing (guards) over assertions
- Prefer `as SpecificType` over `as any` when assertion is unavoidable
- Show multiple fix options when there are meaningful trade-offs


### STEP 5 — Explain why the fix works

1-2 sentences explaining why the fix resolves the type conflict. This helps
the user understand and avoid the same error next time.


### STEP 6 — Offer follow-up

End with:
> "If you see more errors after this fix, paste them and I'll help
> with those too."


## Output format spec

Response sections (always in this order):
1. **What TypeScript is saying** — plain English restatement
2. **Why this happens** — explanation
3. **The fix** — corrected code block
4. **Why it works** — brief rationale

Maximum response length: ~400 words unless the error is genuinely complex.


## Quality checklist

- [ ] Error code looked up in reference table
- [ ] Explanation written without TypeScript jargon (or jargon defined)
- [ ] Fix shown as before/after code blocks
- [ ] Fix does not use `any` or `ts-ignore`
- [ ] At least two fix options shown for ambiguous cases
- [ ] "Why it works" section present
- [ ] Offer to help with follow-up errors


## Edge cases

- **No code context provided**: Ask for the specific line of code where
  the error occurs before attempting a fix.
- **Multiple errors in one paste**: Address the topmost / root cause error
  first; note that fixing it may resolve downstream errors automatically.
- **Error from a third-party library type**: Explain that the fix may
  involve augmenting the library's types with a `.d.ts` declaration file,
  and provide a minimal example.
- **Generic type errors (`T`, `K extends`)**: Draw out the concrete types
  being substituted to make the conflict visible before explaining.
- **User insists on `any` as the fix**: Explain the trade-off clearly —
  `any` disables type-checking for that variable, masking future bugs.
  Provide the proper fix alongside.