# 🛣️ Max Sum Path in Two Arrays

## Problem Link
🔗 [Max Sum Path in Two Arrays – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/max-sum-path-in-two-arrays/1)

---

## Difficulty
**Medium** | Accuracy: `30.9%` | Submissions: `83K+` | Points: `4` | Company: `Amazon`

---

## Tags
- Arrays
- Two Pointers
- Data Structures
- Greedy

---

## Problem Summary

Given two **sorted arrays of distinct integers** in increasing order `a[]` and `b[]`,
which may have some **common elements**, find the **maximum sum path** from the
beginning of any array to the end of any array. You may **switch** from one array
to the other **only at common elements**.

**Note:** When switching, count the common element **only once**.

**Constraints:**
- `1 ≤ a.size(), b.size() ≤ 10⁴`
- `1 ≤ a[i], b[i] ≤ 10⁵`
- Expected Time: `O(n + m)` · Auxiliary Space: `O(1)`

---

## Intuition

### Visualize It as a Merge with Checkpoints

Since both arrays are **sorted**, common elements act as **synchronization points** —
"bridges" where you can freely switch arrays. Between two consecutive common
elements (or from the start to the first common element, or from the last common
element to the end), you're **forced** to walk along just one array's segment.

**Key Insight:** For each "segment" between two bridges, you must choose exactly
**one** of the two arrays' running sums for that stretch — pick whichever gives
the **larger sum**, since only one path can pass through that region.

### Why Two Pointers Work

Because both arrays are sorted, we can merge them like in merge-sort, comparing
`a[i]` and `b[j]`:

- If `a[i] < b[j]`: this element belongs only to the current "segment" of `a` —
  add it to `sumA`, advance `i`.
- If `b[j] < a[i]`: similarly, add it to `sumB`, advance `j`.
- If `a[i] == b[j]`: we've hit a **common element** (a bridge!). Take the
  **better** of the two accumulated segment sums, add the common element once,
  reset both segment sums to `0`, and advance both pointers.

### Verification with Example 1

`a = [2,3,7,10,12]`, `b = [1,5,7,8]`

```
1(b) → sumB=1
2(a) → sumA=2
3(a) → sumA=5
5(b) → sumB=6
7 common! → result += max(5,6)+7 = 13; reset both sums to 0
10(a) → sumA=10
12(a) → sumA=22
8(b) → sumB=8 (b exhausted after this)
End → result += max(22, 8) = 22
Total = 13 + 22 = 35 ✅
```

**Path taken:** `1 + 5 + 7 + 10 + 12 = 35` — matches the official explanation exactly!

---

## Approach

1. Initialise `i = 0`, `j = 0`, `sumA = 0`, `sumB = 0`, `result = 0`.
2. While both `i < n` and `j < m`:
   - If `a[i] < b[j]`: `sumA += a[i]`, `i++`.
   - Else if `b[j] < a[i]`: `sumB += b[j]`, `j++`.
   - Else (equal — common element found):
     - `result += Math.max(sumA, sumB) + a[i]`
     - Reset `sumA = 0`, `sumB = 0`
     - `i++`, `j++`
3. After the loop, add any remaining tail elements to their respective sums:
   - While `i < n`: `sumA += a[i]`, `i++`.
   - While `j < m`: `sumB += b[j]`, `j++`.
4. `result += Math.max(sumA, sumB)`.
5. Return `result`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int maxPathSum(int[] a, int[] b)` — returns
> `int`, not `long`. Max possible sum is `n × 10⁵ ≈ 10⁴ × 10⁵ = 10⁹`, which still
> fits within `int` range (`~2.1×10⁹`), so we compute internally with `long` for
> safety and cast down to `int` only at the very end.

```java
class Solution {
    public int maxPathSum(int[] a, int[] b) {
        int n = a.length, m = b.length;
        int i = 0, j = 0;

        long sumA = 0, sumB = 0; // running sums of the current "segment" in each array
        long result = 0;

        // Merge-style traversal, comparing elements from both sorted arrays
        while (i < n && j < m) {
            if (a[i] < b[j]) {
                sumA += a[i];
                i++;
            } else if (b[j] < a[i]) {
                sumB += b[j];
                j++;
            } else {
                // Common element found — a valid switching point
                result += Math.max(sumA, sumB) + a[i]; // count the common element once
                sumA = 0;
                sumB = 0;
                i++;
                j++;
            }
        }

        // Add any remaining tail elements after one array is exhausted
        while (i < n) {
            sumA += a[i];
            i++;
        }
        while (j < m) {
            sumB += b[j];
            j++;
        }

        // Final segment: take the better of the two remaining sums
        result += Math.max(sumA, sumB);

        return (int) result; // safe: max ~10⁹ fits within int range (~2.1×10⁹)
    }
}
```

---

## Dry Run

### Example 1 — `a = [2,3,7,10,12]`, `b = [1,5,7,8]` → Expected Output: `35`

| Step | i | j | Comparison | Action | sumA | sumB | result |
|:----:|:-:|:-:|:----------:|--------|:----:|:----:|:------:|
| 1 | 0→0 | 0→1 | a[0]=2, b[0]=1 | b<a: sumB+=1 | 0 | 1 | 0 |
| 2 | 0→1 | 1 | a[0]=2, b[1]=5 | a<b: sumA+=2 | 2 | 1 | 0 |
| 3 | 1→2 | 1 | a[1]=3, b[1]=5 | a<b: sumA+=3 | 5 | 1 | 0 |
| 4 | 2 | 1→2 | a[2]=7, b[1]=5 | b<a: sumB+=5 | 5 | 6 | 0 |
| 5 | 2→3 | 2→3 | a[2]=7, b[2]=7 | common! result+=max(5,6)+7=13 | 0 | 0 | **13** |
| 6 | 3 | 3→4 | a[3]=10, b[3]=8 | b<a: sumB+=8 | 0 | 8 | 13 |
| — | 3 | 4 | j=4=m, loop ends | exit | 0 | 8 | 13 |

**Tail processing:** `i=3 < n=5` → add `a[3]=10` and `a[4]=12`:
`sumA = 0 + 10 + 12 = 22`

**Final:** `result += max(sumA=22, sumB=8) = 22` → `13 + 22 = 35` ✅

**Output:** `35` ✅ — matches exactly. Notice the winning path uses `1, 5` from `b`,
switches to `a` at the common element `7`, then rides `a` to the end (`10, 12`) —
exactly the path described in the official explanation.

---

### Example 2 — `a = [1,2,3]`, `b = [3,4,5]` → Expected Output: `15`

| Step | i | j | Comparison | Action | sumA | sumB | result |
|:----:|:-:|:-:|:----------:|--------|:----:|:----:|:------:|
| 1 | 0→1 | 0 | a[0]=1, b[0]=3 | a<b: sumA+=1 | 1 | 0 | 0 |
| 2 | 1→2 | 0 | a[1]=2, b[0]=3 | a<b: sumA+=2 | 3 | 0 | 0 |
| 3 | 2→3 | 0→1 | a[2]=3, b[0]=3 | common! result+=max(3,0)+3=6 | 0 | 0 | **6** |
| — | 3 | 1 | i=3=n, loop ends | exit | 0 | 0 | 6 |

**Tail processing:** `j=1 < m=3` → add `b[1]=4` and `b[2]=5`:
`sumB = 0 + 4 + 5 = 9`

**Final:** `result += max(sumA=0, sumB=9) = 9` → `6 + 9 = 15` ✅

**Output:** `15` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n + m)` | Single merge-style pass; each pointer advances at most once per element |
| **Space** | `O(1)` | Only a handful of running variables — no extra arrays |

For max constraints `n = m = 10⁴`: trivially fast.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Two-pointer merge with segment-sum comparison at sync points |
| **Common Elements = Bridges** | They're the only valid places to switch arrays — everything between two bridges is a forced single-array stretch |
| **Greedy Choice at Each Bridge** | Always take the larger accumulated segment sum — you can't combine both paths for the same stretch |
| **Return Type `long`** | Sum can reach `~10⁴ × 10⁵ = 10⁹`, safely within `long` but tight for `int` — use `long` defensively |
| **Interview Relevance** | Classic Amazon-tagged two-pointer/merge problem — tests handling of "sync point" greedy logic |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Merge Two Sorted Arrays | GFG / LC | The base merge technique used here |
| Shortest Common Supersequence | LeetCode #1092 | Similar sync-point thinking on two sequences |
| Max Sum of Non-Adjacent Elements | GFG | Different problem, same "choose the better path" greedy flavor |
| Path with Maximum Gold | LeetCode #1219 | Different domain, same "maximize path sum" theme |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Try All Switch Combinations — Exponential

For every common element, recursively try both "stay" and "switch" options, exploring
all possible paths.

```java
// Exponential — explores every combination of switches
// Not practical to implement cleanly without the two-pointer insight;
// omitted here since it offers no real learning value over brute enumeration.
```

> ❌ **Verdict:** TLE — exponential blowup with number of common elements.

---

### 2. ✅ Two-Pointer Merge with Segment Sums (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n+m)` time, `O(1)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force (all switch paths) | Exponential | O(n+m) | ❌ Infeasible |
| **Two-Pointer Merge** | **O(n+m)** | **O(1)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#two-pointers` `#arrays` `#greedy` `#medium` `#amazon` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 6, 2026
