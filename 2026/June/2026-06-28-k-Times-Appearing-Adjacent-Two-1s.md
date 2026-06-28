# 🔢 k Times Appearing Adjacent Two 1's

## Problem Link
🔗 [k Times Appearing Adjacent Two 1's – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/count-binary-strings1944/1)

---

## Difficulty
**Medium** | Accuracy: `36.23%` | Submissions: `3K+` | Points: `4`

---

## Tags
- Dynamic Programming
- Binary Strings
- Counting
- Modular Arithmetic

---

## Problem Summary

Given two integers `n` and `k`, count the number of **binary strings of length `n`**
such that the number of positions where **two adjacent 1's appear** is exactly `k`.

Formally: count strings `s` of length `n` where the number of indices `i`
satisfying `s[i] = '1' AND s[i+1] = '1'` equals exactly `k`.

Return the answer **modulo 10⁹+7**.

**Constraints:** `1 ≤ n, k ≤ 10³`
**Expected:** Time `O(n²)` · Space `O(n×k)`

---

## Intuition

### Defining "Adjacent 1-pair"

For string `"11011"`:
```
Pos 0-1: s[0]=1, s[1]=1 → pair ✅
Pos 1-2: s[1]=1, s[2]=0 → no pair ❌
Pos 2-3: s[2]=0, s[3]=1 → no pair ❌
Pos 3-4: s[3]=1, s[4]=1 → pair ✅
Total: 2 adjacent pairs
```

### Key Observations

1. **A new adjacent pair is created only when we append `1` to a string
   that already ends in `1`.**
   - Appending `0` never creates a pair — regardless of what came before.
   - Appending `1` to a string ending in `0` does **not** create a pair.
   - Appending `1` to a string ending in `1` **creates exactly one new pair**.

2. **We need to track two things at each step:**
   - How many adjacent pairs `j` have been formed so far.
   - What the **last digit** is (0 or 1), because it decides whether the
     next `1` will create a new pair.

3. **This gives a clean 3-state DP** with O(n × k) distinct subproblems
   and O(1) work per subproblem → total O(n × k) = O(n²) time.

### DP State

```
dp0[i][j] = # binary strings of length i, with exactly j adjacent pairs, ending in '0'
dp1[i][j] = # binary strings of length i, with exactly j adjacent pairs, ending in '1'
```

### Transitions (from length i → length i+1)

```
Append '0'  →  no new pair, last digit becomes 0:
    dp0[i+1][j]   += dp0[i][j] + dp1[i][j]

Append '1' to string ending in '0'  →  no new pair, last digit becomes 1:
    dp1[i+1][j]   += dp0[i][j]

Append '1' to string ending in '1'  →  NEW pair! j increases by 1:
    dp1[i+1][j+1] += dp1[i][j]        (only when j+1 ≤ k)
```

### Base Cases (length = 1)

```
dp0[1][0] = 1    →  string "0"
dp1[1][0] = 1    →  string "1"
```

### Answer

```
dp0[n][k] + dp1[n][k]
```

---

## Approach

1. Allocate `long[][] dp0` and `long[][] dp1` each of size `(n+1) × (k+1)`.
2. Set base cases: `dp0[1][0] = 1`, `dp1[1][0] = 1`.
3. For each `i` from `1` to `n-1`, for each `j` from `0` to `k`:
   - `dp0[i+1][j]   += (dp0[i][j] + dp1[i][j]) % MOD`
   - `dp1[i+1][j]   += dp0[i][j] % MOD`
   - If `j+1 ≤ k`: `dp1[i+1][j+1] += dp1[i][j] % MOD`
4. Return `(dp0[n][k] + dp1[n][k]) % MOD`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `long countStrings(int n, int k)`

```java
class Solution {
    public long countStrings(int n, int k) {
        int MOD = 1_000_000_007;

        // dp0[i][j] = # strings of length i, j adjacent 1-pairs, ending in 0
        // dp1[i][j] = # strings of length i, j adjacent 1-pairs, ending in 1
        long[][] dp0 = new long[n + 1][k + 1];
        long[][] dp1 = new long[n + 1][k + 1];

        // Base case: single character strings, 0 adjacent pairs
        dp0[1][0] = 1; // string "0"
        dp1[1][0] = 1; // string "1"

        for (int i = 1; i < n; i++) {
            for (int j = 0; j <= k; j++) {

                // Append '0': no new pair, last digit = 0
                dp0[i + 1][j] = (dp0[i + 1][j] + dp0[i][j] + dp1[i][j]) % MOD;

                // Append '1' to string ending in '0': no new pair, last digit = 1
                dp1[i + 1][j] = (dp1[i + 1][j] + dp0[i][j]) % MOD;

                // Append '1' to string ending in '1': creates a new adjacent pair
                if (j + 1 <= k) {
                    dp1[i + 1][j + 1] = (dp1[i + 1][j + 1] + dp1[i][j]) % MOD;
                }
            }
        }

        return (dp0[n][k] + dp1[n][k]) % MOD;
    }
}
```

---

## Dry Run

### Example 1 — `n = 3, k = 2`

**Step i=1 → i=2** (j=0 only has values):

| Transition | Rule | Value |
|---|---|---|
| `dp0[2][0]` += dp0[1][0] + dp1[1][0] | append 0 | **2** |
| `dp1[2][0]` += dp0[1][0] | append 1 to ...0 | **1** |
| `dp1[2][1]` += dp1[1][0] | append 1 to ...1 | **1** |

**Step i=2 → i=3**:

| j | Transition | Value |
|:-:|---|---|
| 0 | `dp0[3][0]` += dp0[2][0]+dp1[2][0] = 2+1 | **3** |
| 0 | `dp1[3][0]` += dp0[2][0] = 2 | **2** |
| 0 | `dp1[3][1]` += dp1[2][0] = 1 | **1** |
| 1 | `dp0[3][1]` += dp0[2][1]+dp1[2][1] = 0+1 | **1** |
| 1 | `dp1[3][1]` += dp0[2][1] = 0 | 1 (unchanged) |
| 1 | `dp1[3][2]` += dp1[2][1] = 1 | **1** |

**Answer:** `dp0[3][2] + dp1[3][2] = 0 + 1 = 1` ✅

**Verification:** Only "111" has 2 adjacent pairs (positions 0-1 and 1-2) ✓

---

### Example 2 — `n = 5, k = 2`

Key table values at each length:

| Length i | dp0[i][2] | dp1[i][2] | Running answer |
|:--------:|:---------:|:---------:|:--------------:|
| 1 | 0 | 0 | 0 |
| 2 | 0 | 0 | 0 |
| 3 | 0 | 1 | 1 |
| 4 | 1 | 1 | 2 |
| 5 | **2** | **4** | **6** |

**Answer:** `2 + 4 = 6` ✅

**Verification of 6 strings:**
```
"00111" → pairs at (2,3),(3,4)          = 2 ✅
"10111" → pairs at (2,3),(3,4)          = 2 ✅
"01110" → pairs at (1,2),(2,3)          = 2 ✅
"11100" → pairs at (0,1),(1,2)          = 2 ✅
"11101" → pairs at (0,1),(1,2)          = 2 ✅
"11011" → pairs at (0,1),(3,4)          = 2 ✅
```

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n × k)` = `O(n²)` | Two nested loops: `i` from 1..n, `j` from 0..k; each O(1) work. Since k ≤ n ≤ 10³, worst case = 10⁶ |
| **Space** | `O(n × k)` | Two 2D arrays `dp0` and `dp1` each of size `(n+1)×(k+1)` |

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | 2D DP with digit tracking (last character matters) |
| **State Design** | Split by last digit (0 or 1) to know if next '1' creates a pair |
| **Critical Insight** | A pair is created ONLY when '1' is appended to a string ending in '1' |
| **Modulo Trap** | Apply `% MOD` at every addition — values can exceed `long` range otherwise |
| **k > n-1 edge** | Max pairs in length n = n-1; if k ≥ n, answer is naturally 0 from the DP |
| **Return type** | GFG expects `long` return — don't cast to `int` prematurely |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Count Binary Strings Without Consecutive 1s | GFG | Special case: k=0 of this problem |
| House Robber (LeetCode #198) | LeetCode | DP tracking previous state |
| Count strings with k non-repeating chars | GFG | Same last-character tracking pattern |
| Distinct Subsequences (LeetCode #115) | LeetCode | Counting with character-match DP |

---

## Alternative Approaches

### 1. 🐢 Brute Force — O(2ⁿ × n)

Generate all 2ⁿ binary strings, count adjacent pairs in each, filter those with exactly k.

```java
class Solution {
    public long countStrings(int n, int k) {
        int MOD = 1_000_000_007;
        long count = 0;

        for (int mask = 0; mask < (1 << n); mask++) {
            int pairs = 0;
            for (int i = 0; i < n - 1; i++) {
                int bit1 = (mask >> i) & 1;
                int bit2 = (mask >> (i + 1)) & 1;
                if (bit1 == 1 && bit2 == 1) pairs++;
            }
            if (pairs == k) count++;
        }
        return count % MOD;
    }
}
```

> ❌ **Verdict:** TLE for n > 25. 2ⁿ grows to 10³⁰⁰ for n=1000 — completely infeasible.

---

### 2. ✅ 2D DP with Two Arrays (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — O(n×k) time, O(n×k) space, handles all edge cases.

---

### 3. 💡 Space-Optimised DP — O(k) Space

Since row `i+1` only depends on row `i`, we can use two 1D arrays and rotate them.

```java
class Solution {
    public long countStrings(int n, int k) {
        int MOD = 1_000_000_007;
        long[] cur0 = new long[k + 1]; // ending in 0
        long[] cur1 = new long[k + 1]; // ending in 1
        cur0[0] = 1; cur1[0] = 1;      // base: length 1

        for (int i = 1; i < n; i++) {
            long[] nxt0 = new long[k + 1];
            long[] nxt1 = new long[k + 1];

            for (int j = 0; j <= k; j++) {
                nxt0[j] = (nxt0[j] + cur0[j] + cur1[j]) % MOD;
                nxt1[j] = (nxt1[j] + cur0[j]) % MOD;
                if (j + 1 <= k) {
                    nxt1[j + 1] = (nxt1[j + 1] + cur1[j]) % MOD;
                }
            }
            cur0 = nxt0;
            cur1 = nxt1;
        }

        return (cur0[k] + cur1[k]) % MOD;
    }
}
```

> ✅ **Verdict:** Accepted — same O(n×k) time but only **O(k) space**. Best for memory.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(2ⁿ × n) | O(1) | ❌ TLE |
| **2D DP** | **O(n×k)** | **O(n×k)** | ✅ **GFG Expected — cleanest** |
| Space-Optimised DP | O(n×k) | O(k) | ✅ Best memory, same speed |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#dynamic-programming` `#binary-strings` `#counting` `#medium` `#modular-arithmetic` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** June 28, 2026
