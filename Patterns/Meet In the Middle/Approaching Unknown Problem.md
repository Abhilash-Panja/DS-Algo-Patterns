# Meet in the Middle — Approaching Unknown Problems

# How to Recognize and Derive MITM on an Unseen Problem

> The goal is NOT to look at a problem and somehow magically remember:
>
> `"This is Meet in the Middle."`
>
> The goal is to follow a sequence of reasoning steps that makes MITM emerge naturally.

---

# 1. Why This File Exists

Knowing MITM solutions is not enough.

In an interview, the difficult part is usually not writing:

```java
generate(...)
Arrays.sort(...)
lowerBound(...)
```

The difficult part is recognizing:

```text
Why is brute force too large?

Why doesn't DP fit?

Why does splitting help?

What should each half store?

How should the two halves meet?
```

This file gives one reusable process for answering those questions.

---

# 2. The 30–90 Second MITM Recognition Framework

When you receive an unseen problem, run these stages in order.

```text
1. Strip the story
        ↓
2. Identify the independent decisions
        ↓
3. Calculate brute-force search space
        ↓
4. Inspect constraints
        ↓
5. Check obvious polynomial alternatives
        ↓
6. Calculate half-search size
        ↓
7. Test whether decisions can be separated
        ↓
8. Design the smallest sufficient partial state
        ↓
9. Write the left/right compatibility equation
        ↓
10. Classify the join
        ↓
11. Choose the data structure
        ↓
12. Verify complexity + memory
```

MITM should appear near the end of the reasoning.

Not at the beginning.

---

# 3. Stage 1 — Strip the Story

Ignore words like:

```text
workers
tasks
coins
cities
teams
projects
scores
```

Rewrite the problem using only:

```text
choices
constraints
objective
```

For example:

> You have `N` projects. Each project has a profit. Choose some projects whose total profit equals `X`.

Strip it to:

```text
Given N numbers.

Each number:
take / skip.

Need:
subset sum = X.
```

Now the combinatorial structure is visible.

---

# 4. Stage 2 — Identify the Independent Decisions

Ask:

> What choices create the search space?

Examples:

### Subset problem

```text
element 1 → take / skip
element 2 → take / skip
...
```

Search space:

```text
2^N
```

---

### Three-choice assignment

```text
item →
option A
option B
ignore
```

Search:

```text
3^N
```

---

### Multiple arrays

```text
choose one from A
choose one from B
choose one from C
choose one from D
```

Search:

```text
|A| × |B| × |C| × |D|
```

MITM starts from understanding what creates the combinatorial explosion.

---

# 5. Stage 3 — Calculate Brute Force

Do not merely say:

> "Brute force is exponential."

Calculate it.

Example:

```text
N = 40

2^40
≈ 1.1 × 10^12
```

That immediately tells you direct subset enumeration is unrealistic.

Another example:

```text
4 arrays
n = 200

200^4
=
1.6 × 10^9
```

Again, too large.

The number itself guides the next decision.

---

# 6. Stage 4 — Read Constraints as Algorithmic Evidence

Constraints are not algorithm names.

They eliminate possibilities.

Suppose:

```text
N <= 40
values <= 10^12
```

That tells us two things.

### Number of decisions

```text
N = 40
```

is too large for:

```text
2^N
```

but may still be small enough for something exponential with a smaller exponent.

### Numeric dimension

```text
10^12
```

makes a sum-indexed DP suspicious.

So the constraints are saying:

```text
Don't enumerate every subset.

Don't create DP over every possible sum.

Find another dimension to exploit.
```

---

# 7. Stage 5 — Check Simpler Alternatives First

Before reaching for MITM, ask:

```text
Can sorting solve it?

Can greedy solve it?

Can two pointers solve it?

Can ordinary DP solve it?

Can bitset DP solve it?

Can graph algorithms solve it?

Can full brute force already pass?
```

MITM is useful because other approaches fail under a specific constraint combination.

Do not force it where a simpler polynomial solution exists.

---

# 8. The Important DP Check

Subset-like problem?

Immediately inspect:

```text
N
```

and:

```text
sum / capacity range
```

Example A:

```text
N = 500
target = 10,000
```

DP may be excellent:

```text
O(N × target)
```

MITM:

```text
2^250
```

impossible.

---

Example B:

```text
N = 40
target = 10^12
```

DP:

```text
O(N × target)
```

impossible.

MITM:

```text
2^20
≈ 10^6
```

potentially excellent.

So ask:

> Which dimension is actually small enough to exploit?

---

# 9. Stage 6 — Calculate the Half Search

When full combinatorial search is too large, develop this automatic habit:

```text
2^N too large
```

Immediately calculate:

```text
2^(N/2)
```

Example:

```text
N = 40

2^40 ≈ 10^12
2^20 ≈ 10^6
```

That is a massive difference.

For a `b`-choice problem:

```text
b^N
```

calculate:

```text
b^(N/2)
```

For multiple variable groups:

```text
n^K
```

ask whether dimensions can be split so each side is closer to:

```text
n^(K/2)
```

---

# 10. The First Major MITM Question

After discovering half enumeration is feasible, ask:

> **Can I divide the decisions into two groups whose partial results can be generated independently?**

For subset sum:

```text
Left elements
Right elements
```

Easy.

---

For:

```text
a+b+c+d = X
```

possible grouping:

```text
(a+b)
(c+d)
```

Easy.

---

But if decisions strongly depend on each other across groups, splitting may not help.

---

# 11. What Does "Independent Enough" Mean?

It does NOT mean the two halves never interact.

They obviously must interact when they meet.

It means:

> I can enumerate a partial solution from one group without needing to know every individual choice made in the other group.

For subset sum:

```text
left subset sum
```

can be generated without knowing the right subset.

Later they interact through:

```text
leftSum + rightSum
```

Perfect.

---

# 12. Stage 7 — Design the Partial State

This is one of the most important stages.

Do NOT automatically say:

```text
state = sum
```

Ask:

> What information about this partial solution will the other half need?

---

## Example 1 — Exact Sum

Need:

```text
L + R = X
```

State:

```text
sum
```

Enough.

---

## Example 2 — Exactly K Elements

Need:

```text
leftCount + rightCount = K
```

and objective depends on sum.

State:

```text
(count, sum)
```

---

## Example 3 — Need Reconstruction

May need:

```text
(sum, mask)
```

---

## Example 4 — Quotas

Could require:

```text
(redCount, blueCount, sum)
```

The state must preserve the smallest amount of information needed for future compatibility.

---

# 13. The State-Sufficiency Test

Take two partial solutions that produce identical stored state.

Ask:

> Can I replace one with the other in every future continuation without changing legality or answer quality?

If YES:

```text
state is sufficient
```

If NO:

```text
you forgot information
```

Example:

```text
sum = 10 using 2 elements
sum = 10 using 5 elements
```

If final solution requires exactly `7` elements:

these are NOT equivalent.

Therefore:

```text
state = sum
```

is insufficient.

---

# 14. Stage 8 — Write the Compatibility Equation

Before choosing any data structure, write exactly how left and right states must combine.

Examples:

### Exact

```text
L + R = X
```

---

### Maximum under limit

```text
L + R <= X
```

---

### Closest

```text
minimize |L + R - X|
```

---

### Exact count

```text
leftCount + rightCount = K
```

plus perhaps:

```text
leftSum + rightSum ≈ target
```

---

### Four groups

```text
(A+B) + (C+D) = X
```

This equation determines the MITM pattern.

Not the story.

---

# 15. Stage 9 — Identify the Repeated Query

For every state from one side, what question will we repeatedly ask about the other side?

This is where the data structure appears.

---

## Question Type A

```text
Does R = X-L exist?
```

Need:

```text
exact lookup
```

Possible structures:

```text
HashSet
HashMap
sorted array + binary search
```

Pattern 1.

---

## Question Type B

```text
What is the largest R <= X-L?
```

Need:

```text
ordered predecessor query
```

Possible structures:

```text
sorted array + upperBound
two pointers
TreeSet
```

Pattern 2.

---

## Question Type C

```text
Which R is closest to X-L?
```

Need:

```text
lowerBound
+
neighbor check
```

Pattern 2.

---

## Question Type D

```text
Which R states are legally allowed?
```

Need metadata first:

```text
right[K-leftCount]
```

Then run A/B/C inside the correct bucket.

Pattern 3.

---

# 16. The Most Important Rule

Do NOT choose the data structure before deriving the query.

Bad reasoning:

```text
This feels like binary search.
```

Better:

```text
For each L,
I need largest R <= X-L.

If R states are sorted,
valid states form a prefix.

Therefore binary search finds the boundary.
```

Data structures should feel inevitable.

---

# 17. Stage 10 — Make Sure the Join Does Not Recreate Brute Force

Suppose:

```text
|Left| = M
|Right| = M
```

If your join does:

```text
for every left
    for every right
```

then:

```text
M²
```

For subset MITM:

```text
M = 2^(N/2)
```

so:

```text
M² = 2^N
```

You have recreated the original search.

Every MITM solution must answer:

> How am I avoiding the Cartesian product between my two partial state sets?

---

# 18. MITM Recognition Flow

Run this quickly in an interview.

```text
Combinatorial choices?
│
├── No
│    → probably not MITM
│
└── Yes
     ↓
What is brute force?
2^N? 3^N? n^K?
     ↓
Does brute force already pass?
│
├── Yes
│    → use simpler brute force
│
└── No
     ↓
Is there a polynomial / DP solution?
│
├── Yes and feasible
│    → probably use that
│
└── No / state dimension too large
     ↓
What happens if I split decisions?
     ↓
Is half-search manageable?
│
├── No
│    → classical MITM not enough
│
└── Yes
     ↓
Can both groups be enumerated independently?
│
├── No
│    → MITM may not fit
│
└── Yes
     ↓
What is the smallest sufficient state?
     ↓
How must states combine?
     ↓
Can I join them efficiently?
│
├── No
│    → MITM gives no useful reduction
│
└── Yes
     ↓
MITM is a serious candidate
```

---

# 19. How to Identify Pattern 1

After deciding MITM fits, ask:

> Is the other-side state uniquely determined by an equality?

Examples:

```text
L + R = X

L - R = X

L = R
```

If yes:

```text
R = exact required value
```

Think:

```text
Pattern 1 — Exact Complement Matching
```

Fingerprint:

```java
need = target - left;
```

followed by:

```java
contains(need)
```

or:

```java
freq.getOrDefault(need, 0)
```

---

# 20. How to Identify Pattern 2

Ask:

> If the exact complement does not exist, could another nearby state still produce the optimum?

If yes, likely Pattern 2.

Examples:

```text
maximum sum <= X
```

requires:

```text
largest R <= X-L
```

---

```text
minimum sum >= X
```

requires:

```text
smallest R >= X-L
```

---

```text
closest sum to X
```

requires:

```text
R nearest to X-L
```

Think:

```text
sort
+
boundary search
```

---

# 21. How to Identify Pattern 3

Ask:

> Are all states from the other half legally compatible?

If NO:

ask why.

Example:

```text
choose exactly K elements
```

Then:

```text
leftCount + rightCount = K
```

So compatibility depends on:

```text
count
```

State becomes:

```text
(count, sum)
```

Think:

```text
classify legal states first
then perform complement search
```

---

# 22. How to Identify Pattern 4

Ask:

> Is the exponential/high-order search created by several separate decision groups rather than take/skip decisions on one array?

Example:

```text
A × B × C × D
```

with condition:

```text
a+b+c+d = X
```

Rewrite:

```text
(A+B)
+
(C+D)
=
X
```

Think:

```text
Pattern 4 — Variable-Group MITM
```

Then Pattern 1/2/3 usually determines the join.

---

# 23. MITM Is Often Layered

Do not expect every problem to fit exactly one label.

Examples:

### CSES 1628

```text
MITM
+
Pattern 1
```

---

### ABC184 F

```text
MITM
+
Pattern 2 boundary
```

---

### LC1755

```text
MITM
+
Pattern 2 closest
```

---

### LC2035

```text
MITM
+
Pattern 3 metadata
+
Pattern 2 closest
```

---

### LC454

```text
Pattern 4 variable grouping
+
Pattern 1 exact matching
```

Think of patterns as reusable reasoning layers.

---

# 24. What Constraints Should Make Me Pause?

Typical subset-style zones:

```text
N <= ~20
```

Full subset brute force may already work.

---

```text
N around 30–45
```

MITM should enter your candidate list if search is exponential.

---

```text
N = 100+
```

ordinary `2^(N/2)` MITM is usually impossible.

Look for different structure.

But these are heuristics.

Always calculate.

---

# 25. Numeric Constraints Matter Too

Suppose:

```text
N = 40
```

and:

```text
target = 500
```

Maybe a DP over target is much easier.

Suppose:

```text
N = 40
target = 10^15
```

MITM becomes much more attractive.

The powerful MITM signal is often:

```text
moderate number of decisions

+

huge numeric state range
```

---

# 26. Positive vs Negative Values

Never assume positivity unless the problem guarantees it.

With positive values:

```text
currentSum > limit
```

might allow pruning.

With negative values:

```text
currentSum > limit
```

does NOT imply the branch is dead.

Example:

```text
sum = 20
future value = -15
```

can return to:

```text
5
```

MITM does not automatically justify pruning.

---

# 27. Equality vs Optimization

One quick classification question:

> Is the problem asking for a valid state or the best valid state?

### Valid state

```text
does exact sum exist?
count exact sums?
```

Pattern 1.

---

### Best valid state

```text
maximum <= X
minimum >= X
closest to X
```

Pattern 2.

---

### Best state under compatibility restrictions

Pattern 3 + Pattern 2.

---

# 28. Existence vs Counting

Another immediate distinction.

### Existence

```text
HashSet
```

may be enough.

---

### Counting

Need multiplicity:

```text
HashMap<State, Frequency>
```

or sorted arrays with duplicate-run counting.

Ask:

> Can multiple partial choices generate the same state and still represent distinct solutions?

If yes, do not deduplicate for counting.

---

# 29. Static vs Dynamic Partial States

Most classical MITM problems are:

```text
generate once
query many times
```

This strongly favors:

```text
primitive arrays
sorting
hash maps
```

If states are constantly inserted or deleted after preprocessing, the data structure question changes.

MITM architecture may remain.

Join structure may not.

---

# 30. Memory Check Before Coding

MITM frequently moves cost from time into memory.

Estimate:

```text
number of states
×
bytes per state
```

Example:

```text
2^20
≈ 1,048,576
```

primitive `long`s:

roughly:

```text
8 MB
```

plus array overhead.

But:

```text
1 million Long objects
```

inside Java collections can consume dramatically more memory.

Prefer primitive arrays when state counts are large and structure is simple.

---

# 31. Balanced Splitting Rule

For binary decisions:

```text
20 + 20
```

is better than:

```text
10 + 30
```

because the larger half dominates.

For unequal variable groups, balance:

```text
product of branching factors
```

rather than simply number of variables.

General principle:

> Minimize the size of the largest partial search space.

---

# 32. Example Recognition Simulation 1

Problem:

```text
N <= 40
numbers <= 10^12
determine whether subset sum X exists
```

Reasoning:

```text
take/skip
→ 2^N

2^40 too large

sum DP impossible due huge values

2^20 manageable

split decisions

state = sum

need:
L + R = X

R = X-L

exact lookup
```

Conclusion:

```text
MITM Pattern 1
```

---

# 33. Example Recognition Simulation 2

Problem:

```text
N <= 40
positive numbers
find maximum subset sum <= X
```

Reasoning:

```text
2^N too large

2^(N/2) feasible

state = sum

need:
L + R <= X

for fixed L:
largest R <= X-L

requires ordered states
```

Conclusion:

```text
MITM Pattern 2
```

---

# 34. Example Recognition Simulation 3

Problem:

```text
30 numbers
choose exactly 15
minimize difference between selected
and unselected sums
```

Reasoning:

```text
large combinatorial search

split 15 + 15

state=sum?
No.

Same sum produced with different counts
has different compatibility.

state=(count,sum)

leftCount + rightCount = 15

inside legal bucket,
find closest sum.
```

Conclusion:

```text
Pattern 3
+
Pattern 2
```

---

# 35. Example Recognition Simulation 4

Problem:

```text
four arrays
size 200
count tuples summing to X
```

Reasoning:

```text
naive n^4
≈ too large

independent dimensions

group:
A+B
C+D

each side n²

need exact:
AB + CD = X

frequency map
```

Conclusion:

```text
Pattern 4
+
Pattern 1
```

---

# 36. The Five Questions to Ask Under Interview Pressure

If time is limited, remember only these.

### Question 1

```text
What is the naive search space?
```

---

### Question 2

```text
If I split the decisions,
does the state count become manageable?
```

---

### Question 3

```text
What must each half remember?
```

---

### Question 4

```text
What mathematical relation must
leftState and rightState satisfy?
```

---

### Question 5

```text
How do I find the compatible right state
without scanning every right state?
```

If you can answer all five, the solution is usually close.

---

# 37. A Useful Interview Scratchpad

Write:

```text
Brute:
__________

Why fails:
__________

Half states:
__________

State:
__________

Compatibility:
__________

Repeated query:
__________

Data structure:
__________

Complexity:
__________
```

Example:

```text
Brute:
2^40

Why fails:
~10^12

Half states:
2^20

State:
sum

Compatibility:
L+R <= X

Repeated query:
largest R <= X-L

DS:
sorted right + upperBound

Complexity:
O(N * 2^(N/2))
```

This forces structured reasoning.

---

# 38. Common Recognition Mistakes

## Mistake 1 — Memorizing `N <= 40`

Wrong:

```text
N <= 40
→ MITM
```

Better:

```text
exponential decisions
+
full search too large
+
half search feasible
+
separable states
+
efficient join
```

---

## Mistake 2 — Seeing subset and jumping to DP

Check whether the numeric DP dimension is feasible.

---

## Mistake 3 — Seeing target and assuming exact complement

Read the objective.

Maybe the problem asks:

```text
closest
maximum
minimum
```

instead.

---

## Mistake 4 — Storing only the obvious objective value

Ask whether compatibility requires metadata.

---

## Mistake 5 — Splitting without designing the join

MITM is:

```text
split
+
enumerate
+
efficient meet
```

not:

```text
split
```

---

# 39. Coding Order

Once the approach is established, code in this order:

```text
1. Choose split.

2. Define state.

3. Write state generator.

4. Verify empty state / boundary states.

5. Generate halves.

6. Organize searched side:
   map / sort / buckets.

7. Implement join.

8. Compute answer.

9. Check overflow.

10. Check duplicates.

11. State complexity.
```

This avoids mixing recognition logic with implementation debugging.

---

# 40. Before Submitting / Before Saying "Done"

Run these checks.

### State correctness

```text
Am I preserving everything needed?
```

### Empty state

```text
Did I include selecting nothing?
```

### Metadata

```text
Are incompatible states prevented from meeting?
```

### Multiplicity

```text
Am I incorrectly deduplicating counting states?
```

### Overflow

```text
Should sums/counts use long?
```

### Binary-search boundary

```text
lowerBound or upperBound?
```

### Negative numbers

```text
Did I use an invalid pruning condition?
```

### Join complexity

```text
Did I accidentally rebuild the Cartesian product?
```

---

# 41. The Final Unknown-Problem Framework

When facing an unseen MITM candidate, mentally say:

```text
1. Strip the story.

2. Identify the independent decisions.

3. Calculate the full search space.

4. Use constraints to eliminate naive / DP options.

5. Calculate the half-search space.

6. Ask whether decisions can be separated.

7. Define the smallest sufficient partial state.

8. Write the compatibility equation.

9. Ask what repeated query this creates.

10. Map that query to:
    exact lookup,
    ordered boundary,
    nearest state,
    or metadata-restricted search.

11. Choose the structure.

12. Verify the join is cheaper than brute force.

13. Verify memory.

14. Only then code.
```

---

# 42. The One Sentence to Remember

> **When brute force is combinatorial and too large, don't immediately search for an algorithm name. Ask whether the decision space can be split, what each side must remember, and whether compatible partial states can be found without rebuilding the original Cartesian search.**

That is how MITM should emerge on an unseen problem.
