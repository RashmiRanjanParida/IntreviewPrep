# Sliding Window + Two Pointers — Complete Guide for Google L6

---

## What Are They?

Two techniques that avoid nested loops by maintaining a **window** or **two positions** that move intelligently.

```
Brute force:             O(n²) — try all pairs/subarrays
Sliding Window/2ptr:     O(n)  — each element visited at most twice
```

---

## Two Techniques

| | Sliding Window | Two Pointers |
|---|---|---|
| Pointers | `left`, `right` same direction → | `left`, `right` opposite ← → |
| Window | expands right, shrinks left | converges inward |
| Use when | subarray/substring with constraint | pair/triplet, sorted array |
| Example | Min Window Substring | Two Sum II, 3Sum |

---

## How to Identify

| Signal | Technique |
|---|---|
| "longest/shortest subarray/substring" | Sliding Window |
| "subarray with sum/product = k" | Sliding Window |
| "find pair/triplet with sum" | Two Pointers |
| "sorted array, two elements" | Two Pointers |
| "container/water between positions" | Two Pointers |
| Window has a constraint to maintain | Sliding Window |

---

## 3 Types of Sliding Window

| Type | Window size | Shrink when | Example |
|---|---|---|---|
| **Fixed** | Always k | After k elements | Max sum subarray size k |
| **Variable maximize** | Grow until invalid, shrink | Window invalid | Longest no-repeat substring |
| **Variable minimize** | Grow until valid, shrink | Window valid | Min window substring |

---

## Skeletons

### Fixed Window
```java
int windowSum = 0, max = 0;
for (int i = 0; i < nums.length; i++) {
    windowSum += nums[i];
    if (i >= k) windowSum -= nums[i - k];   // remove leftmost
    if (i >= k - 1) max = Math.max(max, windowSum);
}
```

### Variable Window — Maximize (shrink when INVALID)
```java
int left = 0, max = 0;
for (int right = 0; right < n; right++) {
    // expand: add s.charAt(right) to window
    while (/* window INVALID */) {
        // shrink: remove s.charAt(left)
        left++;
    }
    max = Math.max(max, right - left + 1);  // update after valid
}
```

### Variable Window — Minimize (shrink when VALID)
```java
int left = 0, min = Integer.MAX_VALUE;
for (int right = 0; right < n; right++) {
    // expand: add to window
    while (/* window VALID */) {
        min = Math.min(min, right - left + 1);  // update before shrink
        // shrink: remove from left
        left++;
    }
}
```

### Two Pointers — Opposite Ends
```java
int left = 0, right = n - 1;
while (left < right) {
    if (condition) {
        // process pair (left, right)
        left++; right--;
    } else if (needLarger) {
        left++;
    } else {
        right--;
    }
}
```

---

## How to Think About Any Problem

```
1. Is it a fixed window or variable window?
   → fixed: window size k given explicitly
   → variable: find longest/shortest

2. What makes window VALID or INVALID?
   → define the constraint clearly

3. What do you track inside the window?
   → frequency map, count, sum, set

4. Maximize or minimize?
   → maximize: update answer when valid, shrink when invalid
   → minimize: update answer when valid, then shrink to find better

5. Two pointers or sliding window?
   → sorted array + pair/triplet → two pointers
   → subarray/substring constraint → sliding window
```

---

---

# Sliding Window Problems

### 1. Longest Substring Without Repeating Characters (LC 3)
Given a string `s`, find the length of the longest substring without repeating characters.

```
Input:  s = "abcabcbb"
Output: 3  → "abc"

Input:  s = "bbbbb"
Output: 1  → "b"

Input:  s = "pwwkew"
Output: 3  → "wke"
```

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Set<Character> set = new HashSet<>();
        int left = 0, max = 0;
        for (int right = 0; right < s.length(); right++) {
            while (set.contains(s.charAt(right))) {
                set.remove(s.charAt(left++));
            }
            set.add(s.charAt(right));
            max = Math.max(max, right - left + 1);
        }
        return max;
    }
}
```
**Type**: Variable maximize. **Invalid**: duplicate char in window.

---

### 2. Minimum Window Substring (LC 76)
Given strings `s` and `t`, return the minimum window substring of `s` such that every character in `t` (including duplicates) is included in the window. Return `""` if no such window exists.

```
Input:  s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"

Input:  s = "a", t = "a"
Output: "a"

Input:  s = "a", t = "aa"
Output: ""  (t has two 'a's, s only has one)
```

```java
class Solution {
    public String minWindow(String s, String t) {
        if (s.isEmpty() || t.isEmpty()) return "";
        Map<Character, Integer> need = new HashMap<>();
        for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);

        int have = 0, total = need.size();
        int left = 0, minLen = Integer.MAX_VALUE, minLeft = 0;
        Map<Character, Integer> window = new HashMap<>();

        for (int right = 0; right < s.length(); right++) {
            // expand
            char c = s.charAt(right);
            window.merge(c, 1, Integer::sum);
            if (need.containsKey(c) && window.get(c).equals(need.get(c)))
                have++;

            // shrink when valid
            while (have == total) {
                if (right - left + 1 < minLen) {
                    minLen = right - left + 1;
                    minLeft = left;
                }
                char lc = s.charAt(left++);
                window.merge(lc, -1, Integer::sum);
                if (need.containsKey(lc) && window.get(lc) < need.get(lc))
                    have--;
            }
        }
        return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
    }
}
```
**Type**: Variable minimize. **Valid**: `have == total` (all chars satisfied).
**Key**: `have` counts how many UNIQUE chars meet required frequency.

---

### 3. Longest Substring with At Most K Distinct Characters (LC 340)
Given string `s` and integer `k`, return the length of the longest substring that contains at most `k` distinct characters.

```
Input:  s = "eceba", k = 2
Output: 3  → "ece"

Input:  s = "aa", k = 1
Output: 2  → "aa"
```

```java
class Solution {
    public int lengthOfLongestSubstringKDistinct(String s, int k) {
        Map<Character, Integer> freq = new HashMap<>();
        int left = 0, max = 0;
        for (int right = 0; right < s.length(); right++) {
            freq.merge(s.charAt(right), 1, Integer::sum);
            while (freq.size() > k) {
                char lc = s.charAt(left++);
                freq.merge(lc, -1, Integer::sum);
                if (freq.get(lc) == 0) freq.remove(lc);
            }
            max = Math.max(max, right - left + 1);
        }
        return max;
    }
}
```
**Invalid**: more than k distinct chars in window.

---

### 4. Longest Repeating Character Replacement (LC 424)
Given string `s` and integer `k`, you can replace at most `k` characters. Return the length of the longest substring containing the same letter after replacements.

```
Input:  s = "ABAB", k = 2
Output: 4  → replace both 'A' or both 'B'

Input:  s = "AABABBA", k = 1
Output: 4  → "AABA" replace 'B' at index 3
```

```java
class Solution {
    public int characterReplacement(String s, int k) {
        int[] freq = new int[26];
        int left = 0, maxFreq = 0, max = 0;
        for (int right = 0; right < s.length(); right++) {
            freq[s.charAt(right) - 'A']++;
            maxFreq = Math.max(maxFreq, freq[s.charAt(right) - 'A']);
            // window size - maxFreq = chars to replace
            if (right - left + 1 - maxFreq > k) {
                freq[s.charAt(left++) - 'A']--;
            }
            max = Math.max(max, right - left + 1);
        }
        return max;
    }
}
```
**Invalid**: `windowSize - maxFreq > k` (need more than k replacements).

---

### 5. Permutation in String (LC 567)
Given strings `s1` and `s2`, return `true` if `s2` contains a permutation of `s1` (i.e., one of `s1`'s permutations is a substring of `s2`).

```
Input:  s1 = "ab", s2 = "eidbaooo"
Output: true  → "ba" is a permutation of "ab"

Input:  s1 = "ab", s2 = "eidboaoo"
Output: false
```

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        if (s1.length() > s2.length()) return false;
        int[] need = new int[26], window = new int[26];
        for (char c : s1.toCharArray()) need[c - 'a']++;

        for (int i = 0; i < s2.length(); i++) {
            window[s2.charAt(i) - 'a']++;
            if (i >= s1.length())
                window[s2.charAt(i - s1.length()) - 'a']--;
            if (Arrays.equals(need, window)) return true;
        }
        return false;
    }
}
```
**Type**: Fixed window of size `s1.length()`.

---

### 6. Find All Anagrams in a String (LC 438)
Given strings `s` and `p`, return all start indices of `p`'s anagrams in `s`.

```
Input:  s = "cbaebabacd", p = "abc"
Output: [0, 6]  → "cba" at 0, "bac" at 6

Input:  s = "abab", p = "ab"
Output: [0, 1, 2]
```

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> result = new ArrayList<>();
        int[] need = new int[26], window = new int[26];
        for (char c : p.toCharArray()) need[c - 'a']++;

        for (int i = 0; i < s.length(); i++) {
            window[s.charAt(i) - 'a']++;
            if (i >= p.length())
                window[s.charAt(i - p.length()) - 'a']--;
            if (Arrays.equals(need, window)) result.add(i - p.length() + 1);
        }
        return result;
    }
}
```
**Type**: Fixed window of size `p.length()`.

---

### 7. Maximum Sum Subarray of Size K (Classic)
Given an array and integer `k`, return the maximum sum of any contiguous subarray of size `k`.

```
Input:  nums = [2,1,5,1,3,2], k = 3
Output: 9  → [5,1,3]

Input:  nums = [2,3,4,1,5], k = 2
Output: 7  → [3,4]
```

```java
public int maxSumSubarray(int[] nums, int k) {
    int windowSum = 0, max = Integer.MIN_VALUE;
    for (int i = 0; i < nums.length; i++) {
        windowSum += nums[i];
        if (i >= k) windowSum -= nums[i - k];
        if (i >= k - 1) max = Math.max(max, windowSum);
    }
    return max;
}
```

---

### 8. Subarray Product Less Than K (LC 713)
Given array `nums` and integer `k`, return the number of contiguous subarrays where the product of all elements is strictly less than `k`.

```
Input:  nums = [10,5,2,6], k = 100
Output: 8  → [10],[5],[2],[6],[10,5],[5,2],[2,6],[5,2,6]

Input:  nums = [1,2,3], k = 0
Output: 0
```

```java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        if (k <= 1) return 0;
        int left = 0, product = 1, count = 0;
        for (int right = 0; right < nums.length; right++) {
            product *= nums[right];
            while (product >= k) product /= nums[left++];
            count += right - left + 1;  // all subarrays ending at right
        }
        return count;
    }
}
```
**Key**: `right - left + 1` counts all valid subarrays ending at `right`.

---

### 9. Minimum Size Subarray Sum (LC 209)
Given array `nums` and integer `target`, return the minimum length of a contiguous subarray whose sum is greater than or equal to `target`. Return `0` if no such subarray exists.

```
Input:  target = 7, nums = [2,3,1,2,4,3]
Output: 2  → [4,3]

Input:  target = 4, nums = [1,4,4]
Output: 1  → [4]

Input:  target = 11, nums = [1,1,1,1,1,1,1,1]
Output: 0
```

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int left = 0, sum = 0, min = Integer.MAX_VALUE;
        for (int right = 0; right < nums.length; right++) {
            sum += nums[right];
            while (sum >= target) {
                min = Math.min(min, right - left + 1);
                sum -= nums[left++];
            }
        }
        return min == Integer.MAX_VALUE ? 0 : min;
    }
}
```
**Type**: Variable minimize. **Valid**: `sum >= target`.

---

### 10. Sliding Window Maximum (LC 239)
Given array `nums` and integer `k`, return an array of the maximum value in each sliding window of size `k`.

```
Input:  nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
        Window [1,3,-1] → 3
        Window [3,-1,-3] → 3
        Window [-1,-3,5] → 5
        Window [-3,5,3] → 5
        Window [5,3,6] → 6
        Window [3,6,7] → 7
```

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int[] ans = new int[n - k + 1];
        Deque<Integer> deque = new ArrayDeque<>();  // monotonic decreasing

        for (int i = 0; i < n; i++) {
            // remove out-of-window elements from front
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
**Type**: Fixed window + monotonic deque. Front = window max.

---

---

# Two Pointers Problems

### 11. Two Sum II (LC 167)
Given a 1-indexed sorted array, find two numbers that add up to `target`. Return their 1-based indices.

```
Input:  numbers = [2,7,11,15], target = 9
Output: [1,2]  → numbers[0] + numbers[1] = 2 + 7 = 9

Input:  numbers = [2,3,4], target = 6
Output: [1,3]  → 2 + 4 = 6
```

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int left = 0, right = numbers.length - 1;
        while (left < right) {
            int sum = numbers[left] + numbers[right];
            if (sum == target) return new int[]{left + 1, right + 1};
            else if (sum < target) left++;
            else right--;
        }
        return new int[]{};
    }
}
```

---

### 12. 3Sum (LC 15)
Given integer array `nums`, return all unique triplets `[nums[i], nums[j], nums[k]]` such that `i != j != k` and their sum is 0.

```
Input:  nums = [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]

Input:  nums = [0,1,1]
Output: []

Input:  nums = [0,0,0]
Output: [[0,0,0]]
```

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();
        for (int i = 0; i < nums.length - 2; i++) {
            if (i > 0 && nums[i] == nums[i-1]) continue;  // skip dups
            int left = i + 1, right = nums.length - 1;
            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                if (sum == 0) {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    while (left < right && nums[left] == nums[left+1]) left++;
                    while (left < right && nums[right] == nums[right-1]) right--;
                    left++; right--;
                } else if (sum < 0) left++;
                else right--;
            }
        }
        return result;
    }
}
```
**Pattern**: fix one element, two pointers on rest.

---

### 13. Container With Most Water (LC 11)
Given `n` non-negative integers representing heights of vertical lines, find two lines that together with the x-axis form a container that holds the most water.

```
Input:  height = [1,8,6,2,5,4,8,3,7]
Output: 49  → lines at index 1 (height 8) and 8 (height 7), width = 7, water = 7*7 = 49

Input:  height = [1,1]
Output: 1
```

```java
class Solution {
    public int maxArea(int[] height) {
        int left = 0, right = height.length - 1, max = 0;
        while (left < right) {
            int water = Math.min(height[left], height[right]) * (right - left);
            max = Math.max(max, water);
            if (height[left] < height[right]) left++;
            else right--;
        }
        return max;
    }
}
```
**Key**: always move the shorter side — moving taller side can only decrease width without gaining height.

---

### 14. Trapping Rain Water (LC 42)
Given `n` non-negative integers representing the elevation map, compute how much water it can trap after raining.

```
Input:  height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6

Input:  height = [4,2,0,3,2,5]
Output: 9
```

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
**Key**: process shorter side — shorter side is the bottleneck for water level.

---

### 15. Sort Colors (LC 75)
Given array with only 0s, 1s, and 2s, sort it in-place so all 0s come first, then 1s, then 2s (Dutch National Flag problem).

```
Input:  nums = [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]

Input:  nums = [2,0,1]
Output: [0,1,2]
```

```java
class Solution {
    public void sortColors(int[] nums) {
        int lo = 0, mid = 0, hi = nums.length - 1;
        while (mid <= hi) {
            if (nums[mid] == 0) {
                int tmp = nums[lo]; nums[lo] = nums[mid]; nums[mid] = tmp;
                lo++; mid++;
            } else if (nums[mid] == 1) {
                mid++;
            } else {
                int tmp = nums[mid]; nums[mid] = nums[hi]; nums[hi] = tmp;
                hi--;
            }
        }
    }
}
```
**3 pointers**: lo=boundary of 0s, mid=current, hi=boundary of 2s.

---

### 16. Remove Duplicates from Sorted Array (LC 26)
Given sorted array, remove duplicates in-place so each element appears only once. Return the new length.

```
Input:  nums = [1,1,2]
Output: 2, nums = [1,2,_]

Input:  nums = [0,0,1,1,1,2,2,3,3,4]
Output: 5, nums = [0,1,2,3,4,_,_,_,_,_]
```

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int slow = 0;
        for (int fast = 1; fast < nums.length; fast++) {
            if (nums[fast] != nums[slow]) {
                nums[++slow] = nums[fast];
            }
        }
        return slow + 1;
    }
}
```
**Pattern**: slow pointer = write position, fast pointer = read position.

---

### 17. Move Zeroes (LC 283)
Given integer array, move all 0s to the end while maintaining relative order of non-zero elements. Do it in-place.

```
Input:  nums = [0,1,0,3,12]
Output: [1,3,12,0,0]

Input:  nums = [0]
Output: [0]
```

```java
class Solution {
    public void moveZeroes(int[] nums) {
        int slow = 0;
        for (int fast = 0; fast < nums.length; fast++) {
            if (nums[fast] != 0) {
                nums[slow++] = nums[fast];
            }
        }
        while (slow < nums.length) nums[slow++] = 0;
    }
}
```

---

### 18. Longest Mountain in Array (LC 845)
A mountain subarray increases then decreases (at least one element on each side of the peak). Return the length of the longest mountain subarray.

```
Input:  arr = [2,1,4,7,3,2,5]
Output: 5  → [1,4,7,3,2]

Input:  arr = [2,2,2]
Output: 0  (no mountain)
```

```java
class Solution {
    public int longestMountain(int[] arr) {
        int n = arr.length, max = 0;
        for (int i = 1; i < n - 1; ) {
            if (arr[i-1] < arr[i] && arr[i] > arr[i+1]) {
                int left = i - 1, right = i + 1;
                while (left > 0 && arr[left-1] < arr[left]) left--;
                while (right < n-1 && arr[right] > arr[right+1]) right++;
                max = Math.max(max, right - left + 1);
                i = right;
            } else i++;
        }
        return max;
    }
}
```

---

### 19. Fruit Into Baskets (LC 904)
You have two baskets and can carry only one type of fruit per basket. Starting from any position in a row of trees, pick fruits into baskets (one type per basket). Return max fruits you can pick (pick consecutive trees, stop when you'd need a 3rd type).

```
Input:  fruits = [1,2,1]
Output: 3  → pick all three trees

Input:  fruits = [0,1,2,2]
Output: 3  → [1,2,2]

Input:  fruits = [1,2,3,2,2]
Output: 4  → [2,3,2,2]
```

```java
class Solution {
    public int totalFruit(int[] fruits) {
        Map<Integer, Integer> basket = new HashMap<>();
        int left = 0, max = 0;
        for (int right = 0; right < fruits.length; right++) {
            basket.merge(fruits[right], 1, Integer::sum);
            while (basket.size() > 2) {
                int lf = fruits[left++];
                basket.merge(lf, -1, Integer::sum);
                if (basket.get(lf) == 0) basket.remove(lf);
            }
            max = Math.max(max, right - left + 1);
        }
        return max;
    }
}
```
**Same as**: Longest Substring with At Most 2 Distinct Characters.

---

### 20. Longest Subarray of 1s After Deleting One Element (LC 1493)
Given binary array `nums`, delete exactly one element. Return the size of the longest non-empty subarray containing only 1s in the resulting array.

```
Input:  nums = [1,1,0,1]
Output: 3  → delete index 2, get [1,1,1]

Input:  nums = [0,1,1,1,0,1,1,0,1]
Output: 5  → delete index 4, get [1,1,1,1,1] starting at idx 1

Input:  nums = [1,1,1]
Output: 2  → must delete one element
```

```java
class Solution {
    public int longestSubarray(int[] nums) {
        int left = 0, zeros = 0, max = 0;
        for (int right = 0; right < nums.length; right++) {
            if (nums[right] == 0) zeros++;
            while (zeros > 1) {
                if (nums[left++] == 0) zeros--;
            }
            max = Math.max(max, right - left);  // right-left not +1 (deleted one)
        }
        return max;
    }
}
```

---

---

# Common Pitfalls

## 1. Update Answer at Wrong Time

```java
// Variable MAXIMIZE — update AFTER shrinking (when window is valid)
while (invalid) { shrink; left++; }
max = Math.max(max, right - left + 1);  // ✅ after while

// Variable MINIMIZE — update BEFORE shrinking (when window just became valid)
while (valid) {
    min = Math.min(min, right - left + 1);  // ✅ inside while
    shrink; left++;
}
```

---

## 2. Window Size Formula

```java
// Current window size
right - left + 1

// After removing one element (LC 1493)
right - left  // not +1
```

---

## 3. `have` Counter in Min Window

```java
// WRONG — increment have for every matching char
if (window.get(c) == need.get(c)) have++;  // but what if window.get(c) > need.get(c)?

// CORRECT — only increment when crossing the threshold
if (need.containsKey(c) && window.get(c).equals(need.get(c))) have++;
// use .equals() not == for Integer comparison
```

---

## 4. Two Pointers — Skipping Duplicates in 3Sum

```java
// Skip duplicate for outer loop
if (i > 0 && nums[i] == nums[i-1]) continue;

// Skip duplicates for inner pointers AFTER finding a valid triplet
while (left < right && nums[left] == nums[left+1]) left++;
while (left < right && nums[right] == nums[right-1]) right--;
left++; right--;  // then move both
```

---

## 5. Fixed Window — Off by One

```java
// Start recording answer when window is full
if (i >= k - 1) ans[i - k + 1] = ...;  // not i >= k
```

---

---

# Quick Reference Card

## Decision Tree

```
Subarray/substring with constraint?
  → Sliding Window

Fixed size k?
  → Fixed window skeleton

Variable, find LONGEST?
  → expand right freely
  → shrink left when INVALID
  → update answer after shrink

Variable, find SHORTEST?
  → expand right freely
  → shrink left when VALID (update inside while)
  → answer = min of all valid windows

Sorted array, find pair/triplet?
  → Two Pointers (opposite ends)

Unsorted, need all pairs?
  → HashMap (not two pointers)
```

## What to Track in Window

```
Char frequency  → int[26] or HashMap
Distinct count  → HashMap size
Sum             → running int
Max/min         → monotonic deque
Condition count → have/total pattern
```

## Complexity

| | Time | Space |
|---|---|---|
| All sliding window | O(n) | O(1) or O(k) |
| Two pointers | O(n) after sort | O(1) |
| 3Sum | O(n²) | O(1) |
| Sliding window max | O(n) | O(k) |

## Problem → Pattern Map

| Problem | Type | Key Data Structure |
|---|---|---|
| Longest no-repeat | Variable maximize | HashSet |
| Min window substring | Variable minimize | 2 HashMaps + have/total |
| K distinct chars | Variable maximize | HashMap |
| Char replacement | Variable maximize | int[26] + maxFreq |
| Permutation in string | Fixed | int[26] comparison |
| Anagrams | Fixed | int[26] comparison |
| Subarray product < k | Variable maximize | running product |
| Min subarray sum | Variable minimize | running sum |
| Sliding window max | Fixed + deque | Monotonic deque |
| Two Sum II | Two pointers | None |
| 3Sum | Two pointers + outer loop | Sort first |
| Container water | Two pointers | None |
| Trapping rain water | Two pointers | maxLeft, maxRight |
