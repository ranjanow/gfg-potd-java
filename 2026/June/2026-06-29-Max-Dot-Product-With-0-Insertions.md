# 🔵 Max Dot Product with 0 Insertions

## Problem Link
🔗 [Max Dot Product with 0 Insertions – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/maximize-dot-product2649/1)

---

## Difficulty
**Medium** | Accuracy: `13.27%` | Submissions: `58K+` | Points: `4`

---

## Tags
- Dynamic Programming
- Arrays
- Subsequence
- Optimization

---

## Problem Summary

Given two arrays `a[]` (size `n`) and `b[]` (size `m`) of **positive integers**, where `m ≤ n`.
You may insert **zeros** anywhere into `b` to make its length equal to `n`.

Find the **maximum possible dot product** of the resulting equal-length arrays.

```
Dot Product = a[0]*b[0] + a[1]*b[1] + ... + a[n-1]*b[n-1]
```

**Constraints:**
- `1 ≤ m ≤ n ≤ 10³`
- `1 ≤ a[i], b[i] ≤ 10³`

---

## Intuition

### Zero-Insertion = Subsequence Selection

Inserting zeros at positions in `b` is equivalent to choosing **which m elements of `a`**
(maintaining relative order) get paired with `b[0], b[1], ..., b[m-1]`.
The remaining `n - m` elements of `a` are paired with the inserted zeros and contribute `0`.

**Example 1:** `a = [2,3,1,7,8]`, `b = [3,6,7]`
```
Pair a[1]=3 with b[0]=3,  a[3]=7 with b[1]=6,  a[4]=8 with b[2]=7
→ 3*3 + 7*6 + 8*7 = 9 + 42 + 56 = 107
```

So the problem reduces to:
> **Find the subsequence of `a` of length `m` that maximises the dot product with `b`.**

### Why Greedy Fails

We can't just greedily pair the largest elements — the **relative order** must be preserved.
Pairing `a[4]` with `b[0]` is invalid if we've already used `b[1]` for an earlier index of `a`.

### DP State

```
dp[i][j] = maximum dot product achievable by:
           - pairing exactly j elements of b  (b[0..j-1])
           - using only the first i elements of a  (a[0..i-1])
```

### Transitions

At each `a[i-1]`:
1. **Skip** it (pair with inserted zero, contributes 0):
   ```
   dp[i][j] = dp[i-1][j]
   ```
2. **Pair** `a[i-1]` with `b[j-1]` (the j-th element of b):
   ```
   dp[i][j] = dp[i-1][j-1] + a[i-1] * b[j-1]
   ```

Take the **maximum** of both options.

### Base Cases

```
dp[i][0] = 0         for all i   → 0 elements of b matched → contribution is 0
dp[0][j] = -∞        for j > 0   → impossible: can't match j b-elements with 0 a-elements
```

**Why `-∞` and not `0`?** If we initialise impossible states to `0`, the transition
`dp[i-1][j-1] + a[i-1]*b[j-1]` would incorrectly add to a "zero" that represents
an infeasible state, giving wrong answers. Using `-∞` ensures these states are
never chosen as optimal.

### Answer

```
dp[n][m]
```

---

## Approach

1. Create `int[][] dp` of size `(n+1) × (m+1)`, fill all with `NEG_INF = Integer.MIN_VALUE / 2`.
2. Set `dp[i][0] = 0` for all `i` from `0` to `n`.
3. For `i` from `1` to `n`, for `j` from `1` to `min(i, m)`:
   - `dp[i][j] = dp[i-1][j]`  ← skip `a[i-1]`
   - `dp[i][j] = max(dp[i][j],  dp[i-1][j-1] + a[i-1]*b[j-1])`  ← pair `a[i-1]` with `b[j-1]`
4. Return `dp[n][m]`.

---

## Java Solution

> ⚠️ **Verify the exact method name in the GFG editor before submitting.**
> Based on URL slug `maximize-dot-product2649`, likely `maxDotProduct`.

```java
import java.util.Arrays;

class Solution {
    public int maxDotProduct(int[] a, int[] b) {
        int n = a.length, m = b.length;
        int NEG_INF = Integer.MIN_VALUE / 2; // avoid overflow when adding

        // dp[i][j] = max dot product pairing b[0..j-1] with a subsequence of a[0..i-1]
        int[][] dp = new int[n + 1][m + 1];

        // Initialize all cells as impossible (-∞)
        for (int[] row : dp) Arrays.fill(row, NEG_INF);

        // Base case: 0 elements of b matched → 0 contribution regardless of i
        for (int i = 0; i <= n; i++) dp[i][0] = 0;

        for (int i = 1; i <= n; i++) {
            // j can be at most i (can't match more b-elements than a-elements processed)
            for (int j = 1; j <= Math.min(i, m); j++) {

                // Option 1: skip a[i-1] (pair it with an inserted zero)
                dp[i][j] = dp[i - 1][j];

                // Option 2: pair a[i-1] with b[j-1]
                dp[i][j] = Math.max(dp[i][j], dp[i - 1][j - 1] + a[i - 1] * b[j - 1]);
            }
        }

        return dp[n][m];
    }
}
```

---

## Dry Run

### Example 1 — `a = [2,3,1,7,8]`, `b = [3,6,7]`  (n=5, m=3)

**DP Table `dp[i][j]`** (NEG_INF shown as `—`):

|  i \ j  |  0  |   1   |   2   |   3   |
|:-------:|:---:|:-----:|:-----:|:-----:|
|  **0**  |  0  |   —   |   —   |   —   |
|  **1**  |  0  |   6   |   —   |   —   |
|  **2**  |  0  |   9   |  24   |   —   |
|  **3**  |  0  |   9   |  24   |  31   |
|  **4**  |  0  |  21   |  51   |  73   |
|  **5**  |  0  |  24   |  69   | **107** |

**Key steps:**
| i | j | Skip → dp[i-1][j] | Pair → dp[i-1][j-1]+a[i-1]*b[j-1] | dp[i][j] |
|:-:|:-:|:-----------------:|:-----------------------------------:|:--------:|
| 1 | 1 | — | 0 + 2×3 = 6 | **6** |
| 2 | 1 | 6 | 0 + 3×3 = 9 | **9** |
| 2 | 2 | — | 6 + 3×6 = 24 | **24** |
| 4 | 2 | 24 | 9 + 7×6 = 51 | **51** |
| 4 | 3 | 31 | 24 + 7×7 = 73 | **73** |
| 5 | 3 | 73 | 51 + 8×7 = **107** | **107** |

**Output:** `107` ✅

**Optimal pairing:** `a[1]=3↔b[0]=3, a[3]=7↔b[1]=6, a[4]=8↔b[2]=7` → `9+42+56=107` ✓

---

### Example 2 — `a = [1,2,3]`, `b = [4]`  (n=3, m=1)

| i | j=1 | Computation |
|:-:|:---:|---|
| 1 | 4   | max(—, 0+1×4) |
| 2 | 8   | max(4, 0+2×4) |
| 3 | **12** | max(8, 0+3×4) |

**Output:** `12` ✅ — pair `a[2]=3` with `b[0]=4`, rest zeroed out ✓

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n × m)` | Two nested loops: i from 1..n, j from 1..min(i,m) |
| **Space** | `O(n × m)` | 2D DP table of size `(n+1) × (m+1)` |

For max constraints `n = m = 10³`: `10⁶` operations, `~4 MB` table — well within limits.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | 2D Subsequence DP (LCS-style) |
| **Zero Insertion = Subsequence** | Inserting zeros into b ≡ choosing m elements from a in order |
| **`-∞` Initialisation** | Critical — prevents infeasible states from polluting valid transitions |
| **`Integer.MIN_VALUE / 2`** | Safe sentinel: even after adding max product (10⁶), stays very negative |
| **`j ≤ min(i, m)` bound** | Optimisation — can't use more b-elements than a-elements processed |
| **Accuracy 13.27%** | Low accuracy due to wrong initialisation (using 0 instead of -∞ for impossible states) |

### Common Mistake That Causes Wrong Answers

```java
// ❌ WRONG — initialising impossible states to 0
int[][] dp = new int[n + 1][m + 1]; // default Java value = 0

// Transition at i=1, j=1 with a[0]=2, b[0]=3:
// dp[1][1] = max(dp[0][1]=0, dp[0][0]=0 + 2*3=6)
// → picks 6, looks correct HERE, but dp[0][1]=0 is actually an impossible state
// → for inputs where using dp[0][j] gives a higher value, it produces wrong output
```

```java
// ✅ CORRECT — impossible states = -∞
Arrays.fill(row, Integer.MIN_VALUE / 2);
for (int i = 0; i <= n; i++) dp[i][0] = 0; // only base cases are 0
```

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Longest Common Subsequence | GFG / LC | Same DP table structure |
| Minimum Ascii Delete Sum | LeetCode #712 | 2D DP on two arrays |
| Maximum Dot Product of Two Subsequences | LeetCode #1458 | Exact same problem (can be negative elements) |
| Edit Distance | LeetCode #72 | 2D DP on strings |

---

## Alternative Approaches

### 1. 🐢 Brute Force — O(C(n,m) × m)

Enumerate all subsequences of `a` of length `m` and compute dot product with `b`.

```java
// TLE — C(1000, 500) is astronomically large
```

> ❌ **Verdict:** TLE — C(n, m) can be 10^300 for n=m=1000.

---

### 2. ✅ 2D DP — Bottom Up (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — O(n×m) time/space.

---

### 3. 💡 Space-Optimised DP — O(m) Space

Since `dp[i][j]` only depends on row `i-1`, we can use two 1D arrays.

```java
class Solution {
    public int maxDotProduct(int[] a, int[] b) {
        int n = a.length, m = b.length;
        int NEG_INF = Integer.MIN_VALUE / 2;

        int[] prev = new int[m + 1];
        Arrays.fill(prev, NEG_INF);
        prev[0] = 0;

        for (int i = 1; i <= n; i++) {
            int[] curr = new int[m + 1];
            Arrays.fill(curr, NEG_INF);
            curr[0] = 0;

            for (int j = 1; j <= Math.min(i, m); j++) {
                curr[j] = prev[j]; // skip a[i-1]
                if (prev[j - 1] != NEG_INF) {
                    curr[j] = Math.max(curr[j], prev[j - 1] + a[i - 1] * b[j - 1]);
                }
            }
            prev = curr;
        }

        return prev[m];
    }
}
```

> ✅ **Verdict:** Accepted — O(n×m) time, **O(m) space**. Best for memory.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(C(n,m)×m) | O(m) | ❌ TLE |
| **2D DP** | **O(n×m)** | **O(n×m)** | ✅ **GFG Expected** |
| Space-Optimised 1D DP | O(n×m) | O(m) | ✅ Same speed, less memory |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#dynamic-programming` `#arrays` `#subsequence` `#dot-product` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** June 29, 2026
