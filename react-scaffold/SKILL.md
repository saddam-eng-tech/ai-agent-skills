name: react-scaffold
description: Generates a complete React TypeScript component folder from a name and description: component file, typed props interface, Vitest test, Storybook story, and barrel export — all in one command. Trigger when a user says "create a new component", "scaffold a component", "generate component boilerplate", "set up a new React component", or "make a [Name] component". Also trigger when the user describes what a component should do and asks Claude to build or start it.

# React Scaffold

Produces a ready-to-use, fully typed React component folder in seconds.
Eliminates the 10-15 minutes spent creating the same 5 files for every
new component.

**Triggers on**: "create a new component", "scaffold", "generate component
boilerplate", "set up [ComponentName]", "make a React component for [purpose]"
**Input**: Component name + brief description of what it does + optional prop list
**Output**: 5 files — component, types (if large), test, story, index


## Step-by-step workflow

### STEP 1 — Gather component spec

From the user's message, extract:
- **Name**: PascalCase component name (ask if missing)
- **Description**: What the component renders / does
- **Props**: Any props the user mentioned (infer sensible ones if not given)
- **Variant**: Is this a UI atom (button, input), a container, or a page section?

If name is missing, ask one question: "What should the component be called?"
Do not ask for anything else — infer from context.


### STEP 2 — Plan the props interface

Design the TypeScript props interface based on:
- User-supplied props
- Component purpose (inferred from description)
- Sensible defaults (e.g. a card always gets `children`, a button gets `onClick`,
  a list gets `items` array)

Include at least:
- Required props with descriptive names
- Optional props with `?` and defaults in destructuring
- Event handlers typed with specific React event types (not just `() => void`)


### STEP 3 — Generate component file

Write `[ComponentName]/[ComponentName].tsx`:

```tsx
import React from 'react';

export interface [ComponentName]Props {
  // ... typed props
}

/**
 * [ComponentName] — [one-line description]
 */
export const [ComponentName]: React.FC<[ComponentName]Props> = ({
  // destructured props with defaults
}) => {
  return (
    // JSX
  );
};

[ComponentName].displayName = '[ComponentName]';
```

Rules:
- No `any` types
- Use `React.FC<Props>` pattern
- Set `displayName` for DevTools
- Keep JSX minimal but functional (not empty shell)


### STEP 4 — Generate test file

Write `[ComponentName]/[ComponentName].test.tsx`:

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { [ComponentName] } from './[ComponentName]';

describe('[ComponentName]', () => {
  it('renders without crashing', () => {
    render(<[ComponentName] [minimalRequiredProps] />);
    // assertion
  });

  it('displays expected content', () => {
    // test for primary visible output
  });

  // test for each significant prop
  // test for event handlers if applicable
});
```

Include tests for:
- Basic render
- Key prop variations
- User interactions (click, type) if the component has event handlers
- Edge cases (empty state, loading, error) if applicable


### STEP 5 — Generate Storybook story

Write `[ComponentName]/[ComponentName].stories.tsx`:

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { [ComponentName] } from './[ComponentName]';

const meta: Meta<typeof [ComponentName]> = {
  title: 'Components/[ComponentName]',
  component: [ComponentName],
  parameters: { layout: 'centered' },
  tags: ['autodocs'],
  argTypes: {
    // one argType per prop
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    // default args
  },
};

// 1-2 additional stories for key variants
```


### STEP 6 — Generate barrel export

Write `[ComponentName]/index.ts`:

```ts
export { [ComponentName] } from './[ComponentName]';
export type { [ComponentName]Props } from './[ComponentName]';
```


### STEP 7 — Write all files to disk and present

Write each file to `/home/claude/[ComponentName]/` then copy to
`/mnt/user-data/outputs/[ComponentName]/`.

Present all files. Print summary:

```
✅ Generated [ComponentName] scaffold:
  [ComponentName].tsx       — component
  [ComponentName].test.tsx  — 4 tests
  [ComponentName].stories.tsx — 2 stories
  index.ts                  — barrel export
```


## Output format spec

All files in `[ComponentName]/` folder. Naming: exact component name,
PascalCase for component files, lowercase `index.ts`.


## Quality checklist

- [ ] Props interface has no `any` types
- [ ] Test file imports from `@testing-library/react` and `userEvent`
- [ ] At least 3 test cases generated
- [ ] Storybook story has `argTypes` for every prop
- [ ] Barrel export exposes both component and Props type
- [ ] `displayName` set on component
- [ ] All files saved to output directory


## Edge cases

- **Name with spaces or lowercase**: Convert to PascalCase automatically
  (e.g. "user card" → `UserCard`)
- **No props mentioned**: Infer minimal sensible props from description;
  always include at minimum `className?: string`
- **User asks for class component**: Generate functional by default; note
  that functional components are the current best practice and offer to
  generate class version if they insist
- **User mentions a specific UI library (MUI, shadcn, Tailwind)**: Adapt
  the JSX and imports accordingly instead of bare HTML elements