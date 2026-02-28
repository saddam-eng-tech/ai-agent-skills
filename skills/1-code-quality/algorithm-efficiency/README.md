# Problem: Algorithm Inefficiency

**Confidence**: 92% | **Impact**: High | **Frequency**: Medium

## The Problem

Claude generates working but inefficient algorithms:
- O(n²) when O(n) is possible
- O(n log n) when O(n) is achievable
- Unnecessary nested loops

```javascript
// ❌ INEFFICIENT - O(n²)
function findDuplicates(arr) {
  const duplicates = [];
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j]) {
        duplicates.push(arr[i]);
      }
    }
  }
  return duplicates;
}

// ✅ EFFICIENT - O(n)
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = new Set();
  for (const num of arr) {
    if (seen.has(num)) duplicates.add(num);
    seen.add(num);
  }
  return [...duplicates];
}
```

## Why It Happens

1. **Pattern Following**: Generates obvious solution without optimization
2. **No Performance Requirement**: Without constraint, chooses quickest to write
3. **Context Missing**: May not understand data scale requirements

## The Solution

### Requirement Pattern

```prompt
PERFORMANCE REQUIREMENT:

1. Target time complexity: O(n log n) max for sorting, O(n) for single-pass
2. Target space complexity: O(n) maximum
3. Use appropriate data structures:
   - Set/Map for deduplication
   - Heap for priority operations
   - Trie for string prefixes
   - Hash tables for lookups
4. Explain the algorithm choice in comments
5. Include complexity comment: // Time: O(n), Space: O(1)

Before writing, explain the algorithm approach.
```

## Verification

- Include time/space complexity comments
- No nested loops (unless O(n log n) merge)
- Use efficient data structures
- Test with large inputs (1M+ items)

## Expected Result

✅ O(n) or O(n log n) algorithms  
✅ Efficient data structures used  
✅ Complexity clearly documented  
✅ 100x-1000x performance improvement  

## References

- Algorithm Design Manual (Skiena)
- Big O Notation Guide
- Data Structures Handbook
