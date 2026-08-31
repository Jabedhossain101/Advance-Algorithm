# Algorithms in TypeScript

A collection of classic Graph and Dynamic Programming algorithms implemented in short, compact TypeScript. Each algorithm includes the underlying theory, a step-by-step workflow, time complexity, and runnable code.

## Table of Contents

- [Common Patterns](#common-patterns)
- [Graph Algorithms](#graph-algorithms)
  - [Breadth First Search (BFS)](#breadth-first-search-bfs)
  - [Depth First Search (DFS)](#depth-first-search-dfs)
  - [Prim's Algorithm (MST)](#prims-algorithm-mst)
  - [Kruskal's Algorithm (MST)](#kruskals-algorithm-mst)
- [Dynamic Programming Algorithms](#dynamic-programming-algorithms)
  - [Matrix Chain Multiplication (MCM)](#matrix-chain-multiplication-mcm)
  - [Longest Common Subsequence (LCS)](#longest-common-subsequence-lcs)

---

## Common Patterns

Two families of algorithms are covered here, and each family shares a common building block:

### 1. Adjacency List (used in BFS, DFS, Prim's, Kruskal's — except Kruskal's uses edges directly)

Instead of using a full adjacency matrix (which wastes memory for sparse graphs), every graph algorithm here builds an **adjacency list**: an array where `graph[u]` holds a list of nodes directly connected to `u`.

```typescript
const graph: number[][] = Array.from({ length: n }, () => []);
edges.forEach(([u, v]) => (graph[u].push(v), graph[v].push(u)));
```

This runs in `O(V + E)` and is reused as the starting point for BFS, DFS, and Prim's.

### 2. DP Table (used in MCM, LCS)

Both dynamic programming problems build a 2D table `d[i][j]` that stores the optimal answer for a subproblem, then combine smaller subproblems into larger ones.

```typescript
const d = Array.from({ length: rows }, () => Array(cols).fill(0));
```

The core idea in DP is always: **solve small ranges first, reuse them to build bigger ranges** — never recompute the same subproblem twice.

---

## Graph Algorithms

### Breadth First Search (BFS)

**Theory:**
BFS explores a graph **level by level** — it visits all direct neighbors of the starting node first, then their neighbors, and so on. It uses a **queue (FIFO)** to always process the earliest-discovered node next. BFS guarantees the **shortest path in terms of number of edges** on unweighted graphs.

**Workflow:**
1. Start at a given node, mark it visited, push it into a queue.
2. Pop the front node from the queue.
3. Visit all its unvisited neighbors — mark them visited and push into the queue.
4. Repeat until the queue is empty.

**Time Complexity:** `O(V + E)`

```typescript
function bfs(n, edges, start) {
  const g = Array.from({ length: n }, () => []), q = [start], vis = new Set(q);
  edges.forEach(([u, v]) => (g[u].push(v), g[v].push(u)));
  for (let i = 0; i < q.length; i++)
    g[q[i]].forEach(v => !vis.has(v) && (vis.add(v), q.push(v)));
  return q;
}

const [n, start] = (prompt("Vertices Start:") || "").split(" ").map(Number);
const edges = (prompt("Edges (u v):") || "").split(";").map(e => e.split(" ").map(Number));
console.log("BFS =", bfs(n, edges, start));
```

**Input:**
```
Vertices Start: 5 0
Edges (u v): 0 1;0 2;1 3;2 4;3 4
```

**Output:**
```
BFS = [0, 1, 2, 3, 4]
```

---

### Depth First Search (DFS)

**Theory:**
DFS explores a graph by going **as deep as possible along one path** before backtracking. It uses **recursion (or a stack)** instead of a queue. Unlike BFS, it does not guarantee shortest paths, but it's useful for detecting cycles, topological sorting, and connected components.

**Workflow:**
1. Start at a given node, mark it visited.
2. Move to an unvisited neighbor, mark it visited, and recurse from there.
3. When no unvisited neighbor remains, backtrack to the previous node.
4. Repeat until every reachable node is visited.

**Time Complexity:** `O(V + E)`

```typescript
function dfs(n, edges, start) {
  const g = Array.from({ length: n }, () => []), vis = new Set(), res = [];
  edges.forEach(([u, v]) => (g[u].push(v), g[v].push(u)));
  const f = (u) => (vis.add(u), res.push(u), g[u].forEach(v => !vis.has(v) && f(v)));
  return f(start), res;
}

const [n, start] = (prompt("Vertices Start:") || "").split(" ").map(Number);
const edges = (prompt("Edges (u v):") || "").split(";").map(e => e.split(" ").map(Number));
console.log("DFS =", dfs(n, edges, start));
```

**Input:**
```
Vertices Start: 5 0
Edges (u v): 0 1;0 2;1 3;2 4;3 4
```

**Output:**
```
DFS = [0, 1, 3, 4, 2]
```

---

### Prim's Algorithm (MST)

**Theory:**
Prim's Algorithm builds a **Minimum Spanning Tree (MST)** — a subset of edges that connects all nodes with the **minimum total edge weight**, without forming any cycle. It grows the tree **one node at a time**, always picking the cheapest edge that connects the current tree to a new, unvisited node. It's a **greedy algorithm**: it never reconsiders a choice once made.

**Workflow:**
1. Start with a single node (node `0`) in the "visited" set.
2. Look at every edge going out from the visited set to an unvisited node.
3. Pick the cheapest such edge and add its new node to the visited set.
4. Repeat until all nodes are included (`n - 1` edges chosen).

**Time Complexity:** `O(V² )` in this simple array-based version (can be improved to `O(E log V)` with a priority queue).

> ⚠️ Prim's always starts from node `0`, so node `0` must be connected to the rest of the graph, or the result will be empty.

```typescript
function prim(n, edges) {
  const vis = new Set([0]), mst = [];
  while (vis.size < n) {
    const e = edges.filter(([u, v]) => vis.has(u) ^ vis.has(v)).sort((a, b) => a[2] - b[2])[0];
    if (!e) break;
    vis.add(e[0]).add(e[1]);
    mst.push(e);
  }
  return mst;
}

const n = +prompt("Vertices:");
const edges = prompt("Edges (u v w):").split(";").map(e => e.trim().split(" ").map(Number));
console.log("MST (Prim) =", prim(n, edges));


```

**Input:**
```
Vertices: 5
Edges (u v w): 0 1 2;0 3 6;1 2 3;1 3 8;1 4 5;2 4 7;3 4 9
```

**Output:**
```
MST = [[0, 1, 2], [1, 2, 3], [0, 3, 6], [1, 4, 5]]
```

---

### Kruskal's Algorithm (MST)

**Theory:**
Kruskal's Algorithm also builds a Minimum Spanning Tree, but with a different strategy: instead of growing from one node, it looks at **all edges sorted by weight** and greedily adds the cheapest edge as long as it **doesn't form a cycle**. Cycle detection is done efficiently using a **Union-Find (Disjoint Set)** structure, which groups nodes into sets and tells us instantly whether two nodes are already connected.

**Workflow:**
1. Sort all edges by weight (ascending).
2. Initialize every node as its own separate group.
3. For each edge, check if its two nodes belong to different groups.
4. If yes, add the edge to the MST and merge the two groups. If no, skip it (it would create a cycle).
5. Repeat until all edges are processed.

**Time Complexity:** `O(E log E)` — dominated by the sort.

> ✅ Unlike Prim's, Kruskal's doesn't need a fixed starting node — it works even if node `0` is isolated from the rest.

```typescript
function kruskal(n, edges) {
  const p = Array.from({ length: n }, (_, i) => i);
  const find = (i) => p[i] === i ? i : (p[i] = find(p[i]));

  return edges.sort((a, b) => a[2] - b[2]).filter(([u, v]) => {
    const rootU = find(u), rootV = find(v);
    return rootU !== rootV && (p[rootU] = rootV, true);
  });
}

const n = +prompt("Vertices:");
const edges = prompt("Edges (u v w):").split(";").map(e => e.trim().split(" ").map(Number));
console.log("MST (Kruskal) =", kruskal(n, edges));
```

**Input:**
```
Vertices: 5
Edges (u v w): 0 1 2;0 3 6;1 2 3;1 3 8;1 4 5;2 4 7;3 4 9
```

**Output:**
```
MST = [[0, 1, 2], [1, 2, 3], [0, 3, 6], [1, 4, 5]]
```

---

## Dynamic Programming Algorithms

### Matrix Chain Multiplication (MCM)

**Theory:**
Given a chain of matrices to multiply, the **order** in which you multiply them changes the total number of scalar multiplications — even though the final result is the same. MCM finds the **cheapest order to parenthesize the multiplications**. It solves this with DP by trying every possible "split point" for each sub-chain and keeping the cheapest one.

**Workflow:**
1. Represent matrix dimensions as an array `p`, where matrix `i` has dimensions `p[i-1] x p[i]`.
2. For every chain length `l` (from 2 up to full length), and every starting index `i`:
   - Try every possible split point `k` inside that chain.
   - Cost of a split = cost of left part + cost of right part + cost of multiplying the two resulting matrices.
   - Keep the minimum cost among all splits.
3. The answer is stored in `d[1][n-1]`, covering the whole chain.

**Time Complexity:** `O(n³)`

```typescript
function mcm(p) {
  const n = p.length, d = Array.from({ length: n }, () => Array(n).fill(0));
  for (let l = 2; l < n; l++)
    for (let i = 1; i < n - l + 1; i++) {
      const j = i + l - 1;
      d[i][j] = Infinity;
      for (let k = i; k < j; k++)
        d[i][j] = Math.min(d[i][j], d[i][k] + d[k + 1][j] + p[i - 1] * p[k] * p[j]);
    }
  return d[1][n - 1];
}

const p = (prompt("Dimensions:") || "").split(" ").map(Number);
console.log("Minimum =", mcm(p));
```

**Input:**
```
Dimensions: 40 20 30 10 30
```
*(Meaning: 4 matrices of sizes 40×20, 20×30, 30×10, 10×30)*

**Output:**
```
Minimum = 26000
```

---

### Longest Common Subsequence (LCS)

**Theory:**
Given two strings, LCS finds the **longest sequence of characters that appears in both strings in the same relative order** (not necessarily contiguous). It's solved with DP by comparing characters of both strings one pair at a time: if they match, extend the previous diagonal result; if not, take the best of skipping a character from either string.

**Workflow:**
1. Build a table `d[i][j]` = length of LCS between the first `i` characters of string `a` and first `j` characters of string `b`.
2. If `a[i-1] === b[j-1]`, the characters match → `d[i][j] = d[i-1][j-1] + 1`.
3. Otherwise, take the best of ignoring one character → `d[i][j] = max(d[i-1][j], d[i][j-1])`.
4. After filling the table, walk backwards from `d[a.length][b.length]` to reconstruct the actual subsequence string.

**Time Complexity:** `O(a.length × b.length)`

```typescript
function lcs(a: string, b: string) {
  const d = Array.from({ length: a.length + 1 }, () => Array(b.length + 1).fill(0));
  for (let i = 1; i <= a.length; i++)
    for (let j = 1; j <= b.length; j++)
      d[i][j] = a[i - 1] === b[j - 1] ? d[i - 1][j - 1] + 1 : Math.max(d[i - 1][j], d[i][j - 1]);
  let i = a.length, j = b.length, s = "";
  while (i && j)
    if (a[i - 1] === b[j - 1]) (s = a[i - 1] + s, i--, j--);
    else d[i - 1][j] > d[i][j - 1] ? i-- : j--;
  return s;
}

const a = prompt("First:") || "", b = prompt("Second:") || "";
console.log("LCS =", lcs(a, b));
console.log("length =", lcs(a, b).length);
```

**Input:**
```
First: ABCBDAB
Second: BDCABA
```

**Output:**
```
LCS = BCBA
length = 4
```

---

## How to Run

All snippets use `prompt()`, so they're written to run in a browser console, on [TypeScript Playground](https://www.typescriptlang.org/play), or on any online TS runner (e.g. Programiz) that supports `prompt()`. Paste the **full block** (function + input + output lines) — a function definition alone won't produce any output since it's never called.

## Summary Table

| Algorithm | Category       | Strategy      | Time Complexity |
|-----------|-----------------|---------------|------------------|
| BFS       | Graph Traversal | Queue-based   | O(V + E)         |
| DFS       | Graph Traversal | Recursion     | O(V + E)         |
| Prim's    | MST             | Greedy        | O(V²)            |
| Kruskal's | MST             | Greedy + Union-Find | O(E log E) |
| MCM       | Dynamic Programming | Interval DP | O(n³)          |
| LCS       | Dynamic Programming | Grid DP     | O(a·b)         |
