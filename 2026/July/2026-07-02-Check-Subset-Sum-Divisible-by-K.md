# ➗ Check Subset Sum Divisible by K

## Problem Link
🔗 [Check Subset Sum Divisible by K – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/subset-with-sum-divisible-by-m2546/1)

---

## Difficulty
**Medium** | Accuracy: `22.41%` | Submissions: `20K+` | Points: `4`

---

## Tags
- Arrays
- Dynamic Programming
- Subset Sum
- Modular Arithmetic

---

## Problem Summary

Given an array `arr[]` of **positive integers** and a value `k`, return `true` if the
sum of **any non-empty subset** of the array is divisible by `k`; otherwise return `false`.

**Constraints:**
- `1 ≤ arr.size(), k ≤ 10³`
- `1 ≤ arr[i] ≤ 10³`
- Expected Time: `O(n × k)` · Auxiliary Space: `O(k)`

---

## Intuition

### Why Brute Force Subsets Fails

There are `2ⁿ - 1` non-empty subsets. For `n = 1000`, that's astronomically large.
We need to avoid enumerating subsets explicitly.

### The Remainder-Reachability Trick

We don't care about the actual subset sums — only their **remainder mod k**.
There are only `k` possible remainders (`0` to `k-1`). So instead of tracking sums,
we track: *"which remainders are achievable using some subset of the elements
processed so far?"*

### The Subtlety: Empty Subset vs Non-Empty Subset

A remainder of `0` is **trivially achievable** by the **empty subset** (sum = 0).
But the problem requires a **non-empty** subset. If we're not careful, our DP will
report `true` even when no real subset achieves remainder `0`.

**Solution:** Maintain **two** parallel boolean arrays:

```
full[j] = true if remainder j is achievable using ANY subset (including empty)
          of elements processed so far

real[j] = true if remainder j is achievable using a NON-EMPTY subset
          of elements processed so far
```

`full[0]` starts `true` (empty subset achieves remainder 0), but `real[]` starts
**entirely false** — no real subset exists yet.

### Transition (processing element `x`, remainder `r = x % k`)

```
newFull[j] = full[j]  OR  full[(j - r + k) % k]
             (skip x)     (add x to a previously reachable "full" remainder)

newReal[j] = real[j]  OR  full[(j - r + k) % k]
             (skip x,      (add x to ANY previously reachable state — including
              keep old       empty — this makes the resulting subset non-empty
              real states)   because x itself is now included)
```

The key insight: adding `x` to *any* prior state (empty or not) always produces a
**non-empty** subset, because `x` is now part of it. That's why `newReal` pulls
from `full` (not `real`) when incorporating `x`.

### Final Answer

```
answer = real[0]   (after processing all n elements)
```

---

## Approach

1. Let array size be `k`. Create `boolean[] full = new boolean[k]` and `boolean[] real = new boolean[k]`.
2. Set `full[0] = true` (base case: empty subset).
3. For each element `x` in `arr`:
   - Compute `r = x % k`.
   - Create `newFull` and `newReal` as copies of `full` and `real`.
   - For each `j` from `0` to `k-1` where `full[j]` is `true`:
     - `newFull[(j + r) % k] = true`
     - `newReal[(j + r) % k] = true`
   - Set `full = newFull`, `real = newReal`.
4. Return `real[0]`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `boolean divisibleByK(int[] arr, int k)`

```java
class Solution {
    public boolean divisibleByK(int[] arr, int k) {
        int n = arr.length;

        // full[j]  = remainder j achievable using ANY subset (incl. empty) so far
        // real[j]  = remainder j achievable using a NON-EMPTY subset so far
        boolean[] full = new boolean[k];
        boolean[] real = new boolean[k];
        full[0] = true; // empty subset achieves remainder 0

        for (int x : arr) {
            int r = x % k;

            boolean[] newFull = full.clone();
            boolean[] newReal = real.clone();

            for (int j = 0; j < k; j++) {
                if (full[j]) {
                    int newRemainder = (j + r) % k;
                    newFull[newRemainder] = true;
                    // Including x makes this subset non-empty, regardless of
                    // whether 'full[j]' itself came from empty or non-empty state
                    newReal[newRemainder] = true;
                }
            }

            full = newFull;
            real = newReal;
        }

        return real[0];
    }
}
```

---

## Dry Run

### Example 1 — `arr = [3, 1, 7, 5]`, `k = 6`

**Initial:** `full = {0:T}`, `real = {}`

| Element | r | full (after) | real (after) |
|:-------:|:-:|---------------|---------------|
| 3 | 3 | {0,3} | {3} |
| 1 | 1 | {0,1,3,4} | {1,3,4} |
| 7 | 1 | {0,1,2,3,4,5} | {1,2,3,4,5} |
| 5 | 5 | {0,1,2,3,4,5} | **{0,1,2,3,4,5}** |

After processing `5`: `newReal[0] = real[0](F) OR full_old[(0-5+6)%6=1](T) = T`

The remainder `1` was reachable in `full` before processing `5` (via subset `{7}`,
since `7 % 6 = 1`). Adding `5` gives `(1+5) % 6 = 0` → subset `{7, 5}` sums to `12`,
and `12 % 6 = 0`. ✅ Matches the official explanation exactly.

**Output:** `real[0] = true` ✅

---

### Example 2 — `arr = [1, 2, 6]`, `k = 5`

**Initial:** `full = {0:T}`, `real = {}`

| Element | r | full (after) | real (after) |
|:-------:|:-:|---------------|---------------|
| 1 | 1 | {0,1} | {1} |
| 2 | 2 | {0,1,2,3} | {1,2,3} |
| 6 | 1 | {0,1,2,3,4} | {1,2,3,4} |

**Output:** `real[0] = false` ✅ — matches: no subset of `{1,2,6}` sums to a multiple of 5
(subset sums are 1, 2, 6, 3, 7, 8, 9 → mod 5: 1, 2, 1, 3, 2, 3, 4 — never 0).

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n × k)` | For each of `n` elements, scan all `k` possible remainders |
| **Space** | `O(k)` | Two boolean arrays of size `k` (`full` and `real`); cloning each iteration is also O(k) |

For max constraints `n = k = 10³`: `10⁶` operations — comfortably fast.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | DP on remainders (subset sum reduced from O(sum) to O(k) states) |
| **Empty vs Non-Empty Trap** | Must track "any subset" and "non-empty subset" separately to avoid false positives |
| **Why `newReal` reads from `full`** | Adding the current element to *any* prior state guarantees the result is non-empty |
| **Modular Reduction** | Reduces problem from tracking `10⁹`-scale sums to just `k` remainder buckets |
| **Interview Relevance** | Classic "subset sum divisible by k" — appears frequently in DP interview rounds |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Subarray Sums Divisible by K | LeetCode #974 | Contiguous version — uses prefix sums instead |
| Partition Equal Subset Sum | LeetCode #416 | Classic 0/1 subset-sum DP |
| Count Subsets with Sum K | GFG | Same DP style, exact sum instead of remainder |
| Coin Change | LeetCode #322 | DP over achievable value states |

---

## Alternative Approaches

### 1. 🐢 Brute Force — All Subsets — O(2ⁿ)

Enumerate every non-empty subset, check if its sum is divisible by `k`.

```java
// TLE for n > ~20
class Solution {
    public boolean divisibleByK(int[] arr, int k) {
        int n = arr.length;
        for (int mask = 1; mask < (1 << n); mask++) { // mask=0 skipped (empty)
            int sum = 0;
            for (int i = 0; i < n; i++) {
                if ((mask & (1 << i)) != 0) sum += arr[i];
            }
            if (sum % k == 0) return true;
        }
        return false;
    }
}
```

> ❌ **Verdict:** TLE — `2¹⁰⁰⁰` is astronomically infeasible. Only works for tiny n.

---

### 2. ✅ DP on Remainders — Full + Real Arrays (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n×k)` time, `O(k)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force (all subsets) | O(2ⁿ) | O(n) | ❌ TLE beyond n≈20 |
| **DP on Remainders** | **O(n×k)** | **O(k)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#dynamic-programming` `#subset-sum` `#modular-arithmetic` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 2, 2026
