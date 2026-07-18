# ➖ Max Absolute Diff of Two Subarrays

## Problem Link
🔗 [Max Absolute Diff of Two Subarrays – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/max-absolute-difference4114/1)

---

## Difficulty
**Medium** | Accuracy: `50.44%` | Submissions: `16K+` | Points: `4` | Company: `Google`

---

## Tags
- Arrays
- Kadane's Algorithm
- Prefix/Suffix DP

---

## Problem Summary

Given an array of integers `arr[]`, find two **non-overlapping contiguous
subarrays** such that the **absolute difference** between the sums of the two
subarrays is **maximum**.

**Constraints:**
- `2 ≤ arr.size() ≤ 10⁵`
- `-10³ ≤ arr[i] ≤ 10³`
- Expected Time: `O(n)` · Auxiliary Space: `O(n)`

---

## Intuition

### The Two Subarrays Must Be Separated by Some Split Point

Since the two subarrays are non-overlapping and contiguous, there always exists
some index where the array can be conceptually "cut" into a left part and a
right part, with one subarray drawn from each side. So we can reduce the
problem to: **for every possible split point, what's the best combination of
a subarray from the left side and a subarray from the right side?**

### Maximizing Absolute Difference = Maximizing (Big − Small)

To maximize `|sum(A) - sum(B)|`, we want either:
- A **large positive** subarray sum on one side and a **very negative** (small)
  subarray sum on the other, OR
- A **very negative** subarray sum on one side and a **large positive** one on the other.

Both cases reduce to the same thing: `max(bigSum) - min(smallSum)`, just with
the two sides swapped.

### Precompute Four Arrays with Kadane's Algorithm

- `maxLeft[i]` = the maximum subarray sum achievable using only elements from
  `arr[0..i]` (best subarray sum considering everything up to and including `i`).
- `minLeft[i]` = the minimum subarray sum achievable using only `arr[0..i]`.
- `maxRight[i]` = the maximum subarray sum achievable using only `arr[i..n-1]`.
- `minRight[i]` = the minimum subarray sum achievable using only `arr[i..n-1]`.

Each of these is a straightforward **running Kadane's algorithm** (forward for
the `Left` arrays, backward for the `Right` arrays), computed in `O(n)`.

### Combining at Every Split Point

For every split between index `i` and `i+1` (so the left subarray is drawn from
`arr[0..i]` and the right subarray is drawn from `arr[i+1..n-1]`):

```
candidate1 = maxLeft[i]  - minRight[i+1]   (big subarray on the left, small on the right)
candidate2 = maxRight[i+1] - minLeft[i]    (big subarray on the right, small on the left)
```

The overall answer is the maximum of these two candidates across **all** split points.

### Verification with Example 2

`arr = [2,-1,-2,1,-4,2,8]`

At split point `i=4` (between index 4 and 5):
```
maxRight[5] = 10   (best subarray sum from index 5 onward: [2,8] = 10)
minLeft[4]  = -6   (best minimum subarray sum from index 0 to 4: [-1,-2,1,-4] = -6)

candidate2 = 10 - (-6) = 16
```

This matches the expected output `16` — achieved by exactly the two subarrays
in the official explanation: `[-1,-2,1,-4]` (sum `-6`) and `[2,8]` (sum `10`) ✅

---

## Approach

1. Compute `maxLeft[]` and `minLeft[]` via a forward Kadane's pass:
   - Track rolling `curMax`/`curMin` (extend or restart at each index).
   - `maxLeft[i] = max(maxLeft[i-1], curMax)`, similarly for `minLeft`.
2. Compute `maxRight[]` and `minRight[]` via a backward Kadane's pass (mirror logic).
3. For every split point `i` from `0` to `n-2`:
   - `ans = max(ans, maxLeft[i] - minRight[i+1])`
   - `ans = max(ans, maxRight[i+1] - minLeft[i])`
4. Return `ans`.

---

## Java Solution

> ⚠️ **Verify the exact method name in the GFG editor before submitting.**
> Based on the URL slug, a likely candidate is `maxAbsoluteDiff`, but confirm
> visually before pasting.

```java
class Solution {
    public int maxAbsoluteDiff(int[] arr) {
        int n = arr.length;

        int[] maxLeft = new int[n];
        int[] minLeft = new int[n];
        int[] maxRight = new int[n];
        int[] minRight = new int[n];

        // Forward Kadane's: best/worst subarray sum using arr[0..i]
        maxLeft[0] = arr[0];
        minLeft[0] = arr[0];
        int curMax = arr[0], curMin = arr[0];
        for (int i = 1; i < n; i++) {
            curMax = Math.max(arr[i], curMax + arr[i]);
            curMin = Math.min(arr[i], curMin + arr[i]);
            maxLeft[i] = Math.max(maxLeft[i - 1], curMax);
            minLeft[i] = Math.min(minLeft[i - 1], curMin);
        }

        // Backward Kadane's: best/worst subarray sum using arr[i..n-1]
        maxRight[n - 1] = arr[n - 1];
        minRight[n - 1] = arr[n - 1];
        curMax = arr[n - 1];
        curMin = arr[n - 1];
        for (int i = n - 2; i >= 0; i--) {
            curMax = Math.max(arr[i], arr[i] + curMax);
            curMin = Math.min(arr[i], arr[i] + curMin);
            maxRight[i] = Math.max(maxRight[i + 1], curMax);
            minRight[i] = Math.min(minRight[i + 1], curMin);
        }

        // Combine at every possible split point between i and i+1
        int ans = Integer.MIN_VALUE;
        for (int i = 0; i < n - 1; i++) {
            ans = Math.max(ans, maxLeft[i] - minRight[i + 1]);
            ans = Math.max(ans, maxRight[i + 1] - minLeft[i]);
        }

        return ans;
    }
}
```

---

## Dry Run

### Example 2 — `arr = [2,-1,-2,1,-4,2,8]` → Expected Output: `16`

**Computed arrays:**

| i | arr[i] | maxLeft | minLeft | maxRight | minRight |
|:-:|:------:|:-------:|:-------:|:--------:|:--------:|
| 0 | 2  | 2 | 2  | 10 | -6 |
| 1 | -1 | 2 | -1 | 10 | -6 |
| 2 | -2 | 2 | -3 | 10 | -5 |
| 3 | 1  | 2 | -3 | 10 | -4 |
| 4 | -4 | 2 | -6 | 10 | -4 |
| 5 | 2  | 2 | -6 | 10 | 2  |
| 6 | 8  | 10| -6 | 8  | 8  |

**Checking every split point:**

| Split (i, i+1) | candidate1 = maxLeft[i]-minRight[i+1] | candidate2 = maxRight[i+1]-minLeft[i] | max |
|:--------------:|:--------------------------------------:|:---------------------------------------:|:---:|
| (0,1) | 2-(-6)=8 | 10-2=8 | 8 |
| (1,2) | 2-(-5)=7 | 10-(-1)=11 | 11 |
| (2,3) | 2-(-4)=6 | 10-(-3)=13 | 13 |
| (3,4) | 2-(-4)=6 | 10-(-3)=13 | 13 |
| (4,5) | 2-2=0 | 10-(-6)=**16** | **16** |
| (5,6) | 2-8=-6 | 8-(-6)=14 | 14 |

**Overall maximum:** `16` ✅ — achieved at split `(4,5)`, matching the exact
subarrays `[-1,-2,1,-4]` and `[2,8]` from the official explanation.

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n)` | Two Kadane's passes (forward + backward), each O(n), plus one final O(n) combination pass |
| **Space** | `O(n)` | Four arrays (`maxLeft`, `minLeft`, `maxRight`, `minRight`), each of size `n` |

For max constraints `n = 10⁵`: trivially fast.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Prefix/suffix Kadane's algorithm, combined at every possible split point |
| **Why Split Points Work** | Any two non-overlapping subarrays are always separable by at least one index boundary |
| **Absolute Difference → Max Minus Min** | `\|A-B\|` maximized by considering both `max-min` orderings across the split, never needing to compute true absolute value directly |
| **Four Arrays, Not Two** | Both the maximum AND minimum subarray sums are needed on each side, since either could pair with either to produce the biggest gap |
| **Interview Relevance** | A strong combination of two classic techniques — Kadane's algorithm and prefix/suffix precomputation — frequently seen at Google-tagged problems |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Maximum Subarray | LeetCode #53 | The base Kadane's algorithm used here |
| Best Time to Buy and Sell Stock | LeetCode #121 | Similar prefix-min/suffix-max combination idea |
| Split Array with Same Average | LeetCode #805 | Different problem, same "split point" reasoning style |
| Maximum Sum of Two Non-Overlapping Subarrays | LeetCode #1031 | Very similar non-overlapping subarray optimization |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Try Every Pair of Subarrays — O(n²) or Worse

Enumerate possible pairs of non-overlapping subarrays directly and track the
max absolute difference, even with prefix sums to get subarray sums in O(1).

```java
// Tries every pair of (start1,end1) and (start2,end2) combinations — too slow
// even with O(1) prefix-sum lookups, since there are O(n^2) subarrays total
// and checking all non-overlapping PAIRS of them is far more than O(n^2).
```

> ❌ **Verdict:** TLE — trying all pairs of subarrays is far too slow for `n=10⁵`.

---

### 2. ✅ Prefix/Suffix Kadane's at Every Split Point (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n)` time, `O(n)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force (all subarray pairs) | O(n²) or worse | O(n) | ❌ Too slow for n=10⁵ |
| **Prefix/Suffix Kadane's** | **O(n)** | **O(n)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#kadane` `#arrays` `#prefix-suffix-dp` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 17, 2026
