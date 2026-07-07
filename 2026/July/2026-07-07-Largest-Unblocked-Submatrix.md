# 🟩 Largest Unblocked Submatrix

## Problem Link
🔗 [Largest Unblocked Submatrix – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/largest-unblocked-submatrix/1)

---

## Difficulty
**Medium** | Accuracy: `52.99%` | Submissions: `16K+` | Points: `4`

---

## Tags
- Greedy
- Sorting
- Arrays
- Algorithms

---

## Problem Summary

Given integers `n` and `m`, and an array `arr[][]` of size `k`, where `arr[i] = [r, c]`
represents a **blocked cell** in an `n × m` grid. Each blocked cell blocks its
**entire row and column**. Find the **largest continuous unblocked area** in the grid.

**Note:** No two blocked cells share the same row or the same column.

**Constraints:**
- `1 ≤ n, m ≤ 10⁴`
- `0 ≤ k ≤ min(n, m)`
- `1 ≤ r ≤ n`, `1 ≤ c ≤ m`
- Expected Time: `O(k log k)` · Auxiliary Space: `O(k)`

---

## Intuition

### Reframing the Problem

Since each blocked cell blocks its **entire** row and column, a cell `(i, j)` is
unblocked **if and only if** row `i` is not blocked **and** column `j` is not blocked.

This means: **any unblocked row combined with any unblocked column always forms a
fully unblocked cell.** So the largest unblocked rectangle is simply:

```
answer = (longest run of consecutive unblocked rows) × (longest run of consecutive unblocked columns)
```

Because picking the widest unblocked row-band and the widest unblocked column-band
guarantees **every** cell in that combined rectangle is unblocked — the blocking
mechanism can never partially interfere within a rectangle formed this way.

### Verification with Example 2

`n=3, m=3, arr=[[3,3]]` → blocked row `3`, blocked column `3`.

- Unblocked rows: `1, 2` → longest run = `2`
- Unblocked columns: `1, 2` → longest run = `2`
- Answer = `2 × 2 = 4` ✅ (matches: cells `(1,1),(1,2),(2,1),(2,2)`)

### Finding the Longest Run — Sort and Scan

Since blocked rows are just `k` scattered values in `[1, n]`, sorting them lets us
scan through and measure the **gaps** between consecutive blocked rows (and the
gaps before the first and after the last). The largest such gap is the longest
run of unblocked rows. Same logic applies independently to columns.

### Handling Boundaries and k = 0

Treat position `0` as a virtual "blocked row" before the grid starts, and `n+1`
as a virtual "blocked row" after the grid ends. This uniformly captures:
- The gap **before** the first real blocked row.
- Gaps **between** consecutive blocked rows.
- The gap **after** the last real blocked row.

If `k = 0` (no blocked cells at all), the single "gap" spans the entire `n` rows
(and `m` columns) — correctly giving `answer = n × m`.

---

## Approach

1. Extract all `r` values into `rows[]` and all `c` values into `cols[]` (size `k` each).
2. Sort `rows[]` and `cols[]`.
3. Compute `maxRowGap` by scanning `rows[]` with a virtual `prev = 0` boundary,
   tracking `gap = rows[i] - prev - 1` at each step, and finally
   `gap = n - prev` after the loop (using the last processed value as `prev`).
4. Compute `maxColGap` the same way using `cols[]` and `m`.
5. Return `maxRowGap × maxColGap`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int largestArea(int n, int m, int k, int[][] mat)`
> — note the driver passes `k` explicitly as a separate parameter, even though it's
> always equal to `mat.length`. We simply ignore the passed-in `k` and use
> `mat.length` internally (safer, and avoids relying on the caller passing a
> consistent value).

```java
import java.util.Arrays;

class Solution {
    public int largestArea(int n, int m, int k, int[][] mat) {
        int size = mat.length; // same as k, but derived directly for safety

        int[] rows = new int[size];
        int[] cols = new int[size];
        for (int i = 0; i < size; i++) {
            rows[i] = mat[i][0];
            cols[i] = mat[i][1];
        }

        Arrays.sort(rows);
        Arrays.sort(cols);

        long maxRowGap = computeMaxGap(rows, n);
        long maxColGap = computeMaxGap(cols, m);

        return (int) (maxRowGap * maxColGap);
    }

    // Finds the longest run of unblocked indices in [1, size],
    // given the sorted positions of blocked indices.
    private long computeMaxGap(int[] sortedPositions, int size) {
        long maxGap = 0;
        int prev = 0; // virtual boundary before the first row/column

        for (int pos : sortedPositions) {
            long gap = pos - prev - 1; // unblocked run strictly between prev and pos
            maxGap = Math.max(maxGap, gap);
            prev = pos;
        }

        // Final run: from the last blocked position to the end of the grid
        long lastGap = size - prev;
        maxGap = Math.max(maxGap, lastGap);

        return maxGap;
    }
}
```

---

## Dry Run

### Example 1 — `n=2, m=2, arr=[[2,2]]` → Expected Output: `1`

**Rows:** `sortedPositions = [2]`, `size = 2`

| pos | prev before | gap = pos-prev-1 | maxGap | prev after |
|:---:|:-----------:|:-----------------:|:------:|:----------:|
| 2 | 0 | 2-0-1=1 | 1 | 2 |

Final: `lastGap = 2-2=0` → `maxRowGap = max(1,0) = 1`

**Cols:** identical structure (blocked col = 2) → `maxColGap = 1`

**Output:** `1 × 1 = 1` ✅

---

### Example 2 — `n=3, m=3, arr=[[3,3]]` → Expected Output: `4`

**Rows:** `sortedPositions = [3]`, `size = 3`

| pos | prev before | gap = pos-prev-1 | maxGap | prev after |
|:---:|:-----------:|:-----------------:|:------:|:----------:|
| 3 | 0 | 3-0-1=2 | 2 | 3 |

Final: `lastGap = 3-3=0` → `maxRowGap = max(2,0) = 2`

**Cols:** identical structure (blocked col = 3) → `maxColGap = 2`

**Output:** `2 × 2 = 4` ✅

---

### Edge Case — `k = 0` (no blocked cells)

`sortedPositions = []` (empty), loop doesn't execute, `prev` stays `0`.
`lastGap = n - 0 = n` → `maxRowGap = n`. Similarly `maxColGap = m`.

**Output:** `n × m` — the entire grid is unblocked, as expected. ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(k log k)` | Sorting `rows[]` and `cols[]`, each of size `k`; the gap scan is O(k) |
| **Space** | `O(k)` | Two arrays of size `k` for extracted rows and columns |

For max constraints `k ≤ min(n,m) ≤ 10⁴`: trivially fast.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Insight** | Blocked rows and blocked columns act independently — the answer is a simple product of two 1D "longest gap" problems |
| **Why Rectangles Always Work** | A cell is blocked only via its own row or column, so any unblocked row × unblocked column combination is automatically valid |
| **Virtual Boundaries Trick** | Treating `0` and `n+1` as sentinel "blocked" positions cleanly handles edge gaps without special-casing |
| **k = 0 Edge Case** | Naturally falls out of the same formula — no special code branch needed |
| **Dimensionality Reduction** | A seemingly 2D grid problem reduces to two independent 1D "max gap between sorted points" problems |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Maximal Rectangle | LeetCode #85 | Different domain (binary matrix), but similar "find largest rectangle" theme |
| Missing Ranges | LeetCode #163 | Same "find gaps in sorted values" technique |
| Meeting Rooms II | LeetCode #253 | Sorting + scanning for intervals, similar mechanical style |
| Non-overlapping Intervals | LeetCode #435 | Sort-then-scan gap/interval reasoning |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Check Every Possible Rectangle — O(n² × m²)

Iterate over every possible top-left and bottom-right corner pair, check if the
rectangle is fully unblocked, and track the maximum area.

```java
// Astronomically slow — O(n²×m²) with n,m up to 10^4
// Not practically implementable within time limits; included only for contrast.
```

> ❌ **Verdict:** TLE — completely infeasible for `n, m` up to `10⁴`.

---

### 2. ✅ Sort + Gap Scan (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(k log k)` time, `O(k)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force (all rectangles) | O(n²×m²) | O(1) | ❌ Completely infeasible |
| **Sort + Gap Scan** | **O(k log k)** | **O(k)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#greedy` `#sorting` `#arrays` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 7, 2026
