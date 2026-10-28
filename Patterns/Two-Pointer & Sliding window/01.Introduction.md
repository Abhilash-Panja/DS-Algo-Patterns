# Two Pointers and Sliding Window - Introduction

*Practical interview notes: derive the invariant first, then choose the pointer movement.*

## 1. Practical One-Line Definition

> Two pointers and sliding windows are ways to maintain a useful boundary or contiguous region while moving through ordered data, so that work already done is reused instead of recomputed.

They are not one algorithm. They are a family of state-maintenance patterns:

- a fixed window keeps its length constant;
- a variable window changes its length to preserve a feasibility condition;
- opposite-direction pointers use sorted order to discard impossible pairs or boundaries;
- same-direction pointers separate a processed prefix from unread input;
- multi-sequence pointers advance the source that contributes the next frontier element.

## 2. Why These Patterns Exist

Many naive solutions repeatedly examine the same elements.

~~~text
Enumerate every subarray or pair
        ↓
Recompute its sum, count, or comparison
        ↓
O(n²) or O(n³) repeated work
        ↓
Identify an order and an invariant
        ↓
Move a boundary and update only what changed
~~~

The optimization is safe only when pointer movement is justified. The question is never merely "I see a subarray, should I use sliding window?" The real question is:

> If I move this boundary, can I prove that the discarded region can no longer produce a better or valid answer?

## 3. What Capability Does This Family Give Me?

~~~text
Fixed window
→ maintain an aggregate for every contiguous region of length k

Variable window
→ maintain the longest, shortest, or count of feasible contiguous regions

Sorted opposite pointers
→ eliminate a whole set of impossible pairs after one comparison

Same-direction pointers
→ compact, partition, or filter an array in one pass and O(1) extra space

Multi-sequence pointers
→ merge sorted sources without repeatedly rescanning them
~~~

## 4. The Core Vocabulary

Use half-open intervals unless a problem strongly suggests otherwise:

~~~text
[left, right) contains indices left through right - 1
window length = right - left
~~~

This convention makes the common operations explicit:

~~~java
add(nums[right]);
right++;

remove(nums[left]);
left++;
~~~

If you prefer inclusive right, that is fine, but state the convention before coding. Most off-by-one errors come from silently mixing the two.

## 5. The Five Questions Before Coding

1. Is the region required to be contiguous, or can elements be chosen independently?
2. Is the window length fixed, or does feasibility determine it?
3. What state must be available for the next move: sum, frequency, distinct count, maximum, or a pointer per source?
4. What property makes a pointer movement safe: positivity, sorted order, monotonic feasibility, or already-processed order?
5. What must remain true after every iteration?

## 6. Running Sum, Prefix Sum, and Hashmap: Do Not Merge Them

### Running sum

Use it when the current region changes by adding one boundary element and removing one boundary element.

~~~text
new sum = old sum + entering value - leaving value
~~~

This is the natural state for a fixed-size window and many positive-number variable windows.

### Prefix sum

Use it when you need arbitrary range sums or when the window cannot be maintained by safe local shrinking.

~~~text
sum(left, right) = prefix[right + 1] - prefix[left]
~~~

### Prefix sum plus hashmap

Use it for exact subarray-sum questions when negative numbers may appear. Negative values break the monotonicity required by many variable-window solutions.

## 7. Constraints as a Filter, Not a Formula

If n is around 10^5, an O(n²) algorithm is usually too slow. That does not mean every answer must be O(n log n) or O(n). First identify the structure:

~~~text
sorting + linear scan       → O(n log n)
one pass with moving ends   → O(n)
hashmap lookups             → O(n) expected
binary search on answer      → O(n log answer-range)
k-way frontier with heap     → O(N log k)
~~~

## 8. Pattern Map

| Pattern | State | Safe movement | Typical complexity |
| --- | --- | --- | --- |
| Fixed-size window | aggregate for exactly k items | remove left, add right | O(n) |
| Variable-size window | aggregate plus feasibility state | expand, then shrink or record | usually O(n) when monotone |
| Sorted opposite pointers | values at two ends | move the side that makes the objective possible | O(n) after sorting |
| Same-direction read/write | processed output prefix | write only accepted values | O(n), often O(1) space |
| Multi-sequence merge | current frontier of each source | advance the source that supplied the next item | O(n + m) for two sources |

## 9. Important Boundaries

Sliding window is appropriate when the input is contiguous and the validity condition changes predictably as boundaries move. It is not a synonym for every subarray problem.

Two pointers are appropriate when pointer movement removes a provably impossible region. They are not a magic replacement for nested loops.

Sorting can create the monotonic structure needed for two pointers, but it can also destroy required original order. Preserve original indices when the output depends on them.

## 10. Java Safety Checklist

- Use long for sums or products that may overflow int.
- Decide whether equal values should move, be skipped, or be counted once.
- State whether right is inclusive or exclusive.
- For a variable window, define exactly what makes the window valid.
- For 3Sum, skip duplicates at both the fixed index and the two-pointer stage.
- For merging into an existing array, fill from the back if writing can overwrite unread values.

## 11. One Sentence to Remember

> Do not move a pointer because a template says so; move it because the invariant proves that the skipped possibilities cannot improve the answer.

