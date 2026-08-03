# Binary Search on Answer — Complete Guide for Google L6

---

## What is Binary Search on Answer?

Normal binary search: search for a **value in a sorted array**.
Binary Search on Answer: search for the **optimal value of the answer itself**.

```
Instead of: "is target at index mid?"
Ask:        "is answer = mid feasible?"

If feasible → try smaller/larger (depending on min/max)
If not      → eliminate half the search space
```

---

## How to Identify

| Signal | Example |
|---|---|
| "minimize the maximum" | Split Array Largest Sum |
| "maximize the minimum" | Magnetic Force Between Balls |
| "find minimum X such that..." | Capacity to Ship |
| Answer has a **monotonic property** | if X works, X+1 also works |
| Brute force O(n²), want O(n log n) | Most problems here |

---

## The Monotonic Property (Key Insight)

```
If eating speed k=10 finishes bananas in time → k=11 also finishes ✅
If eating speed k=5 doesn't finish in time   → k=4 also won't    ❌

Monotonic YES/NO boundary → binary search on k:
❌ ❌ ❌ ✅ ✅ ✅ ✅
         ↑
    find this boundary
```

---

## The 3 Questions (Ask Before Coding)

```
1. What is the search space?   → lo and hi
2. What does feasible(mid) check?
3. Minimize or maximize?       → which template to use
```

---

## 2 Core Templates

### Template 1: Minimize X

```java
int lo = minPossibleAnswer;
int hi = maxPossibleAnswer;

while (lo < hi) {
    int mid = lo + (hi - lo) / 2;   // lower mid
    if (feasible(mid)) hi = mid;     // mid works, try smaller
    else lo = mid + 1;               // mid too small, go right
}
return lo;  // lo == hi == answer
```

### Template 2: Maximize X

```java
int lo = minPossibleAnswer;
int hi = maxPossibleAnswer;

while (lo < hi) {
    int mid = lo + (hi - lo + 1) / 2;  // upper mid — avoid infinite loop
    if (feasible(mid)) lo = mid;         // mid works, try larger
    else hi = mid - 1;                   // mid too large, go left
}
return lo;  // lo == hi == answer
```

---

## Why Upper Mid in Template 2?

```
Maximize: feasible → lo = mid

lo=2, hi=3, lower mid → mid=2
  feasible → lo=mid=2 → lo and hi unchanged → infinite loop! ❌

lo=2, hi=3, upper mid → mid=3
  feasible → lo=3 → lo==hi → exits ✅

Rule: when lo=mid in update, use upper mid (hi-lo+1)/2
      when hi=mid in update, use lower mid (hi-lo)/2
```

---

## The 5 Feasibility Variants

| Variant | Feasible Check | Example |
|---|---|---|
| **Split into groups** | greedy pack, count groups ≤ k | Koko, Capacity, Split Array |
| **Count consecutive** | scan for consecutive valid elements | Bouquets |
| **Division sum** | sum of ceil(n/d) ≤ threshold | Smallest Divisor |
| **Place with gap** | greedily place, check min gap | Magnetic Force |
| **Counting** | count how many satisfy condition | Maximum Candies |

---

## Complete Problem List

### Minimize
| Problem | lo | hi |
|---|---|---|
| Koko Eating Bananas | `1` | `max(piles)` |
| Capacity to Ship | `max(weights)` | `sum(weights)` |
| Split Array Largest Sum | `max(nums)` | `sum(nums)` |
| Min Days for Bouquets | `1` | `max(bloomDay)` |
| Smallest Divisor | `1` | `max(nums)` |
| Minimum Time to Repair Cars | `1` | `max(ranks)*n*n` |

### Maximize
| Problem | lo | hi |
|---|---|---|
| Magnetic Force Between Balls | `1` | `max-min` |
| Maximum Candies per Child | `1` | `max(candies)` |
| Minimum Limit of Balls in Bag | `1` | `max(nums)` |

---

---

# Problems

## Minimize Template Problems

### 1. Koko Eating Bananas (LC 875)
**Description:** Koko has n piles of bananas and h hours before guards return. Each hour she eats at most k bananas from one pile. Find the minimum integer speed k so she can finish all piles within h hours.

**Example:**
- Input: piles = [3,6,7,11], h = 8
- Output: 4  (speed 4: ceil(3/4)+ceil(6/4)+ceil(7/4)+ceil(11/4) = 1+2+2+3 = 8 hours ≤ 8)

```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int lo = 1, hi = 0;
        for (int p : piles) hi = Math.max(hi, p);

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (isFeasible(piles, mid, h)) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }

    boolean isFeasible(int[] piles, int k, int h) {
        int hours = 0;
        for (int p : piles)
            hours += (int) Math.ceil((double) p / k);
        return hours <= h;
    }
}
```

**Search space**: `lo=1` (slowest), `hi=max(piles)` (fastest — each pile in 1 hour).
**Feasible**: total hours at speed k ≤ h.

---

### 2. Capacity to Ship Packages (LC 1011)
**Description:** Packages with given weights must be shipped in order within `days` days. The ship loads packages sequentially each day until adding the next would exceed capacity. Find the minimum ship capacity to accomplish this.

**Example:**
- Input: weights = [1,2,3,4,5,6,7,8,9,10], days = 5
- Output: 15  (days: [1-5]=15, [6-7]=13, [8]=8, [9]=9, [10]=10)

```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        int lo = 0, hi = 0;
        for (int w : weights) {
            lo = Math.max(lo, w);  // must fit heaviest package
            hi += w;               // ship all in one day
        }
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (isFeasible(weights, mid, days)) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }

    boolean isFeasible(int[] weights, int capacity, int days) {
        int daysNeeded = 1, load = 0;
        for (int w : weights) {
            if (load + w > capacity) { daysNeeded++; load = 0; }
            load += w;
        }
        return daysNeeded <= days;
    }
}
```

**Search space**: `lo=max(weights)` (must fit any package), `hi=sum(weights)` (one day).
**Feasible**: greedily load, days needed ≤ days.

---

### 3. Split Array Largest Sum (LC 410)
**Description:** Split array nums into exactly m non-empty contiguous subarrays to minimize the largest subarray sum. Return that minimum possible largest sum.

**Example:**
- Input: nums = [7,2,5,10,8], m = 2
- Output: 18  (split [7,2,5] and [10,8] → largest = max(14,18) = 18)

```java
class Solution {
    public int splitArray(int[] nums, int m) {
        int lo = 0, hi = 0;
        for (int n : nums) {
            lo = Math.max(lo, n);
            hi += n;
        }
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (isFeasible(nums, mid, m)) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }

    boolean isFeasible(int[] nums, int maxSum, int m) {
        int parts = 1, currSum = 0;
        for (int n : nums) {
            if (currSum + n > maxSum) { parts++; currSum = 0; }
            currSum += n;
        }
        return parts <= m;
    }
}
```

**Key insight**: identical to Capacity to Ship — split into m parts = ship in m days.

---

### 4. Minimum Days to Make Bouquets (LC 1482)
**Description:** bloomDay[i] is the day flower i blooms. You need m bouquets, each made from k adjacent bloomed flowers. Return the minimum day number to make all m bouquets, or -1 if impossible.

**Example:**
- Input: bloomDay = [1,10,3,10,2], m = 3, k = 1
- Output: 3  (by day 3: flowers at indices 0,2,4 have bloomed → 3 bouquets of 1)

```java
class Solution {
    public int minDays(int[] bloomDay, int m, int k) {
        if ((long) m * k > bloomDay.length) return -1;
        int lo = 1, hi = 0;
        for (int d : bloomDay) hi = Math.max(hi, d);

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (isFeasible(bloomDay, mid, m, k)) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }

    boolean isFeasible(int[] bloomDay, int day, int m, int k) {
        int bouquets = 0, consecutive = 0;
        for (int d : bloomDay) {
            if (d <= day) {
                consecutive++;
                if (consecutive == k) { bouquets++; consecutive = 0; }
            } else {
                consecutive = 0;
            }
        }
        return bouquets >= m;
    }
}
```

**Feasible**: count consecutive bloomed flowers, every k → 1 bouquet.

---

### 5. Find the Smallest Divisor (LC 1283)
**Description:** Find the smallest positive integer divisor such that the sum of ceil(nums[i]/divisor) for all i does not exceed threshold.

**Example:**
- Input: nums = [1,2,5,9], threshold = 6
- Output: 5  (divisor=5: 1+1+1+2=5 ≤ 6; divisor=4: 1+1+2+3=7 > 6)

```java
class Solution {
    public int smallestDivisor(int[] nums, int threshold) {
        int lo = 1, hi = 0;
        for (int n : nums) hi = Math.max(hi, n);

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (isFeasible(nums, mid, threshold)) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }

    boolean isFeasible(int[] nums, int d, int threshold) {
        int sum = 0;
        for (int n : nums)
            sum += (n + d - 1) / d;  // ceil(n/d)
        return sum <= threshold;
    }
}
```

---

### 6. Minimum Speed to Arrive on Time (LC 1870)
**Description:** Take n trains in sequence, each covering dist[i] km. All trains except the last depart at integer hour marks (wait for next whole hour if you arrive early). Find the minimum integer speed to arrive within `hour` hours, or -1 if impossible.

**Example:**
- Input: dist = [1,3,2], hour = 6
- Output: 1  (speed=1: 1hr + 3hr + 2hr = 6hrs ≤ 6)

```java
class Solution {
    public int minSpeedOnTime(int[] dist, double hour) {
        if (dist.length > Math.ceil(hour)) return -1;
        int lo = 1, hi = (int) 1e7;

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (isFeasible(dist, mid, hour)) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }

    boolean isFeasible(int[] dist, int speed, double hour) {
        double time = 0;
        for (int i = 0; i < dist.length - 1; i++)
            time += Math.ceil((double) dist[i] / speed);
        time += (double) dist[dist.length - 1] / speed;  // last train no ceil
        return time <= hour;
    }
}
```

---

### 7. Minimum Number of Days to Eat N Oranges (LC 1553)
**Description:** You have n oranges. Each day you can eat 1 orange, OR eat n/2 if n is divisible by 2, OR eat 2*n/3 if n is divisible by 3. Find minimum days to eat all n oranges.

**Example:**
- Input: n = 10
- Output: 4  (10 → 5 (eat n/2) → 4 (eat 1) → 2 (eat n/2) → 0 (eat n/2))

```java
class Solution {
    Map<Integer, Integer> memo = new HashMap<>();

    public int minDays(int n) {
        if (n <= 1) return n;
        if (memo.containsKey(n)) return memo.get(n);
        // eat remainder then halve, or eat remainder then take 2/3
        int res = 1 + Math.min(n % 2 + minDays(n / 2),
                               n % 3 + minDays(n / 3));
        memo.put(n, res);
        return res;
    }
}
```

---

### 8. Minimize Max Distance to Gas Station (LC 774)
**Description:** On a highway with existing sorted gas stations, add exactly k new stations to minimize the maximum gap between any two adjacent stations. Return the answer with 1e-6 precision.

**Example:**
- Input: stations = [1,2,3,4,5,6,7,8,9,10], k = 9
- Output: 0.500000  (add stations at 1.5,2.5,3.5,...,9.5)

```java
class Solution {
    public double minmaxGasDist(int[] stations, int k) {
        double lo = 0, hi = stations[stations.length - 1] - stations[0];

        while (hi - lo > 1e-6) {
            double mid = (lo + hi) / 2;
            if (isFeasible(stations, k, mid)) hi = mid;
            else lo = mid;
        }
        return lo;
    }

    boolean isFeasible(int[] stations, int k, double dist) {
        int count = 0;
        for (int i = 1; i < stations.length; i++)
            count += (int)((stations[i] - stations[i-1]) / dist);
        return count <= k;
    }
}
```

---

## Maximize Template Problems

### 9. Magnetic Force Between Balls (LC 1552)
**Description:** Place m balls into n basket positions. Maximize the minimum magnetic force (distance) between any two balls. position[] is unsorted.

**Example:**
- Input: position = [1,2,3,4,7], m = 3
- Output: 3  (place balls at positions 1, 4, 7 → minimum distance = 3)

```java
class Solution {
    public int maxDistance(int[] position, int m) {
        Arrays.sort(position);
        int lo = 1, hi = position[position.length - 1] - position[0];

        while (lo < hi) {
            int mid = lo + (hi - lo + 1) / 2;  // upper mid for maximize
            if (isFeasible(position, m, mid)) lo = mid;
            else hi = mid - 1;
        }
        return lo;
    }

    boolean isFeasible(int[] pos, int m, int minDist) {
        int count = 1, last = pos[0];
        for (int i = 1; i < pos.length; i++) {
            if (pos[i] - last >= minDist) { count++; last = pos[i]; }
        }
        return count >= m;
    }
}
```

**Search space**: `lo=1` (min gap), `hi=max-min` (all balls at ends).
**Feasible**: greedily place balls with min gap, count ≥ m.

---

### 10. Maximum Candies Allocated to K Children (LC 2226)
**Description:** You have n piles of candies. You can split piles but not merge them. Allocate each of k children equal-sized portions. Maximize the size of each child's portion.

**Example:**
- Input: candies = [5,8,6], k = 3
- Output: 5  (split [8]→[5,3] and [6]→[5,1]; give each child one portion of 5)

```java
class Solution {
    public int maximumCandies(int[] candies, long k) {
        int lo = 1, hi = 0;
        for (int c : candies) hi = Math.max(hi, c);

        while (lo < hi) {
            int mid = lo + (hi - lo + 1) / 2;  // upper mid
            if (isFeasible(candies, k, mid)) lo = mid;
            else hi = mid - 1;
        }
        return isFeasible(candies, k, lo) ? lo : 0;
    }

    boolean isFeasible(int[] candies, long k, int size) {
        long count = 0;
        for (int c : candies) count += c / size;
        return count >= k;
    }
}
```

---

---

# Common Pitfalls

## 1. Wrong Mid for Maximize Template

```java
// WRONG — causes infinite loop when lo=mid update doesn't move lo
int mid = lo + (hi - lo) / 2;     // lower mid
if (feasible) lo = mid;            // lo never moves when lo+1==hi

// CORRECT — upper mid breaks the tie
int mid = lo + (hi - lo + 1) / 2; // upper mid
if (feasible) lo = mid;            // lo always moves
```

---

## 2. Wrong lo for Capacity-style Problems

```java
// WRONG — capacity of 1 can't ship heavy packages
int lo = 1;

// CORRECT — must fit heaviest single item
int lo = max(weights);
```

---

## 3. Integer Overflow in Search Space

```java
// bloomDay can be up to 10^9, m*k can overflow int
if ((long) m * k > bloomDay.length) return -1;  // cast to long first

// hi = max(ranks) * n * n can overflow
long hi = (long) maxRank * n * n;  // use long
```

---

## 4. Floating Point Binary Search

```java
// For continuous search space (gas stations, min distance)
// Use epsilon comparison instead of lo < hi
while (hi - lo > 1e-6) {
    double mid = (lo + hi) / 2;
    if (feasible(mid)) hi = mid;
    else lo = mid;
}
// No +1 needed — floating point naturally converges
```

---

## 5. Off-by-one in Feasibility Check

```java
// Koko: total hours must be <= h (not < h)
return hours <= h;

// Bouquets: bouquets must be >= m (not >)
return bouquets >= m;

// Groups: parts must be <= m
return parts <= m;
```

---

---

# Quick Reference Card

## Identify the Pattern

```
"minimize the maximum X"  → Template 1 (minimize)
"maximize the minimum X"  → Template 2 (maximize)
"find minimum X such that condition holds" → Template 1
"find maximum X such that condition holds" → Template 2

Monotonic property check:
  If X works, does X+1 also work? → YES → binary search on answer
```

## Template 1: Minimize

```java
lo = minAnswer; hi = maxAnswer;
while (lo < hi) {
    mid = lo + (hi - lo) / 2;      // lower mid
    if (feasible(mid)) hi = mid;
    else lo = mid + 1;
}
return lo;
```

## Template 2: Maximize

```java
lo = minAnswer; hi = maxAnswer;
while (lo < hi) {
    mid = lo + (hi - lo + 1) / 2;  // upper mid
    if (feasible(mid)) lo = mid;
    else hi = mid - 1;
}
return lo;
```

## Search Space Cheat Sheet

```
Speed/rate problems:     lo=1,           hi=max(input)
Capacity problems:       lo=max(input),  hi=sum(input)
Distance problems:       lo=1,           hi=max-min
Day problems:            lo=1,           hi=max(days)
Divisor problems:        lo=1,           hi=max(nums)
```

## Feasibility Check Patterns

```
Split into groups (minimize max):
  parts=1, curr=0
  if curr+next > mid → parts++, curr=0
  curr += next
  return parts <= k

Place with min gap (maximize min):
  count=1, last=pos[0]
  if pos[i]-last >= mid → count++, last=pos[i]
  return count >= m

Consecutive count:
  consecutive=0, bouquets=0
  if valid → consecutive++; if consecutive==k → bouquets++, consecutive=0
  else consecutive=0
  return bouquets >= m
```

## Complexity

| | Time | Space |
|---|---|---|
| Binary search | O(log(hi-lo)) | O(1) |
| Feasibility check | O(n) typical | O(1) |
| **Total** | **O(n log(hi-lo))** | **O(1)** |
