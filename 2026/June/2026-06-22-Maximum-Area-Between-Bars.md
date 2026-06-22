# 📊 Maximum Area Between Bars

## Problem Link
🔗 [Maximum Area Between Bars – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/maximum-area-between-bars/1)

---

## Difficulty
**Medium** | Accuracy: `39.48%` | Submissions: `15K+` | Points: `4`

---

## Tags
- Array
- Two Pointers
- Greedy

---

## Problem Summary
Given an integer array `height[]`, where `height[i]` is the height of the i-th bar in a row,
find the **maximum rectangular area** that can be formed by selecting **any two bars**.

The critical detail: the **width** of the rectangle equals the **number of bars strictly between**
the two selected bars at their original positions — i.e., `j - i - 1`, **not** `j - i`.

> **Area Formula:** `min(height[i], height[j]) × (j - i - 1)`

---

## Intuition

### Key Observations

1. **Width ≠ distance between indices.**
   Unlike *Container With Most Water* (where width = `j - i`), here width = `j - i - 1`
   (bars *between* the chosen pair, not including them).

2. **Brute force is O(n²)** — check all `n*(n-1)/2` pairs. This TLEs for `n = 10⁵`.

3. **Two-pointer greedy** shrinks the window from the widest possible span inward:
   - Start at `left = 0`, `right = n - 1` (maximum width).
   - Always move the **shorter** of the two boundary bars inward.

### Why Greedy Is Correct

Suppose `height[left] ≤ height[right]`.
For any inner right pointer `k` (`left < k < right`):

```
area(left, k) = min(height.get(left), height.get(k)) × (k - left - 1)
              ≤ height.get(left) × (k - left - 1)
              ≤ height.get(left) × (right - left - 1)   ← current area's upper bound
```

So keeping `left` fixed cannot beat the current pair; we safely **move `left` inward**
to discover potentially taller bars. The symmetric argument holds for the right side.

---

## Approach

1. Initialize `left = 0`, `right = n - 1`, `maxArea = 0`.
2. While `left < right`:
   - Compute `width = right - left - 1`.
   - Compute `h = min(height[left], height[right])`.
   - Update `maxArea = max(maxArea, h × width)`.
   - If `height[left] < height[right]` → `left++`
   - Else → `right--`
3. Return `maxArea`.

---

## Java Solution

> ⚠️ **GFG Driver Note:** GFG passes `height` as `List<Integer>`, **not** `int[]`.
> Use `.size()` / `.get(i)` instead of `.length` / `[i]`.

```java
import java.util.List;

class Solution {
    public int maxArea(List<Integer> height) {
        int left = 0, right = height.size() - 1;
        int maxArea = 0;

        while (left < right) {
            // Width = bars strictly between the two selected bars
            int width = right - left - 1;
            int h = Math.min(height.get(left), height.get(right));

            maxArea = Math.max(maxArea, h * width);

            // Move the shorter bar inward to seek a potentially taller boundary
            if (height.get(left) < height.get(right)) {
                left++;
            } else {
                right--;
            }
        }

        return maxArea;
    }
}
```

---

## Dry Run

**Input:** `height[] = [2, 5, 4, 3, 7]`

| Step | `left` | `right` | `h[left]` | `h[right]` | `width` | `area` | `maxArea` | Action   |
|:----:|:------:|:-------:|:---------:|:----------:|:-------:|:------:|:---------:|:--------:|
|  1   |   0    |    4    |     2     |      7     |    3    |   6    |     6     | `left++` |
|  2   |   1    |    4    |     5     |      7     |    2    |  10    |    10     | `left++` |
|  3   |   2    |    4    |     4     |      7     |    1    |   4    |    10     | `left++` |
|  4   |   3    |    4    |     3     |      7     |    0    |   0    |    10     | `left++` |
|  5   |   4    |    4    |     —     |      —     |    —    |   —    |    10     | **STOP** |

**Output:** `10`

**Verification:** Bars at index `1` (height `5`) and index `4` (height `7`).
Bars between them → indices `2, 3` → **count = 2**.
Area = `min(5, 7) × 2 = 5 × 2 = 10` ✅

---

## Complexity Analysis

### Time Complexity
**`O(n)`** — `left` and `right` together traverse each element at most once; the loop runs at most `n - 1` iterations.

### Space Complexity
**`O(1)`** — Only three integer variables used; no auxiliary data structures.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Two Pointers (Shrinking Window) |
| **Greedy Property** | Moving the shorter bar inward never skips the optimal pair |
| **Width Formula** | `j - i - 1` — bars *between*, not the span itself |
| **vs. LeetCode 11** | Only difference is `j - i - 1` instead of `j - i`; greedy proof is identical |
| **Interview Value** | Tests ability to *adapt* a known pattern when the formula changes slightly |

### Similar / Related Problems
| Problem | Platform | Relevance |
|---|---|---|
| Container With Most Water | LeetCode #11 | Same two-pointer strategy, width = `j - i` |
| Trapping Rain Water | LeetCode #42 | Two-pointer / stack on bar arrays |
| Largest Rectangle in Histogram | LeetCode #84 | Stack-based approach on bar arrays |

---

## Alternative Approaches

### 1. 🐢 Brute Force — `O(n²)` Time, `O(1)` Space
Enumerate every pair `(i, j)` and compute `min(height[i], height[j]) × (j - i - 1)`.

```java
// Brute Force — TLE for n = 10^5
int maxArea = 0;
for (int i = 0; i < height.size() - 1; i++) {
    for (int j = i + 1; j < height.size(); j++) {
        int area = Math.min(height.get(i), height.get(j)) * (j - i - 1);
        maxArea = Math.max(maxArea, area);
    }
}
return maxArea;
```
> ❌ **Verdict:** TLE — `O(n²)` is ~10¹⁰ operations for `n = 10⁵`.

---

### 2. ✅ Two Pointer (Optimal) — `O(n)` Time, `O(1)` Space
Greedy shrinking window as described in the main solution above.
> ✅ **Verdict:** Accepted — matches expected `O(n)` / `O(1)` complexities.

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#two-pointers` `#greedy` `#arrays` `#medium` `#competitive-programming` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/your-username/GeeksforGeeks-POTD-Java)
> 📅 **Date:** June 22, 2026
