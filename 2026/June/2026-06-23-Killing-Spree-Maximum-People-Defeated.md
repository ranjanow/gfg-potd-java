# 🗡️ Killing Spree (Maximum Number of People Defeated)

## Problem Link
🔗 [Killing Spree – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/killing-spree3020/1)

---

## Difficulty
**Easy** | Topic: `Mathematical` | Points: `2`

---

## Tags
- Mathematics
- Binary Search
- Sum of Squares
- Greedy
- Number Theory

---

## Problem Summary

There are **infinitely many people** standing in a row, indexed from `1`.
The person at index `i` has strength `i²`.

You are given a strength `p`. You can defeat a person with strength `x` **only if** `p ≥ x`,
after which your strength **decreases by x**.

Find the **maximum number of people** you can defeat sequentially starting from index `1`.

**Key Formula:**
To defeat the first `k` people, you need:
```
1² + 2² + 3² + ... + k²  =  k(k+1)(2k+1) / 6
```

---

## Intuition

### Key Observations

1. **You must defeat people in order** — person 1 first, then 2, then 3, etc.
   There's no skipping or choosing out of order.

2. **Prefix sum of squares is monotonically increasing.**
   If you can defeat `k` people, defeating `k-1` is always possible.
   If you can't defeat `k` people, you can't defeat `k+1` either.
   → This **monotonicity** is the perfect trigger for **Binary Search**.

3. **Simulation works too** — but binary search is strictly faster.
   Why does simulation feel natural here?
   Because each defeat is independent: subtract `i²`, increment counter, move on.

4. **Overflow danger is real.**
   For a safe upper bound of `k ≈ 1500`, the intermediate product
   `k × (k+1) × (2k+1)` ≈ `6.75 × 10⁹` — this **overflows `int`**.
   Always use `long` in the sum formula.

### Why Binary Search Works

Define `f(k) = 1² + 2² + ... + k² = k(k+1)(2k+1)/6`

We want the **maximum k** such that `f(k) ≤ p`.

- `f(k)` is strictly increasing → binary search for the largest valid `k`.
- Search range: `[0, 1500]` — since `f(1500) ≈ 1.13 × 10⁹` safely covers `p ≤ 10⁹`.

---

## Approach

### Optimal — Binary Search on Answer

1. Set `low = 0`, `high = 1500`, `ans = 0`.
2. Compute `mid = (low + high) / 2`.
3. Compute `f(mid) = mid × (mid+1) × (2×mid+1) / 6` using `long` to prevent overflow.
4. If `f(mid) ≤ p` → `mid` people can be defeated. Save `ans = mid`, search right (`low = mid + 1`).
5. If `f(mid) > p` → too many people. Search left (`high = mid - 1`).
6. Repeat until `low > high`. Return `ans`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int maxPeopleDefeated(int p)`

```java
class Solution {

    // Sum of squares formula: 1² + 2² + ... + n² = n(n+1)(2n+1)/6
    // Uses 'long' to prevent integer overflow for large n (e.g., n = 1500)
    private long sumOfSquares(long n) {
        return n * (n + 1) * (2 * n + 1) / 6;
    }

    public int maxPeopleDefeated(int p) {
        int low = 0;
        int high = 1500; // f(1500) ≈ 1.13×10⁹ > max possible p (10⁹)
        int ans = 0;

        while (low <= high) {
            int mid = low + (high - low) / 2;

            if (sumOfSquares(mid) <= (long) p) {
                // We can defeat 'mid' people; try to do more
                ans = mid;
                low = mid + 1;
            } else {
                // Defeating 'mid' people requires more strength than we have
                high = mid - 1;
            }
        }

        return ans;
    }
}
```

---

## Dry Run

### Example 1 — `p = 14`

**Sum of squares lookup:**
| k | f(k) = 1²+2²+...+k² | f(k) ≤ 14? |
|:-:|:--------------------:|:----------:|
| 1 | 1                    | ✅          |
| 2 | 1 + 4 = 5            | ✅          |
| 3 | 5 + 9 = 14           | ✅          |
| 4 | 14 + 16 = 30         | ❌          |

**Binary Search trace** (`low=0`, `high=1500`):

| Step | low  | high | mid  | f(mid)          | Action          | ans |
|:----:|:----:|:----:|:----:|:---------------:|:---------------:|:---:|
|  1   |  0   | 1500 | 750  | 141,188,250 > 14| high = 749      |  0  |
|  2   |  0   | 749  | 374  | 17,512,775 > 14 | high = 373      |  0  |
|  3   |  0   | 373  | 186  | 2,163,741 > 14  | high = 185      |  0  |
|  ...  |  ... |  ... |  ... |      ...        |      ...        | ... |
|  k   |  0   |  6   |  3   | **14** ≤ 14      | ans=3, low=4   |  3  |
|  k+1 |  4   |  6   |  5   | 55 > 14         | high=4          |  3  |
|  k+2 |  4   |  4   |  4   | 30 > 14         | high=3          |  3  |
|  end |  4   |  3   |  —   | low > high      | **STOP**        |  **3** |

**Output:** `3` ✅

---

### Example 2 — `p = 10`

- f(2) = 1 + 4 = 5 ≤ 10 → can defeat 2 people ✅
- f(3) = 1 + 4 + 9 = 14 > 10 → cannot defeat 3 people ❌

**Output:** `2` ✅

---

## Complexity Analysis

### Optimal Solution (Binary Search)

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(log p)` | Binary search over range `[0, 1500]` ≈ log₂(1500) ≈ 11 iterations; f(k) computed in O(1) |
| **Space** | `O(1)` | Only a few integer variables used |

### Naive Solution (Simulation)

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(p^(1/3))` | The answer k satisfies k³/3 ≈ p (from sum formula), so k ≈ (3p)^(1/3) iterations |
| **Space** | `O(1)` | No extra data structures |

> 📌 **Note:** The naive approach is sometimes quoted as `O(√p)` (a loose upper bound),
> but the tighter and correct bound is `O(p^(1/3))` since
> `f(k) = k(k+1)(2k+1)/6 ≈ k³/3`, giving `k ≈ (3p)^(1/3)`.
> For `p = 10⁹`, this means at most **~1442 iterations**, not ~31,623 (√p).

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Binary Search on Answer |
| **Mathematical Insight** | Sum of first n squares = n(n+1)(2n+1)/6 |
| **Overflow Trap** | Always compute the sum in `long`; intermediate n(n+1)(2n+1) overflows `int` |
| **Monotonicity** | f(k) is strictly increasing → binary search is valid and optimal |
| **Upper Bound Derivation** | k³/3 ≤ p → k ≤ (3p)^(1/3) → k ≤ ~1442 for p ≤ 10⁹ |

### Sum of Squares Formula — Quick Derivation Reminder
```
Use telescoping with (k+1)³ - k³ = 3k² + 3k + 1
Summing from k=1 to n and solving gives:
Σk² = n(n+1)(2n+1)/6
```

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Count pairs with sum ≤ K | GFG | Binary search on count |
| Kth Smallest Element | LeetCode #378 | Binary search on answer |
| Capacity to Ship Packages | LeetCode #1011 | Binary search + monotone function |
| Sqrt(x) | LeetCode #69 | Binary search on integer square root |

---

## Alternative Approaches

### 1. 🐢 Naive Simulation — `O(p^(1/3))` Time, `O(1)` Space

Defeat people one by one. Keep subtracting `i²` from `p` while `p ≥ i²`.

```java
// Naive Simulation — Accepted but slower
class Solution {
    public int maxPeopleDefeated(int p) {
        int count = 0;
        int i = 1;

        // Subtract strength of each person while we have enough strength left
        while ((long) i * i <= p) {
            p -= i * i;
            count++;
            i++;
        }

        return count;
    }
}
```

> ✅ **Verdict:** Accepted — runs in ~1442 iterations max for `p = 10⁹`.
> Clean and simple. Preferred if the constraint allows it and you want minimal code.

---

### 2. ✅ Binary Search (Optimal) — `O(log p)` Time, `O(1)` Space

Binary search on `k` (answer) using the closed-form sum of squares formula.
Described fully above.

> ✅ **Verdict:** Accepted — ~11 iterations for any `p ≤ 10⁹`. Strictly superior in theory.

---

### Comparison Table

| Approach | Time | Space | Code Simplicity | Notes |
|---|---|---|---|---|
| Naive Simulation | O(p^(1/3)) | O(1) | ⭐⭐⭐⭐⭐ | Most readable |
| Binary Search | O(log p) | O(1) | ⭐⭐⭐⭐ | Optimal, needs formula |

> 💡 **Interview Tip:** For **Easy** problems in interviews, demonstrate awareness of
> both approaches. Implement the cleaner simulation but explain why binary search
> is theoretically superior and when it matters (e.g., p up to 10^18).

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#binary-search` `#mathematics` `#sum-of-squares` `#easy` `#number-theory` `#competitive-programming`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** June 23, 2026
