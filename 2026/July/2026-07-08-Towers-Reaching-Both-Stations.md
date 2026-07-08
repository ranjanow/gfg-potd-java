# 📡 Towers Reaching Both Stations

## Problem Link
🔗 [Towers Reaching Both Stations – GeeksforGeeks POTD](https://www.geeksforgeeks.org/problems/geeks-island--170646/1)

---

## Difficulty
**Medium** | Accuracy: `60.27%` | Submissions: `18K+` | Points: `4`

---

## Tags
- DFS / BFS
- Matrix
- Graph Traversal
- Multi-Source BFS

---

## Problem Summary

Given a matrix `mat[][]` of size `n × m`, where `mat[i][j]` is the signal strength
of a communication tower. Two control stations monitor the network:

- **Station P** covers the **top and left** boundaries of the grid.
- **Station Q** covers the **bottom and right** boundaries of the grid.

A signal can propagate from a tower to a **neighbouring** tower (N/S/E/W) **only if**
the neighbouring tower has signal strength **less than or equal to** the current tower's.

Determine the **number of towers `(x, y)`** from which a signal can eventually reach
**both** Station P and Station Q. Any tower located on a boundary covered by a
station can transmit directly to that station.

**Constraints:**
- `1 ≤ n, m ≤ 10³`
- `1 ≤ mat[i][j] ≤ 10³`
- Expected Time: `O(n × m)` · Auxiliary Space: `O(n × m)`

---

## Intuition

### This Is "Pacific Atlantic Water Flow" in Disguise

This problem is structurally identical to the classic **Pacific Atlantic Water Flow**
pattern: two boundary sets, a monotonic propagation condition, and a request for
cells that can reach **both** sets.

### Why Forward Simulation From Every Cell Is Too Slow

Checking, for each of the `n×m` cells, whether it can reach P and whether it can
reach Q via a full DFS/BFS from that cell would cost `O((n×m)²)` in the worst case —
far too slow for `n, m` up to `10³` (`10¹²` operations).

### The Reversal Trick — Multi-Source BFS From the Boundaries

Instead of asking *"can this cell reach the boundary?"*, we flip the question:
*"starting from the boundary, which cells can be reached by walking **backwards**
along valid propagation edges?"*

**Key insight:** A forward edge `A → B` is valid when `strength(B) ≤ strength(A)`.
Walking this edge **backwards** (from B to A) is valid precisely when
`strength(A) ≥ strength(B)` — i.e., **moving to a neighbor with a value greater
than or equal to the current cell's value.**

So: start a **multi-source BFS** from every P-boundary cell (top row + left column),
and expand to a neighbor whenever `strength(neighbor) ≥ strength(current)`. Every
cell reached this way can, in the original forward direction, reach Station P.
Repeat independently for Station Q (bottom row + right column).

### Final Answer

```
answer = count of cells (x, y) where reachP[x][y] == true AND reachQ[x][y] == true
```

### Verification with Example 1

For the given `5×5` grid, running this multi-source BFS twice yields:

**reachP grid** (`T` = can reach Station P):
```
T T T T T
T T T T T
T T T . .
T T . . .
T . . . .
```

**reachQ grid** (`T` = can reach Station Q):
```
. . . . T
. . . T T
. . T T T
T T T T T
T T T T T
```

**Intersection** (both `T`): `(0,4), (1,3), (1,4), (2,2), (3,0), (3,1), (4,0)`
→ **7 cells** ✅ — matches the expected output and the exact cells listed in the
official explanation.

---

## Approach

1. Create `boolean[][] reachP` and `boolean[][] reachQ`, both sized `n × m`.
2. Seed `reachP` with every cell in row `0` and column `0` (mark `true`, enqueue).
3. Seed `reachQ` with every cell in row `n-1` and column `m-1` (mark `true`, enqueue).
4. Run a standard multi-source BFS for each: from any queued cell, expand to an
   unvisited neighbor if `mat[neighbor] ≥ mat[current]`.
5. Count cells where both `reachP[i][j]` and `reachQ[i][j]` are `true`.
6. Return the count.

---

## Java Solution

> ✅ **Confirmed GFG method signature:** `int countCoordinates(int[][] mat)`

```java
import java.util.LinkedList;
import java.util.Queue;

class Solution {
    private int n, m;
    private final int[] dx = {-1, 1, 0, 0};
    private final int[] dy = {0, 0, -1, 1};

    public int countCoordinates(int[][] mat) {
        n = mat.length;
        m = mat[0].length;

        boolean[][] reachP = new boolean[n][m];
        boolean[][] reachQ = new boolean[n][m];

        Queue<int[]> queueP = new LinkedList<>();
        Queue<int[]> queueQ = new LinkedList<>();

        // Seed P: top row + left column. Seed Q: bottom row + right column.
        for (int j = 0; j < m; j++) {
            if (!reachP[0][j]) { reachP[0][j] = true; queueP.add(new int[]{0, j}); }
            if (!reachQ[n - 1][j]) { reachQ[n - 1][j] = true; queueQ.add(new int[]{n - 1, j}); }
        }
        for (int i = 0; i < n; i++) {
            if (!reachP[i][0]) { reachP[i][0] = true; queueP.add(new int[]{i, 0}); }
            if (!reachQ[i][m - 1]) { reachQ[i][m - 1] = true; queueQ.add(new int[]{i, m - 1}); }
        }

        bfs(mat, reachP, queueP);
        bfs(mat, reachQ, queueQ);

        int count = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (reachP[i][j] && reachQ[i][j]) count++;
            }
        }

        return count;
    }

    // Multi-source BFS: expand to a neighbor if its value >= current cell's value
    // (this is the REVERSED direction of the original propagation rule)
    private void bfs(int[][] mat, boolean[][] reach, Queue<int[]> queue) {
        while (!queue.isEmpty()) {
            int[] cur = queue.poll();
            int x = cur[0], y = cur[1];

            for (int d = 0; d < 4; d++) {
                int nx = x + dx[d], ny = y + dy[d];
                if (nx >= 0 && nx < n && ny >= 0 && ny < m
                        && !reach[nx][ny] && mat[nx][ny] >= mat[x][y]) {
                    reach[nx][ny] = true;
                    queue.add(new int[]{nx, ny});
                }
            }
        }
    }
}
```

---

## Dry Run

### Example 2 — `mat = [[2,2],[2,2]]` → Expected Output: `4`

All values equal `2`. Every propagation condition `strength(neighbor) ≥ current`
is trivially satisfied everywhere.

- `reachP` seeds: `(0,0), (0,1), (1,0)` → BFS spreads to `(1,1)` too → all 4 cells `true`.
- `reachQ` seeds: `(1,0), (1,1), (0,1)` → BFS spreads to `(0,0)` too → all 4 cells `true`.

**Intersection:** all 4 cells → **Output: `4`** ✅

---

### Example 1 — Full 5×5 Grid → Expected Output: `7`

```
1 2 2 3 5
3 2 3 4 4
2 4 5 3 1
6 7 1 4 5
5 1 1 2 4
```

**reachP BFS trace (key expansions):**

| From | Neighbor checked | Condition | Result |
|------|-------------------|-----------|--------|
| (0,1)=2 | (1,1)=2 | 2≥2 ✅ | mark (1,1) |
| (0,2)=2 | (1,2)=3 | 3≥2 ✅ | mark (1,2) |
| (0,3)=3 | (1,3)=4 | 4≥3 ✅ | mark (1,3) |
| (2,0)=2 | (2,1)=4 | 4≥2 ✅ | mark (2,1) |
| (3,0)=6 | (3,1)=7 | 7≥6 ✅ | mark (3,1) |
| (1,2)=3 | (2,2)=5 | 5≥3 ✅ | mark (2,2) |
| (1,3)=4 | (1,4)=4 | 4≥4 ✅ | mark (1,4) |

Final `reachP` (16 cells `true`) matches the grid shown in the Intuition section.

**reachQ BFS trace (key expansions):**

| From | Neighbor checked | Condition | Result |
|------|-------------------|-----------|--------|
| (4,0)=5 | (3,0)=6 | 6≥5 ✅ | mark (3,0) |
| (4,1)=1 | (3,1)=7 | 7≥1 ✅ | mark (3,1) |
| (4,2)=1 | (3,2)=1 | 1≥1 ✅ | mark (3,2) |
| (4,3)=2 | (3,3)=4 | 4≥2 ✅ | mark (3,3) |
| (1,4)=4 | (1,3)=4 | 4≥4 ✅ | mark (1,3) |
| (2,4)=1 | (2,3)=3 | 3≥1 ✅ | mark (2,3) |
| (3,2)=1 | (2,2)=5 | 5≥1 ✅ | mark (2,2) |

Final `reachQ` (16 cells `true`) matches the grid shown in the Intuition section.

**Intersection:** `(0,4), (1,3), (1,4), (2,2), (3,0), (3,1), (4,0)` → **7 cells**

**Output:** `7` ✅

---

## Complexity Analysis

| Metric | Value | Reason |
|---|---|---|
| **Time** | `O(n × m)` | Each of the two BFS passes visits every cell at most once; each cell processes 4 neighbor checks (O(1)) |
| **Space** | `O(n × m)` | Two boolean grids (`reachP`, `reachQ`) plus BFS queues, each bounded by total cell count |

For max constraints `n = m = 10³`: `10⁶` cells — well within time limits.

---

## Key Learning

| Concept | Detail |
|---|---|
| **Core Pattern** | Multi-source BFS from boundaries, with the propagation condition **reversed** |
| **Why Reverse the Condition** | Walking backward from the boundary along a "≤" edge requires "≥" in the reverse direction |
| **Isomorphism** | Structurally identical to LeetCode's Pacific Atlantic Water Flow — same technique, different terminology |
| **Avoiding O((nm)²)** | Never simulate reachability from each cell individually — always seed from the boundary and expand outward once |
| **Two Independent BFS Passes** | `reachP` and `reachQ` are computed completely independently; only combined at the very end via intersection |

### Similar / Related Problems

| Problem | Platform | Connection |
|---|---|---|
| Pacific Atlantic Water Flow | LeetCode #417 | The exact same technique — this problem's direct analogue |
| Number of Enclaves | LeetCode #1020 | Boundary-seeded BFS/DFS pattern |
| Surrounded Regions | LeetCode #130 | Another boundary-multi-source-BFS classic |
| Rotting Oranges | LeetCode #994 | Multi-source BFS (different condition, same mechanics) |

---

## Alternative Approaches

### 1. 🐢 Brute Force — DFS/BFS From Every Cell — O((n×m)²)

For each cell, run a full DFS/BFS to check if it can reach P and separately if it
can reach Q.

```java
// TLE — O((n*m)^2) for n,m up to 10^3 → 10^12 operations, completely infeasible
```

> ❌ **Verdict:** TLE — far too slow for the given constraints.

---

### 2. ✅ Multi-Source BFS From Boundaries with Reversed Condition (Optimal)

Described in full above.

> ✅ **Verdict:** Accepted — `O(n×m)` time, `O(n×m)` space. Matches expected complexity exactly.

---

### Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force (per-cell DFS/BFS) | O((n×m)²) | O(n×m) | ❌ Completely infeasible |
| **Multi-Source BFS (reversed condition)** | **O(n×m)** | **O(n×m)** | ✅ **GFG Expected — use this** |

---

## GitHub Repository Tags

`#java` `#dsa` `#geeksforgeeks` `#potd` `#bfs` `#matrix` `#graph` `#medium` `#multi-source-bfs` `#interview-prep`

---

> 📁 **Part of:** [GeeksforGeeks-POTD-Java](https://github.com/ranjanow/gfg-potd-java)
> 📅 **Date:** July 8, 2026
