# How to Break Down an Unseen Problem: A Case Study on LeetCode 1383 (Maximum Performance of a Team)

*This post isn't about memorizing the solution to one problem. It's about the exact sequence of questions you should ask yourself when you open a problem you've genuinely never seen before — demonstrated live on 1383, but written so the process transfers to problems this post never mentions.*

---

## Why This Post Exists

Most people learn a problem's solution and think they've learned the *problem*. They haven't. What actually needs to transfer to a new, unseen problem is the **decision process** — the specific questions you ask, in a specific order, that lead you from "I've never seen this" to "I know exactly what data structure and approach this needs."

So instead of just showing you the solution to 1383, this post shows the **thinking that would get you there even if you'd never heard of the problem** — and ends with a checklist you can run on any new problem, heap-related or not.

---

## Part 1 — The General Problem-Breakdown Process (Before We Touch 1383)

Every unseen problem, heap-based or not, should go through these five stages **in order**. Don't skip a stage just because you think you already see the answer — that's exactly how you miss edge cases.

### Stage 1 — Restate the problem in your own words, stripped of story

Problems are dressed up in stories (engineers, speeds, teams). Before anything else, strip the story and write the **raw mathematical objective** and **raw constraints**. If you can't do this, you don't understand the problem yet — go re-read it.

### Stage 2 — Identify what's "hard to track" vs "easy to track" in the objective

Almost every optimization problem's objective is built from a few operations: sums, mins, maxes, counts, products. Sums and counts are *easy* to track incrementally (add one number, done). Mins and maxes are *hard* to track incrementally **when the underlying set keeps changing** (elements get added and removed).

Your job in this stage is just to label each piece of the objective as "easy" or "hard" to maintain live.

### Stage 3 — Ask: "Can I force the hard part to become easy?"

This is the single highest-leverage question in this whole process. Usually, the hard part (a min or max) becomes *trivial* if you can guarantee an order in which you process elements such that "everyone processed so far" automatically satisfies the min/max constraint. This is almost always achieved by **sorting by the hard attribute first**.

### Stage 4 — With the hard part now pinned, what's left?

Once the min/max is pinned by sort order, what usually remains is a much simpler sub-problem: "pick the best `k` (or best subset) to maximize/minimize the easy part (the sum)." This sub-problem is a **classic pattern** (top-k, bounded heap) that you should be able to recognize instantly.

### Stage 5 — Decide when to evaluate the objective, and what data structure maintains the running aggregate cheaply

Figure out: do you evaluate the objective once, or at every step? (Usually every step — you rarely know in advance which prefix of your sorted order gives the optimum.) Then pick the data structure that lets you update the "easy" aggregate in better than linear time as the eligible set changes — this is where the heap enters, *not before*.

**Notice: the heap is the last thing you reach for, not the first.** Beginners often jump straight to "this needs a heap" without doing Stages 1–4, and end up with a heap that's solving the wrong sub-problem.

---

## Part 2 — Running This Process on LeetCode 1383, Live

Let's actually do it, pretending we've never seen this problem or its solution before.

### The problem, as given

> You're given `n` engineers, each with a `speed[i]` and `efficiency[i]`. Choose at most `k` engineers to maximize `sum(speed of chosen) × min(efficiency of chosen)`. Return the answer mod 10^9+7.

### Stage 1 — Restate it stripped of story

Forget engineers. We have `n` pairs `(a[i], b[i])`. Choose a subset `S` of size at most `k`. Maximize `(Σ a[i] for i in S) × (min b[i] for i in S)`.

That's the entire mathematical content of the problem. Everything about "engineers" and "teams" was flavor text.

### Stage 2 — Label the objective's pieces

- `Σ a[i]` — a **sum**. Easy to track incrementally: add a new element, the sum goes up by that amount; remove one, subtract.
- `min b[i]` — a **min**. Hard to track incrementally: if you remove the element currently holding the minimum, you have to rescan the rest of the set to find the new minimum. That rescan is what kills naive approaches.

So: one easy piece (sum of `a`), one hard piece (min of `b`).

### Stage 3 — Can we force the hard part to become trivial?

Ask the key question: *"Is there an order to process the `n` elements in, such that at any point during the scan, the minimum `b`-value among everyone eligible so far is always something I already know without searching?"*

Yes: **sort all elements by `b` (efficiency) in descending order.** Now scan left to right. At the moment you're standing on the `i`-th element in this sorted order, every element you've already looked at has a `b`-value **greater than or equal to** the current one (because we sorted descending). So: **if the current element is included in your chosen subset, its `b`-value is guaranteed to be the minimum of any subset drawn from elements seen so far.**

This is the entire trick of the problem, and notice we derived it from a generic question ("can I pin the hard part with an ordering?") that has nothing 1383-specific about it.

### Stage 4 — What's left once the min is pinned?

With `min(b)` pinned to "whatever `b`-value I'm currently standing on," the remaining decision is: **from all elements seen so far (all with `b ≥ current b`), pick at most `k` of them to maximize `Σ a[i]`.**

Strip the story again: *"maintain the best `k` elements of a growing stream, to maximize their sum."* You should immediately recognize this as **Pattern 1 (Top-K / Bounded Heap)** from the pattern roadmap — pure top-k, nothing fancier. This is exactly why Stage 3 matters: it reduced an unfamiliar-looking two-variable problem into a textbook one-variable problem you already know how to solve.

### Stage 5 — When to evaluate, and what structure maintains the sum?

**When to evaluate:** at every single step of the scan. Even though later elements (in our descending-`b` order) have a lower "pinned minimum," they might unlock a much larger sum of `a`-values, so the true optimum could be anywhere in the scan — you can't stop early or only check the end.

**What structure:** since Stage 4 told us this is a bounded top-k problem, and we're maximizing a sum with a fixed cap of `k` elements, the answer is a **min-heap of size `k` on the `a`-values** — the moment a bigger `a`-value shows up and we're already at capacity, evict the current smallest. Track the running sum alongside it, updating on every push/pop.

That's it — we've arrived at the full approach without ever "just knowing" it was a heap problem. Every step followed from the previous one.

### The Java solution (this should now look inevitable, not clever)

```java
import java.util.*;

class Solution {
    public int maxPerformance(int n, int[] speed, int[] efficiency, int k) {
        final int MOD = 1_000_000_007;

        // Stage 1: strip to raw pairs (b = efficiency, a = speed)
        int[][] engineers = new int[n][2];
        for (int i = 0; i < n; i++) {
            engineers[i][0] = efficiency[i]; // the "hard" attribute
            engineers[i][1] = speed[i];      // the "easy" attribute
        }

        // Stage 3: sort by the hard attribute descending to pin the min
        Arrays.sort(engineers, (x, y) -> y[0] - x[0]);

        // Stage 4/5: bounded min-heap on the easy attribute + running sum
        PriorityQueue<Integer> minHeapOfSpeed = new PriorityQueue<>();
        long speedSum = 0;
        long best = 0;

        for (int[] engineer : engineers) {
            int eff = engineer[0];
            int spd = engineer[1];

            minHeapOfSpeed.offer(spd);
            speedSum += spd;

            if (minHeapOfSpeed.size() > k) {
                speedSum -= minHeapOfSpeed.poll(); // evict smallest speed
            }

            // Stage 5: evaluate at every step
            best = Math.max(best, speedSum * eff);
        }

        return (int) (best % MOD);
    }
}
```

---

## Part 3 — The Checklist, Generalized (Use This on Any Unseen Problem)

Here's the same five stages, written as a checklist you can literally run through on paper for any new optimization problem you're handed — heap, DP, greedy, whatever it turns out to need:

1. **Strip the story.** Write the objective and constraints in raw math, no domain flavor.
2. **Label every piece of the objective** as easy-to-track-incrementally (sums, counts) or hard-to-track-incrementally (mins, maxes, medians).
3. **Ask if an ordering exists that pins the hard part.** Usually: "if I sort by X, does everything processed so far automatically satisfy some guarantee about the hard part?" If yes, sort by X and treat the current element as defining that hard value.
4. **Re-describe the remaining sub-problem in the simplest possible terms**, ignoring the original story completely. Does it match a pattern you already know (top-k, merge, two-heap, scheduling, etc.)? If you can't name the sub-problem, you likely haven't stripped it down enough — go back to step 1.
5. **Decide when to evaluate the objective** (usually: at every step, not just the end) **and what structure cheaply maintains the easy part** as the eligible set changes (heap, running counter, prefix sum, etc.).

The reason this works on genuinely unseen problems is that **none of these five questions require you to have seen the problem before.** They only require you to be honest about which parts of the objective are easy vs. hard, and disciplined about searching for an ordering that neutralizes the hard part. That discipline — not pattern memorization — is what actually separates "I've grinded 500 problems and still freeze on new ones" from "I can derive the approach to something I've never seen."

---

## Part 4 — Try It Yourself, Right Now

Before you look anything up, run this exact 5-stage checklist on **LeetCode 502 (IPO)** — deliberately chosen because on the surface it looks nothing like 1383 (it's about projects, capital, and profit, not engineers and teams).

Ask yourself:
- What's the raw objective and constraint, stripped of story? *(Stage 1)*
- Which part is easy vs. hard to track as you go? *(Stage 2)*
- Is there an ordering that pins the hard part? *(Stage 3)*
- What simple sub-problem is left once you do that? *(Stage 4)*
- What structure maintains the running piece, and when do you check the objective? *(Stage 5)*

If you can answer all five without opening the solution, you've internalized the process — not just the answer to one problem, which was always the actual goal.