# Backtracking — Complete Guide for Google L6

---

## What is Backtracking?

Backtracking = **brute force with pruning**. Build a solution incrementally, undo your last choice when it leads to a dead end.

```
Make a choice
  → Recurse
  → Undo the choice (backtrack)
```

---

## How to Identify Backtracking Problems

| Signal | Example |
|---|---|
| "find all combinations/subsets/permutations" | Subsets, Permutations |
| "find all valid configurations" | N-Queens, Sudoku |
| "find if path exists" | Word Search |
| "generate all possible X" | Generate Parentheses |
| Exponential search space, need all solutions | Any of the above |

---

## The 4 Decisions (Ask Before Writing Code)

```
1. When to add to result?     → every node OR leaf only
2. Reuse element?             → i (reuse) OR i+1 (no reuse)
3. Duplicates in input?       → sort + skip condition
4. Any pruning condition?     → guard before/inside loop
```

---

## The 4 Backtracking Families

| Family | Reuse? | Order matters? | Visited tracking | Example |
|---|---|---|---|---|
| **Subsets** | No | No | `start` index | Subsets, Subsets II |
| **Combinations** | Sometimes | No | `start` index | Combination Sum |
| **Permutations** | No | Yes | `visited[]` array | Permutations |
| **Board/Grid** | No | Yes | Mark board `'#'` or sets | N-Queens, Word Search |

---

## Decision Table

| Problem | When to add | Loop start | Reuse | Duplicates |
|---|---|---|---|---|
| Subsets | Every node | `i=start` | No (`i+1`) | No |
| Subsets II | Every node | `i=start` | No (`i+1`) | Sort + `i>start&&nums[i]==nums[i-1]` |
| Combination Sum | Leaf (`sum==target`) | `i=start` | Yes (`i`) | No |
| Combination Sum II | Leaf (`sum==target`) | `i=start` | No (`i+1`) | Sort + `i>start&&nums[i]==nums[i-1]` |
| Permutations | Leaf (`size==n`) | `i=0` | No | No |
| Permutations II | Leaf (`size==n`) | `i=0` | No | Sort + `!visited[i-1]` |
| N-Queens | Leaf (`row==n`) | `col=0` | No | 3 sets (col, diag1, diag2) |
| Word Search | Leaf (`index==n`) | 4 dirs | No | Mark `'#'` on board |

---

## Universal Template

```java
void backtrack(int[] nums, int start, List<Integer> curr,
               List<List<Integer>> result, int remaining) {

    // Decision 1: when to add
    result.add(new ArrayList<>(curr));        // subsets: every node
    // OR
    if (remaining == 0) {                     // combinations: leaf only
        result.add(new ArrayList<>(curr));
        return;
    }
    if (remaining < 0) return;               // Decision 4: prune

    for (int i = start; i < nums.length; i++) {

        // Decision 3: skip duplicates at same level
        if (i > start && nums[i] == nums[i-1]) continue;

        curr.add(nums[i]);
        backtrack(nums, i + 1, curr, result, remaining - nums[i]); // Decision 2: i+1 no reuse
        // OR          i                                            // Decision 2: i reuse
        curr.remove(curr.size() - 1);
    }
}
```

---

## The Mental Checklist

```
Before writing backtrack():

□ What does currPath represent?
□ When is a solution complete? → base case
□ What are my choices? → loop range
□ Can I reuse? → i or i+1
□ Duplicates? → sort + skip condition
□ Can I prune early? → guard before/inside loop
□ What do I undo? → remove last element / restore cell
```

---

---

# 1. Subsets Family

## Pattern
- Add to result at **every node** (not just leaf)
- Use `start` index to avoid going backward
- No reuse: pass `i+1`

## Skeleton
```java
void backtrack(int[] nums, int start, List<Integer> curr, List<List<Integer>> result) {
    result.add(new ArrayList<>(curr));  // add at every node
    for (int i = start; i < nums.length; i++) {
        curr.add(nums[i]);
        backtrack(nums, i + 1, curr, result);
        curr.remove(curr.size() - 1);
    }
}
```

---

## Problems

### 1. Subsets (LC 78)
**Description:** Given an integer array of distinct elements, return all possible subsets (the power set). The solution must not contain duplicate subsets.

**Example:**
- Input: nums = [1,2,3]
- Output: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }

    void backtrack(int[] nums, int start, List<Integer> curr, List<List<Integer>> result) {
        result.add(new ArrayList<>(curr));
        for (int i = start; i < nums.length; i++) {
            curr.add(nums[i]);
            backtrack(nums, i + 1, curr, result);
            curr.remove(curr.size() - 1);
        }
    }
}
```

---

### 2. Subsets II (LC 90)
**Description:** Given an integer array that may contain duplicates, return all possible subsets. The solution set must not contain duplicate subsets.

**Example:**
- Input: nums = [1,2,2]
- Output: [[],[1],[1,2],[1,2,2],[2],[2,2]]  (no duplicate [1,2] pairs)

```java
class Solution {
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }

    void backtrack(int[] nums, int start, List<Integer> curr, List<List<Integer>> result) {
        result.add(new ArrayList<>(curr));
        for (int i = start; i < nums.length; i++) {
            if (i > start && nums[i] == nums[i-1]) continue;  // skip dups at same level
            curr.add(nums[i]);
            backtrack(nums, i + 1, curr, result);
            curr.remove(curr.size() - 1);
        }
    }
}
```

**Key**: `i > start` not `i > 0` — only skip duplicates at same recursion level.

---

### 3. Letter Case Permutation (LC 784)
**Description:** Given a string s, transform every letter individually to be lowercase or uppercase. Return a list of all possible strings.

**Example:**
- Input: s = "a1b2"
- Output: ["a1b2","a1B2","A1b2","A1B2"]

```java
class Solution {
    public List<String> letterCasePermutation(String s) {
        List<String> result = new ArrayList<>();
        backtrack(s.toCharArray(), 0, result);
        return result;
    }

    void backtrack(char[] arr, int i, List<String> result) {
        if (i == arr.length) { result.add(new String(arr)); return; }
        backtrack(arr, i + 1, result);  // keep as is
        if (Character.isLetter(arr[i])) {
            arr[i] ^= 32;               // toggle case (ASCII trick)
            backtrack(arr, i + 1, result);
            arr[i] ^= 32;               // restore
        }
    }
}
```

---

### 4. Power Set (Classic)
**Description:** Return all 2ⁿ subsets of a given set of distinct integers, including the empty set and the full set itself.

**Example:**
- Input: nums = [1,2,3]
- Output: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]  (8 = 2³ subsets)

```java
public List<List<Integer>> powerSet(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, int start, List<Integer> curr, List<List<Integer>> result) {
    result.add(new ArrayList<>(curr));
    for (int i = start; i < nums.length; i++) {
        curr.add(nums[i]);
        backtrack(nums, i + 1, curr, result);
        curr.remove(curr.size() - 1);
    }
}
```

---

### 5. Find All Subsets of Size K (Classic)
**Description:** Generate all subsets of exactly size k from a given array of distinct integers.

**Example:**
- Input: nums = [1,2,3,4], k = 2
- Output: [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]  (C(4,2) = 6 subsets)

```java
public List<List<Integer>> subsetsOfSizeK(int[] nums, int k) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, k, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, int start, int k, List<Integer> curr, List<List<Integer>> result) {
    if (curr.size() == k) { result.add(new ArrayList<>(curr)); return; }
    for (int i = start; i < nums.length; i++) {
        curr.add(nums[i]);
        backtrack(nums, i + 1, k, curr, result);
        curr.remove(curr.size() - 1);
    }
}
```

---

---

# 2. Combinations Family

## Pattern
- Add to result only at **leaf** (when condition met)
- Use `start` index to avoid going backward
- Prune when sum exceeds target
- Reuse: pass `i` / No reuse: pass `i+1`

## Skeleton
```java
void backtrack(int[] nums, int start, List<Integer> curr,
               List<List<Integer>> result, int remaining) {
    if (remaining == 0) { result.add(new ArrayList<>(curr)); return; }
    if (remaining < 0) return;  // prune
    for (int i = start; i < nums.length; i++) {
        curr.add(nums[i]);
        backtrack(nums, i + 1, curr, result, remaining - nums[i]); // i for reuse
        curr.remove(curr.size() - 1);
    }
}
```

---

## Problems

### 1. Combination Sum (LC 39)
**Description:** Given an array of distinct integers and a target, find all unique combinations that sum to target. The same number may be used an unlimited number of times.

**Example:**
- Input: candidates = [2,3,6,7], target = 7
- Output: [[2,2,3],[7]]  (2+2+3=7, 7=7; note 2 can be reused)

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(candidates, 0, new ArrayList<>(), result, target);
        return result;
    }

    void backtrack(int[] nums, int start, List<Integer> curr,
                   List<List<Integer>> result, int remaining) {
        if (remaining == 0) { result.add(new ArrayList<>(curr)); return; }
        if (remaining < 0) return;
        for (int i = start; i < nums.length; i++) {
            curr.add(nums[i]);
            backtrack(nums, i, curr, result, remaining - nums[i]);  // i = reuse
            curr.remove(curr.size() - 1);
        }
    }
}
```

---

### 2. Combination Sum II (LC 40)
**Description:** Given a collection of candidates (may contain duplicates) and a target, find all unique combinations that sum to target. Each candidate may only be used once.

**Example:**
- Input: candidates = [10,1,2,7,6,1,5], target = 8
- Output: [[1,1,6],[1,2,5],[1,7],[2,6]]  (no duplicate combinations)

```java
class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> result = new ArrayList<>();
        backtrack(candidates, 0, new ArrayList<>(), result, target);
        return result;
    }

    void backtrack(int[] nums, int start, List<Integer> curr,
                   List<List<Integer>> result, int remaining) {
        if (remaining == 0) { result.add(new ArrayList<>(curr)); return; }
        if (remaining < 0) return;
        for (int i = start; i < nums.length; i++) {
            if (i > start && nums[i] == nums[i-1]) continue;  // skip dups
            curr.add(nums[i]);
            backtrack(nums, i + 1, curr, result, remaining - nums[i]);  // i+1 = no reuse
            curr.remove(curr.size() - 1);
        }
    }
}
```

---

### 3. Combination Sum III (LC 216)
**Description:** Find all valid combinations of k numbers that sum to n. Use only numbers 1-9, each number used at most once. Return all possible valid combinations.

**Example:**
- Input: k = 3, n = 7
- Output: [[1,2,4]]  (1+2+4=7, only one combination using 3 distinct digits 1-9)

```java
class Solution {
    public List<List<Integer>> combinationSum3(int k, int n) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(1, k, n, new ArrayList<>(), result);
        return result;
    }

    void backtrack(int start, int k, int remaining, List<Integer> curr,
                   List<List<Integer>> result) {
        if (curr.size() == k && remaining == 0) {
            result.add(new ArrayList<>(curr)); return;
        }
        if (curr.size() == k || remaining < 0) return;
        for (int i = start; i <= 9; i++) {
            curr.add(i);
            backtrack(i + 1, k, remaining - i, curr, result);
            curr.remove(curr.size() - 1);
        }
    }
}
```

---

### 4. Generate Parentheses (LC 22)
**Description:** Given n pairs of parentheses, generate all combinations of well-formed (valid) parentheses strings.

**Example:**
- Input: n = 3
- Output: ["((()))","(()())","(())()","()(())","()()()"]

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        backtrack(n, 0, 0, new StringBuilder(), result);
        return result;
    }

    void backtrack(int n, int open, int close, StringBuilder curr, List<String> result) {
        if (curr.length() == 2 * n) { result.add(curr.toString()); return; }
        if (open < n) {
            curr.append('(');
            backtrack(n, open + 1, close, curr, result);
            curr.deleteCharAt(curr.length() - 1);
        }
        if (close < open) {
            curr.append(')');
            backtrack(n, open, close + 1, curr, result);
            curr.deleteCharAt(curr.length() - 1);
        }
    }
}
```

**Pruning**: only add `(` if open < n, only add `)` if close < open.

---

### 5. Combinations (LC 77)
**Description:** Given two integers n and k, return all possible combinations of k numbers chosen from the range [1, n].

**Example:**
- Input: n = 4, k = 2
- Output: [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]

```java
class Solution {
    public List<List<Integer>> combine(int n, int k) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(1, n, k, new ArrayList<>(), result);
        return result;
    }

    void backtrack(int start, int n, int k, List<Integer> curr, List<List<Integer>> result) {
        if (curr.size() == k) { result.add(new ArrayList<>(curr)); return; }
        for (int i = start; i <= n; i++) {
            curr.add(i);
            backtrack(i + 1, n, k, curr, result);
            curr.remove(curr.size() - 1);
        }
    }
}
```

---

---

# 3. Permutations Family

## Pattern
- Add to result at **leaf** (`curr.size() == n`)
- Loop from 0 every time (order matters)
- Track used elements with `visited[]`
- No `start` index

## Skeleton
```java
void backtrack(int[] nums, boolean[] visited, List<Integer> curr,
               List<List<Integer>> result) {
    if (curr.size() == nums.length) {
        result.add(new ArrayList<>(curr));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (visited[i]) continue;
        visited[i] = true;
        curr.add(nums[i]);
        backtrack(nums, visited, curr, result);
        curr.remove(curr.size() - 1);
        visited[i] = false;
    }
}
```

---

## Duplicate Condition — Permutations II

```
Subsets/Combos:   i > start && nums[i] == nums[i-1]
Permutations II:  i > 0 && nums[i] == nums[i-1] && !visited[i-1]

!visited[i-1] means:
  nums[i-1] was used and backtracked at this level
  → trying nums[i] (same value) would produce duplicate
  → skip
```

---

## Problems

### 1. Permutations (LC 46)
**Description:** Given an array of distinct integers, return all possible permutations in any order.

**Example:**
- Input: nums = [1,2,3]
- Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]  (3! = 6 permutations)

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, new boolean[nums.length], new ArrayList<>(), result);
        return result;
    }

    void backtrack(int[] nums, boolean[] visited, List<Integer> curr,
                   List<List<Integer>> result) {
        if (curr.size() == nums.length) {
            result.add(new ArrayList<>(curr)); return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (visited[i]) continue;
            visited[i] = true;
            curr.add(nums[i]);
            backtrack(nums, visited, curr, result);
            curr.remove(curr.size() - 1);
            visited[i] = false;
        }
    }
}
```

---

### 2. Permutations II (LC 47)
**Description:** Given a collection of numbers that might contain duplicates, return all possible unique permutations in any order.

**Example:**
- Input: nums = [1,1,2]
- Output: [[1,1,2],[1,2,1],[2,1,1]]  (3 unique permutations, not 3! = 6)

```java
class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, new boolean[nums.length], new ArrayList<>(), result);
        return result;
    }

    void backtrack(int[] nums, boolean[] visited, List<Integer> curr,
                   List<List<Integer>> result) {
        if (curr.size() == nums.length) {
            result.add(new ArrayList<>(curr)); return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (visited[i]) continue;
            if (i > 0 && nums[i] == nums[i-1] && !visited[i-1]) continue;  // skip dups
            visited[i] = true;
            curr.add(nums[i]);
            backtrack(nums, visited, curr, result);
            curr.remove(curr.size() - 1);
            visited[i] = false;
        }
    }
}
```

---

### 3. Next Permutation (LC 31)
**Description:** Given an array of integers, rearrange them into the next lexicographically greater permutation. If no such arrangement exists, rearrange to lowest order (sorted ascending). Must be done in-place.

**Example:**
- Input: nums = [1,2,3] → Output: [1,3,2]
- Input: nums = [3,2,1] → Output: [1,2,3]  (no next, wrap to smallest)
- Input: nums = [1,1,5] → Output: [1,5,1]

```java
class Solution {
    public void nextPermutation(int[] nums) {
        int n = nums.length, i = n - 2;
        while (i >= 0 && nums[i] >= nums[i+1]) i--;
        if (i >= 0) {
            int j = n - 1;
            while (nums[j] <= nums[i]) j--;
            int tmp = nums[i]; nums[i] = nums[j]; nums[j] = tmp;
        }
        // reverse from i+1 to end
        int lo = i + 1, hi = n - 1;
        while (lo < hi) {
            int tmp = nums[lo]; nums[lo++] = nums[hi]; nums[hi--] = tmp;
        }
    }
}
```

---

### 4. Palindrome Permutation II (LC 267)
**Description:** Given a string s, return all palindromic permutations of it. If no palindromic permutation exists, return an empty list.

**Example:**
- Input: s = "aabb"
- Output: ["abba","baab"]  (two palindromes formed from 'a','a','b','b')

```java
class Solution {
    public List<String> generatePalindromes(String s) {
        int[] freq = new int[26];
        for (char c : s.toCharArray()) freq[c - 'a']++;
        String mid = "";
        List<Character> half = new ArrayList<>();
        for (int i = 0; i < 26; i++) {
            if (freq[i] % 2 != 0) { mid += (char)('a' + i); }
            for (int j = 0; j < freq[i] / 2; j++) half.add((char)('a' + i));
        }
        if (mid.length() > 1) return new ArrayList<>();
        List<String> result = new ArrayList<>();
        String midStr = mid;
        backtrack(half, new boolean[half.size()], new StringBuilder(), midStr, result);
        return result;
    }

    void backtrack(List<Character> half, boolean[] visited, StringBuilder curr,
                   String mid, List<String> result) {
        if (curr.length() == half.size()) {
            String s = curr.toString();
            result.add(s + mid + new StringBuilder(s).reverse());
            return;
        }
        for (int i = 0; i < half.size(); i++) {
            if (visited[i]) continue;
            if (i > 0 && half.get(i) == half.get(i-1) && !visited[i-1]) continue;
            visited[i] = true;
            curr.append(half.get(i));
            backtrack(half, visited, curr, mid, result);
            curr.deleteCharAt(curr.length() - 1);
            visited[i] = false;
        }
    }
}
```

---

### 5. Sequence Reconstruction (LC 444)
**Description:** Given a sequence nums (a permutation of 1..n) and a list of subsequences, check whether nums is the only sequence that can be reconstructed from the subsequences. Reconstruction means finding the shortest common supersequence.

**Example:**
- Input: nums = [1,2,3], sequences = [[1,2],[1,3]]
- Output: false  (both [1,2,3] and [1,3,2] can be reconstructed)
- Input: nums = [4,1,5,2,6,3], sequences = [[5,2,6,3],[4,1,5,2]]
- Output: true  (only one valid topological order)

```java
class Solution {
    public boolean sequenceReconstruction(int[] nums, int[][] sequences) {
        int n = nums.length;
        int[] inDegree = new int[n + 1];
        Set<Integer>[] graph = new Set[n + 1];
        for (int i = 1; i <= n; i++) graph[i] = new HashSet<>();
        for (int[] seq : sequences)
            for (int i = 0; i < seq.length - 1; i++)
                if (graph[seq[i]].add(seq[i+1])) inDegree[seq[i+1]]++;
        Queue<Integer> queue = new LinkedList<>();
        for (int i = 1; i <= n; i++) if (inDegree[i] == 0) queue.offer(i);
        int idx = 0;
        while (!queue.isEmpty()) {
            if (queue.size() > 1) return false;  // not unique
            int node = queue.poll();
            if (nums[idx++] != node) return false;
            for (int next : graph[node])
                if (--inDegree[next] == 0) queue.offer(next);
        }
        return idx == n;
    }
}
```

---

---

# 4. Board / Grid Family

## Pattern
- Choices are **positions on a 2D board** not elements from array
- Mark visited **in-place** on board (`'#'`) or with sets
- Restore after backtrack
- Base case = constraint fully satisfied

## Skeleton — Grid Search
```java
boolean backtrack(char[][] board, String word, int i, int j, int index) {
    if (index == word.length()) return true;
    if (i < 0 || i >= board.length || j < 0 || j >= board[0].length) return false;
    if (board[i][j] == '#' || board[i][j] != word.charAt(index)) return false;

    char original = board[i][j];
    board[i][j] = '#';                          // mark visited

    boolean found = backtrack(board, word, i+1, j, index+1)
                 || backtrack(board, word, i-1, j, index+1)
                 || backtrack(board, word, i, j+1, index+1)
                 || backtrack(board, word, i, j-1, index+1);

    board[i][j] = original;                     // restore
    return found;
}
```

## Skeleton — Row by Row (N-Queens style)
```java
void backtrack(int row, int n, Set<Integer> cols,
               Set<Integer> diag1, Set<Integer> diag2,
               char[][] board, List<List<String>> result) {
    if (row == n) { /* add board snapshot */ return; }
    for (int col = 0; col < n; col++) {
        if (cols.contains(col) || diag1.contains(row-col) || diag2.contains(row+col)) continue;
        board[row][col] = 'Q';
        cols.add(col); diag1.add(row-col); diag2.add(row+col);
        backtrack(row+1, n, cols, diag1, diag2, board, result);
        board[row][col] = '.';
        cols.remove(col); diag1.remove(row-col); diag2.remove(row+col);
    }
}
```

---

## Problems

### 1. N-Queens (LC 51)
**Description:** Place n queens on an n×n chessboard such that no two queens attack each other (no two in same row, column, or diagonal). Return all distinct solutions, each representing the board configuration.

**Example:**
- Input: n = 4
- Output: [[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]  (2 solutions for 4-queens)

```java
class Solution {
    Set<Integer> cols = new HashSet<>();
    Set<Integer> diag1 = new HashSet<>();
    Set<Integer> diag2 = new HashSet<>();
    List<List<String>> result = new ArrayList<>();

    public List<List<String>> solveNQueens(int n) {
        char[][] board = new char[n][n];
        for (char[] r : board) Arrays.fill(r, '.');
        backtrack(0, n, board);
        return result;
    }

    void backtrack(int row, int n, char[][] board) {
        if (cols.size() == n) {
            List<String> snapshot = new ArrayList<>();
            for (char[] r : board) snapshot.add(new String(r));
            result.add(snapshot);
            return;
        }
        for (int col = 0; col < n; col++) {
            if (cols.contains(col) || diag1.contains(row-col)
                    || diag2.contains(row+col)) continue;
            board[row][col] = 'Q';
            cols.add(col); diag1.add(row-col); diag2.add(row+col);
            backtrack(row+1, n, board);
            board[row][col] = '.';
            cols.remove(col); diag1.remove(row-col); diag2.remove(row+col);
        }
    }
}
```

**3 sets explanation**:
- `cols`: no two queens in same column
- `diag1` (row-col): no two queens on same ↗ diagonal
- `diag2` (row+col): no two queens on same ↘ diagonal

---

### 2. Word Search (LC 79)
**Description:** Given an m×n grid of characters and a word, return true if the word exists in the grid. The word must be constructed from adjacent (horizontally/vertically) letters, using each cell at most once.

**Example:**
- Input: board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
- Output: true  (path: A→B→C→C→E→D)

```java
class Solution {
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

    public boolean exist(char[][] board, String word) {
        int m = board.length, n = board[0].length;
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                if (backtrack(board, word, i, j, 0)) return true;
        return false;
    }

    boolean backtrack(char[][] board, String word, int i, int j, int index) {
        if (index == word.length()) return true;
        if (i < 0 || i >= board.length) return false;
        if (j < 0 || j >= board[0].length) return false;
        if (board[i][j] == '#' || board[i][j] != word.charAt(index)) return false;
        char original = board[i][j];
        board[i][j] = '#';
        for (int[] dir : dirs)
            if (backtrack(board, word, i+dir[0], j+dir[1], index+1)) return true;
        board[i][j] = original;
        return false;
    }
}
```

---

### 3. Sudoku Solver (LC 37)
**Description:** Write a program to solve a Sudoku puzzle by filling the empty cells ('.') such that each row, column, and 3×3 box contains digits 1-9 with no repetition.

**Example:**
- Input: board with some cells filled and '.' for empty
- Output: the same board filled in-place with the unique valid solution

```java
class Solution {
    public void solveSudoku(char[][] board) {
        backtrack(board);
    }

    boolean backtrack(char[][] board) {
        for (int i = 0; i < 9; i++)
            for (int j = 0; j < 9; j++) {
                if (board[i][j] != '.') continue;
                for (char c = '1'; c <= '9'; c++) {
                    if (!isValid(board, i, j, c)) continue;
                    board[i][j] = c;
                    if (backtrack(board)) return true;
                    board[i][j] = '.';
                }
                return false;  // no valid digit found → backtrack
            }
        return true;  // all cells filled
    }

    boolean isValid(char[][] board, int row, int col, char c) {
        for (int i = 0; i < 9; i++) {
            if (board[row][i] == c) return false;
            if (board[i][col] == c) return false;
            if (board[3*(row/3)+i/3][3*(col/3)+i%3] == c) return false;
        }
        return true;
    }
}
```

---

### 4. Word Search II (LC 212)
**Description:** Given an m×n board of characters and a list of strings words, return all words on the board. Each word must be constructed from adjacent letters (horizontally/vertically), using each cell at most once per word.

**Example:**
- Input: board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]], words = ["oath","pea","eat","rain"]
- Output: ["eat","oath"]

```java
class Solution {
    class TrieNode {
        TrieNode[] children = new TrieNode[26];
        String word = null;
    }

    public List<String> findWords(char[][] board, String[] words) {
        TrieNode root = new TrieNode();
        for (String w : words) {
            TrieNode node = root;
            for (char c : w.toCharArray()) {
                int idx = c - 'a';
                if (node.children[idx] == null) node.children[idx] = new TrieNode();
                node = node.children[idx];
            }
            node.word = w;
        }
        List<String> result = new ArrayList<>();
        int m = board.length, n = board[0].length;
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                backtrack(board, i, j, root, result);
        return result;
    }

    void backtrack(char[][] board, int i, int j, TrieNode node, List<String> result) {
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length) return;
        char c = board[i][j];
        if (c == '#' || node.children[c - 'a'] == null) return;
        node = node.children[c - 'a'];
        if (node.word != null) { result.add(node.word); node.word = null; }
        board[i][j] = '#';
        backtrack(board, i+1, j, node, result);
        backtrack(board, i-1, j, node, result);
        backtrack(board, i, j+1, node, result);
        backtrack(board, i, j-1, node, result);
        board[i][j] = c;
    }
}
```

---

### 5. Robot Room Cleaner (LC 489)
**Description:** Given a robot that can move forward, turn right, and clean its current cell (but cannot sense the grid), implement an algorithm to clean the entire reachable room. Obstacles block movement; the robot starts at an arbitrary position.

**Example:**
- Input: robot API (move(), turnRight(), clean()); room with obstacles
- Output: all reachable cells cleaned (in-place via robot commands)

```java
class Solution {
    int[][] dirs = {{-1,0},{0,1},{1,0},{0,-1}};
    Set<String> visited = new HashSet<>();
    Robot robot;

    public void cleanRoom(Robot robot) {
        this.robot = robot;
        backtrack(0, 0, 0);
    }

    void backtrack(int row, int col, int dir) {
        robot.clean();
        visited.add(row + "," + col);
        for (int i = 0; i < 4; i++) {
            int newDir = (dir + i) % 4;
            int newRow = row + dirs[newDir][0];
            int newCol = col + dirs[newDir][1];
            if (!visited.contains(newRow + "," + newCol) && robot.move()) {
                backtrack(newRow, newCol, newDir);
                // go back
                robot.turnRight(); robot.turnRight(); robot.move();
                robot.turnRight(); robot.turnRight();
            }
            robot.turnRight();
        }
    }
}
```

---

---

# 5. Common Pitfalls

## 1. Reference vs Snapshot

```java
// WRONG — adds reference, all entries end up as []
result.add(currPath);

// CORRECT — snapshot of current state
result.add(new ArrayList<>(currPath));
```

---

## 2. Wrong start index

```java
// WRONG — always increments from same start
backtrack(nums, start + 1, ...);

// CORRECT — increments from current i
backtrack(nums, i + 1, ...);
```

---

## 3. Duplicate skip condition

```java
// WRONG — skips duplicates across ALL levels
if (i > 0 && nums[i] == nums[i-1]) continue;

// CORRECT — skips duplicates at SAME level only
if (i > start && nums[i] == nums[i-1]) continue;  // subsets/combos
if (i > 0 && nums[i] == nums[i-1] && !visited[i-1]) continue;  // permutations
```

---

## 4. Forgetting to restore state

```java
// Board cell
char original = board[i][j];
board[i][j] = '#';
backtrack(...);
board[i][j] = original;  // ← must restore

// visited array
visited[i] = true;
backtrack(...);
visited[i] = false;  // ← must restore
```

---

## 5. Not sorting before duplicate removal

```java
// Duplicate skip only works on SORTED array
// [1,2,1] won't work — must sort to [1,1,2] first
Arrays.sort(nums);  // ← always sort first when input has duplicates
```

---

## 6. StringBuilder undo

```java
// List: remove last
curr.remove(curr.size() - 1);

// StringBuilder: delete last char
curr.deleteCharAt(curr.length() - 1);
```

---

---

# Quick Reference Card

## Family Decision Tree

```
Are you building all subsets (every intermediate state)?
  → Subsets family: add every node, i+1

Are you building all paths to a target sum?
  → Combinations family: add at leaf, i or i+1

Does order matter (1,2 ≠ 2,1)?
  → Permutations family: visited[], loop from 0

Are choices positions on a board/grid?
  → Board family: mark in-place, 4-dir or row-by-row
```

## The 4 Key Decisions

```
1. WHEN to add?
   Every node → subsets
   Leaf only  → combinations, permutations, board

2. REUSE element?
   Yes → pass i     (Combination Sum)
   No  → pass i+1   (all others)

3. DUPLICATES?
   Sort + if (i > start && nums[i] == nums[i-1]) continue   ← subsets/combos
   Sort + if (i > 0 && nums[i] == nums[i-1] && !visited[i-1]) continue  ← perms

4. PRUNE?
   sum > target → return
   candidates[i] > remaining → break (sorted)
   out of bounds → return
   cell visited → return
```

## Complexity

| Family | Time | Space |
|---|---|---|
| Subsets | O(n × 2ⁿ) | O(n) stack |
| Combinations | O(n × 2ⁿ) worst | O(n) stack |
| Permutations | O(n × n!) | O(n) stack |
| Board/Grid | O(4ᵏ) k=word length | O(k) stack |
| N-Queens | O(n!) | O(n) |
