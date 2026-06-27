# 🔲 Ways to Tile the Floor

## Problem Link
🔗 [Ways to Tile the Floor – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/count-the-number-of-ways-to-tile-the-floor-of-size-n-x-m-using-1-x-m-size-tiles0509/1)

---

## Difficulty
**Medium** | Accuracy: `39.12%` | Submissions: `14K+` | Points: `4`

---

## Tags
- Dynamic Programming
- Mathematics
- Linear Recurrence
- Modular Arithmetic

---

## Problem Summary

Given an `n × m` floor and an unlimited supply of `1 × m` tiles, find the total
number of ways to completely tile the floor with **no overlaps and no uncovered cells**.

Each tile can be placed in exactly **one** of two orientations:
- **Horizontal:** covers **1 row × m columns** (fills an entire row)
- **Vertical:** covers **m rows × 1 column** (fills one column across m rows)

Return the total count **modulo 10⁹+7**.

**Note:** `n` and `m` are positive integers with `m ≥ 2`.

**Constraints:**
- `1 ≤ n ≤ 10⁵`
- `2 ≤ m ≤ 10⁵`

---

## Intuition

### Key Observations

1. **A horizontal tile fills exactly one row entirely.**
   Since the tile is `1 × m` and the floor is `n × m`, one horizontal tile covers
   all `m` columns of a single row — no partial rows possible.

2. **Vertical tiles must always come in groups of m.**
   A vertical tile covers `m rows × 1 column`. To avoid uncovered cells in that
   column block, we must place exactly `m` vertical tiles side-by-side (one per
   column), covering an `m × m` block of the floor.

3. **When at row `i`, only two choices exist:**
   - Place **1 horizontal tile** → row `i` is done → remaining: `dp[i-1]` ways.
   - Place **m vertical tiles** (one per column, covering rows `i-m+1` to `i`) →
     valid only when `i ≥ m` → remaining: `dp[i-m]` ways.

4. **For `i < m`: vertical is impossible.** We can only use horizontal tiles.
   Every row must have exactly 1 horizontal tile → exactly **1 way**.

5. **This produces a generalised Fibonacci recurrence:**
   - For `m = 2`: classic Fibonacci (1, 1, 2, 3, 5, 8, ...)
   - For `m = 3`: tribonacci-like (1, 1, 1, 2, 3, 4, 7, ...)
   - For `m = 4` (example 1): 1, 1, 1, 1, 2, 3, 4, 5, 7, ...

### Recurrence

```
dp[0]   = 1              (base: empty floor — 1 way to tile nothing)
dp[i]   = 1              for 1 ≤ i < m  (only horizontal tiles possible)
dp[i]   = dp[i-1]        horizontal tile at row i
        + dp[i-m]        m vertical tiles covering rows i-m+1..i
                         for i ≥ m
```

---

## Approach

1. Allocate `long[] dp` of size `n + 1`.
2. Set `dp[0] = 1`.
3. For `i` from `1` to `n`:
   - Set `dp[i] = dp[i-1] % MOD` (always: horizontal tile option).
   - If `i ≥ m`: `dp[i] = (dp[i] + dp[i-m]) % MOD` (vertical tiles option).
4. Return `(int) dp[n]`.

---

## Java Solution

> ⚠️ **Always check the exact method name in the GFG editor before submitting.**
> Based on the URL slug and GFG conventions, `countWays` is the most likely name
> — but confirm it visually to avoid a compilation error.

```java
class Solution {
    public int countWays(int n, int m) {
        int MOD = 1_000_000_007;

        // dp[i] = number of ways to tile a floor of i rows × m columns
        long[] dp = new long[n + 1];
        dp[0] = 1; // base case: empty floor has exactly 1 tiling (do nothing)

        for (int i = 1; i <= n; i++) {
            // Option 1: place a horizontal tile at row i (always valid)
            dp[i] = dp[i - 1] % MOD;

            // Option 2: place m vertical tiles covering rows i-m+1 to i
            //           (only valid when we have at least m rows available)
            if (i >= m) {
                dp[i] = (dp[i] + dp[i - m]) % MOD;
            }
        }

        return (int) dp[n];
    }
}
```

---

## Dry Run

### Example 1 — `n = 4, m = 4`

| `i` | Condition | `dp[i-1]` | `dp[i-m]` | `dp[i]` |
|:---:|:---------:|:---------:|:---------:|:-------:|
|  0  | base      |     —     |     —     |  **1**  |
|  1  | i < m     |     1     |     —     |  **1**  |
|  2  | i < m     |     1     |     —     |  **1**  |
|  3  | i < m     |     1     |     —     |  **1**  |
|  4  | i == m ✅ |     1     |     1     |  **2**  |

**Output:** `2` ✅

**Explanation:**
- Tiling 1: 4 horizontal tiles (1 per row)
- Tiling 2: 4 vertical tiles (1 per column, all spanning 4 rows)

---

### Example 2 — `n = 2, m = 3`

| `i` | Condition | `dp[i-1]` | `dp[i-m]` | `dp[i]` |
|:---:|:---------:|:---------:|:---------:|:-------:|
|  0  | base      |     —     |     —     |  **1**  |
|  1  | i < m     |     1     |     —     |  **1**  |
|  2  | i < m     |     1     |     —     |  **1**  |

**Output:** `1` ✅

**Explanation:** `n = 2 < m = 3` → vertical tiles impossible (need 3 rows, have 2).
Only choice: 2 horizontal tiles, one per row.

---

### Bonus — `m = 2, n = 6` (Fibonacci pattern)

| `i` | `dp[i]` = `dp[i-1]` + `dp[i-2]` |
|:---:|:--------------------------------:|
|  0  | 1                                |
|  1  | 1                                |
|  2  | 1 + 1 = **2**                    |
|  3  | 2 + 1 = **3**                    |
|  4  | 3 + 2 = **5**                    |
|  5  | 5 + 3 = **8**                    |
|  6  | 8 + 5 = **13**                   |

For `m = 2`, this is exactly the **Fibonacci sequence** ✓

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n)` | Single loop from `1` to `n`; each iteration is O(1) |
| **Space** | `O(n)` | DP array of size `n + 1` |

For max constraints `n = 10⁵`:
- Loop runs `10⁵` times ✅
- Array uses `~800 KB` of memory (long[] of size 10⁵) ✅

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Linear DP with generalised Fibonacci recurrence |
| **Two Choices Only** | Horizontal (always) or m vertical tiles as a block (when i ≥ m) |
| **No Partial Fills** | Horizontal fills a full row; verticals must be placed as a full m×m block |
| **Modulo Trap** | Apply `% MOD` at every addition to prevent `long` overflow |
| **Special Case** | When `n < m`, answer is always `1` (only horizontal tilings possible) |
| **m = 2 Pattern** | Reduces to classic Fibonacci — great sanity check |
| **Interview Relevance** | Generalisation of "Tiling with 1×2 Dominoes" — extremely common |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Tiling with 2×1 Dominoes | GFG / LC | Special case: m=2, gives Fibonacci |
| Climbing Stairs (LeetCode #70) | LeetCode | Same recurrence structure |
| Frog Jump | GFG | Linear DP with similar state transition |
| Count Ways to Reach nth Stair | GFG | Direct analogue of this problem |

---

## Alternative Approaches

### 1. 🐢 Recursion Without Memoization — `O(2ⁿ)` Time

```java
// TLE — exponential without memoization
int solve(int i, int m) {
    if (i == 0) return 1;
    if (i < m)  return solve(i - 1, m);
    return solve(i - 1, m) + solve(i - m, m);
}
```
> ❌ **Verdict:** TLE for large n — exponential recomputation.

---

### 2. ✅ Bottom-Up DP with O(1) Space (when m is known)

If we only need the last `m` values instead of the full array,
we can use a circular buffer of size `m`. But for GFG constraints
(n, m both up to 10⁵ and independent), the standard O(n) array is cleaner.

```java
// O(m) space — sliding window over last m values
class Solution {
    public int countWays(int n, int m) {
        int MOD = 1_000_000_007;
        long[] window = new long[m + 1];
        window[0] = 1;  // dp[0]

        for (int i = 1; i <= n; i++) {
            int idx = i % (m + 1);
            int prevIdx = (i - 1 + m + 1) % (m + 1);
            int mBackIdx = ((i - m) % (m + 1) + m + 1) % (m + 1);

            window[idx] = window[prevIdx];
            if (i >= m) {
                window[idx] = (window[idx] + window[mBackIdx]) % MOD;
            }
        }

        return (int) window[n % (m + 1)];
    }
}
```
> ⚠️ **Note:** The circular indexing is error-prone. For GFG, use the O(n) array
> approach — it's cleaner, accepted, and matches expected auxiliary space.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Recursion (no memo) | O(2ⁿ) | O(n) | ❌ TLE |
| **Bottom-Up DP (O(n) array)** | **O(n)** | **O(n)** | ✅ **GFG Expected — use this** |
| Bottom-Up DP (O(m) window) | O(n) | O(m) | ✅ Correct but tricky indexing |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#dynamic-programming` `#tiling` `#fibonacci` `#linear-dp` `#medium` `#modular-arithmetic`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** June 27, 2026
