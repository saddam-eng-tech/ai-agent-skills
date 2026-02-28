# Problem: Unsafe Debugging & Recovery

**Confidence**: 88% | **Impact**: Medium | **Frequency**: Medium

## The Problem

When Claude breaks code, it's unclear how to fix it safely:
- Complete rewrites instead of minimal fixes
- Introducing new bugs while fixing old ones
- No systematic debugging approach

## The Solution

### Three-Step Isolation Technique

**Step 1: Isolate the Problem**
```prompt
1. Identify the exact error/bug
2. Find the minimum code that reproduces it
3. Create a small test case that fails
```

**Step 2: Minimal Diff Fix**
```prompt
When fixing code:
1. Change ONLY the broken part
2. Keep all else identical
3. Show exact line numbers
4. Test immediately after each change
```

**Step 3: Verify Fix**
```prompt
1. Run original test that was failing
2. Run all other tests
3. No new failures introduced
4. Original error gone
```

## Safe Debugging Workflow

1. **Characterize**: What exactly is broken?
2. **Isolate**: Minimal failing test case
3. **Fix**: Smallest possible change
4. **Verify**: Both old and new tests pass

## Expected Result

✅ Bugs fixed without new ones  
✅ Minimal code changes  
✅ All tests remain passing  
✅ Clear what changed and why  

## References

- Programming Pearls (Bentley)
- Debug It (Paul Butcher)
