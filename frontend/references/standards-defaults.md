# Standards defaults

Used by the orchestrator when no explicit standards doc is found in the repo.
These are Carousell-aligned React/TypeScript defaults. Always confirm with
the user before applying these — they are fallbacks, not facts.

---

## Naming

| Thing | Convention | Example |
|---|---|---|
| Components | PascalCase, matches filename | `ProductCard.tsx` |
| Hooks | `use` prefix, camelCase | `useProductList` |
| Event handlers | `handle` + Event | `handleSubmit`, `handleChange` |
| Redux slices | `<feature>Slice` | `productListSlice` |
| Selectors | `select` + Thing | `selectProductList` |
| Types/interfaces | PascalCase, no `I` prefix | `ProductCardProps` |
| Constants | SCREAMING_SNAKE | `MAX_RETRY_COUNT` |
| Test files | `__tests__/ComponentName.test.tsx` | co-located |

---

## Imports

- Absolute imports via `@/` alias pointing to `src/`
- No deep relative paths (`../../../`) — use alias instead
- Barrel `index.ts` allowed at component folder level
- External libraries before internal imports (ESLint import/order)

---

## State management

- Redux Toolkit (`createSlice`, `createAsyncThunk`)
- Typed hooks: `useAppDispatch`, `useAppSelector` (no raw hooks)
- Selectors co-located in slice file
- No direct store mutation outside `extraReducers` / `reducers`
- Server state: React Query (TanStack Query) — do not put API response cache in Redux

---

## TypeScript

- `strict: true` — no `any` without explicit comment explaining why
- Explicit return types on all exported functions and hooks
- Prefer `interface` for object shapes, `type` for unions/aliases
- No non-null assertions (`!`) unless provably safe — use optional chaining

---

## Folder structure

```
src/
  components/
    FeatureName/
      ComponentName.tsx
      ComponentName.module.css   (if CSS Modules used)
      __tests__/
        ComponentName.test.tsx
      index.ts                   (barrel export)
  hooks/
    useHookName.ts
    __tests__/
      useHookName.test.ts
  store/
    slices/
      featureSlice.ts
    hooks.ts                     (useAppDispatch, useAppSelector)
    index.ts
  types/
    feature.types.ts
```

---

## Testing

- Framework: Vitest
- Library: React Testing Library
- Prefer `screen.getByRole` > `getByText` > `getByTestId`
- Wrap Redux-connected components in a `renderWithStore` helper
- Minimum per component: render test + one user interaction test
- No snapshot tests (brittle, avoid)

---

## Code style

- No `console.log` in production code (use a logger utility)
- No magic numbers — extract to named constants
- No TODOs committed to main (convert to tickets)
- Max 200 lines per component file — split if larger
- Props spread (`{...props}`) only at the outermost element, and only if intentional