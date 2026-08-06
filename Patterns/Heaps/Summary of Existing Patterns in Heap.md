# Heap / Priority Queue — All Patterns, One by One (Muscle Memory Roadmap)

*Goal: don't just solve problems — solve enough problems **per pattern** that you recognize the pattern in under 30 seconds next time. Go in order. Don't skip to Pattern 5 before Pattern 1 feels automatic.*

---

## How to Use This Roadmap

For each pattern below:
1. Read the **"Recognize it when..."** line — this is the trigger you're training your brain to detect.
2. Read the **core idea** — one paragraph, no code.
3. Solve the problems **in order** (easy → hard) without looking at solutions first.
4. After each problem, ask: *"Could I have written the first 5 lines of this solution within 60 seconds of reading it?"* If no, redo a similar problem before moving to the next pattern.

---

## Pattern 1 — Top-K / Kth Element (Bounded Heap)

**Recognize it when...** the problem says "k largest," "k smallest," "k closest," "kth element," especially with a **live/growing stream** of data.

**Core idea:** Keep a heap capped at size `k`. For "k largest," use a **min-heap** — the smallest of your current top-k sits at the top, ready to be evicted the moment something bigger shows up. For "k smallest," flip to a **max-heap**. This avoids sorting the whole dataset (`O(n log n)`) when you only ever need `O(n log k)`.

**Practice, in order:**
1. **Kth Largest Element in an Array** — static version, the "hello world" of this pattern.
2. **Kth Largest Element in a Stream** — same idea, but the heap must persist across repeated `add()` calls. This is where the pattern actually clicks.
3. **K Closest Points to Origin** — distance is your comparator instead of raw value; trains you to heap on a *derived* key.
4. **Top K Frequent Elements** — you must first build a frequency map, *then* apply the top-k heap on frequencies. Two-step composition.
5. **Kth Smallest Element in a Sorted Matrix** — looks like Pattern 3 (merge) but is solvable cleanly with a bounded max-heap of size `k` too — good for seeing the overlap between patterns.
6. **Find K Pairs with Smallest Sums** — combine top-k with generating candidates on the fly instead of them being given upfront.

---

## Pattern 2 — K-Way Merge

**Recognize it when...** you have **multiple sorted sources** (lists, arrays, streams) and need to combine them into one sorted output, or find something *across* all of them (smallest range, kth smallest overall).

**Core idea:** Push the first element of every source into a heap as `{value, sourceId, indexInSource}`. Poll the smallest, emit it, then push the *next* element from that same source. Repeat. The heap never holds more than one element per source at a time, so it stays small even if the total data is huge.

**Practice, in order:**
1. **Merge k Sorted Lists** — the canonical version; linked lists as the "sources."
2. **Kth Smallest Element in a Sorted Matrix** *(revisit)* — solve it this way too: each row is a sorted source.
3. **Smallest Range Covering Elements from K Lists** — harder variant; you must track the *current max* across all heap tops alongside the min, to know your current range width.
4. **Merge k Sorted Arrays** (general version, not just linked lists) — same core idea, different container.
5. **Find K Pairs with Smallest Sums** *(revisit)* — this is really a k-way merge over "virtual" sorted sequences generated from two arrays.

---

## Pattern 3 — Two Heaps (Balance / Running Median)

**Recognize it when...** you need to track the **median**, or generally need to keep a data stream split into a "lower half" and "upper half" that stay balanced as new elements arrive.

**Core idea:** Maintain a **max-heap for the lower half** and a **min-heap for the upper half**. Every new element goes into one side, then you rebalance so their sizes differ by at most 1. The median is always at the top of one (or both) heaps — no scanning required.

**Practice, in order:**
1. **Find Median from Data Stream** — the foundational problem; master this before anything else in this pattern.
2. **Sliding Window Median** — same two-heap idea, but now you also need to *remove* elements as the window slides, which forces you to learn lazy deletion.
3. **IPO** *(cross-pattern, but instructive)* — technically single-heap, but often confused with two-heap problems; worth comparing side-by-side to sharpen the distinction.
4. **Maximize Capital / Schedule to Minimize... (median-based scheduling variants)** — search for problems combining median-balancing with a cost function; good stretch practice once the core two are solid.

---

## Pattern 4 — Sort + Heap + Running Aggregate (Greedy-with-Heap)

**Recognize it when...** the objective mixes an **aggregate** (sum, count, capital) with a **min/max**, and you can only pick "at most/exactly `k`" items, or need a floor/ceiling that changes as you scan.

**Core idea:** Sort by whichever attribute is hard to track incrementally (usually the min/max side) — this "pins" that value for everything processed so far. Then use a heap on the *other* attribute to always keep the best `k` and maintain a running aggregate as you evict/admit elements. *(This is the pattern from the main deep-dive post — revisit that if any step feels fuzzy.)*

**Practice, in order:**
1. **Maximum Subsequence Score** — cleanest, most direct version of this pattern; start here.
2. **Maximum Performance of a Team** — the canonical, slightly harder version (the one we fully derived earlier).
3. **Minimum Cost to Hire K Workers** — same skeleton, but minimizing, so the heap flips to a max-heap.
4. **Course Schedule III** — no explicit `k`; eviction is triggered by a feasibility check instead of a fixed heap size.
5. **IPO** — the running aggregate is your unlocked capital, not a sum you ever evict from; a good test of whether you understand the *why*, not just the *code shape*.

---

## Pattern 5 — Greedy Simulation / Resource-Budget (Heap as a "Deferred Decision" Store)

**Recognize it when...** the problem describes a **simulation over time or distance** (a car running out of fuel, a building with limited ladders/bricks), and a "top-k so far" or "best choice available so far" decision has to be made *without* knowing the future.

**Core idea:** Push candidates into the heap **as you encounter them**, even before you need to use them. Only when you're forced to make a decision (run out of fuel, exceed your budget) do you pop the *best* deferred option from the heap. This "push now, decide later" habit is the key mental shift from Pattern 1.

**Practice, in order:**
1. **Furthest Building You Can Reach** — ladders/bricks; the resource-budget framing that hides the heap.
2. **Minimum Number of Refueling Stops** — the clearest "push now, use later when stuck" example.
3. **Car Pooling / Task Scheduler variants** *(optional stretch)* — some variants of these use a heap for the same deferred-decision reasoning; good for testing transfer.

---

## Pattern 6 — Repeated Extremes / Simulate-and-Rebuild (Max-Heap Self-Reduction)

**Recognize it when...** the problem says things like "repeatedly take the largest element(s), combine or reduce them, put the result back, repeat until..."

**Core idea:** A single max-heap (or min-heap) that you **pull from and push back into repeatedly**, simulating a process step-by-step until some stopping condition. Unlike Pattern 1 (fixed size `k`), the heap size actively shrinks/grows during the simulation itself.

**Practice, in order:**
1. **Last Stone Weight** — simplest version: pop two largest, push back the difference, repeat.
2. **Remove Stones to Minimize the Total** — pop the largest, halve it, push back, repeat `k` times.
3. **Minimize Deviation in Array** — every element can be doubled or halved; you repeatedly shrink the current max while tracking `max - min` as the running objective.
4. **Reorganize String / Task Scheduler** — frequency-based max-heap where you repeatedly place the most frequent remaining character/task, a nice "greedy + heap" variant that also touches Pattern 7 below.

---

## Pattern 7 — Heap-Based Scheduling / Interval Problems (Often Two Heaps or Heap + Sort)

**Recognize it when...** the problem involves **meetings, tasks, servers, or CPUs** with start/end times or processing durations, and you need to know what's "currently busy" vs "currently free."

**Core idea:** Sort events by start time. Use one heap to track **currently occupied resources** (keyed by end time, so the earliest-finishing one is always on top) and often a second heap/structure for **currently free resources**. As you scan events in time order, first free up anything whose end time has passed, then assign/allocate from what's available.

**Practice, in order:**
1. **Meeting Rooms II** — minimum number of rooms needed; single heap of end times.
2. **Single-Threaded CPU** — sort by enqueue time, heap keyed by processing time, simulated clock instead of a running sum.
3. **Meeting Rooms III** — two heaps (available rooms + occupied rooms with end times); harder resource-tracking version.
4. **Process Tasks Using Servers** — two heaps (free servers + busy servers); near-identical skeleton to Meeting Rooms III with different domain language.
5. **Task Scheduler** — frequency-based greedy with a heap, connecting back to Pattern 6.

---

## Pattern 8 — Heap in Graph Algorithms (Lazy Deletion, "Best Edge/Node Next")

**Recognize it when...** you're doing **shortest path** or **minimum spanning tree** work and need to always expand the "currently cheapest" frontier node/edge next.

**Core idea:** This is the same "top extreme, repeatedly" idea as Pattern 6, but applied to graphs. Push `{cost, node}` pairs into a min-heap. Pop the cheapest, and if it's already finalized (a stale, outdated entry), just skip it — this is called **lazy deletion**, and it's the #1 new concept in this pattern versus the earlier ones.

**Practice, in order:**
1. **Network Delay Time** — the cleanest first exposure to Dijkstra's algorithm with a heap.
2. **Path with Minimum Effort** — Dijkstra's, but the "cost" being minimized is a max-along-path instead of a sum — trains you to swap the comparator's meaning, not just its direction.
3. **Cheapest Flights Within K Stops** — Dijkstra's variant with an extra constraint (stops used) baked into the heap's state — forces you to think about what belongs in the heap tuple.
4. **Min Cost to Connect All Points** — Prim's algorithm for MST, same lazy-deletion heap idea applied to building a tree instead of finding a path.
5. **Swim in Rising Water** — Dijkstra's-style heap but on a grid, with the same "minimize the max along the path" twist as Path with Minimum Effort.

---

## Suggested Full Order (If You Want One Straight Line Through All of It)

```
Pattern 1 (Top-K)              -> muscle memory for heap basics + size-k eviction
Pattern 2 (K-Way Merge)         -> muscle memory for "one slot per source" heaps
Pattern 3 (Two Heaps)            -> muscle memory for balancing two live structures
Pattern 6 (Repeated Extremes)     -> muscle memory for "pop, transform, push back"
Pattern 5 (Deferred Decision)      -> muscle memory for "push now, decide later"
Pattern 4 (Sort+Heap+Aggregate)     -> muscle memory for the hardest greedy composition
Pattern 7 (Scheduling/Intervals)     -> muscle memory for time-driven heap simulation
Pattern 8 (Graph Algorithms)          -> muscle memory for lazy deletion + graph frontiers
```

This order is deliberately **not** difficulty-sorted — it's sorted so each new pattern reuses a skill from the previous one and adds exactly one new idea, instead of throwing a completely fresh concept at you every time.

---

*Once you've cleared all 8 patterns, the real test: take a brand-new, never-seen problem and try to name which pattern (or combination of patterns) it is within the first 60 seconds of reading it — before writing any code. That's the actual skill this whole roadmap is training.*