---
name: llm-eval-scaffolder
description: Builds LLM evaluation pipelines with structured test cases, LLM-as-judge prompts, automated scoring scripts, and CI integration to catch prompt regressions before they reach production. Use whenever a user says "set up LLM evals", "test my prompts", "measure prompt quality", "I need an eval pipeline", "LLM-as-judge setup", or "how do I know my prompts are getting worse".
---

# LLM Eval Scaffolder

Generates a complete evaluation pipeline: curated test cases per task category, an LLM-as-judge prompt with scoring rubric, a scoring runner script, and a CI workflow that fails on regression — so prompt changes are measurable before shipping.

**Input**: Description of the LLM task + examples of good/bad outputs
**Output**: Test case dataset + judge prompt + scoring script + CI config

## Trigger phrases
- "set up LLM evals"
- "LLM-as-judge"
- "test my prompts"
- "measure prompt quality"
- "eval pipeline"
- "catch prompt regressions"

## Step-by-step workflow

### STEP 1 — Understand the task

Ask (or infer):
- What does the LLM do? (summarise, classify, extract, generate, QA, etc.)
- What does a good output look like? (ask for 2–3 examples)
- What does a bad output look like? (ask for 1–2 failure examples)
- What dimensions matter? (accuracy, tone, format, completeness, safety)

---

### STEP 2 — Generate test cases

Create a `evals/test_cases.jsonl` file with at least 20 test cases covering:

| Category | Count | Description |
|----------|-------|-------------|
| Golden cases | 5 | Clearly correct, expected to pass |
| Edge cases | 5 | Tricky inputs that reveal model weaknesses |
| Adversarial | 3 | Inputs designed to break the prompt |
| Regression | 4 | Real past failures to prevent re-occurring |
| Boundary | 3 | Empty input, very long input, unusual format |

Format:
```jsonl
{"id": "tc-001", "input": "...", "expected": "...", "category": "golden", "notes": ""}
```

---

### STEP 3 — Write the LLM-as-judge prompt

Create `evals/judge_prompt.md`:

```markdown
# Evaluation Judge Prompt

You are evaluating an AI assistant's response to a task.

## Task description
[What the AI is supposed to do]

## Input
{{input}}

## AI Response
{{response}}

## Expected output / criteria
{{expected}}

## Scoring rubric
Score from 1–5 on each dimension:

| Dimension | 1 (Poor) | 3 (Acceptable) | 5 (Excellent) |
|-----------|----------|----------------|---------------|
| Accuracy | Wrong facts | Mostly correct | Fully correct |
| Completeness | Missing key info | Covers main points | Covers everything |
| Format | Wrong structure | Mostly right format | Perfect format |
| [Custom dimension] | ... | ... | ... |

## Output format (JSON only)
```json
{
  "scores": {"accuracy": N, "completeness": N, "format": N},
  "overall": N,
  "pass": true/false,
  "reason": "one sentence explanation"
}
```
```

Pass threshold: overall ≥ 3.5

---

### STEP 4 — Write the scoring runner

Create `evals/run_evals.py`:

```python
import json
import anthropic
from pathlib import Path

client = anthropic.Anthropic()

def run_target_prompt(input_text: str) -> str:
    """Run the prompt under test."""
    response = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[{"role": "user", "content": input_text}]
    )
    return response.content[0].text

def judge_response(input_text: str, response: str, expected: str) -> dict:
    """Use LLM-as-judge to score the response."""
    judge_prompt = Path("evals/judge_prompt.md").read_text()
    filled = judge_prompt.replace("{{input}}", input_text)\
                         .replace("{{response}}", response)\
                         .replace("{{expected}}", expected)
    
    result = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=512,
        messages=[{"role": "user", "content": filled}]
    )
    return json.loads(result.content[0].text)

def main():
    test_cases = [json.loads(l) for l in Path("evals/test_cases.jsonl").read_text().splitlines()]
    results = []
    
    for tc in test_cases:
        response = run_target_prompt(tc["input"])
        judgment = judge_response(tc["input"], response, tc["expected"])
        results.append({"id": tc["id"], "category": tc["category"], **judgment})
        print(f"{tc['id']}: {'PASS' if judgment['pass'] else 'FAIL'} (score: {judgment['overall']})")
    
    passed = sum(1 for r in results if r["pass"])
    print(f"\nResults: {passed}/{len(results)} passed")
    
    # Write results
    Path("evals/results.json").write_text(json.dumps(results, indent=2))
    
    # Fail CI if pass rate < threshold
    pass_rate = passed / len(results)
    if pass_rate < 0.80:  # 80% pass threshold
        raise SystemExit(f"Eval failed: {pass_rate:.0%} pass rate (threshold: 80%)")

if __name__ == "__main__":
    main()
```

---

### STEP 5 — Write CI workflow

Create `.github/workflows/evals.yml`:

```yaml
name: LLM Evals

on:
  pull_request:
    paths:
      - 'prompts/**'
      - 'evals/**'

jobs:
  run-evals:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      
      - name: Install dependencies
        run: pip install anthropic
      
      - name: Run evals
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: python evals/run_evals.py
      
      - name: Upload results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: eval-results
          path: evals/results.json
```

---

### STEP 6 — Output all files

Write:
- `evals/test_cases.jsonl` — 20+ test cases
- `evals/judge_prompt.md` — scoring rubric
- `evals/run_evals.py` — runner script
- `.github/workflows/evals.yml` — CI workflow
- `evals/README.md` — how to run locally and interpret results

## Output format spec

- `test_cases.jsonl`: one JSON object per line
- `results.json`: array of judgment objects written by runner
- CI fails if pass rate < 80% (configurable)
- Judge scores on 1–5 scale per dimension

## Quality checklist

- [ ] At least 20 test cases generated
- [ ] All 5 categories represented (golden, edge, adversarial, regression, boundary)
- [ ] Judge prompt has explicit pass/fail threshold
- [ ] Runner script handles API errors gracefully
- [ ] CI workflow triggers on prompt file changes
- [ ] README explains how to add new test cases

## Edge cases

- **Non-deterministic outputs**: Use semantic scoring dimensions instead of exact match; the judge handles variation.
- **Very long outputs**: Truncate to 2000 chars before sending to judge to control judge costs.
- **Task with no clear "expected"**: Use criteria-based judging only (no `expected` field); adjust rubric accordingly.
- **User wants cheaper evals**: Suggest using claude-3-haiku as the judge model (90% cheaper, adequate for most rubrics).
