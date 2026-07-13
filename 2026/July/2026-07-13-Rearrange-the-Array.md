# 🔄 Rearrange the Array

## Problem Link
🔗 [Rearrange the Array – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/rearrange-the-array-1639032648/1)

---

## Difficulty
**Hard** | Accuracy: `36.85%` | Submissions: `5K+` | Points: `8` | Average Time: `25m`

---

## Tags
- Permutation Cycles
- Number Theory
- LCM
- Modular Arithmetic

---

## Problem Summary

Consider an array `a[] = [1, 2, 3, ..., n]` and a permutation `b[]` of size `n`
containing all integers from `1` to `n` exactly once.

- `b[]` defines a rearrangement operation.
- During a single operation, every element at position `i` in `a[]` moves to
  position `b[i]` (1-based indexing).
- We must perform **at least one** operation.

Find the **minimum number of operations** required for all elements to return
to their original positions **simultaneously** (i.e., `a[]` becomes
`[1, 2, ..., n]` again).

**Note:** The answer can be large — return it **modulo 10⁹+7**.

**Constraints:**
- `1 ≤ n ≤ 10⁴`
- `a.size() = b.size() = n`
- `b[]` is a permutation of integers from `1` to `n`

---

## Intuition

### Every Permutation Decomposes into Cycles

A permutation `b[]` can always be broken into disjoint **cycles**. For example,
`b = [2,3,1,5,4]` has cycles `(1→2→3→1)` and `(4→5→4)` — element at position 1
moves to position 2, which moves to position 3, which moves back to position 1;
independently, position 4 and 5 swap with each other.

### Each Cycle Returns to Start After Exactly "Cycle Length" Operations

If a cycle has length `L`, applying the operation `L` times brings every element
in that cycle back to its original position (and fewer than `L` times does not,
unless `L` itself divides that smaller count).

### Why the Answer Is the LCM of All Cycle Lengths

For **all** elements across **all** cycles to be back in place **simultaneously**,
the number of operations must be a multiple of **every** individual cycle's length.
The smallest such number is the **Least Common Multiple (LCM)** of all cycle lengths.

### Verification with Example 2

`b = [2,3,1,5,4]` → cycles: `(1,2,3)` of length `3`, and `(4,5)` of length `2`.

```
LCM(3, 2) = 6
```

Matches the expected output `6` — and the given trace explicitly shows the array
cycling through 6 distinct states before returning to `[1,2,3,4,5]`. ✅

### Verification with Example 1

`b = [1,2,3]` → every position maps to itself, so all three cycles have length `1`.

```
LCM(1, 1, 1) = 1
```

Matches expected output `1` — one operation trivially returns everything (since
nothing moved in the first place). ✅

### Why We Can't Just Multiply Cycle Lengths Together

Naively multiplying all cycle lengths would **overcount** — if two cycles share
a common factor (e.g., lengths `4` and `6`), the true LCM (`12`) is **smaller**
than the raw product (`24`). LCM correctly accounts for shared factors.

### Computing LCM Modulo 10⁹+7 Safely

With `n` up to `10⁴`, the LCM of cycle lengths can be **astronomically large** —
far beyond what fits in a `long`, let alone an `int`. We can't compute the LCM
directly and then take the modulo; we must build it up **modularly** using
**prime factorization**:

1. Factorize each cycle length into its prime factors.
2. For each prime, track the **maximum exponent** seen across all cycle lengths
   (this is exactly how LCM is defined via prime factorization).
3. Multiply `prime^maxExponent` for every prime together, taking the modulo at
   every multiplication step (using fast modular exponentiation).

---

## Approach

1. Traverse `b[]` using a `visited[]` array to identify all disjoint cycles and
   their lengths (standard cycle detection in a permutation).
2. For each cycle length `L`, factorize `L` into primes, and update a map
   `maxExp[prime] = max(maxExp[prime], exponent)`.
3. After processing all cycles, compute the final answer:
   `ans = Π (prime^maxExp[prime]) mod (10⁹+7)`, using modular exponentiation for
   each term and multiplying results modulo `10⁹+7`.
4. Return `ans`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int minOperations(int[] b)`

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    static final int MOD = 1_000_000_007;

    public int minOperations(int[] b) {
        int n = b.length;
        boolean[] visited = new boolean[n + 1];
        Map<Integer, Integer> maxExp = new HashMap<>();

        // Find every cycle in the permutation and its length
        for (int i = 1; i <= n; i++) {
            if (!visited[i]) {
                int len = 0;
                int j = i;
                while (!visited[j]) {
                    visited[j] = true;
                    j = b[j - 1]; // b is 1-indexed values, stored 0-indexed
                    len++;
                }
                factorize(len, maxExp);
            }
        }

        // Reconstruct LCM modulo MOD from tracked prime exponents
        long ans = 1;
        for (Map.Entry<Integer, Integer> entry : maxExp.entrySet()) {
            long prime = entry.getKey();
            int exponent = entry.getValue();
            ans = (ans * modPow(prime, exponent, MOD)) % MOD;
        }

        return (int) ans;
    }

    // Factorize 'num' into primes, updating the running max exponent per prime
    private void factorize(int num, Map<Integer, Integer> maxExp) {
        int d = 2;
        while ((long) d * d <= num) {
            if (num % d == 0) {
                int count = 0;
                while (num % d == 0) {
                    num /= d;
                    count++;
                }
                maxExp.merge(d, count, Math::max);
            }
            d++;
        }
        if (num > 1) {
            maxExp.merge(num, 1, Math::max); // remaining num is itself prime
        }
    }

    // Fast modular exponentiation: base^exp mod mod
    private long modPow(long base, long exp, long mod) {
        long result = 1;
        base %= mod;
        while (exp > 0) {
            if ((exp & 1) == 1) {
                result = (result * base) % mod;
            }
            base = (base * base) % mod;
            exp >>= 1;
        }
        return result;
    }
}
```

---

## Dry Run

### Example 2 — `b = [2,3,1,5,4]` → Expected Output: `6`

**Cycle detection:**

| Start `i` | Traversal | Cycle length |
|:---------:|-----------|:------------:|
| 1 | `1 → b[0]=2 → b[1]=3 → b[2]=1` (back to start) | 3 |
| 4 | `4 → b[3]=5 → b[4]=4` (back to start) | 2 |

**Factorization:**
```
factorize(3): 3 is prime → maxExp[3] = 1
factorize(2): 2 is prime → maxExp[2] = 1
```

**Final computation:**
```
ans = 2^1 × 3^1 mod (10⁹+7) = 2 × 3 = 6
```

**Output:** `6` ✅

---

### Example 1 — `b = [1,2,3]` → Expected Output: `1`

**Cycle detection:** every position maps to itself — three cycles, each length `1`.

**Factorization:** `factorize(1)` — the trial-division loop never executes
(`2×2=4 > 1`), and since `num=1` is not `> 1`, **no prime factors are recorded**.
This correctly represents that a cycle of length `1` places no constraint on
the LCM (since `LCM(x, 1) = x` for any `x`).

`maxExp` remains empty → `ans = 1` (empty product).

**Output:** `1` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n + n√n)` in the worst case | Cycle detection is `O(n)`; factorizing each cycle length costs `O(√L)`, and the total cost across all cycles is bounded similarly |
| **Space** | `O(n)` | `visited[]` array of size `n`, plus a prime-exponent map bounded by the number of distinct primes across all cycle lengths |

For max constraints `n = 10⁴`: `√n ≈ 100`, so factorization work is trivially fast
even in the worst case.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Insight** | Every permutation decomposes into disjoint cycles; the answer is the LCM of all cycle lengths |
| **Why LCM, Not Product** | Multiplying cycle lengths directly overcounts shared factors — LCM correctly merges them |
| **Modular LCM via Prime Factorization** | LCM is built by tracking the max exponent of each prime across all numbers, then recombining via modular exponentiation |
| **Cycle Length 1 Is a No-Op for LCM** | Fixed points (`b[i]=i`) contribute nothing to the LCM computation — correctly handled since factorizing `1` yields no primes |
| **Interview Relevance** | Classic "permutation cycle decomposition" pattern, combined with modular LCM — a strong combination of two separate CS fundamentals |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Cycle Length in a Permutation | GFG | The core cycle-detection subroutine used here |
| Minimum Number of Operations to Sort Array | GFG/LC | Related permutation-cycle reasoning, different goal |
| Array Nesting | LeetCode #565 | Finding the longest cycle in a functional graph/permutation |
| GCD and LCM computations | Various | Modular LCM via prime factorization is a broadly reusable number-theory tool |

---

## Alternative Approaches

### 1. 🐢 Naive LCM via BigInteger — Correct but Heavier

Use Java's `BigInteger` to compute the exact LCM without modular reduction, then
take modulo only at the very end.

```java
import java.math.BigInteger;

class Solution {
    public int minOperations(int[] b) {
        int n = b.length;
        boolean[] visited = new boolean[n + 1];
        BigInteger lcm = BigInteger.ONE;

        for (int i = 1; i <= n; i++) {
            if (!visited[i]) {
                int len = 0, j = i;
                while (!visited[j]) {
                    visited[j] = true;
                    j = b[j - 1];
                    len++;
                }
                BigInteger cycleLen = BigInteger.valueOf(len);
                BigInteger gcd = lcm.gcd(cycleLen);
                lcm = lcm.divide(gcd).multiply(cycleLen);
            }
        }

        BigInteger mod = BigInteger.valueOf(1_000_000_007L);
        return lcm.mod(mod).intValue();
    }
}
```

> ✅ **Verdict:** Correct, but `BigInteger` arithmetic is significantly slower than
> primitive `long` operations — the prime-factorization approach is more efficient
> and idiomatic for competitive programming.

---

### 2. ✅ Prime Factorization + Modular Exponentiation (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — efficient, avoids `BigInteger` overhead entirely, and
> scales cleanly for `n` up to `10⁴`.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| BigInteger LCM | O(n√n) with heavier constants | O(n) | ✅ Correct but slower due to BigInteger overhead |
| **Prime Factorization + ModPow** | **O(n√n)** | **O(n)** | ✅ **Cleaner, faster — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#permutation-cycles` `#lcm` `#number-theory` `#hard` `#modular-arithmetic` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 13, 2026
