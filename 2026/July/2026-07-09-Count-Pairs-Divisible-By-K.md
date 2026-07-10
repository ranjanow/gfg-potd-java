# ➗ Count Pairs Divisible By K

## Problem Link
🔗 [Count Pairs Divisible By K – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/count-pairs-in-array-divisible-by-k/1)

---

## Difficulty
**Medium** | Accuracy: `40.47%` | Submissions: `52K+` | Points: `4` | Company: `PayPal`

---

## Tags
- Arrays
- Hashing
- Modular Arithmetic
- Combinatorics

---

## Problem Summary

Given an array `arr[]` and a positive integer `k`, count the **total number of pairs**
in the array whose **sum is divisible by `k`**.

**Constraints:**
- `1 ≤ |arr| ≤ 5×10⁴`
- `1 ≤ arr[i] ≤ 10⁶`
- `1 ≤ k ≤ 5×10⁴`
- Expected Time: `O(n)` · Auxiliary Space: `O(k)`

---

## Intuition

### Reduce to Remainders

Two numbers `a` and `b` sum to a multiple of `k` **if and only if**:

```
(a % k + b % k) % k == 0
```

This means their remainders must either **both be 0**, or **add up to exactly `k`**
(e.g., remainder `1` pairs with remainder `k-1`, remainder `2` pairs with `k-2`, etc.).

### Why We Don't Need to Check Every Pair

Instead of checking all `O(n²)` pairs directly, we only need to know **how many
elements fall into each remainder class**. Build a frequency array
`freq[0..k-1]`, where `freq[r]` = count of elements with `arr[i] % k == r`.

### Counting Pairs by Remainder Class

For each valid remainder pairing `(r, k-r)`:

1. **`r = 0`:** Any two elements in this class sum to a multiple of `k`
   (since `0 + 0 = 0`). Count `C(freq[0], 2) = freq[0] × (freq[0]-1) / 2`.

2. **`r` and `k - r` are different classes (`1 ≤ r < k-r`):** Every element in
   class `r` can pair with every element in class `k-r`.
   Count `freq[r] × freq[k-r]`.

3. **`k` is even and `r = k/2`:** This class pairs with **itself**
   (since `k/2 + k/2 = k`). Count `C(freq[k/2], 2) = freq[k/2] × (freq[k/2]-1) / 2`.

Sum all these contributions for the final answer.

### Verification with Example 1

`arr = [2,2,1,7,5,3]`, `k = 4`

```
Remainders: 2%4=2, 2%4=2, 1%4=1, 7%4=3, 5%4=1, 3%4=3
freq = [0, 2, 2, 2]   (freq[0]=0, freq[1]=2, freq[2]=2, freq[3]=2)

r=0:        C(0,2) = 0
r=1, k-r=3: freq[1]×freq[3] = 2×2 = 4
r=k/2=2:    C(2,2) = 1

Total = 0 + 4 + 1 = 5 ✅
```

Matches expected output `5` exactly — the counted pairs `(2,2)` come from the
remainder-2 self-pairing, while `(1,7)`, `(1,3)`, `(5,7)`, `(5,3)` come from the
remainder-1 ↔ remainder-3 cross pairing, together matching the 5 pairs listed in
the official explanation.

---

## Approach

1. Create `long[] freq = new long[k]`, all zeros.
2. For each element `x` in `arr`: `freq[x % k]++`.
3. Initialise `ans = 0`.
4. `ans += freq[0] × (freq[0]-1) / 2` (self-pairing at remainder 0).
5. For `r` from `1` to `(k-1)/2` (inclusive, integer division):
   - `ans += freq[r] × freq[k-r]`.
6. If `k` is even: `ans += freq[k/2] × (freq[k/2]-1) / 2` (self-pairing at remainder k/2).
7. Return `ans`.

> **Why loop only up to `(k-1)/2`?** Pairing `r` with `k-r` is symmetric — processing
> both `r=1,k-r=k-1` and later `r=k-1,k-r=1` would double-count. Stopping at
> `(k-1)/2` visits each unordered pair `{r, k-r}` exactly once.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `long countKdivPairs(int[] arr, int k)`

```java
class Solution {
    public long countKdivPairs(int[] arr, int k) {
        long[] freq = new long[k];

        // Bucket every element by its remainder mod k
        for (int x : arr) {
            freq[x % k]++;
        }

        long ans = 0;

        // Remainder 0 pairs with itself
        ans += freq[0] * (freq[0] - 1) / 2;

        // Remainder r pairs with remainder (k-r), for 1 <= r < k-r
        for (int r = 1; r <= (k - 1) / 2; r++) {
            ans += freq[r] * freq[k - r];
        }

        // If k is even, remainder k/2 pairs with itself
        if (k % 2 == 0) {
            int half = k / 2;
            ans += freq[half] * (freq[half] - 1) / 2;
        }

        return ans;
    }
}
```

---

## Dry Run

### Example 1 — `arr = [2,2,1,7,5,3]`, `k = 4` → Expected Output: `5`

`freq = [0, 2, 2, 2]`

| Step | Computation | Contribution | Running ans |
|---|---|:-:|:-:|
| r=0 self-pair | `C(0,2) = 0` | 0 | 0 |
| r=1 ↔ r=3 | `freq[1]×freq[3] = 2×2` | 4 | 4 |
| k/2=2 self-pair | `C(freq[2],2) = C(2,2)` | 1 | **5** |

**Output:** `5` ✅

---

### Example 2 — `arr = [5,9,36,74,52,31,42]`, `k = 3` → Expected Output: `7`

**Remainders:** `5%3=2, 9%3=0, 36%3=0, 74%3=2, 52%3=1, 31%3=1, 42%3=0`

`freq = [3, 2, 2]` (freq[0]=3 from {9,36,42}, freq[1]=2 from {52,31}, freq[2]=2 from {5,74})

| Step | Computation | Contribution | Running ans |
|---|---|:-:|:-:|
| r=0 self-pair | `C(3,2) = 3×2/2 = 3` | 3 | 3 |
| r=1 ↔ r=2 | `freq[1]×freq[2] = 2×2` | 4 | **7** |
| k odd → no k/2 case | — | — | 7 |

**Output:** `7` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n)` | Single pass to build `freq[]` (O(n)), plus a loop over `k` remainder classes (O(k)) |
| **Space** | `O(k)` | The `freq[]` array has exactly `k` slots |

For max constraints `n = k = 5×10⁴`: both terms are `5×10⁴` — trivially fast.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Remainder bucketing + combinatorial pair counting |
| **Complementary Remainders** | `r` and `k-r` are the only remainder pairs that sum to a multiple of `k` |
| **Self-Pairing Cases** | Remainder `0` and (if `k` even) remainder `k/2` pair with themselves — use `C(freq, 2)`, not `freq × freq` |
| **Avoiding Double-Counting** | Loop only up to `(k-1)/2` so each `{r, k-r}` pair is processed exactly once |
| **Overflow Awareness** | `freq[r] × freq[k-r]` and `C(freq,2)` can reach ~10⁹ — use `long` throughout |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Subarray Sums Divisible by K | LeetCode #974 | Same remainder-bucketing idea, applied to prefix sums |
| Check Subset Sum Divisible by K | GFG (this repo!) | Related modular DP technique, different goal |
| Two Sum | LeetCode #1 | Simpler complementary-value pairing pattern |
| Pairs of Songs With Total Durations Divisible by 60 | LeetCode #1010 | Nearly identical problem, different framing |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Check All Pairs — O(n²)

```java
class Solution {
    public long countKdivPairs(int[] arr, int k) {
        int n = arr.length;
        long ans = 0;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if ((arr[i] + arr[j]) % k == 0) ans++;
            }
        }
        return ans;
    }
}
```

> ❌ **Verdict:** TLE — `O(n²)` = `2.5×10⁹` operations for `n=5×10⁴`. Too slow.

---

### 2. ✅ Remainder Bucketing + Combinatorics (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n)` time, `O(k)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(1) | ❌ TLE for n=5×10⁴ |
| **Remainder Bucketing** | **O(n)** | **O(k)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#hashing` `#modular-arithmetic` `#combinatorics` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 9, 2026
