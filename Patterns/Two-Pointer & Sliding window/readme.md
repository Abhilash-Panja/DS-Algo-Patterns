# Two Pointers and Sliding Window



## Recommended order

1. [Introduction](1.00%20Introduction.md)
2. [Fixed-Size Sliding Window](1.01%20Pattern%201%20-%20Fixed-Size%20Sliding%20Window.md)
3. [Variable-Size Sliding Window](1.02%20Pattern%202%20-%20Variable-Size%20Sliding%20Window.md)
4. [Sorted Opposite-Direction Pointers](1.03%20Pattern%203%20-%20Sorted%20Opposite-Direction%20Pointers.md)
5. [Same-Direction Read-Write Pointers](1.04%20Pattern%204%20-%20Same-Direction%20Read-Write%20Pointers.md)
6. [Multi-Sequence Merge](1.05%20Pattern%205%20-%20Multi-Sequence%20Merge.md)
7. [Approaching an Unknown Problem](Approaching%20Unknown%20Problem.md)
8. [Pattern Decision Tree](Pattern%20Decision%20Tree.md)
9. [Pattern Summary](Summary%20of%20Existing%20Patterns.md)
10. [Mistake Library](Mistake%20Library.md)
11. [Revision Checklist](Revision%20Checklist.md)
12. [Unseen-Style Test](Unseen%20Test%20Problems.md)

## Source-note integration

The handwritten forms are mapped as follows:

- Form 0 -> fixed-size sliding window.
- Form 1 -> variable-size sliding window.
- Form 2 -> sorted Two Sum, 3Sum composition, and closest/optimization variants.
- Form 3 -> multi-sequence merge.

The notes also add same-direction read/write pointers because it is a major reusable two-pointer family that was missing from the handwritten outline.

## Technical boundaries emphasized

- A variable sliding window needs monotonic feasibility.
- Negative values can invalidate sum-based shrinking.
- Sorting is safe only when order and index requirements are preserved.
- Two sources can be merged with two pointers; k sources usually require a heap.
- Exact subarray sums with negative values belong to prefix sum plus hashmap, not a generic sliding window.

