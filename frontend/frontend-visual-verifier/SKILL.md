---
name: frontend-visual-verifier
description: >
  Visual review sub-agent for the frontend agent team. Navigates a locally
  running dev server via GitHub Copilot browser agent mode (VS Code agent mode
  or Copilot Workspace preview) to visually inspect implemented components,
  then compares against the Figma design or reference image. Produces a
  structured PASS/PARTIAL/FAIL report on layout, spacing, and color fidelity.
  Trigger when invoked by frontend-agent-orchestrator, or when the user says
  "visually check this", "verify the UI", "does this match the design",
  "check the local preview", "compare against Figma", or "run the browser check".
  Falls back to static screenshot diff or manual-review flag if browser tooling
  is unavailable.
---

# Frontend Visual Verifier

Checks that the implemented UI matches the design reference.
Uses GitHub Copilot browser agent (VS Code) to navigate localhost and screenshot.

**Triggers on**: invoked by `frontend-agent-orchestrator`, "visually check this",
"verify the UI", "compare against Figma", "check the local preview"
**Input**: Code report payload + plan payload + dev server URL
**Output**: Visual verifier report (schema section 4)

---

## Step-by-step workflow

### STEP 1 — Confirm browser tooling availability

Check which preview mechanism is available:

**Option A — GitHub Copilot in VS Code (agent mode)**
- Requires VS Code with GitHub Copilot extension in agent mode (`@workspace`)
- Copilot's browser tool can navigate to `localhost` and capture screenshots
- Available when user is running the pipeline from Claude Code or a Copilot-enabled terminal

**Option B — GitHub Copilot Workspace (browser-based)**
- Built-in preview pane available for web-based Copilot Workspace sessions
- Use the workspace preview URL instead of localhost

**Option C — No browser tooling (fallback)**
- If neither A nor B is available, set `screenshot_taken: false`
- Set `manual_review_needed: true` with reason "browser tooling unavailable"
- Still perform static analysis: review the component JSX/TSX against the design
  description for obvious structural mismatches
- Flag specific things to manually verify (e.g. "check spacing on mobile viewport")

Confirm which option is active before proceeding.

---

### STEP 2 — Start / confirm the dev server

Before navigating, confirm the dev server is running:
```
Expected: http://localhost:3000 (or whatever port is in package.json scripts)
```

If not running:
- Instruct the coder agent (or the user) to run `npm run dev` / `yarn dev`
- Wait for "ready" signal before navigating
- Do not time out for more than 30 seconds — if it doesn't start, fall back to Option C

---

### STEP 3 — Navigate to each component

For each component in the plan's `components_to_build`:

Determine the route to navigate to:
- If the component is a full page: navigate to its route (from the router config)
- If it's a sub-component: navigate to a Storybook story
  (`http://localhost:6006` or equivalent) or the page it lives on
- If neither: ask the user for the route/story path and record it

For each component, capture:
1. Default state (initial render)
2. Interactive state if applicable (hover, focus, active, loading, error)
3. Mobile viewport (375px wide) if the feature is responsive

---

### STEP 4 — Compare against design reference

For each captured screenshot, compare against the Figma node or reference image:

#### Layout
- Overall structure matches (columns, rows, card structure)
- Component hierarchy matches (nesting, containment)
- Responsive behavior matches (breakpoints, stacking)

#### Spacing
- Margins and padding visually consistent with design
- Gap between elements matches design tokens
- Note: exact pixel matching is not required — check proportional consistency

#### Color
- Background colors match design tokens
- Text colors match
- Border colors match
- Interactive state colors (hover, focus rings) match

#### Typography
- Font sizes visually consistent
- Font weights correct (regular vs medium vs bold)
- Line height and letter spacing roughly match

---

### STEP 5 — Assign verdict per component

| Match level | Verdict | Meaning |
|---|---|---|
| All layout/spacing/color/type match | `PASS` | Ship it |
| Minor spacing or color discrepancies | `PARTIAL` | Note it, orchestrator decides |
| Major layout mismatch, wrong colors, missing elements | `FAIL` | Block — fix needed |

PARTIAL is not failure. The orchestrator decides whether PARTIAL warrants a fix cycle.
Use FAIL only for obviously broken layouts or completely wrong designs.

---

### STEP 6 — Produce visual verifier report

Return the report payload as defined in
`../frontend-agent-team/references/payload-schema.md` → section 4.

For each component, `notes` should be specific:
- Good: "Bottom padding on card is ~8px, design shows 16px. Use `pb-4` instead of `pb-2`."
- Bad: "Spacing looks off."

Set `manual_review_needed: true` if:
- Browser tooling was unavailable (Option C)
- Screenshots could not be captured for any reason
- The design reference was only an image (not Figma with tokens)
- The component requires a specific data state that couldn't be reproduced locally

Always populate `preview_url` with the URL that was (or would have been) used,
even in fallback mode. It helps the human do a quick manual check.