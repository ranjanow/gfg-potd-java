# ➕ Ways to Express as Sum of Consecutives

## Problem Link
🔗 [Ways to Express as Sum of Consecutives – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/count-of-sum-of-consecutives3741/1)

---

## Difficulty
**Medium** | Accuracy: `24.63%` | Submissions: `23K+` | Points: `4` | Companies: `Visa` `Walmart` `LinkedIn`

---

## Tags
- Mathematics
- Number Theory
- Divisors
- Algorithms

---

## Problem Summary

Given a number `n`, find the number of ways to represent it as a **sum of 2 or more
consecutive natural numbers**.

**Constraints:** `1 ≤ n ≤ 10⁸`
**Expected:** Time `O(√n)` · Space `O(1)`

---

## Intuition

### Setting Up the Algebra

Suppose `n` can be written as the sum of `k` consecutive natural numbers starting
at `a` (where `a ≥ 1`, `k ≥ 2`):

```
n = a + (a+1) + (a+2) + ... + (a+k-1)
  = k×a + (0+1+2+...+(k-1))
  = k×a + k(k-1)/2
```

Rearranging for `a`:

```
a = [n - k(k-1)/2] / k
```

For a valid representation, `a` must be a **positive integer**. So for each `k ≥ 2`,
we check if `n - k(k-1)/2` is positive and divisible by `k`. Iterating `k` up to
`√n` and checking this directly gives a working `O(√n)` solution — but there's an
even more elegant closed-form connection.

### The Elegant Number-Theory Shortcut

**Key theorem:** *The number of ways to write `n` as a sum of 2 or more consecutive
natural numbers equals the number of **odd divisors** of `n`, minus `1`.*

Why? Every way of writing `n` as a sum of `k` consecutive integers corresponds
to a **factorization** `n = k × a + k(k-1)/2`. Working through the algebra, it
turns out each valid `k` corresponds bijectively to an **odd divisor of `n`**
(other than divisor `1` alone, which represents the trivial "sum of just `n`
itself" case that we exclude since we need 2+ terms).

### Verification with Example 1

`n = 10`. Odd divisors of `10` (i.e., `2 × 5`): just `1` and `5` → count = `2`.
`answer = 2 - 1 = 1` ✅ (matches: `10 = 1+2+3+4`)

### Verification with Example 2

`n = 15`. Since `15` is itself odd, **all** its divisors are odd: `1, 3, 5, 15` → count = `4`.
`answer = 4 - 1 = 3` ✅ (matches: `15=1+2+3+4+5`, `15=4+5+6`, `15=7+8`)

### Counting Odd Divisors Efficiently

To count only the **odd** divisors of `n`:
1. **Strip out all factors of 2** from `n`, leaving an odd number `m`
   (e.g., `n=40=2³×5` → `m=5`).
2. **Count all divisors of `m`** — since `m` is odd, every one of its divisors
   is automatically odd, and this count equals the odd-divisor count of the
   original `n`.
3. Divisor counting via trial division up to `√m` runs in `O(√n)`.

---

## Approach

1. While `n` is even: divide `n` by `2` (strip all factors of 2), leaving odd `m`.
2. Count divisors of `m` by trial division from `i = 1` to `√m`:
   - If `m % i == 0`: increment count by `1` if `i × i == m` (perfect square divisor
     counted once), else increment by `2` (both `i` and `m/i` are divisors).
3. Return `divisorCount - 1` (excluding the trivial single-term "sum").

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int getCount(int n)`

```java
class Solution {
    public int getCount(int n) {
        int m = n;

        // Strip all factors of 2, leaving the odd part of n
        while (m % 2 == 0) {
            m /= 2;
        }

        // Count divisors of the odd part (all of them are odd divisors of n)
        int divisorCount = 0;
        for (int i = 1; (long) i * i <= m; i++) {
            if (m % i == 0) {
                divisorCount++;               // i is a divisor
                if (i != m / i) {
                    divisorCount++;           // m/i is a distinct divisor
                }
            }
        }

        // Subtract 1 to exclude the trivial "sum of just n itself" case
        return divisorCount - 1;
    }
}
```

---

## Dry Run

### Example 1 — `n = 10` → Expected Output: `1`

**Step 1 — Strip factors of 2:** `10 → 5` (odd)

**Step 2 — Count divisors of 5:**

| i | m % i == 0? | i == m/i? | divisorCount |
|:-:|:-----------:|:---------:|:------------:|
| 1 | ✅ (5%1=0) | 1≠5 → +2 | 2 |

(loop stops once `i×i > 5`, i.e. after `i=2` since `2×2=4≤5` but `5%2≠0`; `i=3` gives `3×3=9>5`, loop ends)

`divisorCount = 2`

**Step 3:** `answer = 2 - 1 = 1` ✅

---

### Example 2 — `n = 15` → Expected Output: `3`

**Step 1 — Strip factors of 2:** `15` is already odd, no change.

**Step 2 — Count divisors of 15:**

| i | m % i == 0? | i == m/i? | divisorCount |
|:-:|:-----------:|:---------:|:------------:|
| 1 | ✅ (15%1=0) | 1≠15 → +2 | 2 |
| 2 | ❌ (15%2≠0) | — | 2 |
| 3 | ✅ (15%3=0) | 3≠5 → +2 | 4 |

(loop stops after `i=3` since `4×4=16>15`)

`divisorCount = 4`

**Step 3:** `answer = 4 - 1 = 3` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(√n)` | Stripping factors of 2 is `O(log n)`; trial division for divisors runs up to `√m ≤ √n` |
| **Space** | `O(1)` | Only a handful of integer variables — no arrays or extra structures |

For max constraints `n = 10⁸`: `√n = 10⁴` — trivially fast.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Theorem** | Ways to write `n` as sum of 2+ consecutive naturals = (odd divisor count of `n`) − 1 |
| **Why Strip Factors of 2 First** | Isolates the odd part of `n`; all divisors of an odd number are automatically odd |
| **Divisor Counting Trick** | Only need to check up to `√m` — pair `i` with `m/i`, careful not to double-count perfect square roots |
| **The "−1" Subtraction** | Excludes the trivial single-term case (`n` written as just itself, which isn't a "sum of 2+ terms") |
| **Interview Relevance** | Classic number-theory pattern — recognizing the odd-divisor connection is the key insight tested |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Count Divisors of a Number | GFG | The core divisor-counting subroutine used here |
| Sum of Consecutive Numbers | LeetCode #829 (Consecutive Numbers Sum) | Nearly identical problem, different framing |
| Perfect Squares | LeetCode #279 | Different problem, same "trial division to √n" technique |
| Prime Factorization | GFG | Related number-theory decomposition technique |

---

## Alternative Approaches

### 1. 🐢 Direct Algebraic Search — O(√n)

For each possible count `k` of consecutive terms (`k ≥ 2`), check if a valid
starting integer `a` exists using the derived formula.

```java
class Solution {
    public int getCount(int n) {
        int count = 0;

        // k = number of consecutive terms; k(k-1)/2 < n bounds k to O(sqrt(n))
        for (long k = 2; k * (k - 1) / 2 < n; k++) {
            long remaining = n - k * (k - 1) / 2;
            if (remaining > 0 && remaining % k == 0) {
                count++;
            }
        }
        return count;
    }
}
```

> ✅ **Verdict:** Also `O(√n)` and equally valid — a more "from first principles"
> approach that doesn't require knowing the odd-divisor theorem. Both this and
> the optimal solution run in the same asymptotic time.

---

### 2. ✅ Odd Divisor Counting (Optimal, Cleaner)

Described in full above.

> ✅ **Verdict:** Accepted — `O(√n)` time, `O(1)` space. Matches expected complexity exactly,
> and is more elegant once the number-theory connection is recognized.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Direct Algebraic Search | O(√n) | O(1) | ✅ Correct, doesn't need the theorem |
| **Odd Divisor Counting** | **O(√n)** | **O(1)** | ✅ **Cleaner, leverages number theory — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#mathematics` `#number-theory` `#divisors` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 10, 2026
