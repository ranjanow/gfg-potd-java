# 🔤 Max Gap Between Two Same Characters

## Problem Link
🔗 [Max Gap Between Two Same Characters – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/maximum-number-of-characters-between-any-two-same-character4552/1)

---

## Difficulty
**Easy** | Accuracy: `47.57%` | Submissions: `11K+` | Points: `2` | Company: `Zoho`

---

## Tags
- Strings
- Hashing
- Arrays

---

## Problem Summary

Given a string `s` consisting of **lowercase English letters**, find the **maximum
number of characters** between any two **identical** characters. If no character
repeats, return `-1`.

**Constraints:** `1 ≤ |s| ≤ 10⁵`
**Expected:** Time `O(|s|)` · Space `O(1)`

---

## Intuition

### Key Observations

1. **Only the first and last occurrence of each character matter.**
   For any repeated character, the widest possible gap between two of its
   occurrences is always achieved between its **first** and **last** appearance —
   any other pair of occurrences of the same character lies strictly inside
   that range, giving a smaller gap.

2. **Gap formula:** If a character's first occurrence is at index `first` and its
   last occurrence is at index `last`, the number of characters strictly
   **between** them is:
   ```
   gap = last - first - 1
   ```

3. **Since only 26 lowercase letters exist**, we can track the first occurrence
   of each letter in a fixed-size array of `26` slots — this gives us **O(1)
   auxiliary space** (constant, independent of string length).

4. **Single pass suffices:** As we scan `s` left to right, the first time we see
   a character, we record its index. Every subsequent time we see the same
   character, we compute the gap against its **recorded first occurrence** and
   update the running maximum — the last time we see it during the scan will
   naturally give the correct "last occurrence" gap.

### Verification with Example

`s = "socks"`:
```
s(0) o(1) c(2) k(3) s(4)
```
`'s'` first appears at index `0`, and appears again at index `4`.
`gap = 4 - 0 - 1 = 3` → matches expected output `3` ✅

---

## Approach

1. Create `int[] firstOccurrence = new int[26]`, initialised to `-1`.
2. Initialise `maxGap = -1`.
3. Traverse `s` from index `0` to `n-1`:
   - Let `idx = s.charAt(i) - 'a'`.
   - If `firstOccurrence[idx] == -1`: set `firstOccurrence[idx] = i` (first time seen).
   - Else: compute `gap = i - firstOccurrence[idx] - 1` and update
     `maxGap = max(maxGap, gap)`.
4. Return `maxGap`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int maxCharGap(String s)`

```java
class Solution {
    public int maxCharGap(String s) {
        int n = s.length();

        // firstOccurrence[c] = index of first appearance of character c, or -1
        int[] firstOccurrence = new int[26];
        java.util.Arrays.fill(firstOccurrence, -1);

        int maxGap = -1;

        for (int i = 0; i < n; i++) {
            int idx = s.charAt(i) - 'a';

            if (firstOccurrence[idx] == -1) {
                // First time seeing this character — record its position
                firstOccurrence[idx] = i;
            } else {
                // Repeated character — compute gap against its first occurrence
                int gap = i - firstOccurrence[idx] - 1;
                maxGap = Math.max(maxGap, gap);
            }
        }

        return maxGap;
    }
}
```

---

## Dry Run

### Example 1 — `s = "socks"` → Expected Output: `3`

| i | char | idx | firstOccurrence[idx] before | Action | maxGap |
|:-:|:----:|:---:|:----------------------------:|--------|:------:|
| 0 | 's' | 18 | -1 | Record: firstOccurrence[18]=0 | -1 |
| 1 | 'o' | 14 | -1 | Record: firstOccurrence[14]=1 | -1 |
| 2 | 'c' | 2  | -1 | Record: firstOccurrence[2]=2 | -1 |
| 3 | 'k' | 10 | -1 | Record: firstOccurrence[10]=3 | -1 |
| 4 | 's' | 18 | 0  | gap = 4-0-1=3 → maxGap=3 | **3** |

**Output:** `3` ✅

---

### Example 2 — `s = "for"` → Expected Output: `-1`

| i | char | idx | firstOccurrence[idx] before | Action | maxGap |
|:-:|:----:|:---:|:----------------------------:|--------|:------:|
| 0 | 'f' | 5 | -1 | Record: firstOccurrence[5]=0 | -1 |
| 1 | 'o' | 14 | -1 | Record: firstOccurrence[14]=1 | -1 |
| 2 | 'r' | 17 | -1 | Record: firstOccurrence[17]=2 | -1 |

No character repeats → `maxGap` stays at its initial value.

**Output:** `-1` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n)` | Single pass through the string; each step is O(1) array lookup/update |
| **Space** | `O(1)` | Fixed-size array of `26` slots — independent of input length |

For max constraints `n = 10⁵`: trivially fast.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Track first occurrence per character; compute gap on repeat |
| **Why First-to-Last Is Optimal** | Any pair of same-character occurrences other than (first, last) gives a smaller or equal gap |
| **Fixed Alphabet Trick** | Only 26 lowercase letters → O(1) space using a small fixed array instead of a HashMap |
| **Single-Pass Sufficiency** | No need to store all occurrence indices — only the first one, updated max as we scan |
| **Interview Relevance** | Common warm-up pattern: "first occurrence tracking" appears across many string problems |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Partition Labels | LeetCode #763 | Uses last-occurrence tracking of characters |
| Longest Substring Without Repeating Characters | LeetCode #3 | Same first-occurrence array idea, different goal |
| First Unique Character in a String | LeetCode #387 | Frequency + first-occurrence tracking |
| Isomorphic Strings | LeetCode #205 | Character position mapping pattern |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Check All Pairs — O(n²)

For every pair of indices `(i, j)` with `s[i] == s[j]`, compute the gap and track the maximum.

```java
// O(n²) — wasteful, checks all pairs even for characters seen many times
class Solution {
    public int maxCharGap(String s) {
        int n = s.length();
        int maxGap = -1;

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (s.charAt(i) == s.charAt(j)) {
                    maxGap = Math.max(maxGap, j - i - 1);
                    break; // only the FIRST match with i matters for maximizing gap
                }
            }
        }
        return maxGap;
    }
}
```

> ❌ **Verdict:** Works but `O(n²)` worst case is too slow for `n = 10⁵` (`10¹⁰` operations).

---

### 2. ✅ First-Occurrence Array — Single Pass (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n)` time, `O(1)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force (all pairs) | O(n²) | O(1) | ❌ TLE for n=10⁵ |
| **First-Occurrence Array** | **O(n)** | **O(1)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#strings` `#hashing` `#easy` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 5, 2026
