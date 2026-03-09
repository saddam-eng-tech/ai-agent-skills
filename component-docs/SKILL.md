name: component-docs
description: Generates JSDoc comments for a React TypeScript component's props, functions, and exported members, AND produces a Storybook stories file with argTypes for every prop and multiple story variants. Trigger when a user says "write docs for this component", "add JSDoc", "create Storybook story", "document my component", "add prop documentation", or "generate a stories file". Also trigger when users share a component and ask "how do I document this" or "can you add comments to this".

# Component Docs Writer

Produces JSDoc-annotated component source + a complete `.stories.tsx` file
with controls for every prop — from a bare TypeScript component.

**Triggers on**: "write docs for", "add JSDoc", "create story", "document
this component", "add Storybook stories", "generate stories file",
"add prop documentation"
**Input**: React TypeScript component file
**Output**: JSDoc-annotated component file + `[ComponentName].stories.tsx`


## Step-by-step workflow

### STEP 1 — Read and analyse the component

Use `view` for file paths, or read from conversation.

Extract:
- Component name
- Props interface — every field with its type and whether it's optional
- What the component renders (visible output, purpose)
- Any interesting behaviour (animations, state changes, async)
- Event callback props (names, signatures)
- Children support


### STEP 2 — Write JSDoc for the props interface

For each prop in the interface, add a `/** */` JSDoc comment above it:

```tsx
export interface ButtonProps {
  /** The text or element to display inside the button. */
  children: React.ReactNode;

  /** Visual style variant of the button. @default 'primary' */
  variant?: 'primary' | 'secondary' | 'danger';

  /** Whether the button is in a disabled state, preventing interaction. */
  disabled?: boolean;

  /** Callback fired when the button is clicked. */
  onClick?: React.MouseEventHandler<HTMLButtonElement>;

  /** Additional CSS class names to apply to the root element. */
  className?: string;
}
```

Rules:
- First sentence: what the prop IS (noun phrase, starts capital)
- Second sentence (if needed): what happens when it changes
- Always add `@default [value]` for optional props that have a default
- For callback props: describe what triggers the call and what args it receives
- For `children`: always note what type is accepted


### STEP 3 — Write JSDoc for the component itself

Add a JSDoc block above the component:

```tsx
/**
 * [ComponentName] — [one sentence purpose].
 *
 * [Optional: 2nd sentence describing when/why to use it.]
 *
 * @example
 * ```tsx
 * <[ComponentName] [minimal required props] />
 * ```
 *
 * @example
 * ```tsx
 * // [Variant name] variant
 * <[ComponentName] [variant props] />
 * ```
 */
```

Include:
- 1-line summary
- 1-2 usage examples using `@example`
- `@see` link if relevant related component exists


### STEP 4 — Write the Storybook stories file

Create `[ComponentName].stories.tsx` using CSF3 format:

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { [ComponentName] } from './[ComponentName]';

const meta: Meta<typeof [ComponentName]> = {
  title: 'Components/[ComponentName]',
  component: [ComponentName],
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: '[One sentence describing the component for Storybook docs]',
      },
    },
  },
  tags: ['autodocs'],
  argTypes: {
    // One entry per prop
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger'],
      description: 'Visual style variant',
      table: { defaultValue: { summary: 'primary' } },
    },
    disabled: {
      control: 'boolean',
      description: 'Disables the button',
    },
    onClick: {
      action: 'clicked',
      description: 'Callback when button is clicked',
    },
    // ... etc
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    // minimal working args
  },
};

export const [VariantName]: Story = {
  name: '[Human readable name]',
  args: {
    // variant-specific args
  },
};

// 1 story per significant prop variant
// Always include: Default, disabled state if applicable, different variant if applicable
```

**argType control mapping:**
| TypeScript type | Storybook control |
|----------------|------------------|
| `string` | `'text'` |
| `number` | `'number'` |
| `boolean` | `'boolean'` |
| string union | `'select'` with `options` |
| `React.ReactNode` | `'text'` (simplified) |
| callback `() => void` | `action: 'actionName'` |
| `Date` | `'date'` |


### STEP 5 — Output both files

Write:
1. The annotated component to `/mnt/user-data/outputs/[ComponentName].tsx`
2. The stories file to `/mnt/user-data/outputs/[ComponentName].stories.tsx`

Present both files.

Print summary:
```
✅ Documentation generated:
  [ComponentName].tsx           — JSDoc added to N props + component
  [ComponentName].stories.tsx   — N stories, N argTypes
```


## Output format spec

- Annotated component: same file structure, JSDoc blocks added only
  (no logic changes)
- Stories file: CSF3 format, `autodocs` tag, `parameters.docs.description`
- Minimum 2 stories: Default + at least one variant


## Quality checklist

- [ ] Every prop has a JSDoc `/** */` comment
- [ ] Component has a JSDoc block with `@example`
- [ ] Default values documented with `@default`
- [ ] Stories file uses CSF3 format (`StoryObj`)
- [ ] Every prop has an `argType` entry in the meta
- [ ] At least 2 exported stories (Default + 1 variant)
- [ ] No logic changes made to the component


## Edge cases

- **Component has no props interface**: Infer from usage and write the
  interface with JSDoc, then suggest the user add it to the original file.
- **Callback with complex arguments**: Document each argument with `@param`
  style in the JSDoc for that prop.
- **Component uses children extensively**: Create a story that demonstrates
  children composition, not just text.
- **Internal-only functions (not exported)**: Add JSDoc only if the function
  is complex (> 5 lines) or non-obvious.
- **Component file is very long (> 200 lines)**: Only document the exported
  component and its public interface; skip private helpers.