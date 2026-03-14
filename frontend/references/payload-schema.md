# Inter-agent payload schemas

All agents communicate via structured payloads. These are the canonical shapes.

---

## 1. Plan payload (Orchestrator → Coder)

```json
{
  "plan_id": "string (uuid)",
  "ticket_summary": "string (≤3 sentences)",
  "acceptance_criteria": ["string"],
  "components_to_build": [
    {
      "name": "ComponentName",
      "path": "src/components/path/ComponentName.tsx",
      "description": "what it renders",
      "props": ["prop: type"],
      "redux_slices": ["sliceName — what it manages"],
      "figma_node_id": "string | null"
    }
  ],
  "files_to_create": ["relative/path/to/file.ts"],
  "files_to_modify": ["relative/path/to/existing.ts"],
  "standards_summary": {
    "naming": "string (e.g. PascalCase components, camelCase hooks)",
    "imports": "string (e.g. absolute paths via @/)",
    "state_management": "string (e.g. Redux Toolkit, no direct store mutation)",
    "testing": "string (e.g. Vitest + RTL, co-located __tests__)"
  },
  "figma_design_tokens": {
    "colors": {},
    "spacing": {},
    "typography": {}
  },
  "fix_instructions": "string | null (populated on fix cycles)"
}
```

---

## 2. Code report payload (Coder → Orchestrator)

```json
{
  "plan_id": "string",
  "files_created": ["relative/path"],
  "files_modified": ["relative/path"],
  "open_questions": ["string"],
  "known_deviations": [
    { "file": "string", "reason": "string" }
  ],
  "test_files_created": ["relative/path"],
  "build_status": "clean | warnings | errors",
  "build_notes": "string | null"
}
```

---

## 3. Logic verifier report (LogicVerifier → Orchestrator)

```json
{
  "plan_id": "string",
  "verdict": "PASS | FAIL",
  "findings": [
    {
      "severity": "BLOCK | WARN | INFO",
      "file": "relative/path",
      "line": "number | null",
      "rule": "string (e.g. no-any, naming-convention)",
      "message": "string",
      "suggestion": "string | null"
    }
  ],
  "categories": {
    "type_safety": "PASS | FAIL",
    "naming_conventions": "PASS | FAIL",
    "patterns": "PASS | FAIL",
    "tests_present": "PASS | FAIL",
    "standards_compliance": "PASS | FAIL"
  }
}
```

---

## 4. Visual verifier report (VisualVerifier → Orchestrator)

```json
{
  "plan_id": "string",
  "verdict": "PASS | PARTIAL | FAIL",
  "preview_url": "string (localhost URL used)",
  "components_checked": [
    {
      "name": "string",
      "route_or_story": "string",
      "layout_match": "PASS | PARTIAL | FAIL",
      "spacing_match": "PASS | PARTIAL | FAIL",
      "color_match": "PASS | PARTIAL | FAIL",
      "notes": "string | null"
    }
  ],
  "screenshot_taken": "boolean",
  "manual_review_needed": "boolean",
  "manual_review_reason": "string | null"
}
```

---

## 5. Fix delegation payload (Orchestrator → Coder, fix cycle)

Same shape as Plan payload (#1) but with:
- `fix_instructions` populated with BLOCK findings from both verifiers
- `components_to_build` scoped only to failing components
- `fix_cycle: number` added at root