# Prompt Pattern: Constraint-First

**When to Use**: Security-critical code, compliance requirements

## Pattern

```
1. CONSTRAINTS FIRST (What you cannot do)
2. THEN requirements (What you must do)
3. THEN examples
4. THEN verification
```

## Example

```prompt
CONSTRAINTS:
❌ No hardcoded secrets
❌ No string concatenation in SQL
❌ No unhandled async errors
❌ No console.log in production code

REQUIREMENTS:
✅ All credentials from environment
✅ Parameterized SQL queries
✅ Complete error handling
✅ Use logger.info instead

EXAMPLE:
[show good example]

VERIFICATION:
[show how to test it]
```

## Why It Works

1. Negative constraints are clearer than positive ones
2. Prevents common mistakes upfront
3. No ambiguity about what's forbidden
4. Easy to verify compliance

## Best For

✅ Security requirements  
✅ Compliance code  
✅ Production changes  
✅ Risk-critical features  
