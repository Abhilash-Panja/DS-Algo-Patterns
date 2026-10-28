# Two Pointers and Sliding Window



## Recommended order

1. [Introduction](./01.Introduction.md)
2. [Fixed-Size Sliding Window](./02.Fixed-Size%20Sliding%20Window.md)
3. [Variable-Size Sliding Window](./03.Variable-Size%20Sliding%20Window.md)
4. [Sorted Opposite-Direction Pointers](./04.Sorted%20Opposite-Direction%20Pointers.md)
5. [Same-Direction Read-Write Pointers](./05.Same-Direction%20Read-Write%20Pointers.md)
6. [Multi-Sequence Merge](./06.Multi-Sequence%20Merge.md)
7. [Approaching an Unknown Problem](./07.Approaching%20an%20Unknown%20Two-Pointer%20or%20Sliding-Window%20Problem.md)
8. [Pattern Decision Tree](./08.Pattern%20Decision%20Tree.md)
9. [Pattern Summary](./09.Two%20Pointers%20and%20Sliding%20Window%20-%20Pattern%20Summary.md)
10. [Mistake Library](./10.%20Mistake%20Library%20-%20Two%20Pointers%20and%20Sliding%20Window.md)
11. [Revision Checklist](./11.Revision%20Checklist%20-%20Two%20Pointers%20and%20Sliding%20Window.md)
12. [Unseen-Style Test](./12.Unseen-Style%20Test%20-%20Two%20Pointers%20and%20Sliding%20Window.md)

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

