# ⛰️ Mountain Subarray Queries

## Problem Link
🔗 [Mountain Subarray Queries – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/mountain-subarray-problem/1)

---

## Difficulty
**Medium** | Accuracy: `46.22%` | Submissions: `17K+` | Points: `4` | Average Time: `30m` | Company: `Amazon`

---

## Tags
- Arrays
- Prefix/Suffix Precomputation
- Queries

---

## Problem Summary

Given an array `arr[]` and a list of queries, for each query `[l, r]`,
determine whether the subarray `arr[l...r]` is a **mountain array**.

A subarray is a mountain if there exists an index `k` (`l ≤ k ≤ r`) such that:
```
arr[l] ≤ arr[l+1] ≤ ... ≤ arr[k] ≥ arr[k+1] ≥ ... ≥ arr[r]
```

- Elements of a mountain subarray first **non-decrease**, then **non-increase**.
- A subarray that's **entirely** non-decreasing or entirely non-increasing is
  also considered a mountain.

**Constraints:**
- `1 ≤ arr.size(), queries.size() ≤ 10⁵`
- `1 ≤ arr[i] ≤ 10⁶`
- `0 ≤ l ≤ r < arr.size()`
- Expected Time: `O(n + q)` · Auxiliary Space: `O(n)`

---

## Intuition

### Reframing "Is It a Mountain?" in Terms of Turning Points

A subarray fails to be a mountain **only** if it contains a **decrease followed
later by an increase** — a "valley" pattern, which creates two turning points
instead of one. So the check becomes:

> Within `[l, r-1]` (the gaps between consecutive elements), does any **strict
> decrease** happen **before** some later **strict increase**?

If the **last** strict increase in the range comes **before** the **first**
strict decrease in the range, there's no valley — the subarray is a valid
mountain. If a strict increase happens *after* some strict decrease, that's
exactly the invalid valley pattern.

### Two Precomputed Arrays, Built Once

Define, over the **gaps** between consecutive elements (position `i`
represents the gap between `arr[i]` and `arr[i+1]`):

```
lastIncUpTo[i] = the largest gap position p ≤ i where arr[p] < arr[p+1]
                 (or -1 if no such increase exists up to i)

firstDecFrom[i] = the smallest gap position p ≥ i where arr[p] > arr[p+1]
                  (or n if no such decrease exists from i onward)
```

Both are computable in a **single linear pass** each — `lastIncUpTo` left to
right, `firstDecFrom` right to left.

### Answering Each Query in O(1)

For a query `[l, r]` (with `l < r`):

```
lastInc  = lastIncUpTo[r-1]     → is it ≥ l? If yes, it's within our range.
firstDec = firstDecFrom[l]      → is it ≤ r-1? If yes, it's within our range.

mountain ⟺ (effective lastInc) < (effective firstDec)
```

If `lastIncUpTo[r-1] < l`, there's no increase within `[l, r-1]` at all — treat
it as `-1` (a sentinel guaranteed smaller than any valid decrease position). If
`firstDecFrom[l] > r-1`, there's no decrease within the range — treat it as `n`
(a sentinel guaranteed larger than any valid increase position). This lets a
**single comparison** decide every query.

### Verification with Example 1

`arr = [2,3,2,4,4,6,3,2]`

**Query `[0,2]`** → subarray `[2,3,2]`:
```
lastIncUpTo[1] = 0  (0 ≥ l=0, so effectiveLastInc = 0)
firstDecFrom[0] = 1 (1 ≤ r-1=1, so effectiveFirstDec = 1)
Mountain: 0 < 1 → TRUE ✅
```

**Query `[1,3]`** → subarray `[3,2,4]`:
```
lastIncUpTo[2] = 2  (2 ≥ l=1, so effectiveLastInc = 2)
firstDecFrom[1] = 1 (1 ≤ r-1=2, so effectiveFirstDec = 1)
Mountain: 2 < 1? NO → FALSE ✅
```

Both match the expected output `[true, false]` exactly — the decrease
(`3→2` at gap 1) happening **before** the later increase (`2→4` at gap 2)
correctly identifies the valley pattern in query 2.

---

## Approach

1. Build `lastIncUpTo[]` (size `n`) left to right:
   `lastIncUpTo[i] = i` if `arr[i] < arr[i+1]`, else `lastIncUpTo[i-1]`.
2. Build `firstDecFrom[]` (size `n`) right to left, with sentinel
   `firstDecFrom[n-1] = n`:
   `firstDecFrom[i] = i` if `arr[i] > arr[i+1]`, else `firstDecFrom[i+1]`.
3. For each query `[l, r]`:
   - If `l == r`: answer is trivially `true` (single element).
   - Else: compute `effectiveLastInc` and `effectiveFirstDec` as described
     above, and answer `effectiveLastInc < effectiveFirstDec`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `ArrayList<Boolean> processQueries(int[] arr, int[][] queries)`
> — note the return type is `ArrayList<Boolean>`, not `boolean[]`.

```java
import java.util.ArrayList;

class Solution {
    public ArrayList<Boolean> processQueries(int[] arr, int[][] queries) {
        int n = arr.length;
        int q = queries.length;

        // lastIncUpTo[i] = largest gap p <= i with arr[p] < arr[p+1], else -1
        int[] lastIncUpTo = new int[n];
        lastIncUpTo[0] = (n > 1 && arr[0] < arr[1]) ? 0 : -1;
        for (int i = 1; i <= n - 2; i++) {
            lastIncUpTo[i] = (arr[i] < arr[i + 1]) ? i : lastIncUpTo[i - 1];
        }

        // firstDecFrom[i] = smallest gap p >= i with arr[p] > arr[p+1], else n
        int[] firstDecFrom = new int[n];
        firstDecFrom[n - 1] = n; // sentinel: no gap possible from the last index
        for (int i = n - 2; i >= 0; i--) {
            firstDecFrom[i] = (arr[i] > arr[i + 1]) ? i : firstDecFrom[i + 1];
        }

        ArrayList<Boolean> result = new ArrayList<>();
        for (int idx = 0; idx < q; idx++) {
            int l = queries[idx][0], r = queries[idx][1];

            if (l == r) {
                result.add(true); // single element is trivially a mountain
                continue;
            }

            int lastInc = lastIncUpTo[r - 1];
            int effectiveLastInc = (lastInc >= l) ? lastInc : -1;

            int firstDec = firstDecFrom[l];
            int effectiveFirstDec = (firstDec <= r - 1) ? firstDec : n;

            result.add(effectiveLastInc < effectiveFirstDec);
        }

        return result;
    }
}
```

---

## Dry Run

### Example 1 — `arr = [2,3,2,4,4,6,3,2]`, `queries = [[0,2],[1,3]]` → Expected: `[true, false]`

**Gaps (position i = between arr[i] and arr[i+1]):**

| gap | values | type |
|:---:|:------:|:----:|
| 0 | 2 vs 3 | increase |
| 1 | 3 vs 2 | decrease |
| 2 | 2 vs 4 | increase |
| 3 | 4 vs 4 | equal (neither) |
| 4 | 4 vs 6 | increase |
| 5 | 6 vs 3 | decrease |
| 6 | 3 vs 2 | decrease |

**`lastIncUpTo[]`:** `[0, 0, 2, 2, 4, 4, 4]`
**`firstDecFrom[]`:** `[1, 1, 5, 5, 5, 5, 6, 8]`

**Query `[0,2]`:** `lastIncUpTo[1]=0` (≥0 ✓) → `0`; `firstDecFrom[0]=1` (≤1 ✓) → `1`.
`0 < 1` → **`true`** ✅

**Query `[1,3]`:** `lastIncUpTo[2]=2` (≥1 ✓) → `2`; `firstDecFrom[1]=1` (≤2 ✓) → `1`.
`2 < 1`? No → **`false`** ✅

---

### Example 2 — `arr = [2,2,2,2]`, `queries = [[0,2],[1,3]]` → Expected: `[true, true]`

All gaps are equal (`2 vs 2`) — neither strict increase nor strict decrease anywhere.

**`lastIncUpTo[]`:** `[-1, -1, -1]` (no increase found anywhere)
**`firstDecFrom[]`:** `[4, 4, 4, 4]` (no decrease found anywhere, sentinel = n = 4)

**Query `[0,2]`:** `lastIncUpTo[1]=-1` (< l=0) → sentinel `-1`; `firstDecFrom[0]=4` (> r-1=1) → sentinel `4`.
`-1 < 4` → **`true`** ✅

**Query `[1,3]`:** Same reasoning → **`true`** ✅

Both sentinels naturally satisfy the comparison, correctly identifying an
all-equal array as trivially a mountain (both non-decreasing and
non-increasing simultaneously).

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n + q)` | Two linear passes to build `lastIncUpTo[]` and `firstDecFrom[]` (`O(n)`), then `O(1)` per query (`O(q)` total) |
| **Space** | `O(n)` | Two arrays of size `n` |

For max constraints `n = q = 10⁵`: trivially fast — matches the expected
complexity exactly.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Reframing** | "Is it a mountain?" becomes "does a decrease ever precede a later increase?" — a much easier condition to check with precomputed thresholds |
| **Two Complementary Precomputations** | `lastIncUpTo` (prefix-style, left to right) and `firstDecFrom` (suffix-style, right to left) — a very common pairing for range queries |
| **Sentinel Values Matter** | Using `-1` and `n` as sentinels lets the *same* comparison logic handle "not found" cases without extra branching |
| **O(1) Per Query via Precomputation** | Classic pattern: spend `O(n)` once to answer any number of queries in `O(1)` each — essential whenever `q` can be as large as `n` |
| **Contrast with a Related Pitfall** | This is the *static, offline* version of the "longest bitonic subarray" idea — solved cleanly via prefix/suffix precomputation, sidestepping the kind of subtle two-pointer resume bug that can show up in similar problems |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Longest Bitonic Subarray | GFG (this repo!) | Same core "mountain" concept, but asks for the longest one instead of query-checking |
| Longest Mountain in Array | LeetCode #845 | The strict version of the same underlying pattern |
| Range Minimum Query | Various | Same "precompute once, answer O(1) per query" philosophy |
| Valid Mountain Array | LeetCode #941 | The single-array (no queries) version of this exact check |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Check Each Query Directly — O(q × n)

For each query, scan `arr[l..r]` directly to verify the non-decreasing-then-non-increasing pattern.

```java
import java.util.ArrayList;

class Solution {
    public ArrayList<Boolean> processQueries(int[] arr, int[][] queries) {
        int q = queries.length;
        ArrayList<Boolean> result = new ArrayList<>();

        for (int idx = 0; idx < q; idx++) {
            int l = queries[idx][0], r = queries[idx][1];
            int i = l;
            while (i < r && arr[i] <= arr[i + 1]) i++;
            while (i < r && arr[i] >= arr[i + 1]) i++;
            result.add(i == r);
        }

        return result;
    }
}
```

> ❌ **Verdict:** TLE — `O(q × n)` = `10¹⁰` operations for `n=q=10⁵`. Too slow.

---

### 2. ✅ Precomputed Threshold Arrays (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n+q)` time, `O(n)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force (per-query scan) | O(q×n) | O(1) | ❌ TLE for large n, q |
| **Precomputed Thresholds** | **O(n+q)** | **O(n)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#arrays` `#prefix-suffix` `#queries` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 19, 2026
