# Problem: Test Hallucinations (Fake Tests)

**Confidence**: 84% | **Impact**: High | **Frequency**: High

## The Problem

When using TDD with Claude, it generates tests that look real but don't validate implementation:

```javascript
// ❌ FAKE TEST - Uses mocks, doesn't test real behavior
describe('calculateTotal', () => {
  it('should sum numbers correctly', () => {
    const mockAdd = jest.fn().mockReturnValue(10);
    const result = mockAdd(5, 5);
    expect(result).toBe(10); // Tests mock, not function!
  });
});

// ✅ REAL TEST - Tests actual implementation
describe('calculateTotal', () => {
  it('should sum numbers correctly', () => {
    const result = calculateTotal([5, 3, 2]);
    expect(result).toBe(10); // Tests real function!
  });
});
```

## Why It Happens

1. **Speed Optimization**: Mocks are faster to write than real implementation
2. **TDD Misunderstanding**: Claude interprets TDD as "write tests first with mocks"
3. **Pattern Confusion**: Confuses mock testing with real testing

## The Solution

### TDD Framework Requirement

```prompt
TEST-DRIVEN DEVELOPMENT REQUIREMENT:

1. Write REAL tests first (no mocks for happy path)
2. Tests must validate ACTUAL function behavior
3. Use mocks ONLY for external dependencies:
   - Database calls
   - HTTP requests
   - File system
   - NOT for testing the function itself
4. Happy path: 100% real test
5. Error path: Can use mocks for dependencies

Example:
```

### Test Validation Checklist

- [ ] No jest.fn() for functions being tested
- [ ] No mockReturnValue for core logic
- [ ] Tests call actual functions
- [ ] Tests fail when implementation is wrong
- [ ] All assertions test real behavior

## Expected Result

✅ Real tests validate implementation  
✅ Tests fail when code breaks  
✅ Mocks used only for external dependencies  
✅ Test suite catches bugs  

## References

- TDD: Test-Driven Development (Kent Beck)
- Mock Objects Anti-patterns
- Real Unit Testing (Osherove)
