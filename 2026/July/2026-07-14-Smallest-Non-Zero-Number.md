# 🔢 Smallest Non-Zero Number

## Problem Link
🔗 [Smallest Non-Zero Number – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/find-smallest-non-zero-number4510/1)

---

## Difficulty
**Medium** | Accuracy: `48.41%` | Submissions: `15K+` | Points: `4`

---

## Tags
- Arrays
- Data Structures
- Greedy
- Simulation

---

## Problem Summary

Given an array `arr[]`, find the smallest number `x` such that when `x` is
processed sequentially with each element of the array (index `0` to `n-1`), it
**never becomes negative**, under the following rules:

1. If `x` is **greater than** the current array element, `x` is **increased**
   by the difference between `x` and the array element.
2. If `x` is **less than or equal to** the current array element, `x` is
   **decreased** by the difference between the array element and `x`.

**Constraints:**
- `1 ≤ arr.size() ≤ 10⁶`
- `1 ≤ arr[i] ≤ 10⁴`
- Expected Time: `O(n)` · Auxiliary Space: `O(1)`

---

## Intuition

### The Hidden Simplification — Both Rules Are the Same Formula!

Let's algebraically expand both cases:

**Case 1 (`x > arr[i]`):** `x_new = x + (x - arr[i]) = 2x - arr[i]`

**Case 2 (`x ≤ arr[i]`):** `x_new = x - (arr[i] - x) = 2x - arr[i]`

**Both cases produce the exact same formula!** The comparison in the problem
statement is a red herring — regardless of which branch applies, the update is
always:

```
x_new = 2×x - arr[i]
```

### Verification with Example 1

`arr = [3,4,3,2,4]`, starting `x=4`:
```
x=4  → 2×4-3=5
x=5  → 2×5-4=6
x=6  → 2×6-3=9
x=9  → 2×9-2=16
x=16 → 2×16-4=28
```
Matches the official trace exactly (`5, 6, 9, 16, 28`) ✅

### Why Forward Simulation Alone Doesn't Find the Minimum

Given a starting `x₀`, the entire sequence is **deterministic** — there's no
freedom after choosing `x₀`. So we can't binary search or greedily adjust
mid-sequence; we need the **smallest `x₀`** such that *every* intermediate value
along this deterministic chain stays non-negative.

### The Key Insight — Work Backward!

Instead of guessing `x₀` and simulating forward (which would need `O(n)` per
guess, and we don't even know a good search range), reverse the relation:

```
x_new = 2×x_old - arr[i]   ⟺   x_old = (x_new + arr[i]) / 2
```

Starting from the **end** of the array with the loosest possible constraint
(`x_n ≥ 0`, so treat the required minimum at the end as `0`), we can propagate
**backward**, computing at each step the **minimum value required** to guarantee
all future steps stay non-negative:

```
required[i] = ceil( (required[i+1] + arr[i]) / 2 )
```

After processing all elements from the end back to the start, `required[0]` is
exactly the smallest valid `x₀`.

### Why the Ceiling Matters

Since `x₀` must be an **integer** (and the whole forward chain stays integer as
long as `x₀` is), whenever `(required[i+1] + arr[i])` is odd, we must **round up**
— using a value one less would make some future intermediate result fractional
or, when floored, actually negative. The ceiling guarantees feasibility.

### Verification with Example 1 (Backward Pass)

`arr = [3,4,3,2,4]`, process from `i=4` down to `i=0`, `required = 0` initially:

```
i=4 (arr=4): required = ceil((0+4)/2)  = 2
i=3 (arr=2): required = ceil((2+2)/2)  = 2
i=2 (arr=3): required = ceil((2+3)/2)  = 3
i=1 (arr=4): required = ceil((3+4)/2)  = 4
i=0 (arr=3): required = ceil((4+3)/2)  = 4
```

Final `required = 4` → matches expected output `4` exactly ✅

### Verification with Example 2

`arr = [4,4]`:
```
i=1 (arr=4): required = ceil((0+4)/2) = 2
i=0 (arr=4): required = ceil((2+4)/2) = 3
```

Final `required = 3` → matches expected output `3` exactly ✅

---

## Approach

1. Initialise `required = 0`.
2. Traverse `arr[]` from the **last** index down to the **first**:
   - `required = ceil((required + arr[i]) / 2)`, computed via integer arithmetic
     as `(required + arr[i] + 1) / 2`.
3. Return `required` (this is the minimum valid starting `x`).

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int find(int[] arr)`

```java
class Solution {
    public int find(int[] arr) {
        int n = arr.length;
        long required = 0; // minimum value needed at the current position

        // Process backward: propagate the minimum requirement toward the start
        for (int i = n - 1; i >= 0; i--) {
            // Ceiling division by 2: ceil(a/2) = (a+1)/2 for non-negative a
            required = (required + arr[i] + 1) / 2;
        }

        return (int) required;
    }
}
```

---

## Dry Run

### Example 1 — `arr = [3,4,3,2,4]` → Expected Output: `4`

| i | arr[i] | required before | Computation | required after |
|:-:|:------:|:----------------:|-------------|:---------------:|
| 4 | 4 | 0 | (0+4+1)/2 = 2 | 2 |
| 3 | 2 | 2 | (2+2+1)/2 = 2 | 2 |
| 2 | 3 | 2 | (2+3+1)/2 = 3 | 3 |
| 1 | 4 | 3 | (3+4+1)/2 = 4 | 4 |
| 0 | 3 | 4 | (4+3+1)/2 = 4 | 4 |

**Output:** `4` ✅

---

### Example 2 — `arr = [4,4]` → Expected Output: `3`

| i | arr[i] | required before | Computation | required after |
|:-:|:------:|:----------------:|-------------|:---------------:|
| 1 | 4 | 0 | (0+4+1)/2 = 2 | 2 |
| 0 | 4 | 2 | (2+4+1)/2 = 3 | 3 |

**Output:** `3` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n)` | Single backward pass through the array; each step is O(1) |
| **Space** | `O(1)` | Only a single running variable `required` |

For max constraints `n = 10⁶`: trivially fast. Note that `required` stays bounded
by roughly `max(arr) ≤ 10⁴` throughout — no overflow risk even with plain `int`,
though `long` is used defensively.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Insight #1** | Both branches of the problem's if/else collapse into the identical formula `2x - arr[i]` — always verify whether "different cases" are actually different! |
| **Core Insight #2** | Forward simulation can't find the *minimum* valid start — reversing the relation and propagating the *requirement* backward is the key technique |
| **Ceiling Division Trick** | `(a + b + 1) / 2` computes `ceil((a+b)/2)` using pure integer arithmetic — no floating point needed |
| **Why Values Stay Bounded** | The backward recurrence averages the current requirement with `arr[i]`, naturally keeping `required` within the range of the array's values |
| **Interview Relevance** | "Reverse the recurrence to compute a minimum feasible starting value" is a reusable pattern across many simulation-style problems |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Gas Station | LeetCode #134 | Different mechanism, same "find minimum feasible start" theme |
| Minimum Initial Energy to Survive | Various | Nearly identical backward-propagation-of-requirement pattern |
| Candy | LeetCode #135 | Greedy with forward/backward passes to satisfy constraints |
| Jump Game | LeetCode #55 | Different problem, but backward-feasibility-propagation is a shared idea |

---

## Alternative Approaches

### 1. 🐢 Brute Force — Try Every Starting Value — O(n × maxAnswer)

Try `x = 1, 2, 3, ...` in increasing order, fully simulating forward each time
until we find the smallest `x` that never goes negative.

```java
class Solution {
    public int find(int[] arr) {
        int n = arr.length;
        for (long x = 1; ; x++) {
            long cur = x;
            boolean valid = true;
            for (int i = 0; i < n; i++) {
                cur = 2 * cur - arr[i];
                if (cur < 0) { valid = false; break; }
            }
            if (valid) return (int) x;
        }
    }
}
```

> ❌ **Verdict:** TLE — the correct answer can require testing many candidate
> values, each costing `O(n)`, giving `O(n × answer)` in the worst case. Too slow
> for `n` up to `10⁶`.

---

### 2. ✅ Backward Requirement Propagation (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n)` time, `O(1)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force (try each x) | O(n × answer) | O(1) | ❌ TLE — no efficient search bound |
| **Backward Requirement Propagation** | **O(n)** | **O(1)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#arrays` `#greedy` `#simulation` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 14, 2026
