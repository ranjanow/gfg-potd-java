# 🧵 Count Matching Subsequences

## Problem Link
🔗 [Count Matching Subsequences – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/find-number-of-times-a-string-occurs-as-a-subsequence3020/1)

---

## Difficulty
**Medium** | Accuracy: `57.65%` | Submissions: `43K+` | Points: `4` | Company: `Accolite`

---

## Tags
- Dynamic Programming
- Strings
- 2D DP
- Subsequence
- Modular Arithmetic

---

## Problem Summary

Given two strings `s1` and `s2`, count the number of **distinct subsequences** of `s1`
that are exactly equal to `s2`. Return the count **modulo 1e9+7**.

A subsequence is formed by deleting zero or more characters from `s1` **without changing
the relative order** of the remaining characters.

**Constraints:**
- `1 ≤ s1.size(), s2.size() ≤ 10³`
- Expected Time: `O(n × m)` | Expected Space: `O(n × m)`

---

## Intuition

### Key Observations

1. **Overlapping subproblems exist.**
   To count how many ways `s2[0..j]` can be formed from `s1[0..i]`,
   the answer depends on smaller sub-answers — classic **DP**.

2. **At each character of s1, we have two choices:**
   - **Skip** `s1[i]` → the count stays the same as `dp[i-1][j]`.
   - **Use** `s1[i]` (only valid when `s1[i] == s2[j]`) → add `dp[i-1][j-1]` ways
     (all the ways we had matched `s2[0..j-1]` before this character).

3. **Why naive recursion is slow:**
   Without memoization, we recompute the same `(i, j)` pairs exponentially.
   Memoizing all `n × m` pairs gives us the optimal `O(n × m)` solution.

4. **Modulo 1e9+7** is required because counts can grow astronomically
   (e.g., a string of 1000 identical characters has C(1000, 500) subsequences).
   Use `long` for all intermediate values before casting to `int`.

### DP State Definition

```
dp[i][j] = number of distinct ways to form s2[0..j-1]
            using characters from s1[0..i-1]
```

### Recurrence

```
if s1[i-1] == s2[j-1]:
    dp[i][j] = dp[i-1][j-1] + dp[i-1][j]   ← use it OR skip it
else:
    dp[i][j] = dp[i-1][j]                   ← must skip s1[i-1]
```

### Base Cases

```
dp[i][0] = 1   for all i   → empty s2 is always a valid subsequence (take nothing)
dp[0][j] = 0   for j > 0  → can't form non-empty s2 from empty s1
```

---

## Approach

1. Let `n = s1.length()`, `m = s2.length()`, `MOD = 1_000_000_007`.
2. Create `long[][] dp` of size `(n+1) × (m+1)`.
3. Fill base case: `dp[i][0] = 1` for all `i` from `0` to `n`.
4. For each `i` from `1` to `n`:
   - For each `j` from `1` to `m`:
     - If `s1.charAt(i-1) == s2.charAt(j-1)`:
       `dp[i][j] = (dp[i-1][j-1] + dp[i-1][j]) % MOD`
     - Else:
       `dp[i][j] = dp[i-1][j]`
5. Return `(int) dp[n][m]`.

---

## Java Solution

> ⚠️ **Check the method name** in the GFG editor before submitting.
> Based on URL slug and GFG conventions, `countWays` is the most likely name.

```java
class Solution {
    public int countWays(String s1, String s2) {
        int MOD = 1_000_000_007;
        int n = s1.length(), m = s2.length();

        // dp[i][j] = ways to form s2[0..j-1] from s1[0..i-1]
        long[][] dp = new long[n + 1][m + 1];

        // Base case: empty s2 is always matched (1 way: take nothing)
        for (int i = 0; i <= n; i++) dp[i][0] = 1;

        // dp[0][j] = 0 for j > 0 (default, already 0)

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    // Use s1[i-1] to match s2[j-1]  +  skip s1[i-1]
                    dp[i][j] = (dp[i - 1][j - 1] + dp[i - 1][j]) % MOD;
                } else {
                    // Characters don't match — must skip s1[i-1]
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }

        return (int) dp[n][m];
    }
}
```

---

## Dry Run

**Input:** `s1 = "aaa"`, `s2 = "aa"`
*(Chosen because it clearly shows both match branches and proves the formula)*

`n = 3`, `m = 2`

**DP Table** (`dp[i][j]`):

|   | `""` | `"a"` (j=1) | `"aa"` (j=2) |
|:-:|:----:|:-----------:|:------------:|
| `""` (i=0) | 1 | 0 | 0 |
| `"a"` (i=1) | 1 | **1** | 0 |
| `"aa"` (i=2) | 1 | **2** | **1** |
| `"aaa"` (i=3) | 1 | **3** | **3** |

**Step-by-step:**

| i | j | s1[i-1] | s2[j-1] | Match? | dp[i][j] = | Result |
|:-:|:-:|:-------:|:-------:|:------:|-----------|:------:|
| 1 | 1 | 'a' | 'a' | ✅ | dp[0][0] + dp[0][1] = 1 + 0 | **1** |
| 1 | 2 | 'a' | 'a' | ✅ | dp[0][1] + dp[0][2] = 0 + 0 | **0** |
| 2 | 1 | 'a' | 'a' | ✅ | dp[1][0] + dp[1][1] = 1 + 1 | **2** |
| 2 | 2 | 'a' | 'a' | ✅ | dp[1][1] + dp[1][2] = 1 + 0 | **1** |
| 3 | 1 | 'a' | 'a' | ✅ | dp[2][0] + dp[2][1] = 1 + 2 | **3** |
| 3 | 2 | 'a' | 'a' | ✅ | dp[2][1] + dp[2][2] = 2 + 1 | **3** |

**Output:** `dp[3][2] = 3` ✅

**Verification:** From "aaa", pairs to form "aa": (0,1), (0,2), (1,2) = C(3,2) = **3** ✓

---

**Checking Example 1:** `s1 = "geeksforgeeks"`, `s2 = "gks"` → `4` ✅
**Checking Example 2:** `s1 = "problemoftheday"`, `s2 = "geek"` → `0` ✅
*(No 'g' in s1 → dp[n][1] = 0 → dp[n][4] = 0)*

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n × m)` | Two nested loops over all `(i, j)` pairs |
| **Space** | `O(n × m)` | 2D DP table of size `(n+1) × (m+1)` |

Where `n = s1.length()` and `m = s2.length()`.
For max constraints: `n = m = 10³` → table size = `10⁶` cells — well within limits.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | 2D Dynamic Programming on strings |
| **State** | `dp[i][j]` = ways to form `s2[0..j-1]` from `s1[0..i-1]` |
| **Critical Base Case** | `dp[i][0] = 1` for all `i` — empty s2 is always achievable |
| **Modulo Trap** | Apply `% MOD` at every addition — never at the end only |
| **`long` vs `int`** | Use `long` for dp array; sum of two `long` values won't overflow |
| **Interview Frequency** | Very common in top-tier interviews (Amazon, Google, Accolite) |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Distinct Subsequences (LeetCode #115) | LeetCode | Exact same problem |
| Longest Common Subsequence | GFG / LC | DP on 2 strings, same state space |
| Edit Distance (LeetCode #72) | LeetCode | DP on 2 strings, similar recurrence |
| Is Subsequence (LeetCode #392) | LeetCode | Simpler version — just check existence |

---

## Alternative Approaches

### 1. 🐢 Recursion + Memoization (Top-Down DP)

Same recurrence as above, implemented recursively with a memo table.

```java
class Solution {
    private int MOD = 1_000_000_007;
    private long[][] memo;

    public int countWays(String s1, String s2) {
        int n = s1.length(), m = s2.length();
        memo = new long[n][m];
        for (long[] row : memo) java.util.Arrays.fill(row, -1);
        return (int) solve(s1, s2, n - 1, m - 1);
    }

    private long solve(String s1, String s2, int i, int j) {
        if (j < 0) return 1;  // matched all of s2
        if (i < 0) return 0;  // exhausted s1, s2 still remains

        if (memo[i][j] != -1) return memo[i][j];

        if (s1.charAt(i) == s2.charAt(j)) {
            memo[i][j] = (solve(s1, s2, i - 1, j - 1)
                        + solve(s1, s2, i - 1, j)) % MOD;
        } else {
            memo[i][j] = solve(s1, s2, i - 1, j);
        }
        return memo[i][j];
    }
}
```

> ✅ **Verdict:** Accepted — same O(n×m) time/space. Slightly higher constant due to
> recursion overhead but cleaner to derive mentally.

---

### 2. ✅ Space-Optimised DP — `O(m)` Space

Since `dp[i][j]` only depends on row `i-1`, we can compress to a 1D array.
Traverse `j` **right to left** to avoid overwriting values we still need.

```java
class Solution {
    public int countWays(String s1, String s2) {
        int MOD = 1_000_000_007;
        int n = s1.length(), m = s2.length();

        long[] dp = new long[m + 1];
        dp[0] = 1; // base case: empty s2 always matchable

        for (int i = 0; i < n; i++) {
            // Right to left: ensures dp[j-1] used is from the PREVIOUS row
            for (int j = m; j >= 1; j--) {
                if (s1.charAt(i) == s2.charAt(j - 1)) {
                    dp[j] = (dp[j] + dp[j - 1]) % MOD;
                }
                // else: dp[j] unchanged (inherits previous row value)
            }
        }

        return (int) dp[m];
    }
}
```

> ✅ **Verdict:** Accepted — O(n×m) time, **O(m) space**. Preferred in memory-constrained
> environments. GFG expects O(n×m) space but this also passes.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Recursion (no memo) | O(2^n) | O(n+m) | ❌ TLE |
| Top-Down DP (Memoization) | O(n×m) | O(n×m) | ✅ Clean, recursive |
| **Bottom-Up 2D DP** | **O(n×m)** | **O(n×m)** | ✅ **GFG Expected — use this** |
| Space-Optimised 1D DP | O(n×m) | O(m) | ✅ Best for memory |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#dynamic-programming` `#strings` `#subsequence` `#medium` `#modular-arithmetic` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** June 26, 2026
