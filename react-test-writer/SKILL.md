name: react-test-writer
description: >
  Reads a React TypeScript component file and generates a comprehensive
  Vitest + React Testing Library test suite covering rendering, prop variations,
  user interactions, accessibility, and error states. Trigger when a user says
  "write tests for this component", "generate unit tests", "add test coverage",
  "create tests for [ComponentName]", "help me test this React component", or
  pastes a component file and asks about testing. Also trigger when users ask
  "how do I test [specific hook/prop behaviour]" and have shared a component.

# React Test Writer

Generates a complete Vitest + RTL test file for any React TypeScript component
— from simple renders to user interactions and accessibility checks.

**Triggers on**: "write tests for", "generate tests", "add test coverage",
"create test file", "unit test this component", "test this React component"
**Input**: React component .tsx file (pasted or uploaded)
**Output**: `[ComponentName].test.tsx` test file


## Step-by-step workflow

### STEP 1 — Analyse the component

Read the component file. Identify:

1. **Component name** and exported identifier
2. **Props interface** — required vs optional, event callbacks, children
3. **Internal state** (`useState`) — what triggers changes
4. **Render output** — what text, roles, or elements are always present
5. **Conditional renders** — what shows when a prop/state is truthy/falsy
6. **Event handlers** — `onClick`, `onChange`, `onSubmit`, etc.
7. **Async behaviour** — loading states, data fetching
8. **Accessibility attributes** — `aria-*`, `role`, `aria-label`


### STEP 2 — Plan the test cases

Build a test plan covering these categories:

| Category | What to test |
|----------|-------------|
| **Render** | Mounts without crashing; key elements visible |
| **Props** | Each required prop; optional prop with/without |
| **Interactions** | Each event handler fires; state changes update UI |
| **Conditional UI** | Loading state, empty state, error state |
| **Accessibility** | Role present, aria-label correct, keyboard nav |
| **Callbacks** | Passed callbacks called with correct arguments |

Aim for 5-10 tests. Prioritise breadth over exhaustive edge cases.


### STEP 3 — Write the test file

Use this structure:

```tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { vi } from 'vitest';
import { [ComponentName] } from './[ComponentName]';

// Helper: minimal required props
const defaultProps = {
  // ... minimum required props
};

describe('[ComponentName]', () => {

  // — RENDER ——————————————————————————————
  describe('rendering', () => {
    it('renders without crashing', () => {
      render(<[ComponentName] {...defaultProps} />);
      // Use role-based queries, not test IDs
      expect(screen.getByRole('[role]')).toBeInTheDocument();
    });
  });

  // — PROPS ———————————————————————————————
  describe('props', () => {
    it('displays the [prop] value', () => { ... });
    it('applies disabled state when disabled prop is true', () => { ... });
  });

  // — INTERACTIONS ————————————————————————
  describe('interactions', () => {
    it('calls onAction when [element] is clicked', async () => {
      const onAction = vi.fn();
      render(<[ComponentName] {...defaultProps} onAction={onAction} />);
      await userEvent.click(screen.getByRole('button'));
      expect(onAction).toHaveBeenCalledTimes(1);
    });
  });

  // — ACCESSIBILITY ———————————————————————
  describe('accessibility', () => {
    it('has correct ARIA attributes', () => { ... });
  });

});
```

**Query priority** (always prefer in this order):
1. `getByRole` — most accessible, mirrors what a screen reader sees
2. `getByLabelText` — for form elements
3. `getByText` — for visible text
4. `getByPlaceholderText` — inputs
5. `getByTestId` — last resort only

**Avoid**:
- `container.querySelector` — brittle
- `getByClassName` — brittle
- snapshot tests — noisy, low value unless specifically requested


### STEP 4 — Handle async components

If the component has async behaviour (data fetching, loading states):

```tsx
it('shows loading state while fetching', async () => {
  render(<[ComponentName] {...defaultProps} />);
  expect(screen.getByRole('progressbar')).toBeInTheDocument();
  await waitFor(() => {
    expect(screen.queryByRole('progressbar')).not.toBeInTheDocument();
  });
});
```

Mock external calls with `vi.mock`:
```tsx
vi.mock('./api', () => ({ fetchData: vi.fn().mockResolvedValue([]) }));
```


### STEP 5 — Write setup boilerplate check

Inspect if the user's project likely uses a setup file. If no setup file
is mentioned, add a comment at the top:

```tsx
// Requires @testing-library/jest-dom in vitest.setup.ts:
// import '@testing-library/jest-dom';
```


### STEP 6 — Save and present

Write to `/mnt/user-data/outputs/[ComponentName].test.tsx`. Present the file.

Print summary:
```
✅ Test file generated: [ComponentName].test.tsx
  - Test cases: N
  - Render tests: N
  - Interaction tests: N
  - Accessibility tests: N
  - Async tests: N
```


## Output format spec

- Single `[ComponentName].test.tsx` file
- Uses `describe` nesting: component → category → individual test
- Uses `vi.fn()` for mock functions, not Jest `jest.fn()`
- Uses `userEvent` for interactions, not `fireEvent`
- Imports at top: RTL, userEvent, vi, component


## Quality checklist

- [ ] At least 5 test cases generated
- [ ] No `getByTestId` unless absolutely unavoidable
- [ ] Event handlers tested with `vi.fn()` and `expect().toHaveBeenCalled()`
- [ ] Async tests use `waitFor` or `findBy*` queries
- [ ] At least one accessibility test included
- [ ] defaultProps helper defined to avoid repeating required props
- [ ] No snapshot tests (unless user specifically requested)


## Edge cases

- **Component with no props**: Write render + content tests only; note
  "No props interface found — testing render and visible output only."
- **Component uses context/store**: Wrap with a minimal provider in tests;
  create a `renderWithProviders` helper at the top of the test file.
- **Component is a form**: Always test submit handler called on form submit
  AND validation messages shown on invalid input.
- **Children-based components**: Test that children are rendered correctly
  using `render(<Component>child text</Component>)`.
- **Component conditionally renders null**: Add test case verifying nothing
  is rendered under the null-triggering condition.