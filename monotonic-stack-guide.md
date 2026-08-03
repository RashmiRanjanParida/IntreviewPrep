# Monotonic Stack — Complete Guide for Google L6

---

## What is a Monotonic Stack?

A stack where elements are always in **increasing or decreasing order** from bottom to top.

When a new element violates the order → **pop** until order is restored → **push**.

```
Monotonic Decreasing Stack (large → small, bottom → top):

Push 5: [5]
Push 3: [5, 3]
Push 7: 7 > 3 → pop 3, 7 > 5 → pop 5, push 7 → [7]
Push 2: [7, 2]
Push 6: 6 > 2 → pop 2, 6 < 7 → stop, push 6 → [7, 6]
```

---

## The Magic — What Popping Tells You

```
When you pop X because new element Y violates order:

Decreasing stack: Y > X → Y is NEXT GREATER element of X
Increasing stack: Y < X → Y is NEXT SMALLER element of X

This gives O(n) solution to problems that are O(n²) brute force.
Each element is pushed and popped at most once → O(n) total.
```

---

## How to Identify

| Signal | Problem |
|---|---|
| "next greater/smaller element" | Daily Temperatures, Next Greater Element |
| "previous greater/smaller" | Largest Rectangle in Histogram |
| "trapped water", "container" | Trapping Rain Water |
| "span", "dominance range" | Stock Span |
| Brute force needs nested loops O(n²) | Most problems here |

---

## Two Types

| Type | Order (bottom→top) | Pop when | Finds |
|---|---|---|---|
| Decreasing | large → small | `new > top` | Next Greater |
| Increasing | small → large | `new < top` | Next Smaller |

---

## Store Index vs Value

```
Store INDEX when:
  → need distance/position (i - popped)
  → need to access value AND position

Store VALUE when:
  → only need value at pop time
  → doing map lookups (Next Greater Element I)
```

---

## Universal Skeleton

```java
Deque<Integer> stack = new ArrayDeque<>();  // prefer Deque over Stack

for (int i = 0; i < n; i++) {
    // Decreasing stack — pop when new element is GREATER
    while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
        int idx = stack.pop();
        // nums[i] is the NEXT GREATER of nums[idx]
        answer[idx] = i;  // or nums[i], or i - idx
    }
    stack.push(i);  // push index
}
// remaining elements have no next greater → default value (0 or -1)
```

---

## The 5 Questions Before Coding

```
1. Increasing or decreasing stack?
   → next greater: decreasing
   → next smaller: increasing

2. Store index or value?
   → need position/distance: index
   → need map lookup: value

3. When do we pop?
   → decreasing: new > top
   → increasing: new < top

4. What does popping tell us?
   → right boundary, next greater/smaller, area, water

5. What to do with remaining elements?
   → no answer: fill with -1 or 0
   → need to process: append sentinel (0 or MAX_VALUE)
```

---

---

# Problems

## Core Problems

### 1. Daily Temperatures (LC 739)
**Description:** Given an array of daily temperatures, return an array where answer[i] is the number of days you have to wait after day i to get a warmer temperature. 0 if no warmer day exists.

**Example:**
- Input: temperatures = [73,74,75,71,69,72,76,73]
- Output: [1,1,4,2,1,1,0,0]  (day 0: next warmer is day 1 (+1), day 2: next warmer is day 6 (+4))

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        Deque<Integer> stack = new ArrayDeque<>();
        int[] ans = new int[temperatures.length];

        for (int i = 0; i < temperatures.length; i++) {
            while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
                int idx = stack.pop();
                ans[idx] = i - idx;  // days until warmer
            }
            stack.push(i);
        }
        return ans;  // remaining stay 0 (no warmer day)
    }
}
```

**Stack**: Decreasing. **Pop when**: new temp > top. **Answer at pop**: `i - popped` (distance).

---

### 2. Next Greater Element I (LC 496)
**Description:** nums1 is a subset of nums2. For each element in nums1, find the first greater element to its right in nums2. Return -1 if no such element exists.

**Example:**
- Input: nums1 = [4,1,2], nums2 = [1,3,4,2]
- Output: [-1,3,-1]  (4: no greater; 1: next greater in nums2 is 3; 2: no greater)

```java
class Solution {
    public int[] nextGreaterElement(int[] nums1, int[] nums2) {
        Map<Integer, Integer> map = new HashMap<>();
        Deque<Integer> stack = new ArrayDeque<>();

        for (int n : nums2) {
            while (!stack.isEmpty() && stack.peek() < n) {
                map.put(stack.pop(), n);  // popped's next greater = n
            }
            stack.push(n);  // store value (not index) for map lookup
        }

        int[] ans = new int[nums1.length];
        Arrays.fill(ans, -1);
        for (int i = 0; i < nums1.length; i++)
            ans[i] = map.getOrDefault(nums1[i], -1);
        return ans;
    }
}
```

**Stack**: Decreasing. **Store**: values (for map lookup). **Remaining**: -1 (no next greater).

---

### 3. Next Greater Element II (LC 503)
**Description:** Given a circular array, find the next greater element for each element. The search wraps around to the beginning.

**Example:**
- Input: nums = [1,2,1]
- Output: [2,-1,2]  (1: next greater is 2; 2: no greater; 1: wraps around to find 2)

```java
class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] ans = new int[n];
        Arrays.fill(ans, -1);
        Deque<Integer> stack = new ArrayDeque<>();

        // iterate twice to simulate circular array
        for (int i = 0; i < 2 * n; i++) {
            while (!stack.isEmpty() && nums[stack.peek()] < nums[i % n]) {
                ans[stack.pop()] = nums[i % n];
            }
            if (i < n) stack.push(i);  // only push indices in first pass
        }
        return ans;
    }
}
```

**Trick**: iterate `2*n` times using `i % n`. Only push in first pass to avoid duplicates.

---

### 4. Largest Rectangle in Histogram (LC 84)
**Description:** Given heights of bars in a histogram, find the area of the largest rectangle that can be formed using contiguous bars.

**Example:**
- Input: heights = [2,1,5,6,2,3]
- Output: 10  (bars at indices 2,3 with height 5 → area = 5×2 = 10)

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int[] h = new int[heights.length + 1];  // append 0 sentinel
        for (int i = 0; i < heights.length; i++) h[i] = heights[i];

        Deque<Integer> stack = new ArrayDeque<>();
        int max = 0;

        for (int i = 0; i < h.length; i++) {
            while (!stack.isEmpty() && h[stack.peek()] > h[i]) {
                int top = stack.pop();
                int left = stack.isEmpty() ? -1 : stack.peek();
                int width = i - left - 1;
                max = Math.max(max, h[top] * width);
            }
            stack.push(i);
        }
        return max;
    }
}
```

**Stack**: Increasing. **Pop when**: new < top. **At pop**: `right=i`, `left=stack.peek()`, `width=right-left-1`.
**Sentinel**: append 0 to flush all remaining bars.

---

### 5. Trapping Rain Water (LC 42)
**Description:** Given an elevation map (array of bar heights), calculate the total amount of water that can be trapped after raining.

**Example:**
- Input: height = [0,1,0,2,1,0,1,3,2,1,2,1]
- Output: 6

**Two Pointer (O(1) space — preferred)**:
```java
class Solution {
    public int trap(int[] height) {
        int left = 0, right = height.length - 1;
        int maxLeft = 0, maxRight = 0, water = 0;

        while (left < right) {
            if (height[left] < height[right]) {
                if (height[left] >= maxLeft) maxLeft = height[left];
                else water += maxLeft - height[left];
                left++;
            } else {
                if (height[right] >= maxRight) maxRight = height[right];
                else water += maxRight - height[right];
                right--;
            }
        }
        return water;
    }
}
```

**Monotonic Stack (O(n) space)**:
```java
class Solution {
    public int trap(int[] height) {
        Deque<Integer> stack = new ArrayDeque<>();
        int water = 0;

        for (int i = 0; i < height.length; i++) {
            while (!stack.isEmpty() && height[stack.peek()] < height[i]) {
                int bottom = stack.pop();
                if (stack.isEmpty()) break;
                int left = stack.peek();
                int width = i - left - 1;
                int h = Math.min(height[left], height[i]) - height[bottom];
                water += h * width;
            }
            stack.push(i);
        }
        return water;
    }
}
```

---

### 6. Online Stock Span (LC 901)
**Description:** Design a class that collects daily stock prices and returns the stock's span for that day — the maximum number of consecutive days (up to and including today) where the price was ≤ today's price.

**Example:**
- Input: prices = [100,80,60,70,60,75,85]
- Output: [1,1,1,2,1,4,6]  (price 75: span=4 because [60,70,60,75] all ≤ 75)

```java
class StockSpanner {
    Deque<int[]> stack = new ArrayDeque<>();  // [price, span]

    public int next(int price) {
        int span = 1;
        while (!stack.isEmpty() && stack.peek()[0] <= price)
            span += stack.pop()[1];  // absorb span of popped days
        stack.push(new int[]{price, span});
        return span;
    }
}
```

**Key insight**: store `[price, span]` pairs. When popping, absorb their span into current.

---

### 7. Sum of Subarray Minimums (LC 907)
**Description:** Find the sum of min(b) for every (contiguous) subarray b of array arr. Return result mod 10^9 + 7.

**Example:**
- Input: arr = [3,1,2,4]
- Output: 17  (subarrays: [3]=3,[1]=1,[2]=2,[4]=4,[3,1]=1,[1,2]=1,[2,4]=2,[3,1,2]=1,[1,2,4]=1,[3,1,2,4]=1 → sum=17)

```java
class Solution {
    public int sumSubarrayMins(int[] arr) {
        int MOD = 1_000_000_007, n = arr.length;
        long ans = 0;
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i <= n; i++) {
            while (!stack.isEmpty() && (i == n || arr[stack.peek()] >= arr[i])) {
                int mid = stack.pop();
                int left = stack.isEmpty() ? -1 : stack.peek();
                int right = i;
                // arr[mid] is min for subarrays starting in (left, mid] ending in [mid, right)
                long count = (long)(mid - left) * (right - mid);
                ans = (ans + count * arr[mid]) % MOD;
            }
            stack.push(i);
        }
        return (int) ans;
    }
}
```

**Key**: for each element as minimum, count subarrays where it's the minimum.
`left boundary * right boundary = number of such subarrays`.

---

### 8. Maximal Rectangle (LC 85)
**Description:** Given a binary matrix of '0's and '1's, find the largest rectangle containing only '1's and return its area.

**Example:**
- Input: matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
- Output: 6

```java
class Solution {
    public int maximalRectangle(char[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        int[] heights = new int[n];
        int max = 0;

        for (int i = 0; i < m; i++) {
            // build histogram for each row
            for (int j = 0; j < n; j++)
                heights[j] = matrix[i][j] == '1' ? heights[j] + 1 : 0;
            max = Math.max(max, largestRectangle(heights));
        }
        return max;
    }

    int largestRectangle(int[] heights) {
        int[] h = new int[heights.length + 1];
        for (int i = 0; i < heights.length; i++) h[i] = heights[i];
        Deque<Integer> stack = new ArrayDeque<>();
        int max = 0;

        for (int i = 0; i < h.length; i++) {
            while (!stack.isEmpty() && h[stack.peek()] > h[i]) {
                int top = stack.pop();
                int left = stack.isEmpty() ? -1 : stack.peek();
                max = Math.max(max, h[top] * (i - left - 1));
            }
            stack.push(i);
        }
        return max;
    }
}
```

**Pattern**: build histogram row by row → apply Largest Rectangle in Histogram.

---

### 9. Remove K Digits (LC 402)
**Description:** Remove k digits from number string to produce the smallest possible number. Leading zeros in the result should be removed.

**Example:**
- Input: num = "1432219", k = 3
- Output: "1219"  (remove 4, 3, 2 from left to right when next digit is smaller)

```java
class Solution {
    public String removeKdigits(String num, int k) {
        Deque<Character> stack = new ArrayDeque<>();

        for (char c : num.toCharArray()) {
            // maintain increasing stack (smaller digits at bottom)
            while (k > 0 && !stack.isEmpty() && stack.peek() > c) {
                stack.pop();
                k--;
            }
            stack.push(c);
        }

        // remove remaining digits from top if k still > 0
        while (k-- > 0) stack.pop();

        // build result
        StringBuilder sb = new StringBuilder();
        boolean leadingZero = true;
        for (char c : stack) {
            if (leadingZero && c == '0') continue;
            leadingZero = false;
            sb.append(c);
        }
        return sb.length() == 0 ? "0" : sb.toString();
    }
}
```

**Stack**: Increasing (want smallest digits first). **Pop when**: new < top AND k > 0.

---

### 10. Largest Rectangle with All 1s per Row (Variation)
**Description:** Count total number of rectangles of all sizes containing only 1s in a binary matrix (variation of LC 85 counting all rectangles rather than just max).

**Example:**
- Input: matrix = [["1","1"],["1","1"]]
- Output: 9  (four 1×1 + two 1×2 + two 2×1 + one 2×2 = 9 rectangles)

```java
public int countRectangles(char[][] matrix) {
    int m = matrix.length, n = matrix[0].length;
    int[] heights = new int[n];
    int total = 0;

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++)
            heights[j] = matrix[i][j] == '1' ? heights[j] + 1 : 0;

        // for each bar, count rectangles it can form downward
        Deque<Integer> stack = new ArrayDeque<>();
        for (int j = 0; j <= n; j++) {
            int curr = j == n ? 0 : heights[j];
            while (!stack.isEmpty() && heights[stack.peek()] >= curr) {
                int h = heights[stack.pop()];
                int w = stack.isEmpty() ? j : j - stack.peek() - 1;
                total += h * w;
            }
            stack.push(j);
        }
    }
    return total;
}
```

---

## Advanced Problems

### 11. 132 Pattern (LC 456)
**Description:** Find if there exist indices i < j < k such that nums[i] < nums[k] < nums[j]. The "132 pattern" refers to the relative ordering (small, large, medium).

**Example:**
- Input: nums = [3,1,4,2]
- Output: true  (i=1,j=2,k=3: nums[1]=1 < nums[3]=2 < nums[2]=4)

```java
class Solution {
    public boolean find132pattern(int[] nums) {
        int n = nums.length, third = Integer.MIN_VALUE;
        Deque<Integer> stack = new ArrayDeque<>();

        // iterate from right to left
        for (int i = n - 1; i >= 0; i--) {
            if (nums[i] < third) return true;  // found nums[i] < third (k value)
            while (!stack.isEmpty() && stack.peek() < nums[i])
                third = stack.pop();  // track largest value smaller than current
            stack.push(nums[i]);
        }
        return false;
    }
}
```

**Trick**: traverse right-to-left. Stack maintains candidates for `nums[j]`. `third` = best `nums[k]`.

---

### 12. Sliding Window Maximum (LC 239)
**Description:** Given an array and a sliding window of size k, return an array of the maximum value in each window position as it slides from left to right.

**Example:**
- Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
- Output: [3,3,5,5,6,7]  (windows: [1,3,-1]→3, [3,-1,-3]→3, [-1,-3,5]→5, ...)

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int[] ans = new int[n - k + 1];
        Deque<Integer> deque = new ArrayDeque<>();  // monotonic decreasing deque

        for (int i = 0; i < n; i++) {
            // remove elements outside window
            while (!deque.isEmpty() && deque.peekFirst() < i - k + 1)
                deque.pollFirst();

            // maintain decreasing order
            while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i])
                deque.pollLast();

            deque.offerLast(i);

            if (i >= k - 1) ans[i - k + 1] = nums[deque.peekFirst()];
        }
        return ans;
    }
}
```

**Monotonic Deque**: remove from front when out of window, remove from back when smaller than new element. Front always = window maximum.

---

### 13. Decode String (LC 394)
**Description:** Decode a string encoded as k[encoded_string], where encoded_string is repeated k times. Brackets may be nested.

**Example:**
- Input: s = "3[a2[c]]"
- Output: "accaccacc"  (2[c]→"cc", a2[c]→"acc", 3[acc]→"accaccacc")

```java
class Solution {
    public String decodeString(String s) {
        Deque<Integer> countStack = new ArrayDeque<>();
        Deque<StringBuilder> strStack = new ArrayDeque<>();
        StringBuilder curr = new StringBuilder();
        int k = 0;

        for (char c : s.toCharArray()) {
            if (Character.isDigit(c)) {
                k = k * 10 + (c - '0');
            } else if (c == '[') {
                countStack.push(k);
                strStack.push(curr);
                curr = new StringBuilder();
                k = 0;
            } else if (c == ']') {
                int repeat = countStack.pop();
                StringBuilder prev = strStack.pop();
                for (int i = 0; i < repeat; i++) prev.append(curr);
                curr = prev;
            } else {
                curr.append(c);
            }
        }
        return curr.toString();
    }
}
```

---

### 14. Valid Parentheses (LC 20)
**Description:** Given a string containing only '(', ')', '{', '}', '[', ']', determine if the input string is valid. Brackets must close in the correct order.

**Example:**
- Input: s = "()[]{}"  → Output: true
- Input: s = "([)]"    → Output: false
- Input: s = "{[]}"    → Output: true

```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> stack = new ArrayDeque<>();
        for (char c : s.toCharArray()) {
            if (c == '(' || c == '[' || c == '{') stack.push(c);
            else {
                if (stack.isEmpty()) return false;
                char top = stack.pop();
                if (c == ')' && top != '(') return false;
                if (c == ']' && top != '[') return false;
                if (c == '}' && top != '{') return false;
            }
        }
        return stack.isEmpty();
    }
}
```

---

### 15. Number of Visible People in a Queue (LC 1944)
**Description:** People stand in a queue with given heights. Person i can see person j (j > i) if all people between them have height strictly less than both. Return how many people each person can see.

**Example:**
- Input: heights = [10,6,8,5,11,9]
- Output: [3,1,2,1,1,0]  (person 0 of height 10 can see persons 1,2,4)

```java
class Solution {
    public int[] canSeePersonsCount(int[] heights) {
        int n = heights.length;
        int[] ans = new int[n];
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = n - 1; i >= 0; i--) {
            int count = 0;
            while (!stack.isEmpty() && heights[i] > stack.peek()) {
                stack.pop();
                count++;  // person i can see this shorter person
            }
            if (!stack.isEmpty()) count++;  // can see the first taller person
            ans[i] = count;
            stack.push(heights[i]);
        }
        return ans;
    }
}
```

---

---

# Common Pitfalls

## 1. Stack vs Deque in Java

```java
// WRONG — Stack is legacy, synchronized, slow
Stack<Integer> stack = new Stack<>();

// CORRECT — Deque is preferred
Deque<Integer> stack = new ArrayDeque<>();
stack.push(i);      // push to front
stack.peek();       // peek front
stack.pop();        // pop from front
stack.isEmpty();
```

---

## 2. Index vs Value — Wrong Choice

```java
// Need distance → must store index
ans[idx] = i - idx;  // impossible without index

// Need map lookup → store value
map.put(stack.pop(), nums[i]);  // store value for O(1) lookup
```

---

## 3. Wrong Stack Direction

```java
// Next GREATER → decreasing stack, pop when new > top
while (!stack.isEmpty() && nums[stack.peek()] < nums[i])  // pop smaller

// Next SMALLER → increasing stack, pop when new < top
while (!stack.isEmpty() && nums[stack.peek()] > nums[i])  // pop larger
```

---

## 4. Forgetting Sentinel for Histogram

```java
// Without sentinel — remaining bars never processed
heights = [2,1,5,6,2,3]
// after loop: stack=[1,4,5] → never computed their areas

// With sentinel — forces all pops
int[] h = Arrays.copyOf(heights, heights.length + 1);
h[heights.length] = 0;  // 0 is smaller than everything → flushes stack
```

---

## 5. Circular Array — Iterate 2n

```java
// Circular: wrap around to find next greater
for (int i = 0; i < 2 * n; i++) {
    while (!stack.isEmpty() && nums[stack.peek()] < nums[i % n])
        ans[stack.pop()] = nums[i % n];
    if (i < n) stack.push(i);  // only push first n elements
}
```

---

## 6. Width Formula in Histogram

```java
// After popping top:
int left = stack.isEmpty() ? -1 : stack.peek();
int width = i - left - 1;
// NOT: i - top - 1 (wrong — ignores elements between left and top)
// NOT: i - left   (off by one — boundaries are exclusive)
```

---

---

# Quick Reference Card

## Pattern Decision

```
Next GREATER element    → decreasing stack, pop when new > top
Next SMALLER element    → increasing stack, pop when new < top
Previous GREATER        → what's in stack when you push = left boundary
Previous SMALLER        → same, for increasing stack
Histogram area          → increasing stack, width = right - left - 1
Water trapped           → two pointers (O(1)) or decreasing stack
Sliding window max      → monotonic deque (remove from both ends)
```

## At Pop Time — What You Know

```
Popped index = top
Right boundary = i (current)           ← first element that caused the pop
Left boundary  = stack.peek()          ← nearest surviving element
              = -1 if stack empty      ← extends to beginning

Width (histogram) = i - left - 1
Water height      = min(height[left], height[right]) - height[bottom]
Distance          = i - top
```

## Sentinel Values

```
Histogram (flush remaining): append 0 at end
Next greater (no answer):    fill with -1
Daily temps (no answer):     fill with 0 (Java default)
Circular array:              iterate 2n times
```

## Complexity

| | Time | Space |
|---|---|---|
| All monotonic stack problems | O(n) | O(n) |
| Two pointer (rain water) | O(n) | O(1) |
| Sliding window max (deque) | O(n) | O(k) |

Each element pushed and popped at most once → O(n) guaranteed.

## Problem → Pattern Map

| Problem | Stack Type | Store | Sentinel |
|---|---|---|---|
| Daily Temperatures | Decreasing | Index | None (0 default) |
| Next Greater I | Decreasing | Value | None (-1 fill) |
| Next Greater II | Decreasing | Index | 2n iteration |
| Largest Rectangle | Increasing | Index | Append 0 |
| Trapping Rain Water | Decreasing | Index | None |
| Stock Span | Decreasing | [price,span] | None |
| Sum Subarray Mins | Increasing | Index | Append 0 |
| Maximal Rectangle | Increasing | Index | Append 0 per row |
| Remove K Digits | Increasing | Value | None |
| 132 Pattern | Decreasing | Value | Traverse right→left |
| Sliding Window Max | Decreasing Deque | Index | None |
