# Heap / Priority Queue — Complete Guide for Google L6

---

## What is a Heap?

A heap is a **complete binary tree** where parent ≥ children (max-heap) or parent ≤ children (min-heap).

```
Min-Heap: smallest element always at root → always gives you the MINIMUM
Max-Heap: largest element always at root  → always gives you the MAXIMUM

Java: PriorityQueue<Integer> pq = new PriorityQueue<>();              // min-heap
      PriorityQueue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
```

**Operations**: `offer(x)` — O(log n), `poll()` — O(log n), `peek()` — O(1)

---

## How to Identify Heap Problems

| Signal | Pattern |
|---|---|
| "K largest / K smallest" | maintain size-k heap of opposite type |
| "K closest" | max-heap of size k |
| "median of stream" | two heaps (max-heap left + min-heap right) |
| "next scheduled / earliest" | min-heap by time/cost |
| "merge K sorted lists" | min-heap with (value, listIndex, nodeIndex) |
| "greedy + always pick best next" | min/max heap for greedy decisions |
| "task scheduling with cooldown" | heap + idle slots |

---

## How to Think About Heap Problems

```
1. Do I need the min or max repeatedly?
   → Yes → heap

2. Fixed window of K elements?
   → K smallest → max-heap of size k (eject largest when size > k)
   → K largest  → min-heap of size k (eject smallest when size > k)

3. Two partitions (median, balanced)?
   → Two heaps: max-heap(left) and min-heap(right), balance sizes

4. Multiple sorted sources?
   → Merge with min-heap seeded with first elements

5. Greedy "always pick cheapest/shortest next"?
   → Min-heap ordered by cost/time
```

---

## Key Templates

### Template 1: K Smallest (max-heap of size k)
```java
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
for (int num : nums) {
    maxHeap.offer(num);
    if (maxHeap.size() > k) maxHeap.poll();  // eject largest
}
// maxHeap now contains k smallest; peek() = kth smallest
```

### Template 2: K Largest (min-heap of size k)
```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
for (int num : nums) {
    minHeap.offer(num);
    if (minHeap.size() > k) minHeap.poll();  // eject smallest
}
// minHeap now contains k largest; peek() = kth largest
```

### Template 3: Two Heaps (Median)
```java
PriorityQueue<Integer> lo = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
PriorityQueue<Integer> hi = new PriorityQueue<>(); // min-heap

void addNum(int num) {
    lo.offer(num);
    hi.offer(lo.poll());   // balance: push to hi via lo
    if (lo.size() < hi.size()) lo.offer(hi.poll());  // lo always >= hi in size
}

double findMedian() {
    return lo.size() > hi.size() ? lo.peek() : (lo.peek() + hi.peek()) / 2.0;
}
```

### Template 4: Merge K Sorted (min-heap)
```java
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
// seed with first element of each list: [value, listIndex, elementIndex]
for (int i = 0; i < lists.length; i++) {
    if (lists[i] != null) pq.offer(new int[]{lists[i].val, i, 0});
}
while (!pq.isEmpty()) {
    int[] curr = pq.poll();
    // process curr[0]
    // add next element from same list
}
```

---

---

# Problems

### 1. Kth Largest Element in an Array (LC 215)
Given an integer array `nums` and an integer `k`, return the kth largest element (not the kth distinct element).

```
Input:  nums = [3,2,1,5,6,4], k = 2
Output: 5

Input:  nums = [3,2,3,1,2,4,5,5,6], k = 4
Output: 4
```

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        for (int num : nums) {
            minHeap.offer(num);
            if (minHeap.size() > k) minHeap.poll();
        }
        return minHeap.peek();
    }
}
```
**Why min-heap for k largest?** The heap keeps the k largest elements. When size exceeds k, we eject the smallest — which is the smallest of the k+1 elements. What remains is the k largest. `peek()` gives the kth largest (smallest of the k largest).

Time: O(n log k) | Space: O(k)

---

### 2. K Closest Points to Origin (LC 973)
Given array of `points` on a plane, return the `k` closest points to the origin `(0,0)`. Distance = sqrt(x²+y²) but you can compare x²+y² directly.

```
Input:  points = [[1,3],[-2,2]], k = 1
Output: [[-2,2]]  → dist²: 1²+3²=10, (-2)²+2²=8, closer

Input:  points = [[3,3],[5,-1],[-2,4]], k = 2
Output: [[3,3],[-2,4]]
```

```java
class Solution {
    public int[][] kClosest(int[][] points, int k) {
        // max-heap by distance — eject farthest when > k
        PriorityQueue<int[]> maxHeap = new PriorityQueue<>(
            (a, b) -> (b[0]*b[0] + b[1]*b[1]) - (a[0]*a[0] + a[1]*a[1])
        );
        for (int[] p : points) {
            maxHeap.offer(p);
            if (maxHeap.size() > k) maxHeap.poll();
        }
        return maxHeap.toArray(new int[k][]);
    }
}
```

---

### 3. Find K Pairs with Smallest Sums (LC 373)
Given two sorted arrays, return the `k` pairs `(u, v)` with the smallest sums where `u` is from `nums1` and `v` is from `nums2`.

```
Input:  nums1 = [1,7,11], nums2 = [2,4,6], k = 3
Output: [[1,2],[1,4],[1,6]]  → sums: 3,5,7

Input:  nums1 = [1,1,2], nums2 = [1,2,3], k = 2
Output: [[1,1],[1,1]]
```

```java
class Solution {
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        List<List<Integer>> result = new ArrayList<>();
        if (nums1.length == 0 || nums2.length == 0) return result;

        // [sum, i, j] → min-heap by sum
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        // seed: pair each nums1[i] with nums2[0]
        for (int i = 0; i < Math.min(nums1.length, k); i++)
            pq.offer(new int[]{nums1[i] + nums2[0], i, 0});

        while (!pq.isEmpty() && result.size() < k) {
            int[] curr = pq.poll();
            int i = curr[1], j = curr[2];
            result.add(Arrays.asList(nums1[i], nums2[j]));
            if (j + 1 < nums2.length)
                pq.offer(new int[]{nums1[i] + nums2[j+1], i, j+1});
        }
        return result;
    }
}
```

---

### 4. Merge K Sorted Lists (LC 23)
Given an array of `k` linked lists, each sorted in ascending order, merge all of them into one sorted linked list.

```
Input:  lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]

Input:  lists = []
Output: []
```

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
        for (ListNode node : lists)
            if (node != null) pq.offer(node);

        ListNode dummy = new ListNode(0), curr = dummy;
        while (!pq.isEmpty()) {
            ListNode node = pq.poll();
            curr.next = node;
            curr = curr.next;
            if (node.next != null) pq.offer(node.next);
        }
        return dummy.next;
    }
}
```
**Seed** with first node of each list. After polling, push next node of same list.

Time: O(N log k) where N = total nodes, k = number of lists

---

### 5. Find Median from Data Stream (LC 295)
Design a data structure that supports `addNum(int num)` and `findMedian()` where numbers are added from a data stream.

```
addNum(1) → lo=[1], hi=[]
addNum(2) → lo=[1], hi=[2]
findMedian() → (1+2)/2 = 1.5

addNum(3) → lo=[2], hi=[3], wait... lo=[1,2], hi=[3]
findMedian() → 2.0
```

```java
class MedianFinder {
    PriorityQueue<Integer> lo = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
    PriorityQueue<Integer> hi = new PriorityQueue<>(); // min-heap

    public void addNum(int num) {
        lo.offer(num);
        hi.offer(lo.poll());
        if (lo.size() < hi.size()) lo.offer(hi.poll());
    }

    public double findMedian() {
        return lo.size() > hi.size() ? lo.peek() : (lo.peek() + hi.peek()) / 2.0;
    }
}
```
**Invariant**: `lo` has ≥ `hi` in size; `lo.peek() ≤ hi.peek()` (all of lo ≤ all of hi).

---

### 6. Top K Frequent Elements (LC 347)
Given integer array `nums` and integer `k`, return the `k` most frequently occurring elements.

```
Input:  nums = [1,1,1,2,2,3], k = 2
Output: [1,2]

Input:  nums = [1], k = 1
Output: [1]
```

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> freq = new HashMap<>();
        for (int n : nums) freq.merge(n, 1, Integer::sum);

        // min-heap by frequency
        PriorityQueue<Integer> minHeap = new PriorityQueue<>((a, b) -> freq.get(a) - freq.get(b));
        for (int num : freq.keySet()) {
            minHeap.offer(num);
            if (minHeap.size() > k) minHeap.poll();
        }

        int[] result = new int[k];
        for (int i = k - 1; i >= 0; i--) result[i] = minHeap.poll();
        return result;
    }
}
```

---

### 7. Task Scheduler (LC 621)
Given array of tasks (letters A-Z) and cooldown `n`, return the minimum CPU intervals needed to execute all tasks. Same task must have at least `n` intervals between executions; CPU can be idle.

```
Input:  tasks = ["A","A","A","B","B","B"], n = 2
Output: 8  → A→B→idle→A→B→idle→A→B

Input:  tasks = ["A","A","A","B","B","B"], n = 0
Output: 6  → no cooldown needed
```

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        int[] freq = new int[26];
        for (char t : tasks) freq[t - 'A']++;

        // max-heap by frequency
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        for (int f : freq) if (f > 0) maxHeap.offer(f);

        Queue<int[]> cooldown = new LinkedList<>(); // [remainingCount, availableTime]
        int time = 0;
        while (!maxHeap.isEmpty() || !cooldown.isEmpty()) {
            time++;
            if (!maxHeap.isEmpty()) {
                int count = maxHeap.poll() - 1;
                if (count > 0) cooldown.offer(new int[]{count, time + n});
            }
            if (!cooldown.isEmpty() && cooldown.peek()[1] == time)
                maxHeap.offer(cooldown.poll()[0]);
        }
        return time;
    }
}
```

---

### 8. Reorganize String (LC 767)
Given string `s`, rearrange characters so no two adjacent characters are the same. Return any valid rearrangement, or `""` if impossible.

```
Input:  s = "aab"
Output: "aba"

Input:  s = "aaab"
Output: ""  (impossible — too many 'a's)
```

```java
class Solution {
    public String reorganizeString(String s) {
        int[] freq = new int[26];
        for (char c : s.toCharArray()) freq[c - 'a']++;

        PriorityQueue<int[]> maxHeap = new PriorityQueue<>((a, b) -> b[1] - a[1]);
        for (int i = 0; i < 26; i++)
            if (freq[i] > 0) maxHeap.offer(new int[]{i, freq[i]});

        StringBuilder sb = new StringBuilder();
        while (maxHeap.size() >= 2) {
            int[] first = maxHeap.poll();
            int[] second = maxHeap.poll();
            sb.append((char)('a' + first[0]));
            sb.append((char)('a' + second[0]));
            if (--first[1] > 0) maxHeap.offer(first);
            if (--second[1] > 0) maxHeap.offer(second);
        }
        if (!maxHeap.isEmpty()) {
            if (maxHeap.peek()[1] > 1) return "";
            sb.append((char)('a' + maxHeap.poll()[0]));
        }
        return sb.toString();
    }
}
```

---

### 9. Smallest Range Covering Elements from K Lists (LC 632)
Given `k` sorted lists of integers, find the smallest range `[a,b]` such that at least one number from each list is in the range.

```
Input:  nums = [[4,10,15,24,26],[0,9,12,20],[5,18,22,30]]
Output: [20,24]
        → 20 is in list 2, 24 is in list 1, 22 is in list 3
```

```java
class Solution {
    public int[] smallestRange(List<List<Integer>> nums) {
        // min-heap: [value, row, col]
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        int maxVal = Integer.MIN_VALUE;
        for (int i = 0; i < nums.size(); i++) {
            pq.offer(new int[]{nums.get(i).get(0), i, 0});
            maxVal = Math.max(maxVal, nums.get(i).get(0));
        }

        int[] ans = {pq.peek()[0], maxVal};
        while (true) {
            int[] curr = pq.poll();
            int row = curr[1], col = curr[2];
            if (col + 1 == nums.get(row).size()) break;
            int nextVal = nums.get(row).get(col + 1);
            pq.offer(new int[]{nextVal, row, col + 1});
            maxVal = Math.max(maxVal, nextVal);
            if (maxVal - pq.peek()[0] < ans[1] - ans[0])
                ans = new int[]{pq.peek()[0], maxVal};
        }
        return ans;
    }
}
```

---

### 10. Trapping Rain Water II (LC 407)
Given 2D integer array `heightMap` representing elevation of each cell, return total volume of water that can be trapped after raining (3D generalization of Trapping Rain Water).

```
Input:  heightMap = [[1,4,3,1,3,2],[3,2,1,3,2,4],[2,3,3,2,3,1]]
Output: 4

Input:  heightMap = [[3,3,3,3,3],[3,2,2,2,3],[3,2,1,2,3],[3,2,2,2,3],[3,3,3,3,3]]
Output: 10
```

```java
class Solution {
    public int trapRainWater(int[][] heightMap) {
        int m = heightMap.length, n = heightMap[0].length;
        boolean[][] visited = new boolean[m][n];
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);

        // seed boundary
        for (int i = 0; i < m; i++) {
            pq.offer(new int[]{heightMap[i][0], i, 0}); visited[i][0] = true;
            pq.offer(new int[]{heightMap[i][n-1], i, n-1}); visited[i][n-1] = true;
        }
        for (int j = 1; j < n-1; j++) {
            pq.offer(new int[]{heightMap[0][j], 0, j}); visited[0][j] = true;
            pq.offer(new int[]{heightMap[m-1][j], m-1, j}); visited[m-1][j] = true;
        }

        int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
        int water = 0;
        while (!pq.isEmpty()) {
            int[] curr = pq.poll();
            for (int[] d : dirs) {
                int r = curr[1]+d[0], c = curr[2]+d[1];
                if (r < 0 || r >= m || c < 0 || c >= n || visited[r][c]) continue;
                visited[r][c] = true;
                water += Math.max(0, curr[0] - heightMap[r][c]);
                pq.offer(new int[]{Math.max(curr[0], heightMap[r][c]), r, c});
            }
        }
        return water;
    }
}
```

---

### 11. Minimum Cost to Connect Sticks (LC 1167)
Given array `sticks`, combine any two sticks into one (cost = sum of their lengths). Return minimum total cost to combine all sticks into one.

```
Input:  sticks = [2,4,3]
Output: 14  → combine 2+3=5 (cost 5), then 5+4=9 (cost 9) → total 14

Input:  sticks = [1,8,3,5]
Output: 30  → 1+3=4(4), 4+5=9(9), 9+8=17(17) → 4+9+17=30
```

```java
class Solution {
    public int connectSticks(int[] sticks) {
        PriorityQueue<Integer> pq = new PriorityQueue<>();
        for (int s : sticks) pq.offer(s);
        int cost = 0;
        while (pq.size() > 1) {
            int combined = pq.poll() + pq.poll();
            cost += combined;
            pq.offer(combined);
        }
        return cost;
    }
}
```
**Same pattern as Huffman coding** — always merge two smallest.

---

### 12. Design Twitter (LC 355)
Design a Twitter-like system supporting: `postTweet(userId, tweetId)`, `getNewsFeed(userId)` (return 10 most recent tweet IDs from user + followees), `follow(followerId, followeeId)`, `unfollow(followerId, followeeId)`.

```
twitter.postTweet(1, 5)   // user 1 posts tweet 5
twitter.getNewsFeed(1)    // [5]
twitter.follow(1, 2)      // user 1 follows user 2
twitter.postTweet(2, 6)   // user 2 posts tweet 6
twitter.getNewsFeed(1)    // [6, 5]  (most recent first)
twitter.unfollow(1, 2)
twitter.getNewsFeed(1)    // [5]
```

```java
class Twitter {
    Map<Integer, List<int[]>> tweets = new HashMap<>(); // userId → [(time, tweetId)]
    Map<Integer, Set<Integer>> following = new HashMap<>();
    int time = 0;

    public void postTweet(int userId, int tweetId) {
        tweets.computeIfAbsent(userId, k -> new ArrayList<>()).add(new int[]{time++, tweetId});
    }

    public List<Integer> getNewsFeed(int userId) {
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> b[0] - a[0]); // max by time
        Set<Integer> users = new HashSet<>();
        users.add(userId);
        users.addAll(following.getOrDefault(userId, new HashSet<>()));

        for (int u : users) {
            List<int[]> list = tweets.getOrDefault(u, Collections.emptyList());
            if (!list.isEmpty()) {
                int idx = list.size() - 1;
                pq.offer(new int[]{list.get(idx)[0], list.get(idx)[1], u, idx});
            }
        }

        List<Integer> result = new ArrayList<>();
        while (!pq.isEmpty() && result.size() < 10) {
            int[] curr = pq.poll();
            result.add(curr[1]);
            int u = curr[2], idx = curr[3] - 1;
            if (idx >= 0) {
                List<int[]> list = tweets.get(u);
                pq.offer(new int[]{list.get(idx)[0], list.get(idx)[1], u, idx});
            }
        }
        return result;
    }

    public void follow(int followerId, int followeeId) {
        following.computeIfAbsent(followerId, k -> new HashSet<>()).add(followeeId);
    }

    public void unfollow(int followerId, int followeeId) {
        following.getOrDefault(followerId, new HashSet<>()).remove(followeeId);
    }
}
```

---

### 13. IPO (LC 502)
You can do at most `k` projects. Each project has a `profit` and minimum `capital` required. Starting with `w` capital, maximize your final capital by selecting at most `k` projects optimally.

```
Input:  k=2, w=0, profits=[1,2,3], capital=[0,1,1]
Output: 4
        → do project 0 (capital 0, profit 1) → w=1
        → do project 2 (capital 1, profit 3) → w=4
```

```java
class Solution {
    public int findMaximizedCapital(int k, int w, int[] profits, int[] capital) {
        int n = profits.length;
        int[][] projects = new int[n][2];
        for (int i = 0; i < n; i++) projects[i] = new int[]{capital[i], profits[i]};
        Arrays.sort(projects, (a, b) -> a[0] - b[0]); // sort by capital

        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        int idx = 0;
        for (int i = 0; i < k; i++) {
            while (idx < n && projects[idx][0] <= w) maxHeap.offer(projects[idx++][1]);
            if (maxHeap.isEmpty()) break;
            w += maxHeap.poll();
        }
        return w;
    }
}
```
**Greedy + two heaps**: unlock available projects (sorted by capital), always pick highest profit.

---

### 14. Ugly Number II (LC 264)
An ugly number is a positive integer whose prime factors are limited to 2, 3, and 5. Given integer `n`, return the `nth` ugly number.

```
Input:  n = 10
Output: 12
        Sequence: 1,2,3,4,5,6,8,9,10,12

Input:  n = 1
Output: 1
```

```java
class Solution {
    public int nthUglyNumber(int n) {
        PriorityQueue<Long> pq = new PriorityQueue<>();
        Set<Long> seen = new HashSet<>();
        pq.offer(1L); seen.add(1L);
        long curr = 1;
        for (int i = 0; i < n; i++) {
            curr = pq.poll();
            for (long factor : new long[]{2, 3, 5}) {
                if (seen.add(curr * factor)) pq.offer(curr * factor);
            }
        }
        return (int) curr;
    }
}
```

---

### 15. Meeting Rooms II (LC 253)
Given array of meeting time intervals `[start, end]`, return the minimum number of conference rooms required.

```
Input:  intervals = [[0,30],[5,10],[15,20]]
Output: 2  → room1: [0,30]; room2: [5,10] then [15,20]

Input:  intervals = [[7,10],[2,4]]
Output: 1  → no overlap
```

```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
        PriorityQueue<Integer> endTimes = new PriorityQueue<>(); // min-heap of end times

        for (int[] interval : intervals) {
            if (!endTimes.isEmpty() && endTimes.peek() <= interval[0])
                endTimes.poll(); // reuse room
            endTimes.offer(interval[1]);
        }
        return endTimes.size();
    }
}
```
**Min-heap of end times**: if earliest-ending room is free before next meeting starts, reuse it.

---

---

# Common Pitfalls

## 1. Max-heap vs Min-heap Confusion

```java
// K LARGEST → min-heap (eject smallest, keep k largest)
PriorityQueue<Integer> pq = new PriorityQueue<>();
if (pq.size() > k) pq.poll(); // ejects smallest ✅

// K SMALLEST → max-heap (eject largest, keep k smallest)
PriorityQueue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder());
if (pq.size() > k) pq.poll(); // ejects largest ✅
```

---

## 2. Integer Comparison in Comparator

```java
// WRONG — can overflow for large negative integers
(a, b) -> a - b

// CORRECT — safe comparison
(a, b) -> Integer.compare(a, b)
// or for custom objects:
(a, b) -> a[0] - b[0]  // OK if values stay within int range
```

---

## 3. Two Heaps Invariant

```java
// After addNum, invariant must hold:
// 1. lo.size() >= hi.size() (lo can have one extra)
// 2. lo.peek() <= hi.peek() (all left half <= all right half)

// Always push through lo to ensure invariant 2:
lo.offer(num);
hi.offer(lo.poll());   // biggest of lo goes to hi
if (lo.size() < hi.size()) lo.offer(hi.poll()); // rebalance
```

---

## 4. Modifying Heap While Iterating

```java
// Heap does not support iteration in sorted order
// poll() repeatedly for sorted order O(n log n)
// Or convert to list: new ArrayList<>(pq) — not sorted!
```

---

## 5. Lazy Deletion Pattern

When you can't remove from middle of heap, mark as deleted:
```java
Set<Integer> deleted = new HashSet<>();
// Instead of removing x from heap:
deleted.add(x);
// When polling:
while (!pq.isEmpty() && deleted.contains(pq.peek()))
    pq.poll();
```

---

---

# Quick Reference Card

## Decision Table

| Scenario | Heap Type | Size |
|---|---|---|
| K largest | min-heap | k |
| K smallest | max-heap | k |
| Kth largest | min-heap | k |
| K closest | max-heap by dist | k |
| K most frequent | min-heap by freq | k |
| Median stream | max-heap + min-heap | — |
| Merge K sorted | min-heap | k |
| Always pick max | max-heap | all |
| Always pick min | min-heap | all |

## Java Heap Cheat Sheet

```java
// Min-heap (default)
PriorityQueue<Integer> pq = new PriorityQueue<>();

// Max-heap
PriorityQueue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder());

// Custom comparator (min by first element of array)
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);

// Operations
pq.offer(x);    // add
pq.poll();      // remove and return min/max
pq.peek();      // view min/max without removing
pq.size();
pq.isEmpty();
```

## Complexity

| Operation | Time |
|---|---|
| offer | O(log n) |
| poll | O(log n) |
| peek | O(1) |
| Build from n elements | O(n) |
| K largest from n | O(n log k) |
| Merge k lists, N total | O(N log k) |
