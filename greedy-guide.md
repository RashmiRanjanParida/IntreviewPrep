# Greedy Algorithms — Complete Guide for Google L6

---

## What is Greedy?

At each step, make the **locally optimal choice** hoping it leads to the global optimum.

```
Greedy works when:
  Local optimum = Global optimum
  (provable by exchange argument or matroid theory)

Greedy fails when:
  Future choices depend on current in non-obvious ways
  → Use DP instead
```

---

## How to Identify Greedy

| Signal | Pattern |
|---|---|
| "minimum/maximum result" + no dependency chain | Often greedy |
| "always pick the earliest/smallest/largest" | Greedy |
| "interval scheduling — minimum rooms/overlaps" | Interval greedy |
| "can we reach the end?" | Greedy scan |
| "assign tasks optimally" | Sort + greedy |
| Solved with DP but O(n) exists | Greedy |

**Test for greedy**: Can I prove that choosing the "best" option now never makes things worse later? If yes → greedy.

---

## How to Think About Greedy Problems

```
1. What is the "greedy choice" at each step?
   → Earliest deadline, smallest gap, largest coverage

2. Why is this safe?
   → Exchange argument: swapping any other choice makes things ≤ not better

3. What order should I process elements?
   → Sort first! Almost all greedy problems start with a sort.

4. What do I track as I scan?
   → Depends: current end, current sum, rooms in use, etc.
```

---

---

# Interval Problems

### How to Think About Intervals

```
1. Sort by start? end? We choose based on what we want to minimize/maximize.

Sort by END → greedy works for minimum rooms, maximum non-overlapping
Sort by START → greedy works for merging, covering

Intervals overlap when: a.start < b.end (open interval)
                    or: a.start <= b.end (closed interval) — check problem statement
```

---

### 1. Meeting Rooms I (LC 252)
Given array of meeting time intervals `[start, end]`, determine if a person can attend all meetings (no overlap).

```
Input:  intervals = [[0,30],[5,10],[15,20]]
Output: false  → [0,30] overlaps with [5,10]

Input:  intervals = [[7,10],[2,4]]
Output: true   → no overlap
```

```java
class Solution {
    public boolean canAttendMeetings(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
        for (int i = 1; i < intervals.length; i++)
            if (intervals[i][0] < intervals[i-1][1]) return false;
        return true;
    }
}
```

---

### 2. Meeting Rooms II (LC 253)
Given array of meeting intervals, return the minimum number of conference rooms required.

```
Input:  intervals = [[0,30],[5,10],[15,20]]
Output: 2  → room1: [0,30]; room2: [5,10] then [15,20]

Input:  intervals = [[2,7],[2,6],[4,8],[3,5]]
Output: 4  → all four overlap at time 4
```

```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
        PriorityQueue<Integer> endTimes = new PriorityQueue<>(); // earliest end

        for (int[] iv : intervals) {
            if (!endTimes.isEmpty() && endTimes.peek() <= iv[0])
                endTimes.poll(); // reuse this room
            endTimes.offer(iv[1]);
        }
        return endTimes.size();
    }
}
```

**Alternative — two sorted arrays**:
```java
int[] starts = ..., ends = ...;
Arrays.sort(starts); Arrays.sort(ends);
int rooms = 0, endPtr = 0;
for (int s : starts) {
    if (s >= ends[endPtr]) endPtr++;
    else rooms++;
}
return rooms;
```

---

### 3. Non-Overlapping Intervals (LC 435)
Given array of intervals, return the minimum number of intervals to remove so the rest do not overlap.

```
Input:  intervals = [[1,2],[2,3],[3,4],[1,3]]
Output: 1  → remove [1,3]

Input:  intervals = [[1,2],[1,2],[1,2]]
Output: 2  → remove two of the three

Input:  intervals = [[1,2],[2,3]]
Output: 0  → no overlap (touching endpoints don't count)
```

```java
class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[1] - b[1]); // sort by END
        int count = 0, prevEnd = Integer.MIN_VALUE;
        for (int[] iv : intervals) {
            if (iv[0] >= prevEnd) {
                prevEnd = iv[1];   // keep this interval
            } else {
                count++;           // remove — overlap detected, keep earlier end
            }
        }
        return count;
    }
}
```

**Key**: Sort by end → always keep the interval that ends earliest (maximizes future space).

---

### 4. Interval Scheduling Maximization
Given intervals, find the maximum number of non-overlapping intervals you can select. (Classic activity selection — inverse of LC 435.)

```
Input:  intervals = [[1,4],[2,3],[3,5],[6,8]]
Output: 3  → select [2,3],[3,5],[6,8]

Input:  intervals = [[1,10],[2,3],[4,5],[6,7]]
Output: 3  → select [2,3],[4,5],[6,7], skip [1,10]
```

```java
public int maxNonOverlapping(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]); // sort by end
    int count = 0, prevEnd = Integer.MIN_VALUE;
    for (int[] iv : intervals) {
        if (iv[0] >= prevEnd) {
            count++;
            prevEnd = iv[1];
        }
    }
    return count;
}
```

---

### 5. Merge Intervals (LC 56)
Given array of intervals, merge all overlapping intervals and return the result.

```
Input:  intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]  → [1,3] and [2,6] merge to [1,6]

Input:  intervals = [[1,4],[4,5]]
Output: [[1,5]]  → touching endpoints merge
```

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start
        List<int[]> result = new ArrayList<>();
        int[] curr = intervals[0];

        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] <= curr[1]) {
                curr[1] = Math.max(curr[1], intervals[i][1]); // merge
            } else {
                result.add(curr);
                curr = intervals[i];
            }
        }
        result.add(curr);
        return result.toArray(new int[0][]);
    }
}
```

---

### 6. Insert Interval (LC 57)
Given sorted non-overlapping intervals and a new interval, insert it and merge if necessary. Return the result.

```
Input:  intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]

Input:  intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
Output: [[1,2],[3,10],[12,16]]  → merges [3,5],[6,7],[8,10]
```

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> result = new ArrayList<>();
        int i = 0, n = intervals.length;

        // add all non-overlapping before
        while (i < n && intervals[i][1] < newInterval[0])
            result.add(intervals[i++]);

        // merge overlapping
        while (i < n && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        result.add(newInterval);

        // add remaining
        while (i < n) result.add(intervals[i++]);
        return result.toArray(new int[0][]);
    }
}
```

---

---

# Jump Game Family

### 7. Jump Game (LC 55)
Given integer array `nums` where `nums[i]` is the max jump length at position `i`, return `true` if you can reach the last index starting from index 0.

```
Input:  nums = [2,3,1,1,4]
Output: true  → jump 1 step to index 1, then 3 steps to last

Input:  nums = [3,2,1,0,4]
Output: false  → always reach index 3 with value 0, stuck
```

```java
class Solution {
    public boolean canJump(int[] nums) {
        int maxReach = 0;
        for (int i = 0; i <= maxReach && i < nums.length; i++)
            maxReach = Math.max(maxReach, i + nums[i]);
        return maxReach >= nums.length - 1;
    }
}
```

**Key**: Track maximum reachable index. If you can't step forward (i > maxReach), stuck.

---

### 8. Jump Game II (LC 45)
Given integer array `nums` where `nums[i]` is the max jump length, return the minimum number of jumps to reach the last index (guaranteed reachable).

```
Input:  nums = [2,3,1,1,4]
Output: 2  → index 0→1→4 (jump 1, then jump 3)

Input:  nums = [2,3,0,1,4]
Output: 2  → index 0→1→4
```

```java
class Solution {
    public int jump(int[] nums) {
        int jumps = 0, currEnd = 0, farthest = 0;
        for (int i = 0; i < nums.length - 1; i++) {
            farthest = Math.max(farthest, i + nums[i]);
            if (i == currEnd) {  // must take a jump here
                jumps++;
                currEnd = farthest;
            }
        }
        return jumps;
    }
}
```

**Key**: At each "boundary" (end of current jump range), take one jump to reach farthest.

---

---

# Gas Station / Circuit

### 9. Gas Station (LC 134)
There are `n` gas stations in a circle. `gas[i]` = gas at station `i`, `cost[i]` = gas to travel from `i` to next. Return starting station index to complete the circuit, or `-1` if impossible.

```
Input:  gas = [1,2,3,4,5], cost = [3,4,5,1,2]
Output: 3  → start at station 3, net gas: 3, 4, -3 → complete circuit

Input:  gas = [2,3,4], cost = [3,4,3]
Output: -1  → total gas (9) < total cost (10)
```

```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int totalGas = 0, tank = 0, start = 0;
        for (int i = 0; i < gas.length; i++) {
            totalGas += gas[i] - cost[i];
            tank += gas[i] - cost[i];
            if (tank < 0) {
                start = i + 1;  // can't start at or before i
                tank = 0;
            }
        }
        return totalGas >= 0 ? start : -1;
    }
}
```

**Key**: If total gas ≥ total cost → solution exists and it's unique. Greedy finds it.

---

---

# Greedy + Sorting

### 10. Assign Cookies (LC 455)
Each child `i` has greed factor `g[i]`. Each cookie `j` has size `s[j]`. A child is content if `s[j] >= g[i]`. Maximize number of content children.

```
Input:  g = [1,2,3], s = [1,1]
Output: 1  → only one cookie satisfies child with g=1

Input:  g = [1,2], s = [1,2,3]
Output: 2  → both children satisfied
```

```java
class Solution {
    public int findContentChildren(int[] g, int[] s) {
        Arrays.sort(g); Arrays.sort(s);
        int i = 0, j = 0;
        while (i < g.length && j < s.length) {
            if (s[j] >= g[i]) i++; // child i satisfied
            j++;
        }
        return i;
    }
}
```

---

### 11. Partition Labels (LC 763)
Partition string `s` into as many parts as possible so that each letter appears in at most one part. Return a list of the sizes of these parts.

```
Input:  s = "ababcbacadefegdehijhklij"
Output: [9,7,8]
        "ababcbaca" | "defegde" | "hijhklij"
        'a' only in part1, 'd' only in part2, etc.

Input:  s = "eccbbbbdec"
Output: [10]  → 'e' appears at start and end, forces whole string
```

```java
class Solution {
    public List<Integer> partitionLabels(String s) {
        int[] last = new int[26];
        for (int i = 0; i < s.length(); i++)
            last[s.charAt(i) - 'a'] = i;

        List<Integer> result = new ArrayList<>();
        int start = 0, end = 0;
        for (int i = 0; i < s.length(); i++) {
            end = Math.max(end, last[s.charAt(i) - 'a']); // extend end to cover this char
            if (i == end) {
                result.add(end - start + 1);
                start = end + 1;
            }
        }
        return result;
    }
}
```

---

### 12. Task Scheduler (LC 621)
Same as heap problem 7 — greedy solution using math formula.

```
Input:  tasks = ["A","A","A","B","B","B"], n = 2
Output: 8  → A→B→idle→A→B→idle→A→B
Formula: max(tasks.length, (maxFreq-1)*(n+1) + countOfMaxFreq)
         = max(6, (3-1)*(2+1)+2) = max(6,8) = 8
```

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        int[] freq = new int[26];
        for (char t : tasks) freq[t - 'A']++;
        Arrays.sort(freq);
        int maxFreq = freq[25];
        int idleSlots = (maxFreq - 1) * n;

        for (int i = 24; i >= 0 && freq[i] > 0; i--)
            idleSlots -= Math.min(freq[i], maxFreq - 1);

        return tasks.length + Math.max(0, idleSlots);
    }
}
```

**Formula**: (maxFreq-1)*n idle slots minimum; reduce by other tasks that fit in gaps.

---

### 13. Candy (LC 135)
`n` children stand in a row with ratings. Give each child at least 1 candy. Children with higher rating than adjacent neighbor must get more candy. Return minimum total candies.

```
Input:  ratings = [1,0,2]
Output: 5  → [2,1,2]

Input:  ratings = [1,2,2]
Output: 4  → [1,2,1] (equal ratings don't need more candy)
```

```java
class Solution {
    public int candy(int[] ratings) {
        int n = ratings.length;
        int[] candy = new int[n];
        Arrays.fill(candy, 1);

        // left to right: if rating[i] > rating[i-1], get one more
        for (int i = 1; i < n; i++)
            if (ratings[i] > ratings[i-1]) candy[i] = candy[i-1] + 1;

        // right to left: if rating[i] > rating[i+1], ensure candy[i] > candy[i+1]
        for (int i = n - 2; i >= 0; i--)
            if (ratings[i] > ratings[i+1]) candy[i] = Math.max(candy[i], candy[i+1] + 1);

        int sum = 0;
        for (int c : candy) sum += c;
        return sum;
    }
}
```

---

### 14. Best Time to Buy and Sell Stock II (LC 122)
Given `prices[i]` = stock price on day `i`, return the maximum profit. You may buy and sell multiple times but can only hold one share at a time.

```
Input:  prices = [7,1,5,3,6,4]
Output: 7  → buy day1(1), sell day2(5)=+4; buy day3(3), sell day4(6)=+3

Input:  prices = [1,2,3,4,5]
Output: 4  → buy day0, sell day4 (or capture each daily gain: 1+1+1+1=4)
```

```java
class Solution {
    public int maxProfit(int[] prices) {
        int profit = 0;
        for (int i = 1; i < prices.length; i++)
            if (prices[i] > prices[i-1])
                profit += prices[i] - prices[i-1]; // capture every upward move
        return profit;
    }
}
```

**Greedy insight**: summing all positive day-to-day differences = same as optimal buy-sell.

---

### 15. Minimum Number of Arrows to Burst Balloons (LC 452)
Balloons are represented as `[xstart, xend]` on a flat wall. An arrow shot at x bursts all balloons where `xstart <= x <= xend`. Return minimum arrows to burst all balloons.

```
Input:  points = [[10,16],[2,8],[1,6],[7,12]]
Output: 2  → shoot at x=6 (bursts [2,8],[1,6]), shoot at x=11 (bursts [10,16],[7,12])

Input:  points = [[1,2],[3,4],[5,6],[7,8]]
Output: 4  → no overlaps, need one arrow each
```

```java
class Solution {
    public int findMinArrowShots(int[][] points) {
        Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1])); // sort by end
        int arrows = 1, end = points[0][1];
        for (int i = 1; i < points.length; i++) {
            if (points[i][0] > end) { // balloon starts after current arrow
                arrows++;
                end = points[i][1];
            }
        }
        return arrows;
    }
}
```

**Same pattern as Non-Overlapping Intervals** — sort by end, greedily place arrow at earliest end.

---

### 16. Minimum Platforms at Station
Given arrival and departure times of trains, find the minimum number of platforms needed so no train waits.

```
Input:  arrival   = [900,940,950,1100,1500,1800]
        departure = [910,1200,1120,1130,1900,2000]
Output: 3  → at time 950, trains from 900,940,950 are all present

Input:  arrival   = [900,1100,1235]
        departure = [1000,1200,1240]
Output: 1  → no overlaps
```

```java
public int minPlatforms(int[] arrival, int[] departure) {
    Arrays.sort(arrival);
    Arrays.sort(departure);
    int platforms = 0, max = 0;
    int i = 0, j = 0;
    while (i < arrival.length) {
        if (arrival[i] <= departure[j]) { platforms++; i++; }
        else { platforms--; j++; }
        max = Math.max(max, platforms);
    }
    return max;
}
```

---

### 17. Car Pooling (LC 1094)
A car with capacity `capacity` picks up/drops off passengers. `trips[i] = [numPassengers, from, to]`. Return `true` if all passengers can be transported without exceeding capacity at any point.

```
Input:  trips = [[2,1,5],[3,3,7]], capacity = 4
Output: false  → at stop 3-5: 2+3=5 passengers > 4

Input:  trips = [[2,1,5],[3,3,7]], capacity = 5
Output: true

Input:  trips = [[3,2,7],[3,7,9],[8,3,9]], capacity = 11
Output: true
```

```java
class Solution {
    public boolean carPooling(int[][] trips, int capacity) {
        int[] stops = new int[1001];
        for (int[] t : trips) {
            stops[t[1]] += t[0];  // pick up
            stops[t[2]] -= t[0];  // drop off
        }
        int inCar = 0;
        for (int s : stops) {
            inCar += s;
            if (inCar > capacity) return false;
        }
        return true;
    }
}
```

**Difference array** trick — mark pick up/drop off events at positions.

---

### 18. Wiggle Subsequence (LC 376)
A wiggle sequence alternates between increases and decreases. Return the length of the longest wiggle subsequence of `nums`.

```
Input:  nums = [1,7,4,9,2,5]
Output: 6  → entire array is a wiggle sequence

Input:  nums = [1,17,5,10,13,15,10,5,16,8]
Output: 7  → [1,17,10,13,10,16,8]

Input:  nums = [1,2,3,4,5,6,7,8,9]
Output: 2  → only [1,9] or any two adjacent elements
```

```java
class Solution {
    public int wiggleMaxLength(int[] nums) {
        int up = 1, down = 1;
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] > nums[i-1]) up = down + 1;
            else if (nums[i] < nums[i-1]) down = up + 1;
        }
        return Math.max(up, down);
    }
}
```

---

### 19. Shortest Superstring (Greedy approximation — NP-hard exact)
Find order of strings minimizing total length with maximum overlap.

*Exact: Bitmask DP. Greedy approximation: always merge most-overlapping pair.*

---

### 20. Minimum Cost to Hire K Workers (LC 857)
There are `n` workers, each with a quality and minimum wage expectation. You must hire exactly `k` workers and form a paid group where every worker is paid in proportion to their quality (same wage/quality ratio for all). Return minimum total cost.

```
Input:  quality=[10,20,5], wage=[70,50,30], k=2
Output: 105.0  → hire workers 0 and 2
        ratio for worker0: 70/10=7, ratio for worker2: 30/5=6
        use ratio=7 (max in group): pay worker0=70, worker2=35 → total=105

Input:  quality=[3,1,10,10,1], wage=[4,8,2,2,7], k=3
Output: 30.666...
```

```java
class Solution {
    public double mincostToHireWorkers(int[] quality, int[] wage, int k) {
        int n = quality.length;
        double[][] workers = new double[n][2];
        for (int i = 0; i < n; i++)
            workers[i] = new double[]{(double) wage[i] / quality[i], quality[i]};

        Arrays.sort(workers, (a, b) -> Double.compare(a[0], b[0]));

        PriorityQueue<Double> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        double sumQ = 0, ans = Double.MAX_VALUE;

        for (double[] w : workers) {
            double ratio = w[0], q = w[1];
            maxHeap.offer(q);
            sumQ += q;
            if (maxHeap.size() > k) sumQ -= maxHeap.poll();
            if (maxHeap.size() == k) ans = Math.min(ans, ratio * sumQ);
        }
        return ans;
    }
}
```

**Key insight**: Fix the captain (highest ratio worker), then hire k-1 with minimum quality.

---

---

# Common Pitfalls

## 1. Wrong Sort Key

```java
// Non-overlapping intervals → sort by END (not start!)
Arrays.sort(intervals, (a, b) -> a[1] - b[1]);

// Merge intervals → sort by START
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

// Meeting rooms → sort by START (process in arrival order)
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
```

---

## 2. Overlap Condition

```java
// Open intervals [a, b) and [c, d): overlap if c < b
if (intervals[i][0] < prevEnd)  // open

// Closed intervals [a, b] and [c, d]: overlap if c <= b
if (intervals[i][0] <= prevEnd)  // closed

// Check problem statement: are endpoints inclusive?
```

---

## 3. Gas Station — Off by One

```java
// When tank < 0, next valid start is i+1 (not i)
if (tank < 0) { start = i + 1; tank = 0; }
```

---

## 4. Jump Game — Fence Post

```java
// Don't check last index — we need jumps TO it, not from it
for (int i = 0; i < nums.length - 1; i++)  // stop before last
```

---

---

# Quick Reference Card

## Sort Key by Problem Type

```
Merge intervals       → sort by start
Non-overlapping       → sort by end
Minimum rooms         → sort by start, min-heap of ends
Minimum arrows        → sort by end
Earliest deadline     → sort by end (EDF scheduling)
Shortest job first    → sort by duration
```

## Problem → Pattern Map

| Problem | Sort | Track |
|---|---|---|
| Non-overlapping intervals | by end | prevEnd |
| Meeting rooms II | by start | min-heap of ends |
| Merge intervals | by start | curr interval |
| Jump Game | none | maxReach |
| Jump Game II | none | currEnd, farthest |
| Gas Station | none | tank, start |
| Partition Labels | none | last occurrence |
| Candy | none | two passes |

## Complexity

| Pattern | Time | Space |
|---|---|---|
| Sort + linear scan | O(n log n) | O(1) |
| Sort + heap | O(n log n) | O(n) |
| No sort | O(n) | O(1) |
