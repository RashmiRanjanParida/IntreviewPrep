# Dynamic Programming — Complete Guide for Google L6

---

## How to Identify DP Problems

Ask yourself:
1. Does the problem ask to **optimize** (min/max) or **count** ways?
2. Can the problem be broken into **overlapping subproblems**?
3. Does the **optimal solution depend on optimal solutions of subproblems**?

If yes to all three → DP.

---

## The 5 DP Families

| Family | Signal | State | Direction |
|---|---|---|---|
| 1D Linear | single array, optimize over prefix | `dp[i]` | left → right |
| 2D Grid | grid movement, path problems | `dp[i][j]` | top-left → bottom-right (or reverse) |
| Knapsack | subset selection, "can you reach X" | `dp[j]` (capacity) | depends on reuse |
| Interval | merge/split/burst, "what to do last" | `dp[i][j]` | small → large intervals |
| Sequence | two strings, compare characters | `dp[i][j]` | left → right, top → bottom |

---

---

# 1. Linear (1D) DP

## How to Identify

- **Single array** as input
- Answer depends on **previous elements** in the array
- Keywords: "subarray", "subsequence", "at each house", "jump"
- State is just one index: `dp[i]` = answer for first `i` elements

## Signal Words
- "maximum subarray sum" → Kadane's
- "minimum cost" over array → linear DP
- "number of ways to reach" → counting DP
- "longest increasing subsequence" → LIS

## Skeleton

```java
int[] dp = new int[n];
dp[0] = base;

for (int i = 1; i < n; i++) {
    dp[i] = /* recurrence using dp[i-1], dp[i-2], etc. */;
}

return dp[n - 1];  // or scan dp[] for answer
```

## Key Patterns

```
dp[i] = dp[i-1] + nums[i]           // running sum
dp[i] = max(dp[i-1] + nums[i], nums[i])  // Kadane's
dp[i] = max(dp[i-1], dp[i-2] + nums[i]) // House Robber
dp[i] = min over j<i of dp[j] + cost(j,i) // jump/split
```

---

## Problems

### 1. Maximum Subarray (LC 53)
**Description:** Find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

**Example:**
- Input: nums = [-2,1,-3,4,-1,2,1,-5,4]
- Output: 6  (subarray [4,-1,2,1] has the largest sum = 6)

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int prev = nums[0], max = nums[0];
        for (int i = 1; i < nums.length; i++) {
            prev = Math.max(prev + nums[i], nums[i]);
            max = Math.max(max, prev);
        }
        return max;
    }
}
```
**Recurrence**: `dp[i] = max(dp[i-1] + nums[i], nums[i])` — extend or restart.

---

### 2. Climbing Stairs (LC 70)
**Description:** You are climbing a staircase with n steps. Each time you can climb 1 or 2 steps. Count the number of distinct ways to reach the top.

**Example:**
- Input: n = 5
- Output: 8  (1+1+1+1+1, 1+1+1+2, 1+1+2+1, 1+2+1+1, 2+1+1+1, 1+2+2, 2+1+2, 2+2+1)

```java
class Solution {
    public int climbStairs(int n) {
        if (n <= 2) return n;
        int a = 1, b = 2;
        for (int i = 3; i <= n; i++) {
            int c = a + b;
            a = b; b = c;
        }
        return b;
    }
}
```
**Recurrence**: `dp[i] = dp[i-1] + dp[i-2]` — Fibonacci.

---

### 3. House Robber (LC 198)
**Description:** You are a robber planning to rob houses along a street. Adjacent houses have security systems — you cannot rob two adjacent houses. Given an array of money in each house, find the maximum amount you can rob.

**Example:**
- Input: nums = [2,7,9,3,1]
- Output: 12  (rob houses 0,2,4: 2+9+1=12)

```java
class Solution {
    public int rob(int[] nums) {
        if (nums.length == 1) return nums[0];
        int prev2 = nums[0], prev1 = Math.max(nums[0], nums[1]);
        for (int i = 2; i < nums.length; i++) {
            int curr = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1; prev1 = curr;
        }
        return prev1;
    }
}
```
**Recurrence**: `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`.

---

### 4. House Robber II (LC 213)
**Description:** Same as House Robber but houses are arranged in a circle — the first and last houses are adjacent and cannot both be robbed.

**Example:**
- Input: nums = [2,3,2]
- Output: 3  (rob house 1 only; can't rob 0 and 2 together since they're adjacent)

```java
class Solution {
    public int rob(int[] nums) {
        if (nums.length == 1) return nums[0];
        return Math.max(robRange(nums, 0, nums.length - 2),
                        robRange(nums, 1, nums.length - 1));
    }

    private int robRange(int[] nums, int lo, int hi) {
        int prev2 = 0, prev1 = 0;
        for (int i = lo; i <= hi; i++) {
            int curr = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1; prev1 = curr;
        }
        return prev1;
    }
}
```
**Trick**: split into two linear problems — exclude first or exclude last house.

---

### 5. Coin Change (LC 322)
**Description:** Given an array of coin denominations and an amount, return the minimum number of coins needed to make up that amount. Return -1 if the amount cannot be made.

**Example:**
- Input: coins = [1,5,6,9], amount = 11
- Output: 2  (5+6=11, only 2 coins)

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, Integer.MAX_VALUE);
        dp[0] = 0;
        for (int i = 1; i <= amount; i++)
            for (int c : coins)
                if (c <= i && dp[i - c] != Integer.MAX_VALUE)
                    dp[i] = Math.min(dp[i], dp[i - c] + 1);
        return dp[amount] == Integer.MAX_VALUE ? -1 : dp[amount];
    }
}
```
**Recurrence**: `dp[i] = min over coins of dp[i - coin] + 1`.

---

### 6. Longest Increasing Subsequence (LC 300)
**Description:** Given an integer array, return the length of the longest strictly increasing subsequence (elements need not be contiguous).

**Example:**
- Input: nums = [10,9,2,5,3,7,101,18]
- Output: 4  (subsequence [2,3,7,101] or [2,5,7,101])

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int[] dp = new int[nums.length];
        Arrays.fill(dp, 1);
        int max = 1;
        for (int i = 1; i < nums.length; i++) {
            for (int j = 0; j < i; j++)
                if (nums[j] < nums[i])
                    dp[i] = Math.max(dp[i], dp[j] + 1);
            max = Math.max(max, dp[i]);
        }
        return max;
    }
}
```
**Recurrence**: `dp[i] = max over j<i where nums[j]<nums[i] of dp[j]+1`.

---

### 7. Jump Game II (LC 45)
**Description:** Given an array where nums[i] is the maximum jump length from index i, return the minimum number of jumps to reach the last index. It is guaranteed you can always reach the last index.

**Example:**
- Input: nums = [2,3,1,1,4]
- Output: 2  (jump 1 step from index 0 to 1, then 3 steps to last index)

```java
class Solution {
    public int jump(int[] nums) {
        int jumps = 0, currEnd = 0, farthest = 0;
        for (int i = 0; i < nums.length - 1; i++) {
            farthest = Math.max(farthest, i + nums[i]);
            if (i == currEnd) {
                jumps++;
                currEnd = farthest;
            }
        }
        return jumps;
    }
}
```
**Pattern**: greedy DP — track farthest reachable at each step.

---

### 8. Word Break (LC 139)
**Description:** Given a string s and a dictionary of strings wordDict, return true if s can be segmented into a space-separated sequence of one or more dictionary words.

**Example:**
- Input: s = "leetcode", wordDict = ["leet","code"]
- Output: true  ("leetcode" = "leet" + "code")

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        Set<String> dict = new HashSet<>(wordDict);
        int n = s.length();
        boolean[] dp = new boolean[n + 1];
        dp[0] = true;
        for (int i = 1; i <= n; i++)
            for (int j = 0; j < i; j++)
                if (dp[j] && dict.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }
        return dp[n];
    }
}
```
**Recurrence**: `dp[i] = true if dp[j] && s[j..i] in dict for some j<i`.

---

### 9. Decode Ways (LC 91)
**Description:** A message encoded as digits ('A'=1, 'B'=2, ..., 'Z'=26). Count the number of ways to decode the digit string.

**Example:**
- Input: s = "226"
- Output: 3  ("BZ"=2,26; "VF"=22,6; "BBF"=2,2,6 → 3 ways)

```java
class Solution {
    public int numDecodings(String s) {
        int n = s.length();
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = s.charAt(0) == '0' ? 0 : 1;
        for (int i = 2; i <= n; i++) {
            int one = Integer.parseInt(s.substring(i - 1, i));
            int two = Integer.parseInt(s.substring(i - 2, i));
            if (one >= 1) dp[i] += dp[i - 1];
            if (two >= 10 && two <= 26) dp[i] += dp[i - 2];
        }
        return dp[n];
    }
}
```
**Recurrence**: `dp[i] = dp[i-1] (1-digit) + dp[i-2] (2-digit if valid)`.

---

### 10. Perfect Squares (LC 279)
**Description:** Given an integer n, return the minimum number of perfect square numbers (1, 4, 9, 16, ...) that sum to n.

**Example:**
- Input: n = 12
- Output: 3  (12 = 4 + 4 + 4)

```java
class Solution {
    public int numSquares(int n) {
        int[] dp = new int[n + 1];
        Arrays.fill(dp, Integer.MAX_VALUE);
        dp[0] = 0;
        for (int i = 1; i <= n; i++)
            for (int j = 1; j * j <= i; j++)
                dp[i] = Math.min(dp[i], dp[i - j * j] + 1);
        return dp[n];
    }
}
```
**Recurrence**: `dp[i] = min over all squares j² ≤ i of dp[i - j²] + 1`.

---

---

# 2. 2D Grid DP

## How to Identify

- **Grid/matrix** as input
- Movement is **restricted** (usually right/down or reverse)
- Keywords: "path", "minimum cost path", "number of paths", "square"
- If movement is free (any direction) → use Dijkstra instead

## DP vs Dijkstra Decision

| Condition | Use |
|---|---|
| Can only go right/down (restricted) | DP |
| Can go any direction | Dijkstra |
| Negative weights, restricted movement | DP |

## Skeleton (Forward Fill)

```java
int[][] dp = new int[m][n];
dp[0][0] = grid[0][0];

// base cases
for (int i = 1; i < m; i++) dp[i][0] = dp[i-1][0] + grid[i][0];
for (int j = 1; j < n; j++) dp[0][j] = dp[0][j-1] + grid[0][j];

// fill
for (int i = 1; i < m; i++)
    for (int j = 1; j < n; j++)
        dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];

return dp[m-1][n-1];
```

## Skeleton (Reverse Fill — when future affects present)

```java
// fill bottom-right to top-left
for (int i = m - 1; i >= 0; i--)
    for (int j = n - 1; j >= 0; j--)
        dp[i][j] = /* depends on dp[i+1][j], dp[i][j+1] */;

return dp[0][0];
```

---

## Problems

### 1. Unique Paths (LC 62)
**Description:** A robot is on an m×n grid at the top-left corner and wants to reach the bottom-right corner. It can only move right or down. Count the number of distinct paths.

**Example:**
- Input: m = 3, n = 7
- Output: 28

```java
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];
        for (int i = 0; i < m; i++) dp[i][0] = 1;
        for (int j = 0; j < n; j++) dp[0][j] = 1;
        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++)
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
        return dp[m-1][n-1];
    }
}
```

---

### 2. Unique Paths II (LC 63)
**Description:** Same as Unique Paths but the grid has obstacles (marked 1). Count the number of distinct paths avoiding obstacles.

**Example:**
- Input: obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]
- Output: 2  (obstacle in middle blocks one path)

```java
class Solution {
    public int uniquePathsWithObstacles(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        int[][] dp = new int[m][n];
        for (int i = 0; i < m && grid[i][0] == 0; i++) dp[i][0] = 1;
        for (int j = 0; j < n && grid[0][j] == 0; j++) dp[0][j] = 1;
        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++)
                if (grid[i][j] == 0)
                    dp[i][j] = dp[i-1][j] + dp[i][j-1];
        return dp[m-1][n-1];
    }
}
```

---

### 3. Minimum Path Sum (LC 64)
**Description:** Given an m×n grid of non-negative integers, find a path from top-left to bottom-right (only right or down moves) that minimizes the sum of all numbers along the path.

**Example:**
- Input: grid = [[1,3,1],[1,5,1],[4,2,1]]
- Output: 7  (path 1→3→1→1→1 = 7)

```java
class Solution {
    public int minPathSum(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        int[][] dp = new int[m][n];
        dp[0][0] = grid[0][0];
        for (int i = 1; i < m; i++) dp[i][0] = dp[i-1][0] + grid[i][0];
        for (int j = 1; j < n; j++) dp[0][j] = dp[0][j-1] + grid[0][j];
        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++)
                dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
        return dp[m-1][n-1];
    }
}
```

---

### 4. Maximal Square (LC 221)
**Description:** Given an m×n binary matrix filled with '0's and '1's, find the largest square containing only '1's and return its area.

**Example:**
- Input: matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
- Output: 4  (2×2 square of 1s exists → area = 4)

```java
class Solution {
    public int maximalSquare(char[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        int[][] dp = new int[m][n];
        int max = 0;
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++) {
                dp[i][j] = matrix[i][j] == '1' ? 1 : 0;
                max = Math.max(max, dp[i][j]);
            }
        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++) {
                if (dp[i][j] == 0) continue;
                dp[i][j] = Math.min(Math.min(dp[i-1][j], dp[i][j-1]), dp[i-1][j-1]) + 1;
                max = Math.max(max, dp[i][j]);
            }
        return max * max;
    }
}
```
**Recurrence**: `dp[i][j] = min(left, above, diagonal) + 1` when cell is '1'.

---

### 5. Dungeon Game (LC 174)
**Description:** A knight must rescue a princess in a dungeon grid. Rooms have demons (negative values) or magic orbs (positive values). The knight starts top-left, must reach bottom-right, and must maintain HP > 0 at all times. Return the minimum initial HP needed.

**Example:**
- Input: dungeon = [[-2,-3,3],[-5,-10,1],[10,30,-5]]
- Output: 7  (take path right,right,down,down with minimum HP 7)

```java
class Solution {
    public int calculateMinimumHP(int[][] dungeon) {
        int m = dungeon.length, n = dungeon[0].length;
        int[][] dp = new int[m][n];
        dp[m-1][n-1] = Math.max(1, 1 - dungeon[m-1][n-1]);
        for (int i = m-2; i >= 0; i--)
            dp[i][n-1] = Math.max(1, dp[i+1][n-1] - dungeon[i][n-1]);
        for (int j = n-2; j >= 0; j--)
            dp[m-1][j] = Math.max(1, dp[m-1][j+1] - dungeon[m-1][j]);
        for (int i = m-2; i >= 0; i--)
            for (int j = n-2; j >= 0; j--) {
                int need = Math.min(dp[i+1][j], dp[i][j+1]);
                dp[i][j] = Math.max(1, need - dungeon[i][j]);
            }
        return dp[0][0];
    }
}
```
**Key insight**: fill reverse when future determines present. Min value = 1 (can't enter dead).

---

### 6. Triangle (LC 120)
**Description:** Given a triangle array, return the minimum path sum from top to bottom. At each step you may move to an adjacent number in the row below (index j or j+1).

**Example:**
- Input: triangle = [[2],[3,4],[6,5,7],[4,1,8,3]]
- Output: 11  (path 2→3→5→1 = 11)

```java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        int n = triangle.size();
        int[] dp = new int[n];
        for (int i = 0; i < n; i++) dp[i] = triangle.get(n-1).get(i);
        for (int i = n-2; i >= 0; i--)
            for (int j = 0; j <= i; j++)
                dp[j] = Math.min(dp[j], dp[j+1]) + triangle.get(i).get(j);
        return dp[0];
    }
}
```
**Fill bottom → top**: `dp[j] = min(dp[j], dp[j+1]) + triangle[i][j]`.

---

### 7. Minimum Falling Path Sum (LC 931)
**Description:** Given an n×n matrix, choose one element from each row such that no two chosen elements are in the same or adjacent columns. Return the minimum sum of chosen elements (a "falling path").

**Example:**
- Input: matrix = [[2,1,3],[6,5,4],[7,8,9]]
- Output: 13  (1+4+8=13, using columns 1,2,1)

```java
class Solution {
    public int minFallingPathSum(int[][] matrix) {
        int n = matrix.length;
        int[][] dp = new int[n][n];
        for (int j = 0; j < n; j++) dp[n-1][j] = matrix[n-1][j];
        for (int i = n-2; i >= 0; i--)
            for (int j = 0; j < n; j++) {
                int best = dp[i+1][j];
                if (j > 0) best = Math.min(best, dp[i+1][j-1]);
                if (j < n-1) best = Math.min(best, dp[i+1][j+1]);
                dp[i][j] = matrix[i][j] + best;
            }
        int ans = Integer.MAX_VALUE;
        for (int j = 0; j < n; j++) ans = Math.min(ans, dp[0][j]);
        return ans;
    }
}
```

---

### 8. Count Square Submatrices with All Ones (LC 1277)
**Description:** Given an m×n binary matrix, return the total number of square submatrices that have all ones (count all sizes, not just the maximum).

**Example:**
- Input: matrix = [[0,1,1,1],[1,1,1,1],[0,1,1,1]]
- Output: 15  (ten 1×1 + four 2×2 + one 3×3 squares = 15)

```java
class Solution {
    public int countSquares(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length, ans = 0;
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++) {
                if (matrix[i][j] == 1 && i > 0 && j > 0)
                    matrix[i][j] = Math.min(Math.min(matrix[i-1][j],
                        matrix[i][j-1]), matrix[i-1][j-1]) + 1;
                ans += matrix[i][j];
            }
        return ans;
    }
}
```
**Key insight**: `dp[i][j]` = side of largest square ending at (i,j) = count of squares ending here.

---

### 9. Cherry Pickup II (LC 1463)
**Description:** Two robots start at the top row (columns 0 and n-1) of a grid and move down simultaneously. Each step they can move to the cell directly below or diagonally below. Maximize the total cherries collected (if both visit same cell, count once).

**Example:**
- Input: grid = [[3,1,1],[2,5,1],[1,5,5],[2,1,1]]
- Output: 24  (robot 1 picks 3+2+5+2=12, robot 2 picks 1+1+5+5=12... optimal is 24)

```java
class Solution {
    int[][][] memo;
    int[][] grid;
    int m, n;

    public int cherryPickup(int[][] grid) {
        this.grid = grid;
        m = grid.length; n = grid[0].length;
        memo = new int[m][n][n];
        for (int[][] a : memo) for (int[] b : a) Arrays.fill(b, -1);
        return dp(0, 0, n - 1);
    }

    private int dp(int row, int j1, int j2) {
        if (row == m) return 0;
        if (memo[row][j1][j2] != -1) return memo[row][j1][j2];
        int cherries = grid[row][j1] + (j1 == j2 ? 0 : grid[row][j2]);
        int best = 0;
        for (int d1 = -1; d1 <= 1; d1++)
            for (int d2 = -1; d2 <= 1; d2++) {
                int nj1 = j1 + d1, nj2 = j2 + d2;
                if (nj1 >= 0 && nj1 < n && nj2 >= 0 && nj2 < n)
                    best = Math.max(best, dp(row + 1, nj1, nj2));
            }
        return memo[row][j1][j2] = cherries + best;
    }
}
```
**Pattern**: two entities moving simultaneously → combine into one state.

---

### 10. Maximal Rectangle (LC 85)
**Description:** Given a binary matrix of '0's and '1's, find the largest rectangle containing only '1's and return its area. (Builds a histogram row by row and applies largest rectangle in histogram.)

**Example:**
- Input: matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
- Output: 6  (3-wide, 2-tall rectangle of 1s in rows 1-2, columns 2-4)

```java
class Solution {
    public int maximalRectangle(char[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        int[] heights = new int[n];
        int max = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++)
                heights[j] = matrix[i][j] == '1' ? heights[j] + 1 : 0;
            max = Math.max(max, largestRectangleInHistogram(heights));
        }
        return max;
    }

    private int largestRectangleInHistogram(int[] heights) {
        Deque<Integer> stack = new ArrayDeque<>();
        int max = 0;
        for (int i = 0; i <= heights.length; i++) {
            int h = i == heights.length ? 0 : heights[i];
            while (!stack.isEmpty() && heights[stack.peek()] > h) {
                int height = heights[stack.pop()];
                int width = stack.isEmpty() ? i : i - stack.peek() - 1;
                max = Math.max(max, height * width);
            }
            stack.push(i);
        }
        return max;
    }
}
```

---

---

# 3. Knapsack DP

## How to Identify

- **Subset selection** from an array
- "Can you reach exactly X?", "count ways to sum to X"
- "partition", "target sum", "select items within capacity"
- Each item either taken or not (0/1) OR taken unlimited times (unbounded)

## 0/1 vs Unbounded

| | 0/1 Knapsack | Unbounded Knapsack |
|---|---|---|
| Item reuse | at most once | unlimited |
| Inner loop | **decreasing** (target → 0) | **increasing** (0 → target) |
| Why | reads previous-row values | reads current-row values (reuse ok) |

## Skeleton — 0/1 Knapsack

```java
boolean[] dp = new boolean[target + 1];
dp[0] = true;

for (int num : nums) {
    for (int j = target; j >= num; j--) {  // decreasing!
        dp[j] = dp[j] || dp[j - num];
    }
}
return dp[target];
```

## Skeleton — Unbounded Knapsack

```java
int[] dp = new int[amount + 1];
dp[0] = 1;  // or 0 depending on problem

for (int coin : coins) {
    for (int j = coin; j <= amount; j++) {  // increasing!
        dp[j] += dp[j - coin];
    }
}
return dp[amount];
```

## Loop Order Summary

| Problem | Outer | Inner | Order |
|---|---|---|---|
| Partition Subset | items | capacity | decreasing |
| Target Sum | items | capacity | temp array |
| Coin Change (min) | capacity | coins | either |
| Coin Change 2 (count) | coins | capacity | increasing |
| Combinations | items | capacity | decreasing |
| Permutations | capacity | items | increasing |

---

## Problems

### 1. Partition Equal Subset Sum (LC 416)
**Description:** Given a non-empty array of positive integers, determine if the array can be partitioned into two subsets such that the sum of elements in both subsets is equal.

**Example:**
- Input: nums = [1,5,11,5]
- Output: true  ([1,5,5] and [11] both sum to 11)

```java
class Solution {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for (int n : nums) sum += n;
        if (sum % 2 != 0) return false;
        int target = sum / 2;
        boolean[] dp = new boolean[target + 1];
        dp[0] = true;
        for (int num : nums)
            for (int j = target; j >= num; j--)
                dp[j] = dp[j] || dp[j - num];
        return dp[target];
    }
}
```

---

### 2. Target Sum (LC 494)
**Description:** Given an integer array and a target, assign '+' or '-' to each element, then evaluate the sum. Return the number of different expressions that evaluate to target.

**Example:**
- Input: nums = [1,1,1,1,1], target = 3
- Output: 5  (five ways to choose signs giving sum = 3)

```java
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        int totalSum = 0;
        for (int num : nums) totalSum += num;
        if (Math.abs(target) > totalSum) return 0;
        int size = 2 * totalSum + 1, offset = totalSum;
        int[] dp = new int[size];
        dp[offset] = 1;
        for (int num : nums) {
            int[] next = new int[size];
            for (int j = 0; j < size; j++) {
                if (dp[j] == 0) continue;
                next[j + num] += dp[j];
                next[j - num] += dp[j];
            }
            dp = next;
        }
        return dp[target + offset];
    }
}
```

---

### 3. Coin Change 2 (LC 518)
**Description:** Given an amount and a list of coin denominations (unlimited supply of each), return the number of combinations that make up the amount.

**Example:**
- Input: amount = 5, coins = [1,2,5]
- Output: 4  (5=5; 5=2+2+1; 5=2+1+1+1; 5=1+1+1+1+1)

```java
class Solution {
    public int change(int amount, int[] coins) {
        int[] dp = new int[amount + 1];
        dp[0] = 1;
        for (int coin : coins)
            for (int j = coin; j <= amount; j++)
                dp[j] += dp[j - coin];
        return dp[amount];
    }
}
```
**Key**: outer=coins, inner=amount (increasing) → combinations, not permutations.

---

### 4. Subset Sum (classic)
**Description:** Given an array of non-negative integers and a target sum, determine whether any subset of the array sums to exactly that target.

**Example:**
- Input: nums = [3,34,4,12,5,2], target = 9
- Output: true  ([4,5] or [3,4,2] sum to 9)

```java
public boolean subsetSum(int[] nums, int target) {
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;
    for (int num : nums)
        for (int j = target; j >= num; j--)
            dp[j] = dp[j] || dp[j - num];
    return dp[target];
}
```

---

### 5. Last Stone Weight II (LC 1049)
**Description:** You have a collection of stones. Each time, smash two stones together — the larger loses the difference in weight, and the smaller is destroyed. Return the smallest possible weight of the last remaining stone (or 0).

**Example:**
- Input: stones = [2,7,4,1,8,1]
- Output: 1  (7 vs 8 → 1; 2 vs 4 → 2; 2 vs 1 → 1; 1 vs 1 → 0+1=1)

```java
class Solution {
    public int lastStoneWeightII(int[] stones) {
        int sum = 0;
        for (int s : stones) sum += s;
        int target = sum / 2;
        boolean[] dp = new boolean[target + 1];
        dp[0] = true;
        for (int s : stones)
            for (int j = target; j >= s; j--)
                dp[j] = dp[j] || dp[j - s];
        for (int j = target; j >= 0; j--)
            if (dp[j]) return sum - 2 * j;
        return sum;
    }
}
```
**Reduction**: minimize |S1 - S2| = minimize |total - 2*S1| → knapsack to maximize S1 ≤ total/2.

---

### 6. Ones and Zeroes (LC 474)
**Description:** Given an array of binary strings and integers m (max zeros) and n (max ones), find the maximum size of a subset such that there are at most m zeros and n ones.

**Example:**
- Input: strs = ["10","0001","111001","1","0"], m = 5, n = 3
- Output: 4  (["10","0001","1","0"] uses 4 zeros and 3 ones ≤ m=5, n=3)

```java
class Solution {
    public int findMaxForm(String[] strs, int m, int n) {
        int[][] dp = new int[m + 1][n + 1];
        for (String s : strs) {
            int zeros = 0, ones = 0;
            for (char c : s.toCharArray()) if (c == '0') zeros++; else ones++;
            for (int i = m; i >= zeros; i--)
                for (int j = n; j >= ones; j--)
                    dp[i][j] = Math.max(dp[i][j], dp[i - zeros][j - ones] + 1);
        }
        return dp[m][n];
    }
}
```
**2D knapsack**: two capacities (zeros budget m, ones budget n).

---

### 7. Combination Sum IV (LC 377)
**Description:** Given an array of distinct integers and a target, return the number of possible ordered combinations (permutations) that add up to the target. Order matters: [1,1,2] and [1,2,1] are different.

**Example:**
- Input: nums = [1,2,3], target = 4
- Output: 7  ([1,1,1,1],[1,1,2],[1,2,1],[2,1,1],[1,3],[3,1],[2,2])

```java
class Solution {
    public int combinationSum4(int[] nums, int target) {
        int[] dp = new int[target + 1];
        dp[0] = 1;
        for (int j = 1; j <= target; j++)       // outer = capacity
            for (int num : nums)                 // inner = items
                if (j >= num) dp[j] += dp[j - num];
        return dp[target];
    }
}
```
**Key**: swap loop order (outer=capacity, inner=items) → permutations instead of combinations.

---

### 8. Knapsack 0/1 (Classic)
**Description:** Given n items each with a weight and value, and a knapsack of capacity W, find the maximum value you can carry. Each item can be taken at most once.

**Example:**
- Input: weights = [1,3,4,5], values = [1,4,5,7], W = 7
- Output: 9  (take items with weight 3 and 4: value = 4+5=9)

```java
public int knapsack(int[] weights, int[] values, int W) {
    int n = weights.length;
    int[] dp = new int[W + 1];
    for (int i = 0; i < n; i++)
        for (int j = W; j >= weights[i]; j--)
            dp[j] = Math.max(dp[j], dp[j - weights[i]] + values[i]);
    return dp[W];
}
```

---

### 9. Minimum Subset Sum Difference (classic)
**Description:** Given an array, partition it into two subsets and minimize the absolute difference of their sums.

**Example:**
- Input: nums = [1,6,11,5]
- Output: 1  ([1,5,6]=12 and [11]=11, diff=1)

```java
public int minimumDifference(int[] nums) {
    int sum = 0;
    for (int n : nums) sum += n;
    int target = sum / 2;
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;
    for (int num : nums)
        for (int j = target; j >= num; j--)
            dp[j] = dp[j] || dp[j - num];
    for (int j = target; j >= 0; j--)
        if (dp[j]) return sum - 2 * j;
    return sum;
}
```

---

### 10. Count of Subsets with Given Sum (classic)
**Description:** Given an array of non-negative integers and a target sum, count the number of subsets whose elements sum to exactly target.

**Example:**
- Input: nums = [2,3,5,6,8,10], target = 10
- Output: 3  ([2,8],[2,3,5],[10])

```java
public int countSubsets(int[] nums, int target) {
    int[] dp = new int[target + 1];
    dp[0] = 1;
    for (int num : nums)
        for (int j = target; j >= num; j--)
            dp[j] += dp[j - num];
    return dp[target];
}
```

---

---

# 4. Interval DP

## How to Identify

- Problem on **contiguous subarray or substring**
- "merge", "burst", "remove", "split", "parenthesize"
- Answer depends on **what you do last** (not first)
- Thinking "what to do first" leads to corrupted subproblems

## Why "Last Operation"?

If you think "what to do first" — after the operation, the structure changes, making previously computed subproblems invalid. Thinking "what to do last" keeps boundaries fixed → clean subproblems.

## Skeleton

```java
int[][] dp = new int[n][n];
// base case: single elements

for (int len = 2; len <= n; len++) {          // interval length
    for (int i = 0; i <= n - len; i++) {      // start index
        int j = i + len - 1;                  // end index
        dp[i][j] = Integer.MAX_VALUE;         // or MIN if maximizing
        for (int k = i; k < j; k++) {         // split point
            dp[i][j] = Math.min(dp[i][j],
                dp[i][k] + dp[k+1][j] + cost(i, k, j));
        }
    }
}
return dp[0][n-1];
```

---

## Problems

### 1. Burst Balloons (LC 312)
**Description:** Given n balloons, each with a number. When you burst balloon i, you gain nums[left] * nums[i] * nums[right] coins. Burst all balloons to maximize coins. Padding with 1s at both ends.

**Example:**
- Input: nums = [3,1,5,8]
- Output: 167  (burst order: 1,5,3,8 → 3+30+15+40+72... = 167)

```java
class Solution {
    public int maxCoins(int[] nums) {
        int n = nums.length;
        int[] arr = new int[n + 2];
        arr[0] = arr[n + 1] = 1;
        for (int i = 0; i < n; i++) arr[i + 1] = nums[i];
        int size = n + 2;
        int[][] dp = new int[size][size];
        for (int len = 2; len < size; len++)
            for (int i = 0; i <= size - len - 1; i++) {
                int j = i + len;
                for (int k = i + 1; k < j; k++)
                    dp[i][j] = Math.max(dp[i][j],
                        dp[i][k] + arr[i] * arr[k] * arr[j] + dp[k][j]);
            }
        return dp[0][n + 1];
    }
}
```
**Key**: `dp[i][j]` = open interval — boundaries i,j not burst. k = last balloon burst.

---

### 2. Matrix Chain Multiplication (Classic)
**Description:** Given dimensions of n matrices to multiply in sequence, find the minimum number of scalar multiplications needed. The order of multiplication can be changed but not the sequence.

**Example:**
- Input: dims = [10,30,5,60]  (three matrices: 10×30, 30×5, 5×60)
- Output: 4500  ((A×B)×C costs 10×30×5 + 10×5×60 = 1500+3000=4500; A×(B×C) costs 27000)

```java
public int minMultiplications(int[] dims) {
    int n = dims.length - 1;
    int[][] dp = new int[n + 1][n + 1];
    for (int i = 1; i < n; i++)
        dp[i][i + 1] = dims[i - 1] * dims[i] * dims[i + 1];
    for (int len = 2; len < n; len++)
        for (int i = 1; i <= n - len; i++) {
            int j = i + len;
            dp[i][j] = Integer.MAX_VALUE;
            for (int k = i + 1; k < j; k++)
                dp[i][j] = Math.min(dp[i][j],
                    dp[i][k] + dp[k + 1][j] + dims[i - 1] * dims[k] * dims[j]);
        }
    return dp[1][n];
}
```

---

### 3. Minimum Cost to Merge Stones (LC 1000)
**Description:** There are n piles of stones. Merge exactly k consecutive piles in one step, costing the sum of those piles. Find the minimum cost to reduce to one pile (return -1 if impossible).

**Example:**
- Input: stones = [3,2,4,1], k = 2
- Output: 20  (merge 2+4=6: cost 6; merge 3+6=9: cost 9; merge 9+1=10: cost... total 20)

```java
class Solution {
    public int mergeStones(int[] stones, int k) {
        int n = stones.length;
        if ((n - 1) % (k - 1) != 0) return -1;
        int[] prefix = new int[n + 1];
        for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + stones[i];
        int[][] dp = new int[n][n];
        for (int len = k; len <= n; len++)
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                dp[i][j] = Integer.MAX_VALUE;
                for (int mid = i; mid < j; mid += k - 1)
                    dp[i][j] = Math.min(dp[i][j], dp[i][mid] + dp[mid + 1][j]);
                if ((j - i) % (k - 1) == 0)
                    dp[i][j] += prefix[j + 1] - prefix[i];
            }
        return dp[0][n - 1];
    }
}
```

---

### 4. Strange Printer (LC 664)
**Description:** A printer can only print a sequence of the same character in one turn, and can overwrite previously printed characters. Given a string, return the minimum number of turns to print it.

**Example:**
- Input: s = "aaabbb"
- Output: 2  (print "aaa", then "bbb" → 2 turns)

```java
class Solution {
    public int strangePrinter(String s) {
        int n = s.length();
        int[][] dp = new int[n][n];
        for (int len = 1; len <= n; len++)
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                dp[i][j] = len;
                for (int k = i; k < j; k++) {
                    int val = dp[i][k] + dp[k + 1][j];
                    if (s.charAt(k) == s.charAt(j)) val--;
                    dp[i][j] = Math.min(dp[i][j], val);
                }
            }
        return dp[0][n - 1];
    }
}
```

---

### 5. Palindrome Partitioning II (LC 132)
**Description:** Given a string s, partition it such that every substring of the partition is a palindrome. Return the minimum number of cuts needed.

**Example:**
- Input: s = "aab"
- Output: 1  (["aa","b"] requires 1 cut; "aa" and "b" are both palindromes)

```java
class Solution {
    public int minCut(String s) {
        int n = s.length();
        boolean[][] isPalin = new boolean[n][n];
        for (int len = 1; len <= n; len++)
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                isPalin[i][j] = s.charAt(i) == s.charAt(j)
                    && (len <= 2 || isPalin[i + 1][j - 1]);
            }
        int[] dp = new int[n];
        Arrays.fill(dp, Integer.MAX_VALUE);
        for (int i = 0; i < n; i++) {
            if (isPalin[0][i]) { dp[i] = 0; continue; }
            for (int j = 1; j <= i; j++)
                if (isPalin[j][i] && dp[j - 1] != Integer.MAX_VALUE)
                    dp[i] = Math.min(dp[i], dp[j - 1] + 1);
        }
        return dp[n - 1];
    }
}
```

---

### 6. Longest Palindromic Subsequence (LC 516)
**Description:** Given a string s, return the length of the longest palindromic subsequence in s (elements need not be contiguous).

**Example:**
- Input: s = "bbbab"
- Output: 4  (longest palindromic subsequence is "bbbb")

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        int n = s.length();
        int[][] dp = new int[n][n];
        for (int i = 0; i < n; i++) dp[i][i] = 1;
        for (int len = 2; len <= n; len++)
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                if (s.charAt(i) == s.charAt(j))
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                else
                    dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
            }
        return dp[0][n - 1];
    }
}
```

---

### 7. Remove Boxes (LC 546)
**Description:** Given several boxes with different colors (represented by integers). Remove boxes in groups of the same color to earn points: removing k boxes of the same color earns k² points. Return the maximum points you can get.

**Example:**
- Input: boxes = [1,3,2,2,2,3,4,3,1]
- Output: 23

```java
class Solution {
    int[][][] memo;

    public int removeBoxes(int[] boxes) {
        int n = boxes.length;
        memo = new int[n][n][n];
        return dp(boxes, 0, n - 1, 0);
    }

    private int dp(int[] boxes, int l, int r, int k) {
        if (l > r) return 0;
        if (memo[l][r][k] != 0) return memo[l][r][k];
        int res = (k + 1) * (k + 1) + dp(boxes, l + 1, r, 0);
        for (int m = l + 1; m <= r; m++)
            if (boxes[m] == boxes[l])
                res = Math.max(res, dp(boxes, l + 1, m - 1, 0) + dp(boxes, m, r, k + 1));
        return memo[l][r][k] = res;
    }
}
```

---

### 8. Scramble String (LC 87)
**Description:** We can scramble a string by choosing a non-leaf node and swapping its two subtrees. Given two strings s1 and s2 of the same length, return true if s2 is a scrambled string of s1.

**Example:**
- Input: s1 = "great", s2 = "rgeat"
- Output: true  ("great" → split "gr"|"eat" → swap → "rg"|"eat" → "rgeat")

```java
class Solution {
    Map<String, Boolean> memo = new HashMap<>();

    public boolean isScramble(String s1, String s2) {
        if (s1.equals(s2)) return true;
        String key = s1 + "#" + s2;
        if (memo.containsKey(key)) return memo.get(key);
        int n = s1.length();
        int[] count = new int[26];
        for (int i = 0; i < n; i++) {
            count[s1.charAt(i) - 'a']++;
            count[s2.charAt(i) - 'a']--;
        }
        for (int c : count) if (c != 0) { memo.put(key, false); return false; }
        for (int i = 1; i < n; i++) {
            if ((isScramble(s1.substring(0, i), s2.substring(0, i)) &&
                 isScramble(s1.substring(i), s2.substring(i))) ||
                (isScramble(s1.substring(0, i), s2.substring(n - i)) &&
                 isScramble(s1.substring(i), s2.substring(0, n - i)))) {
                memo.put(key, true); return true;
            }
        }
        memo.put(key, false); return false;
    }
}
```

---

### 9. Zuma Game (LC 488)
**Description:** Given a board of colored balls and a hand of balls, insert balls from your hand into the board to remove groups of 3+ same-colored consecutive balls. Return the minimum balls needed from hand to clear the board, or -1 if impossible.

**Example:**
- Input: board = "WRRBBW", hand = "RB"
- Output: -1  (cannot clear: no way to create 3 consecutive of any color)

```java
class Solution {
    Map<String, Integer> memo = new HashMap<>();

    public int findMinStep(String board, String hand) {
        int[] handCount = new int[26];
        for (char c : hand.toCharArray()) handCount[c - 'A']++;
        int res = dfs(board, handCount);
        return res == Integer.MAX_VALUE ? -1 : res;
    }

    private int dfs(String board, int[] hand) {
        if (board.isEmpty()) return 0;
        String key = board + Arrays.toString(hand);
        if (memo.containsKey(key)) return memo.get(key);
        int res = Integer.MAX_VALUE;
        int i = 0;
        while (i < board.length()) {
            int j = i;
            while (j < board.length() && board.charAt(j) == board.charAt(i)) j++;
            int need = 3 - (j - i);
            if (hand[board.charAt(i) - 'A'] >= need) {
                hand[board.charAt(i) - 'A'] -= need;
                String next = clean(board.substring(0, i) + board.substring(j));
                int sub = dfs(next, hand);
                if (sub != Integer.MAX_VALUE) res = Math.min(res, sub + need);
                hand[board.charAt(i) - 'A'] += need;
            }
            i = j;
        }
        memo.put(key, res);
        return res;
    }

    private String clean(String s) {
        StringBuilder sb = new StringBuilder(s);
        int i = 0;
        while (i < sb.length()) {
            int j = i;
            while (j < sb.length() && sb.charAt(j) == sb.charAt(i)) j++;
            if (j - i >= 3) { sb.delete(i, j); i = 0; }
            else i = j;
        }
        return sb.toString();
    }
}
```

---

### 10. Optimal BST (Classic)
**Description:** Given n keys in sorted order, each with a search frequency, build a Binary Search Tree that minimizes the expected search cost (sum of depth × frequency for all keys).

**Example:**
- Input: freq = [34,8,50,3]  (keys are 1,2,3,4 with these frequencies)
- Output: 142  (optimal BST puts key 3 at root, minimizing weighted depth)

```java
public int optimalBST(int[] freq) {
    int n = freq.length;
    int[][] dp = new int[n][n];
    int[] prefix = new int[n + 1];
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + freq[i];

    for (int len = 1; len <= n; len++)
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            int rangeSum = prefix[j + 1] - prefix[i];
            dp[i][j] = Integer.MAX_VALUE;
            for (int k = i; k <= j; k++) {
                int left = k > i ? dp[i][k - 1] : 0;
                int right = k < j ? dp[k + 1][j] : 0;
                dp[i][j] = Math.min(dp[i][j], left + right + rangeSum);
            }
        }
    return dp[0][n - 1];
}
```

---

---

# 5. Sequence DP

## How to Identify

- **Two strings/sequences** as input
- Compare characters from both strings
- Keywords: "edit distance", "longest common", "interleaving", "matching"
- State needs two indices: `dp[i][j]` for prefix of each string

## Template

```
      ""  s2[0]  s2[1]  ...  s2[n]
  ""   ?    ?      ?           ?
s1[0]  ?    ?      ?           ?
s1[1]  ?    ?      ?           ?
 ...
s1[m]  ?    ?      ?        answer
```

Fill left → right, top → bottom.

## Skeleton

```java
int[][] dp = new int[m + 1][n + 1];
// fill base cases: dp[0][j] and dp[i][0]

for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        if (s1.charAt(i-1) == s2.charAt(j-1))
            dp[i][j] = /* match case */;
        else
            dp[i][j] = /* mismatch case */;

return dp[m][n];
```

## Character Comparison Pattern

| Problem | Match | Mismatch |
|---|---|---|
| LCS | `dp[i-1][j-1] + 1` | `max(dp[i-1][j], dp[i][j-1])` |
| Edit Distance | `dp[i-1][j-1]` | `min(up, left, diag) + 1` |
| Interleaving | `dp[i-1][j] \|\| dp[i][j-1]` | — |
| Wildcard | `dp[i-1][j-1]` | special `*` handling |

---

## Problems

### 1. Longest Common Subsequence (LC 1143)
**Description:** Given two strings text1 and text2, return the length of their longest common subsequence. A subsequence need not be contiguous but must maintain relative order.

**Example:**
- Input: text1 = "abcde", text2 = "ace"
- Output: 3  ("ace" is the longest common subsequence)

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                if (text1.charAt(i-1) == text2.charAt(j-1))
                    dp[i][j] = dp[i-1][j-1] + 1;
                else
                    dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
        return dp[m][n];
    }
}
```

---

### 2. Edit Distance (LC 72)
**Description:** Given two strings word1 and word2, return the minimum number of operations (insert, delete, or replace a character) to convert word1 to word2.

**Example:**
- Input: word1 = "horse", word2 = "ros"
- Output: 3  (horse→rorse→rose→ros, 3 operations)

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) dp[i][0] = i;
        for (int j = 1; j <= n; j++) dp[0][j] = j;
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                if (word1.charAt(i-1) == word2.charAt(j-1))
                    dp[i][j] = dp[i-1][j-1];
                else
                    dp[i][j] = Math.min(Math.min(dp[i-1][j], dp[i][j-1]), dp[i-1][j-1]) + 1;
        return dp[m][n];
    }
}
```

---

### 3. Interleaving String (LC 97)
**Description:** Given strings s1, s2, and s3, determine if s3 is formed by interleaving s1 and s2 (preserving relative order of characters from each string).

**Example:**
- Input: s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
- Output: true  (s1[0,1] + s2[0,3] + s1[2,4] + s2[4] = "aa"+"dbbc"+"cc"+"a")

```java
class Solution {
    public boolean isInterleave(String s1, String s2, String s3) {
        int m = s1.length(), n = s2.length();
        if (m + n != s3.length()) return false;
        boolean[][] dp = new boolean[m + 1][n + 1];
        dp[0][0] = true;
        for (int i = 1; i <= m; i++) dp[i][0] = dp[i-1][0] && s1.charAt(i-1) == s3.charAt(i-1);
        for (int j = 1; j <= n; j++) dp[0][j] = dp[0][j-1] && s2.charAt(j-1) == s3.charAt(j-1);
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                dp[i][j] = (dp[i-1][j] && s1.charAt(i-1) == s3.charAt(i+j-1))
                         || (dp[i][j-1] && s2.charAt(j-1) == s3.charAt(i+j-1));
        return dp[m][n];
    }
}
```

---

### 4. Distinct Subsequences (LC 115)
**Description:** Given strings s and t, return the number of distinct ways to choose characters from s to form t as a subsequence (characters must maintain relative order).

**Example:**
- Input: s = "rabbbit", t = "rabbit"
- Output: 3  (three ways to pick "rabbit" from "rabbbit" by choosing different b's)

```java
class Solution {
    public int numDistinct(String s, String t) {
        int m = s.length(), n = t.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 0; i <= m; i++) dp[i][0] = 1;
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++) {
                dp[i][j] = dp[i-1][j];
                if (s.charAt(i-1) == t.charAt(j-1))
                    dp[i][j] += dp[i-1][j-1];
            }
        return dp[m][n];
    }
}
```

---

### 5. Shortest Common Supersequence (LC 1092)
**Description:** Given two strings str1 and str2, return the shortest string that has both str1 and str2 as subsequences. If there are multiple valid outputs, return any of them.

**Example:**
- Input: str1 = "abac", str2 = "cab"
- Output: "cabac"  (length 5; "abac" and "cab" are both subsequences)

```java
class Solution {
    public String shortestCommonSupersequence(String str1, String str2) {
        int m = str1.length(), n = str2.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                if (str1.charAt(i-1) == str2.charAt(j-1))
                    dp[i][j] = dp[i-1][j-1] + 1;
                else
                    dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
        // reconstruct
        StringBuilder sb = new StringBuilder();
        int i = m, j = n;
        while (i > 0 && j > 0) {
            if (str1.charAt(i-1) == str2.charAt(j-1)) {
                sb.append(str1.charAt(i-1)); i--; j--;
            } else if (dp[i-1][j] > dp[i][j-1]) {
                sb.append(str1.charAt(i-1)); i--;
            } else {
                sb.append(str2.charAt(j-1)); j--;
            }
        }
        while (i > 0) { sb.append(str1.charAt(i-1)); i--; }
        while (j > 0) { sb.append(str2.charAt(j-1)); j--; }
        return sb.reverse().toString();
    }
}
```

---

### 6. Wildcard Matching (LC 44)
**Description:** Given an input string s and a pattern p with '?' (matches any single char) and '*' (matches any sequence including empty), determine if the pattern matches the whole string.

**Example:**
- Input: s = "adceb", p = "*a*b"
- Output: true  (* matches "", then "a", * matches "dce", then "b")

```java
class Solution {
    public boolean isMatch(String s, String p) {
        int m = s.length(), n = p.length();
        boolean[][] dp = new boolean[m + 1][n + 1];
        dp[0][0] = true;
        for (int j = 1; j <= n; j++) dp[0][j] = dp[0][j-1] && p.charAt(j-1) == '*';
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++) {
                char pc = p.charAt(j-1);
                if (pc == '*')
                    dp[i][j] = dp[i-1][j] || dp[i][j-1];
                else
                    dp[i][j] = dp[i-1][j-1] && (pc == '?' || pc == s.charAt(i-1));
            }
        return dp[m][n];
    }
}
```

---

### 7. Regular Expression Matching (LC 10)
**Description:** Implement regular expression matching with '.' (matches any single character) and '*' (matches zero or more of the preceding element).

**Example:**
- Input: s = "aab", p = "c*a*b"
- Output: true  (c*=0 c's, a*=2 a's, b=b → "aab")

```java
class Solution {
    public boolean isMatch(String s, String p) {
        int m = s.length(), n = p.length();
        boolean[][] dp = new boolean[m + 1][n + 1];
        dp[0][0] = true;
        for (int j = 2; j <= n; j += 2)
            if (p.charAt(j-1) == '*') dp[0][j] = dp[0][j-2];
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++) {
                char pc = p.charAt(j-1);
                if (pc == '*') {
                    dp[i][j] = dp[i][j-2];  // use * as zero
                    if (p.charAt(j-2) == '.' || p.charAt(j-2) == s.charAt(i-1))
                        dp[i][j] = dp[i][j] || dp[i-1][j];  // use * as one+
                } else {
                    dp[i][j] = dp[i-1][j-1] && (pc == '.' || pc == s.charAt(i-1));
                }
            }
        return dp[m][n];
    }
}
```

---

### 8. Delete Operation for Two Strings (LC 583)
**Description:** Given two strings word1 and word2, return the minimum number of steps to make them equal, where each step deletes one character from either string.

**Example:**
- Input: word1 = "sea", word2 = "eat"
- Output: 2  (delete 's' from word1, delete 't' from word2 → both become "ea")

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                if (word1.charAt(i-1) == word2.charAt(j-1))
                    dp[i][j] = dp[i-1][j-1] + 1;
                else
                    dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
        int lcs = dp[m][n];
        return m + n - 2 * lcs;  // delete everything not in LCS
    }
}
```

---

### 9. Minimum ASCII Delete Sum (LC 712)
**Description:** Given two strings s1 and s2, return the lowest ASCII sum of deleted characters to make them equal.

**Example:**
- Input: s1 = "sea", s2 = "eat"
- Output: 231  (delete 's'(115) from s1 and 't'(116) from s2 → both "ea"; 115+116=231)

```java
class Solution {
    public int minimumDeleteSum(String s1, String s2) {
        int m = s1.length(), n = s2.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) dp[i][0] = dp[i-1][0] + s1.charAt(i-1);
        for (int j = 1; j <= n; j++) dp[0][j] = dp[0][j-1] + s2.charAt(j-1);
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                if (s1.charAt(i-1) == s2.charAt(j-1))
                    dp[i][j] = dp[i-1][j-1];
                else
                    dp[i][j] = Math.min(dp[i-1][j] + s1.charAt(i-1),
                                        dp[i][j-1] + s2.charAt(j-1));
        return dp[m][n];
    }
}
```

---

### 10. Longest Common Substring (Classic — not subsequence)
**Description:** Find the length of the longest contiguous substring common to both strings (characters must be adjacent, unlike LCS).

**Example:**
- Input: s1 = "abcde", s2 = "abgde"
- Output: 2  ("ab" is the longest contiguous common part before they diverge)

```java
public int longestCommonSubstring(String s1, String s2) {
    int m = s1.length(), n = s2.length(), max = 0;
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i-1) == s2.charAt(j-1)) {
                dp[i][j] = dp[i-1][j-1] + 1;
                max = Math.max(max, dp[i][j]);
            }
            // no else — reset to 0 (default) if mismatch
        }
    return max;
}
```
**Key difference from LCS**: mismatch resets to 0 (must be contiguous). No `max(up, left)`.

---

---

# 6. DP on Trees

## How to Identify

- Input is a **tree** (not a grid or array)
- Answer at a node depends on answers from its **children**
- Keywords: "diameter", "path sum", "rob houses on tree", "cameras"
- Post-order traversal (process children before parent)

## Key Insight

Tree DP = DFS + memoization at each node. Each node returns one or more values up to its parent. Parent combines children's values to compute its own answer.

## Skeleton

```java
// returns [include_root, exclude_root] or single value
private int[] dfs(TreeNode node) {
    if (node == null) return new int[]{0, 0};

    int[] left = dfs(node.left);
    int[] right = dfs(node.right);

    int include = node.val + left[1] + right[1];  // take root, skip children
    int exclude = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);

    return new int[]{include, exclude};
}
```

## Return Value Patterns

| Pattern | When to use |
|---|---|
| Single int | diameter, height, path sum |
| `int[2]` {take, skip} | House Robber, cameras |
| `int[3]` {covered, not covered, has camera} | Binary Tree Cameras |
| Global variable + local return | diameter (max updated globally) |

---

## Problems

### 1. Diameter of Binary Tree (LC 543)
**Description:** Given a binary tree, return the length of its diameter — the longest path between any two nodes. The path may or may not pass through the root.

**Example:**
- Input: root = [1,2,3,4,5]
- Output: 3  (longest path: [4,2,1,3] or [5,2,1,3], length = 3 edges)

```java
class Solution {
    int diameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        height(root);
        return diameter;
    }

    private int height(TreeNode node) {
        if (node == null) return 0;
        int left = height(node.left);
        int right = height(node.right);
        diameter = Math.max(diameter, left + right);  // update global
        return 1 + Math.max(left, right);             // return height to parent
    }
}
```
**Pattern**: return height upward, update answer globally.

---

### 2. House Robber III (LC 337)
**Description:** Houses are arranged on a binary tree. Adjacent nodes (parent-child) cannot both be robbed. Find the maximum amount you can rob.

**Example:**
- Input: root = [3,2,3,null,3,null,1]
- Output: 7  (rob root(3) + grandchildren(3+1) = 7)

```java
class Solution {
    public int rob(TreeNode root) {
        int[] res = dfs(root);
        return Math.max(res[0], res[1]);
    }

    // returns [rob_this_node, skip_this_node]
    private int[] dfs(TreeNode node) {
        if (node == null) return new int[]{0, 0};
        int[] left = dfs(node.left);
        int[] right = dfs(node.right);
        int rob = node.val + left[1] + right[1];         // take root, skip children
        int skip = Math.max(left[0], left[1])
                 + Math.max(right[0], right[1]);          // best of each child
        return new int[]{rob, skip};
    }
}
```

---

### 3. Binary Tree Maximum Path Sum (LC 124)
**Description:** A path in a binary tree is a sequence of nodes where each pair of adjacent nodes has an edge between them. Find the maximum sum of any non-empty path (can start and end anywhere).

**Example:**
- Input: root = [-10,9,20,null,null,15,7]
- Output: 42  (path 15→20→7 = 42)

```java
class Solution {
    int max = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        dfs(root);
        return max;
    }

    private int dfs(TreeNode node) {
        if (node == null) return 0;
        int left = Math.max(0, dfs(node.left));   // ignore negative paths
        int right = Math.max(0, dfs(node.right));
        max = Math.max(max, node.val + left + right);  // path through root
        return node.val + Math.max(left, right);        // return one side to parent
    }
}
```
**Key**: path through a node uses BOTH children. Return only ONE side to parent (can't fork upward).

---

### 4. Binary Tree Cameras (LC 968)
**Description:** Place cameras on some nodes of a binary tree. Each camera monitors its parent, itself, and its children. Return the minimum number of cameras needed to monitor all nodes.

**Example:**
- Input: root = [0,0,null,0,null,0,null,null,0]
- Output: 2  (place cameras at depth 2 and 4)

```java
class Solution {
    int cameras = 0;

    public int minCameraCover(TreeNode root) {
        // 0 = not covered, 1 = covered no camera, 2 = has camera
        if (dfs(root) == 0) cameras++;  // root not covered
        return cameras;
    }

    private int dfs(TreeNode node) {
        if (node == null) return 1;  // null = covered (doesn't need camera)
        int left = dfs(node.left), right = dfs(node.right);
        if (left == 0 || right == 0) { cameras++; return 2; }  // child uncovered
        if (left == 2 || right == 2) return 1;                  // child has camera
        return 0;                                                // not covered yet
    }
}
```

---

### 5. Longest Univalue Path (LC 687)
**Description:** Given a binary tree, return the length of the longest path where each node in the path has the same value. The path can pass through the root of any subtree.

**Example:**
- Input: root = [5,4,5,1,1,null,5]
- Output: 2  (path 5→5→5 through right children has length 2)

```java
class Solution {
    int ans = 0;

    public int longestUnivaluePath(TreeNode root) {
        dfs(root);
        return ans;
    }

    private int dfs(TreeNode node) {
        if (node == null) return 0;
        int left = dfs(node.left), right = dfs(node.right);
        int leftPath = (node.left != null && node.left.val == node.val) ? left + 1 : 0;
        int rightPath = (node.right != null && node.right.val == node.val) ? right + 1 : 0;
        ans = Math.max(ans, leftPath + rightPath);
        return Math.max(leftPath, rightPath);
    }
}
```

---

### 6. Count Good Nodes in Binary Tree (LC 1448)
**Description:** A node X is "good" if there is no node with a greater value on the path from the root to X. Count all good nodes in the binary tree.

**Example:**
- Input: root = [3,1,4,3,null,1,5]
- Output: 4  (nodes 3,4,3,5 are all good; 1s are not because 3 is on path above them)

```java
class Solution {
    public int goodNodes(TreeNode root) {
        return dfs(root, Integer.MIN_VALUE);
    }

    private int dfs(TreeNode node, int maxSoFar) {
        if (node == null) return 0;
        int count = node.val >= maxSoFar ? 1 : 0;
        int newMax = Math.max(maxSoFar, node.val);
        return count + dfs(node.left, newMax) + dfs(node.right, newMax);
    }
}
```

---

### 7. Path Sum III (LC 437)
**Description:** Count the number of paths in a binary tree that sum to a given target. Paths must go downward but need not start at root or end at a leaf.

**Example:**
- Input: root = [10,5,-3,3,2,null,11,3,-2,null,1], targetSum = 8
- Output: 3  (paths: 5→3, 5→2→1, -3→11 all sum to 8)

```java
class Solution {
    public int pathSum(TreeNode root, int targetSum) {
        Map<Long, Integer> prefixCount = new HashMap<>();
        prefixCount.put(0L, 1);
        return dfs(root, 0L, targetSum, prefixCount);
    }

    private int dfs(TreeNode node, long curr, int target, Map<Long, Integer> map) {
        if (node == null) return 0;
        curr += node.val;
        int count = map.getOrDefault(curr - target, 0);
        map.merge(curr, 1, Integer::sum);
        count += dfs(node.left, curr, target, map) + dfs(node.right, curr, target, map);
        map.merge(curr, -1, Integer::sum);  // backtrack
        return count;
    }
}
```
**Pattern**: prefix sum on tree path — same idea as prefix sum on array.

---

### 8. Distribute Coins in Binary Tree (LC 979)
**Description:** Each node in a binary tree has some coins; total coins = total nodes = n. Move coins so every node has exactly 1. Return the minimum number of moves (each move transfers 1 coin across one edge).

**Example:**
- Input: root = [3,0,0]
- Output: 2  (move 2 coins from root: 1 left, 1 right)

```java
class Solution {
    int moves = 0;

    public int distributeCoins(TreeNode root) {
        dfs(root);
        return moves;
    }

    private int dfs(TreeNode node) {
        if (node == null) return 0;
        int left = dfs(node.left), right = dfs(node.right);
        moves += Math.abs(left) + Math.abs(right);
        return node.val + left + right - 1;  // excess coins (can be negative)
    }
}
```

---

### 9. Maximum Product of Splitted Binary Tree (LC 1339)
**Description:** Split a binary tree into two by removing one edge. Maximize the product of the two subtree sums. Return the answer modulo 10^9 + 7.

**Example:**
- Input: root = [1,2,3,4,5,6]
- Output: 110  (split gives subtrees with sums 11 and 10; 11×10=110)

```java
class Solution {
    long total = 0, ans = 0;
    static final long MOD = 1_000_000_007;

    public int maxProduct(TreeNode root) {
        total = subtreeSum(root);
        subtreeSum2(root);
        return (int)(ans % MOD);
    }

    private long subtreeSum(TreeNode node) {
        if (node == null) return 0;
        return node.val + subtreeSum(node.left) + subtreeSum(node.right);
    }

    private long subtreeSum2(TreeNode node) {
        if (node == null) return 0;
        long s = node.val + subtreeSum2(node.left) + subtreeSum2(node.right);
        ans = Math.max(ans, s * (total - s));
        return s;
    }
}
```

---

### 10. Unique Binary Search Trees II (LC 95)
**Description:** Given an integer n, return all structurally unique BSTs containing exactly the values 1 to n. Each value appears exactly once.

**Example:**
- Input: n = 3
- Output: [[1,null,2,null,3],[1,null,3,2],[2,1,3],[3,1,null,null,2],[3,2,null,1]]  (5 trees)

```java
class Solution {
    public List<TreeNode> generateTrees(int n) {
        return generate(1, n);
    }

    private List<TreeNode> generate(int lo, int hi) {
        List<TreeNode> res = new ArrayList<>();
        if (lo > hi) { res.add(null); return res; }
        for (int root = lo; root <= hi; root++) {
            for (TreeNode left : generate(lo, root - 1))
                for (TreeNode right : generate(root + 1, hi)) {
                    TreeNode node = new TreeNode(root);
                    node.left = left; node.right = right;
                    res.add(node);
                }
        }
        return res;
    }
}
```

---

---

# 7. Bitmask DP

## How to Identify

- **Small n** (usually n ≤ 20)
- Need to track **which items/nodes have been visited**
- Keywords: "visit all", "assign to", "cover all", "shortest path visiting all nodes"
- State includes a **bitmask** representing the set of visited items

## Core Idea

Use an integer as a set. Bit `i` = 1 means item `i` is included.

```java
// Check if bit i is set
(mask >> i) & 1 == 1

// Set bit i
mask | (1 << i)

// Clear bit i
mask & ~(1 << i)

// All n bits set (all visited)
(1 << n) - 1

// Iterate over all subsets
for (int mask = 0; mask < (1 << n); mask++)

// Iterate over set bits
for (int sub = mask; sub > 0; sub = (sub - 1) & mask)
```

## Skeleton

```java
int n = ...; // small, ≤ 20
int FULL = (1 << n) - 1;
int[][] dp = new int[1 << n][n];
// dp[mask][i] = answer when visited set = mask, currently at node i

for (int mask = 0; mask <= FULL; mask++)
    for (int i = 0; i < n; i++) {
        if ((mask >> i & 1) == 0) continue;  // i must be in mask
        for (int j = 0; j < n; j++) {
            if ((mask >> j & 1) == 1) continue;  // j not yet visited
            int newMask = mask | (1 << j);
            dp[newMask][j] = Math.min(dp[newMask][j], dp[mask][i] + cost[i][j]);
        }
    }
```

---

## Problems

### 1. Traveling Salesman Problem (Classic)
**Description:** Given n cities and distances between every pair, find the shortest tour that visits every city exactly once and returns to the starting city. (NP-hard, but DP with bitmask solves it in O(2ⁿ × n²).)

**Example:**
- Input: dist = [[0,10,15,20],[10,0,35,25],[15,35,0,30],[20,25,30,0]]
- Output: 80  (tour: 0→1→3→2→0, cost = 10+25+30+15 = 80)

```java
public int tsp(int[][] dist) {
    int n = dist.length;
    int FULL = (1 << n) - 1;
    int[][] dp = new int[1 << n][n];
    for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE / 2);
    dp[1][0] = 0;  // start at city 0, visited = {0}

    for (int mask = 1; mask <= FULL; mask++) {
        for (int u = 0; u < n; u++) {
            if ((mask >> u & 1) == 0) continue;
            for (int v = 0; v < n; v++) {
                if ((mask >> v & 1) == 1) continue;
                int newMask = mask | (1 << v);
                dp[newMask][v] = Math.min(dp[newMask][v], dp[mask][u] + dist[u][v]);
            }
        }
    }

    int ans = Integer.MAX_VALUE;
    for (int u = 1; u < n; u++)
        ans = Math.min(ans, dp[FULL][u] + dist[u][0]);
    return ans;
}
```

---

### 2. Minimum Cost to Visit All Nodes (LC 847)
**Description:** An undirected connected graph with n nodes. Find the shortest path length that visits every node (can revisit nodes). Return the shortest path distance.

**Example:**
- Input: graph = [[1,2,3],[0],[0],[0]]
- Output: 4  (visit 1,2,3 starting from 0 and returning: 0→1→0→2→0→3 = 5... optimal is 4)

```java
class Solution {
    public int shortestPathLength(int[][] graph) {
        int n = graph.length;
        int FULL = (1 << n) - 1;
        int[][] dist = new int[1 << n][n];
        for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);

        Queue<int[]> queue = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            dist[1 << i][i] = 0;
            queue.offer(new int[]{1 << i, i});
        }

        while (!queue.isEmpty()) {
            int[] curr = queue.poll();
            int mask = curr[0], u = curr[1];
            for (int v : graph[u]) {
                int newMask = mask | (1 << v);
                if (dist[newMask][v] > dist[mask][u] + 1) {
                    dist[newMask][v] = dist[mask][u] + 1;
                    queue.offer(new int[]{newMask, v});
                }
            }
        }

        int ans = Integer.MAX_VALUE;
        for (int u = 0; u < n; u++)
            ans = Math.min(ans, dist[FULL][u]);
        return ans;
    }
}
```

---

### 3. Partition into K Equal Subsets (LC 698)
**Description:** Given an integer array and integer k, determine if it is possible to divide the array into k non-empty subsets whose sums are all equal.

**Example:**
- Input: nums = [4,3,2,3,5,2,1], k = 4
- Output: true  ([5],[1,4],[2,3],[2,3] each sum to 5)

```java
class Solution {
    public boolean canPartitionKSubsets(int[] nums, int k) {
        int sum = 0;
        for (int n : nums) sum += n;
        if (sum % k != 0) return false;
        int target = sum / k, n = nums.length;
        int[] dp = new int[1 << n];
        Arrays.fill(dp, -1);
        dp[0] = 0;
        Arrays.sort(nums);

        for (int mask = 0; mask < (1 << n); mask++) {
            if (dp[mask] == -1) continue;
            for (int i = 0; i < n; i++) {
                if ((mask >> i & 1) == 1) continue;
                int next = mask | (1 << i);
                if (dp[mask] + nums[i] <= target)
                    dp[next] = (dp[mask] + nums[i]) % target;
            }
        }
        return dp[(1 << n) - 1] == 0;
    }
}
```

---

### 4. Maximum AND Sum of Array (LC 2172)
**Description:** Given an integer array of length n and numSlots slots (each holding at most 2 numbers), assign all numbers to slots to maximize the sum of (number AND slot_index) for each assignment.

**Example:**
- Input: nums = [1,2,3,4,5,6], numSlots = 3
- Output: 9  (assign 1→1, 2→2, 3→3 → AND sums: 1,2,3; assign rest → 4&1+5&2+6&3=0+0+2=2; total 9)

```java
class Solution {
    public int maximumANDSum(int[] nums, int numSlots) {
        int n = nums.length;
        int[] dp = new int[1 << (numSlots * 2)];  // each slot has 2 bits
        int ans = 0;
        for (int mask = 0; mask < dp.length; mask++) {
            int count = Integer.bitCount(mask);
            if (count >= n) continue;
            for (int slot = 1; slot <= numSlots; slot++) {
                int bit1 = (slot - 1) * 2, bit2 = bit1 + 1;
                for (int bit : new int[]{bit1, bit2}) {
                    if ((mask >> bit & 1) == 0) {
                        int newMask = mask | (1 << bit);
                        dp[newMask] = Math.max(dp[newMask], dp[mask] + (nums[count] & slot));
                        ans = Math.max(ans, dp[newMask]);
                    }
                }
            }
        }
        return ans;
    }
}
```

---

### 5. Stickers to Spell Word (LC 691)
**Description:** Given an array of sticker strings and a target string, find the minimum number of stickers to spell out the target. Each sticker can be used multiple times. Letters can be cut from stickers individually.

**Example:**
- Input: stickers = ["with","example","science"], target = "thehat"
- Output: 3  (use "with" + "example" + "thehat"... use stickers "with", "example", "science" strategically)

```java
class Solution {
    public int minStickers(String[] stickers, String target) {
        int n = target.length();
        int[] dp = new int[1 << n];
        Arrays.fill(dp, Integer.MAX_VALUE);
        dp[0] = 0;

        for (int mask = 0; mask < (1 << n); mask++) {
            if (dp[mask] == Integer.MAX_VALUE) continue;
            for (String sticker : stickers) {
                int cur = mask;
                int[] freq = new int[26];
                for (char c : sticker.toCharArray()) freq[c - 'a']++;
                for (int i = 0; i < n; i++) {
                    if ((cur >> i & 1) == 1) continue;
                    char c = target.charAt(i);
                    if (freq[c - 'a'] > 0) { freq[c - 'a']--; cur |= (1 << i); }
                }
                dp[cur] = Math.min(dp[cur], dp[mask] + 1);
            }
        }
        return dp[(1 << n) - 1] == Integer.MAX_VALUE ? -1 : dp[(1 << n) - 1];
    }
}
```

---

### 6. Find the Shortest Superstring (LC 943)
**Description:** Given an array of strings words, find the shortest string that contains each word in words as a substring. If multiple valid answers exist, return any of them.

**Example:**
- Input: words = ["alex","loves","leetcode"]
- Output: "alexlovesleetcode"  (or any valid shortest superstring)

```java
class Solution {
    public String shortestSuperstring(String[] words) {
        int n = words.length;
        int[][] overlap = new int[n][n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++) if (i != j) {
                String a = words[i], b = words[j];
                for (int k = Math.min(a.length(), b.length()); k >= 0; k--)
                    if (b.startsWith(a.substring(a.length() - k))) { overlap[i][j] = k; break; }
            }

        int[][] dp = new int[1 << n][n];
        int[][] parent = new int[1 << n][n];
        for (int[] row : parent) Arrays.fill(row, -1);

        for (int mask = 1; mask < (1 << n); mask++)
            for (int last = 0; last < n; last++) {
                if ((mask >> last & 1) == 0) continue;
                int prev = mask ^ (1 << last);
                if (prev == 0) continue;
                for (int prev_last = 0; prev_last < n; prev_last++) {
                    if ((prev >> prev_last & 1) == 0) continue;
                    int val = dp[prev][prev_last] + overlap[prev_last][last];
                    if (val > dp[mask][last]) { dp[mask][last] = val; parent[mask][last] = prev_last; }
                }
            }

        int FULL = (1 << n) - 1, last = 0;
        for (int i = 1; i < n; i++) if (dp[FULL][i] > dp[FULL][last]) last = i;

        // reconstruct
        StringBuilder sb = new StringBuilder();
        int mask = FULL;
        List<Integer> order = new ArrayList<>();
        while (last != -1) {
            order.add(last);
            int tmp = parent[mask][last];
            mask ^= (1 << last);
            last = tmp;
        }
        Collections.reverse(order);
        sb.append(words[order.get(0)]);
        for (int i = 1; i < order.size(); i++) {
            int prev = order.get(i-1), cur = order.get(i);
            sb.append(words[cur].substring(overlap[prev][cur]));
        }
        return sb.toString();
    }
}
```

---

### 7. Number of Ways to Wear Different Hats (LC 1434)
**Description:** There are n people and 40 different types of hats. Each person has a list of preferred hats. Count the number of ways to assign each person a different hat (from their preference list). Return result mod 10^9+7.

**Example:**
- Input: hats = [[3,4],[4,5],[5]]
- Output: 1  (only one valid assignment: person 0→hat3, person 1→hat4, person 2→hat5)

```java
class Solution {
    static final int MOD = 1_000_000_007;

    public int numberWays(List<List<Integer>> hats) {
        int n = hats.size();
        List<Integer>[] people = new List[41];
        for (int i = 1; i <= 40; i++) people[i] = new ArrayList<>();
        for (int i = 0; i < n; i++)
            for (int h : hats.get(i)) people[h].add(i);

        long[] dp = new long[1 << n];
        dp[0] = 1;
        int FULL = (1 << n) - 1;

        for (int hat = 1; hat <= 40; hat++) {
            long[] next = dp.clone();
            for (int mask = 0; mask <= FULL; mask++) {
                if (dp[mask] == 0) continue;
                for (int person : people[hat]) {
                    if ((mask >> person & 1) == 0)
                        next[mask | (1 << person)] = (next[mask | (1 << person)] + dp[mask]) % MOD;
                }
            }
            dp = next;
        }
        return (int) dp[FULL];
    }
}
```

---

### 8. Minimum XOR Sum of Two Arrays (LC 1879)
**Description:** Given two integer arrays nums1 and nums2, permute nums2 to minimize the sum of XOR of corresponding pairs: sum of (nums1[i] XOR nums2[perm[i]]).

**Example:**
- Input: nums1 = [1,2], nums2 = [2,3]
- Output: 2  (pair (1,3) and (2,2): 1^3 + 2^2 = 2+0 = 2)

```java
class Solution {
    public int minimumXORSum(int[] nums1, int[] nums2) {
        int n = nums1.length;
        int[] dp = new int[1 << n];
        Arrays.fill(dp, Integer.MAX_VALUE);
        dp[0] = 0;
        for (int mask = 0; mask < (1 << n); mask++) {
            if (dp[mask] == Integer.MAX_VALUE) continue;
            int i = Integer.bitCount(mask);  // index in nums1
            if (i == n) continue;
            for (int j = 0; j < n; j++) {
                if ((mask >> j & 1) == 1) continue;
                dp[mask | (1 << j)] = Math.min(dp[mask | (1 << j)],
                    dp[mask] + (nums1[i] ^ nums2[j]));
            }
        }
        return dp[(1 << n) - 1];
    }
}
```

---

### 9. Matchsticks to Square (LC 473)
**Description:** Given an array of matchstick lengths, determine if you can form a perfect square using all matchsticks. Each matchstick must be used exactly once.

**Example:**
- Input: matchsticks = [1,1,2,2,2]
- Output: true  (four sides: [1,1],[2],[2],[2] each length 2 → square with side 2)

```java
class Solution {
    public boolean makesquare(int[] matchsticks) {
        int sum = 0;
        for (int m : matchsticks) sum += m;
        if (sum % 4 != 0) return false;
        int target = sum / 4, n = matchsticks.length;
        int[] dp = new int[1 << n];
        Arrays.fill(dp, -1);
        dp[0] = 0;
        Arrays.sort(matchsticks);

        for (int mask = 1; mask < (1 << n); mask++) {
            for (int i = 0; i < n; i++) {
                if ((mask >> i & 1) == 0) continue;
                int prev = mask ^ (1 << i);
                if (dp[prev] < 0) continue;
                int cur = dp[prev] + matchsticks[i];
                if (cur <= target) { dp[mask] = cur % target; break; }
            }
        }
        return dp[(1 << n) - 1] == 0;
    }
}
```

---

### 10. Maximize Score After N Operations (LC 1799)
**Description:** You have 2n numbers. In the i-th operation (1-indexed), pick two numbers, score += i × gcd(a,b), then remove them. Maximize total score after n operations.

**Example:**
- Input: nums = [3,4,6,8]
- Output: 11  (op1: gcd(3,6)=3 → score 3; op2: gcd(4,8)=4 → score 8; total = 3+8=11)

```java
class Solution {
    public int maxScore(int[] nums) {
        int n = nums.length;
        int[][] gcd = new int[n][n];
        for (int i = 0; i < n; i++)
            for (int j = i + 1; j < n; j++)
                gcd[i][j] = gcd(nums[i], nums[j]);

        int[] dp = new int[1 << n];
        for (int mask = 0; mask < (1 << n); mask++) {
            int bits = Integer.bitCount(mask);
            if (bits % 2 != 0) continue;
            int op = bits / 2 + 1;
            for (int i = 0; i < n; i++) {
                if ((mask >> i & 1) == 0) continue;
                for (int j = i + 1; j < n; j++) {
                    if ((mask >> j & 1) == 0) continue;
                    int prev = mask ^ (1 << i) ^ (1 << j);
                    dp[mask] = Math.max(dp[mask], dp[prev] + op * gcd[i][j]);
                }
            }
        }
        return dp[(1 << n) - 1];
    }

    private int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }
}
```

---

---

# 8. Top-Down vs Bottom-Up

## When to Use Each

| | Top-Down (Memoization) | Bottom-Up (Tabulation) |
|---|---|---|
| Style | Recursion + cache | Iterative table |
| State space | Only computes needed states | Computes all states |
| Code clarity | Closer to brute force, easier to write | More structured, harder to set up |
| Stack overflow | Risk on deep recursion | No risk |
| When to prefer | Complex state, sparse subproblems, easier to think recursively | Simple state, dense subproblems, need space optimization |

## Top-Down Template

```java
Map<String, Integer> memo = new HashMap<>();  // or int[] / int[][]

private int dp(int i, int j, /* other state */) {
    // 1. base case
    if (i < 0 || j < 0) return 0;

    // 2. check cache
    String key = i + "," + j;
    if (memo.containsKey(key)) return memo.get(key);

    // 3. recurrence
    int result = /* combine subproblems */;

    // 4. cache and return
    memo.put(key, result);
    return result;
}
```

## Bottom-Up Template

```java
int[][] dp = new int[m + 1][n + 1];

// 1. base cases
for (int i = 0; i <= m; i++) dp[i][0] = ...;
for (int j = 0; j <= n; j++) dp[0][j] = ...;

// 2. fill table
for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        dp[i][j] = /* recurrence */;

return dp[m][n];
```

## Decision Guide

```
Start with top-down when:
  - State has 3+ dimensions (Cherry Pickup, Bitmask)
  - Not all states are reachable (sparse)
  - Recursion is natural (tree DP)

Switch to bottom-up when:
  - Need space optimization (rolling array)
  - State is 1D or 2D and all states needed
  - Stack depth is a concern
```

## Converting Top-Down to Bottom-Up

```
Top-down:    dp(i, j) uses dp(i-1, j) and dp(i, j-1)
Bottom-up:   fill i=1..m outer, j=1..n inner (same direction as dependencies)

Top-down:    dp(i, j) uses dp(i+1, j) and dp(i, j+1)  (Dungeon Game)
Bottom-up:   fill i=m..0 outer, j=n..0 inner (reverse direction)

Key rule: fill in the OPPOSITE order of recursion direction
```

---

---

# 9. Common Pitfalls

## 1. Integer Overflow in Initialization

```java
// WRONG — overflow when doing dp[i] + cost
int[] dp = new int[n];
Arrays.fill(dp, Integer.MAX_VALUE);
dp[i] = Math.min(dp[i], dp[j] + cost);  // MAX_VALUE + cost overflows!

// CORRECT
Arrays.fill(dp, Integer.MAX_VALUE / 2);  // safe buffer
// OR check before adding:
if (dp[j] != Integer.MAX_VALUE)
    dp[i] = Math.min(dp[i], dp[j] + cost);
```

---

## 2. Wrong Loop Order (0/1 vs Unbounded)

```java
// 0/1 Knapsack — each item used ONCE → decreasing
for (int num : nums)
    for (int j = target; j >= num; j--)  // ← decreasing
        dp[j] = dp[j] || dp[j - num];

// Unbounded — item reusable → increasing
for (int coin : coins)
    for (int j = coin; j <= amount; j++)  // ← increasing
        dp[j] += dp[j - coin];
```

---

## 3. Using -1 as Sentinel in Counting DP

```java
// WRONG — -1 means "not computed" but valid answer can also be 0
int[] dp = new int[n];
Arrays.fill(dp, -1);
// dp[i] = 0 means "0 ways" but also looks like "not computed"

// CORRECT — use separate visited array or use Integer[]
Integer[] dp = new Integer[n];
// dp[i] == null means not computed; dp[i] = 0 means valid answer of 0
```

---

## 4. Area vs Side Length

```java
// Maximal Square — problem asks for AREA not side
int max = 0;  // max = largest side found
// ...
return max * max;  // ← area, not max
```

---

## 5. Off-by-One in Base Cases

```java
// Sequence DP — dp is (m+1) x (n+1), strings are 0-indexed
// dp[i][j] = answer for word1[0..i-1] and word2[0..j-1]
// So character at row i is word1.charAt(i-1), not word1.charAt(i)

if (word1.charAt(i-1) == word2.charAt(j-1))  // ← i-1 and j-1
```

---

## 6. Forgetting to Handle Empty Interval in Interval DP

```java
// dp[i][j] where i > j = empty interval = 0 (base case)
// Java int[][] initializes to 0 — this works automatically
// But be careful with MAX_VALUE initialization:
dp[i][j] = Integer.MAX_VALUE;  // do this inside the len>=2 loop only
// Single elements (len=1) should stay at 0
```

---

## 7. Modifying Input Array Without Realizing

```java
// Count Square Submatrices — solution modifies matrix in-place
// If problem says "don't modify input", use separate dp array
matrix[i][j] = Math.min(...) + 1;  // modifies input!
```

---

## 8. Counting Combinations vs Permutations

```java
// Combinations (order doesn't matter): outer=items, inner=capacity
for (int coin : coins)
    for (int j = coin; j <= amount; j++)
        dp[j] += dp[j - coin];

// Permutations (order matters): outer=capacity, inner=items
for (int j = 1; j <= target; j++)
    for (int num : nums)
        if (j >= num) dp[j] += dp[j - num];
```

---

---

# Quick Reference Card

## DP Family Decision Tree

```
Single array, optimize over prefix?
  → 1D Linear DP

Grid input, restricted movement (right/down)?
  → 2D Grid DP (forward fill)
  → Future determines present? → reverse fill

Grid, free movement any direction?
  → Dijkstra (not DP)

Subset selection, "reach exactly X", "partition"?
  → Knapsack DP
  → Item reused? → unbounded (inner loop increasing)
  → Item once?   → 0/1 (inner loop decreasing)

Contiguous subarray, "merge/burst/split", "what to do last"?
  → Interval DP (fill by length)

Two strings, compare characters?
  → Sequence DP (2D table)

Input is a tree?
  → Tree DP (post-order DFS, return values upward)

Small n (≤ 20), visit all / assign all?
  → Bitmask DP
```

## Loop Order Cheat Sheet

```
0/1 Knapsack:      for item → for j=target..0    (decreasing)
Unbounded:         for item → for j=0..target    (increasing)
Combinations:      outer=items, inner=capacity
Permutations:      outer=capacity, inner=items
Interval DP:       outer=length, inner=start, innermost=split
Sequence DP:       outer=i (s1), inner=j (s2)
Bitmask DP:        outer=mask, inner=bit to set/clear
Tree DP:           post-order DFS (children before parent)
```

## Complexity Cheat Sheet

| Family | Time | Space | Optimized Space |
|---|---|---|---|
| 1D Linear | O(n) or O(n²) | O(n) | O(1) or O(k) |
| 2D Grid | O(m×n) | O(m×n) | O(n) — one row |
| Knapsack 0/1 | O(n×W) | O(n×W) | O(W) |
| Knapsack Unbounded | O(n×W) | O(n×W) | O(W) |
| Interval DP | O(n³) | O(n²) | Cannot reduce |
| Sequence DP | O(m×n) | O(m×n) | O(min(m,n)) |
| Tree DP | O(n) | O(h) stack | O(h) |
| Bitmask DP | O(2ⁿ × n) | O(2ⁿ × n) | O(2ⁿ) sometimes |

## Top-Down vs Bottom-Up

```
Top-Down:    sparse states, complex state space, tree DP, bitmask DP
Bottom-Up:   dense states, need space optimization, 1D/2D DP

Converting: fill in OPPOSITE direction of recursion dependencies
```

## Space Optimization

```
1D DP:        already O(n)
Grid DP:      O(n) — keep only previous row
Knapsack:     O(target) — 1D array, mind loop direction
Interval DP:  O(n²) — cannot reduce (need all intervals)
Sequence DP:  O(min(m,n)) — keep only previous row
Tree DP:      O(h) — recursion stack only
Bitmask DP:   O(2ⁿ) — sometimes drop last dimension
```
