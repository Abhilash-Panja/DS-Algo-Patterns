# 1.01 — Pattern 01: Recursive Decomposition & Return-Value Reasoning

> **Pattern family:** Recursion & Backtracking  
> **Priority:** Foundation / MUST UNDERSTAND  
> **Prerequisite:** `1.00 Introduction.md`  
> **Next:** `1.02 Structural Recursion & Branch Skipping.md`

## Learning objective

Learn to **derive** a recursive solution from the contract of a smaller problem, rather than writing self-calling code from memory. This chapter builds on the handwritten Fibonacci, GCD and recursion-tree work. GCD is integrated as a full worked example **after Fibonacci**, because it shows the crucial progression from mathematical equivalence to a useful recursive transition.

> **One-line definition:** Solve the current problem by trusting a precisely defined answer to one or more strictly smaller instances, then transforming or combining those answers.

---

## 1. Why recursive decomposition exists

Consider summing an array `[4, 2, 7, 3]`.

An ordinary loop already solves it in `O(n)` time and `O(1)` space. We do **not** use recursion here for speed; we use the example to discover a transferable mental model:

```text
Sum of the whole array
        ↓
Current element + sum of the remaining elements
        ↓
The remainder is the same kind of problem
        ↓
Define a contract for sum(i)
        ↓
Combine arr[i] with the answer to sum(i + 1)
```

Define `sum(i)` as **the sum of elements at indices `i` through `n - 1`**:

\[
\operatorname{sum}(i) = arr[i] + \operatorname{sum}(i+1),\qquad \operatorname{sum}(n)=0
\]

```java
static int sum(int[] arr, int i) {
    if (i == arr.length) return 0;
    return arr[i] + sum(arr, i + 1);
}
```

**Capability gained:** We can locally reason about a large problem while relying on the return-value meaning of a smaller call.

## 2. Recognition signals (not keyword matching)

Ask these **before** choosing recursion:

1. **Same problem, smaller input?** If I remove, fix, halve or transform part of the state, does the remainder still ask for the same kind of answer?
2. **Useful returned result?** Could an answer for the smaller state help determine the current answer?
3. **Precise contract?** Can I complete the sentence: `solve(state)` returns **_____**?
4. **Guaranteed progress?** What measure moves toward a base case?
5. **Combination rule?** How does the current state use the returned value(s)?
6. **Efficiency check?** Do the same subproblems repeat, or does a loop handle the work better?

**Typical language:** compute a result for a remaining suffix, combine left and right results, calculate from smaller values, reduce a pair of numbers, determine a property of a nested structure.

**Disguised version:** “Nested containers each hold another container. Find the maximum depth.” What matters is the nested self-similar structure, not the word *recursion*.

## 3. The recursive contract is the center of the pattern

**First write:** `solve(state)` **returns** ...

| Problem | Contract | Transition | Base result |
|---|---|---|---|
| Suffix sum | Sum of `arr[i..n-1]` | `arr[i] + sum(i+1)` | `sum(n)=0` |
| Suffix maximum | Maximum of non-empty `arr[i..n-1]` | `max(arr[i], max(i+1))` | Last element |
| Count even values | Count even values in suffix | `(arr[i] % 2 == 0 ? 1 : 0) + count(i+1)` | `count(n)=0` |
| Factorial | Value of `n!` | `n * factorial(n-1)` | `factorial(0)=1` |
| GCD | Greatest common divisor of `(a,b)` | `gcd(b,a%b)` | `gcd(a,0)=a` for `a>0` |

The smaller-call **contract** cannot change from one recursion depth to the next.

### State, base, progress, combination, invariant

```text
State       → What smaller instance is being solved?
Contract    → What does this call promise to return?
Base case   → What answer is known immediately?
Progress    → Why must the recursive process terminate?
Transition  → Which smaller calls are made?
Invariant   → What meaning or mathematical fact is preserved?
Combination → How is the current answer formed?
```

**Correctness proof template:**

1. **Base:** Show the base result satisfies the contract.
2. **Inductive assumption:** Assume each strictly smaller call satisfies the same contract.
3. **Step:** Explain why the transition and combination produce the current state's correct result.
4. **Termination:** Show a decreasing non-negative measure (or another well-founded progression).

---

## 4. Worked example A — Maximum element of an array

**Problem:** Return the maximum value in a **non-empty** integer array.

### 4.1 Strip the story and start naively

Input: `[3, 8, 2, 6]` → output: `8`.

A loop solves this in `O(n)` time and `O(1)` auxiliary space, which is preferable in production for a simple array scan. The recursive version teaches return-value reasoning.

### 4.2 Find the smaller problem

The maximum of a non-empty suffix is either its first value or the maximum of the remaining suffix:

\[
M(i)=\max(arr[i],M(i+1));\quad M(n-1)=arr[n-1]
\]

**Contract:** `maxFrom(i)` returns the maximum value in the suffix `arr[i..n-1]`.

**Invariant:** This meaning remains true for every valid `i`.

### 4.3 Java implementation

```java
class Solution {
    public int findMax(int[] arr) {
        if (arr == null || arr.length == 0) {
            throw new IllegalArgumentException("Array must be non-empty");
        }
        return maxFrom(arr, 0);
    }

    private int maxFrom(int[] arr, int i) {
        if (i == arr.length - 1) return arr[i];
        int remainingMax = maxFrom(arr, i + 1);
        return Math.max(arr[i], remainingMax);
    }
}
```

**Why not `return 0` as the base?** It fails for negative arrays such as `[-8, -3]`.

### 4.4 Dry run: calls descend; answers unwind

```text
maxFrom(0)
  maxFrom(1)
    maxFrom(2)
      maxFrom(3) → 6
    max(2, 6) → 6
  max(8, 6) → 8
max(3, 8) → 8
```

| Call | Calculation | Return |
|---|---|---|
| `maxFrom(3)` | Base: only `6` remains | `6` |
| `maxFrom(2)` | `max(2,6)` | `6` |
| `maxFrom(1)` | `max(8,6)` | `8` |
| `maxFrom(0)` | `max(3,8)` | `8` |

### 4.5 Complexity and interview explanation

`T(n)=T(n-1)+O(1)` gives **`O(n)` time**. There are up to `n` active calls: **`O(n)` auxiliary stack space**.

> “I define a helper returning the maximum of the suffix starting at index `i`. Its base case is the last element. Otherwise I compare the current value with the recursively returned maximum of the remaining suffix. Each call shortens the suffix, so the result is correct by induction. This takes O(n) time and O(n) stack space; an iterative scan saves the stack.”

---

## 5. Three important recursive shapes

### A. Linear recursion — one smaller answer

```java
int solve(int n) {
    if (n == 0) return BASE;
    return combine(n, solve(n - 1));
}
```

Use when the current result follows from one smaller state's answer. Examples: factorial, suffix sum, Euclidean GCD (with a different state reduction).

### B. Multiple-call recursion — combine several results

```java
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```

**Two calls ≠ backtracking.** Here we compute and combine values; we are not exploring mutable choices with undo.

### C. Divide into disjoint smaller regions

```java
int solve(int left, int right) {
    if (left == right) return valueAt(left);
    int mid = left + (right - left) / 2;
    int a = solve(left, mid);
    int b = solve(mid + 1, right);
    return combine(a, b);
}
```

This is a boundary with **divide and conquer**: splitting and combination costs determine its recurrence.

---

## 6. Worked example B — Fibonacci: correct recurrence vs. efficient execution

This connects to the recursion tree and call-stack work from the handwritten notes.

For non-negative `n`:

\[
F(0)=0,\quad F(1)=1,\quad F(n)=F(n-1)+F(n-2)
\]

```java
static long fib(int n) {
    if (n < 0 || n > 92) {
        throw new IllegalArgumentException("n must be between 0 and 92");
    }
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```

> **Contract:** `fib(n)` returns the `n`-th Fibonacci number.

### 6.1 Recursion tree vs. call stack

```text
                  fib(4)
                 /      \
             fib(3)    fib(2)
             /   \      /   \
         fib(2) fib(1) fib(1) fib(0)
         /   \
      fib(1) fib(0)
```

- **Tree:** All calls, including repeated `fib(2)`, contribute to time.
- **Stack:** Only one active root-to-current-call chain contributes to peak stack space.

### 6.2 What becomes inefficient?

`fib(2)` and other states are recomputed. The recurrence

\[
T(n)=T(n-1)+T(n-2)+O(1)
\]

leads to **`Θ(φ^n)` time**, where `φ ≈ 1.618`, and **`O(n)` stack space**.

**Lesson:** A mathematically correct recurrence does not guarantee efficient execution. Memoization or iteration avoids repeated computation; those techniques belong more deeply in Dynamic Programming.

---

## 7. Worked example C — GCD & the Euclidean Algorithm (from subtraction to modulo)

> **Why GCD belongs here:** Unlike array sum, GCD does not reduce the state by subtracting one element. It discovers a **mathematically equivalent smaller problem**, proves that the desired answer is preserved, then expresses that transformation recursively.

### 7.1 What is GCD?

The **greatest common divisor** of two non-negative integers, not both zero, is the largest positive integer dividing both.

```text
Divisors of 12 = {1, 2, 3, 4, 6, 12}
Divisors of  8 = {1, 2, 4, 8}
Common         = {1, 2, 4}
gcd(12, 8)     = 4
```

**Problem-solving question:** Can we replace `(a,b)` with smaller numbers **without changing their GCD**?

### 7.2 Euclidean division: represent two numbers

For `a >= 0`, `b > 0`:

\[
a=bq+r,\qquad q=\lfloor a/b\rfloor,\qquad 0\le r<b
\]

Example:

```text
a = 17, b = 5
17 = 5 × 3 + 2
q = 3, r = 2
```

```java
int q = a / b;
int r = a % b;
```

### 7.3 Prove the invariant in BOTH directions

**Direction 1:** If `d` divides `a` and `b`, write `a = dx`, `b = dy`. Since `r = a - bq`:

\[
r=dx-dyq=d(x-yq)
\]

Thus `d` divides `r`, so every common divisor of `(a,b)` is also a common divisor of `(b,r)`.

**Direction 2:** If `d` divides `b` and `r`, then from `a = bq + r`, `d` also divides `a`.

The sets of common divisors are identical. Therefore:

\[
\boxed{\gcd(a,b)=\gcd(b,r)=\gcd(b,a\bmod b)}
\]

This identity is not an arbitrary coding trick. It is the **correctness invariant** behind the recursive transition.

### 7.4 Step 1 — Derive the subtraction version

If `d` divides `a` and `b`, it divides `a-b`. Replacing the larger number by its difference with the smaller preserves common divisors.

**For strictly positive inputs:**

```java
static int gcdSubtract(int a, int b) {
    if (a == b) return a;
    if (a > b) return gcdSubtract(a - b, b);
    return gcdSubtract(a, b - a);
}
```

**Limitation:** This implementation needs `a > 0` and `b > 0`. With `(5,0)` it repeatedly calls itself with the same arguments. `(0,0)` is not a valid ordinary GCD input.

### 7.5 Step 2 — Identify the expensive repeated operation

```text
gcd(1_000_000_000, 1)
→ gcd(999_999_999, 1)
→ gcd(999_999_998, 1)
→ ...
→ gcd(1, 1)
```

We subtract the **same divisor over and over**.

Ask: **How many complete copies of `b` can we remove from `a`?**

Exactly `q = a / b` complete copies, leaving `r = a % b`.

```text
Repeated subtraction
       ↓
Repeatedly remove b from a
       ↓
Quotient q counts how many copies fit
       ↓
Remainder r tells what remains
       ↓
Use modulo to skip the chain
```

For `a >= b > 0`:

\[
\gcd(a,b)=\gcd(a-b,b)=\gcd(a-2b,b)=\cdots=\gcd(a-qb,b)=\gcd(r,b)
\]

By symmetry:

\[
\gcd(r,b)=\gcd(b,r)=\gcd(b,a\bmod b)
\]

**Subtlety:** When the remainder is `0`, the subtraction version can stop once the two positive inputs become equal; the modulo version reaches the zero-base case directly. The intermediate stopping behavior need not be identical for the returned GCD to agree.

### 7.6 Step 3 — Efficient recursive version

```java
static int gcd(int a, int b) {
    if (b == 0) return a;
    return gcd(b, a % b);
}
```

**Preconditions:** `a >= 0`, `b >= 0`, and **not both zero**. The implementation deliberately returns a non-negative GCD under those assumptions. For arbitrary signed `int` inputs, normalizing with `Math.abs` requires care because `Math.abs(Integer.MIN_VALUE)` overflows; use `long` or an explicit signed-input contract.

| Component | For GCD |
|---|---|
| **State** | The pair `(a,b)` |
| **Contract** | `gcd(a,b)` returns the GCD of the pair |
| **Base case** | `b == 0` → return `a` |
| **Transition** | `(a,b) → (b,a%b)` |
| **Invariant** | The GCD does not change under the transition |
| **Progress** | For `b>0`, `0 <= a%b < b` |
| **Combination** | Return the smaller call's answer unchanged |

**Why does `gcd(a,0)=a`?** For `a>0`, every positive integer divides `0`, and the greatest divisor of `a` is `a` itself. Note that `gcd(0,0)` is generally left undefined in this mathematical definition.

### 7.7 Dry run: function calls and stack unwinding

```text
gcd(48, 18)   → remainder 12
  gcd(18, 12) → remainder 6
    gcd(12, 6) → remainder 0
      gcd(6, 0) → return 6
    return 6
  return 6
return 6
```

| Call | Remainder | Next state / return |
|---|---:|---|
| `gcd(48,18)` | 12 | `(18,12)` |
| `gcd(18,12)` | 6 | `(12,6)` |
| `gcd(12,6)` | 0 | `(6,0)` |
| `gcd(6,0)` | — | Return `6` |

**Notice:** Unlike Fibonacci, we do not combine two results. Every waiting caller simply forwards the child's returned answer. This is recursion in **tail position**.

### 7.8 What if `a < b`?

No special swap is necessary for the non-negative-input implementation.

```text
gcd(3,7)
→ gcd(7,3)   because 3 % 7 = 3
→ gcd(3,1)
→ gcd(1,0)
→ 1
```

### 7.9 Complexity: why modulo is logarithmic

| Method | Time for positive integer inputs | Recursive auxiliary space |
|---|---|---|
| Repeated subtraction | `O(max(a,b))` upper bound | `O(max(a,b))` upper bound |
| Euclidean modulo | `O(log(min(a,b)+1))` | `O(log(min(a,b)+1))` |
| Iterative modulo | `O(log(min(a,b)+1))` | `O(1)` |

The `+1` expression makes the bound readable even near the smallest positive input. This uses the usual **constant-time integer arithmetic** model.

**Why logarithmic?** Let `a >= b > 0` and `r = a % b`.

- If `r < b/2`, the second argument is already less than half of `b`.
- Otherwise `r >= b/2`. In the next call, because `b < 2r`, the next remainder is `b % r = b-r <= b/2`.

So after **at most two reductions**, the positive remainder falls by at least a factor of two. The number of recursive calls is logarithmic.

**Caution:** It is **not** true that *every single modulo step* halves the second argument.

### 7.10 Iterative version — same invariant, less stack

```java
static int gcdIterative(int a, int b) {
    while (b != 0) {
        int remainder = a % b;
        a = b;
        b = remainder;
    }
    return a;
}
```

The same non-negative-input preconditions apply. Java does **not** guarantee tail-call optimization, so iteration is useful when you want `O(1)` auxiliary space.

### 7.11 Boundary checks and counterexamples

| Claim or code habit | Why it's wrong | Safer rule |
|---|---|---|
| “Any GCD subtraction code handles zero.” | `gcdSubtract(5,0)` never progresses. | Require strictly positive subtraction inputs or handle zero. |
| “`a%b` is less than both `a` and `b`.” | With `a<b`, `a%b=a`. | For `b>0`, the remainder is **less than `b`**. |
| “Modulo halves the number every step.” | Some consecutive remainders shrink by less than half. | Halving is guaranteed within **at most two** steps. |
| “`gcd(0,0)=0` follows from the divisor definition.” | There is no greatest positive common divisor of `(0,0)`. | Explicitly define a separate convention if an API needs one. |
| “Tail recursion uses constant Java stack.” | Java does not promise tail-call elimination. | Use a loop for constant auxiliary space. |

### 7.12 Interview explanation (about 60 seconds)

> “The key insight is that `a = b*q + r`, so the common divisors of `a` and `b` are exactly the common divisors of `b` and `r`. That means their GCD is preserved when we replace `(a,b)` with `(b,a%b)`. We repeat this smaller equivalent problem until the second argument becomes zero; then the first value is the GCD. Each remainder is smaller than the previous positive divisor, and within at most two steps it shrinks by at least half, giving logarithmic time. The recursive version also uses logarithmic stack space.”

### 7.13 What to transfer to unseen recursion problems

```text
Discover a relationship
       ↓
Prove what answer remains preserved
       ↓
Express a smaller state
       ↓
Identify a stopping state
       ↓
Check termination and efficiency
       ↓
Only then write the recursive function
```

**The Euclidean version is repeated subtraction with bulk reductions expressed by `%`, justified by a mathematical invariant—not merely a memorized formula.**

---

## 8. Choosing recursion vs. neighboring techniques

| Question | Recursive decomposition | Backtracking | Dynamic programming |
|---|---|---|---|
| What does state represent? | Smaller instance | Partial chosen configuration | Reusable subproblem |
| What does the call do? | Return/transform a smaller answer | Explore candidate branches | Reuse computed answers |
| Is undo needed? | Usually not | Often, with mutable state | Not inherent |
| What is expensive? | Function calls / combination | Search space size | Repeated states |
| Representative example | GCD, suffix maximum | N-Queens | Memoized Fibonacci |

**When NOT to use recursive decomposition as the implementation:**

- A simple loop is clearer, uses less memory and has the same time complexity.
- The recursion depth may exceed Java stack capacity.
- Overlapping subproblems are repeatedly recomputed (consider memoization/DP).
- The key problem is choosing among many partial candidate configurations (use a backtracking mental model).
- The input supports a more specialized invariant, greedy rule or structural shortcut.

---

## 9. Code fingerprints and templates

### Return and combine

```java
Result solve(State state) {
    if (isBase(state)) return baseAnswer(state);
    Result smaller = solve(reduce(state));
    return combine(state, smaller);
}
```

### Return a smaller equivalent answer unchanged (GCD-shaped)

```java
int solve(int a, int b) {
    if (b == 0) return a;
    return solve(b, a % b);
}
```

### Divide and combine

```java
Result solve(State state) {
    if (isBase(state)) return baseAnswer(state);
    Result left = solve(leftPart(state));
    Result right = solve(rightPart(state));
    return combine(left, right);
}
```

**Fingerprint:** `return combine(current, solve(smaller));` usually means value-returning decomposition.  
**Fingerprint:** `return solve(reduced);` often means the answer is **invariant** under a transformation (as in GCD).

---

## 10. Curated problem progression

The goal is **pattern transfer**, not maximizing the question count.

| Level | Problem / link | What new idea it teaches | Priority |
|---|---|---|---|
| Anchor | [Fibonacci Number](https://leetcode.com/problems/fibonacci-number/) | Recurrence, base case, repeated work | MUST SOLVE |
| Anchor | **GCD of two numbers** (worked example in §7) | Mathematical invariant → smaller equivalent state | MUST SOLVE |
| Core | [Pow(x, n)](https://leetcode.com/problems/powx-n/) | Halve problem size instead of subtracting one | MUST SOLVE |
| Core | [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) | Returning a transformed structure while unwinding | SHOULD SOLVE |
| Variation | [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) | Compose a current decision with a returned list | SHOULD SOLVE |
| Boundary | [K-th Symbol in Grammar](https://leetcode.com/problems/k-th-symbol-in-grammar/) | Recursive structure without generating all states | OPTIONAL HERE; MAINLY PATTERN 02 |

**Redundancy filter:** Factorial and suffix sum remain teaching exercises, not additional required platform problems. GCD adds a genuinely different insight (invariant-preserving transformation), so it deserves a full worked section.

---

## 11. Follow-up training: what changes?

| Interview modification | What breaks or changes? |
|---|---|
| Input grows very large | Recursive stack may be unacceptable; consider iteration. |
| Same state is revisited | Return-value contract survives, but memoization/DP can avoid repeated work. |
| Need an actual path instead of a scalar | Contract may return a structure or track reconstruction information. |
| Need only a yes/no answer | Consider `boolean` and early return. |
| One state splits into two independent halves | Derive the new divide-and-conquer recurrence. |
| Input may contain cycles | “Smaller” may not hold; specify visited logic or a proof of progress. |
| GCD inputs may be negative | Normalize safely; use `long` if `Integer.MIN_VALUE` is possible. |
| Tail-recursive implementation must use constant stack | Replace recursion with a loop in Java. |

---

## 12. Mistake library

### Recognition mistakes

- **Mistake:** “The statement says *recursively*, so recursion must be optimal.”  
  **Why:** The word describes an implementation option, not complexity.  
  **Prevent:** Compare loop/DP/structural shortcut first.
- **Mistake:** “Every reduction must be `n-1`.”  
  **Why:** GCD requires an invariant-preserving state transform.  
  **Prevent:** Derive the relationship from problem structure.

### Reasoning mistakes

- **Mistake:** Cannot define `solve(state)`'s meaning.  
  **Detect:** Different levels seem to return different kinds of answers.  
  **Prevent:** Write the contract before code.
- **Mistake:** States get smaller but the returned value is wrong.  
  **Detect:** Base case or combination violates the contract.  
  **Prevent:** Prove base + inductive step.

### Implementation mistakes

- **Mistake:** `0` as a suffix maximum base value.  
  **Detect:** Negative-only input fails.  
  **Prevent:** Use a legitimate value from a non-empty suffix.
- **Mistake:** `% b` before checking `b == 0`.  
  **Detect:** Arithmetic exception.  
  **Prevent:** Handle the GCD base case first.
- **Mistake:** Subtraction-based GCD applied to zero.  
  **Detect:** Non-progressing recursion.  
  **Prevent:** Enforce positive inputs or handle zero.

### Complexity mistakes

- **Mistake:** Every two-call recursive function is declared `O(2^n)`.  
  **Prevent:** Derive its actual recurrence and count the work.
- **Mistake:** Total recursion-tree nodes are treated as simultaneous stack frames.  
  **Prevent:** Time follows all executed calls; stack space follows maximum active depth.
- **Mistake:** Claims every Euclidean step halves the input.  
  **Prevent:** Use the correct **at most two steps** argument.

### Interview mistakes

- **Mistake:** Coding `return gcd(b,a%b)` without explaining why it works.  
  **Prevent:** Cite `a=bq+r` and the equality of common-divisor sets.
- **Mistake:** Ignoring preconditions and boundary inputs.  
  **Prevent:** State input domain before the recursive skeleton.

---

## 13. Pattern 01 revision table

| Signal | Contract / invariant | Typical Java shape | Main trap |
|---|---|---|---|
| Need a suffix property | `solve(i)` answers for `i..n-1` | `return combine(arr[i],solve(i+1))` | Empty/negative bases |
| Smaller numerical expression | Exact returned value for `n` | `return combine(n,solve(smaller))` | No progress / overflow |
| Mathematical equivalence | Answer is preserved by transformation | `return solve(transform(state))` | Unproven invariant |
| Two smaller independent answers | Both calls satisfy their contracts | `combine(solve(left),solve(right))` | Incorrect time recurrence |
| Repeated recursive states | Same contract invoked many times | Memoized state lookup | Exponential recomputation |

---

## 14. Mastery checklist

- [ ] I can define the exact return-value contract for a recursive call.
- [ ] I can derive base cases from the problem, not choose arbitrary values.
- [ ] I can prove progress toward termination.
- [ ] I can explain call entry vs. stack unwinding.
- [ ] I can distinguish recursion tree size from maximum call depth.
- [ ] I can derive the suffix maximum solution and complexity.
- [ ] I can explain why Fibonacci recomputes states.
- [ ] I can derive `a=bq+r` and prove GCD preservation in **both directions**.
- [ ] I can explain repeated subtraction → quotient → remainder → modulo.
- [ ] I can write recursive and iterative GCD from memory.
- [ ] I can handle `a<b`, zero inputs, and signed-input assumptions correctly.
- [ ] I can explain Euclidean time complexity without the false “halves every step” claim.
- [ ] I can compare recursion against iteration, backtracking and DP.
- [ ] I can give a 60–90-second explanation of GCD before coding.

## 15. Active recall (answer without notes)

1. What does a recursive function's **contract** mean?
2. Why is choosing the correct base *value* as important as reaching a base case?
3. Why can a recursive tree have many nodes but a relatively small active stack?
4. What makes two recursive calls different from backtracking choices?
5. Why does Fibonacci's valid recurrence still recompute work?
6. How does `a=bq+r` lead to **both directions** of the GCD proof?
7. Why does subtraction preserve the GCD?
8. Which repeated operation does `%` eliminate?
9. Why is `gcd(3,7)` valid without an explicit swap?
10. What goes wrong with subtraction-GCD if one argument equals zero?
11. Why isn't `a%b` always smaller than `a`?
12. Why is Euclid's algorithm logarithmic if one step need not halve its divisor?
13. How does an invariant-preserving recursive transform differ from combining subresults?
14. When should an iterative Java implementation be preferred?
15. How would you explain GCD to an interviewer before writing code?

## 16. Revision schedule

| When | Reconstruct without reading |
|---|---|
| **24 hours** | Suffix maximum contract, Fibonacci tree, GCD two-way proof, `gcd(48,18)` stack dry run |
| **7 days** | GCD subtraction → modulo derivation, Pow(x,n), recursive linked-list reversal |
| **30 days** | Contract/invariant/base/progress/complexity for three unseen reductions; iterative GCD |
| **Interview week (5 minutes)** | Return contract, base case, termination measure, GCD invariant, `%` zero trap, stack cost |

---

## 17. Unseen-style diagnostic (do not look up the solution)

**Problem — Repeated digit reduction:** Given a positive integer, repeatedly sum its decimal digits until one digit remains.

Example: `9875 → 29 → 11 → 2`.

Before coding, provide:

1. Problem stripped of story
2. Brute-force approach
3. Bottleneck
4. Structural clues
5. Suspected recursive pattern
6. State and invariant
7. Data structure / return-value contract
8. Expected time and auxiliary space

Then ask whether a mathematical invariant permits an even faster solution. **Do not memorize a shortcut without deriving it.**

---

## Final mental model

```text
Define exactly what solve(state) returns
                 ↓
Find a truly smaller / simpler state
                 ↓
Prove correctness of the transition
                 ↓
Derive a correct base answer
                 ↓
Check progress and termination
                 ↓
Combine or forward the recursive result
                 ↓
Analyze repeated work and stack space
                 ↓
Decide whether recursion is the best implementation
```

> **Carry into Pattern 02:** Sometimes an entire search branch can be skipped after proving its structure or number of outcomes. GCD taught us to replace repeated work with a mathematically justified reduction; structural recursion extends that discipline to indexed results.
