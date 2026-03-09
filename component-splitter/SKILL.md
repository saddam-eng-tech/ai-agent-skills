name: component-splitter
description: Analyses a large (100+ line) React component, identifies natural split boundaries separating UI, logic, and data concerns, then produces refactored smaller components with a migration guide. Trigger when a user says "this component is too big", "split this component", "refactor this into smaller components", "break this up", "this component does too much", or shares a component > 100 lines asking how to improve its structure. Also trigger when PR review feedback mentions "component too large" or "separate concerns".

# Component Splitter

Decomposes large, multi-concern React components into focused, maintainable
pieces — with a concrete split plan and all output files ready to use.

**Triggers on**: "split this component", "too big", "break this up",
"refactor into smaller components", "this component does too much",
"separate concerns", component > 100 lines with mixed logic/UI
**Input**: Large React component .tsx file (ideally 100+ lines)
**Output**: Set of smaller focused files + migration guide


## Step-by-step workflow

### STEP 1 — Read the component

Use `view` for file path, or read from conversation.

Count lines. If < 80 lines, output:
> "This component is [N] lines — below the typical threshold for splitting.
> It may still benefit from separation if it has mixed concerns; shall I
> analyse anyway?"

For 80+ lines, proceed.


### STEP 2 — Identify responsibilities

Map out what the component is doing by scanning for these concern categories:

| Concern | Signs in code |
|---------|--------------|
| **Data fetching** | `useEffect` with `fetch` / API calls / query hooks |
| **Business logic** | Data transformation, filtering, sorting, calculations |
| **Form handling** | `useState` for form fields, validation logic, `onSubmit` |
| **UI rendering** | JSX, conditional renders, style logic |
| **State management** | Multiple `useState` / `useReducer` calls |
| **Context provision** | `Context.Provider` in JSX |
| **Side effects** | Timer setup, subscription, event listeners |

List every concern found with the approximate lines where they appear.


### STEP 3 — Design the split plan

Apply these standard decomposition patterns:

**Pattern A: Container/Presentational**
- Create `[Name]Container.tsx` — handles data fetching + state
- Create `[Name].tsx` (presentational) — receives data as props, pure UI
- *Use when*: Component fetches its own data AND renders complex UI

**Pattern B: Custom hook extraction**
- Extract logic into `use[Name].ts` custom hook
- Keep UI in the component, call the hook
- *Use when*: Heavy business logic or multiple related state values

**Pattern C: Sub-component extraction**
- Extract repeated or self-contained JSX sections into named sub-components
- *Use when*: Sections of JSX are large and conceptually distinct

**Pattern D: Compound pattern**
- Combine patterns: hook for logic + sub-components for UI sections
- *Use when*: Component has data logic AND large multi-section UI

Choose the best-fit pattern. Explain the choice.


### STEP 4 — Write the split plan table

Present before writing any code:

```
## Split Plan for [ComponentName]

Current: 1 file, [N] lines, [N] responsibilities

Proposed split (Pattern [A/B/C/D]):

| File | Responsibility | Lines (est.) |
|------|---------------|-------------|
| use[Name].ts | Data fetching, state, derived values | ~40 |
| [Name].tsx | Presentational UI only | ~50 |
| [Name]Form.tsx | Form section extracted | ~30 |

Total: 3 files, ~40 lines avg each (was [N] lines in one)
```


### STEP 5 — Generate all split files

For each planned file, write the complete implementation:

**Custom hook (`use[Name].ts`):**
```tsx
import { useState, useEffect, useCallback } from 'react';
import type { [DataType] } from './types';

interface Use[Name]Return {
  data: [DataType][];
  isLoading: boolean;
  error: string | null;
  handleAction: (id: string) => void;
}

export function use[Name](): Use[Name]Return {
  // all the logic
}
```

**Presentational component (`[Name].tsx`):**
```tsx
import React from 'react';
import type { [DataType] } from './types';

interface [Name]Props {
  // all data as props, no fetching here
}

export const [Name]: React.FC<[Name]Props> = (props) => {
  // pure JSX only
};
```

**Updated parent / entry point:**
Show how the original file is simplified to just wire the pieces:
```tsx
import { use[Name] } from './use[Name]';
import { [Name] } from './[Name]';

export const [Name]Container: React.FC = () => {
  const { data, isLoading, handleAction } = use[Name]();
  return <[Name] data={data} isLoading={isLoading} onAction={handleAction} />;
};
```


### STEP 6 — Write the migration guide

Create `MIGRATION.md`:

```markdown
# [ComponentName] Split Migration Guide

## What changed
- `[ComponentName].tsx` (original) → split into N files
- No behaviour changed — this is a pure refactor

## File map
| Old | New |
|-----|-----|
| [ComponentName].tsx (lines 1-40) | use[Name].ts |
| [ComponentName].tsx (lines 41-120) | [Name].tsx |

## How to update imports
Before: `import { [ComponentName] } from './[ComponentName]'`
After: `import { [Name]Container } from './[Name]Container'`
(or keep the same export name — see [Name]Container.tsx)

## Testing
- Existing tests for [ComponentName] should still pass
- New unit tests for use[Name] hook: [suggested test file]
```


### STEP 7 — Output all files

Write each file to `/mnt/user-data/outputs/[ComponentName]-split/`.
Present all files plus `MIGRATION.md`.

Print summary:
```
✅ Split complete: [ComponentName] → N files
  Files created: [list]
  Avg lines per file: N (was N)
  Pattern used: [A/B/C/D] — [name]
```


## Output format spec

- Output folder: `[ComponentName]-split/`
- Includes: all component files, hook file if applicable, MIGRATION.md
- No behaviour changes in refactored code
- All types preserved exactly


## Quality checklist

- [ ] All original functionality preserved in split files
- [ ] No props or state removed — just reorganised
- [ ] Custom hook has explicit return type interface
- [ ] Presentational component has no data fetching
- [ ] MIGRATION.md explains import changes
- [ ] File line counts shown in plan table
- [ ] Pattern choice explained in one sentence


## Edge cases

- **Component under 80 lines but has mixed concerns**: Proceed with
  smaller targeted extraction (likely Pattern B: hook only).
- **Component is already presentational (no side effects)**: Extract
  only UI sub-components (Pattern C). Note: "No logic to extract — only
  splitting UI sections."
- **Component is tightly coupled to a specific store/context**: Note the
  coupling in the migration guide; suggest the hook interface as the
  abstraction boundary.
- **Shared types between split files**: Create a `types.ts` file in the
  same folder and import from there in all split files.
- **User wants a specific split point**: Accept their guidance and apply
  it, overriding the auto-detected pattern.