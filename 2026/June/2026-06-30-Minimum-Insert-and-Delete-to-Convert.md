# 🔄 Minimum Insert and Delete to Convert

## Problem Link
🔗 [Minimum Insert and Delete to Convert – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/minimum-insertions-to-make-two-arrays-equal/1)

---

## Difficulty
**Hard** | Accuracy: `27.85%` | Submissions: `25K+` | Points: `8` | Companies: `Google` `Codenation`

---

## Tags
- Arrays
- Hashing
- Dynamic Programming
- Binary Search
- Longest Increasing Subsequence (LIS)

---

## Problem Summary

Given two arrays `a[]` (size `n`) and `b[]` (size `m`), find the **minimum number of
insertions and deletions** on array `a[]` needed to make it **identical** to `b[]`.

**Note:** `b[]` is **sorted** and all its elements are **distinct**.
Operations can be performed at any index, not necessarily at the end.

**Constraints:**
- `1 ≤ n, m ≤ 10⁵`
- `1 ≤ a[i], b[i] ≤ 10⁵`
- Expected Time: `O(n log n)` · Space: `O(n)`

---

## Intuition

### The Classic Reduction

This is the well-known **"min insertions + deletions to make A == B"** problem.
The standard approach: find the **Longest Common Subsequence (LCS)** of `a` and `b`.
Every element in the LCS can stay in place; everything else in `a` must be deleted,
and everything else in `b` must be inserted.

```
Answer = (n - LCS_length) deletions + (m - LCS_length) insertions
       = n + m - 2 × LCS_length
```

Computing LCS directly is **O(n×m)** — way too slow for `n, m ≤ 10⁵` (10¹⁰ ops).

### The Key Trick: b is Sorted and Distinct!

Because `b[]` is **sorted with distinct elements**, we don't need general LCS.
We can reduce LCS to **LIS (Longest Increasing Subsequence)**:

1. Map every value in `b` to its **index** (its rank in sorted order) using a HashMap.
2. Walk through `a`, and for every element that **exists in `b`**, replace it with
   its rank from the map. Discard elements of `a` not present in `b`.
3. The **LIS** of this rank-sequence equals the **LCS** of `a` and `b`.

### Why LIS = LCS Here

Since `b` is sorted, "appearing in the same relative order as in `b`" is
**equivalent to** "appearing in increasing rank order." Any subsequence of `a`
that forms an increasing sequence of ranks is, by definition, a subsequence
that also appears in the same order in `b` — i.e., a common subsequence.
The **longest** such subsequence is exactly the LCS.

LIS can be computed in **O(k log k)** using **patience sorting / binary search**,
giving us the required **O(n log n)** overall.

---

## Approach

1. Build `HashMap<Integer, Integer> rankOf` mapping each `b[i] → i` (O(m)).
2. Traverse `a[]`; for each element present in `rankOf`, append its rank to a list
   `filtered[]` (O(n) with O(1) average HashMap lookups).
3. Compute `L = LIS length of filtered[]` using the **tails array + binary search**
   technique (O(k log k), where k ≤ n).
4. Return `n + m - 2 * L`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int minInsAndDel(int[] a, int[] b)`

```java
import java.util.*;

class Solution {
    public int minInsAndDel(int[] a, int[] b) {
        int n = a.length, m = b.length;

        // Step 1: Map each value in b to its rank (sorted index)
        HashMap<Integer, Integer> rankOf = new HashMap<>();
        for (int i = 0; i < m; i++) {
            rankOf.put(b[i], i);
        }

        // Step 2: Filter a[] to ranks of elements that also exist in b[]
        ArrayList<Integer> filtered = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (rankOf.containsKey(a[i])) {
                filtered.add(rankOf.get(a[i]));
            }
        }

        // Step 3: Compute LIS length of 'filtered' using patience sorting (O(k log k))
        int lcsLength = lengthOfLIS(filtered);

        // Step 4: Answer = total deletions + total insertions
        return (n - lcsLength) + (m - lcsLength);
    }

    // Standard O(k log k) LIS using a 'tails' array
    private int lengthOfLIS(ArrayList<Integer> nums) {
        ArrayList<Integer> tails = new ArrayList<>();

        for (int num : nums) {
            // Binary search for the first element in 'tails' >= num
            int lo = 0, hi = tails.size();
            while (lo < hi) {
                int mid = lo + (hi - lo) / 2;
                if (tails.get(mid) < num) {
                    lo = mid + 1;
                } else {
                    hi = mid;
                }
            }

            // If num extends the longest sequence so far, append it
            if (lo == tails.size()) {
                tails.add(num);
            } else {
                // Otherwise, replace to keep 'tails' as tight as possible
                tails.set(lo, num);
            }
        }

        return tails.size();
    }
}
```

---

## Dry Run

### Example 1 — `a = [1, 2, 5, 3, 1]`, `b = [1, 3, 5]`

**Step 1 — Rank map:** `{1:0, 3:1, 5:2}`

**Step 2 — Filter a[]:**
| a[i] | In b? | Rank |
|:----:|:-----:|:----:|
| 1 | ✅ | 0 |
| 2 | ❌ | — |
| 5 | ✅ | 2 |
| 3 | ✅ | 1 |
| 1 | ✅ | 0 |

`filtered = [0, 2, 1, 0]`

**Step 3 — LIS of `[0, 2, 1, 0]`:**

| num | tails before | Binary search result | tails after |
|:---:|:------------:|:---------------------:|:-----------:|
| 0 | `[]` | insert at 0 | `[0]` |
| 2 | `[0]` | insert at 1 | `[0, 2]` |
| 1 | `[0, 2]` | replace at 1 | `[0, 1]` |
| 0 | `[0, 1]` | replace at 0 | `[0, 1]` |

`LIS length = 2`

**Step 4 — Answer:**
```
n + m - 2×LCS = 5 + 3 - 2×2 = 8 - 4 = 4
```

**Output:** `4` ✅ (matches the GFG explanation: delete 2, insert 3, delete last two = 1+1+2=4)

---

### Example 2 — `a = [1, 4]`, `b = [1, 4]`

**Rank map:** `{1:0, 4:1}`
**Filtered:** `[0, 1]`
**LIS:** `[0, 1]` → length 2

**Answer:** `2 + 2 - 2×2 = 0` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n log n)` | HashMap build: O(m). Filtering a[]: O(n) avg. LIS via binary search: O(n log n) dominates |
| **Space** | `O(n + m)` | HashMap O(m), filtered list O(n), tails array O(n) — simplifies to O(n) per problem statement (treating n, m as same order) |

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Reduction** | min insert+delete = n + m − 2×LCS(a, b) |
| **Sorted + Distinct Trick** | When one array is sorted & distinct, LCS reduces to LIS — O(n log n) instead of O(n×m) |
| **Patience Sorting** | The `tails` array technique for O(k log k) LIS — a must-know pattern |
| **HashMap for Rank Lookup** | O(1) average lookup to map values to their sorted position |
| **Interview Relevance** | Frequently asked at Google — tests recognizing when LCS can be simplified |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Longest Increasing Subsequence | LeetCode #300 | Core LIS technique used here |
| Minimum ASCII Delete Sum (LC #712) | LeetCode | General LCS-based edit problem |
| Longest Common Subsequence | GFG / LC #1143 | The O(n×m) version this optimizes |
| Russian Doll Envelopes | LeetCode #354 | Another LIS-reduction trick |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Standard 2D LCS DP — O(n × m)

```java
class Solution {
    public int minInsAndDel(int[] a, int[] b) {
        int n = a.length, m = b.length;
        int[][] dp = new int[n + 1][m + 1];

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (a[i - 1] == b[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        int lcs = dp[n][m];
        return (n - lcs) + (m - lcs);
    }
}
```

> ❌ **Verdict:** TLE — `O(n×m)` = `10¹⁰` operations for `n=m=10⁵`. Far too slow.

---

### 2. ✅ HashMap + LIS via Patience Sorting (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n log n)` time, `O(n)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| 2D LCS DP | O(n×m) | O(n×m) | ❌ TLE for n,m up to 10⁵ |
| **HashMap + LIS** | **O(n log n)** | **O(n)** | ✅ **Exploits sorted+distinct b — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#dynamic-programming` `#binary-search` `#hashing` `#lis` `#hard` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** June 30, 2026
