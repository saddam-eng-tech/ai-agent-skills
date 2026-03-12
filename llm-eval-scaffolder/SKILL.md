name: llm-eval-scaffolder
description: Builds LLM evaluation pipelines with structured test cases, LLM-as-judge prompts, automated scoring scripts, and CI integration to catch prompt regressions before they reach production. Use whenever a user says "set up LLM evals", "test my prompts", "measure prompt quality", "I need an eval pipeline", "LLM-as-judge setup", or "how do I know my prompts are getting worse".

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

## Source
`SKILL-48.md`