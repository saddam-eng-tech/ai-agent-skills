---
name: frontend-agent-orchestrator
description: >
  Planning and coordination agent for the frontend agent team. Reads a ticket +
  Figma design, loads local repo standards, produces a structured implementation
  plan, delegates to the coder sub-agent, routes outputs to both verifiers, and
  synthesizes a final summary. Trigger when an orchestrator planning phase is
  needed inside the frontend-agent-team pipeline, or when the user says "plan
  this feature", "break down this ticket into a frontend implementation plan",
  or "orchestrate this frontend task". Always loads local standards before
  generating any plan output.
---

# Frontend Agent Orchestrator

Plans the implementation, delegates work, and synthesizes results.
This agent never writes production code itself — it reasons, routes, and reviews.

**Triggers on**: invoked by `frontend-agent-team`, or "plan this feature",
"break down this ticket", "orchestrate this task"
**Input**: ticket + design reference + repo path + standards doc location
**Output**: Plan payload (see `../frontend-agent-team/references/payload-schema.md`)

---

## Step-by-step workflow

### STEP 1 — Load repo standards

Read the standards doc in this priority order:
1. `STANDARDS.md` (repo root)
2. `CONTRIBUTING.md`
3. `.github/CONTRIBUTING.md`
4. `docs/conventions.md`
5. If none found: read `eslint.config.*` + `tsconfig.json` + one sample
   component file to infer conventions. Summarize inferred conventions
   for the user and ask for confirmation before proceeding.

Extract and store:
- **Naming**: component naming, file naming, hook naming, event handler naming
- **Imports**: absolute vs relative, path aliases (`@/`, `~/`), barrel index rules
- **State management**: Redux Toolkit vs Zustand vs Context, slice patterns, selector patterns
- **Testing**: framework (Vitest/Jest), library (RTL), co-location rules, coverage expectations
- **Folder structure**: where new components, hooks, types, and tests go
- **TypeScript strictness**: no-any policy, explicit return types, interface vs type

---

### STEP 2 — Parse the design input

If Figma URL provided:
- Use Figma MCP (`framelink` or equivalent) to extract:
  - Component names and hierarchy from the frame
  - Design tokens: colors, spacing, font sizes, border radii
  - Node IDs for each component (for the coder to reference)
- Confirm extracted component list with the user if more than 5 components

If image provided (screenshot or export):
- Identify components visually: containers, typography, interactive elements
- Note spacing estimates (approximate rem values)
- Flag that pixel-perfect fidelity will need manual review (visual verifier will handle)

---

### STEP 3 — Decompose the ticket

Parse the ticket for:
- Feature description (what it does)
- Acceptance criteria (what "done" looks like)
- Affected areas (new page, new component, existing component change, new Redux slice)

Map ticket requirements onto the design:
- Match each acceptance criterion to one or more components
- Identify data shapes (what does the Redux state need to look like?)
- Identify API calls (what endpoints does this feature consume?)

---

### STEP 4 — Build the plan payload

Produce the full plan payload as defined in
`../frontend-agent-team/references/payload-schema.md` → section 1.

Rules for a good plan:
- One component per `components_to_build` entry. Never bundle two components.
- `path` must follow the repo's folder structure exactly.
- `redux_slices` only if this feature genuinely adds/modifies state.
- `standards_summary` must be drawn from STEP 1 — never generic defaults.
- `figma_design_tokens` populated only when Figma MCP was used.
- If on a fix cycle, `fix_instructions` must include the exact BLOCK findings
  from the verifier reports, translated into actionable coder instructions.

---

### STEP 5 — Delegate to coder

Hand the plan payload to the **`frontend-coder-agent`** skill.

Communicate clearly:
- "Here is the plan payload. Implement all components_to_build in order.
  Follow standards_summary strictly. Report back with the code report payload."

Wait for the code report before continuing.

---

### STEP 6 — Route to verifiers

On receiving the code report from the coder:

1. Summarize files created/modified for the verifiers
2. Invoke **`frontend-logic-verifier`** with: code report + plan payload + files
3. Invoke **`frontend-visual-verifier`** with: code report + plan payload + dev server URL
4. Both verifiers can run in parallel if subagents are available

---

### STEP 7 — Evaluate and loop or finalize

Receive both verifier reports.

If both `verdict == "PASS"` or `"PARTIAL"` with no BLOCK findings:
→ Proceed to STEP 8 (summary)

If any BLOCK findings:
→ Check `fix_cycle` counter
→ If `< 2`: build fix delegation payload (schema section 5), return to STEP 5
→ If `>= 2`: stop and report to the human:
  ```
  ## Fix limit reached

  After 2 fix cycles, the following issues remain unresolved:
  [list BLOCK findings with file + line + message]

  Recommended next steps:
  [specific suggestions per finding]
  ```

---

### STEP 8 — Produce final summary

Follow the final summary format defined in
`../frontend-agent-team/SKILL.md` → STEP 4.

Include:
- One-paragraph narrative of what was built
- Complete file manifest
- Verified acceptance criteria checklist
- Any deviations from the original plan (with reasons)
- PR title + body (conventional commits format: `feat(scope): description`)