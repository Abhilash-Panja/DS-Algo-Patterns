# Priority Queue — Quick Notes (DSA Problem-Solving Lens)

*Not an exam-theory page. This is the "what do I actually type and when do I reach for it" cheat sheet.*

---

## 1. One-Line Definition (Practical, Not Textbook)

> A Priority Queue is a box where you keep throwing numbers in, and no matter what order you threw them in, **`peek()`/`poll()` always hands you back the smallest (or largest) one instantly.**

That's it. Forget "abstract data type" definitions — in problem-solving, a PQ is just **"a live-updating min/max that survives insertions and deletions."**

---

## 2. When Should You Even Think of Using a PQ?

Scan the problem for these **trigger words / situations** — if you spot any, a heap is a strong candidate:

| Signal in the problem | Why PQ fits |
|---|---|
| "k largest / k smallest / k closest" | Heap keeps top-k without full sort |
| "kth largest element" (in a stream) | Heap updates in O(log k) per insert |
| "merge k sorted lists/arrays" | Heap picks the next smallest across k pointers |
| "schedule tasks / meetings / CPU jobs" | Heap tracks "what finishes/starts next" |
| "running median" | Two heaps (max-heap + min-heap) |
| "minimum cost to connect / Dijkstra / Prim's" | Heap picks cheapest next edge/node |
| "at most k things, maximize/minimize a sum" | Heap maintains best k while sum updates |
| "repeatedly pick the max/min and do something to it" | Heap resimulates the "current max/min" after every operation |

**If none of these ring a bell but you keep needing "give me the current best element" over and over as data changes → still use a heap.**

---

## 3. Java Syntax — Copy-Paste Cheat Sheet

### Declare

```java
// Min-heap (default) — smallest at top
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max-heap — largest at top
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
// same thing, written explicitly:
PriorityQueue<Integer> maxHeap2 = new PriorityQueue<>((a, b) -> b - a);
```

### Core operations (memorize these 4)

```java
pq.offer(x);      // insert x            -> O(log n)
pq.poll();         // remove & return top -> O(log n)
pq.peek();          // just look at top    -> O(1)
pq.size();           // how many elements   -> O(1)
```

### Custom objects (the one you'll use 90% of the time in interviews)

```java
// Example: min-heap of {value, index} pairs, sorted by value
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
pq.offer(new int[]{5, 0});
pq.offer(new int[]{2, 1});
int[] top = pq.poll(); // {2, 1}
```

```java
// Example: heap of custom class objects
PriorityQueue<Task> pq = new PriorityQueue<>((a, b) -> a.deadline - b.deadline);
```

### Two-heap pattern (running median, balancing problems)

```java
PriorityQueue<Integer> lowerHalf = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
PriorityQueue<Integer> upperHalf = new PriorityQueue<>(); // min-heap
// keep lowerHalf.size() == upperHalf.size() or +1, rebalance after every insert
```

---

## 4. Time Complexity Cheat Sheet

| Operation | Complexity |
|---|---|
| Insert (`offer`) | O(log n) |
| Remove top (`poll`) | O(log n) |
| Peek top (`peek`) | O(1) |
| Build heap from array (`new PriorityQueue<>(list)`) | O(n) |
| Search for arbitrary element | O(n) — heaps are **not** good for this |

**Rule of thumb:** if the problem needs `n` insertions with heap bounded at size `k`, total cost is **O(n log k)** — much better than sorting everything (`O(n log n)`) when `k << n`.

---

## 5. The 4 Implementation Templates You'll Reuse Forever

### Template A — "Top K" (bounded-size heap)
```java
PriorityQueue<Integer> pq = new PriorityQueue<>(); // min-heap
for (int num : nums) {
    pq.offer(num);
    if (pq.size() > k) pq.poll(); // evict smallest, keep k largest
}
// pq now holds the k largest elements; pq.peek() = kth largest
```
**Used in:** Kth Largest Element in a Stream, K Closest Points to Origin, Top K Frequent Elements.

### Template B — "Sort + Heap + Running Sum" (the pattern from the main post)
```java
Arrays.sort(items, (a, b) -> b[0] - a[0]); // fix one attribute
PriorityQueue<Integer> pq = new PriorityQueue<>();
long runningSum = 0, best = 0;
for (int[] item : items) {
    pq.offer(item[1]); runningSum += item[1];
    if (pq.size() > k) runningSum -= pq.poll();
    best = Math.max(best, runningSum * item[0]);
}
```
**Used in:** Maximum Performance of a Team, Maximum Subsequence Score, Minimum Cost to Hire K Workers.

### Template C — "Merge K Sorted Sources"
```java
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]); // {value, sourceIndex, elementIndex}
// seed with the first element of each source, then repeatedly poll + push the "next" from that source
```
**Used in:** Merge k Sorted Lists, Smallest Range Covering Elements from k Lists, Kth Smallest Element in a Sorted Matrix.

### Template D — "Simulate & Re-Push" (repeatedly act on the current extreme)
```java
PriorityQueue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
while (pq.size() > 1) {
    int a = pq.poll(), b = pq.poll();
    // do something with a, b (combine, reduce, etc.)
    if (/* result should go back in */) pq.offer(result);
}
```
**Used in:** Last Stone Weight, Minimize Deviation in Array, Huffman-style merging problems.

---

## 6. Quick Decision Flow (Read Top to Bottom, Stop at First Match)

1. **Need the current min/max of a set that keeps changing?** → heap.
2. **Need only the top/bottom `k` out of `n`, and `k` is much smaller than `n`?** → Template A (bounded heap), don't fully sort.
3. **Objective mixes a sum with a min/max, and you can only pick `k` items?** → Template B (sort to pin one side, heap on the other).
4. **Need to merge/interleave multiple sorted streams into one?** → Template C.
5. **Need to repeatedly grab the current biggest/smallest and transform it?** → Template D.
6. **Need the median of a live stream?** → Two-heap pattern (Section 3).
7. **None of the above, but you keep re-scanning for min/max after every update?** → still heap; you're just inventing a new template.

---

## 7. Gotchas Beginners Hit Every Single Time

- `peek()` does **not** remove the element — only `poll()` does. Confusing these silently breaks your loop.
- Default `PriorityQueue<>()` in Java is a **min-heap**. If your problem wants "largest first," you forgot the comparator — this is the #1 wrong-answer cause.
- Java's `PriorityQueue` is **not thread-safe** and does **not** support `O(1)` arbitrary removal — `pq.remove(x)` is O(n). If you need fast arbitrary deletion, you need a heap + hashmap ("indexed heap") — a more advanced structure, not covered here.
- Iterating a `PriorityQueue` directly (`for (int x : pq)`) does **not** give you sorted order — only `poll()` repeatedly does. This trips people up in debugging.
- Overflow: if you're summing large values, use `long` for the running sum, not `int` (classic silent bug in Template B).

---

## 8. The One Sentence to Remember

> **"Whenever a problem forces you to keep asking 'what's currently the best/worst element in a set that keeps changing,' stop thinking and reach for a heap — then decide if you also need a sort step (Template B), multiple sources (Template C), or repeated re-insertion (Template D)."**