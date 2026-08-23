# Graph Algorithms — Complete Guide
> BFS, DFS, Dijkstra, Bellman-Ford, Topological Sort, Union-Find + MST

---

## Table of Contents
1. [How to Identify Which Algorithm to Use](#1-how-to-identify-which-algorithm-to-use)
2. [The 5-Question Framework](#2-the-5-question-framework)
3. [BFS — Breadth First Search](#3-bfs--breadth-first-search)
4. [DFS — Depth First Search](#4-dfs--depth-first-search)
5. [Dijkstra](#5-dijkstra)
6. [Bellman-Ford](#6-bellman-ford)
7. [Topological Sort](#7-topological-sort)
8. [Union-Find + MST](#8-union-find--mst)
9. [Problem List with Solutions](#9-problem-list-with-solutions)
10. [2-Week Practice Plan](#10-2-week-practice-plan)

---

## 1. How to Identify Which Algorithm to Use

```
Is the graph WEIGHTED?
│
├── NO (all edges = 1 or unweighted)
│    └── BFS → shortest path
│         DFS → connectivity, cycles, paths, tree properties
│
└── YES (edges have different costs)
     ├── All weights NON-NEGATIVE → Dijkstra
     ├── Negative weights         → Bellman-Ford
     ├── Weights only 0 or 1      → 0-1 BFS (deque)
     └── All-pairs shortest path  → Floyd-Warshall
```

```
Quick cheat sheet:
─────────────────────────────────────────────────────────
"Shortest path"          → BFS (unweighted) / Dijkstra (weighted)
"Count groups/islands"   → DFS or BFS (connectivity)
"Can I finish / cycle?"  → DFS (cycle detection)
"All possible paths"     → DFS (backtracking)
"Level by level"         → BFS
"Dependencies/ordering"  → Topological Sort (DFS or BFS/Kahn's)
─────────────────────────────────────────────────────────
```

---

## 2. The 5-Question Framework

Ask these before writing ANY code:

```
1. State?       → what does ONE node represent?
                  (cell, word, number, combination, bus...)

2. Neighbors?   → what is ONE valid move from current state?
                  (4-dirs, 8-dirs, change 1 char, ±1 operation...)

3. Start?       → where does traversal begin?
                  (single source or multi-source?)

4. End/Goal?    → when do you stop?
                  (target found, all nodes visited, component done...)

5. Invalid?     → what do you prune/skip?
                  (out of bounds, wall, visited, dead end, constraint exceeded...)
```

Once you answer these 5, the code writes itself.

---

## 3. BFS — Breadth First Search

### When to Use
- Shortest path in **unweighted** graphs
- Level-by-level exploration
- Multi-source problems (seed multiple start nodes)
- State space search (string/combo transformations)

### How to Think About Neighbors

```
Problem Type         Neighbor Generation
────────────────────────────────────────────────────────
Grid (4-dir)         int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}}
Grid (8-dir)         int[][] dirs = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}}
String transform     change 1 char to each valid char at each position
Combination          ±1 on each digit/dial (with wrap)
State space          apply each valid operation to current state
```

### The BFS Mental Model

```
BFS explores nodes LEVEL BY LEVEL:
Level 0: start node
Level 1: all nodes 1 step away
Level 2: all nodes 2 steps away
...
First time you reach a node = SHORTEST path to it
```

### Skeleton 1: Basic BFS (distance)

```java
int bfs(Map<Integer, List<Integer>> graph, int start, int target) {
    Set<Integer> visited = new HashSet<>();
    Queue<Integer> queue = new LinkedList<>();
    visited.add(start);
    queue.offer(start);
    int steps = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();   // snapshot current level
        steps++;
        for (int i = 0; i < size; i++) {
            int node = queue.poll();
            for (int neighbor : graph.getOrDefault(node, List.of())) {
                if (neighbor == target) return steps;
                if (visited.contains(neighbor)) continue;
                visited.add(neighbor);           // mark at INSERT time
                queue.offer(neighbor);
            }
        }
    }
    return -1;
}
```

### Skeleton 2: Grid BFS

```java
int bfsGrid(char[][] grid, int sr, int sc, int er, int ec) {
    int rows = grid.length, cols = grid[0].length;
    boolean[][] visited = new boolean[rows][cols];
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

    Queue<int[]> queue = new LinkedList<>();
    visited[sr][sc] = true;
    queue.offer(new int[]{sr, sc, 0});  // {row, col, distance}

    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        int r = curr[0], c = curr[1], dist = curr[2];
        if (r == er && c == ec) return dist;
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
            if (visited[nr][nc] || grid[nr][nc] == '#') continue;
            visited[nr][nc] = true;
            queue.offer(new int[]{nr, nc, dist + 1});
        }
    }
    return -1;
}
```

### Skeleton 3: Multi-Source BFS

```java
// Seed ALL sources at once before starting BFS
Queue<int[]> queue = new LinkedList<>();
boolean[][] visited = new boolean[rows][cols];

for (int r = 0; r < rows; r++)
    for (int c = 0; c < cols; c++)
        if (isSource(grid[r][c])) {
            queue.offer(new int[]{r, c});
            visited[r][c] = true;
        }

int steps = 0;
while (!queue.isEmpty()) {
    int size = queue.size();
    steps++;
    for (int i = 0; i < size; i++) {
        int[] cell = queue.poll();
        for (int[] d : dirs) {
            int nr = cell[0] + d[0], nc = cell[1] + d[1];
            if (/* valid and not visited */) {
                visited[nr][nc] = true;
                queue.offer(new int[]{nr, nc});
            }
        }
    }
}
```

### Key Rules — BFS

```
✅ Mark visited at INSERT (offer) time, NOT at poll time
✅ Use size = queue.size() snapshot when you need level/distance tracking
✅ For multi-source: seed ALL sources before the while loop
✅ State must capture EVERYTHING needed to make a decision
   → if constraint exists, add it to state: (node, k) or (r, c, k)
```

### The "Change State" Trick for String/Combo BFS

```java
// For string/combo state — mutate char[], restore after each try
char[] arr = word.toCharArray();
for (int i = 0; i < arr.length; i++) {
    char original = arr[i];
    for (char c = 'a'; c <= 'z'; c++) {
        if (c == original) continue;
        arr[i] = c;
        String next = new String(arr);
        // process next...
    }
    arr[i] = original;  // ALWAYS restore before next position
}
```

---

## 4. DFS — Depth First Search

### The 4 DFS Families

```
┌──────────────────────────────────────────────────────────────┐
│ 1. CONNECTIVITY     "How many groups? Are A and B connected?" │
│    → Number of Islands, Number of Provinces                   │
│                                                               │
│ 2. CYCLE DETECTION  "Is there a loop? Can this be ordered?"  │
│    → Course Schedule, Detect cycle in directed graph          │
│                                                               │
│ 3. PATH/BACKTRACKING "Find ALL paths / does ANY path exist?"  │
│    → All Paths Source to Target, Word Search                  │
│                                                               │
│ 4. TREE PROPERTIES  "Compute something bottom-up"             │
│    → Max depth, diameter, subtree sums                        │
└──────────────────────────────────────────────────────────────┘
```

### Visited Strategy by Family — The Most Important Table

```
Family              Visited Strategy
────────────────────────────────────────────────────────────────
Connectivity        Permanent visited[] — once marked, never unmark
                    boolean[] visited = new boolean[n]

Cycle Detection     3-state: 0=white, 1=gray, 2=black
(directed)          GRAY = on current path → finding gray = cycle
                    Must UN-mark (gray→white) on backtrack

Path/Backtracking   Mark on ENTER, UNMARK on EXIT
                    Same node can appear in different paths
                    visited[node] = true  ... visited[node] = false

Tree Properties     No visited needed (trees have no cycles)
                    Just recurse and return value bottom-up
```

### Skeleton 1: Connectivity (Count Components)

```java
// Outer loop
boolean[] visited = new boolean[n];
int count = 0;
for (int i = 0; i < n; i++) {
    if (!visited[i]) {
        dfs(i, visited, graph);
        count++;   // each new DFS = new component
    }
}
return count;

// DFS inner
void dfs(int node, boolean[] visited, Map<Integer, List<Integer>> graph) {
    visited[node] = true;
    for (int neighbor : graph.getOrDefault(node, List.of())) {
        if (!visited[neighbor]) dfs(neighbor, visited, graph);
    }
}
```

### Skeleton 2: Cycle Detection (Directed Graph)

```java
int[] state = new int[n]; // 0=white, 1=gray, 2=black

boolean hasCycle(int node, int[] state) {
    state[node] = 1;  // GRAY
    for (int neighbor : graph.getOrDefault(node, List.of())) {
        if (state[neighbor] == 1) return true;   // gray = cycle!
        if (state[neighbor] == 0 && hasCycle(neighbor, state)) return true;
        // state[neighbor] == 2 → already proven safe, skip
    }
    state[node] = 2;  // BLACK — done, no cycle through here
    return false;
}
```

### Skeleton 3: Path / Backtracking

```java
void dfs(int node, List<Integer> path, boolean[] visited) {
    visited[node] = true;
    path.add(node);

    if (node == target) {
        result.add(new ArrayList<>(path));  // copy — don't add reference
    } else {
        for (int neighbor : graph.get(node)) {
            if (!visited[neighbor]) {
                dfs(neighbor, path, visited);
            }
        }
    }

    // BACKTRACK — undo so other paths can use this node
    path.remove(path.size() - 1);
    visited[node] = false;
}
```

### Skeleton 4: Tree Properties (Bottom-Up)

```java
int dfs(TreeNode node) {
    if (node == null) return 0;      // base case
    int left  = dfs(node.left);      // get left subtree result
    int right = dfs(node.right);     // get right subtree result
    return 1 + Math.max(left, right); // combine + return to parent
}
```

### Key Rules — DFS

```
✅ Connectivity   → permanent visited[], count++ each new DFS start
✅ Cycle detect   → 3-state (white/gray/black), gray = "on my path"
✅ Backtracking   → mark on enter, UNMARK on exit
✅ Tree props     → no visited needed, return values bubble up
✅ Iterative DFS  → use Deque as stack, mark visited at POP time
```

---

## 5. Dijkstra

### When to Use
- Shortest path in **weighted** graph
- All edge weights **non-negative**
- "Minimize cost/time/effort/distance" from source to destination

### Core Idea

```
Always expand the node with SMALLEST known distance first.
Use a min-heap (PriorityQueue) — not a plain queue.

Unlike BFS (all edges = 1), Dijkstra handles varying costs.
The min-heap ensures we always process the globally cheapest
unprocessed node — greedy choice is safe with non-negative weights.
```

### The Cost Formula — This Changes by Problem

```
Standard (minimize sum):     newDist = dist[node] + weight
Minimize max effort:         newDist = Math.max(dist[node], diff)
Maximize probability:        newDist = dist[node] * probability  (flip to log)
Compound state (+ k stops):  dist[node][stops] = min cost with stops used
```

### Skeleton 1: Standard Dijkstra

```java
int[] dijkstra(List<int[]>[] adj, int n, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // min-heap: {cost, node}
    PriorityQueue<int[]> pq = new PriorityQueue<>(
        (a, b) -> Integer.compare(a[0], b[0])
    );
    pq.offer(new int[]{0, src});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int cost = curr[0], node = curr[1];

        if (cost > dist[node]) continue;  // stale entry — skip

        for (int[] edge : adj[node]) {
            int neighbor = edge[0], weight = edge[1];
            int newDist = dist[node] + weight;
            if (newDist < dist[neighbor]) {
                dist[neighbor] = newDist;
                pq.offer(new int[]{newDist, neighbor});
            }
        }
    }
    return dist;
}
```

### Skeleton 2: Grid Dijkstra

```java
int dijkstraGrid(int[][] heights) {
    int rows = heights.length, cols = heights[0].length;
    int[][] dist = new int[rows][cols];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
    dist[0][0] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>(
        (a, b) -> Integer.compare(a[0], b[0])
    );
    pq.offer(new int[]{0, 0, 0});  // {cost, row, col}

    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int cost = curr[0], r = curr[1], c = curr[2];

        if (r == rows-1 && c == cols-1) return cost;  // early return
        if (cost > dist[r][c]) continue;               // stale

        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
            int newCost = /* compute based on problem */;
            if (newCost < dist[nr][nc]) {
                dist[nr][nc] = newCost;
                pq.offer(new int[]{newCost, nr, nc});
            }
        }
    }
    return dist[rows-1][cols-1];
}
```

### Skeleton 3: Compound State Dijkstra

```java
// State = (node, extra_dimension) e.g. stops used, obstacles removed
int[][] dist = new int[n][k+2];  // dist[node][stops]
for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
dist[src][0] = 0;

PriorityQueue<int[]> pq = new PriorityQueue<>(
    (a, b) -> Integer.compare(a[0], b[0])
);
pq.offer(new int[]{0, src, 0});  // {cost, node, stops}

while (!pq.isEmpty()) {
    int[] curr = pq.poll();
    int cost = curr[0], node = curr[1], stops = curr[2];

    if (node == dst) return cost;
    if (cost > dist[node][stops]) continue;
    if (stops > k) continue;  // constraint exceeded

    for (int[] edge : adj[node]) {
        int newCost = cost + edge[1];
        if (newCost < dist[edge[0]][stops+1]) {
            dist[edge[0]][stops+1] = newCost;
            pq.offer(new int[]{newCost, edge[0], stops+1});
        }
    }
}
return -1;
```

### Building Adjacency List from Edge List

```java
// edge list → adjacency list (do this every time before Dijkstra)
List<int[]>[] adj = new ArrayList[n];
for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();

for (int[] edge : edges) {
    // Directed: u → v
    adj[edge[0]].add(new int[]{edge[1], edge[2]});  // {neighbor, weight}

    // Undirected: add both directions
    // adj[edge[1]].add(new int[]{edge[0], edge[2]});
}
```

### Key Rules — Dijkstra

```
✅ Always use min-heap (PriorityQueue), NOT a plain queue
✅ Stale check: if (cost > dist[node]) continue
✅ Initialize dist[src] = 0, everything else = Integer.MAX_VALUE
✅ Heap entry must include cost as FIRST element (used for ordering)
✅ For grid problems: heap entry = {cost, row, col}
✅ For compound state: heap entry = {cost, node, extraDimension}
✅ Only works with NON-NEGATIVE weights
```

### Complexity

```
Time:  O((V + E) log V)
Space: O(V + E)

Dense graph (E ≈ V²) → O(V² log V)
Sparse graph (E ≈ V) → O(V log V)
```

---

## 6. Problem List with Solutions

---

### BFS Problems

---

#### 1. Word Ladder (LC 127)
**Description:** Transform `beginWord` to `endWord` one letter at a time. Each intermediate word must be in the given word list. Return the minimum number of transformations, or 0 if no such sequence exists.

**Example:**
- Input: beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
- Output: 5  ("hit" → "hot" → "dot" → "dog" → "cog")

**Pattern:** String-state BFS, single source
**State:** word (String)
**Neighbors:** change 1 char to any letter, check if in wordSet
**Cost metric:** number of transformations

```java
int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> wordSet = new HashSet<>(wordList);
    if (!wordSet.contains(endWord)) return 0;

    Queue<String> queue = new LinkedList<>();
    queue.offer(beginWord);
    wordSet.remove(beginWord);
    int steps = 1;

    while (!queue.isEmpty()) {
        int size = queue.size();
        steps++;
        for (int i = 0; i < size; i++) {
            char[] word = queue.poll().toCharArray();
            for (int j = 0; j < word.length; j++) {
                char original = word[j];
                for (char c = 'a'; c <= 'z'; c++) {
                    if (c == original) continue;
                    word[j] = c;
                    String next = new String(word);
                    if (next.equals(endWord)) return steps;
                    if (wordSet.contains(next)) {
                        wordSet.remove(next);
                        queue.offer(next);
                    }
                }
                word[j] = original;
            }
        }
    }
    return 0;
}
```

---

#### 2. Rotting Oranges (LC 994)
**Description:** Grid with 0=empty, 1=fresh orange, 2=rotten orange. Each minute, rotten oranges rot adjacent (4-directional) fresh ones. Return minimum minutes until no fresh oranges remain, or -1 if impossible.

**Example:**
- Input: grid = [[2,1,1],[1,1,0],[0,1,1]]
- Output: 4

**Pattern:** Multi-source BFS
**State:** (row, col)
**Neighbors:** 4 directions
**Key:** Seed ALL rotten oranges before BFS, count fresh

```java
int orangesRotting(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    Queue<int[]> queue = new LinkedList<>();
    int fresh = 0;

    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == 2) queue.offer(new int[]{r, c});
            if (grid[r][c] == 1) fresh++;
        }

    if (fresh == 0) return 0;

    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    int minutes = 0;

    while (!queue.isEmpty() && fresh > 0) {
        minutes++;
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int[] cell = queue.poll();
            for (int[] d : dirs) {
                int nr = cell[0]+d[0], nc = cell[1]+d[1];
                if (nr>=0 && nr<rows && nc>=0 && nc<cols && grid[nr][nc]==1) {
                    grid[nr][nc] = 2;
                    fresh--;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }
    }
    return fresh == 0 ? minutes : -1;
}
```

---

#### 3. Open the Lock (LC 752)
**Description:** 4-dial combination lock starts at "0000". Each turn rotates one dial by 1 position (0↔9 wrap). Given dead ends and a target, return minimum turns to reach target, or -1 if impossible.

**Example:**
- Input: deadends = ["0201","0101","0102","1212","2002"], target = "0202"
- Output: 6

**Pattern:** Combination-state BFS
**State:** 4-digit string
**Neighbors:** 8 (each dial ±1, wrap 0↔9)
**Invalid:** deadends set

```java
int openLock(String[] deadends, String target) {
    Set<String> dead = new HashSet<>(Arrays.asList(deadends));
    if (dead.contains("0000")) return -1;
    if (target.equals("0000")) return 0;

    Set<String> visited = new HashSet<>();
    Queue<String> queue = new LinkedList<>();
    visited.add("0000");
    queue.offer("0000");
    int steps = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();
        steps++;
        for (int i = 0; i < size; i++) {
            char[] arr = queue.poll().toCharArray();
            for (int j = 0; j < 4; j++) {
                char original = arr[j];
                for (int dir : new int[]{1, -1}) {
                    arr[j] = (char)('0' + (original - '0' + dir + 10) % 10);
                    String next = new String(arr);
                    if (next.equals(target)) return steps;
                    if (!visited.contains(next) && !dead.contains(next)) {
                        visited.add(next);
                        queue.offer(next);
                    }
                }
                arr[j] = original;
            }
        }
    }
    return -1;
}
```

---

#### 4. Minimum Genetic Mutation (LC 433)
**Description:** A gene string is 8 characters from {A,C,G,T}. One mutation changes one character. Each intermediate gene must exist in the bank. Return minimum mutations to go from startGene to endGene, or -1 if impossible.

**Example:**
- Input: startGene = "AACCGGTT", endGene = "AAACGGTA", bank = ["AACCGGTA","AACCGCTA","AAACGGTA"]
- Output: 2  (AACCGGTT → AACCGGTA → AAACGGTA)

**Pattern:** String-state BFS (same as Word Ladder)
**State:** gene string (8 chars, ACGT)
**Neighbors:** change 1 char to A/C/G/T, must be in bank

```java
int minMutation(String startGene, String endGene, String[] bank) {
    Set<String> bankSet = new HashSet<>(Arrays.asList(bank));
    if (!bankSet.contains(endGene)) return -1;

    Queue<String> queue = new LinkedList<>();
    Set<String> visited = new HashSet<>();
    queue.offer(startGene);
    visited.add(startGene);
    int mutations = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();
        mutations++;
        for (int i = 0; i < size; i++) {
            char[] arr = queue.poll().toCharArray();
            for (int j = 0; j < 8; j++) {
                char original = arr[j];
                for (char c : new char[]{'A','C','G','T'}) {
                    if (c == original) continue;
                    arr[j] = c;
                    String next = new String(arr);
                    if (next.equals(endGene)) return mutations;
                    if (!visited.contains(next) && bankSet.contains(next)) {
                        visited.add(next);
                        queue.offer(next);
                    }
                }
                arr[j] = original;
            }
        }
    }
    return -1;
}
```

---

#### 5. Bus Routes (LC 815)
**Description:** Given bus routes (each route is a list of stops), find the minimum number of buses to ride to travel from source to target stop. You may board any bus at any stop it serves.

**Example:**
- Input: routes = [[1,2,7],[3,6,7]], source = 1, target = 6
- Output: 2  (bus 0: 1→7, bus 1: 7→6)

**Pattern:** Non-obvious state BFS (state = bus, not stop)
**State:** bus index
**Neighbors:** all buses sharing a stop with current bus
**Key insight:** counting bus transfers, not stop hops

```java
int numBusesToDestination(int[][] routes, int source, int target) {
    if (source == target) return 0;

    Map<Integer, List<Integer>> stopToBuses = new HashMap<>();
    for (int i = 0; i < routes.length; i++)
        for (int stop : routes[i])
            stopToBuses.computeIfAbsent(stop, k -> new ArrayList<>()).add(i);

    Set<Integer> visitedBuses = new HashSet<>();
    Set<Integer> visitedStops = new HashSet<>();
    Queue<Integer> queue = new LinkedList<>();
    int buses = 0;

    for (int bus : stopToBuses.getOrDefault(source, List.of())) {
        queue.offer(bus);
        visitedBuses.add(bus);
    }
    visitedStops.add(source);

    while (!queue.isEmpty()) {
        int size = queue.size();
        buses++;
        for (int i = 0; i < size; i++) {
            int bus = queue.poll();
            for (int stop : routes[bus]) {
                if (stop == target) return buses;
                if (visitedStops.contains(stop)) continue;
                visitedStops.add(stop);
                for (int nextBus : stopToBuses.getOrDefault(stop, List.of())) {
                    if (!visitedBuses.contains(nextBus)) {
                        visitedBuses.add(nextBus);
                        queue.offer(nextBus);
                    }
                }
            }
        }
    }
    return -1;
}
```

---

#### 6. Shortest Path in Binary Matrix (LC 1091)
**Description:** Find the shortest clear path from top-left (0,0) to bottom-right (n-1,n-1) in an n×n binary grid. Only 0-cells are passable, movement is 8-directional. Return path length in cells, or -1.

**Example:**
- Input: grid = [[0,0,0],[1,1,0],[1,1,0]]
- Output: 4

**Pattern:** 8-directional grid BFS
**State:** (row, col)
**Neighbors:** 8 directions
**Invalid:** cell == 1 (wall), out of bounds

```java
int shortestPathBinaryMatrix(int[][] grid) {
    int n = grid.length;
    if (grid[0][0] == 1 || grid[n-1][n-1] == 1) return -1;
    if (n == 1) return 1;

    boolean[][] visited = new boolean[n][n];
    int[][] dirs = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};
    Queue<int[]> queue = new LinkedList<>();
    visited[0][0] = true;
    queue.offer(new int[]{0, 0});
    int cells = 1;

    while (!queue.isEmpty()) {
        int size = queue.size();
        cells++;
        for (int i = 0; i < size; i++) {
            int[] cell = queue.poll();
            for (int[] d : dirs) {
                int nr = cell[0]+d[0], nc = cell[1]+d[1];
                if (nr<0||nr>=n||nc<0||nc>=n||visited[nr][nc]||grid[nr][nc]==1) continue;
                if (nr==n-1 && nc==n-1) return cells;
                visited[nr][nc] = true;
                queue.offer(new int[]{nr, nc});
            }
        }
    }
    return -1;
}
```

---

### DFS Problems

---

#### 7. Number of Provinces (LC 547)
**Description:** There are n cities. isConnected[i][j] = 1 if cities i and j are directly connected. A province is a group of directly or indirectly connected cities. Return the number of provinces.

**Example:**
- Input: isConnected = [[1,1,0],[1,1,0],[0,0,1]]
- Output: 2  (cities 0,1 form one province; city 2 is another)

**Pattern:** Connectivity — adjacency matrix
**Family:** Connectivity
**Visited:** permanent boolean[]

```java
int findCircleNum(int[][] isConnected) {
    int n = isConnected.length;
    boolean[] visited = new boolean[n];
    int provinces = 0;

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfs(isConnected, i, visited);
            provinces++;
        }
    }
    return provinces;
}

void dfs(int[][] isConnected, int city, boolean[] visited) {
    visited[city] = true;
    for (int j = 0; j < isConnected.length; j++)
        if (isConnected[city][j] == 1 && !visited[j])
            dfs(isConnected, j, visited);
}
```

---

#### 8. Number of Islands (LC 200)
**Description:** Given a grid of '1' (land) and '0' (water), count the number of islands. An island is a group of adjacent '1's connected 4-directionally, surrounded by water or boundary.

**Example:**
- Input: grid = [["1","1","0","0"],["1","1","0","0"],["0","0","1","0"],["0","0","0","1"]]
- Output: 3

**Pattern:** Connectivity — grid
**Family:** Connectivity
**Visited:** mark grid cell as '0' (sink island) or separate visited[]

```java
int numIslands(char[][] grid) {
    int m = grid.length, n = grid[0].length;
    boolean[][] visited = new boolean[m][n];
    int islands = 0;

    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (!visited[i][j] && grid[i][j] == '1') {
                dfs(grid, i, j, visited, m, n);
                islands++;
            }
    return islands;
}

void dfs(char[][] grid, int r, int c, boolean[][] visited, int m, int n) {
    if (r<0||r>=m||c<0||c>=n||visited[r][c]||grid[r][c]=='0') return;
    visited[r][c] = true;
    dfs(grid, r-1, c, visited, m, n);
    dfs(grid, r+1, c, visited, m, n);
    dfs(grid, r, c-1, visited, m, n);
    dfs(grid, r, c+1, visited, m, n);
}
```

---

#### 9. Course Schedule (LC 207)
**Description:** There are numCourses courses (0 to numCourses-1). prerequisites[i] = [a, b] means you must take b before a. Determine if it's possible to finish all courses (no circular dependency).

**Example:**
- Input: numCourses = 2, prerequisites = [[1,0]]
- Output: true
- Input: numCourses = 2, prerequisites = [[1,0],[0,1]]
- Output: false  (cycle: course 0 requires 1, course 1 requires 0)

**Pattern:** Cycle detection in directed graph
**Family:** Cycle Detection
**Visited:** 3-state (0=white, 1=gray, 2=black)

```java
boolean canFinish(int numCourses, int[][] prerequisites) {
    Map<Integer, List<Integer>> graph = new HashMap<>();
    for (int[] pre : prerequisites)
        graph.computeIfAbsent(pre[1], k -> new ArrayList<>()).add(pre[0]);

    int[] state = new int[numCourses];
    for (int i = 0; i < numCourses; i++)
        if (state[i] == 0 && hasCycle(i, graph, state)) return false;
    return true;
}

boolean hasCycle(int node, Map<Integer, List<Integer>> graph, int[] state) {
    state[node] = 1;  // GRAY
    for (int neighbor : graph.getOrDefault(node, List.of())) {
        if (state[neighbor] == 1) return true;
        if (state[neighbor] == 0 && hasCycle(neighbor, graph, state)) return true;
    }
    state[node] = 2;  // BLACK
    return false;
}
```

---

### Dijkstra Problems

---

#### 10. Network Delay Time (LC 743)
**Description:** Send a signal from node k to all n nodes. times[i] = [u, v, w] is a directed edge. Return the time it takes for ALL nodes to receive the signal, or -1 if any node is unreachable.

**Example:**
- Input: times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2
- Output: 2  (node 4 receives at time 2, the last to receive)

**Pattern:** Standard Dijkstra
**State:** node
**Answer:** max of dist[] (last node to receive signal)

```java
int networkDelayTime(int[][] times, int n, int k) {
    List<int[]>[] adj = new ArrayList[n];
    for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
    for (int[] edge : times)
        adj[edge[0]-1].add(new int[]{edge[1]-1, edge[2]});

    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k-1] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> Integer.compare(a[0],b[0]));
    pq.offer(new int[]{0, k-1});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int cost = curr[0], node = curr[1];
        if (cost > dist[node]) continue;
        for (int[] edge : adj[node]) {
            int newDist = dist[node] + edge[1];
            if (newDist < dist[edge[0]]) {
                dist[edge[0]] = newDist;
                pq.offer(new int[]{newDist, edge[0]});
            }
        }
    }

    int max = 0;
    for (int d : dist) {
        if (d == Integer.MAX_VALUE) return -1;
        max = Math.max(max, d);
    }
    return max;
}
```

---

#### 11. Cheapest Flights Within K Stops (LC 787)
**Description:** Find cheapest price to fly from src to dst with at most k stops. flights[i] = [from, to, price]. Return -1 if no such route exists.

**Example:**
- Input: n=3, flights=[[0,1,100],[1,2,100],[0,2,500]], src=0, dst=2, k=1
- Output: 200  (0→1→2 costs 200, uses 1 stop)

**Pattern:** Compound state Dijkstra
**State:** (node, stops_used)
**Key:** dist[node][stops], prune when stops >= k

```java
int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    List<int[]>[] adj = new ArrayList[n];
    for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
    for (int[] edge : flights)
        adj[edge[0]].add(new int[]{edge[1], edge[2]});

    int[][] dist = new int[n][k+1];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
    dist[src][0] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> Integer.compare(a[0],b[0]));
    pq.offer(new int[]{0, src, 0});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int cost = curr[0], node = curr[1], stops = curr[2];
        if (node == dst) return cost;
        if (cost > dist[node][stops]) continue;
        if (stops >= k) continue;
        for (int[] edge : adj[node]) {
            int newCost = cost + edge[1];
            if (newCost < dist[edge[0]][stops+1]) {
                dist[edge[0]][stops+1] = newCost;
                pq.offer(new int[]{newCost, edge[0], stops+1});
            }
        }
    }
    return -1;
}
```

---

#### 12. Path With Minimum Effort (LC 1631)
**Description:** Move from top-left to bottom-right of a heights grid. Effort = maximum absolute difference between adjacent cells on the path. Find the minimum possible effort.

**Example:**
- Input: heights = [[1,2,2],[3,8,2],[5,3,5]]
- Output: 2  (path: 1→3→5→3→5, max diff = 2)

**Pattern:** Grid Dijkstra, cost = max diff (not sum)
**State:** (row, col)
**Cost formula:** Math.max(effort, abs(diff))

```java
int minimumEffortPath(int[][] heights) {
    int rows = heights.length, cols = heights[0].length;
    int[][] dist = new int[rows][cols];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
    dist[0][0] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> Integer.compare(a[0],b[0]));
    pq.offer(new int[]{0, 0, 0});
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int effort = curr[0], r = curr[1], c = curr[2];
        if (r == rows-1 && c == cols-1) return effort;
        if (effort > dist[r][c]) continue;
        for (int[] d : dirs) {
            int nr = r+d[0], nc = c+d[1];
            if (nr<0||nr>=rows||nc<0||nc>=cols) continue;
            int newEffort = Math.max(effort, Math.abs(heights[nr][nc]-heights[r][c]));
            if (newEffort < dist[nr][nc]) {
                dist[nr][nc] = newEffort;
                pq.offer(new int[]{newEffort, nr, nc});
            }
        }
    }
    return dist[rows-1][cols-1];
}
```

---

---

#### 13. Course Schedule II (LC 210)
**Description:** Return a valid order to finish all numCourses courses given prerequisites. prerequisites[i] = [a, b] means b before a. Return empty array if impossible (cycle exists).

**Example:**
- Input: numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
- Output: [0,1,2,3] or [0,2,1,3]

**Pattern:** Topological Sort — Kahn's algorithm
**State:** course (integer)
**Key:** in-degree decrement triggers readiness, cycle = order.size() < n

```java
int[] findOrder(int numCourses, int[][] prerequisites) {
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int i = 0; i < numCourses; i++) adj.put(i, new ArrayList<>());
    int[] in = new int[numCourses];

    for (int[] edge : prerequisites) {
        adj.get(edge[1]).add(edge[0]);
        in[edge[0]]++;              // edge[0] depends on edge[1]
    }

    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++)
        if (in[i] == 0) queue.offer(i);

    int[] order = new int[numCourses];
    int idx = 0;

    while (!queue.isEmpty()) {
        int curr = queue.poll();
        order[idx++] = curr;
        for (int next : adj.get(curr)) {
            in[next]--;
            if (in[next] == 0) queue.offer(next);
        }
    }
    return idx == numCourses ? order : new int[0];
}
```

---

#### 14. Alien Dictionary (LC 269)
**Description:** Given words sorted lexicographically in an alien language, derive the ordering of characters. Return any valid ordering, or "" if no valid ordering exists (contradiction or cycle).

**Example:**
- Input: words = ["wrt","wrf","er","ett","rftt"]
- Output: "wertf"  (w<e, r<t, t<f, e<r inferred from adjacent word comparisons)

**Pattern:** Topological Sort — build graph from word comparisons
**Key:** compare adjacent words → first difference = directed edge → Kahn's

```java
String alienOrder(String[] words) {
    Map<Character, List<Character>> adj = new HashMap<>();
    int[] inDegree = new int[26];
    boolean[] exists = new boolean[26];

    for (String word : words)
        for (char c : word.toCharArray()) {
            exists[c - 'a'] = true;
            adj.putIfAbsent(c, new ArrayList<>());
        }

    for (int i = 0; i < words.length - 1; i++) {
        String w1 = words[i], w2 = words[i + 1];
        int minLen = Math.min(w1.length(), w2.length());
        boolean found = false;
        for (int j = 0; j < minLen; j++) {
            if (w1.charAt(j) != w2.charAt(j)) {
                adj.get(w1.charAt(j)).add(w2.charAt(j));
                inDegree[w2.charAt(j) - 'a']++;
                found = true;
                break;
            }
        }
        if (!found && w1.length() > w2.length()) return ""; // invalid prefix
    }

    Queue<Character> queue = new LinkedList<>();
    for (int i = 0; i < 26; i++)
        if (exists[i] && inDegree[i] == 0) queue.offer((char)('a' + i));

    StringBuilder order = new StringBuilder();
    while (!queue.isEmpty()) {
        char curr = queue.poll();
        order.append(curr);
        for (char next : adj.get(curr)) {
            inDegree[next - 'a']--;
            if (inDegree[next - 'a'] == 0) queue.offer(next);
        }
    }

    int totalChars = 0;
    for (boolean e : exists) if (e) totalChars++;
    return order.length() == totalChars ? order.toString() : "";
}
```

---

#### 15. Min Cost to Connect All Points (LC 1584)
**Description:** Given points on a 2D plane, return the minimum cost to connect all points. Cost of connecting two points is their Manhattan distance. (This is a Minimum Spanning Tree problem.)

**Example:**
- Input: points = [[0,0],[2,2],[3,10],[5,2],[7,0]]
- Output: 20

**Pattern:** Prim's MST on implicit graph
**Key:** no adj list given — compute Manhattan distance on the fly, stale check required

```java
int minCostConnectPoints(int[][] points) {
    int n = points.length;
    boolean[] isMST = new boolean[n];
    PriorityQueue<int[]> heap = new PriorityQueue<>(
        (a, b) -> Integer.compare(a[0], b[0])
    );
    heap.offer(new int[]{0, 0});
    int minWeight = 0, nodesAdded = 0;

    while (!heap.isEmpty()) {
        int[] curr = heap.poll();
        int currPoint = curr[1];
        if (isMST[currPoint]) continue;    // stale check — MUST have this
        isMST[currPoint] = true;
        nodesAdded++;
        minWeight += curr[0];
        if (nodesAdded == n) break;
        for (int i = 0; i < n; i++)
            if (!isMST[i])
                heap.offer(new int[]{distance(points[currPoint], points[i]), i});
    }
    return minWeight;
}

int distance(int[] x, int[] y) {
    return Math.abs(x[0]-y[0]) + Math.abs(x[1]-y[1]);
}
```

---

## 6. Bellman-Ford

### When to Use
- Shortest path with **negative weights**
- **Negative cycle detection**
- "At most K edges" shortest path

### Core Idea

```
Relax ALL edges V-1 times.
After round i, dist[v] = shortest path using at most i edges.
After V-1 rounds, all shortest paths are found.
Vth relaxation still updates → negative cycle exists.
```

### Skeleton

```java
int[] bellmanFord(int n, int[][] edges, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // V-1 rounds of relaxation
    for (int i = 0; i < n - 1; i++) {
        for (int[] edge : edges) {
            int u = edge[0], v = edge[1], w = edge[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v])
                dist[v] = dist[u] + w;
        }
    }

    // Vth round — detect negative cycle
    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], w = edge[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v])
            return null; // negative cycle
    }
    return dist;
}
```

### "At Most K Edges" Variant — Snapshot Trick

```java
// Run exactly K rounds, use snapshot to prevent chaining within same round
for (int i = 0; i < k; i++) {
    int[] temp = dist.clone();   // snapshot of previous round
    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], w = edge[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < temp[v])
            temp[v] = dist[u] + w;
    }
    dist = temp;
}
```

### Bellman-Ford vs Dijkstra

```
                Bellman-Ford        Dijkstra
────────────────────────────────────────────────
Negative weights    ✅ YES          ❌ NO
Negative cycles     ✅ Detects      ❌ Breaks
Time                O(V × E)        O((V+E) log V)
Data structure      Arrays only     Min-heap
```

---

## 7. Topological Sort

### When to Use
- Task ordering with dependencies
- Detect cycle in directed graph
- "Can you finish all courses?"
- Build system / package resolution

### Kahn's Algorithm (BFS-based) — Preferred

```
Intuition:
  in-degree = number of prerequisites remaining
  in-degree 0 = ready to process (no blockers)
  After processing node: reduce in-degree of dependents
  If dependent hits 0 → it's now ready

Cycle detection:
  Nodes in a cycle NEVER reach in-degree 0
  → they never enter queue → order.size() < n → cycle!
```

```java
List<Integer> kahnTopoSort(int n, int[][] prerequisites) {
    Map<Integer, List<Integer>> graph = new HashMap<>();
    int[] inDegree = new int[n];
    for (int i = 0; i < n; i++) graph.put(i, new ArrayList<>());

    for (int[] pre : prerequisites) {
        graph.get(pre[1]).add(pre[0]);
        inDegree[pre[0]]++;
    }

    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < n; i++)
        if (inDegree[i] == 0) queue.offer(i);

    List<Integer> order = new ArrayList<>();
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order.add(node);
        for (int neighbor : graph.get(node)) {
            inDegree[neighbor]--;
            if (inDegree[neighbor] == 0) queue.offer(neighbor);
        }
    }
    return order.size() == n ? order : List.of(); // empty = cycle
}
```

### Why in-degree decreases

```
in-degree[v]-- when u is processed means:
"one of v's prerequisites (u) is now complete"

When in-degree[v] hits 0:
"ALL of v's prerequisites are done → v is now available"

Possible values of order.size():
  n       → no cycle, valid topo order
  0       → entire graph is one big cycle (no in-degree 0 node to start)
  1..n-1  → partial cycle, some nodes permanently blocked
```

### DFS-based Topological Sort

```java
// Collect nodes in reverse finish order
void dfs(int node, int[] state, List<Integer> order) {
    state[node] = 1; // GRAY
    for (int neighbor : graph.get(node)) {
        if (state[neighbor] == 1) { hasCycle = true; return; }
        if (state[neighbor] == 0) dfs(neighbor, state, order);
    }
    state[node] = 2; // BLACK
    order.add(node); // add on finish
}
// After all DFS calls: Collections.reverse(order) = topo order
```

---

## 8. Union-Find + MST

### Union-Find

```
Solves: "Are A and B in the same connected component?"
        "How many components exist?"
        "Does adding this edge create a cycle?"

Near O(1) per operation with path compression + union by rank.
```

```java
class UnionFind {
    int[] parent, rank;
    int components;

    UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        components = n;
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // path compression
        return parent[x];
    }

    boolean union(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return false; // already connected
        if (rank[px] < rank[py])      parent[px] = py;
        else if (rank[px] > rank[py]) parent[py] = px;
        else { parent[py] = px; rank[px]++; }
        components--;
        return true;
    }

    boolean connected(int x, int y) { return find(x) == find(y); }
}
```

### Kruskal's MST

```java
int kruskal(int n, int[][] edges) {
    Arrays.sort(edges, (a, b) -> a[0] - b[0]); // sort by weight
    UnionFind uf = new UnionFind(n);
    int mstWeight = 0, edgesUsed = 0;

    for (int[] edge : edges) {
        int w = edge[0], u = edge[1], v = edge[2];
        if (uf.union(u, v)) {           // false = would create cycle
            mstWeight += w;
            if (++edgesUsed == n - 1) break;
        }
    }
    return edgesUsed == n - 1 ? mstWeight : -1;
}
```

### Prim's MST

```java
// Heap-based Prim's — good for sparse graphs
int prims(int n, List<int[]>[] adj) {
    boolean[] inMST = new boolean[n];
    PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> a[0]-b[0]);
    pq.offer(new int[]{0, 0}); // {weight, node}
    int mstWeight = 0, nodesAdded = 0;

    while (!pq.isEmpty() && nodesAdded < n) {
        int[] curr = pq.poll();
        int w = curr[0], node = curr[1];
        if (inMST[node]) continue;      // stale check
        inMST[node] = true;
        mstWeight += w;
        nodesAdded++;
        for (int[] edge : adj[node])
            if (!inMST[edge[0]]) pq.offer(new int[]{edge[1], edge[0]});
    }
    return nodesAdded == n ? mstWeight : -1;
}
```

### Kruskal's vs Prim's

```
                Kruskal's           Prim's
────────────────────────────────────────────────
Approach        Edge-based          Node-based
Data structure  Union-Find          Min-Heap
Sort needed?    Yes (all edges)     No
Time            O(E log E)          O(E log V)
Best for        Sparse graphs       Dense graphs
```

---

## Quick Reference Card

```
┌──────────────┬──────────────────┬─────────────────────────────────┐
│ Algorithm     │ Data Structure   │ Use When                        │
├──────────────┼──────────────────┼─────────────────────────────────┤
│ BFS           │ Queue (FIFO)     │ Shortest path, unweighted       │
│ DFS           │ Stack/Recursion  │ Connectivity, cycles, paths     │
│ Dijkstra      │ Min-Heap (PQ)   │ Shortest path, weighted ≥ 0     │
│ Bellman-Ford  │ Array relaxation │ Shortest path, negative weights │
│ Floyd-Warshall│ 2D DP array     │ All-pairs shortest path         │
│ Union-Find    │ Parent array     │ Online connectivity queries     │
└──────────────┴──────────────────┴─────────────────────────────────┘

Critical rules:
BFS          → mark visited at OFFER time, not poll time
DFS          → visited strategy depends on family (see table above)
Dijkstra     → stale check: if (cost > dist[node]) continue
             → only works with NON-NEGATIVE weights
             → heap entry cost must be FIRST element for ordering
Bellman-Ford → use snapshot (temp = dist.clone()) for K-edge variant
Topo Sort    → in-degree-- when node processed, 0 = ready
             → order.size() < n means cycle exists
Prim's/UF    → stale check at pop: if (inMST[node]) continue
             → union() returns false if already connected (cycle)
```

---

## 10. 2-Week Practice Plan

### Week 1 — Speed (45 min per problem, no hints)

```
Day 1: Word Ladder (127) + Number of Islands (200)
Day 2: Course Schedule II (210) + Rotting Oranges (994)
Day 3: Open the Lock (752) + Number of Provinces (547)
Day 4: Network Delay Time (743) + Cheapest Flights K Stops (787)
Day 5: Min Cost Connect All Points (1584) + Binary Matrix (1091)
Day 6: Alien Dictionary (269) + Genetic Mutation (433)
Day 7: Review all bugs from the week — rewrite 2 problems clean
```

### Week 2 — Hard Problems + Mock Interviews

```
Day 1: Sliding Puzzle (773)          — BFS, board as string state
Day 2: Pacific Atlantic (417)        — DFS from two boundary sets
Day 3: Accounts Merge (721)          — Union-Find or DFS on shared emails
Day 4: Path With Min Effort (1631) + Swim in Rising Water (778)
Day 5: Shortest Path Obstacles (1293) — Dijkstra 3D state (r,c,k)
Day 6: Full mock: 2 problems in 45 min, explain out loud
Day 7: Full mock: 2 problems in 45 min, explain out loud
```

### Staff-Level Checklist Per Problem

```
□ Named which algorithm and WHY before coding
□ Stated complexity (time + space) unprompted
□ Caught edge cases before being asked
  (n==1, empty input, disconnected graph, cycle, negative weights)
□ Mentioned optimization if exists
  (dense→array over heap, bidirectional BFS, path compression)
□ Code is clean and readable — not just correct
□ Explained tradeoffs when multiple approaches exist
```

---

## 11. BFS — 20 Problems

---

### B1. Number of Islands (LC 200) — Medium
**Description:** Count islands in a grid. `1` = land, `0` = water. Connected land cells (4-dir) form one island.
**Example:**
```
Input:  [["1","1","0"],["0","1","0"],["0","0","1"]]
Output: 2
```
```java
public int numIslands(char[][] grid) {
    int rows = grid.length, cols = grid[0].length, count = 0;
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == '1') { bfs(grid, r, c); count++; }
        }
    }
    return count;
}
private void bfs(char[][] grid, int r, int c) {
    int rows = grid.length, cols = grid[0].length;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    Queue<int[]> q = new LinkedList<>();
    grid[r][c] = '0'; q.offer(new int[]{r, c});
    while (!q.isEmpty()) {
        int[] cur = q.poll();
        for (int[] d : dirs) {
            int nr = cur[0]+d[0], nc = cur[1]+d[1];
            if (nr<0||nr>=rows||nc<0||nc>=cols||grid[nr][nc]!='1') continue;
            grid[nr][nc] = '0'; q.offer(new int[]{nr, nc});
        }
    }
}
```
**Key Insight:** Mark cells `'0'` at insert time, not poll time — avoids re-queueing same cell.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### B2. Rotting Oranges (LC 994) — Medium
**Description:** Grid with fresh=1, rotten=2, empty=0. Each minute rotten oranges rot adjacent fresh ones. Return minutes until no fresh remain, or -1.
**Example:**
```
Input:  [[2,1,1],[1,1,0],[0,1,1]]
Output: 4
```
```java
public int orangesRotting(int[][] grid) {
    int rows = grid.length, cols = grid[0].length, fresh = 0, minutes = 0;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    Queue<int[]> q = new LinkedList<>();
    for (int r = 0; r < rows; r++) for (int c = 0; c < cols; c++) {
        if (grid[r][c] == 2) q.offer(new int[]{r, c});
        else if (grid[r][c] == 1) fresh++;
    }
    while (!q.isEmpty() && fresh > 0) {
        int size = q.size(); minutes++;
        for (int i = 0; i < size; i++) {
            int[] cur = q.poll();
            for (int[] d : dirs) {
                int nr = cur[0]+d[0], nc = cur[1]+d[1];
                if (nr<0||nr>=rows||nc<0||nc>=cols||grid[nr][nc]!=1) continue;
                grid[nr][nc] = 2; fresh--; q.offer(new int[]{nr, nc});
            }
        }
    }
    return fresh == 0 ? minutes : -1;
}
```
**Key Insight:** Multi-source BFS — seed ALL rotten oranges before starting. Minutes = levels traversed.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### B3. 01 Matrix (LC 542) — Medium
**Description:** Given binary matrix, return matrix where each cell = distance to nearest 0.
**Example:**
```
Input:  [[0,0,0],[0,1,0],[1,1,1]]
Output: [[0,0,0],[0,1,0],[1,2,1]]
```
```java
public int[][] updateMatrix(int[][] mat) {
    int rows = mat.length, cols = mat[0].length;
    int[][] dist = new int[rows][cols];
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    Queue<int[]> q = new LinkedList<>();
    for (int r = 0; r < rows; r++) for (int c = 0; c < cols; c++) {
        if (mat[r][c] == 0) q.offer(new int[]{r, c});
        else dist[r][c] = Integer.MAX_VALUE;
    }
    while (!q.isEmpty()) {
        int[] cur = q.poll();
        for (int[] d : dirs) {
            int nr = cur[0]+d[0], nc = cur[1]+d[1];
            if (nr<0||nr>=rows||nc<0||nc>=cols) continue;
            if (dist[nr][nc] > dist[cur[0]][cur[1]] + 1) {
                dist[nr][nc] = dist[cur[0]][cur[1]] + 1;
                q.offer(new int[]{nr, nc});
            }
        }
    }
    return dist;
}
```
**Key Insight:** Multi-source BFS from all 0s simultaneously. Each 1 gets distance when first reached = shortest.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### B4. Word Ladder (LC 127) — Hard
**Description:** Transform `beginWord` to `endWord` one character at a time; each intermediate word must be in `wordList`. Return min steps, or 0.
**Example:**
```
Input:  beginWord="hit", endWord="cog", wordList=["hot","dot","dog","lot","log","cog"]
Output: 5  (hit→hot→dot→dog→cog)
```
```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> dict = new HashSet<>(wordList);
    if (!dict.contains(endWord)) return 0;
    Queue<String> q = new LinkedList<>();
    q.offer(beginWord); dict.remove(beginWord);
    int steps = 1;
    while (!q.isEmpty()) {
        int size = q.size(); steps++;
        for (int i = 0; i < size; i++) {
            char[] arr = q.poll().toCharArray();
            for (int j = 0; j < arr.length; j++) {
                char orig = arr[j];
                for (char c = 'a'; c <= 'z'; c++) {
                    arr[j] = c;
                    String next = new String(arr);
                    if (next.equals(endWord)) return steps;
                    if (dict.remove(next)) q.offer(next);
                }
                arr[j] = orig;
            }
        }
    }
    return 0;
}
```
**Key Insight:** BFS on implicit graph. Remove from dict when offered (not polled) to avoid duplicates. Try all 26 chars at each position.
**Complexity:** Time: O(n*m*26) | Space: O(n*m)  n=words, m=word length

---

### B5. Shortest Path in Binary Matrix (LC 1091) — Medium
**Description:** 8-directional grid of 0s and 1s. Find shortest clear path (all 0s) from top-left to bottom-right. Return -1 if none.
**Example:**
```
Input:  [[0,1],[1,0]]
Output: 2
```
```java
public int shortestPathBinaryMatrix(int[][] grid) {
    int n = grid.length;
    if (grid[0][0] == 1 || grid[n-1][n-1] == 1) return -1;
    int[][] dirs = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};
    Queue<int[]> q = new LinkedList<>();
    grid[0][0] = 1; q.offer(new int[]{0, 0, 1});
    while (!q.isEmpty()) {
        int[] cur = q.poll();
        int r = cur[0], c = cur[1], dist = cur[2];
        if (r == n-1 && c == n-1) return dist;
        for (int[] d : dirs) {
            int nr = r+d[0], nc = c+d[1];
            if (nr<0||nr>=n||nc<0||nc>=n||grid[nr][nc]!=0) continue;
            grid[nr][nc] = 1; q.offer(new int[]{nr, nc, dist+1});
        }
    }
    return -1;
}
```
**Key Insight:** 8-directional BFS. Mark visited by setting cell to 1. Check start/end blocked immediately.
**Complexity:** Time: O(n^2) | Space: O(n^2)

---

### B6. Perfect Squares (LC 279) — Medium
**Description:** Find minimum number of perfect squares that sum to `n`.
**Example:**
```
Input:  n = 12
Output: 3  (4+4+4)
```
```java
public int numSquares(int n) {
    Queue<Integer> q = new LinkedList<>();
    boolean[] visited = new boolean[n + 1];
    q.offer(0); visited[0] = true;
    int steps = 0;
    while (!q.isEmpty()) {
        int size = q.size(); steps++;
        for (int i = 0; i < size; i++) {
            int curr = q.poll();
            for (int j = 1; j * j <= n; j++) {
                int next = curr + j * j;
                if (next == n) return steps;
                if (next < n && !visited[next]) {
                    visited[next] = true; q.offer(next);
                }
            }
        }
    }
    return steps;
}
```
**Key Insight:** BFS on numbers — nodes are integers 0..n, edges add perfect squares. First time reaching n = min steps.
**Complexity:** Time: O(n*sqrt(n)) | Space: O(n)

---

### B7. Open the Lock (LC 752) — Medium
**Description:** Start at `"0000"`. Each step rotates one dial ±1 (wraps). `deadends` are blocked states. Find min turns to reach `target`.
**Example:**
```
Input:  deadends=["0201","0101","0102","1212","2002"], target="0202"
Output: 6
```
```java
public int openLock(String[] deadends, String target) {
    Set<String> dead = new HashSet<>(Arrays.asList(deadends));
    if (dead.contains("0000")) return -1;
    Queue<String> q = new LinkedList<>();
    Set<String> visited = new HashSet<>();
    q.offer("0000"); visited.add("0000");
    int steps = 0;
    while (!q.isEmpty()) {
        int size = q.size(); steps++;
        for (int i = 0; i < size; i++) {
            char[] arr = q.poll().toCharArray();
            for (int j = 0; j < 4; j++) {
                for (int d : new int[]{1, -1}) {
                    char orig = arr[j];
                    arr[j] = (char)('0' + (arr[j] - '0' + d + 10) % 10);
                    String next = new String(arr);
                    if (next.equals(target)) return steps;
                    if (!dead.contains(next) && visited.add(next)) q.offer(next);
                    arr[j] = orig;
                }
            }
        }
    }
    return -1;
}
```
**Key Insight:** State = 4-digit string. Neighbors = 8 states (4 dials × 2 directions). Standard BFS on state space.
**Complexity:** Time: O(10^4 * 8) | Space: O(10^4)

---

### B8. Walls and Gates (LC 286) — Medium (Premium)
**Description:** Grid with INF=empty room, -1=wall, 0=gate. Fill each empty room with distance to nearest gate.
**Example:**
```
Input:  [[INF,-1,0,INF],[INF,INF,INF,-1],[INF,-1,INF,-1],[0,-1,INF,INF]]
Output: [[3,-1,0,1],[2,2,1,-1],[1,-1,2,-1],[0,-1,3,4]]
```
```java
public void wallsAndGates(int[][] rooms) {
    int rows = rooms.length, cols = rooms[0].length;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    Queue<int[]> q = new LinkedList<>();
    for (int r = 0; r < rows; r++) for (int c = 0; c < cols; c++)
        if (rooms[r][c] == 0) q.offer(new int[]{r, c});
    while (!q.isEmpty()) {
        int[] cur = q.poll();
        for (int[] d : dirs) {
            int nr = cur[0]+d[0], nc = cur[1]+d[1];
            if (nr<0||nr>=rows||nc<0||nc>=cols||rooms[nr][nc]!=Integer.MAX_VALUE) continue;
            rooms[nr][nc] = rooms[cur[0]][cur[1]] + 1;
            q.offer(new int[]{nr, nc});
        }
    }
}
```
**Key Insight:** Multi-source BFS from all gates. Each room gets filled exactly once — when first reached = shortest distance.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### B9. As Far from Land as Possible (LC 1162) — Medium
**Description:** Given binary grid (0=water, 1=land), find water cell maximally far from any land cell. Return -1 if no water/land.
**Example:**
```
Input:  [[1,0,1],[0,0,0],[1,0,1]]
Output: 2
```
```java
public int maxDistance(int[][] grid) {
    int n = grid.length;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    Queue<int[]> q = new LinkedList<>();
    for (int r = 0; r < n; r++) for (int c = 0; c < n; c++)
        if (grid[r][c] == 1) q.offer(new int[]{r, c});
    if (q.isEmpty() || q.size() == n * n) return -1;
    int dist = -1;
    while (!q.isEmpty()) {
        int size = q.size(); dist++;
        for (int i = 0; i < size; i++) {
            int[] cur = q.poll();
            for (int[] d : dirs) {
                int nr = cur[0]+d[0], nc = cur[1]+d[1];
                if (nr<0||nr>=n||nc<0||nc>=n||grid[nr][nc]!=0) continue;
                grid[nr][nc] = 1; q.offer(new int[]{nr, nc});
            }
        }
    }
    return dist;
}
```
**Key Insight:** Multi-source BFS from land. The last water cell reached = farthest. Answer = levels - 1.
**Complexity:** Time: O(n^2) | Space: O(n^2)

---

### B10. Nearest Exit from Entrance in Maze (LC 1926) — Medium
**Description:** Grid with `.`=open, `+`=wall. Find min steps from `entrance` to any border cell (not entrance itself).
**Example:**
```
Input:  maze=[["+","+",".","+"],[".",".",".","+"],["+"," +","+","."]], entrance=[1,2]
Output: 1
```
```java
public int nearestExit(char[][] maze, int[] entrance) {
    int rows = maze.length, cols = maze[0].length;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    Queue<int[]> q = new LinkedList<>();
    maze[entrance[0]][entrance[1]] = '+';
    q.offer(new int[]{entrance[0], entrance[1], 0});
    while (!q.isEmpty()) {
        int[] cur = q.poll();
        int r = cur[0], c = cur[1], steps = cur[2];
        for (int[] d : dirs) {
            int nr = r+d[0], nc = c+d[1];
            if (nr<0||nr>=rows||nc<0||nc>=cols||maze[nr][nc]=='+') continue;
            if (nr==0||nr==rows-1||nc==0||nc==cols-1) return steps+1;
            maze[nr][nc] = '+'; q.offer(new int[]{nr, nc, steps+1});
        }
    }
    return -1;
}
```
**Key Insight:** Mark entrance as wall to block return. Check border condition before marking to find exit.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### B11. Binary Tree Level Order Traversal (LC 102) — Medium
**Description:** Return nodes level by level, each level as a sublist.
**Example:**
```
Input:  [3,9,20,null,null,15,7]
Output: [[3],[9,20],[15,7]]
```
```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = q.poll();
            level.add(node.val);
            if (node.left != null) q.offer(node.left);
            if (node.right != null) q.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```
**Key Insight:** `size = q.size()` snapshot before processing each level — separates levels cleanly.
**Complexity:** Time: O(n) | Space: O(n)

---

### B12. Binary Tree Right Side View (LC 199) — Medium
**Description:** Return values visible when looking from the right side (last node at each level).
**Example:**
```
Input:  [1,2,3,null,5,null,4]
Output: [1,3,4]
```
```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            TreeNode node = q.poll();
            if (i == size - 1) result.add(node.val);
            if (node.left != null) q.offer(node.left);
            if (node.right != null) q.offer(node.right);
        }
    }
    return result;
}
```
**Key Insight:** Level order BFS; capture the last node (`i == size - 1`) at each level.
**Complexity:** Time: O(n) | Space: O(n)

---

### B13. Binary Tree Zigzag Level Order (LC 103) — Medium
**Description:** Level order but alternate left-to-right and right-to-left each level.
**Example:**
```
Input:  [3,9,20,null,null,15,7]
Output: [[3],[20,9],[15,7]]
```
```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root); boolean leftToRight = true;
    while (!q.isEmpty()) {
        int size = q.size();
        LinkedList<Integer> level = new LinkedList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = q.poll();
            if (leftToRight) level.addLast(node.val);
            else level.addFirst(node.val);
            if (node.left != null) q.offer(node.left);
            if (node.right != null) q.offer(node.right);
        }
        result.add(level); leftToRight = !leftToRight;
    }
    return result;
}
```
**Key Insight:** Use `LinkedList` as deque — `addFirst` for right-to-left direction. Toggle direction flag per level.
**Complexity:** Time: O(n) | Space: O(n)

---

### B14. All Nodes Distance K in Binary Tree (LC 863) — Medium
**Description:** Return all nodes exactly `k` distance from target node.
**Example:**
```
Input:  root=[3,5,1,6,2,0,8,null,null,7,4], target=5, k=2
Output: [7,4,1]
```
```java
public List<Integer> distanceK(TreeNode root, TreeNode target, int k) {
    Map<Integer, List<Integer>> graph = new HashMap<>();
    buildGraph(root, null, graph);
    List<Integer> result = new ArrayList<>();
    Queue<Integer> q = new LinkedList<>();
    Set<Integer> visited = new HashSet<>();
    q.offer(target.val); visited.add(target.val);
    int dist = 0;
    while (!q.isEmpty()) {
        if (dist == k) { result.addAll(q); return result; }
        int size = q.size(); dist++;
        for (int i = 0; i < size; i++) {
            int node = q.poll();
            for (int neighbor : graph.getOrDefault(node, List.of())) {
                if (visited.add(neighbor)) q.offer(neighbor);
            }
        }
    }
    return result;
}
private void buildGraph(TreeNode node, TreeNode parent, Map<Integer, List<Integer>> graph) {
    if (node == null) return;
    graph.computeIfAbsent(node.val, k -> new ArrayList<>());
    if (parent != null) {
        graph.get(node.val).add(parent.val);
        graph.get(parent.val).add(node.val);
    }
    buildGraph(node.left, node, graph);
    buildGraph(node.right, node, graph);
}
```
**Key Insight:** Convert tree to undirected graph (add parent edges). Then standard BFS for distance k.
**Complexity:** Time: O(n) | Space: O(n)

---

### B15. Minimum Height Trees (LC 310) — Medium
**Description:** Find all roots that produce minimum height trees in a graph of n nodes.
**Example:**
```
Input:  n=6, edges=[[3,0],[3,1],[3,2],[3,4],[5,4]]
Output: [3,4]
```
```java
public List<Integer> findMinHeightTrees(int n, int[][] edges) {
    if (n == 1) return List.of(0);
    List<Set<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new HashSet<>());
    for (int[] e : edges) { adj.get(e[0]).add(e[1]); adj.get(e[1]).add(e[0]); }
    Queue<Integer> leaves = new LinkedList<>();
    for (int i = 0; i < n; i++) if (adj.get(i).size() == 1) leaves.offer(i);
    int remaining = n;
    while (remaining > 2) {
        int size = leaves.size(); remaining -= size;
        for (int i = 0; i < size; i++) {
            int leaf = leaves.poll();
            int neighbor = adj.get(leaf).iterator().next();
            adj.get(neighbor).remove(leaf);
            if (adj.get(neighbor).size() == 1) leaves.offer(neighbor);
        }
    }
    return new ArrayList<>(leaves);
}
```
**Key Insight:** Topological trim from leaves inward. Last 1-2 nodes remaining = MHT roots (centroid of tree).
**Complexity:** Time: O(n) | Space: O(n)

---

### B16. Snakes and Ladders (LC 909) — Medium
**Description:** n×n board numbered 1..n² in boustrophedon order. Snakes/ladders teleport you. Min dice rolls to reach n².
**Example:**
```
Input:  board=[[-1,-1,-1,-1,-1,-1],[-1,-1,-1,-1,-1,-1],[-1,-1,-1,-1,-1,-1],[-1,35,-1,-1,13,-1],[-1,-1,-1,-1,-1,-1],[-1,15,-1,-1,-1,-1]]
Output: 4
```
```java
public int snakesAndLadders(int[][] board) {
    int n = board.length;
    boolean[] visited = new boolean[n * n + 1];
    Queue<int[]> q = new LinkedList<>();
    q.offer(new int[]{1, 0}); visited[1] = true;
    while (!q.isEmpty()) {
        int[] cur = q.poll(); int cell = cur[0], moves = cur[1];
        for (int dice = 1; dice <= 6; dice++) {
            int next = cell + dice;
            if (next > n * n) break;
            int[] pos = getPos(next, n);
            if (board[pos[0]][pos[1]] != -1) next = board[pos[0]][pos[1]];
            if (next == n * n) return moves + 1;
            if (!visited[next]) { visited[next] = true; q.offer(new int[]{next, moves+1}); }
        }
    }
    return -1;
}
private int[] getPos(int num, int n) {
    int row = (num - 1) / n, col = (num - 1) % n;
    if (row % 2 == 1) col = n - 1 - col;
    return new int[]{n - 1 - row, col};
}
```
**Key Insight:** BFS on cell numbers 1..n². `getPos` converts cell# to board coordinates with boustrophedon logic.
**Complexity:** Time: O(n^2) | Space: O(n^2)

---

### B17. Minimum Genetic Mutation (LC 433) — Medium
**Description:** Mutate gene string one character at a time (A/C/G/T) to reach `endGene`. Each intermediate must be in bank. Min mutations?
**Example:**
```
Input:  startGene="AACCGGTT", endGene="AAACGGTA", bank=["AACCGGTA","AACCGCTA","AAACGGTA"]
Output: 2
```
```java
public int minMutation(String startGene, String endGene, String[] bank) {
    Set<String> bankSet = new HashSet<>(Arrays.asList(bank));
    if (!bankSet.contains(endGene)) return -1;
    Queue<String> q = new LinkedList<>();
    q.offer(startGene); bankSet.remove(startGene);
    int steps = 0;
    char[] genes = {'A', 'C', 'G', 'T'};
    while (!q.isEmpty()) {
        int size = q.size(); steps++;
        for (int i = 0; i < size; i++) {
            char[] arr = q.poll().toCharArray();
            for (int j = 0; j < arr.length; j++) {
                char orig = arr[j];
                for (char g : genes) {
                    arr[j] = g;
                    String next = new String(arr);
                    if (next.equals(endGene)) return steps;
                    if (bankSet.remove(next)) q.offer(next);
                }
                arr[j] = orig;
            }
        }
    }
    return -1;
}
```
**Key Insight:** Same pattern as Word Ladder but with 4-char alphabet. Remove from bank on offer to avoid revisits.
**Complexity:** Time: O(B*L*4)  B=bank size, L=gene length | Space: O(B)

---

### B18. Time Needed to Inform All Employees (LC 1376) — Medium
**Description:** Manager `headID` sends message; each employee notifies their subordinates in `informTime[i]` minutes. Find total time.
**Example:**
```
Input:  n=6, headID=2, manager=[-1,2,5,2,2,2], informTime=[0,0,1,0,0,2]
Output: 3
```
```java
public int numOfMinutes(int n, int headID, int[] manager, int[] informTime) {
    Map<Integer, List<Integer>> subordinates = new HashMap<>();
    for (int i = 0; i < n; i++) {
        if (manager[i] != -1)
            subordinates.computeIfAbsent(manager[i], k -> new ArrayList<>()).add(i);
    }
    Queue<int[]> q = new LinkedList<>(); // {employee, time_reached}
    q.offer(new int[]{headID, 0});
    int maxTime = 0;
    while (!q.isEmpty()) {
        int[] cur = q.poll(); int emp = cur[0], time = cur[1];
        maxTime = Math.max(maxTime, time);
        for (int sub : subordinates.getOrDefault(emp, List.of()))
            q.offer(new int[]{sub, time + informTime[emp]});
    }
    return maxTime;
}
```
**Key Insight:** BFS propagating accumulated time. Each node tracks absolute time it was notified, not relative delay.
**Complexity:** Time: O(n) | Space: O(n)

---

### B19. Shortest Path with Alternating Colors (LC 1129) — Medium
**Description:** Directed graph with red and blue edges. Find shortest path from node 0 to each node using alternating colors. -1 if unreachable.
**Example:**
```
Input:  n=3, redEdges=[[0,1],[1,2]], blueEdges=[]
Output: [0,1,-1]
```
```java
public int[] shortestAlternatingPaths(int n, int[][] redEdges, int[][] blueEdges) {
    List<int[]>[] adj = new ArrayList[n]; // {neighbor, color}
    for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
    for (int[] e : redEdges) adj[e[0]].add(new int[]{e[1], 0});
    for (int[] e : blueEdges) adj[e[0]].add(new int[]{e[1], 1});
    int[] ans = new int[n]; Arrays.fill(ans, -1);
    boolean[][] visited = new boolean[n][2]; // [node][color of edge used to arrive]
    Queue<int[]> q = new LinkedList<>(); // {node, last_color, dist}
    q.offer(new int[]{0, 0, 0}); q.offer(new int[]{0, 1, 0});
    visited[0][0] = visited[0][1] = true; ans[0] = 0;
    while (!q.isEmpty()) {
        int[] cur = q.poll(); int node = cur[0], color = cur[1], dist = cur[2];
        for (int[] edge : adj[node]) {
            int next = edge[0], edgeColor = edge[1];
            if (edgeColor == color || visited[next][edgeColor]) continue;
            visited[next][edgeColor] = true;
            if (ans[next] == -1) ans[next] = dist + 1;
            q.offer(new int[]{next, edgeColor, dist + 1});
        }
    }
    return ans;
}
```
**Key Insight:** State = `(node, last_edge_color)`. Must alternate, so visited is 2D: `[node][color_used_to_arrive]`.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### B20. Jump Game IV (LC 1345) — Hard
**Description:** Array of integers. From index `i` you can jump to `i±1` or any index `j` where `arr[i]==arr[j]`. Min jumps to reach last index.
**Example:**
```
Input:  arr=[100,-23,-23,404,100,23,23,23,3,404]
Output: 3
```
```java
public int minJumps(int[] arr) {
    int n = arr.length;
    Map<Integer, List<Integer>> sameVal = new HashMap<>();
    for (int i = 0; i < n; i++) sameVal.computeIfAbsent(arr[i], k -> new ArrayList<>()).add(i);
    boolean[] visited = new boolean[n];
    Queue<Integer> q = new LinkedList<>();
    q.offer(0); visited[0] = true;
    int steps = 0;
    while (!q.isEmpty()) {
        int size = q.size(); steps++;
        for (int i = 0; i < size; i++) {
            int idx = q.poll();
            for (int next : new int[]{idx-1, idx+1}) {
                if (next >= 0 && next < n && !visited[next]) {
                    if (next == n-1) return steps;
                    visited[next] = true; q.offer(next);
                }
            }
            List<Integer> same = sameVal.remove(arr[idx]); // remove after using!
            if (same != null) for (int next : same) {
                if (!visited[next]) {
                    if (next == n-1) return steps;
                    visited[next] = true; q.offer(next);
                }
            }
        }
    }
    return -1;
}
```
**Key Insight:** `sameVal.remove(arr[idx])` — remove the group after visiting once. Without this, each node re-processes the entire group → O(n²).
**Complexity:** Time: O(n) | Space: O(n)


---

## 12. DFS — 20 Problems

---

### D1. Max Area of Island (LC 695) — Medium
**Description:** Return the maximum area of an island (connected 1s) in a binary grid.
**Example:**
```
Input:  [[0,0,1,0],[0,1,1,0],[0,0,0,1]]
Output: 3
```
```java
public int maxAreaOfIsland(int[][] grid) {
    int rows = grid.length, cols = grid[0].length, max = 0;
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            if (grid[r][c] == 1) max = Math.max(max, dfs(grid, r, c));
    return max;
}
private int dfs(int[][] grid, int r, int c) {
    if (r<0||r>=grid.length||c<0||c>=grid[0].length||grid[r][c]!=1) return 0;
    grid[r][c] = 0;
    return 1 + dfs(grid,r+1,c) + dfs(grid,r-1,c) + dfs(grid,r,c+1) + dfs(grid,r,c-1);
}
```
**Key Insight:** DFS returns area of current island. Mark visited by setting to 0. Sum = 1 (current) + 4 recursive calls.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### D2. Surrounded Regions (LC 130) — Medium
**Description:** Flip all `'O'`s not connected to the border to `'X'`.
**Example:**
```
Input:  [["X","X","X"],["X","O","X"],["X","X","X"]]
Output: [["X","X","X"],["X","X","X"],["X","X","X"]]
```
```java
public void solve(char[][] board) {
    int rows = board.length, cols = board[0].length;
    // Mark border-connected O's as safe
    for (int r = 0; r < rows; r++) { dfs(board, r, 0); dfs(board, r, cols-1); }
    for (int c = 0; c < cols; c++) { dfs(board, 0, c); dfs(board, rows-1, c); }
    for (int r = 0; r < rows; r++) for (int c = 0; c < cols; c++) {
        if (board[r][c] == 'O') board[r][c] = 'X';
        else if (board[r][c] == 'S') board[r][c] = 'O';
    }
}
private void dfs(char[][] board, int r, int c) {
    if (r<0||r>=board.length||c<0||c>=board[0].length||board[r][c]!='O') return;
    board[r][c] = 'S';
    dfs(board,r+1,c); dfs(board,r-1,c); dfs(board,r,c+1); dfs(board,r,c-1);
}
```
**Key Insight:** 3-pass: mark border O's as `'S'`, flip remaining O's to X, restore S back to O.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### D3. Pacific Atlantic Water Flow (LC 417) — Medium
**Description:** Rain water flows to adjacent cells with equal/lower height. Find cells that can flow to both Pacific (top/left) and Atlantic (bottom/right).
**Example:**
```
Input:  [[1,2,2,3,5],[3,2,3,4,4],[2,4,5,3,1],[6,7,1,4,5],[5,1,1,2,4]]
Output: [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]
```
```java
public List<List<Integer>> pacificAtlantic(int[][] heights) {
    int rows = heights.length, cols = heights[0].length;
    boolean[][] pac = new boolean[rows][cols], atl = new boolean[rows][cols];
    for (int r = 0; r < rows; r++) { dfs(heights,r,0,pac); dfs(heights,r,cols-1,atl); }
    for (int c = 0; c < cols; c++) { dfs(heights,0,c,pac); dfs(heights,rows-1,c,atl); }
    List<List<Integer>> res = new ArrayList<>();
    for (int r = 0; r < rows; r++) for (int c = 0; c < cols; c++)
        if (pac[r][c] && atl[r][c]) res.add(Arrays.asList(r, c));
    return res;
}
private void dfs(int[][] h, int r, int c, boolean[][] visited) {
    visited[r][c] = true;
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    for (int[] d : dirs) {
        int nr=r+d[0], nc=c+d[1];
        if (nr<0||nr>=h.length||nc<0||nc>=h[0].length||visited[nr][nc]||h[nr][nc]<h[r][c]) continue;
        dfs(h, nr, nc, visited);
    }
}
```
**Key Insight:** Reverse flow — DFS uphill from ocean borders. Cells reachable from both oceans = answer. Avoids complex downstream tracking.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### D4. Longest Increasing Path in Matrix (LC 329) — Hard
**Description:** Find the length of the longest strictly increasing path in a matrix (4-directional).
**Example:**
```
Input:  [[9,9,4],[6,6,8],[2,1,1]]
Output: 4  (1→2→6→9)
```
```java
int[][] memo;
public int longestIncreasingPath(int[][] matrix) {
    int rows = matrix.length, cols = matrix[0].length, max = 0;
    memo = new int[rows][cols];
    for (int r = 0; r < rows; r++) for (int c = 0; c < cols; c++)
        max = Math.max(max, dfs(matrix, r, c));
    return max;
}
private int dfs(int[][] m, int r, int c) {
    if (memo[r][c] != 0) return memo[r][c];
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    memo[r][c] = 1;
    for (int[] d : dirs) {
        int nr=r+d[0], nc=c+d[1];
        if (nr<0||nr>=m.length||nc<0||nc>=m[0].length||m[nr][nc]<=m[r][c]) continue;
        memo[r][c] = Math.max(memo[r][c], 1 + dfs(m, nr, nc));
    }
    return memo[r][c];
}
```
**Key Insight:** DFS with memoization. No visited array needed — strict increase prevents cycles. Each cell computed once.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### D5. All Paths From Source to Target (LC 797) — Medium
**Description:** Find all paths from node 0 to node n-1 in a DAG.
**Example:**
```
Input:  graph=[[1,2],[3],[3],[]]
Output: [[0,1,3],[0,2,3]]
```
```java
public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
    List<List<Integer>> result = new ArrayList<>();
    dfs(graph, 0, new ArrayList<>(Arrays.asList(0)), result);
    return result;
}
private void dfs(int[][] graph, int node, List<Integer> path, List<List<Integer>> result) {
    if (node == graph.length - 1) { result.add(new ArrayList<>(path)); return; }
    for (int next : graph[node]) {
        path.add(next);
        dfs(graph, next, path, result);
        path.remove(path.size() - 1);
    }
}
```
**Key Insight:** DAG → no cycle → no visited needed. Backtrack by removing last element after each recursive call.
**Complexity:** Time: O(2^n * n) | Space: O(n)

---

### D6. Path Sum II (LC 113) — Medium
**Description:** Find all root-to-leaf paths where the sum equals `targetSum`.
**Example:**
```
Input:  root=[5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum=22
Output: [[5,4,11,2],[5,8,4,5]]
```
```java
public List<List<Integer>> pathSum(TreeNode root, int target) {
    List<List<Integer>> result = new ArrayList<>();
    dfs(root, target, new ArrayList<>(), result);
    return result;
}
private void dfs(TreeNode node, int rem, List<Integer> path, List<List<Integer>> result) {
    if (node == null) return;
    path.add(node.val);
    if (node.left == null && node.right == null && rem == node.val)
        result.add(new ArrayList<>(path));
    dfs(node.left, rem - node.val, path, result);
    dfs(node.right, rem - node.val, path, result);
    path.remove(path.size() - 1);
}
```
**Key Insight:** Add node before recursing, remove after (backtracking). Check leaf condition AND target match.
**Complexity:** Time: O(n^2) worst | Space: O(n)

---

### D7. Binary Tree Maximum Path Sum (LC 124) — Hard
**Description:** Find max path sum in binary tree. Path can start/end at any node.
**Example:**
```
Input:  root=[-10,9,20,null,null,15,7]
Output: 42  (15→20→7)
```
```java
int maxSum;
public int maxPathSum(TreeNode root) {
    maxSum = Integer.MIN_VALUE;
    gain(root);
    return maxSum;
}
private int gain(TreeNode node) {
    if (node == null) return 0;
    int leftGain = Math.max(0, gain(node.left));  // ignore negative paths
    int rightGain = Math.max(0, gain(node.right));
    maxSum = Math.max(maxSum, node.val + leftGain + rightGain); // path through node
    return node.val + Math.max(leftGain, rightGain); // return only one branch upward
}
```
**Key Insight:** Each node contributes to two decisions: (1) max path THROUGH it (update global max), (2) max path RETURNING to parent (return one side only).
**Complexity:** Time: O(n) | Space: O(h)  h=tree height

---

### D8. Diameter of Binary Tree (LC 543) — Easy
**Description:** Length of the longest path between any two nodes (may not pass through root).
**Example:**
```
Input:  root=[1,2,3,4,5]
Output: 3  (4→2→1→3 or 5→2→1→3)
```
```java
int diameter = 0;
public int diameterOfBinaryTree(TreeNode root) {
    depth(root);
    return diameter;
}
private int depth(TreeNode node) {
    if (node == null) return 0;
    int left = depth(node.left), right = depth(node.right);
    diameter = Math.max(diameter, left + right);
    return 1 + Math.max(left, right);
}
```
**Key Insight:** At each node, `left + right` = path through that node. Global max across all nodes = diameter. Return only `1 + max(left, right)` upward.
**Complexity:** Time: O(n) | Space: O(h)

---

### D9. Lowest Common Ancestor (LC 236) — Medium
**Description:** Find LCA of two nodes p and q in a binary tree.
**Example:**
```
Input:  root=[3,5,1,6,2,0,8,null,null,7,4], p=5, q=1
Output: 3
```
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) return root; // p and q on different sides
    return left != null ? left : right; // both on same side
}
```
**Key Insight:** If both children return non-null, current node = LCA. If one side is null, propagate the non-null result upward.
**Complexity:** Time: O(n) | Space: O(h)

---

### D10. Is Graph Bipartite? (LC 785) — Medium
**Description:** Color graph with 2 colors such that no two adjacent nodes share the same color.
**Example:**
```
Input:  graph=[[1,3],[0,2],[1,3],[0,2]]
Output: true
```
```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n]; Arrays.fill(color, -1);
    for (int i = 0; i < n; i++) {
        if (color[i] == -1 && !dfs(graph, i, 0, color)) return false;
    }
    return true;
}
private boolean dfs(int[][] graph, int node, int c, int[] color) {
    color[node] = c;
    for (int neighbor : graph[node]) {
        if (color[neighbor] == c) return false;
        if (color[neighbor] == -1 && !dfs(graph, neighbor, 1 - c, color)) return false;
    }
    return true;
}
```
**Key Insight:** Try 2-coloring via DFS. If any adjacent nodes get the same color → not bipartite. `1 - c` alternates colors.
**Complexity:** Time: O(V+E) | Space: O(V)

---

### D11. Keys and Rooms (LC 841) — Medium
**Description:** n rooms, room 0 is unlocked. Each room has keys to other rooms. Can you visit all rooms?
**Example:**
```
Input:  rooms=[[1],[2],[3],[]]
Output: true
```
```java
public boolean canVisitAllRooms(List<List<Integer>> rooms) {
    boolean[] visited = new boolean[rooms.size()];
    dfs(rooms, 0, visited);
    for (boolean v : visited) if (!v) return false;
    return true;
}
private void dfs(List<List<Integer>> rooms, int room, boolean[] visited) {
    visited[room] = true;
    for (int key : rooms.get(room))
        if (!visited[key]) dfs(rooms, key, visited);
}
```

---

### D12. Number of Provinces (LC 547) — Medium
**Description:** n cities, `isConnected[i][j]=1` means cities i,j are directly connected. Find number of provinces (connected components).
**Example:**
```
Input:  isConnected=[[1,1,0],[1,1,0],[0,0,1]]
Output: 2
```
```java
public int findCircleNum(int[][] isConnected) {
    int n = isConnected.length, count = 0;
    boolean[] visited = new boolean[n];
    for (int i = 0; i < n; i++) {
        if (!visited[i]) { dfs(isConnected, i, visited); count++; }
    }
    return count;
}
private void dfs(int[][] graph, int city, boolean[] visited) {
    visited[city] = true;
    for (int j = 0; j < graph.length; j++)
        if (graph[city][j] == 1 && !visited[j]) dfs(graph, j, visited);
}
```

---

### D13. Reorder Routes to Make All Paths Lead to City Zero (LC 1466) — Medium
**Description:** Directed graph where all roads go away from city 0. Find minimum edges to reverse so all cities can reach city 0.
**Example:**
```
Input:  n=6, connections=[[0,1],[1,3],[2,3],[4,0],[4,5]]
Output: 3
```
```java
public int minReorder(int n, int[][] connections) {
    Map<Integer, List<int[]>> adj = new HashMap<>(); // {neighbor, isOriginalDirection}
    for (int[] c : connections) {
        adj.computeIfAbsent(c[0], k->new ArrayList<>()).add(new int[]{c[1], 1}); // original
        adj.computeIfAbsent(c[1], k->new ArrayList<>()).add(new int[]{c[0], 0}); // reverse
    }
    boolean[] visited = new boolean[n];
    return dfs(adj, 0, visited);
}
private int dfs(Map<Integer, List<int[]>> adj, int node, boolean[] visited) {
    visited[node] = true; int count = 0;
    for (int[] edge : adj.getOrDefault(node, List.of())) {
        if (!visited[edge[0]]) count += edge[1] + dfs(adj, edge[0], visited);
    }
    return count;
}
```
**Key Insight:** Build undirected graph, tag original direction (1) vs reverse (0). DFS from 0 — original-direction edges away from 0 need reversal.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### D14. Find if Path Exists in Graph (LC 1971) — Easy
**Description:** Undirected graph, return true if path exists from `source` to `destination`.
**Example:**
```
Input:  n=3, edges=[[0,1],[1,2],[2,0]], source=0, destination=2
Output: true
```
```java
public boolean validPath(int n, int[][] edges, int source, int destination) {
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] e : edges) {
        adj.computeIfAbsent(e[0], k->new ArrayList<>()).add(e[1]);
        adj.computeIfAbsent(e[1], k->new ArrayList<>()).add(e[0]);
    }
    boolean[] visited = new boolean[n];
    return dfs(adj, source, destination, visited);
}
private boolean dfs(Map<Integer, List<Integer>> adj, int node, int dest, boolean[] visited) {
    if (node == dest) return true;
    visited[node] = true;
    for (int next : adj.getOrDefault(node, List.of()))
        if (!visited[next] && dfs(adj, next, dest, visited)) return true;
    return false;
}
```

---

### D15. Count Distinct Islands (LC 694) — Medium (Premium)
**Description:** Count distinct island shapes (normalized by translation).
**Example:**
```
Input:  [[1,1,0,1,1],[1,0,0,0,0],[0,0,0,0,1],[1,1,0,1,1]]
Output: 3
```
```java
public int numDistinctIslands(int[][] grid) {
    Set<String> shapes = new HashSet<>();
    for (int r = 0; r < grid.length; r++) for (int c = 0; c < grid[0].length; c++) {
        if (grid[r][c] == 1) {
            StringBuilder path = new StringBuilder();
            dfs(grid, r, c, r, c, path);
            shapes.add(path.toString());
        }
    }
    return shapes.size();
}
private void dfs(int[][] grid, int r, int c, int baseR, int baseC, StringBuilder path) {
    if (r<0||r>=grid.length||c<0||c>=grid[0].length||grid[r][c]!=1) return;
    grid[r][c] = 0;
    path.append((r-baseR)+","+( c-baseC)+" ");
    dfs(grid,r+1,c,baseR,baseC,path); dfs(grid,r-1,c,baseR,baseC,path);
    dfs(grid,r,c+1,baseR,baseC,path); dfs(grid,r,c-1,baseR,baseC,path);
}
```
**Key Insight:** Normalize shape by recording relative offsets `(r-baseR, c-baseC)`. Same offsets = same shape.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### D16. Path Sum III (LC 437) — Medium
**Description:** Count paths in binary tree that sum to `targetSum`. Path must go downward but doesn't need to start/end at root/leaf.
**Example:**
```
Input:  root=[10,5,-3,3,2,null,11,3,-2,null,1], targetSum=8
Output: 3
```
```java
public int pathSum(TreeNode root, int targetSum) {
    Map<Long, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0L, 1);
    return dfs(root, 0L, targetSum, prefixCount);
}
private int dfs(TreeNode node, long currSum, int target, Map<Long, Integer> prefixCount) {
    if (node == null) return 0;
    currSum += node.val;
    int count = prefixCount.getOrDefault(currSum - target, 0);
    prefixCount.merge(currSum, 1, Integer::sum);
    count += dfs(node.left, currSum, target, prefixCount);
    count += dfs(node.right, currSum, target, prefixCount);
    prefixCount.merge(currSum, -1, Integer::sum); // backtrack
    return count;
}
```
**Key Insight:** Prefix sum on tree path. `currSum - target` in map = path ending at current node sums to target. Must backtrack the map.
**Complexity:** Time: O(n) | Space: O(n)

---

### D17. Reconstruct Itinerary (LC 332) — Hard
**Description:** Given airline tickets as `[from, to]`, reconstruct itinerary using all tickets starting from `JFK`. Lexicographically smallest.
**Example:**
```
Input:  tickets=[["MUC","LHR"],["JFK","MUC"],["SFO","SJC"],["LHR","SFO"]]
Output: ["JFK","MUC","LHR","SFO","SJC"]
```
```java
public List<String> findItinerary(List<List<String>> tickets) {
    Map<String, PriorityQueue<String>> adj = new HashMap<>();
    for (List<String> t : tickets)
        adj.computeIfAbsent(t.get(0), k -> new PriorityQueue<>()).add(t.get(1));
    LinkedList<String> result = new LinkedList<>();
    dfs("JFK", adj, result);
    return result;
}
private void dfs(String airport, Map<String, PriorityQueue<String>> adj, LinkedList<String> result) {
    PriorityQueue<String> next = adj.get(airport);
    while (next != null && !next.isEmpty()) dfs(next.poll(), adj, result);
    result.addFirst(airport); // add on the way BACK (Hierholzer's)
}
```
**Key Insight:** Hierholzer's algorithm for Eulerian path. Add node to result on the way BACK — ensures dead ends are handled. MinPQ gives lexicographic order.
**Complexity:** Time: O(E log E) | Space: O(V+E)

---

### D18. Course Schedule (DFS Cycle Detection) (LC 207) — Medium
**Description:** n courses, prerequisites as directed edges. Detect if a valid ordering exists (no cycle).
**Example:**
```
Input:  numCourses=2, prerequisites=[[1,0],[0,1]]
Output: false  (cycle)
```
```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] p : prerequisites)
        adj.computeIfAbsent(p[1], k -> new ArrayList<>()).add(p[0]);
    int[] state = new int[numCourses]; // 0=unvisited, 1=visiting, 2=done
    for (int i = 0; i < numCourses; i++)
        if (state[i] == 0 && hasCycle(i, adj, state)) return false;
    return true;
}
private boolean hasCycle(int node, Map<Integer, List<Integer>> adj, int[] state) {
    state[node] = 1; // visiting (gray)
    for (int next : adj.getOrDefault(node, List.of())) {
        if (state[next] == 1) return true; // back edge = cycle
        if (state[next] == 0 && hasCycle(next, adj, state)) return true;
    }
    state[node] = 2; return false; // done (black)
}
```
**Key Insight:** 3-state DFS (white/gray/black). Gray node = currently on recursion stack → finding it again = cycle.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### D19. Number of Enclaves (LC 1020) — Medium
**Description:** Count land cells (`1`) that cannot reach the grid border by moving 4-directionally.
**Example:**
```
Input:  [[0,0,0,0],[1,0,1,0],[0,1,1,0],[0,0,0,0]]
Output: 3
```
```java
public int numEnclaves(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    for (int r = 0; r < rows; r++) { dfs(grid, r, 0); dfs(grid, r, cols-1); }
    for (int c = 0; c < cols; c++) { dfs(grid, 0, c); dfs(grid, rows-1, c); }
    int count = 0;
    for (int[] row : grid) for (int cell : row) if (cell == 1) count++;
    return count;
}
private void dfs(int[][] grid, int r, int c) {
    if (r<0||r>=grid.length||c<0||c>=grid[0].length||grid[r][c]!=1) return;
    grid[r][c] = 0;
    dfs(grid,r+1,c); dfs(grid,r-1,c); dfs(grid,r,c+1); dfs(grid,r,c-1);
}
```
**Key Insight:** Same as LC 130. Flood-fill border-connected land, then count remaining 1s.
**Complexity:** Time: O(mn) | Space: O(mn)

---

### D20. Reachable Nodes With Restrictions (LC 2368) — Medium
**Description:** Undirected tree with restricted nodes. From node 0, count reachable nodes without passing through restricted.
**Example:**
```
Input:  n=7, edges=[[0,1],[1,2],[3,1],[4,0],[0,5],[5,6]], restricted=[4,5]
Output: 4
```
```java
public int reachableNodes(int n, int[][] edges, int[] restricted) {
    Set<Integer> blocked = new HashSet<>();
    for (int r : restricted) blocked.add(r);
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] e : edges) {
        adj.computeIfAbsent(e[0], k->new ArrayList<>()).add(e[1]);
        adj.computeIfAbsent(e[1], k->new ArrayList<>()).add(e[0]);
    }
    boolean[] visited = new boolean[n];
    return dfs(adj, 0, visited, blocked);
}
private int dfs(Map<Integer, List<Integer>> adj, int node, boolean[] visited, Set<Integer> blocked) {
    visited[node] = true; int count = 1;
    for (int next : adj.getOrDefault(node, List.of()))
        if (!visited[next] && !blocked.contains(next)) count += dfs(adj, next, visited, blocked);
    return count;
}
```
**Key Insight:** Standard DFS — skip restricted nodes. Count starts at 1 (current node) and accumulates from reachable children.
**Complexity:** Time: O(V+E) | Space: O(V+E)


---

## 13. Topological Sort — 20 Problems

---

### T1. Course Schedule (Kahn's BFS) (LC 207) — Medium
**Description:** Detect cycle using Kahn's (in-degree) method. Return true if all courses can be finished.
**Example:**
```
Input:  numCourses=2, prerequisites=[[1,0]]
Output: true
```
```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    int[] inDegree = new int[numCourses];
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] p : prerequisites) {
        adj.computeIfAbsent(p[1], k -> new ArrayList<>()).add(p[0]);
        inDegree[p[0]]++;
    }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) if (inDegree[i] == 0) q.offer(i);
    int processed = 0;
    while (!q.isEmpty()) {
        int curr = q.poll(); processed++;
        for (int next : adj.getOrDefault(curr, List.of()))
            if (--inDegree[next] == 0) q.offer(next);
    }
    return processed == numCourses;
}
```
**Key Insight:** If processed count < n → cycle exists (those nodes never reached in-degree 0).
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T2. Find Eventual Safe States (LC 802) — Medium
**Description:** A node is "safe" if every path from it leads to a terminal node (no outgoing edges) without cycles.
**Example:**
```
Input:  graph=[[1,2],[2,3],[5],[0],[5],[],[]]
Output: [2,4,5,6]
```
```java
public List<Integer> eventualSafeNodes(int[][] graph) {
    int n = graph.length;
    int[] state = new int[n]; // 0=unvisited, 1=visiting, 2=safe
    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < n; i++) if (dfs(graph, i, state)) result.add(i);
    return result;
}
private boolean dfs(int[][] graph, int node, int[] state) {
    if (state[node] == 1) return false; // cycle
    if (state[node] == 2) return true;  // already safe
    state[node] = 1;
    for (int next : graph[node]) if (!dfs(graph, next, state)) { state[node] = 1; return false; }
    state[node] = 2; return true;
}
```
**Key Insight:** A node is safe iff it's NOT part of a cycle and all paths lead out. State=1 during DFS means on current path.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T3. Parallel Courses (LC 1136) — Medium (Premium)
**Description:** n courses, each takes 1 semester. Relations = prerequisites. Find minimum semesters to complete all.
**Example:**
```
Input:  n=3, relations=[[1,3],[2,3]]
Output: 2  (take 1,2 together in sem 1, then 3 in sem 2)
```
```java
public int minimumSemesters(int n, int[][] relations) {
    int[] inDegree = new int[n + 1];
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] r : relations) {
        adj.computeIfAbsent(r[0], k -> new ArrayList<>()).add(r[1]);
        inDegree[r[1]]++;
    }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 1; i <= n; i++) if (inDegree[i] == 0) q.offer(i);
    int sem = 0, taken = 0;
    while (!q.isEmpty()) {
        int size = q.size(); sem++;
        for (int i = 0; i < size; i++) {
            int cur = q.poll(); taken++;
            for (int next : adj.getOrDefault(cur, List.of()))
                if (--inDegree[next] == 0) q.offer(next);
        }
    }
    return taken == n ? sem : -1;
}
```
**Key Insight:** Kahn's BFS where each level = one semester (all courses with in-degree 0 taken in parallel).
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T4. Parallel Courses III (LC 2050) — Hard
**Description:** n courses, prerequisites, each with a time cost. Find minimum time to complete all (with parallelism).
**Example:**
```
Input:  n=3, relations=[[1,3],[2,3]], time=[3,2,5]
Output: 8  (start 1,2 in parallel; 3 starts when both done = max(3,2)+5=8)
```
```java
public int minimumTime(int n, int[][] relations, int[] time) {
    int[] inDegree = new int[n + 1];
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] r : relations) {
        adj.computeIfAbsent(r[0], k->new ArrayList<>()).add(r[1]);
        inDegree[r[1]]++;
    }
    int[] earliest = new int[n + 1]; // earliest completion time
    Queue<Integer> q = new LinkedList<>();
    for (int i = 1; i <= n; i++) if (inDegree[i] == 0) { q.offer(i); earliest[i] = time[i-1]; }
    while (!q.isEmpty()) {
        int cur = q.poll();
        for (int next : adj.getOrDefault(cur, List.of())) {
            earliest[next] = Math.max(earliest[next], earliest[cur] + time[next-1]);
            if (--inDegree[next] == 0) q.offer(next);
        }
    }
    int ans = 0;
    for (int i = 1; i <= n; i++) ans = Math.max(ans, earliest[i]);
    return ans;
}
```
**Key Insight:** Topo sort + critical path. `earliest[next] = max(earliest[next], earliest[cur] + time[next])` — wait for the slowest prerequisite.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T5. Largest Color Value in a Directed Graph (LC 1857) — Hard
**Description:** Nodes colored with lowercase letters. Find max frequency of any single color on any path. Return -1 if cycle.
**Example:**
```
Input:  colors="abaca", edges=[[0,1],[0,2],[2,3],[3,4]]
Output: 3  (color 'a' appears 3 times on path 0→2→3→4)
```
```java
public int largestPathValue(String colors, int[][] edges) {
    int n = colors.length();
    int[] inDegree = new int[n];
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] e : edges) { adj.computeIfAbsent(e[0], k->new ArrayList<>()).add(e[1]); inDegree[e[1]]++; }
    int[][] dp = new int[n][26]; // dp[node][color] = max count of color on any path ending at node
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < n; i++) if (inDegree[i] == 0) { q.offer(i); dp[i][colors.charAt(i)-'a'] = 1; }
    int processed = 0, ans = 0;
    while (!q.isEmpty()) {
        int cur = q.poll(); processed++;
        for (int c = 0; c < 26; c++) ans = Math.max(ans, dp[cur][c]);
        for (int next : adj.getOrDefault(cur, List.of())) {
            for (int c = 0; c < 26; c++)
                dp[next][c] = Math.max(dp[next][c], dp[cur][c] + (colors.charAt(next)-'a'==c?1:0));
            if (--inDegree[next] == 0) q.offer(next);
        }
    }
    return processed == n ? ans : -1;
}
```
**Key Insight:** Topo sort + DP. `dp[node][c]` = max count of color `c` on any path ending at `node`. Propagate forward.
**Complexity:** Time: O(26*(V+E)) | Space: O(26*V)

---

### T6. Sort Items by Groups Respecting Dependencies (LC 1203) — Hard
**Description:** n items in groups. Items and groups have dependencies. Return valid order or empty if impossible.
**Example:**
```
Input:  n=8, m=2, group=[-1,-1,1,0,0,1,0,-1], beforeItems=[[],[6],[5],[6],[3,6],[],[],[]]
Output: [6,3,4,1,5,2,0,7]
```
```java
public int[] sortItems(int n, int m, int[] group, List<List<Integer>> beforeItems) {
    // Assign unique group IDs to -1 items
    for (int i = 0; i < n; i++) if (group[i] == -1) group[i] = m++;
    Map<Integer,List<Integer>> itemAdj=new HashMap<>(), groupAdj=new HashMap<>();
    int[] itemIn=new int[n], groupIn=new int[m];
    for (int i = 0; i < n; i++) {
        itemAdj.computeIfAbsent(i, k->new ArrayList<>());
        groupAdj.computeIfAbsent(group[i], k->new ArrayList<>());
        for (int before : beforeItems.get(i)) {
            itemAdj.get(before).add(i); itemIn[i]++;
            if (group[before] != group[i]) { groupAdj.get(group[before]).add(group[i]); groupIn[group[i]]++; }
        }
    }
    List<Integer> itemOrder = topoSort(itemAdj, itemIn, n);
    List<Integer> groupOrder = topoSort(groupAdj, groupIn, m);
    if (itemOrder.isEmpty() || groupOrder.isEmpty()) return new int[0];
    Map<Integer, List<Integer>> groupItems = new HashMap<>();
    for (int item : itemOrder) groupItems.computeIfAbsent(group[item], k->new ArrayList<>()).add(item);
    int[] result = new int[n]; int idx = 0;
    for (int g : groupOrder) for (int item : groupItems.getOrDefault(g, List.of())) result[idx++] = item;
    return result;
}
private List<Integer> topoSort(Map<Integer, List<Integer>> adj, int[] inDegree, int n) {
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < n; i++) if (inDegree[i] == 0) q.offer(i);
    List<Integer> order = new ArrayList<>();
    while (!q.isEmpty()) { int cur = q.poll(); order.add(cur); for (int next : adj.getOrDefault(cur, List.of())) if (--inDegree[next] == 0) q.offer(next); }
    return order.size() == n ? order : List.of();
}
```
**Key Insight:** Two-level topological sort — one for items within groups, one for groups themselves. Merge results by group order.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T7. Find All Possible Recipes from Given Supplies (LC 2115) — Medium
**Description:** Given ingredients and recipes (each recipe needs ingredients/other recipes), find all makeable recipes.
**Example:**
```
Input:  recipes=["bread"], ingredients=[["yeast","flour"]], supplies=["yeast","flour","corn"]
Output: ["bread"]
```
```java
public List<String> findAllRecipes(String[] recipes, List<List<String>> ingredients, String[] supplies) {
    Map<String, List<String>> adj = new HashMap<>(); // ingredient -> recipes needing it
    Map<String, Integer> inDegree = new HashMap<>();
    for (String r : recipes) inDegree.put(r, 0);
    for (int i = 0; i < recipes.length; i++) {
        for (String ing : ingredients.get(i)) {
            adj.computeIfAbsent(ing, k->new ArrayList<>()).add(recipes[i]);
            inDegree.merge(recipes[i], 1, Integer::sum);
        }
    }
    Queue<String> q = new LinkedList<>(Arrays.asList(supplies));
    List<String> result = new ArrayList<>();
    while (!q.isEmpty()) {
        String cur = q.poll();
        for (String recipe : adj.getOrDefault(cur, List.of())) {
            if (inDegree.merge(recipe, -1, Integer::sum) == 0) { result.add(recipe); q.offer(recipe); }
        }
    }
    return result;
}
```
**Key Insight:** Topo sort where initial supplies = nodes with in-degree 0. Completed recipes become new "supplies" and can unlock more recipes.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T8. All Ancestors of a Node in a DAG (LC 2192) — Medium
**Description:** For each node in a DAG, return a sorted list of all its ancestors.
**Example:**
```
Input:  n=8, edgeList=[[0,3],[0,4],[1,3],[2,4],[2,7],[3,5],[3,6],[3,7],[4,6]]
Output: [[],[],[],[0,1],[0,2],[0,1,3],[0,1,2,3,4],[0,1,2,3]]
```
```java
public List<List<Integer>> getAncestors(int n, int[][] edges) {
    List<Set<Integer>> ancestors = new ArrayList<>();
    for (int i = 0; i < n; i++) ancestors.add(new TreeSet<>());
    int[] inDegree = new int[n];
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] e : edges) { adj.computeIfAbsent(e[0], k->new ArrayList<>()).add(e[1]); inDegree[e[1]]++; }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < n; i++) if (inDegree[i] == 0) q.offer(i);
    while (!q.isEmpty()) {
        int cur = q.poll();
        for (int next : adj.getOrDefault(cur, List.of())) {
            ancestors.get(next).add(cur);
            ancestors.get(next).addAll(ancestors.get(cur)); // propagate ancestors
            if (--inDegree[next] == 0) q.offer(next);
        }
    }
    return new ArrayList<>(ancestors);
}
```
**Key Insight:** Topo sort + propagate ancestor sets forward. Each node's ancestor set = union of all parents' ancestor sets + the parents themselves.
**Complexity:** Time: O(V^2 + E) | Space: O(V^2)

---

### T9. Longest Cycle in a Graph (LC 2360) — Hard
**Description:** Directed graph where each node has at most one outgoing edge. Find length of longest cycle or -1.
**Example:**
```
Input:  edges=[3,3,4,2,3]
Output: 3  (2→4→3→2)
```
```java
public int longestCycle(int[] edges) {
    int n = edges.length, ans = -1;
    int[] visited = new int[n]; // 0=unvisited, -1=done, else=visit_time
    int time = 1;
    for (int i = 0; i < n; i++) {
        if (visited[i] != 0) continue;
        int start = time, node = i;
        while (node != -1 && visited[node] == 0) {
            visited[node] = time++; node = edges[node];
        }
        if (node != -1 && visited[node] >= start) // found cycle in current traversal
            ans = Math.max(ans, time - visited[node]);
        // mark all nodes in current traversal as done
        node = i;
        while (node != -1 && visited[node] >= start) { visited[node] = -1; node = edges[node]; }
    }
    return ans;
}
```
**Key Insight:** Functional graph (each node ≤ 1 outgoing edge). Track visit timestamp; cycle detected when revisiting a node from the same traversal.
**Complexity:** Time: O(V) | Space: O(V)

---

### T10. Longest Path With Different Adjacent Characters (LC 2246) — Hard
**Description:** Rooted tree, each node has a character. Find longest path where no two adjacent nodes share the same character.
**Example:**
```
Input:  parent=[-1,0,0,1,1,2], s="abacbe"
Output: 3  (path 1→3 has length 3... actually let me recheck)
Output: 3
```
```java
public int longestPath(int[] parent, String s) {
    int n = parent.length;
    Map<Integer, List<Integer>> children = new HashMap<>();
    for (int i = 1; i < n; i++) children.computeIfAbsent(parent[i], k->new ArrayList<>()).add(i);
    int[] ans = {1};
    dfs(0, s, children, ans);
    return ans[0];
}
private int dfs(int node, String s, Map<Integer, List<Integer>> children, int[] ans) {
    int best1 = 0, best2 = 0; // top two child path lengths
    for (int child : children.getOrDefault(node, List.of())) {
        int childLen = dfs(child, s, children, ans);
        if (s.charAt(child) == s.charAt(node)) continue; // skip same-char children
        if (childLen > best1) { best2 = best1; best1 = childLen; }
        else if (childLen > best2) best2 = childLen;
    }
    ans[0] = Math.max(ans[0], best1 + best2 + 1);
    return best1 + 1;
}
```
**Key Insight:** At each node, combine top-2 valid child paths (different character). Return only best single branch upward.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T11. Sequence Reconstruction (LC 444) — Medium (Premium)
**Description:** Check if `org` is the UNIQUE shortest supersequence reconstructable from `seqs`.
**Example:**
```
Input:  org=[1,2,3], seqs=[[1,2],[1,3],[2,3]]
Output: true
```
```java
public boolean sequenceReconstruction(int[] org, List<List<Integer>> seqs) {
    Map<Integer,Set<Integer>> adj = new HashMap<>();
    Map<Integer,Integer> inDegree = new HashMap<>();
    for (int n : org) { adj.put(n, new HashSet<>()); inDegree.put(n, 0); }
    for (List<Integer> seq : seqs) for (int i = 1; i < seq.size(); i++) {
        int u=seq.get(i-1), v=seq.get(i);
        if (!adj.containsKey(u)||!adj.containsKey(v)) return false;
        if (adj.get(u).add(v)) inDegree.merge(v, 1, Integer::sum);
    }
    Queue<Integer> q = new LinkedList<>();
    for (int n : org) if (inDegree.get(n) == 0) q.offer(n);
    int idx = 0;
    while (!q.isEmpty()) {
        if (q.size() > 1) return false; // ambiguous → not unique
        int cur = q.poll();
        if (org[idx++] != cur) return false;
        for (int next : adj.get(cur)) if (inDegree.merge(next,-1,Integer::sum)==0) q.offer(next);
    }
    return idx == org.length;
}
```
**Key Insight:** Queue must have exactly ONE element at all times (unique ordering). Multiple elements = ambiguous = not unique.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T12. Course Schedule IV (LC 1462) — Medium
**Description:** Given direct and transitive prerequisites, answer queries: "is course a a prerequisite of course b?"
**Example:**
```
Input:  numCourses=2, prerequisites=[[1,0]], queries=[[0,1],[1,0]]
Output: [false,true]
```
```java
public List<Boolean> checkIfPrerequisite(int numCourses, int[][] prerequisites, int[][] queries) {
    boolean[][] reachable = new boolean[numCourses][numCourses];
    int[] inDegree = new int[numCourses];
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] p : prerequisites) {
        adj.computeIfAbsent(p[0], k->new ArrayList<>()).add(p[1]);
        inDegree[p[1]]++;
    }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) if (inDegree[i] == 0) q.offer(i);
    while (!q.isEmpty()) {
        int cur = q.poll();
        for (int next : adj.getOrDefault(cur, List.of())) {
            reachable[cur][next] = true;
            for (int k = 0; k < numCourses; k++) if (reachable[k][cur]) reachable[k][next] = true;
            if (--inDegree[next] == 0) q.offer(next);
        }
    }
    List<Boolean> res = new ArrayList<>();
    for (int[] query : queries) res.add(reachable[query[0]][query[1]]);
    return res;
}
```
**Key Insight:** Topo sort + propagate reachability matrix. When processing edge `(cur→next)`, all nodes that can reach `cur` can also reach `next`.
**Complexity:** Time: O(V^2 + E) | Space: O(V^2)

---

### T13. Build a Matrix With Conditions (LC 2392) — Hard
**Description:** Build k×k matrix with values 1..k, such that row/column order satisfies given conditions.
**Example:**
```
Input:  k=3, rowConditions=[[1,2],[3,2]], colConditions=[[2,1],[3,2]]
Output: [[3,0,0],[0,0,1],[0,2,0]]
```
```java
public int[][] buildMatrix(int k, int[][] rowConditions, int[][] colConditions) {
    int[] rowOrder = topoSort(k, rowConditions);
    int[] colOrder = topoSort(k, colConditions);
    if (rowOrder == null || colOrder == null) return new int[0][0];
    int[] rowPos = new int[k+1], colPos = new int[k+1];
    for (int i = 0; i < k; i++) { rowPos[rowOrder[i]] = i; colPos[colOrder[i]] = i; }
    int[][] matrix = new int[k][k];
    for (int v = 1; v <= k; v++) matrix[rowPos[v]][colPos[v]] = v;
    return matrix;
}
private int[] topoSort(int k, int[][] conditions) {
    int[] inDegree = new int[k+1];
    Map<Integer,List<Integer>> adj = new HashMap<>();
    for (int[] c : conditions) { adj.computeIfAbsent(c[0],x->new ArrayList<>()).add(c[1]); inDegree[c[1]]++; }
    Queue<Integer> q = new LinkedList<>();
    for (int i = 1; i <= k; i++) if (inDegree[i] == 0) q.offer(i);
    int[] order = new int[k]; int idx = 0;
    while (!q.isEmpty()) { int cur=q.poll(); order[idx++]=cur; for(int next:adj.getOrDefault(cur,List.of())) if(--inDegree[next]==0) q.offer(next); }
    return idx == k ? order : null;
}
```
**Key Insight:** Two independent topo sorts (rows and cols). Place value `v` at `[rowPos[v]][colPos[v]]`.
**Complexity:** Time: O(k^2) | Space: O(k)

---

### T14. Alien Dictionary (LC 269) — Hard (already in file, variant)
**Description:** See existing entry. Key variations: prefix contradiction, multiple valid orderings.
**Key Insight (additional):** Always check `if w1.length() > w2.length() && w1.startsWith(w2) → return ""` before adding edges.

---

### T15. Minimum Number of Vertices to Reach All Nodes (LC 1557) — Medium
**Description:** Find smallest set of vertices from which all nodes in a DAG are reachable.
**Example:**
```
Input:  n=6, edges=[[0,1],[0,2],[2,5],[3,4],[4,2]]
Output: [0,3]
```
```java
public List<Integer> findSmallestSetOfVertices(int n, List<List<Integer>> edges) {
    boolean[] hasIncoming = new boolean[n];
    for (List<Integer> edge : edges) hasIncoming[edge.get(1)] = true;
    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < n; i++) if (!hasIncoming[i]) result.add(i);
    return result;
}
```
**Key Insight:** In a DAG, any node with no incoming edges MUST be in the set. Any node with incoming edges is already reachable from its parents.
**Complexity:** Time: O(V+E) | Space: O(V)

---

### T16. All Paths From Source Lead to Destination (LC 1059) — Medium (Premium)
**Description:** Check if all paths from `source` end at `destination` (no other dead ends or cycles).
**Example:**
```
Input:  n=4, edges=[[1,0],[2,0],[3,0]], source=3, destination=0
Output: false
```
```java
public boolean leadsToDestination(int n, int[][] edges, int source, int destination) {
    Map<Integer,List<Integer>> adj = new HashMap<>();
    for (int[] e : edges) adj.computeIfAbsent(e[0], k->new ArrayList<>()).add(e[1]);
    int[] state = new int[n]; // 0=unvisited, 1=visiting, 2=safe
    return dfs(source, destination, adj, state);
}
private boolean dfs(int node, int dest, Map<Integer,List<Integer>> adj, int[] state) {
    if (adj.getOrDefault(node, List.of()).isEmpty()) return node == dest; // dead end must be dest
    if (state[node] == 1) return false; // cycle
    if (state[node] == 2) return true;
    state[node] = 1;
    for (int next : adj.get(node)) if (!dfs(next, dest, adj, state)) { return false; }
    state[node] = 2; return true;
}
```
**Key Insight:** Dead ends must be `destination`. Any cycle → false. Any dead end other than destination → false.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T17. Number of Ways to Arrive at Destination (LC 1976) — Medium
**Description:** Count the number of ways to reach destination in shortest time. Answer mod 1e9+7.
**Example:**
```
Input:  n=7, roads=[[0,6,7],[0,1,2],[1,2,3],[1,3,3],[6,3,3],[3,5,1],[6,5,2],[3,4,2],[5,4,2]]
Output: 4
```
```java
public int countPaths(int n, int[][] roads) {
    long MOD = 1_000_000_007;
    Map<Integer,List<long[]>> adj = new HashMap<>();
    for (int[] r : roads) { adj.computeIfAbsent(r[0],k->new ArrayList<>()).add(new long[]{r[1],r[2]}); adj.computeIfAbsent(r[1],k->new ArrayList<>()).add(new long[]{r[0],r[2]}); }
    long[] dist = new long[n]; Arrays.fill(dist, Long.MAX_VALUE); dist[0] = 0;
    long[] ways = new long[n]; ways[0] = 1;
    PriorityQueue<long[]> pq = new PriorityQueue<>((a,b)->Long.compare(a[0],b[0]));
    pq.offer(new long[]{0, 0});
    while (!pq.isEmpty()) {
        long[] cur = pq.poll(); long d=cur[0]; int node=(int)cur[1];
        if (d > dist[node]) continue;
        for (long[] edge : adj.getOrDefault(node, List.of())) {
            int next=(int)edge[0]; long nd=d+edge[1];
            if (nd < dist[next]) { dist[next]=nd; ways[next]=ways[node]; pq.offer(new long[]{nd,next}); }
            else if (nd == dist[next]) ways[next]=(ways[next]+ways[node])%MOD;
        }
    }
    return (int)ways[n-1];
}
```
**Key Insight:** Dijkstra + path counting. When new shorter path found: reset count. When equal path found: add count.
**Complexity:** Time: O((V+E) log V) | Space: O(V+E)

---

### T18. Loud and Rich (LC 851) — Medium
**Description:** `richer[i]=[a,b]` means a has more money than b. `quiet[i]` = quietness of person i. Find for each person x, the least quiet person among all who have ≥ as much money as x.
**Example:**
```
Input:  richer=[[1,0],[2,1],[3,1],[3,7],[4,3],[5,3],[6,3]], quiet=[3,2,5,4,6,1,7,0]
Output: [5,5,2,5,4,5,6,7]
```
```java
public int[] loudAndRich(int[][] richer, int[] quiet) {
    int n = quiet.length;
    Map<Integer,List<Integer>> adj = new HashMap<>(); // b -> [a: a richer than b]
    int[] inDegree = new int[n];
    for (int[] r : richer) { adj.computeIfAbsent(r[0],k->new ArrayList<>()).add(r[1]); inDegree[r[1]]++; }
    int[] ans = new int[n];
    for (int i = 0; i < n; i++) ans[i] = i;
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < n; i++) if (inDegree[i] == 0) q.offer(i);
    while (!q.isEmpty()) {
        int cur = q.poll();
        for (int next : adj.getOrDefault(cur, List.of())) {
            if (quiet[ans[cur]] < quiet[ans[next]]) ans[next] = ans[cur];
            if (--inDegree[next] == 0) q.offer(next);
        }
    }
    return ans;
}
```
**Key Insight:** Topo sort from richest (no one richer) downward. Propagate "quietest person in richer group" from richer to poorer.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T19. Course Schedule III (LC 630) — Hard
**Description:** n courses `[duration, lastDay]`. Maximize courses you can take (take in any order, can't exceed lastDay).
**Example:**
```
Input:  courses=[[100,200],[200,1300],[1000,1250],[2000,3200]]
Output: 3
```
```java
public int scheduleCourse(int[][] courses) {
    Arrays.sort(courses, (a, b) -> a[1] - b[1]); // sort by deadline
    PriorityQueue<Integer> heap = new PriorityQueue<>(Collections.reverseOrder()); // max-heap of durations
    int time = 0;
    for (int[] course : courses) {
        heap.offer(course[0]); time += course[0];
        if (time > course[1]) time -= heap.poll(); // remove longest taken course
    }
    return heap.size();
}
```
**Key Insight:** Greedy — sort by deadline. Take each course; if deadline exceeded, swap out the longest-duration course taken so far (may be the current one).
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### T20. Rank Transform of a Matrix (LC 1632) — Hard
**Description:** Replace each element with its rank. Same-row/column equal elements get same rank. Rank starts at 1.
**Example:**
```
Input:  matrix=[[1,2],[3,4]]
Output: [[1,2],[2,3]]
```
```java
public int[][] matrixRankTransform(int[][] matrix) {
    int m=matrix.length, n=matrix[0].length;
    int[] rank = new int[m + n]; // rank[i] for row i, rank[m+j] for col j
    Map<Integer, List<int[]>> valCells = new TreeMap<>();
    for (int i=0;i<m;i++) for(int j=0;j<n;j++) valCells.computeIfAbsent(matrix[i][j],k->new ArrayList<>()).add(new int[]{i,j});
    int[][] result = new int[m][n];
    int[] parent = new int[m+n];
    for (int i=0;i<m+n;i++) parent[i]=i;
    for (Map.Entry<Integer,List<int[]>> e : valCells.entrySet()) {
        Arrays.fill(parent, 0, m+n, 0); for(int i=0;i<m+n;i++) parent[i]=i;
        int[] maxRank = new int[m+n];
        for (int[] cell : e.getValue()) { int r=cell[0],c=cell[1]; union(parent,maxRank,r,m+c,Math.max(rank[r],rank[m+c])); }
        for (int[] cell : e.getValue()) {
            int r=cell[0],c=cell[1],root=find(parent,r);
            result[r][c] = rank[r] = rank[m+c] = maxRank[root]+1;
        }
    }
    return result;
}
private int find(int[] p, int x) { return p[x]==x?x:(p[x]=find(p,p[x])); }
private void union(int[] p, int[] maxRank, int a, int b, int r) { a=find(p,a);b=find(p,b); if(a!=b)p[a]=b; maxRank[find(p,b)]=Math.max(maxRank[find(p,b)],r); }
```
**Key Insight:** Process values in sorted order. Group same-value cells in same row/col using Union-Find; assign rank = max existing rank + 1 in each group.
**Complexity:** Time: O(mn log(mn)) | Space: O(mn)


---

## 14. Dijkstra — 20 Problems

---

### Dij1. Network Delay Time (LC 743) — Medium
*(See existing entry #10 in file — standard Dijkstra template)*

---

### Dij2. Cheapest Flights Within K Stops (LC 787) — Medium
*(See existing entry #11 — compound state Dijkstra)*

---

### Dij3. Path With Minimum Effort (LC 1631) — Medium
*(See existing entry #12 — grid Dijkstra with max-diff cost)*

---

### Dij4. Swim in Rising Water (LC 778) — Hard
**Description:** Grid of heights. You can swim when time `t` ≥ max cell height on path. Find min time to reach bottom-right.
**Example:**
```
Input:  grid=[[0,2],[1,3]]
Output: 3
```
```java
public int swimInWater(int[][] grid) {
    int n = grid.length;
    int[][] dist = new int[n][n];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
    dist[0][0] = grid[0][0];
    PriorityQueue<int[]> pq = new PriorityQueue<>((a,b)->a[0]-b[0]);
    pq.offer(new int[]{grid[0][0], 0, 0});
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    while (!pq.isEmpty()) {
        int[] cur = pq.poll(); int t=cur[0],r=cur[1],c=cur[2];
        if (r==n-1&&c==n-1) return t;
        if (t > dist[r][c]) continue;
        for (int[] d : dirs) {
            int nr=r+d[0],nc=c+d[1];
            if (nr<0||nr>=n||nc<0||nc>=n) continue;
            int nt = Math.max(t, grid[nr][nc]);
            if (nt < dist[nr][nc]) { dist[nr][nc]=nt; pq.offer(new int[]{nt,nr,nc}); }
        }
    }
    return dist[n-1][n-1];
}
```
**Key Insight:** Cost = max height seen so far (bottleneck path). Same pattern as LC 1631 but using `Math.max` instead of `Math.max(abs(diff))`.
**Complexity:** Time: O(n^2 log n) | Space: O(n^2)

---

### Dij5. Path with Maximum Probability (LC 1514) — Medium
**Description:** Undirected graph with edge probabilities. Find max probability path from start to end.
**Example:**
```
Input:  n=3, edges=[[0,1],[1,2],[0,2]], succProb=[0.5,0.5,0.2], start=0, end=2
Output: 0.25
```
```java
public double maxProbability(int n, int[][] edges, double[] succProb, int start, int end) {
    Map<Integer,List<double[]>> adj = new HashMap<>();
    for (int i=0;i<edges.length;i++) {
        adj.computeIfAbsent(edges[i][0],k->new ArrayList<>()).add(new double[]{edges[i][1],succProb[i]});
        adj.computeIfAbsent(edges[i][1],k->new ArrayList<>()).add(new double[]{edges[i][0],succProb[i]});
    }
    double[] prob = new double[n]; prob[start] = 1.0;
    PriorityQueue<double[]> pq = new PriorityQueue<>((a,b)->Double.compare(b[0],a[0])); // MAX heap
    pq.offer(new double[]{1.0, start});
    while (!pq.isEmpty()) {
        double[] cur = pq.poll(); double p=cur[0]; int node=(int)cur[1];
        if (node == end) return p;
        if (p < prob[node]) continue;
        for (double[] edge : adj.getOrDefault(node, List.of())) {
            int next=(int)edge[0]; double np=p*edge[1];
            if (np > prob[next]) { prob[next]=np; pq.offer(new double[]{np,next}); }
        }
    }
    return 0.0;
}
```
**Key Insight:** Max-heap (reverse order). Multiply probabilities instead of adding costs. "Shortest path" → "longest product path."
**Complexity:** Time: O((V+E) log V) | Space: O(V+E)

---

### Dij6. Second Minimum Time to Reach Destination (LC 2045) — Hard
**Description:** Undirected unweighted graph. Traffic signal alternates green/red each `change` seconds. Find second minimum time to reach n.
**Example:**
```
Input:  n=5, edges=[[1,2],[1,3],[1,4],[3,4],[4,5]], time=3, change=5
Output: 13
```
```java
public int secondMinimum(int n, int[][] edges, int time, int change) {
    Map<Integer,List<Integer>> adj = new HashMap<>();
    for (int[] e : edges) { adj.computeIfAbsent(e[0],k->new ArrayList<>()).add(e[1]); adj.computeIfAbsent(e[1],k->new ArrayList<>()).add(e[0]); }
    int[] dist1 = new int[n+1], dist2 = new int[n+1];
    Arrays.fill(dist1, Integer.MAX_VALUE); Arrays.fill(dist2, Integer.MAX_VALUE);
    dist1[1] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((a,b)->a[0]-b[0]);
    pq.offer(new int[]{0, 1});
    while (!pq.isEmpty()) {
        int[] cur = pq.poll(); int t=cur[0],node=cur[1];
        if (t > dist2[node]) continue;
        // wait at red light if in red phase
        int cycles = t / change;
        if (cycles % 2 == 1) t = (cycles + 1) * change; // wait for green
        int nt = t + time;
        for (int next : adj.getOrDefault(node, List.of())) {
            if (nt < dist1[next]) { dist2[next]=dist1[next]; dist1[next]=nt; pq.offer(new int[]{nt,next}); }
            else if (nt > dist1[next] && nt < dist2[next]) { dist2[next]=nt; pq.offer(new int[]{nt,next}); }
        }
    }
    return dist2[n];
}
```
**Key Insight:** Track both shortest and second-shortest times per node. Red-light delay: if in odd half-cycle, wait until next green.
**Complexity:** Time: O((V+E) log V) | Space: O(V+E)

---

### Dij7. Minimum Cost to Make at Least One Valid Path in a Grid (LC 1368) — Hard
**Description:** Grid with directions 1=right,2=left,3=down,4=up. Moving with grid direction costs 0, against costs 1. Find min cost to reach bottom-right.
**Example:**
```
Input:  grid=[[1,1,1,1],[2,2,2,2],[1,1,1,1],[2,2,2,2]]
Output: 3
```
```java
public int minCost(int[][] grid) {
    int rows=grid.length,cols=grid[0].length;
    int[][] dist=new int[rows][cols]; for(int[] r:dist) Arrays.fill(r,Integer.MAX_VALUE); dist[0][0]=0;
    int[][] dirs={{0,1},{0,-1},{1,0},{-1,0}};
    Deque<int[]> dq = new ArrayDeque<>(); dq.offerFirst(new int[]{0,0,0});
    while (!dq.isEmpty()) {
        int[] cur=dq.pollFirst(); int cost=cur[0],r=cur[1],c=cur[2];
        if (cost>dist[r][c]) continue;
        for (int d=0;d<4;d++) {
            int nr=r+dirs[d][0],nc=c+dirs[d][1]; if(nr<0||nr>=rows||nc<0||nc>=cols) continue;
            int nc2=(grid[r][c]-1==d)?0:1; // 0 cost if following grid direction
            int nd=cost+nc2;
            if (nd<dist[nr][nc]) { dist[nr][nc]=nd; if(nc2==0) dq.offerFirst(new int[]{nd,nr,nc}); else dq.offerLast(new int[]{nd,nr,nc}); }
        }
    }
    return dist[rows-1][cols-1];
}
```
**Key Insight:** 0-1 BFS (deque). Free moves (following grid direction) go to front, cost-1 moves go to back. Same guarantee as Dijkstra but O(mn).
**Complexity:** Time: O(mn)  0-1 BFS | Space: O(mn)

---

### Dij8. Find the City With the Smallest Number of Neighbors at a Threshold Distance (LC 1334) — Medium
**Description:** Find city with fewest other cities reachable within `distanceThreshold`.
**Example:**
```
Input:  n=4, edges=[[0,1,3],[1,2,1],[1,3,4],[2,3,1]], distanceThreshold=4
Output: 3
```
```java
public int findTheCity(int n, int[][] edges, int distanceThreshold) {
    int[][] dist = new int[n][n];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE / 2);
    for (int i = 0; i < n; i++) dist[i][i] = 0;
    for (int[] e : edges) { dist[e[0]][e[1]]=e[2]; dist[e[1]][e[0]]=e[2]; }
    // Floyd-Warshall for all-pairs shortest paths
    for (int k=0;k<n;k++) for(int i=0;i<n;i++) for(int j=0;j<n;j++)
        dist[i][j]=Math.min(dist[i][j],dist[i][k]+dist[k][j]);
    int ans=-1, minNeighbors=n;
    for (int i=0;i<n;i++) { int cnt=0; for(int j=0;j<n;j++) if(i!=j&&dist[i][j]<=distanceThreshold) cnt++; if(cnt<=minNeighbors){minNeighbors=cnt;ans=i;} }
    return ans;
}
```
**Key Insight:** Small n (≤100) → Floyd-Warshall O(n³) is fine. Prefer city with higher index on tie (iterating forward guarantees this).
**Complexity:** Time: O(n^3)  Floyd-Warshall | Space: O(n^2)

---

### Dij9. Minimum Time to Visit a Cell In a Grid (LC 2577) — Hard
**Description:** Grid where `grid[i][j]` = earliest time you can enter that cell. Move 4-dirs, each step takes 1 second. Find min time to reach bottom-right.
**Example:**
```
Input:  grid=[[0,1,3,2],[5,1,2,5],[4,3,8,6]]
Output: 7
```
```java
public int minimumTime(int[][] grid) {
    int rows=grid.length,cols=grid[0].length;
    if (grid[0][1]>1&&grid[1][0]>1) return -1;
    int[][] dist=new int[rows][cols]; for(int[] r:dist) Arrays.fill(r,Integer.MAX_VALUE);
    dist[0][0]=0;
    PriorityQueue<int[]> pq=new PriorityQueue<>((a,b)->a[0]-b[0]);
    pq.offer(new int[]{0,0,0});
    int[][] dirs={{0,1},{0,-1},{1,0},{-1,0}};
    while (!pq.isEmpty()) {
        int[] cur=pq.poll(); int t=cur[0],r=cur[1],c=cur[2];
        if (r==rows-1&&c==cols-1) return t;
        if (t>dist[r][c]) continue;
        for (int[] d:dirs) {
            int nr=r+d[0],nc=c+d[1]; if(nr<0||nr>=rows||nc<0||nc>=cols) continue;
            int nt=t+1;
            if (nt<grid[nr][nc]) { int diff=grid[nr][nc]-nt; nt=grid[nr][nc]+(diff%2==0?0:1); } // wait by bouncing
            if (nt<dist[nr][nc]) { dist[nr][nc]=nt; pq.offer(new int[]{nt,nr,nc}); }
        }
    }
    return -1;
}
```
**Key Insight:** If you arrive too early, you must wait by bouncing back-and-forth (+2 each bounce). Add 1 if parity mismatch (grid time is odd distance away).
**Complexity:** Time: O(mn log(mn)) | Space: O(mn)

---

### Dij10. The Maze II (LC 505) — Medium (Premium)
**Description:** Ball rolls until hitting a wall. Find shortest distance path from start to destination.
**Example:**
```
Input:  maze=[[0,0,1,0,0],[0,0,0,0,0],[0,0,0,1,0],[1,1,0,1,1],[0,0,0,0,0]], start=[0,4], dest=[4,4]
Output: 12
```
```java
public int shortestDistance(int[][] maze, int[] start, int[] dest) {
    int rows=maze.length,cols=maze[0].length;
    int[][] dist=new int[rows][cols]; for(int[] r:dist) Arrays.fill(r,Integer.MAX_VALUE); dist[start[0]][start[1]]=0;
    PriorityQueue<int[]> pq=new PriorityQueue<>((a,b)->a[0]-b[0]);
    pq.offer(new int[]{0,start[0],start[1]});
    int[][] dirs={{0,1},{0,-1},{1,0},{-1,0}};
    while (!pq.isEmpty()) {
        int[] cur=pq.poll(); int d=cur[0],r=cur[1],c=cur[2];
        if (r==dest[0]&&c==dest[1]) return d;
        if (d>dist[r][c]) continue;
        for (int[] dir:dirs) {
            int nr=r,nc=c,steps=0;
            while(nr+dir[0]>=0&&nr+dir[0]<rows&&nc+dir[1]>=0&&nc+dir[1]<cols&&maze[nr+dir[0]][nc+dir[1]]==0){nr+=dir[0];nc+=dir[1];steps++;}
            if (d+steps<dist[nr][nc]) { dist[nr][nc]=d+steps; pq.offer(new int[]{d+steps,nr,nc}); }
        }
    }
    return -1;
}
```
**Key Insight:** Roll until hitting a wall — each "edge" has variable length. Dijkstra on stop positions (not individual cells).
**Complexity:** Time: O(mn log(mn)) | Space: O(mn)

---

### Dij11. Minimum Obstacle Removal to Reach Corner (LC 2290) — Hard
**Description:** Grid of 0s and 1s. Move 4-dirs. Removing an obstacle (1) costs 1. Find min obstacles to remove to reach bottom-right.
**Example:**
```
Input:  grid=[[0,1,1],[1,1,0],[1,1,0]]
Output: 2
```
```java
public int minimumObstacles(int[][] grid) {
    int rows=grid.length,cols=grid[0].length;
    int[][] dist=new int[rows][cols]; for(int[] r:dist) Arrays.fill(r,Integer.MAX_VALUE); dist[0][0]=0;
    Deque<int[]> dq=new ArrayDeque<>(); dq.offerFirst(new int[]{0,0,0});
    int[][] dirs={{0,1},{0,-1},{1,0},{-1,0}};
    while (!dq.isEmpty()) {
        int[] cur=dq.pollFirst(); int cost=cur[0],r=cur[1],c=cur[2];
        if (cost>dist[r][c]) continue;
        for (int[] d:dirs) {
            int nr=r+d[0],nc=c+d[1]; if(nr<0||nr>=rows||nc<0||nc>=cols) continue;
            int nc2=grid[nr][nc]; int nd=cost+nc2;
            if (nd<dist[nr][nc]) { dist[nr][nc]=nd; if(nc2==0) dq.offerFirst(new int[]{nd,nr,nc}); else dq.offerLast(new int[]{nd,nr,nc}); }
        }
    }
    return dist[rows-1][cols-1];
}
```
**Key Insight:** 0-1 BFS. Moving to 0 costs 0 (front of deque), moving to 1 costs 1 (back of deque). Equivalent to Dijkstra but O(mn).
**Complexity:** Time: O(mn)  0-1 BFS | Space: O(mn)

---

### Dij12. Number of Restricted Paths From First to Last Node (LC 1786) — Medium
**Description:** Count paths from 1 to n where each step goes to a node closer (smaller distToEnd) to n. Answer mod 1e9+7.
**Example:**
```
Input:  n=5, edges=[[1,2,3],[1,3,3],[2,3,1],[1,4,2],[5,2,2],[3,5,1],[5,4,10]]
Output: 3
```
```java
public int countRestrictedPaths(int n, int[][] edges) {
    long MOD=1_000_000_007;
    Map<Integer,List<int[]>> adj=new HashMap<>();
    for (int[] e:edges){adj.computeIfAbsent(e[0],k->new ArrayList<>()).add(new int[]{e[1],e[2]});adj.computeIfAbsent(e[1],k->new ArrayList<>()).add(new int[]{e[0],e[2]});}
    long[] dist=new long[n+1]; Arrays.fill(dist,Long.MAX_VALUE); dist[n]=0;
    PriorityQueue<long[]> pq=new PriorityQueue<>((a,b)->Long.compare(a[0],b[0]));
    pq.offer(new long[]{0,n});
    while (!pq.isEmpty()){long[]cur=pq.poll();long d=cur[0];int node=(int)cur[1];if(d>dist[node])continue;for(int[]e:adj.getOrDefault(node,List.of())){long nd=d+e[1];if(nd<dist[e[0]]){dist[e[0]]=nd;pq.offer(new long[]{nd,e[0]});}}}
    long[] ways=new long[n+1]; ways[n]=1;
    Integer[] order=new Integer[n]; for(int i=0;i<n;i++) order[i]=i+1;
    Arrays.sort(order,(a,b)->Long.compare(dist[b],dist[a])); // process far to near
    for (int node:order) for(int[]e:adj.getOrDefault(node,List.of())) if(dist[e[0]]<dist[node]) ways[node]=(ways[node]+ways[e[0]])%MOD;
    return (int)ways[1];
}
```
**Key Insight:** Dijkstra from n for distances. Then DP in order of decreasing distance — paths go from higher dist to lower dist nodes.
**Complexity:** Time: O((V+E) log V) | Space: O(V+E)

---

### Dij13. Design Graph With Shortest Path Calculator (LC 2642) — Hard
**Description:** Design a graph that supports adding edges and finding shortest path between nodes.
**Example:**
```java
Graph g = new Graph(4, [[0,2,5],[0,1,2],[1,2,1],[3,0,3]]);
g.shortestPath(3, 2); // → 6
g.addEdge([1,3,4]);
g.shortestPath(3, 2); // → 6 (still same)
```
```java
class Graph {
    private Map<Integer,List<int[]>> adj = new HashMap<>();
    private int n;
    public Graph(int n, int[][] edges) {
        this.n = n;
        for (int[] e : edges) adj.computeIfAbsent(e[0],k->new ArrayList<>()).add(new int[]{e[1],e[2]});
    }
    public void addEdge(int[] edge) {
        adj.computeIfAbsent(edge[0],k->new ArrayList<>()).add(new int[]{edge[1],edge[2]});
    }
    public int shortestPath(int node1, int node2) {
        long[] dist=new long[n]; Arrays.fill(dist,Long.MAX_VALUE); dist[node1]=0;
        PriorityQueue<long[]> pq=new PriorityQueue<>((a,b)->Long.compare(a[0],b[0]));
        pq.offer(new long[]{0,node1});
        while (!pq.isEmpty()){long[]cur=pq.poll();long d=cur[0];int nd=(int)cur[1];if(d>dist[nd])continue;if(nd==node2)return(int)d;for(int[]e:adj.getOrDefault(nd,List.of())){long nnd=d+e[1];if(nnd<dist[e[0]]){dist[e[0]]=nnd;pq.offer(new long[]{nnd,e[0]});}}}
        return -1;
    }
}
```
**Key Insight:** Run Dijkstra fresh per query — since edges are only added (not removed), no need to invalidate cached shortest paths.
**Complexity:** Time: O((V+E) log V) per query | Space: O(V+E)

---

### Dij14. Reachable Nodes in Subdivided Graph (LC 882) — Hard
**Description:** Graph where each edge is subdivided with `cnt[i]` extra nodes. From node 0 with moves ≤ M, count reachable original + subdivided nodes.
**Example:**
```
Input:  edges=[[0,1,10],[0,2,1],[1,2,2]], maxMoves=6, n=3
Output: 13
```
```java
public int reachableNodes(int[][] edges, int maxMoves, int n) {
    Map<Integer,Map<Integer,Integer>> adj=new HashMap<>();
    for (int[]e:edges){adj.computeIfAbsent(e[0],k->new HashMap<>()).put(e[1],e[2]);adj.computeIfAbsent(e[1],k->new HashMap<>()).put(e[0],e[2]);}
    int[] dist=new int[n]; Arrays.fill(dist,Integer.MAX_VALUE); dist[0]=0;
    PriorityQueue<int[]> pq=new PriorityQueue<>((a,b)->a[0]-b[0]);
    pq.offer(new int[]{0,0});
    while (!pq.isEmpty()){int[]cur=pq.poll();int d=cur[0],node=cur[1];if(d>dist[node])continue;for(Map.Entry<Integer,Integer>e:adj.getOrDefault(node,Map.of()).entrySet()){int nd=d+e.getValue()+1;if(nd<dist[e.getKey()]){dist[e.getKey()]=nd;pq.offer(new int[]{nd,e.getKey()});}}}
    int ans=0;
    for (int i=0;i<n;i++) if(dist[i]<=maxMoves) ans++;
    for (int[]e:edges){int fromU=dist[e[0]]<=maxMoves?maxMoves-dist[e[0]]:0;int fromV=dist[e[1]]<=maxMoves?maxMoves-dist[e[1]]:0;ans+=Math.min(e[2],fromU+fromV);}
    return ans;
}
```
**Key Insight:** Dijkstra on original nodes. For each edge, count subdivided nodes reachable from both ends; capped at total subdivided nodes on that edge.
**Complexity:** Time: O((V+E) log V) | Space: O(V+E)

---

### Dij15. Minimum Weighted Subgraph With the Required Paths (LC 2203) — Hard
**Description:** Find min cost subgraph where paths from `src1→dest` and `src2→dest` both exist.
**Example:**
```
Input:  n=6, edges=[[0,2,2],[0,5,6],[1,0,3],[1,4,5],[2,1,1],[2,3,3],[2,3,4],[3,4,2],[4,5,1]], src1=0, src2=1, dest=5
Output: 9
```
```java
public long minimumWeight(int n, int[][] edges, int src1, int src2, int dest) {
    List<long[]>[] fwd=new ArrayList[n],rev=new ArrayList[n];
    for(int i=0;i<n;i++){fwd[i]=new ArrayList<>();rev[i]=new ArrayList<>();}
    for(int[]e:edges){fwd[e[0]].add(new long[]{e[1],e[2]});rev[e[1]].add(new long[]{e[0],e[2]});}
    long[] d1=dijkstra(n,fwd,src1),d2=dijkstra(n,fwd,src2),d3=dijkstra(n,rev,dest);
    long ans=Long.MAX_VALUE;
    for(int i=0;i<n;i++){if(d1[i]!=Long.MAX_VALUE&&d2[i]!=Long.MAX_VALUE&&d3[i]!=Long.MAX_VALUE) ans=Math.min(ans,d1[i]+d2[i]+d3[i]);}
    return ans==Long.MAX_VALUE?-1:ans;
}
private long[] dijkstra(int n,List<long[]>[] adj,int src){long[]dist=new long[n];Arrays.fill(dist,Long.MAX_VALUE);dist[src]=0;PriorityQueue<long[]>pq=new PriorityQueue<>((a,b)->Long.compare(a[0],b[0]));pq.offer(new long[]{0,src});while(!pq.isEmpty()){long[]cur=pq.poll();long d=cur[0];int nd=(int)cur[1];if(d>dist[nd])continue;for(long[]e:adj[nd]){long nnd=d+e[1];if(nnd<dist[(int)e[0]]){dist[(int)e[0]]=nnd;pq.offer(new long[]{nnd,e[0]});}}}return dist;}
```
**Key Insight:** Both paths must merge at some node `i`. Run Dijkstra from src1, src2, and backward from dest. Answer = min of `d1[i]+d2[i]+d3[i]` over all i.
**Complexity:** Time: O((V+E) log V) | Space: O(V+E)

---

### Dij16. Minimum Cost to Reach Destination in Time (LC 1928) — Hard
**Description:** Weighted graph with `fees[i]` per node. Max time `maxTime`. Minimize fees from 0 to n-1.
**Example:**
```
Input:  maxTime=30, edges=[[0,1,10],[1,2,10],[2,5,10],[0,3,1],[3,4,10],[4,5,15]], passingFees=[5,1,2,20,20,3]
Output: 11
```
```java
public int minCost(int maxTime, int[][] edges, int[] passingFees) {
    int n=passingFees.length;
    int[][] dp=new int[maxTime+1][n]; for(int[]r:dp) Arrays.fill(r,Integer.MAX_VALUE/2);
    dp[0][0]=passingFees[0];
    for(int t=1;t<=maxTime;t++){dp[t][0]=passingFees[0];for(int[]e:edges){int u=e[0],v=e[1],w=e[2];if(t>=w){if(dp[t-w][u]<Integer.MAX_VALUE/2) dp[t][v]=Math.min(dp[t][v],dp[t-w][u]+passingFees[v]);if(dp[t-w][v]<Integer.MAX_VALUE/2) dp[t][u]=Math.min(dp[t][u],dp[t-w][v]+passingFees[u]);}}}
    int ans=Integer.MAX_VALUE;
    for(int t=1;t<=maxTime;t++) ans=Math.min(ans,dp[t][n-1]);
    return ans==Integer.MAX_VALUE?-1:ans;
}
```
**Key Insight:** DP where `dp[t][v]` = min fee to reach node v in exactly time t. Iterate over time and relax edges.
**Complexity:** Time: O(T*E)  T=maxTime | Space: O(T*V)

---

### Dij17. Minimum Number of Flights to Reach Destination (bellman-ford variant)
**Description:** See LC 787 Cheapest Flights — Bellman-Ford with K relaxations is cleaner for "at most K stops":
```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    int[] prices = new int[n]; Arrays.fill(prices, Integer.MAX_VALUE); prices[src] = 0;
    for (int i = 0; i <= k; i++) {
        int[] temp = Arrays.copyOf(prices, n);
        for (int[] f : flights) {
            if (prices[f[0]] == Integer.MAX_VALUE) continue;
            temp[f[1]] = Math.min(temp[f[1]], prices[f[0]] + f[2]);
        }
        prices = temp;
    }
    return prices[dst] == Integer.MAX_VALUE ? -1 : prices[dst];
}
```
**Key Insight:** Bellman-Ford limited to k+1 rounds. Use temp copy to avoid using edges from same round (prevents using more than 1 edge per iteration).
**Complexity:** Time: O(K*E)  K=max stops | Space: O(V)

---

### Dij18. K Shortest Paths (Yen's / Modified Dijkstra concept)
**Description:** Find K-th shortest path from src to dst. (LC 2462 / interview variant)
```java
// Modified Dijkstra: allow visiting a node multiple times, up to K times
public int findKthSmallest(int n, int[][] edges, int src, int dst, int k) {
    Map<Integer,List<int[]>> adj=new HashMap<>();
    for(int[]e:edges){adj.computeIfAbsent(e[0],k2->new ArrayList<>()).add(new int[]{e[1],e[2]});}
    int[] count=new int[n];
    PriorityQueue<int[]> pq=new PriorityQueue<>((a,b)->a[0]-b[0]);
    pq.offer(new int[]{0,src});
    while (!pq.isEmpty()){int[]cur=pq.poll();int d=cur[0],node=cur[1];count[node]++;if(node==dst&&count[node]==k) return d;if(count[node]>k) continue;for(int[]e:adj.getOrDefault(node,List.of())) pq.offer(new int[]{d+e[1],e[0]});}
    return -1;
}
```
**Key Insight:** When a node is reached for the k-th time via Dijkstra, that distance = k-th shortest path to it. No visited check — allow revisits up to k times.
**Complexity:** Time: O(K*(V+E) log V) | Space: O(V+E)

---

### Dij19. Minimum Fuel Cost to Report to the Capital (LC 2477) — Medium
**Description:** Tree of cities. Groups travel together (car seats = `seats`). Find min fuel (edges traversed, with grouping).
**Example:**
```
Input:  roads=[[0,1],[0,2],[0,3]], seats=5
Output: 3  (each neighbor sends 1 representative to 0)
```
```java
public long minimumFuelCost(int[][] roads, int seats) {
    int n=roads.length+1;
    Map<Integer,List<Integer>> adj=new HashMap<>();
    for(int[]r:roads){adj.computeIfAbsent(r[0],k->new ArrayList<>()).add(r[1]);adj.computeIfAbsent(r[1],k->new ArrayList<>()).add(r[0]);}
    long[] ans={0};
    dfs(0,-1,adj,seats,ans);
    return ans[0];
}
private long dfs(int node,int parent,Map<Integer,List<Integer>> adj,int seats,long[]ans){
    long people=1;
    for(int child:adj.getOrDefault(node,List.of()))if(child!=parent){people+=dfs(child,node,adj,seats,ans);}
    if(node!=0) ans[0]+=(long)Math.ceil((double)people/seats);
    return people;
}
```
**Key Insight:** DFS post-order. Count people in subtree; fuel for this edge = ceil(people/seats). Accumulate fuel on the way up.
**Complexity:** Time: O(V+E) | Space: O(V+E)

---

### Dij20. Bus Routes (LC 815) — Hard
**Description:** Bus routes, each a circular loop. Find min buses to take from source to target stop.
**Example:**
```
Input:  routes=[[1,2,7],[3,6,7]], source=1, target=6
Output: 2
```
```java
public int numBusesToDestination(int[][] routes, int source, int target) {
    if (source==target) return 0;
    Map<Integer,List<Integer>> stopToRoutes=new HashMap<>();
    for(int r=0;r<routes.length;r++) for(int stop:routes[r]) stopToRoutes.computeIfAbsent(stop,k->new ArrayList<>()).add(r);
    Set<Integer> visitedStops=new HashSet<>(),visitedRoutes=new HashSet<>();
    Queue<Integer> q=new LinkedList<>(); q.offer(source); visitedStops.add(source);
    int buses=0;
    while(!q.isEmpty()){int size=q.size();buses++;for(int i=0;i<size;i++){int stop=q.poll();for(int route:stopToRoutes.getOrDefault(stop,List.of())){if(visitedRoutes.add(route)){for(int nextStop:routes[route]){if(nextStop==target) return buses;if(visitedStops.add(nextStop)) q.offer(nextStop);}}}}}
    return -1;
}
```
**Key Insight:** BFS on stops. Key: mark entire routes as visited (not just stops) to avoid re-processing the same route. Levels = number of buses taken.
**Complexity:** Time: O(stops + routes) | Space: O(stops + routes)


---

## 15. Union-Find — 20 Problems

### Union-Find Template
```java
class UnionFind {
    int[] parent, rank;
    int components;
    UnionFind(int n) { parent=new int[n]; rank=new int[n]; components=n; for(int i=0;i<n;i++) parent[i]=i; }
    int find(int x) { if(parent[x]!=x) parent[x]=find(parent[x]); return parent[x]; }
    boolean union(int x, int y) { int px=find(x),py=find(y); if(px==py) return false; if(rank[px]<rank[py]){int t=px;px=py;py=t;} parent[py]=px; if(rank[px]==rank[py]) rank[px]++; components--; return true; }
    boolean connected(int x,int y){return find(x)==find(y);}
}
```

---

### U1. Number of Provinces (LC 547) — Medium
**Description:** n cities, `isConnected[i][j]=1` means directly connected. Count provinces (connected components).
**Example:**
```
Input:  isConnected=[[1,1,0],[1,1,0],[0,0,1]]
Output: 2
```
```java
public int findCircleNum(int[][] isConnected) {
    int n=isConnected.length;
    UnionFind uf=new UnionFind(n);
    for(int i=0;i<n;i++) for(int j=i+1;j<n;j++) if(isConnected[i][j]==1) uf.union(i,j);
    return uf.components;
}
```
**Key Insight:** Each `union` reduces component count by 1. Final `components` = number of provinces.
**Complexity:** Time: O(n^2 * alpha(n)) | Space: O(n)

---

### U2. Redundant Connection (LC 684) — Medium
**Description:** Tree with one extra edge. Find the edge that if removed, restores a tree. Return last one if multiple.
**Example:**
```
Input:  edges=[[1,2],[1,3],[2,3]]
Output: [2,3]
```
```java
public int[] findRedundantConnection(int[][] edges) {
    UnionFind uf=new UnionFind(edges.length+1);
    for(int[] e:edges) if(!uf.union(e[0],e[1])) return e;
    return new int[0];
}
```
**Key Insight:** First edge whose two endpoints are already in the same component = the cycle-creating edge = redundant.
**Complexity:** Time: O(E * alpha(V)) | Space: O(V)

---

### U3. Accounts Merge (LC 721) — Medium
**Description:** List of accounts `[name, email1, email2, ...]`. Merge accounts sharing any email. Return merged sorted.
**Example:**
```
Input:  [["John","a@m","b@m"],["John","b@m","c@m"],["Mary","d@m"]]
Output: [["John","a@m","b@m","c@m"],["Mary","d@m"]]
```
```java
public List<List<String>> accountsMerge(List<List<String>> accounts) {
    Map<String,String> emailToName=new HashMap<>();
    Map<String,String> parent=new HashMap<>();
    for(List<String> acc:accounts){ String name=acc.get(0); for(int i=1;i<acc.size();i++){emailToName.put(acc.get(i),name);parent.putIfAbsent(acc.get(i),acc.get(i));} for(int i=2;i<acc.size();i++) union(parent,acc.get(1),acc.get(i)); }
    Map<String,TreeSet<String>> groups=new HashMap<>();
    for(String email:parent.keySet()) groups.computeIfAbsent(find(parent,email),k->new TreeSet<>()).add(email);
    List<List<String>> result=new ArrayList<>();
    for(Map.Entry<String,TreeSet<String>> e:groups.entrySet()){List<String> list=new ArrayList<>();list.add(emailToName.get(e.getKey()));list.addAll(e.getValue());result.add(list);}
    return result;
}
private String find(Map<String,String> parent,String x){if(!parent.get(x).equals(x)) parent.put(x,find(parent,parent.get(x)));return parent.get(x);}
private void union(Map<String,String> parent,String a,String b){String pa=find(parent,a),pb=find(parent,b);if(!pa.equals(pb)) parent.put(pa,pb);}
```
**Key Insight:** Union all emails within the same account. Group emails by root representative. Add name to each group.
**Complexity:** Time: O(n*m log(n*m)) | Space: O(n*m)  m=emails per account

---

### U4. Number of Operations to Make Network Connected (LC 1319) — Medium
**Description:** n computers, edges = cables. Find min cables to move to connect all. Return -1 if impossible.
**Example:**
```
Input:  n=4, connections=[[0,1],[0,2],[1,2]]
Output: 1
```
```java
public int makeConnected(int n, int[][] connections) {
    if(connections.length<n-1) return -1; // not enough cables
    UnionFind uf=new UnionFind(n);
    for(int[]c:connections) uf.union(c[0],c[1]);
    return uf.components-1; // need (components-1) cables to connect them
}
```
**Key Insight:** Need at least n-1 cables. If we have them, answer = components-1 (one cable bridges two components).
**Complexity:** Time: O(E * alpha(V)) | Space: O(V)

---

### U5. Satisfiability of Equality Equations (LC 990) — Medium
**Description:** Equations like `"a==b"` and `"a!=b"`. Return false if contradictory.
**Example:**
```
Input:  ["a==b","b!=c","b==c"]
Output: false
```
```java
public boolean equationsPossible(String[] equations) {
    UnionFind uf=new UnionFind(26);
    for(String eq:equations) if(eq.charAt(1)=='=') uf.union(eq.charAt(0)-'a',eq.charAt(3)-'a');
    for(String eq:equations) if(eq.charAt(1)=='!' && uf.connected(eq.charAt(0)-'a',eq.charAt(3)-'a')) return false;
    return true;
}
```
**Key Insight:** Two-pass — first union all equality pairs, then check inequality pairs. Order matters.
**Complexity:** Time: O(n * alpha(26)) | Space: O(1)

---

### U6. Most Stones Removed with Same Row or Column (LC 947) — Medium
**Description:** Stones on a 2D plane. Can remove a stone if another stone shares its row or column. Max removable?
**Example:**
```
Input:  stones=[[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]
Output: 5
```
```java
public int removeStones(int[][] stones) {
    Map<Integer,Integer> parent=new HashMap<>();
    for(int[]s:stones){union(parent,s[0],~s[1]);} // ~ to avoid row/col number clash
    Set<Integer> roots=new HashSet<>();
    for(int[]s:stones) roots.add(find(parent,s[0]));
    return stones.length-roots.size();
}
private int find(Map<Integer,Integer> p,int x){p.putIfAbsent(x,x);if(p.get(x)!=x) p.put(x,find(p,p.get(x)));return p.get(x);}
private void union(Map<Integer,Integer> p,int a,int b){int pa=find(p,a),pb=find(p,b);if(pa!=pb) p.put(pa,pb);}
```
**Key Insight:** Union all stones in the same row and column. `~col` distinguishes column IDs from row IDs. Max removable = total - components (keep 1 per component).
**Complexity:** Time: O(n * alpha(n)) | Space: O(n)

---

### U7. Smallest String With Swaps (LC 1202) — Medium
**Description:** Can swap chars at index pairs repeatedly. Find lexicographically smallest string.
**Example:**
```
Input:  s="dcab", pairs=[[0,3],[1,2]]
Output: "bacd"
```
```java
public String smallestStringWithSwaps(String s, List<List<Integer>> pairs) {
    int n=s.length();
    UnionFind uf=new UnionFind(n);
    for(List<Integer> p:pairs) uf.union(p.get(0),p.get(1));
    Map<Integer,PriorityQueue<Character>> groups=new HashMap<>();
    for(int i=0;i<n;i++) groups.computeIfAbsent(uf.find(i),k->new PriorityQueue<>()).add(s.charAt(i));
    StringBuilder sb=new StringBuilder();
    for(int i=0;i<n;i++) sb.append(groups.get(uf.find(i)).poll());
    return sb.toString();
}
```
**Key Insight:** Indices in same component can be freely rearranged. Sort each component's characters and assign in index order.
**Complexity:** Time: O((n+p) * alpha(n)) | Space: O(n)

---

### U8. Similar String Groups (LC 839) — Hard
**Description:** Two strings are similar if one swap makes them equal. Find number of groups.
**Example:**
```
Input:  strs=["tars","rats","arts","star"]
Output: 2
```
```java
public int numSimilarGroups(String[] strs) {
    int n=strs.length;
    UnionFind uf=new UnionFind(n);
    for(int i=0;i<n;i++) for(int j=i+1;j<n;j++) if(isSimilar(strs[i],strs[j])) uf.union(i,j);
    return uf.components;
}
private boolean isSimilar(String a,String b){int diff=0;for(int i=0;i<a.length();i++) if(a.charAt(i)!=b.charAt(i)) if(++diff>2) return false;return diff==0||diff==2;}
```
**Key Insight:** Check all pairs — O(n²·k). Two strings similar if they differ in exactly 0 or 2 positions. Union similar pairs, count components.
**Complexity:** Time: O(n^2 * k * alpha(n)) | Space: O(n)

---

### U9. Checking Existence of Edge Length Limited Paths (LC 1697) — Hard
**Description:** Offline queries: given threshold limit, does path exist using only edges with weight < limit?
**Example:**
```
Input:  n=3, edgeList=[[0,1,2],[1,2,4],[2,0,8],[1,0,16]], queries=[[0,1,2],[0,2,5]]
Output: [false,true]
```
```java
public boolean[] distanceLimitedPathsExist(int n, int[][] edgeList, int[][] queries) {
    Arrays.sort(edgeList,(a,b)->a[2]-b[2]);
    int q=queries.length;
    Integer[] idx=new Integer[q]; for(int i=0;i<q;i++) idx[i]=i;
    Arrays.sort(idx,(a,b)->queries[a][2]-queries[b][2]);
    UnionFind uf=new UnionFind(n);
    boolean[] ans=new boolean[q]; int ei=0;
    for(int qi:idx){int limit=queries[qi][2];while(ei<edgeList.length&&edgeList[ei][2]<limit) uf.union(edgeList[ei][0],edgeList[ei++][1]);ans[qi]=uf.connected(queries[qi][0],queries[qi][1]);}
    return ans;
}
```
**Key Insight:** Sort edges and queries by weight/limit. Process queries in order — add all edges below the current limit, then check connectivity. Offline trick avoids recomputing.
**Complexity:** Time: O((E+Q) log(E+Q)) | Space: O(V+Q)

---

### U10. Number of Good Paths (LC 2421) — Hard
**Description:** Tree, each node has a value. Good path: start/end nodes same value, all intermediate nodes ≤ that value. Count good paths.
**Example:**
```
Input:  vals=[1,3,2,1,3], edges=[[0,1],[0,2],[2,3],[2,4]]
Output: 6
```
```java
public int numberOfGoodPaths(int[] vals, int[][] edges) {
    int n=vals.length;
    Map<Integer,List<Integer>> adj=new HashMap<>();
    for(int[]e:edges){adj.computeIfAbsent(e[0],k->new ArrayList<>()).add(e[1]);adj.computeIfAbsent(e[1],k->new ArrayList<>()).add(e[0]);}
    UnionFind uf=new UnionFind(n);
    Map<Integer,Integer> maxVal=new HashMap<>(); // root -> max val in component
    for(int i=0;i<n;i++) maxVal.put(i,vals[i]);
    Map<Integer,List<Integer>> count=new HashMap<>(); // root -> nodes with maxVal
    for(int i=0;i<n;i++) count.computeIfAbsent(i,k->new ArrayList<>()).add(i);
    // Sort nodes by value, process in increasing order
    Integer[]nodes=new Integer[n]; for(int i=0;i<n;i++) nodes[i]=i;
    Arrays.sort(nodes,(a,b)->vals[a]-vals[b]);
    int ans=n; // each node alone
    for(int node:nodes){for(int nei:adj.getOrDefault(node,List.of())){if(vals[nei]<=vals[node]){int rn=uf.find(node),rni=uf.find(nei);if(rn!=rni){if(maxVal.get(rn).equals(maxVal.get(rni))){ans+=count.get(rn).size()*(long)count.get(rni).size();}uf.union(node,nei);int nr=uf.find(node);maxVal.put(nr,Math.max(maxVal.getOrDefault(rn,0),maxVal.getOrDefault(rni,0)));List<Integer>merged=new ArrayList<>(count.get(rn));merged.addAll(count.get(rni));count.put(nr,merged);}}}}
    return ans;
}
```
**Key Insight:** Process nodes in increasing value order. When merging two components with the same max value, they contribute `size1 × size2` new good paths.
**Complexity:** Time: O((V+E) * alpha(V)) | Space: O(V+E)

---

### U11. Count Unreachable Pairs of Nodes (LC 2316) — Medium
**Description:** Count pairs of nodes that cannot reach each other.
**Example:**
```
Input:  n=7, edges=[[0,2],[0,5],[2,4]]
Output: 14
```
```java
public long countPairs(int n, int[][] edges) {
    UnionFind uf=new UnionFind(n);
    for(int[]e:edges) uf.union(e[0],e[1]);
    Map<Integer,Long> compSize=new HashMap<>();
    for(int i=0;i<n;i++) compSize.merge(uf.find(i),1L,Long::sum);
    long ans=0,seen=0;
    for(long size:compSize.values()){ans+=size*seen;seen+=size;}
    return ans;
}
```
**Key Insight:** For each component of size `s`, it contributes `s × (all previously seen nodes)` unreachable pairs. Iterate components once.
**Complexity:** Time: O(E * alpha(V)) | Space: O(V)

---

### U12. Find All People With Secret (LC 2092) — Hard
**Description:** At time 0, person 0 and `firstPerson` share a secret. Meetings happen; if both attendees know the secret, they share it. Return all who know the secret after all meetings.
**Example:**
```
Input:  n=6, meetings=[[1,2,5],[2,3,8],[1,5,10]], firstPerson=1
Output: [0,1,2,3,5]
```
```java
public List<Integer> findAllPeople(int n, int[][] meetings, int firstPerson) {
    Arrays.sort(meetings,(a,b)->a[2]-b[2]);
    UnionFind uf=new UnionFind(n); uf.union(0,firstPerson);
    int i=0,m=meetings.length;
    while(i<m){int j=i,t=meetings[i][2];while(j<m&&meetings[j][2]==t){uf.union(meetings[j][0],meetings[j][1]);j++;}for(int k=i;k<j;k++){if(!uf.connected(meetings[k][0],0)){uf.parent[meetings[k][0]]=meetings[k][0];uf.parent[meetings[k][1]]=meetings[k][1];}}i=j;}
    List<Integer> res=new ArrayList<>(); for(int p=0;p<n;p++) if(uf.connected(p,0)) res.add(p);
    return res;
}
```
**Key Insight:** Process same-time meetings together. After each batch, disconnect people NOT connected to person 0 (they didn't learn the secret in this round).
**Complexity:** Time: O(m log m + m * alpha(n)) | Space: O(n+m)

---

### U13. Redundant Connection II (LC 685) — Hard
**Description:** Directed graph that was a rooted tree + one extra edge. Find the edge to remove.
**Example:**
```
Input:  edges=[[1,2],[1,3],[2,3]]
Output: [2,3]
```
```java
public int[] findRedundantDirectedConnection(int[][] edges) {
    int n=edges.length;
    int[] parent=new int[n+1]; Arrays.fill(parent,-1);
    int[] cand1=null,cand2=null;
    for(int[]e:edges){if(parent[e[1]]==-1) parent[e[1]]=e[0];else{cand1=new int[]{parent[e[1]],e[1]};cand2=new int[]{e[0],e[1]};e[1]=0;}}
    int[] uf=new int[n+1]; for(int i=0;i<=n;i++) uf[i]=i;
    for(int[]e:edges){if(e[1]==0) continue;int pu=find(uf,e[0]),pv=find(uf,e[1]);if(pu==pv) return cand1==null?e:cand1;uf[pu]=pv;}
    return cand2;
}
private int find(int[]p,int x){return p[x]==x?x:(p[x]=find(p,p[x]));}
```
**Key Insight:** Two cases: (1) node with two parents (take the second parent edge), (2) cycle (take the cycle edge). If both, the second parent edge is in the cycle → return cand1.
**Complexity:** Time: O(E * alpha(V)) | Space: O(V)

---

### U14. GCD Sort of an Array (LC 1998) — Hard
**Description:** Can swap `nums[i]` and `nums[j]` if `gcd(nums[i],nums[j])>1`. Return true if array can be sorted.
**Example:**
```
Input:  nums=[7,21,3]
Output: true  (swap 7 and 21 since gcd=7)
```
```java
public boolean gcdSort(int[] nums) {
    int max=Arrays.stream(nums).max().getAsInt();
    int[]parent=new int[max+1]; for(int i=0;i<=max;i++) parent[i]=i;
    // Sieve: union each number with its prime factors' "representative"
    int[]spf=new int[max+1]; for(int i=0;i<=max;i++) spf[i]=i;
    for(int i=2;i*i<=max;i++) if(spf[i]==i) for(int j=i*i;j<=max;j+=i) if(spf[j]==j) spf[j]=i;
    for(int n:nums){int x=n;while(x>1){int p=spf[x];union(parent,n,p);x/=p;}}
    int[]sorted=nums.clone(); Arrays.sort(sorted);
    for(int i=0;i<nums.length;i++) if(find(parent,nums[i])!=find(parent,sorted[i])) return false;
    return true;
}
private int find(int[]p,int x){return p[x]==x?x:(p[x]=find(p,p[x]));}
private void union(int[]p,int a,int b){p[find(p,a)]=find(p,b);}
```
**Key Insight:** Use sieve to union each number with its prime factors. Numbers sharing any prime can be in the same component (transitively swappable).
**Complexity:** Time: O(n*sqrt(M) + n log n)  M=max val | Space: O(M)

---

### U15. Minimize Hamming Distance After Swap Operations (LC 1722) — Medium
**Description:** Can swap `source[i]` and `source[j]` (same allowedSwaps component). Minimize sum of differing positions.
**Example:**
```
Input:  source=[1,2,3,4], target=[2,1,4,3], allowedSwaps=[[0,1],[2,3]]
Output: 0
```
```java
public int minimumHammingDistance(int[] source, int[] target, int[][] allowedSwaps) {
    int n=source.length;
    UnionFind uf=new UnionFind(n);
    for(int[]s:allowedSwaps) uf.union(s[0],s[1]);
    Map<Integer,Map<Integer,Integer>> groups=new HashMap<>();
    for(int i=0;i<n;i++){int root=uf.find(i);groups.computeIfAbsent(root,k->new HashMap<>()).merge(source[i],1,Integer::sum);}
    int dist=0;
    for(int i=0;i<n;i++){int root=uf.find(i);Map<Integer,Integer> pool=groups.get(root);if(pool.getOrDefault(target[i],0)>0) pool.merge(target[i],-1,Integer::sum);else dist++;}
    return dist;
}
```
**Key Insight:** Group source elements by connected component. For each target element, try to match it from the same component's source pool. Unmatched = Hamming distance.
**Complexity:** Time: O((n+p) * alpha(n)) | Space: O(n)

---

### U16. Lexicographically Smallest Equivalent String (LC 1061) — Medium
**Description:** `s1[i]` and `s2[i]` are equivalent. Use equivalences to make `baseStr` lexicographically smallest.
**Example:**
```
Input:  s1="parker",s2="morris",baseStr="parser"
Output: "makkek"
```
```java
public String smallestEquivalentString(String s1, String s2, String baseStr) {
    int[]parent=new int[26]; for(int i=0;i<26;i++) parent[i]=i;
    for(int i=0;i<s1.length();i++) union(parent,s1.charAt(i)-'a',s2.charAt(i)-'a');
    StringBuilder sb=new StringBuilder();
    for(char c:baseStr.toCharArray()) sb.append((char)('a'+find(parent,c-'a')));
    return sb.toString();
}
private int find(int[]p,int x){return p[x]==x?x:(p[x]=find(p,p[x]));}
private void union(int[]p,int a,int b){int pa=find(p,a),pb=find(p,b);if(pa<pb) p[pb]=pa;else p[pa]=pb;} // smaller = root
```
**Key Insight:** Always make the smaller character the root. When `find(c)` returns the root, it's the lexicographically smallest equivalent.
**Complexity:** Time: O(n * alpha(26)) | Space: O(1)

---

### U17. Minimum Cost to Connect All Points — Kruskal's MST (LC 1584) — Medium
**Description:** Find minimum cost to connect all points using Manhattan distance edges (alternative to Prim's).
```java
public int minCostConnectPoints(int[][] points) {
    int n=points.length;
    List<int[]> edges=new ArrayList<>(); // {cost, i, j}
    for(int i=0;i<n;i++) for(int j=i+1;j<n;j++) edges.add(new int[]{Math.abs(points[i][0]-points[j][0])+Math.abs(points[i][1]-points[j][1]),i,j});
    edges.sort((a,b)->a[0]-b[0]);
    UnionFind uf=new UnionFind(n);
    int cost=0,edges_used=0;
    for(int[]e:edges){if(uf.union(e[1],e[2])){cost+=e[0];if(++edges_used==n-1) break;}}
    return cost;
}
```
**Key Insight:** Kruskal's: sort all edges, greedily pick cheapest that doesn't form a cycle. Stop after n-1 edges (MST complete).
**Complexity:** Time: O(n^2 log n) | Space: O(n^2)

---

### U18. Number of Islands II (LC 305) — Hard (Premium)
**Description:** Add land cells one by one; after each, return number of islands.
**Example:**
```
Input:  m=3,n=3, positions=[[0,0],[0,1],[1,2],[2,1]]
Output: [1,1,2,3]
```
```java
public List<Integer> numIslands2(int m, int n, int[][] positions) {
    UnionFind uf=new UnionFind(m*n);
    boolean[]land=new boolean[m*n];
    int[][]dirs={{0,1},{0,-1},{1,0},{-1,0}};
    List<Integer> result=new ArrayList<>();
    int islands=0;
    for(int[]pos:positions){int r=pos[0],c=pos[1],idx=r*n+c;if(!land[idx]){land[idx]=true;islands++;uf.components++;for(int[]d:dirs){int nr=r+d[0],nc=c+d[1];if(nr>=0&&nr<m&&nc>=0&&nc<n&&land[nr*n+nc]) if(uf.union(idx,nr*n+nc)) islands--;}}result.add(islands);}
    return result;
}
```
**Key Insight:** Each new land cell starts as a new island (+1). For each adjacent land cell, if in different component, union and decrease count.
**Complexity:** Time: O(k * alpha(mn)) | Space: O(mn)

---

### U19. Sentence Similarity II (LC 737) — Medium (Premium)
**Description:** Two words are similar if they are the same or connected through a chain of similar pairs.
**Example:**
```
Input:  sentence1=["great","acting","skills"], sentence2=["fine","drama","talent"], similarPairs=[["great","fine"],["drama","acting"],["skills","talent"]]
Output: true
```
```java
public boolean areSentencesSimilarTwo(String[] s1, String[] s2, List<List<String>> pairs) {
    if(s1.length!=s2.length) return false;
    Map<String,String> parent=new HashMap<>();
    for(List<String> p:pairs){parent.putIfAbsent(p.get(0),p.get(0));parent.putIfAbsent(p.get(1),p.get(1));String pa=find(parent,p.get(0)),pb=find(parent,p.get(1));if(!pa.equals(pb)) parent.put(pa,pb);}
    for(int i=0;i<s1.length;i++){if(s1[i].equals(s2[i])) continue;parent.putIfAbsent(s1[i],s1[i]);parent.putIfAbsent(s2[i],s2[i]);if(!find(parent,s1[i]).equals(find(parent,s2[i]))) return false;}
    return true;
}
private String find(Map<String,String> p,String x){if(!p.get(x).equals(x)) p.put(x,find(p,p.get(x)));return p.get(x);}
```

---

### U20. Earliest Moment When Everyone Become Friends (LC 1101) — Medium (Premium)
**Description:** Friendship events sorted by time. Find earliest time when all n people are friends.
**Example:**
```
Input:  n=6, logs=[[20190101,0,1],[20190104,3,4],[20190107,2,3],[20190211,1,5],[20190224,2,4],[20190301,0,3],[20190312,1,2],[20190322,4,5]]
Output: 20190301
```
```java
public int earliestAcq(int[][] logs, int n) {
    Arrays.sort(logs,(a,b)->a[0]-b[0]);
    UnionFind uf=new UnionFind(n);
    for(int[]log:logs){uf.union(log[1],log[2]);if(uf.components==1) return log[0];}
    return -1;
}
```
**Key Insight:** Sort by timestamp. Union friends; when `components == 1`, everyone is connected — return current timestamp.
**Complexity:** Time: O(n log n + n * alpha(n)) | Space: O(n)

