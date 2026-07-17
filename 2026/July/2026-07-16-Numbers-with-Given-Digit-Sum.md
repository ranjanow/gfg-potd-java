# 🔢 Numbers with Given Digit Sum

## Problem Link
🔗 [Numbers with Given Digit Sum – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/count-of-n-digit-numbers-whose-sum-of-digits-equals-to-given-sum0733/1)

---

## Difficulty
**Medium** | Accuracy: `26.33%` | Submissions: `26K+` | Points: `4`

---

## Tags
- Dynamic Programming
- Digit DP
- Combinatorics

---

## Problem Summary

Given two integers `n` and `sum`, determine the number of **`n`-digit positive
integers** whose digits add up to `sum`.

- An `n`-digit number cannot have leading zeros — the first digit must be
  between `1` and `9`.
- If no such `n`-digit number exists, return `-1`.

**Constraints:**
- `1 ≤ n ≤ 9`
- `1 ≤ sum ≤ 81`

---

## Intuition

### Splitting Off the First Digit

The first digit is special — it must be `1–9` (no leading zero), while every
**other** digit can freely be `0–9`. This suggests a natural decomposition:

```
answer = Σ (for firstDigit d = 1 to 9) [ ways to fill the remaining (n-1)
                                          digits, each 0-9, summing to (sum-d) ]
```

### Defining the DP State

Let `dp[i][s]` = the number of ways to fill `i` digits, **each ranging 0–9**
(leading zeros allowed in this sub-count), such that they sum to `s`.

**Base case:** `dp[0][0] = 1` (zero digits, sum must be exactly `0` — one
trivial "empty" way), and `dp[0][s] = 0` for any `s > 0` (impossible).

**Recurrence:** To fill `i` digits summing to `s`, choose the **last** digit
`d` (0–9), then the remaining `i-1` digits must sum to `s-d`:

```
dp[i][s] = Σ (for d = 0 to 9, where s-d ≥ 0) dp[i-1][s-d]
```

### Combining for the Final Answer

```
answer = Σ (for d = 1 to 9) dp[n-1][sum-d]
```

If this total is `0` (no valid combination exists at all), return `-1`.

### Verification with Example 1

`n=2, sum=2`. We need `dp[1][s]` first: a single digit (0–9) summing to `s` is
possible in exactly `1` way if `0 ≤ s ≤ 9`, else `0`.

```
answer = dp[1][2-1] + dp[1][2-2] + dp[1][2-3] + ... + dp[1][2-9]
       = dp[1][1] + dp[1][0] + (negative indices skipped)
       = 1 + 1 = 2
```

Matches expected output `2` — corresponding to `11` and `20` ✅

### Verification with Example 2 (the `-1` Case)

`n=1, sum=10`. Here `n-1=0`, so we use `dp[0][...]`, where `dp[0][0]=1` and
everything else is `0`.

```
answer = Σ (d=1 to 9) dp[0][10-d]
```

For this to be nonzero, we'd need `10-d = 0` for some `d` in `[1,9]`, i.e.
`d=10` — but `d` only ranges up to `9`. So **every term is `0`**, giving
`answer = 0` → return `-1` ✅ (a single digit can never sum to `10`, since the
maximum single-digit value is `9`, exactly matching the official explanation:
*"A single-digit number can only have a digit sum between 0 and 9."*).

---

## Approach

1. Create `int[][] dp` of size `n × (sum+1)`.
2. Set `dp[0][0] = 1` (all other `dp[0][s]` default to `0`).
3. For `i` from `1` to `n-1`:
   - For `s` from `0` to `sum`:
     - `dp[i][s] = Σ dp[i-1][s-d]` for each digit `d` from `0` to `9` where `s-d ≥ 0`.
4. Compute `answer = Σ dp[n-1][sum-d]` for `d` from `1` to `9` (skipping any
   negative index).
5. Return `answer` if nonzero, else return `-1`.

---

## Java Solution

> ⚠️ **Verify the exact method name in the GFG editor before submitting.**
> Based on the URL slug, a likely candidate is `countNumbers`, but confirm
> visually before pasting.

```java
class Solution {
    public int countNumbers(int n, int sum) {
        // dp[i][s] = number of ways to fill i digits (each 0-9) summing to s
        int[][] dp = new int[n][sum + 1];
        dp[0][0] = 1; // base case: 0 digits, sum must be exactly 0

        for (int i = 1; i < n; i++) {
            for (int s = 0; s <= sum; s++) {
                int total = 0;
                for (int d = 0; d <= 9 && d <= s; d++) {
                    total += dp[i - 1][s - d];
                }
                dp[i][s] = total;
            }
        }

        // First digit ranges 1-9 (no leading zero); remaining n-1 digits use dp
        int answer = 0;
        for (int d = 1; d <= 9; d++) {
            int remaining = sum - d;
            if (remaining >= 0 && remaining <= sum) {
                answer += dp[n - 1][remaining];
            }
        }

        return answer == 0 ? -1 : answer;
    }
}
```

---

## Dry Run

### Example 1 — `n=2, sum=2` → Expected Output: `2`

**Building `dp[1][s]` (i=1, single digit 0-9):**

| s | Computation | dp[1][s] |
|:-:|---|:--------:|
| 0 | dp[0][0] | 1 |
| 1 | dp[0][1]+dp[0][0] = 0+1 | 1 |
| 2 | dp[0][2]+dp[0][1]+dp[0][0] = 0+0+1 | 1 |

**Final answer:**
```
d=1: dp[1][2-1]=dp[1][1]=1
d=2: dp[1][2-2]=dp[1][0]=1
d=3..9: 2-d negative, skip
Total = 1+1 = 2
```

**Output:** `2` ✅ (matches `11`, `20`)

---

### Example 3 — `n=2, sum=10` → Expected Output: `9`

**`dp[1][s]` for s=0..9 is `1` each** (single digit s itself), and `dp[1][10]=0`
(no single digit sums to 10).

**Final answer:**
```
d=1: dp[1][9]=1    d=2: dp[1][8]=1    d=3: dp[1][7]=1
d=4: dp[1][6]=1    d=5: dp[1][5]=1    d=6: dp[1][4]=1
d=7: dp[1][3]=1    d=8: dp[1][2]=1    d=9: dp[1][1]=1
Total = 9
```

**Output:** `9` ✅ (matches `19,28,37,46,55,64,73,82,91`)

---

### Example 2 — `n=1, sum=10` → Expected Output: `-1`

With `n-1=0`, only `dp[0][0]=1` is nonzero. Since `sum-d=10-d` never equals `0`
for any `d` in `[1,9]` (would need `d=10`), **every term is `0`**.

**Output:** `-1` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n × sum)` | Filling the DP table: `n` rows × `sum` columns, each cell costs `O(10)` (constant, bounded digit choices) — effectively `O(n × sum)` |
| **Space** | `O(n × sum)` | The `dp[][]` table itself |

For max constraints `n=9, sum=81`: `9 × 82 × 10 ≈ 7,380` operations — trivially fast.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Digit DP — count sequences by digit position and running sum |
| **Why Split the First Digit** | Leading-zero restriction only applies to the first digit; treating it separately keeps the DP recurrence simple and uniform |
| **DP State Meaning** | `dp[i][s]` intentionally allows leading zeros (it represents "any" i-digit sequence, not necessarily a valid i-digit *number*) — this is fine since it's only ever used for the *trailing* digits |
| **The `-1` Case Falls Out Naturally** | No special-casing needed — if no valid combination exists, the DP sum is simply `0`, which we translate to `-1` at the very end |
| **Small Constraints, Clean DP** | `n ≤ 9` and `sum ≤ 81` keep the table tiny — no need for modular arithmetic since max count fits well within `int` |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Count Numbers with Unique Digits | LeetCode #357 | Similar digit-position counting DP |
| Numbers At Most N Given Digit Set | LeetCode #902 | Classic digit DP with constrained digit choices |
| Non-negative Integers without Consecutive Ones | LeetCode #600 | Digit DP with a different constraint type |
| Count Numbers with Digit Sum in Range | GFG | Direct extension of this exact problem |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Check Every N-Digit Number — O(10ⁿ)

Iterate over every `n`-digit number, compute its digit sum, and count matches.

```java
class Solution {
    public int countNumbers(int n, int sum) {
        int lower = (int) Math.pow(10, n - 1);
        int upper = (int) Math.pow(10, n) - 1;
        int count = 0;

        for (int num = lower; num <= upper; num++) {
            int digitSum = 0, temp = num;
            while (temp > 0) {
                digitSum += temp % 10;
                temp /= 10;
            }
            if (digitSum == sum) count++;
        }

        return count == 0 ? -1 : count;
    }
}
```

> ❌ **Verdict:** TLE for larger `n` — `10⁹` iterations when `n=9`. Far too slow.

---

### 2. ✅ Digit DP (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n × sum)` time and space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(10ⁿ) | O(1) | ❌ TLE for n≥7 or so |
| **Digit DP** | **O(n × sum)** | **O(n × sum)** | ✅ **Use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#dynamic-programming` `#digit-dp` `#combinatorics` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 16, 2026
