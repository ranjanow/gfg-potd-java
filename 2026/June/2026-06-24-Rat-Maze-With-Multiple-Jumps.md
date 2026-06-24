# 🐀 Rat Maze With Multiple Jumps

## Problem Link
🔗 [Rat Maze With Multiple Jumps – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/rat-maze-with-multiple-jumps3852/1)

---

## Difficulty
**Medium** | Topics: `Dynamic Programming` `Backtracking` `Matrix`

---

## Tags
- Matrix
- Dynamic Programming
- Backtracking
- Path Finding
- Greedy

---

## Problem Summary

Given an `n × n` matrix `mat[][]`, where `mat[i][j]` is the **maximum number of steps**
the rat can jump either **right** or **down** from cell `(i, j)`.

- A value of `0` means the cell is **blocked** (dead end).
- A non-zero value means the rat can jump **1 to mat[i][j]** steps in either direction.

Find a path from the **top-left `(0, 0)`** to the **bottom-right `(n-1, n-1)`** and return
a same-sized matrix with `1` marking path cells and `0` elsewhere.

**Rules when multiple paths exist:**
1. Choose the path with the **fewest total jumps** (shortest hops).
2. For the **same hop distance** at a cell, prefer **right (forward)** over **down**.

Return `{{-1}}` if no path exists.

---

## Intuition

### Key Observations

1. **Only right and down moves** — no cycles possible. The rat always moves closer to
   the destination, so backtracking is safe (no infinite loops).

2. **Naive backtracking** explores every possible jump combination from every cell.
   For large `n` and high cell values this becomes exponential — too slow.

3. **The smart question is:** *"From cell `(i, j)`, can destination `(n-1, n-1)` be reached?"*
   Answer this **once per cell**, store it, and reuse.
   → This is the classic **DP (Bottom-Up Reachability)** pattern.

4. **CRF Matrix (Can Reach From):**
   - `CRF[i][j] = true` → destination is reachable from `(i, j)`.
   - `CRF[n-1][n-1] = true` by definition.
   - Fill from **bottom-right to top-left** — so every cell a jump can reach is already computed.

5. **Two-phase algorithm:**
   - **Phase 1:** Fill CRF. O(n² × k) where k = max cell value.
   - **Phase 2:** Greedily trace path using CRF. O(n × k).

### Why Bottom-Up Order Works

For cell `(i, j)` the rat can jump right to `(i, j+a)` or down `(i+a, j)`.
Processing **right-to-left within bottom-to-top** rows ensures both `(i, j+a)` (same row, higher j)
and `(i+a, j)` (lower row) are always computed **before** `(i, j)`.

---

## Approach

### Phase 1 — Build CRF (Reachability DP)

1. Create `boolean[][] crf` of size `n × n`, all `false`.
2. Set `crf[n-1][n-1] = true`.
3. Iterate `i` from `n-1` down to `0`; for each row, `j` from `n-1` down to `0`.
4. Skip if `(i, j)` is destination or blocked (`mat[i][j] == 0`).
5. For each jump `a` from `1` to `mat[i][j]`:
   - If `crf[i][j+a]` (right) OR `crf[i+a][j]` (down) is true → `crf[i][j] = true`, break.
6. If `crf[0][0]` is false → return `{{-1}}`.

### Phase 2 — Trace the Path

1. Start at `(r=0, c=0)`. Create `int[][] sol` all zeros.
2. While not at destination:
   - Mark `sol[r][c] = 1`.
   - For `a` from `1` to `mat[r][c]`:
     - If right `(r, c+a)` is in bounds and `crf[r][c+a]` → move right, break.
     - Else if down `(r+a, c)` is in bounds and `crf[r+a][c]` → move down, break.
3. Mark `sol[n-1][n-1] = 1`. Return `sol`.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `ArrayList<ArrayList<Integer>> shortestDist(int[][] mat)`

```java
import java.util.*;

class Solution {
    public ArrayList<ArrayList<Integer>> shortestDist(int[][] mat) {
        int n = mat.length;

        // Phase 1: Build CRF (Can Reach From) reachability matrix bottom-up
        boolean[][] crf = new boolean[n][n];
        crf[n - 1][n - 1] = true;  // destination can always reach itself

        for (int i = n - 1; i >= 0; i--) {
            for (int j = n - 1; j >= 0; j--) {
                if (i == n - 1 && j == n - 1) continue; // already initialized
                if (mat[i][j] == 0) continue;            // blocked cell, skip

                for (int a = 1; a <= mat[i][j] && !crf[i][j]; a++) {
                    if (j + a < n && crf[i][j + a]) crf[i][j] = true; // jump right
                    if (i + a < n && crf[i + a][j]) crf[i][j] = true; // jump down
                }
            }
        }

        // If source cannot reach destination — return {{-1}}
        if (!crf[0][0]) {
            ArrayList<ArrayList<Integer>> noPath = new ArrayList<>();
            ArrayList<Integer> row = new ArrayList<>();
            row.add(-1);
            noPath.add(row);
            return noPath;
        }

        // Phase 2: Trace the optimal path greedily using CRF
        int[][] sol = new int[n][n];
        int r = 0, c = 0;

        while (r != n - 1 || c != n - 1) {
            sol[r][c] = 1;
            boolean moved = false;

            for (int a = 1; a <= mat[r][c] && !moved; a++) {
                // Prefer right (forward) over down for the same hop distance
                if (c + a < n && crf[r][c + a]) {
                    c += a;
                    moved = true;
                } else if (r + a < n && crf[r + a][c]) {
                    r += a;
                    moved = true;
                }
            }
        }
        sol[n - 1][n - 1] = 1;

        // Convert int[][] → ArrayList<ArrayList<Integer>> for GFG driver
        ArrayList<ArrayList<Integer>> result = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            ArrayList<Integer> row = new ArrayList<>();
            for (int j = 0; j < n; j++) {
                row.add(sol[i][j]);
            }
            result.add(row);
        }
        return result;
    }
}
```

---

## Dry Run

**Input:**
```
mat[][] = { {2, 1, 0, 0},
            {3, 0, 0, 1},
            {0, 1, 0, 1},
            {0, 0, 0, 1} }
```

### Phase 1 — CRF Construction (key cells shown)

| Cell | mat value | Checks | CRF result |
|:----:|:---------:|--------|:----------:|
| (3,3) | 1 | Destination — pre-set | `true` |
| (2,3) | 1 | down a=1 → (3,3) = true | `true` |
| (1,3) | 1 | down a=1 → (2,3) = true | `true` |
| (2,1) | 1 | right→(2,2)=0, down→(3,1)=0 | `false` |
| (1,0) | 3 | a=1: right→(1,1)=0, down→(2,0)=0 `·` a=2: right→(1,2)=0, down→(3,0)=0 `·` a=3: right→(1,3)=true ✅ | `true` |
| (0,1) | 1 | right→(0,2)=0, down→(1,1)=0 | `false` |
| (0,0) | 2 | a=1: right→(0,1)=false, down→(1,0)=true ✅ | `true` |

**CRF matrix:**
```
T  F  F  F
T  F  F  T
F  F  F  T
F  F  F  T
```
`T = true (reachable)`, `F = false (not reachable)`

### Phase 2 — Path Tracing

| Step | (r,c) | mat[r][c] | Action | Next |
|:----:|:-----:|:---------:|--------|:----:|
| 1 | (0,0) | 2 | a=1: right→crf[0][1]=F; down→crf[1][0]=T ✅ | (1,0) |
| 2 | (1,0) | 3 | a=1: right→F; down→F `·` a=2: right→F; down→F `·` a=3: right→crf[1][3]=T ✅ | (1,3) |
| 3 | (1,3) | 1 | a=1: right out of bounds; down→crf[2][3]=T ✅ | (2,3) |
| 4 | (2,3) | 1 | a=1: right out of bounds; down→crf[3][3]=T ✅ | (3,3) |
| 5 | (3,3) | — | Destination reached | — |

**Output:**
```
1  0  0  0
1  0  0  1
0  0  0  1
0  0  0  1
```
✅ Matches expected output.

---

## Complexity Analysis

### Optimal Solution (DP)

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n² × k)` | For each of the n² cells, check up to k=max(mat[i][j]) jumps in Phase 1; Phase 2 traces at most n steps each checking k jumps |
| **Space** | `O(n²)` | CRF matrix + solution matrix, both n×n |

### Brute Force (Backtracking)

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(k^(n²))` | At each cell up to k branching choices; total path length up to n² in worst case |
| **Space** | `O(n²)` | Recursion stack depth + solution matrix |

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Technique** | Bottom-Up DP (Reachability) |
| **CRF Pattern** | Pre-compute "can destination be reached from here" to avoid repeated work |
| **Processing Order** | Must fill right-to-left within bottom-to-top so all jump targets are already computed |
| **Tie-breaking** | Prefer right over down at same jump size `a`; try smaller `a` first for fewest jumps |
| **No-cycle Guarantee** | Only right/down moves → no cell is revisited → backtracking is always safe |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Rat in a Maze | GFG / LeetCode | Simpler version (jumps of 1 only, all 4 directions) |
| Jump Game II | LeetCode #45 | Reachability with variable jumps (1D version) |
| Unique Paths | LeetCode #62 | Count paths in grid, right/down only |
| Minimum Cost Path | GFG | DP path-finding in a matrix |

---

## Alternative Approaches

### 1. 🐢 Backtracking (Brute Force)

Recursively try all valid jumps from `(0, 0)`. Mark cell, recurse, backtrack on failure.

```java
import java.util.*;

class Solution {
    private int n;

    public ArrayList<ArrayList<Integer>> shortestDist(int[][] mat) {
        n = mat.length;
        int[][] sol = new int[n][n];

        if (!solve(mat, sol, 0, 0)) {
            // No path — return {{-1}}
            ArrayList<ArrayList<Integer>> noPath = new ArrayList<>();
            ArrayList<Integer> row = new ArrayList<>();
            row.add(-1);
            noPath.add(row);
            return noPath;
        }

        // Convert int[][] → ArrayList<ArrayList<Integer>>
        ArrayList<ArrayList<Integer>> result = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            ArrayList<Integer> row = new ArrayList<>();
            for (int j = 0; j < n; j++) row.add(sol[i][j]);
            result.add(row);
        }
        return result;
    }

    private boolean solve(int[][] mat, int[][] sol, int r, int c) {
        // Base: reached destination
        if (r == n - 1 && c == n - 1) {
            sol[r][c] = 1;
            return true;
        }

        if (mat[r][c] == 0) return false; // blocked cell

        sol[r][c] = 1;

        // Try all jump sizes; right first, then down (same hop preference)
        for (int a = 1; a <= mat[r][c]; a++) {
            if (c + a < n && solve(mat, sol, r, c + a)) return true; // right
            if (r + a < n && solve(mat, sol, r + a, c)) return true; // down
        }

        sol[r][c] = 0; // backtrack
        return false;
    }
}
```

> ⚠️ **Verdict:** Accepted for small `n` but exponential in worst case — TLE for large inputs.

---

### 2. ✅ DP — CRF Matrix (Optimal)

Bottom-up reachability + greedy path trace. Described in full above.

> ✅ **Verdict:** Accepted — `O(n² × k)` time, `O(n²)` space. Correct and efficient.

---

### Comparison Table

| Approach | Time | Space | Correctness |
|---|---|---|---|
| Backtracking | O(k^(n²)) | O(n²) | ✅ Correct but slow |
| **DP — CRF Matrix** | **O(n² × k)** | **O(n²)** | ✅ **Optimal — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#dynamic-programming` `#backtracking` `#matrix` `#path-finding` `#medium` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** June 24, 2026
