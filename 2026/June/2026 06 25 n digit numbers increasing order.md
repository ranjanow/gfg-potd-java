# 🔢 N-Digit Numbers with Digits in Increasing Order

## Problem Link
🔗 [N-Digit Numbers with Digits in Increasing Order – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/n-digit-numbers-with-digits-in-increasing-order5903/1)

---

## Difficulty
**Easy** | Topics: `Backtracking` `Recursion` `Combinatorics`

---

## Tags
- Backtracking
- Recursion
- Combinatorics
- Digit DP
- Arrays

---

## Problem Summary

Given an integer `n`, return **all n-digit numbers** (in increasing order) such that
their digits are in **strictly increasing order from left to right**.

- **`n = 1`:** include digit `0` — output is `[0, 1, 2, ..., 9]` (10 numbers).
- **`n ≥ 2`:** digits `1–9` only — no leading zero, so `0` never appears.
- The output list itself must be in **increasing numeric order**.

**Constraints:** `1 ≤ n ≤ 9`

---

## Intuition

### Key Observations

1. **Digits are chosen from {1..9} with no repetition, always in ascending order.**
   This is exactly choosing an **n-element combination** from the 9 available digits
   (1–9). For every such combination, there is exactly **one** valid arrangement
   (strictly increasing). So the answer has exactly **C(9, n)** elements.

2. **Why brute force (iterating all integers) is wasteful:**
   For n = 5, we'd scan up to 99,999 numbers and check each — O(upper_bound × digits).
   But there are only C(9, 5) = **126** valid numbers. Backtracking generates
   exactly those 126 without scanning anything invalid.

3. **Backtracking fits perfectly:**
   At each recursive level, we pick the next digit — always **strictly greater** than
   the previous one. We stop when we've picked `n` digits. No backtracking "reset"
   is needed because we never place a wrong digit; we just try the next candidate.

4. **The output is naturally sorted** because we iterate digits `1 → 9` at every
   level — smaller first-digits produce smaller numbers, so results come out in
   ascending order automatically. No sorting step needed.

### Recursion Tree Insight (n = 2)

```
Start (prevDigit = 0, so first digit starts from 1)
 ├── 1  → try 2..9  →  12, 13, 14, 15, 16, 17, 18, 19
 ├── 2  → try 3..9  →  23, 24, 25, 26, 27, 28, 29
 ├── 3  → try 4..9  →  34, 35, 36, 37, 38, 39
 ├── 4  → try 5..9  →  45, 46, 47, 48, 49
 ├── 5  → try 6..9  →  56, 57, 58, 59
 ├── 6  → try 7..9  →  67, 68, 69
 ├── 7  → try 8..9  →  78, 79
 └── 8  → try 9    →  89
Total = C(9, 2) = 36 numbers ✓
```

---

## Approach

1. Create an empty `ArrayList<Integer>` for results.
2. Set `startDigit = (n == 1) ? -1 : 0`.
3. Call `generate(currentNumber=0, prevDigit=startDigit, digitsLeft=n, result)`.
4. In the recursive function:
   - **Base case:** if `digitsLeft == 0` → add `currentNumber` to result and return.
   - **Recursive case:** for each digit `d` from `prevDigit + 1` to `9`:
     - Recurse with `currentNumber * 10 + d`, `prevDigit = d`, `digitsLeft - 1`.
5. Return the populated result list.

> **Why the ternary `(n == 1) ? -1 : 0`?**
> - `startDigit = -1` → loop starts at `d = 0` → includes `0` for single-digit case.
> - `startDigit =  0` → loop starts at `d = 1` → excludes `0` for multi-digit numbers
>   (an n-digit number with leading zero is not truly n digits when stored as `int`).

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `ArrayList<Integer> increasingNumbers(int n)`

```java
import java.util.*;

class Solution {

    // Backtracking helper
    // number     → integer built so far
    // prevDigit  → last digit placed
    // remaining  → how many more digits to place
    private void generate(int number, int prevDigit, int remaining,
                          ArrayList<Integer> result) {
        // Base case: n digits placed → valid number, add to result
        if (remaining == 0) {
            result.add(number);
            return;
        }

        // Try every digit strictly greater than the previous one
        for (int d = prevDigit + 1; d <= 9; d++) {
            generate(number * 10 + d, d, remaining - 1, result);
        }
    }

    public ArrayList<Integer> increasingNumbers(int n) {
        ArrayList<Integer> result = new ArrayList<>();

        // n=1 → include 0 (prevDigit=-1 lets loop start at d=0)
        // n≥2 → exclude 0 (prevDigit=0  lets loop start at d=1, no leading zero)
        int startDigit = (n == 1) ? -1 : 0;

        generate(0, startDigit, n, result);
        return result;
    }
}
```

---

## Dry Run

**Input:** `n = 2`

**Trace (first branch and a mid branch shown):**

| Call | `number` | `prevDigit` | `remaining` | Action |
|:----:|:--------:|:-----------:|:-----------:|--------|
| generate(0, 0, 2) | 0 | 0 | 2 | try d = 1..9 |
| → generate(1, 1, 1) | 1 | 1 | 1 | try d = 2..9 |
| → → generate(12, 2, 0) | 12 | 2 | 0 | ✅ **add 12** |
| → → generate(13, 3, 0) | 13 | 3 | 0 | ✅ **add 13** |
| → → … | … | … | … | add 14..19 |
| → generate(2, 2, 1) | 2 | 2 | 1 | try d = 3..9 |
| → → generate(23, 3, 0) | 23 | 3 | 0 | ✅ **add 23** |
| → → … | … | … | … | add 24..29 |
| … | … | … | … | continues until d=8→89 |

**Output (n=2):**
```
[12, 13, 14, 15, 16, 17, 18, 19,
 23, 24, 25, 26, 27, 28, 29,
 34, 35, 36, 37, 38, 39,
 45, 46, 47, 48, 49,
 56, 57, 58, 59,
 67, 68, 69,
 78, 79,
 89]
```
Total = **36 = C(9, 2)** ✅

---

**Input:** `n = 1`

`startDigit = -1` → loop starts at `d = 0`

Output: `[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]` — all 10 single-digit numbers ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(C(9, n))` | Exactly C(9, n) valid numbers generated; each costs O(1) work. Max is C(9, 4) = C(9, 5) = 126 for n=4 or n=5. Since n ≤ 9 and digits are fixed at 1-9, this is effectively **O(1)** |
| **Space** | `O(n)` | Recursion depth is at most `n` (≤ 9). Output list excluded |

### Output Size by n

| n | Count formula | Numbers in output |
|:-:|:-------------:|:-----------------:|
| 1 | C(10,1) = 10  | 0–9               |
| 2 | C(9,2)  = 36  | 12 … 89           |
| 3 | C(9,3)  = 84  | 123 … 789         |
| 4 | C(9,4)  = 126 | 1234 … 6789       |
| 5 | C(9,5)  = 126 | 12345 … 56789     |
| 6 | C(9,6)  = 84  | 123456 … 456789   |
| 9 | C(9,9)  = 1   | 123456789         |

> 📌 For `n=1` the pool is 0–9 (10 choices); for `n≥2` the pool is 1–9 (9 choices).
> Output peaks at `n = 4` and `n = 5` (both 126), then shrinks symmetrically.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Backtracking / Combinatorial Generation |
| **Why No Explicit Sorting** | Iterating digits in order at every level guarantees ascending output |
| **n=1 Special Case** | `prevDigit = -1` seeds the call so `0` is included; for `n ≥ 2`, `prevDigit = 0` excludes leading zeros |
| **Output Count Formula** | `n=1` → 10 numbers (C(10,1)); `n≥2` → C(9,n) numbers |
| **Interview Angle** | Tests understanding of combination generation via backtracking |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Combinations (LeetCode #77) | LeetCode | Choose k numbers from 1..n in order — same pattern |
| Combination Sum (LeetCode #39) | LeetCode | Backtracking with pruning |
| Letter Combinations of Phone Number | LeetCode #17 | Digit-to-char backtracking |
| Subsets (LeetCode #78) | LeetCode | Generate all 2^n subsets — superset of this problem |

---

## Alternative Approaches

### 1. 🐢 Brute Force — O(upper_bound × n)

Iterate all integers from `0` to `(upper_bound for n digits)`, check if each has
strictly increasing digits.

```java
// Brute Force — clean but unnecessarily scans all integers
class Solution {
    public ArrayList<Integer> increasingNumbers(int n) {
        ArrayList<Integer> result = new ArrayList<>();
        int lowerBound = (int) Math.pow(10, n - 1); // e.g., 10 for n=2
        int upperBound = (int) Math.pow(10, n);      // e.g., 100 for n=2

        for (int num = lowerBound; num < upperBound; num++) {
            if (isStrictlyIncreasing(num)) {
                result.add(num);
            }
        }
        return result;
    }

    private boolean isStrictlyIncreasing(int num) {
        int prevDigit = 10; // sentinel: higher than any digit

        while (num > 0) {
            int d = num % 10;
            if (d >= prevDigit) return false;
            prevDigit = d;
            num /= 10;
        }
        return true;
    }
}
```

> ❌ **Verdict:** Accepted for small `n`, but scans up to 10^9 numbers for n=9.
> Wastes time on every invalid number. Not recommended.

---

### 2. ✅ Backtracking (Optimal)

Generate only valid combinations directly. No wasted work.
Described in full above.

> ✅ **Verdict:** Accepted — generates exactly C(9, n) numbers with O(n) stack depth.

---

### Comparison Table

| Approach | Time | Space | Output Order | Notes |
|---|---|---|---|---|
| Brute Force | O(10^n × n) | O(1) | ✅ Sorted | Scans all integers — wasteful |
| **Backtracking** | **O(C(9,n))** | **O(n)** | **✅ Sorted** | **Generates only valid numbers** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#backtracking` `#recursion` `#combinatorics` `#easy` `#digit-dp` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** June 25, 2026
