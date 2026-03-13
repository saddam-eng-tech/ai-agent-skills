---
name: token-cost-estimator
description: Calculates token usage and estimated API costs for Claude tasks by analysing prompt structure, context size, and expected output length — then checks against context window limits and suggests optimisations to reduce cost. Use whenever a user says "how much will this cost", "estimate tokens for my prompt", "is this too long for the context window", "optimise my prompt for cost", or "how do I reduce Claude API costs".
---

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

## Step-by-step workflow

### STEP 1 — Identify the target model and input

Ask (or infer from context):
- Which Claude model? (claude-3-5-sonnet, claude-3-opus, claude-3-haiku, etc.)
- Is this a single prompt or a multi-turn workflow?
- What is the expected output length? (short reply / paragraph / full document)

If model is not specified, default to claude-3-5-sonnet-20241022.

---

### STEP 2 — Count input tokens

Break the prompt into components and estimate tokens for each:

| Component | Token estimate rule |
|-----------|-------------------|
| System prompt | Count words × 1.3 |
| User message | Count words × 1.3 |
| Conversation history | Sum of all previous turns |
| Tool definitions | ~200–500 tokens per tool |
| File/document attachments | Characters ÷ 4 |
| Images | 1000–2000 tokens per image (model-dependent) |

Total input tokens = sum of all components.

---

### STEP 3 — Estimate output tokens

- Short reply (1–3 sentences): ~50–150 tokens
- Paragraph response: ~150–400 tokens
- Structured output (JSON, list): ~200–800 tokens
- Full document/code file: ~500–4000 tokens
- Use user's description or previous outputs to calibrate.

---

### STEP 4 — Check context window limits

| Model | Context window |
|-------|---------------|
| claude-3-5-sonnet | 200,000 tokens |
| claude-3-opus | 200,000 tokens |
| claude-3-haiku | 200,000 tokens |
| claude-3-5-haiku | 200,000 tokens |

Flag if: input + expected output > 80% of context window.
Flag as critical if: input alone > 90% of context window.

---

### STEP 5 — Calculate cost

Use current Anthropic pricing (as of 2025):

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|-----------------------|------------------------|
| claude-3-5-sonnet | $3.00 | $15.00 |
| claude-3-opus | $15.00 | $75.00 |
| claude-3-haiku | $0.25 | $1.25 |
| claude-3-5-haiku | $0.80 | $4.00 |

Cost = (input_tokens / 1,000,000 × input_price) + (output_tokens / 1,000,000 × output_price)

For workflows with multiple calls, multiply by expected call count.

---

### STEP 6 — Suggest optimisations

Rank suggestions by savings potential:

1. **Prompt caching** — If system prompt > 1024 tokens and reused across calls, enable prompt caching (90% cost reduction on cached portion)
2. **Model downgrade** — If task doesn't require reasoning, suggest haiku (10–40× cheaper)
3. **Prompt compression** — Identify verbose sections that can be shortened without losing meaning
4. **Output length control** — Add explicit instructions to limit response length
5. **Chunking** — Break large documents into smaller calls instead of one massive context

---

### STEP 7 — Present the report

```
## Token Cost Estimate

**Model**: claude-3-5-sonnet
**Scenario**: [description]

### Token Breakdown
| Component | Tokens |
|-----------|--------|
| System prompt | ~450 |
| User message | ~280 |
| History (3 turns) | ~1,200 |
| **Total input** | **~1,930** |
| Expected output | ~400 |
| **Total per call** | **~2,330** |

### Cost Per Call
- Input: 1,930 tokens × $3.00/1M = $0.0058
- Output: 400 tokens × $15.00/1M = $0.0060
- **Total: ~$0.012 per call**

### At Scale
- 100 calls/day: ~$1.16/day / ~$35/month
- 1,000 calls/day: ~$11.60/day / ~$350/month

### Context Window
✅ 2,330 / 200,000 tokens (1.2%) — well within limits

### Top Optimisations
1. 💰 Enable prompt caching on system prompt → save ~$0.004/call (40%)
2. 🔄 Switch to claude-3-haiku for this task → save ~$0.010/call (85%)
3. ✂️ Compress history summarisation → reduce input by ~400 tokens
```
