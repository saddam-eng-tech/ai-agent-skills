# JS → TS Migrator

Converts React JavaScript files to production-grade TypeScript by inferring
types from usage patterns — not just slapping `: any` everywhere.

**Triggers on**: "convert to TypeScript", "migrate to TS", "add types",
"turn jsx into tsx", "JS to TS", "add type safety to this component"
**Input**: A .jsx / .js React component file (pasted or uploaded)
**Output**: Equivalent .tsx file with full TypeScript types


## Step-by-step workflow

### STEP 1 — Read the source file

Use `view` if a file path is given. Otherwise read from the conversation text.

Check for: PropTypes definitions (they reveal shape), defaultProps (reveal
optionals and defaults), function signatures, JSDoc comments, hook usage.


### STEP 2 — Extract type information

Scan the file and build a type map:

**Props** — gather from:
1. PropTypes definitions (highest confidence)
2. How props are destructured and used inside the component
3. defaultProps (marks optional)
4. JSDoc `@param` comments

**State** — gather from `useState(initialValue)`:
- Infer state type from initial value
- If initial value is `null` / `undefined`, look at first setState call

**Refs** — gather from `useRef(initialValue)`:
- `useRef(null)` on a button → `useRef<HTMLButtonElement>(null)`
- Look at how `.current` is used

**Event handlers** — infer from:
- `onChange` on `<input>` → `React.ChangeEvent<HTMLInputElement>`
- `onClick` on `<button>` → `React.MouseEvent<HTMLButtonElement>`
- `onSubmit` on `<form>` → `React.FormEvent<HTMLFormElement>`
- Keyboard events, focus events, drag events — use specific types

**Callbacks from parent** — typed as specific function signatures based on
what arguments they're called with in the component.


### STEP 3 — Handle PropTypes removal

If PropTypes are present:
1. Extract the shape into a TypeScript interface
2. Remove the `import PropTypes from 'prop-types'` line
3. Remove the `ComponentName.propTypes = { ... }` block
4. Move defaultProps values into parameter destructuring defaults


### STEP 4 — Write the TypeScript file

Produce the `.tsx` file following this pattern:

```tsx
import React from 'react';

interface [ComponentName]Props {
  // Required props (no ?)
  requiredProp: string;
  // Optional props (with ?)
  optionalProp?: number;
  // Callbacks
  onAction?: (value: string) => void;
}

const [ComponentName]: React.FC<[ComponentName]Props> = ({
  requiredProp,
  optionalProp = 42,   // defaults here
  onAction,
}) => {
  const [state, setState] = React.useState<StateType>(initial);
  const ref = React.useRef<HTMLElementType>(null);

  // ... rest of component
};

export default [ComponentName];
```

Rules:
- **Never use `any`** unless genuinely unavoidable (and comment why)
- **Never use non-null assertion `!`** unless you can prove it's safe
- Remove all PropTypes imports and definitions
- Add `displayName` if not present
- Type all event handlers with specific React event types


### STEP 5 — Flag manual review items

After producing the migrated file, add a comment block at the top listing
anything that needs a human decision:

```tsx
/*
 * MIGRATION NOTES — please review:
 * 1. Line 45: fetchData return type inferred as any — verify API response shape
 * 2. Line 78: onSuccess callback — confirm argument types match parent usage
 * 3. Prop `config` typed as Record<string, unknown> — consider a specific interface
 */
```


### STEP 6 — Output the file

Save to `/mnt/user-data/outputs/[ComponentName].tsx` and present it.

Print summary:
```
✅ Migration complete: [originalFile].jsx → [ComponentName].tsx
  - Props typed: N
  - State typed: N hooks
  - Event handlers typed: N
  - PropTypes removed
  - ⚠️ Manual review items: N (see comments at top)
```


## Output format spec

- Single `.tsx` file
- Migration notes comment block at file top if issues found
- No PropTypes imports remaining
- All TypeScript types explicitly declared (no inferred-to-any)


## Quality checklist

- [ ] No `any` types (except commented exceptions)
- [ ] No `!` non-null assertions without explanation
- [ ] All useState hooks have explicit generic type parameters
- [ ] All useRef hooks have explicit element type
- [ ] All event handlers use specific React event types (not `Event`)
- [ ] PropTypes import and definitions removed
- [ ] Interface exported so parent can use it
- [ ] Migration notes added for unresolvable ambiguities


## Edge cases

- **Component uses PropTypes.shape({})**: Extract nested shape into a
  separate interface.
- **Component renders other components**: Type their props as
  `React.ComponentProps<typeof OtherComponent>` if no interface exists.
- **File uses `React.forwardRef`**: Migrate to TypeScript forwardRef pattern
  with explicit type parameter `React.forwardRef<RefType, PropsType>`.
- **Spread props `{...props}`**: Type as `React.HTMLAttributes<HTMLElement>`
  or the specific element type.
- **HOC wrapping**: Note in migration comments that HOC types need separate
  attention.
