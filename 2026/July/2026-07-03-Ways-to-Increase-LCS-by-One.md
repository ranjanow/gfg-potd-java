# 🔗 Ways to Increase LCS by One

## Problem Link
🔗 [Ways to Increase LCS by One – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/count-ways-to-increase-lcs-length-of-two-strings-by-one2236/1)

---

## Difficulty
**Medium** | Accuracy: `22.73%` | Submissions: `3K+` | Points: `4`

---

## Tags
- Dynamic Programming
- Strings
- LCS (Longest Common Subsequence)
- Prefix/Suffix DP

---

## Problem Summary

Given two strings `s1` and `s2` of lengths `n1` and `n2`, find the number of ways to
insert **exactly one character** into `s1` such that the length of the
**Longest Common Subsequence (LCS)** of the two strings increases by **exactly 1**.

Each distinct `(position, character)` combination that achieves this counts as
**one way**.

**Constraints:** `1 ≤ n1, n2 ≤ 100`
**Expected:** Time `O(n1 × n2)` · Space `O(n1 × n2)`

---

## Intuition

### Why Brute Force Fails

Trying every insertion position (`n1 + 1` gaps) × every possible character (26)
× recomputing LCS from scratch (`O(n1 × n2)`) each time gives
`O(26 × n1 × (n1 × n2))` — far too slow, and wasteful since most of the LCS
computation is repeated work.

### The Prefix + Suffix LCS Trick

Split the problem into two independently precomputable pieces:

```
pre[i][j] = LCS length of s1[0..i-1]  and  s2[0..j-1]   (standard LCS DP)
suf[i][j] = LCS length of s1[i..n1-1] and  s2[j..n2-1]   (LCS of suffixes)
```

**Key Lemma:** For any insertion gap `i` in `s1` (0-indexed, `0 ≤ i ≤ n1`) and any
position `j` in `s2` where we "use" `s2[j]` as the newly inserted matching character:

```
combinedLCS(i, j) = pre[i][j] + 1 + suf[i][j+1]
```

This represents: *best LCS achievable in the left part* + *the new inserted match*
+ *best LCS achievable in the right part*.

**Crucially:** `pre[i][j] + suf[i][j+1] ≤ LCS(s1, s2)` always holds (you can never
beat the true optimum by splitting). So inserting character `c = s2[j]` at gap `i`
increases the LCS by exactly 1 **if and only if**:

```
max over all j where s2[j] == c  of  (pre[i][j] + suf[i][j+1])  ==  originalLCS
```

### Why We Check Per-Character, Not Per-(i, j)

The problem counts **ways**, not raw `(i, j)` pairs. If the same character `c`
could achieve the increase via multiple different `j` positions in `s2`, it's
still **one valid way** for that `(gap i, character c)` combination — the resulting
string after insertion is identical regardless of which `s2[j]` "enabled" it.

---

## Approach

1. Compute `pre[i][j]` — standard bottom-up LCS DP table, size `(n1+1) × (n2+1)`.
2. Compute `suf[i][j]` — LCS of suffixes, built from bottom-right to top-left,
   same table size.
3. Let `originalLCS = pre[n1][n2]`.
4. For each gap position `i` from `0` to `n1`:
   - Create `int[26] bestForChar`, initialised to `-1` (sentinel: "no match found yet").
   - For each `j` from `0` to `n2-1`:
     - `c = s2.charAt(j) - 'a'`
     - `candidate = pre[i][j] + suf[i][j+1]`
     - `bestForChar[c] = max(bestForChar[c], candidate)`
   - For each character `c` from `0` to `25`:
     - If `bestForChar[c] == originalLCS` → increment answer by 1.
5. Return the total count.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int waysToIncreaseLCSBy1(String S1, String S2)`

```java
class Solution {
    public int waysToIncreaseLCSBy1(String s1, String s2) {
        int n1 = s1.length(), n2 = s2.length();

        // pre[i][j] = LCS length of s1[0..i-1] and s2[0..j-1]
        int[][] pre = new int[n1 + 1][n2 + 1];
        for (int i = 1; i <= n1; i++) {
            for (int j = 1; j <= n2; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    pre[i][j] = pre[i - 1][j - 1] + 1;
                } else {
                    pre[i][j] = Math.max(pre[i - 1][j], pre[i][j - 1]);
                }
            }
        }

        // suf[i][j] = LCS length of s1[i..n1-1] and s2[j..n2-1]
        int[][] suf = new int[n1 + 1][n2 + 1];
        for (int i = n1 - 1; i >= 0; i--) {
            for (int j = n2 - 1; j >= 0; j--) {
                if (s1.charAt(i) == s2.charAt(j)) {
                    suf[i][j] = suf[i + 1][j + 1] + 1;
                } else {
                    suf[i][j] = Math.max(suf[i + 1][j], suf[i][j + 1]);
                }
            }
        }

        int originalLCS = pre[n1][n2];
        int ans = 0;

        // Try every insertion gap (0 to n1) in s1
        for (int i = 0; i <= n1; i++) {
            int[] bestForChar = new int[26];
            java.util.Arrays.fill(bestForChar, -1); // -1 = character not seen in s2

            // For each character position in s2, track the best achievable
            // combined LCS if that character were inserted at gap i
            for (int j = 0; j < n2; j++) {
                int c = s2.charAt(j) - 'a';
                int candidate = pre[i][j] + suf[i][j + 1];
                bestForChar[c] = Math.max(bestForChar[c], candidate);
            }

            // A character is a valid insertion at gap i if its best achievable
            // combined LCS exactly equals the original LCS (i.e., +1 overall)
            for (int c = 0; c < 26; c++) {
                if (bestForChar[c] == originalLCS) {
                    ans++;
                }
            }
        }

        return ans;
    }
}
```

---

## Dry Run

### Example 1 — `s1 = "abab"`, `s2 = "abc"` → Expected Output: `3`

`n1 = 4`, `n2 = 3`, `originalLCS = pre[4][3] = 2`

**pre[i][j] table:**

|   | "" | a | b | c |
|:-:|:--:|:-:|:-:|:-:|
| "" | 0 | 0 | 0 | 0 |
| a  | 0 | 1 | 1 | 1 |
| ab | 0 | 1 | 2 | 2 |
| aba| 0 | 1 | 2 | 2 |
| abab| 0| 1 | 2 | 2 |

**suf[i][j] table:**

|   | "" (j=3) | c (j=2) | bc (j=1) | abc (j=0) |
|:-:|:--------:|:-------:|:--------:|:---------:|
| "" (i=4)   | 0 | 0 | 0 | 0 |
| b (i=3)    | 0 | 0 | 1 | 1 |
| ab (i=2)   | 0 | 0 | 1 | 2 |
| bab (i=1)  | 0 | 0 | 1 | 2 |
| abab (i=0) | 0 | 0 | 1 | 2 |

**Checking each gap `i` for character `'c'` (index 2 in s2):**

| Gap `i` | pre[i][2] | suf[i][3] | candidate | == originalLCS(2)? |
|:-------:|:---------:|:---------:|:---------:|:-------------------:|
| 0 | 0 | 0 | 0 | ❌ |
| 1 | 1 | 0 | 1 | ❌ |
| 2 | 2 | 0 | **2** | ✅ → `"ab"+c+"ab"` = **"abcab"** |
| 3 | 2 | 0 | **2** | ✅ → `"aba"+c+"b"` = **"abacb"** |
| 4 | 2 | 0 | **2** | ✅ → `"abab"+c` = **"ababc"** |

Characters `'a'` and `'b'` never reach `candidate == 2` at any gap (verified: best is always 1).

**Total valid ways:** `3` ✅ — exactly matches `"abcab"`, `"abacb"`, `"ababc"`.

---

### Example 2 — `s1 = "abcabc"`, `s2 = "abcd"` → Expected Output: `4`

`n1 = 6`, `n2 = 4`, `originalLCS = 3` (LCS = "abc")

**Spot-check for gap `i = 3`, character `'d'` (index 3 in s2):**
```
pre[3][3] = LCS("abc", "abc") = 3
suf[3][4] = LCS("abc", "")    = 0
candidate = 3 + 0 = 3 = originalLCS ✅
```
→ Insert `'d'` after `"abc"` → `"abc"+"d"+"abc"` = **"abcdabc"** ✓ matches given example.

Repeating this check for gaps `i = 4, 5, 6` (each after progressively more of `s1`)
similarly yields `candidate = 3` for character `'d'`, giving the remaining 3 ways:
`"abcadbc"`, `"abcabdc"`, `"abcabcd"`.

**Total valid ways:** `4` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n1 × n2)` | `pre` table: O(n1×n2). `suf` table: O(n1×n2). Counting step: O(n1 × (n2 + 26)) ≈ O(n1×n2) since 26 is constant |
| **Space** | `O(n1 × n2)` | Two 2D tables `pre` and `suf`, each of size `(n1+1) × (n2+1)` |

For max constraints `n1 = n2 = 100`: `10⁴` cells per table — trivially fast.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Prefix LCS + Suffix LCS split, combined via a bridging character |
| **Key Lemma** | `pre[i][j] + suf[i][j+1] ≤ LCS(s1,s2)` always — equality signals a valid insertion |
| **Per-Character Tracking** | Track the *best* achievable value per character (not per raw j) to correctly count distinct "ways" |
| **Sentinel Value `-1`** | Distinguishes "character never appears in s2" from "candidate value 0" |
| **Interview Relevance** | Advanced LCS variant — tests whether you can decompose a global optimum into prefix+suffix parts |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Longest Common Subsequence | GFG / LC #1143 | The base pre[][] DP used here |
| Count Ways to Increase LCS by One (Editorial) | GFG Article | This exact problem's official technique |
| Palindrome Partitioning II | LeetCode #132 | Similar prefix/suffix DP decomposition style |
| Split Array Largest Sum | LeetCode #410 | Another prefix-based optimal split problem |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Try Every Insertion — O(26 × n1² × n2)

For every gap and every character, build the new string and recompute LCS from scratch.

```java
// TLE — recomputes O(n1*n2) LCS for every (gap, char) combination
class Solution {
    public int waysToIncreaseLCSBy1(String s1, String s2) {
        int n1 = s1.length();
        int originalLCS = lcs(s1, s2);
        int ans = 0;

        for (int i = 0; i <= n1; i++) {
            for (char c = 'a'; c <= 'z'; c++) {
                String newS1 = s1.substring(0, i) + c + s1.substring(i);
                if (lcs(newS1, s2) == originalLCS + 1) ans++;
            }
        }
        return ans;
    }

    private int lcs(String a, String b) {
        int n = a.length(), m = b.length();
        int[][] dp = new int[n + 1][m + 1];
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= m; j++)
                dp[i][j] = a.charAt(i - 1) == b.charAt(j - 1)
                    ? dp[i - 1][j - 1] + 1
                    : Math.max(dp[i - 1][j], dp[i][j - 1]);
        return dp[n][m];
    }
}
```

> ❌ **Verdict:** TLE — `O(26 × n1² × n2)` ≈ `26 × 100² × 100` = `2.6×10⁷` per single
> call is borderline, but conceptually wasteful and fails on larger hidden constraints.

---

### 2. ✅ Prefix + Suffix LCS Decomposition (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n1×n2)` time and space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force (recompute LCS) | O(26×n1²×n2) | O(n1×n2) | ❌ Wasteful, slow |
| **Prefix + Suffix LCS** | **O(n1×n2)** | **O(n1×n2)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#dynamic-programming` `#lcs` `#strings` `#medium` `#prefix-suffix-dp` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 3, 2026
