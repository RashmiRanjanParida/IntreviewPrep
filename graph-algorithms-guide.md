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
