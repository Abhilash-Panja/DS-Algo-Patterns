# Meet in the Middle — Pattern Decision Tree

# Goal

Use this tree only **after MITM has become a serious candidate**.

It answers:

> Which MITM pattern am I actually dealing with?

---

# 1. First — Is This Even MITM?

```text
Does the problem contain a
combinatorial search space?
│
├── No
│    → Probably not MITM
│
└── Yes
     ↓
Is full enumeration too expensive?
│
├── No
│    → Use simpler enumeration
│
└── Yes
     ↓
Does splitting the independent decisions
make each partial search manageable?
│
├── No
│    → Classical MITM probably not enough
│
└── Yes
     ↓
Can each group be enumerated
independently into compact states?
│
├── No
│    → MITM likely not appropriate
│
└── Yes
     ↓
Can states from the two groups
be joined substantially faster than
trying every pair?
│
├── No
│    → Splitting gives no useful benefit
│
└── Yes
     ↓
MITM is viable
```

---

# 2. Main Pattern Decision Tree

```text
MITM is viable
│
↓
Where does the combinatorial search come from?
│
├── Multiple independent variable groups
│      such as A × B × C × D
│
│      → PATTERN 4
│        Variable-Group MITM
│
│        Then ask:
│        "How should grouped states meet?"
│                  │
│                  ├── Exact equality
│                  │     → Pattern 4 + Pattern 1
│                  │
│                  ├── Best / closest value
│                  │     → Pattern 4 + Pattern 2
│                  │
│                  └── Extra legality metadata
│                        → Pattern 4 + Pattern 3
│
└── One set of combinatorial decisions
       such as take / skip
       ↓
Do all partial states with the same
objective value have identical future compatibility?
│
├── No
│     → PATTERN 3
│       Constraint-Aware MITM
│
│       Preserve metadata:
│       (count, sum), etc.
│
│       Then inside compatible classes:
│       exact?
│       → Pattern 1
│
│       best/closest?
│       → Pattern 2
│
└── Yes
      ↓
Is the required partner an EXACT state?
│
├── Yes
│     → PATTERN 1
│       Exact Complement Matching
│
│       Example:
│       L + R = X
│
│       Need:
│       R = X-L
│
└── No
      ↓
Do I need an extremal or nearest compatible state?
│
├── Yes
│     → PATTERN 2
│       Best Compatible State
│
│       Example:
│       largest R <= X-L
│
│       or
│
│       R nearest to X-L
│
└── No
      → Re-examine the state/relationship.
        It may not fit the four core MITM families.
```

---

# 3. Pattern 1 Decision

Think Pattern 1 when:

```text
Full answer condition
can be rearranged into:

rightState
=
one exact value determined by leftState
```

Examples:

```text
L + R = X
→ R = X-L
```

```text
L = R
```

```text
L - R = X
→ R = L-X
```

Question:

```text
Does this exact state exist?
```

or:

```text
How many such states exist?
```

Use:

```text
HashSet
HashMap
binary search
two pointers
```

depending on existence/counting/storage needs.

---

# 4. Pattern 2 Decision

Think Pattern 2 when exact complement absence does NOT kill the candidate.

Example:

```text
maximize L + R <= X
```

For fixed `L`:

```text
R need not equal X-L
```

We want:

```text
largest R <= X-L
```

Another example:

```text
minimize |L+R-X|
```

Need:

```text
R nearest to X-L
```

Signals:

```text
maximum under limit
minimum above limit
closest possible
minimum difference
best achievable
```

Core structure:

```text
sort states
↓
ordered boundary exists
↓
binary search / two pointers
```

---

# 5. Pattern 3 Decision

Think Pattern 3 when:

```text
state value alone
does not determine legality
```

Test:

```text
Can two states have the same sum
but require different states from
the opposite half?
```

If yes:

```text
sum is insufficient
```

Example:

```text
exactly K selected
```

Two partial states:

```text
sum = 20, count = 2
sum = 20, count = 7
```

are not equivalent.

Need:

```text
(count, sum)
```

Compatibility:

```text
leftCount + rightCount = K
```

Pattern 3 usually wraps Pattern 1 or Pattern 2.

---

# 6. Pattern 4 Decision

Think Pattern 4 when brute force comes from:

```text
several independent choice dimensions
```

Example:

```text
A × B × C × D
```

rather than:

```text
take/skip across N elements
```

Try grouping:

```text
A+B
C+D
```

or more generally:

```text
f(group1)
g(group2)
```

Then join.

Example:

```text
a+b+c+d = X
```

becomes:

```text
(a+b) + (c+d) = X
```

Then:

```text
Pattern 4 + Pattern 1
```

---

# 7. Objective-Based Shortcut

After creating partial states:

```text
What does the objective ask?
│
├── Exact equality
│     → Pattern 1
│
├── Maximum <= boundary
│     → Pattern 2
│       predecessor / upperBound
│
├── Minimum >= boundary
│     → Pattern 2
│       successor / lowerBound
│
├── Closest to target
│     → Pattern 2
│       lowerBound + neighbors
│
└── Additional legality condition
      → Pattern 3 first
        then re-run this decision
```

---

# 8. State-Design Shortcut

```text
What must the opposite half know
about this partial solution?
│
├── Only total value
│     → state = value
│
├── How many choices were used
│     → state = (count, value)
│
├── Which choices were used
│     → state may need mask/indices
│
├── Resource usage
│     → state = (resourceUsed, value)
│
├── Multiple quotas
│     → state = (meta1, meta2, value)
│
└── Nothing compact seems sufficient
      → MITM may not be the best approach
```

---

# 9. Join-Structure Decision Tree

```text
I know which right states are legal.
Now what query do I need?
│
├── "Does exact X exist?"
│      → HashSet
│
├── "How many exact X?"
│      → HashMap frequency
│
├── "Largest value <= X?"
│      → sorted + upperBound - 1
│
├── "Smallest value >= X?"
│      → sorted + lowerBound
│
├── "Closest value to X?"
│      → lowerBound
│        check idx and idx-1
│
├── "Many monotonic pair queries?"
│      → consider two pointers
│
└── "Dynamic insertion/deletion?"
       → consider ordered dynamic structure
         rather than static array
```

---

# 10. Complexity Decision Tree

```text
Full search = 2^N?
│
├── 2^N already manageable
│     → don't over-engineer
│
└── too large
      ↓
2^(N/2) manageable?
│
├── No
│     → classical MITM insufficient
│
└── Yes
      → continue
```

For variable groups:

```text
Full search =
s1*s2*...*sk
```

Try splitting so:

```text
product(group1)
≈
product(group2)
```

Then confirm both fit memory/time.

---

# 11. Quick Pattern Table

| Question                             | Pattern       |
| ------------------------------------ | ------------- |
| `L + R == X`                         | Pattern 1     |
| Count pairs where `L + R == X`       | Pattern 1     |
| `max L+R <= X`                       | Pattern 2     |
| `min L+R >= X`                       | Pattern 2     |
| closest `L+R` to `X`                 | Pattern 2     |
| exactly `K` selected                 | Pattern 3     |
| same sum but different count matters | Pattern 3     |
| four independent arrays              | Pattern 4     |
| `A+B+C+D=X`                          | Pattern 4 + 1 |
| 4 groups, maximum total `<= X`       | Pattern 4 + 2 |
| exact K choices + closest sum        | Pattern 3 + 2 |

---

# 12. Problem-to-Pattern Examples

## CSES 1628

```text
Count subsets sum exactly X.
```

Reasoning:

```text
take/skip
+
N <= 40
+
exact complement
```

Result:

```text
Pattern 1
```

---

## AtCoder ABC184 F

```text
Maximum subset sum <= T.
```

Reasoning:

```text
take/skip
+
N <= 40
+
best state below boundary
```

Result:

```text
Pattern 2
```

---

## LeetCode 1755

```text
Closest subset sum to goal.
```

Reasoning:

```text
take/skip
+
N <= 40
+
nearest compatible state
```

Result:

```text
Pattern 2
```

---

## LeetCode 2035

```text
Choose exactly half the elements.
Minimize partition difference.
```

Reasoning:

```text
count affects compatibility
+
closest sum objective
```

Result:

```text
Pattern 3
+
Pattern 2
```

---

## LeetCode 454

```text
Choose one from four arrays.
Count sum = 0.
```

Reasoning:

```text
four independent dimensions
+
exact complement
```

Result:

```text
Pattern 4
+
Pattern 1
```

---

# 13. Pattern Confusion — Exact vs Closest

Ask:

```text
If X-L does not exist,
is this left state useless?
```

If YES:

```text
Pattern 1
```

If NO, and a nearby value may still be good:

```text
Pattern 2
```

This is the cleanest distinction.

---

# 14. Pattern Confusion — Pattern 2 vs Pattern 3

Ask:

```text
Can every right state potentially pair
with this left state?
```

If YES:

```text
Pattern 2
```

If NO because:

```text
count
quota
parity
resource use
```

must match:

```text
Pattern 3 first
```

Then Pattern 2 may operate inside the legal group.

---

# 15. Pattern Confusion — Pattern 1 vs Pattern 4

Pattern 1 answers:

```text
How do states meet?
```

Pattern 4 answers:

```text
Where did the two state spaces come from?
```

Therefore they can coexist.

Example:

```text
A+B+C+D=X
```

Pattern 4 creates:

```text
AB states
CD states
```

Pattern 1 joins them exactly.

---

# 16. Pattern Confusion — MITM vs DP

Ask:

```text
Which state-space dimension is manageable?
```

If:

```text
sum/capacity small
```

DP may be better.

If:

```text
N moderate
but numeric range huge
```

MITM may be better.

---

# 17. Pattern Confusion — MITM vs Ordinary Two Pointers

Suppose:

```text
find two elements summing to X
```

That is not automatically MITM.

There is no expensive partial combinatorial search.

But if:

```text
each "element"
is itself a generated subset state
```

then two pointers may be the **join mechanism inside MITM**.

---

# 18. Final Interview Decision Tree

This is the compact version worth memorizing.

```text
Combinatorial search too large?
│
└── Yes
     ↓
Does half/group enumeration become manageable?
│
└── Yes
     ↓
Can groups be summarized independently?
│
└── Yes
     ↓
Does compatibility require extra metadata?
│
├── Yes
│     → Pattern 3
│       bucket compatible states
│       ↓
│       exact or best?
│
└── No
      ↓
Did the search come from multiple
independent variable groups?
│
├── Yes
│     → Pattern 4
│       create grouped state spaces
│
└── No
      ↓
Now inspect the join
      │
      ├── exact complement
      │     → Pattern 1
      │
      └── best / nearest complement
            → Pattern 2
```

---

# 19. The Four Questions to Memorize

If you forget the entire tree, remember these:

```text
1. Can I halve/group the search space?

2. What does each partial solution need to remember?

3. Which partial states are allowed to meet?

4. Among allowed states, do I need:
   exact,
   largest/smallest,
   or closest?
```

Those four questions reconstruct almost the entire MITM map.

---

# 20. The Final Pattern Map

```text
Meet in the Middle
│
├── Pattern 1
│   Exact Complement
│
│   "Which exact state do I need?"
│
├── Pattern 2
│   Best Compatible State
│
│   "Which ordered state is best?"
│
├── Pattern 3
│   Constraint-Aware State
│
│   "Which states are legally compatible?"
│
└── Pattern 4
    Variable-Group MITM

    "How should I split the dimensions
     that created the search space?"
```

---

# 21. The One Sentence to Remember

> **First decide how to split the search, then decide what each partial state must remember, then decide which states may meet, and only then decide whether the join needs exact lookup or ordered optimization.**

That is the complete MITM decision system.
