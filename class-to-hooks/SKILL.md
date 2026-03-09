name: class-to-hooks
description: Converts a React class component to a functional component using hooks, mapping lifecycle methods to useEffect patterns, this.state to useState, instance methods to functions or useCallback, refs to useRef, and context to useContext. Trigger when a user says "convert this class component to functional", "refactor to hooks", "migrate from class to function", "modernise this component", or pastes a class component and asks for a hooks version. Also trigger when users have a component using componentDidMount, componentDidUpdate, or componentWillUnmount.

# Class → Hooks Refactorer

Transforms React class components into modern functional components with
hooks, following the exact lifecycle mapping table — no guessing.

**Triggers on**: "convert class to functional", "refactor to hooks",
"migrate to hooks", "modernise this class component", "rewrite with hooks",
"remove class component"
**Input**: React class component .tsx / .jsx file
**Output**: Equivalent functional component using hooks


## Step-by-step workflow

### STEP 1 — Read and inventory the class component

Use `view` if file path is given, otherwise read from conversation.

Build an inventory:

```
State properties: [list from this.state = {}]
Lifecycle methods: [list all present]
Instance methods: [list non-lifecycle methods]
Refs: [list from createRef / callback refs]
Context: [list from static contextType or Consumer]
Props: [from PropTypes or TypeScript interface]
```


### STEP 2 — Map lifecycle methods to hooks

Apply this exact mapping table:

| Class lifecycle | Hooks equivalent |
|----------------|-----------------|
| `constructor` (state init) | `useState(initialValue)` |
| `constructor` (other setup) | variable declarations / `useRef` |
| `componentDidMount` | `useEffect(() => { ... }, [])` |
| `componentDidUpdate(prevProps, prevState)` | `useEffect(() => { ... }, [deps])` — deps are the values compared to prev |
| `componentWillUnmount` | cleanup `return () => { ... }` in a useEffect |
| `shouldComponentUpdate` | `React.memo()` wrapper + custom comparison fn |
| `getDerivedStateFromProps` | derive value during render (no hook needed) OR `useEffect` |
| `getSnapshotBeforeUpdate` | `useRef` to capture pre-update value |
| `componentDidCatch / getDerivedStateFromError` | **Cannot be converted** — keep as class or use error boundary library |

**`componentDidUpdate` migration is the trickiest:**
- Identify which state/props are being compared
- Those comparisons become the dependency array
- `prevProps.x !== props.x` → `[props.x]` in the dep array
- Multiple conditions → split into separate `useEffect` calls


### STEP 3 — Convert state

For each `this.state` property, create a `useState` call:

```tsx
// Class
this.state = { count: 0, name: '' };

// Hooks
const [count, setCount] = useState<number>(0);
const [name, setName] = useState<string>('');
```

Replace all `this.setState({ key: value })` with the individual setter:
- `this.setState({ count: 5 })` → `setCount(5)`
- `this.setState(prev => ({ count: prev.count + 1 }))` → `setCount(prev => prev + 1)`


### STEP 4 — Convert instance methods

Non-lifecycle methods become plain `const` functions inside the component:

```tsx
// Class
handleClick() { this.setState({ count: this.state.count + 1 }); }

// Hooks
const handleClick = () => setCount(prev => prev + 1);
```

If a method is passed as a prop to children, wrap in `useCallback` with
the correct dependencies.


### STEP 5 — Convert refs

```tsx
// Class
this.inputRef = React.createRef();
// Usage: this.inputRef.current

// Hooks
const inputRef = React.useRef<HTMLInputElement>(null);
// Usage: inputRef.current
```


### STEP 6 — Convert context

```tsx
// Class (static contextType)
static contextType = ThemeContext;
// Usage: this.context.theme

// Hooks
const theme = useContext(ThemeContext);
```


### STEP 7 — Write the functional component

Produce the complete functional component. Preserve all logic. Do not
simplify or remove functionality.

Add a comment block above the component:
```tsx
/*
 * Converted from class component to functional hooks.
 * Original: [filename]
 * Conversion notes:
 * - componentDidMount → useEffect([], [])
 * - [any tricky mappings noted]
 */
```


### STEP 8 — Flag unconvertible patterns

If `componentDidCatch` or `getDerivedStateFromError` is present, add:
```tsx
// ⚠️ ERROR BOUNDARY: This class pattern cannot be converted to hooks.
// Recommendation: Wrap this component with an ErrorBoundary class component
// or use the 'react-error-boundary' library.
```


### STEP 9 — Output

Save to `/mnt/user-data/outputs/[ComponentName].tsx`. Present the file.

Print summary:
```
✅ Refactor complete: [ComponentName] (class → functional)
  - State properties converted: N → N useState hooks
  - Lifecycle methods converted: [list]
  - Instance methods converted: N
  - Refs converted: N
  - ⚠️ Unconvertible patterns: [list or 'none']
```


## Output format spec

- Single `.tsx` file
- Conversion comment block at top
- Props interface unchanged from original (or inferred from PropTypes)
- No `this.` references remaining


## Quality checklist

- [ ] No `this.` references remaining in output
- [ ] Every lifecycle method accounted for in mapping
- [ ] State setters use functional updater pattern where previous state is needed
- [ ] Methods passed as props wrapped in `useCallback`
- [ ] All `createRef` calls converted to `useRef`
- Conversion notes comment block present
- [ ] Unconvertible patterns flagged clearly


## Edge cases

- **Component extends another class component**: Note this dependency — the
  parent class cannot be expressed in hooks; suggest composition instead.
- **`this.forceUpdate()`**: Replace with a dummy state toggle:
  `const [, forceRender] = useReducer(x => x + 1, 0);`
- **`PureComponent`**: Add `React.memo()` wrapper around the functional
  component.
- **Component has both `componentDidUpdate` with multiple conditions**:
  Split into separate `useEffect` calls, one per concern.
- **Method called in multiple places including lifecycle**: Extract to a
  `useCallback` or a plain utility function outside the component if it
  has no state dependencies.