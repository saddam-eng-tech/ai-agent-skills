# Problem: Hardcoded Secrets in Code

**Confidence**: 95% | **Impact**: Critical | **Frequency**: Very High

## The Problem

Claude frequently generates code with embedded secrets:
- API keys hardcoded in strings
- Database passwords in configuration
- Auth tokens in source files

```javascript
// ❌ WRONG - What Claude generates
const apiKey = "sk-abc123def456";
const dbPassword = "admin123";
```

## Why It Happens

1. **Pattern Matching**: Claude sees placeholder examples and preserves them literally
2. **No Security Context**: Without explicit requirements, treats secrets like regular values
3. **Example Bias**: Follows patterns from seen code without sanitization

## The Solution

### Step 1: Give Explicit Security Requirements

```prompt
SECURITY REQUIREMENT: Never hardcode credentials.

Rules:
1. ALL secrets MUST come from environment variables
2. Use pattern: process.env.SECRET_NAME
3. Validate that env var exists at startup
4. Never log or print secrets
5. Document required env vars in comments
```

### Step 2: Run Verification Script

See `verify-no-secrets.js` in parent directory.

## Expected Result

✅ All credentials from environment variables  
✅ Code validates env vars exist at startup  
✅ No secrets in git history  
✅ Security review passes  

## References

- OWASP: Sensitive Data Exposure
- CWE-798: Use of Hard-Coded Credentials
- 12-Factor App: Store config in environment
