name: token-cost-estimator
description: Calculates token usage and estimated API costs for Claude tasks by analysing prompt structure, context size, and expected output length — then checks against context window limits and suggests optimisations to reduce cost. Use whenever a user says "how much will this cost", "estimate tokens for my prompt", "is this too long for the context window", "optimise my prompt for cost", or "how do I reduce Claude API costs".

# Token Cost Estimator

Breaks down a prompt or workflow into its token components, calculates expected cost against current Claude pricing, flags context window risks, and proposes concrete optimisations (caching, prompt compression, chunking) ranked by savings potential.

**Input**: Prompt text or workflow description + target Claude model
**Output**: Token breakdown + cost estimate + context window check + optimisation suggestions

## Trigger phrases
- "estimate token cost"
- "how much will this cost"
- "context window too large"
- "reduce Claude API cost"
- "prompt token count"
- "optimise for cost"

## Source
`SKILL-49.md`