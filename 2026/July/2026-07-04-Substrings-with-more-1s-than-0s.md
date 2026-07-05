# 1️⃣ Substrings with more 1's than 0's

## Problem Link
🔗 [Substrings with more 1's than 0's – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/count-the-substring--170645/1)

---

## Difficulty
**Hard** | Accuracy: `57.87%` | Submissions: `21K+` | Points: `8`

---

## Tags
- Strings
- Dynamic Programming
- Prefix Sum
- Data Structures
- Algorithms

---

## Problem Summary

Given a binary string `s` consisting only of `0`s and `1`s, count the number of
**substrings** that have **more `1`s than `0`s**.

**Constraints:** `1 ≤ |s| ≤ 6 × 10⁴`
**Expected:** Time `O(|s|)` · Space `O(|s|)`

---

## Intuition

### Step 1 — Convert to a Prefix Sum Problem

Map `'1' → +1` and `'0' → -1`. Build a prefix-sum array `P[0..n]`:

```
P[0] = 0
P[i] = P[i-1] + (s[i-1] == '1' ? +1 : -1)
```

For substring `s[i..j-1]` (0-indexed, corresponding to prefix pair `(i, j)`):

```
count(1s) - count(0s) in s[i..j-1]  =  P[j] - P[i]
```

We want this difference `> 0`, i.e. **`P[i] < P[j]`**.

> **The problem reduces to:** count pairs `(i, j)` with `0 ≤ i < j ≤ n` such that `P[i] < P[j]`.

### Verification with Example 1

`s = "011"` → `P = [0, -1, 0, 1]`

| Pair (i,j) | P[i] | P[j] | P[i]<P[j]? | Substring |
|:----------:|:----:|:----:|:----------:|:---------:|
| (0,1) | 0 | -1 | ❌ | "0" |
| (0,2) | 0 | 0  | ❌ | "01" |
| (0,3) | 0 | 1  | ✅ | **"011"** |
| (1,2) | -1| 0  | ✅ | **"1"** |
| (1,3) | -1| 1  | ✅ | **"11"** |
| (2,3) | 0 | 1  | ✅ | **"1"** |

**4 valid pairs** → matches expected output `4` exactly ✅

### Step 2 — Why This Is Normally O(n log n), and How to Get O(n)

Counting "pairs with `P[i] < P[j]`, `i < j`" is a classic **counting-inversions-style**
problem, usually solved via a **Fenwick Tree / BIT** with coordinate compression in
`O(n log n)`. But the problem expects **O(n)** — and here's the trick that makes it possible:

> **Since `P[j] = P[j-1] ± 1` (consecutive prefix values always differ by exactly 1),**
> the "number of previous values less than the current value" can be updated
> **incrementally in O(1) per step**, instead of re-querying from scratch.

### The O(1)-Per-Step Update

Let `L(v, freq)` = count of previously-seen prefix values strictly less than `v`.
Maintain a running variable `curL` = this count for the **previous** prefix value,
and a frequency array `freq[]` of all prefix values inserted so far.

When moving from `curP` to `newP = curP ± 1`:

```
If newP = curP + 1:
    L(newP, freq_before_inserting_curP) = curL + freq[curP]
    (moving up by 1 absorbs exactly the frequency AT curP)

If newP = curP - 1:
    L(newP, freq_before_inserting_curP) = curL - freq[newP]
    (moving down by 1 removes exactly the frequency AT the new lower value)
```

Then insert `curP` into `freq[]`, and add `1` more if `curP < newP` (since `curP`
itself just became a valid "earlier, smaller" value for comparison against `newP`).

This gives an **O(1) amortized update per character** → overall **O(n) time**.

---

## Approach

1. Let `n = s.length()`, `offset = n` (to allow negative array indices).
2. Create `int[] freq = new int[2n+1]` (all zeros).
3. Initialize `curP = 0`, `curL = 0`, `ans = 0`.
4. For `k` from `1` to `n`:
   - `step = (s.charAt(k-1) == '1') ? 1 : -1`
   - `newP = curP + step`
   - If `step == 1`: `LBeforeInsert = curL + freq[curP + offset]`
   - Else: `LBeforeInsert = curL - freq[newP + offset]`
   - Insert `curP` into frequency table: `freq[curP + offset]++`
   - `LAfterInsert = LBeforeInsert + (curP < newP ? 1 : 0)`
   - `ans += LAfterInsert`
   - Update: `curP = newP`, `curL = LAfterInsert`
5. Return `ans`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int countSubstring(String S)` — returns `int`,
> not `long`. Since max possible answer (`~1.8×10⁹` for `n=6×10⁴`) still fits within
> `int` range (`~2.1×10⁹`), we compute internally with `long` for safety and cast
> down to `int` only at the very end.

```java
class Solution {
    public int countSubstring(String s) {
        int n = s.length();
        int offset = n; // shift so prefix values (range -n..n) map to array indices 0..2n

        int[] freq = new int[2 * n + 1];

        long ans = 0;
        int curP = 0;      // current prefix value P[k-1]
        long curL = 0;      // L(P[k-1], values inserted so far, excluding P[k-1] itself)

        for (int k = 1; k <= n; k++) {
            int step = (s.charAt(k - 1) == '1') ? 1 : -1;
            int newP = curP + step;

            long lBeforeInsert;
            if (step == 1) {
                // Moving up by 1: absorb the frequency count sitting exactly at curP
                lBeforeInsert = curL + freq[curP + offset];
            } else {
                // Moving down by 1: remove the frequency count sitting exactly at newP
                lBeforeInsert = curL - freq[newP + offset];
            }

            // Insert curP (= P[k-1]) into the frequency table for future steps
            freq[curP + offset]++;

            // curP itself is now a valid "earlier" value — count it if it's < newP
            long lAfterInsert = lBeforeInsert + (curP < newP ? 1 : 0);

            ans += lAfterInsert;

            curP = newP;
            curL = lAfterInsert;
        }

        return (int) ans; // safe: max ~1.8×10⁹ fits within int range (~2.1×10⁹)
    }
}
```

---

## Dry Run

### Example 1 — `s = "011"` → Expected Output: `4`

`n = 3`, `offset = 3`, `freq` size `7`

| k | char | step | curP→newP | lBeforeInsert | freq[curP] updated | curP<newP? | lAfterInsert (added to ans) | Running ans |
|:-:|:----:|:----:|:---------:|:--------------:|:-------------------:|:----------:|:----------------------------:|:-----------:|
| 1 | '0' | -1 | 0 → -1 | `0 - freq[-1]=0` = 0 | freq[0]=1 | 0<-1? ❌ | 0 | 0 |
| 2 | '1' | +1 | -1 → 0 | `0 + freq[-1]=0` = 0 | freq[-1]=1 | -1<0? ✅ | 1 | 1 |
| 3 | '1' | +1 | 0 → 1  | `1 + freq[0]=1` = 2 | freq[0]=2 | 0<1? ✅ | 3 | **4** |

**Output:** `4` ✅ — matches exactly.

---

### Example 2 — `s = "0000"` → Expected Output: `0`

Since every step decreases `P` by 1 (strictly decreasing sequence `0,-1,-2,-3,-4`),
**no** prefix value is ever less than a later one. Every `lAfterInsert` stays `0`
throughout all 4 iterations.

**Output:** `0` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n)` | Single pass through the string; each step does O(1) frequency lookups/updates |
| **Space** | `O(n)` | `freq[]` array of size `2n+1` |

For max constraints `n = 6×10⁴`: trivially fast, well within limits.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Reduction** | Map `1→+1, 0→-1`; substring condition becomes `P[i] < P[j]` |
| **Why Normally O(n log n)** | Counting `P[i]<P[j]` pairs is generally an inversion-counting problem (BIT/Fenwick) |
| **The ±1 Step Trick** | Since prefix values change by exactly ±1 each step, "count less than" updates in O(1) via frequency deltas |
| **Order of Operations Matters** | Query using OLD frequency state, THEN insert the current value — inserting too early corrupts the count |
| **Return Type `long`** | Max possible answer ≈ `n²/2` ≈ `1.8×10⁹` for `n=6×10⁴` — very close to `int` overflow, use `long` defensively |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Count Of Substrings With More 1s Than 0s | GFG Article | This exact problem's official editorial technique |
| Count of Smaller Numbers After Self | LeetCode #315 | Classic inversion counting (usually needs BIT) |
| Subarray Sums Divisible by K | LeetCode #974 | Same prefix-sum-bucket counting style |
| Count Inversions in an Array | GFG | General O(n log n) version without the ±1 shortcut |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Check Every Substring — O(n²)

For every substring, count 1s and 0s directly (or use a precomputed prefix sum for O(1) diff lookup).

```java
class Solution {
    public int countSubstring(String s) {
        int n = s.length();
        int[] prefix = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            prefix[i] = prefix[i - 1] + (s.charAt(i - 1) == '1' ? 1 : -1);
        }

        long ans = 0;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j <= n; j++) {
                if (prefix[j] - prefix[i] > 0) ans++;
            }
        }
        return (int) ans;
    }
}
```

> ❌ **Verdict:** TLE — `O(n²)` = `3.6×10⁹` operations for `n=6×10⁴`. Too slow.

---

### 2. 💡 Fenwick Tree (BIT) with Coordinate Compression — O(n log n)

Insert prefix values into a BIT one at a time; query "count of inserted values less
than current" via prefix sum on the BIT.

```java
class Solution {
    private int[] bit;
    private int size;

    private void update(int i) {
        for (; i <= size; i += i & (-i)) bit[i]++;
    }

    private int query(int i) {
        int sum = 0;
        for (; i > 0; i -= i & (-i)) sum += bit[i];
        return sum;
    }

    public int countSubstring(String s) {
        int n = s.length();
        int offset = n + 1;
        size = 2 * n + 2;
        bit = new int[size + 1];

        long ans = 0;
        int curP = 0;
        update(curP + offset); // insert P[0]

        for (int k = 1; k <= n; k++) {
            int step = (s.charAt(k - 1) == '1') ? 1 : -1;
            curP += step;
            // count of previously inserted values strictly less than curP
            ans += query(curP + offset - 1);
            update(curP + offset);
        }
        return (int) ans;
    }
}
```

> ✅ **Verdict:** Accepted — `O(n log n)` time, `O(n)` space. Correct but slightly
> slower than the optimal O(n) trick; good fallback if the ±1 insight isn't spotted.

---

### 3. ✅ O(1)-Per-Step Frequency Update (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n)` time, `O(n)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(n) | ❌ TLE for n=6×10⁴ |
| Fenwick Tree (BIT) | O(n log n) | O(n) | ✅ Correct, safe fallback |
| **±1 Step Frequency Trick** | **O(n)** | **O(n)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#prefix-sum` `#strings` `#dynamic-programming` `#hard` `#fenwick-tree` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 4, 2026
