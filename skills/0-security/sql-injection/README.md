# Problem: SQL Injection Vulnerabilities

**Confidence**: 95% | **Impact**: Critical | **Frequency**: Very High

## The Problem

Claude generates string concatenation instead of parameterized queries:

```javascript
// ❌ WRONG - String concatenation
const userInput = req.query.id;
const query = "SELECT * FROM users WHERE id = " + userInput;

// ❌ WRONG - Template literals (not parameterized)
const sql = `SELECT * FROM users WHERE email = '${email}'`;
```

## Why It Happens

1. **Simplicity Pattern**: Concatenation looks simpler syntactically
2. **No Security Constraint**: Without specific requirement, uses easier approach
3. **Context Loss**: May forget requirement in longer conversations

## The Solution

### Requirement Pattern

```prompt
SECURITY REQUIREMENT - SQL Injection Prevention:

1. NEVER concatenate user input into SQL queries
2. NEVER use template literals for SQL
3. ALWAYS use parameterized queries

For this database (PostgreSQL/MySQL):
- PostgreSQL: Use $1, $2 placeholders
- MySQL: Use ? placeholders
- Pass values separately: db.query(sql, [value1, value2])

Example:
const query = 'SELECT * FROM users WHERE id = $1';
const result = await db.query(query, [userId]);
```

### Verification Script

See `verify-sql-safety.js` in parent directory.

## Expected Result

✅ All queries use parameterized syntax  
✅ User input never concatenated into SQL  
✅ Template literals never used for SQL  
✅ Security audit passes  

## References

- OWASP SQL Injection
- CWE-89: SQL Injection
- PostgreSQL Security (Prepared Statements)
