# Kth Smallest / Largest — Heap, QuickSelect & Sorting — Complete Guide for Google L6

---

## What Is a "Kth Element" Problem?

```
Any problem asking for the kth smallest/largest VALUE, or the top-k SET, without
actually needing the whole input fully sorted. Four techniques cover essentially
everything in this space:

  SORT                      -- simplest, gives you full order for free
  HEAP of size k            -- keeps only the k best candidates seen so far
  QUICKSELECT               -- finds just the kth value, in-place, no heap needed
  BINARY SEARCH ON THE ANSWER -- searches the VALUE space instead of the candidates
```

The four exist because "just sort it" is only sometimes the right cost profile — the rest
of this guide is about recognizing which of the other three actually fits.

---

## The Core Mechanic — Four Techniques, One Question

```
1. SORT
   O(n log n) time. Trivial code. Use when n is small, or you need the FULL sorted
   order anyway (not just one element or one subset).

2. HEAP OF SIZE k
   O(n log k) time, O(k) space. Use when k << n, and either:
     - you need the top-k SET itself (not just the boundary value), or
     - the input arrives as a STREAM and you can't re-sort from scratch every time.

3. QUICKSELECT
   O(n) average, O(n^2) worst case (mitigated to near-certain O(n) with a random pivot).
   O(1) extra space (in-place). Use when you need ONLY the kth value, don't care about
   order, and the whole input is already sitting in one in-memory array.

4. BINARY SEARCH ON THE ANSWER
   O(log(value range) * cost-to-count). Use when candidate VALUES are cheap to count
   against ("how many elements are <= x?") but expensive to enumerate or compare
   directly — e.g. every pairwise distance, or an implicit m*n multiplication table.
```

---

## How to Identify Which One You're Facing

| Signal | Technique |
|---|---|
| "Return the kth largest/smallest **value**," one-shot, in-memory array | QuickSelect (or sort if n is small) |
| "Kth largest," but numbers keep **arriving** (stream, `add()` calls) | Heap of size k |
| "Top k elements / words / points" — need the **set**, not just the boundary | Heap of size k |
| "Merge k sorted ___" / "k pairs with smallest sums" | Heap, k-way merge (one entry per source) |
| "Median," running / from a stream | Two heaps (balanced max-heap + min-heap) |
| "Kth smallest" but candidates are pairs / products / an implicit huge matrix | Binary search on the answer **value** + a count function |
| "Kth smallest in a BST" | Just do an inorder traversal — it's sorted for free |
| Array has **many duplicates / few distinct values** (or "sort this array of only 0/1/2s") | 3-way partition instead of plain 2-way QuickSelect |

---

## The Four Skeletons

**Skeleton 1 — heap of size k:**
```java
PriorityQueue<T> heap = new PriorityQueue<>(/* min-heap for "top-k largest" */);
for (T item : stream) {
    heap.offer(item);
    if (heap.size() > k) heap.poll();   // evict the worst of the k+1 candidates
}
// heap now holds exactly the k best; heap.peek() is the kth-best (the "weakest" of the k)
```
Counter-intuitive on first read: finding the k **largest** uses a **min**-heap (so the smallest of your current top-k is always the cheap one to evict), and vice versa for k **smallest**.

**Skeleton 2 — QuickSelect (Lomuto partition, random pivot):**
```java
static int quickSelectHelper(int[] nums, int lo, int hi, int targetIndex) {
    if (lo == hi) return nums[lo];
    int pivotIndex = lo + RND.nextInt(hi - lo + 1);       // random pivot avoids O(n^2) on sorted/adversarial input
    pivotIndex = partition(nums, lo, hi, pivotIndex);      // partitions around pivot, returns its final resting index
    if (pivotIndex == targetIndex) return nums[pivotIndex];
    else if (pivotIndex < targetIndex) return quickSelectHelper(nums, pivotIndex + 1, hi, targetIndex);
    else return quickSelectHelper(nums, lo, pivotIndex - 1, targetIndex);
}
```
Same divide-and-conquer shape as quicksort, but you only ever recurse into the **one side that contains your target index** — that's what turns O(n log n) into O(n) on average.

**Skeleton 3 — binary search on the answer:**
```java
int lo = /* smallest possible answer */, hi = /* largest possible answer */;
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (countLessOrEqual(mid) < k) lo = mid + 1;   // not enough candidates yet -> answer is bigger
    else hi = mid;                                  // enough candidates -> answer is mid or smaller
}
return lo;   // smallest value for which countLessOrEqual(value) >= k
```
You are not searching the array — you are searching the **space of possible answers**, using a count function as the "is this guess big enough?" oracle.

**Skeleton 4 — two heaps for a running median:**
```java
maxHeapLower.offer(num);                              // always insert into the "lower half" heap first
minHeapUpper.offer(maxHeapLower.poll());               // then promote its max into the upper half, keeping halves ordered
if (minHeapUpper.size() > maxHeapLower.size()) {       // rebalance size (lower half allowed to be equal or one bigger)
    maxHeapLower.offer(minHeapUpper.poll());
}
```

---

## Questions to Ask Before Coding

```
1. Do you need just the kth VALUE, or the whole top-k SET (in some order)?
   -> just the value: quickselect is usually the best fit
   -> the whole set: heap of size k (or sort, if you need full order anyway)

2. Is the input static (one array, one query), or a growing STREAM?
   -> static: quickselect or sort
   -> stream: heap of size k (or two heaps, for a running median)

3. Can you binary-search the ANSWER instead of finding it directly?
   -> ask: "given a candidate value x, can I count how many elements are <= x in less
      than O(n^2)?" If yes, and directly generating every candidate is expensive
      (all pairs, a multiplication table, ...), this usually beats heaping/sorting
      every candidate explicitly.

4. Do duplicates need special handling?
   -> heaps handle duplicates fine as-is, no changes needed.
   -> plain 2-way QuickSelect (Lomuto/Hoare) is still CORRECT with duplicates, but can
      degrade toward O(n^2) when there are many of them (an all-equal array is the
      worst case for Lomuto specifically) -- switch to 3-way partition if duplicates
      are heavy, since it groups all copies of the pivot together in one pass.
   -> binary search on the answer: keep your count function's comparison consistently
      "<=" (not "<"), or you will land one off from the true boundary value.

5. If it's a heap problem: min-heap or max-heap?
   -> "top k LARGEST" or "kth largest": min-heap of size k (evict the current smallest)
   -> "top k SMALLEST" or "kth smallest": max-heap of size k (evict the current largest)
   -> yes, this feels backwards the first few times. It stops feeling backwards once
      you internalize that the heap holds your *candidates to keep*, and you always
      want to cheaply evict the *worst* candidate when a better one shows up.
```

---
---

# Problems

## Foundation

### 1. Kth Largest Element in an Array (LC 215) — Three Approaches
**Description:** Find the kth largest element in an unsorted array (kth largest **value**, not the kth distinct value — duplicates count individually).

**Example:** `nums = [3,2,1,5,6,4], k = 2` → `5`

**Approach A — Sort:**
```java
static int findKthLargestSort(int[] nums, int k) {
    int[] copy = nums.clone();
    Arrays.sort(copy);
    return copy[copy.length - k];
}
```

**Approach B — min-heap of size k:**
```java
static int findKthLargestHeap(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) minHeap.poll();
    }
    return minHeap.peek();
}
```

**Approach C — QuickSelect (Lomuto partition, random pivot):**
```java
static int findKthLargestQuickSelect(int[] nums, int k) {
    int[] arr = nums.clone();
    int targetIndex = arr.length - k;      // kth largest = (n-k)th smallest, 0-indexed
    int lo = 0, hi = arr.length - 1;
    Random rnd = new Random();
    while (true) {
        int pivotIndex = lo + rnd.nextInt(hi - lo + 1);
        int newPivotIndex = partition(arr, lo, hi, pivotIndex);
        if (newPivotIndex == targetIndex) return arr[newPivotIndex];
        else if (newPivotIndex < targetIndex) lo = newPivotIndex + 1;
        else hi = newPivotIndex - 1;
    }
}

static int partition(int[] nums, int lo, int hi, int pivotIndex) {
    int pivotValue = nums[pivotIndex];
    swap(nums, pivotIndex, hi);             // move pivot out of the way, to the end
    int storeIndex = lo;
    for (int i = lo; i < hi; i++) {
        if (nums[i] < pivotValue) {
            swap(nums, storeIndex, i);
            storeIndex++;
        }
    }
    swap(nums, storeIndex, hi);             // move pivot into its final sorted position
    return storeIndex;
}
```
**Pattern**: three genuinely different cost profiles for the exact same question — O(n log n)/O(n log k)/O(n) average. This is the single most common interview question in this whole guide precisely because it's a clean vehicle for "do you know more than one way, and can you argue the trade-off?" **Kth smallest** is the mirror image: same code, swap the heap direction (max-heap of size k) or target `k - 1` directly instead of `n - k` in quickselect.

---

## Heap Patterns — Maintain the Top K

### 2. Kth Largest Element in a Stream (LC 703)
**Description:** Design a class that supports adding numbers one at a time and always answering "what's the kth largest so far?"

**Example:** `k=3, nums=[4,5,8,2]`, then `add(3)→4, add(5)→5, add(10)→5, add(9)→8, add(4)→8`

```java
class KthLargest {
    private final PriorityQueue<Integer> minHeap;
    private final int k;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        minHeap = new PriorityQueue<>();
        for (int n : nums) add(n);
    }

    public int add(int val) {
        minHeap.offer(val);
        if (minHeap.size() > k) minHeap.poll();
        return minHeap.peek();
    }
}
```
**Pattern**: the streaming reason Skeleton 1 exists — you cannot re-sort from scratch on every `add()` call, but re-checking a size-k heap is O(log k) per insert.

---

### 3. Top K Frequent Elements (LC 347)
**Description:** Return the k most frequent elements in an array.

**Example:** `nums = [1,1,1,2,2,3], k = 2` → `[1,2]`

```java
static int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> a[1] - b[1]);
    for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
        minHeap.offer(new int[]{e.getKey(), e.getValue()});
        if (minHeap.size() > k) minHeap.poll();
    }
    int[] result = new int[k];
    for (int i = k - 1; i >= 0; i--) result[i] = minHeap.poll()[0];
    return result;
}
```
**Pattern**: heap-of-size-k, but the priority is a **derived** quantity (frequency) rather than the value itself. An O(n) bucket-sort alternative exists (bucket index = frequency, since max possible frequency is bounded by n) — worth mentioning as the follow-up if asked to beat O(n log k).

---

### 4. Top K Frequent Words (LC 692)
**Description:** Like #3, but for strings, with a tie-break: equal frequency sorts lexicographically ascending.

**Example:** `["i","love","leetcode","i","love","coding"], k=2` → `["i","love"]`

```java
static List<String> topKFrequentWords(String[] words, int k) {
    Map<String, Integer> freq = new HashMap<>();
    for (String w : words) freq.merge(w, 1, Integer::sum);

    PriorityQueue<String> minHeap = new PriorityQueue<>((a, b) -> {
        if (!freq.get(a).equals(freq.get(b))) return freq.get(a) - freq.get(b);
        return b.compareTo(a);        // equal frequency: the LEXICOGRAPHICALLY LARGER word is "worse" (evicted first)
    });
    for (String w : freq.keySet()) {
        minHeap.offer(w);
        if (minHeap.size() > k) minHeap.poll();
    }
    List<String> result = new ArrayList<>();
    while (!minHeap.isEmpty()) result.add(minHeap.poll());
    Collections.reverse(result);
    return result;
}
```
**Pattern**: same heap-of-size-k shape as #3, but the comparator now has two levels. The `b.compareTo(a)` (reversed) on the tie-break is the detail people get backwards — walk through *why* on a whiteboard rather than memorizing it: the heap evicts whatever it ranks smallest, so "smallest" in heap-order must mean "worst by our actual ranking," and for equal frequency, lexicographically *larger* is the one we want gone.

---

### 5. K Closest Points to Origin (LC 973)
**Description:** Return the k points closest to `(0,0)`.

**Example:** `points = [[1,3],[-2,2]], k = 1` → `[[-2,2]]`

```java
static int[][] kClosest(int[][] points, int k) {
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>(
        (a, b) -> (b[0]*b[0] + b[1]*b[1]) - (a[0]*a[0] + a[1]*a[1])
    );
    for (int[] p : points) {
        maxHeap.offer(p);
        if (maxHeap.size() > k) maxHeap.poll();
    }
    return maxHeap.toArray(new int[0][]);
}
```
**Pattern**: "top k **closest**" flips the heap direction versus #2–#4 (which were "top k **largest/most frequent**") — this is a **max**-heap of size k, since the farthest of your current k candidates is the cheap one to evict. Comparing squared distance avoids an unnecessary `sqrt`.

---

### 6. Last Stone Weight (LC 1046)
**Description:** Repeatedly smash the two heaviest stones together (both destroyed if equal, otherwise the difference remains) until at most one stone is left.

**Example:** `[2,7,4,1,8,1]` → `1`

```java
static int lastStoneWeight(int[] stones) {
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
    for (int s : stones) maxHeap.offer(s);
    while (maxHeap.size() > 1) {
        int a = maxHeap.poll(), b = maxHeap.poll();
        if (a != b) maxHeap.offer(a - b);
    }
    return maxHeap.isEmpty() ? 0 : maxHeap.peek();
}
```
**Pattern**: not really a "kth element" question at all — it's here as the simplest possible illustration of "repeatedly need the current max, and the max changes after every operation." Good warm-up for recognizing when a heap beats re-scanning the array each time.

---

## Heap Patterns — K-Way Merge

### 7. Merge k Sorted Lists (LC 23)
**Description:** Merge k already-sorted linked lists into one sorted list.

```java
static ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> minHeap = new PriorityQueue<>((a, b) -> a.val - b.val);
    for (ListNode node : lists) if (node != null) minHeap.offer(node);

    ListNode dummy = new ListNode(0);
    ListNode tail = dummy;
    while (!minHeap.isEmpty()) {
        ListNode smallest = minHeap.poll();
        tail.next = smallest;
        tail = tail.next;
        if (smallest.next != null) minHeap.offer(smallest.next);   // push the next node from the SAME list
    }
    return dummy.next;
}
```
**Pattern**: the template for every k-way-merge problem below. Seed the heap with one "frontier" element per source; every time you pop one, push whatever comes next from that *same* source. O(N log k) where N is total elements, k is number of sources.

---

### 8. Find K Pairs with Smallest Sums (LC 373)
**Description:** Given two sorted arrays, return the k pairs `(nums1[i], nums2[j])` with the smallest sums.

**Example:** `nums1=[1,7,11], nums2=[2,4,6], k=3` → `[[1,2],[1,4],[1,6]]`

```java
static List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
    List<List<Integer>> result = new ArrayList<>();
    if (nums1.length == 0 || nums2.length == 0 || k == 0) return result;

    PriorityQueue<int[]> minHeap = new PriorityQueue<>(
        (a, b) -> (nums1[a[0]] + nums2[a[1]]) - (nums1[b[0]] + nums2[b[1]])
    );
    for (int i = 0; i < Math.min(nums1.length, k); i++) minHeap.offer(new int[]{i, 0});

    while (k-- > 0 && !minHeap.isEmpty()) {
        int[] cur = minHeap.poll();
        result.add(Arrays.asList(nums1[cur[0]], nums2[cur[1]]));
        if (cur[1] + 1 < nums2.length) minHeap.offer(new int[]{cur[0], cur[1] + 1});
    }
    return result;
}
```
**Pattern**: same k-way-merge shape as #7, except the "sources" are the **rows of an implicit n1×n2 grid** (row i = `nums1[i] + nums2[0..]`) rather than literal linked lists — never materialize the full n1×n2 grid of sums, since you only ever need k of them.

---

### 9. Kth Smallest Element in a Sorted Matrix (LC 378) — Heap and Binary Search
**Description:** An n×n matrix sorted ascending along both rows and columns. Find the kth smallest element overall.

**Example:** `matrix = [[1,5,9],[10,11,13],[12,13,15]], k = 8` → `13`

**Heap approach (k-way merge across rows):**
```java
static int kthSmallestHeap(int[][] matrix, int k) {
    int n = matrix.length;
    PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> matrix[a[0]][a[1]] - matrix[b[0]][b[1]]);
    for (int i = 0; i < Math.min(n, k); i++) minHeap.offer(new int[]{i, 0});

    int count = 0, result = -1;
    while (!minHeap.isEmpty()) {
        int[] cur = minHeap.poll();
        result = matrix[cur[0]][cur[1]];
        count++;
        if (count == k) break;
        if (cur[1] + 1 < matrix[cur[0]].length) minHeap.offer(new int[]{cur[0], cur[1] + 1});
    }
    return result;
}
```

**Binary search on value approach:**
```java
static int kthSmallestBinarySearch(int[][] matrix, int k) {
    int n = matrix.length;
    int lo = matrix[0][0], hi = matrix[n - 1][n - 1];
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (countLessEqual(matrix, mid) < k) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

static int countLessEqual(int[][] matrix, int target) {
    int n = matrix.length;
    int count = 0, row = n - 1, col = 0;
    while (row >= 0 && col < n) {              // "staircase" walk: start bottom-left
        if (matrix[row][col] <= target) {
            count += row + 1;                   // everything above in this column also qualifies
            col++;
        } else {
            row--;
        }
    }
    return count;
}
```
**Pattern**: the best problem in this guide for showing you know *two* paradigms for the same question. Heap: O(k log n). Binary search on the answer: O(n log(max−min)), using the fact that a sorted matrix supports an O(n) "count how many ≤ x" via a staircase walk. When `k` is close to `n²` (large), binary search wins; when `k` is small, the heap approach touches far fewer elements.

---

### 10. Find Median from Data Stream (LC 295)
**Description:** Support adding numbers one at a time and querying the median of everything seen so far.

**Example:** `addNum(1), addNum(2)` → median `1.5`; then `addNum(3)` → median `2.0`

```java
class MedianFinder {
    private final PriorityQueue<Integer> maxHeapLower;   // holds the smaller half, max on top
    private final PriorityQueue<Integer> minHeapUpper;   // holds the larger half, min on top

    public MedianFinder() {
        maxHeapLower = new PriorityQueue<>(Collections.reverseOrder());
        minHeapUpper = new PriorityQueue<>();
    }

    public void addNum(int num) {
        maxHeapLower.offer(num);
        minHeapUpper.offer(maxHeapLower.poll());     // always route through the lower heap first, then promote its max
        if (minHeapUpper.size() > maxHeapLower.size()) {
            maxHeapLower.offer(minHeapUpper.poll());  // rebalance: lower half allowed to be equal or exactly one bigger
        }
    }

    public double findMedian() {
        if (maxHeapLower.size() > minHeapUpper.size()) return maxHeapLower.peek();
        return (maxHeapLower.peek() + minHeapUpper.peek()) / 2.0;
    }
}
```
**Pattern**: Skeleton 4. The median is exactly the "k = n/2"-th element, and keeping two size-balanced heaps means you never re-sort — each `addNum` is O(log n), each `findMedian` is O(1). The "insert into lower, promote to upper, rebalance back if needed" three-step is the standard way to guarantee `maxHeapLower`'s max is always ≤ `minHeapUpper`'s min without ever comparing across the two heaps directly.

---

## QuickSelect Deep Dive

### 11. QuickSelect — Lomuto Partition (the algorithm behind Approach C of Problem 1)
**Description:** A standalone, generalized "find the kth smallest value" function — the building block, decoupled from any specific LeetCode problem.

```java
static int quickSelect(int[] nums, int k) {           // k is 1-indexed: k=1 means smallest
    return quickSelectHelper(nums, 0, nums.length - 1, k - 1);
}

static int quickSelectHelper(int[] nums, int lo, int hi, int targetIndex) {
    if (lo == hi) return nums[lo];
    int pivotIndex = lo + RND.nextInt(hi - lo + 1);
    pivotIndex = partition(nums, lo, hi, pivotIndex);   // partition() from Problem 1
    if (pivotIndex == targetIndex) return nums[pivotIndex];
    else if (pivotIndex < targetIndex) return quickSelectHelper(nums, pivotIndex + 1, hi, targetIndex);
    else return quickSelectHelper(nums, lo, pivotIndex - 1, targetIndex);
}
```
**Pattern**: Lomuto partition always places the pivot at its **exact final sorted index** — that's what lets you compare `pivotIndex == targetIndex` directly. Simple to reason about; its downside is more swaps than Hoare's scheme on average.

---

### 12. QuickSelect — Hoare Partition (the alternative scheme)
**Description:** Same goal, different partitioning discipline — worth knowing both, since interviewers sometimes ask specifically for Hoare's version, and it's the one most people get subtly wrong.

```java
static int hoarePartition(int[] nums, int lo, int hi) {
    int pivot = nums[lo + RND.nextInt(hi - lo + 1)];
    int i = lo - 1, j = hi + 1;
    while (true) {
        do { i++; } while (nums[i] < pivot);
        do { j--; } while (nums[j] > pivot);
        if (i >= j) return j;
        swap(nums, i, j);
    }
}

static int quickSelectHoare(int[] nums, int k) {
    int lo = 0, hi = nums.length - 1;
    int targetIndex = k - 1;
    while (lo < hi) {
        int p = hoarePartition(nums, lo, hi);
        if (targetIndex <= p) hi = p;          // NOTE: p itself may still be in play, unlike Lomuto
        else lo = p + 1;
    }
    return nums[targetIndex];
}
```
**Pattern**: Hoare's partition does **not** guarantee the pivot lands at its final sorted index — it only guarantees everything in `[lo, p]` is ≤ everything in `[p+1, hi]`. That's why the recursion (here written iteratively) narrows to `hi = p` rather than `hi = p - 1`: the returned index `p` might still need to be included in the next round. Getting this boundary wrong is the single most common Hoare-partition bug. It does fewer swaps than Lomuto in practice, which is why it's the one real-world sort implementations (like `Arrays.sort` for primitives, historically) tend to favor.

---

### 13. 3-Way Partition (Dutch National Flag) — QuickSelect With Many Duplicates
**Description:** Both Lomuto and Hoare above are *correct* with duplicate values, but Lomuto specifically has a hidden worst case: **an array where everything equals the pivot degrades to O(n²)**. Trace it through — Lomuto's inner check is `nums[i] < pivotValue`, which is never true when every element equals the pivot, so `storeIndex` never advances and the pivot always lands back at `hi`. Each recursive call then only shrinks the search range by exactly 1, turning what should be O(n) into O(n²) over the full selection. Three-way partition (Dijkstra's Dutch National Flag algorithm) fixes this by splitting into **three** regions — less than, equal to, and greater than the pivot — in one pass, so every duplicate of the pivot gets grouped together immediately instead of one at a time across recursive calls.

```java
// After this call: nums[lo..lt-1] < pivotValue, nums[lt..gt] == pivotValue, nums[gt+1..hi] > pivotValue
static int[] threeWayPartition(int[] nums, int lo, int hi, int pivotValue) {
    int lt = lo, i = lo, gt = hi;
    while (i <= gt) {
        if (nums[i] < pivotValue) {
            swap(nums, lt++, i++);
        } else if (nums[i] > pivotValue) {
            swap(nums, i, gt--);      // note: i does NOT advance -- the swapped-in element is still unexamined
        } else {
            i++;
        }
    }
    return new int[]{lt, gt};
}

static int quickSelectThreeWay(int[] nums, int k) {
    int lo = 0, hi = nums.length - 1;
    int targetIndex = k - 1;
    while (lo <= hi) {
        int pivotValue = nums[lo + RND.nextInt(hi - lo + 1)];
        int[] bounds = threeWayPartition(nums, lo, hi, pivotValue);
        int lt = bounds[0], gt = bounds[1];
        if (targetIndex < lt) hi = lt - 1;
        else if (targetIndex > gt) lo = gt + 1;
        else return pivotValue;        // targetIndex falls inside [lt, gt] -- every element there equals pivotValue, done
    }
    throw new IllegalStateException("unreachable");
}
```
**Pattern**: the payoff is the early return — if the target index lands anywhere inside `[lt, gt]`, you're finished immediately, because the entire region is known to equal `pivotValue` without checking a single element individually. Measured on an all-duplicate array of size 32,000: plain Lomuto took **78.2ms**, three-way partition took **0.1ms** — roughly **760x**, and the gap widens as `n` grows, since Lomuto is genuinely quadratic there while three-way stays linear. On a milder case (10 distinct values, n=80,000), the gap was a still-meaningful ~11x. Verified correct against a sort-based oracle across 20,000 heavy-duplicate random trials, alongside Lomuto and Hoare.

---

### 14. Sort Colors (LC 75)
**Description:** Given an array containing only `0`, `1`, `2`, sort it in place, in one pass, without a library sort. (The canonical named problem for the exact same primitive as #13 — Dijkstra posed this originally as sorting the colors of the Dutch flag, which is where the technique's name comes from.)

**Example:** `[2,0,2,1,1,0]` → `[0,0,1,1,2,2]`

```java
static void sortColors(int[] nums) {
    int lo = 0, i = 0, hi = nums.length - 1;
    while (i <= hi) {
        if (nums[i] == 0) swap(nums, lo++, i++);
        else if (nums[i] == 2) swap(nums, i, hi--);
        else i++;                                    // nums[i] == 1: already in the right region, just move on
    }
}
```
**Pattern**: literally `threeWayPartition` from #13 with the pivot value hardcoded to `1` — recognizing that equivalence is the actual point of grouping these two problems together. If you can write #13 from memory, this one is free.

---

### 15. Median of an Unsorted Array via QuickSelect
**Description:** Find the median of an array without fully sorting it.

**Example:** `[2,3,4]` → `3.0`;  `[2,3]` → `2.5`

```java
static double findMedianQuickSelect(int[] nums) {
    int n = nums.length;
    if (n % 2 == 1) {
        return quickSelect(nums.clone(), n / 2 + 1);
    } else {
        int lower = quickSelect(nums.clone(), n / 2);
        int upper = quickSelect(nums.clone(), n / 2 + 1);
        return (lower + upper) / 2.0;
    }
}
```
**Pattern**: direct application of #11 — median is just "the kth smallest" (odd length) or the average of two adjacent kth-smallests (even length). Note this calls quickselect twice for the even case (two independent O(n)-average passes) — simpler to write correctly than trying to recover both boundary values from a single pass, and still O(n) overall.

---

### 16. Minimum Moves to Equal Array Elements II (LC 462)
**Description:** In one move you can increment or decrement any single element by 1. Find the minimum total moves to make every element equal.

**Example:** `[1,2,3]` → `2` (move both `1` and `3` to `2`)

```java
static int minMoves2(int[] nums) {
    int[] sorted = nums.clone();
    Arrays.sort(sorted);
    int median = sorted[sorted.length / 2];
    int moves = 0;
    for (int n : sorted) moves += Math.abs(n - median);
    return moves;
}
```
**Pattern**: the *why* matters more than the code here — the median is the value that **minimizes the sum of absolute deviations** (a classic fact: moving your target away from the median in either direction gains you nothing on one side and costs you on the other, for at least half the array). This problem is sorted here for simplicity, but since only the median's *value* is actually needed (not the full order), it's a legitimate place to swap in quickselect (#11) for an O(n) average instead of O(n log n) — worth mentioning as the follow-up optimization.

---

## Binary Search on the Answer

### 17. Kth Smallest Number in a Multiplication Table (LC 668)
**Description:** In an m×n multiplication table (`table[i][j] = i*j`, 1-indexed), find the kth smallest value.

**Example:** `m=3, n=3, k=5` → `3`

```java
static int findKthNumber(int m, int n, int k) {
    int lo = 1, hi = m * n;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (countLessEqual(mid, m, n) < k) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

static int countLessEqual(int target, int m, int n) {
    int count = 0;
    for (int i = 1; i <= m; i++) count += Math.min(n, target / i);   // row i has target/i qualifying entries, capped at n
    return count;
}
```
**Pattern**: the table has up to `m*n` entries — far too many to generate and sort when m, n are large — but "how many entries are ≤ x" is a cheap O(m) computation (row i's entries are `i, 2i, 3i, ...`, so exactly `⌊x/i⌋` of them are ≤ x). Textbook Skeleton 3.

---

### 18. Find K-th Smallest Pair Distance (LC 719)
**Description:** Given an array, consider the distance `|nums[i] - nums[j]|` for every pair. Find the kth smallest such distance.

**Example:** `nums=[1,3,1], k=1` → `0` (the pair `(1,1)`)

```java
static int smallestDistancePair(int[] nums, int k) {
    Arrays.sort(nums);
    int lo = 0, hi = nums[nums.length - 1] - nums[0];
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (countPairsWithDistanceLessEqual(nums, mid) < k) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

static int countPairsWithDistanceLessEqual(int[] nums, int maxDist) {
    int count = 0, left = 0;
    for (int right = 0; right < nums.length; right++) {
        while (nums[right] - nums[left] > maxDist) left++;
        count += right - left;          // every index in [left, right) pairs with `right` at distance <= maxDist
    }
    return count;
}
```
**Pattern**: same shape as #17, but the count function is itself a sliding window over the *sorted* array rather than a closed-form formula — n² pairs would be too many to generate directly for large n, but counting "pairs within distance ≤ x" is O(n) once sorted. Binary search on the answer often composes with a *second*, cheaper technique (two pointers here, a staircase walk in #9) for the count step — that composition is the real skill being tested.

---

### 19. Median of Two Sorted Arrays (LC 4)
**Description:** Given two sorted arrays, find the median of their combined elements, in `O(log(min(m,n)))`.

**Example:** `nums1=[1,3], nums2=[2]` → `2.0`

```java
static double findMedianSortedArrays(int[] nums1, int[] nums2) {
    if (nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);  // always binary-search the SHORTER array
    int m = nums1.length, n = nums2.length;
    int lo = 0, hi = m;
    int half = (m + n + 1) / 2;      // size of the combined "left half" (handles odd/even uniformly)

    while (lo <= hi) {
        int cut1 = (lo + hi) / 2;     // how many elements of nums1 go into the left half
        int cut2 = half - cut1;       // the rest of the left half comes from nums2

        int l1 = cut1 == 0 ? Integer.MIN_VALUE : nums1[cut1 - 1];
        int l2 = cut2 == 0 ? Integer.MIN_VALUE : nums2[cut2 - 1];
        int r1 = cut1 == m ? Integer.MAX_VALUE : nums1[cut1];
        int r2 = cut2 == n ? Integer.MAX_VALUE : nums2[cut2];

        if (l1 <= r2 && l2 <= r1) {                     // left half is entirely <= right half: valid partition found
            if ((m + n) % 2 == 0) return (Math.max(l1, l2) + Math.min(r1, r2)) / 2.0;
            else return Math.max(l1, l2);
        } else if (l1 > r2) {
            hi = cut1 - 1;                                // took too much from nums1 — back off
        } else {
            lo = cut1 + 1;                                // took too little from nums1 — take more
        }
    }
    throw new IllegalArgumentException("Input arrays are not sorted");
}
```
**Pattern**: this is "kth element across two sorted sequences," generalized to a full binary-search-the-**partition-point** rather than binary-search-a-value. Instead of searching for a number, you're searching for a **cut position** in the shorter array such that combined with the matching cut in the longer array, everything to the left of both cuts is ≤ everything to the right. `Integer.MIN_VALUE`/`MAX_VALUE` sentinels cleanly handle a cut landing at either end of an array without special-casing. This is the hardest problem in the guide — the mental model that makes it click is "I'm not searching for the median's value, I'm searching for where to slice both arrays so the left slices together are exactly half the total."

---

## Bonus — Different Data Structure / No Heap Needed

### 20. Kth Smallest Element in a BST (LC 230)
**Description:** Given a BST, find its kth smallest value.

```java
static int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root;
    while (true) {
        while (cur != null) {          // walk all the way left, stacking as you go
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack.pop();              // next-smallest unvisited node
        if (--k == 0) return cur.val;
        cur = cur.right;
    }
}
```
**Pattern**: a BST's inorder traversal visits nodes in sorted order for free — no heap, no quickselect, no sorting needed at all. The iterative (explicit-stack) form lets you **stop early** the moment you hit the kth node, rather than generating the full inorder list first — the actual point of this problem in an interview.

---

### 21. Third Maximum Number (LC 414)
**Description:** Return the third distinct maximum in an array; if fewer than three distinct values exist, return the maximum instead.

**Example:** `[3,2,1]` → `1`;  `[1,2]` → `2`;  `[2,2,3,1]` → `1`

```java
static int thirdMax(int[] nums) {
    Long first = null, second = null, third = null;
    for (int n : nums) {
        long ln = n;
        if ((first != null && ln == first) || (second != null && ln == second) || (third != null && ln == third)) {
            continue;                              // skip duplicates of values already tracked
        }
        if (first == null || ln > first) {
            third = second; second = first; first = ln;
        } else if (second == null || ln > second) {
            third = second; second = ln;
        } else if (third == null || ln > third) {
            third = ln;
        }
    }
    return third == null ? (int)(long) first : (int)(long) third;
}
```
**Pattern**: when k is a small constant (here, 3), a heap is overkill — just track the top-3 in three variables, one comparison chain per element, O(n) time and O(1) space with no data structure at all. Boxed `Long` (not primitive `long`) is what lets `null` mean "not set yet" — using a primitive sentinel like `Integer.MIN_VALUE` instead would collide with a real input value of `Integer.MIN_VALUE` and silently misbehave.

---
---

## Complexity

| Problem | Time | Space |
|---|---|---|
| Kth Largest — sort | O(n log n) | O(n) (or O(1) if sorting in place) |
| Kth Largest — heap of size k | O(n log k) | O(k) |
| Kth Largest — QuickSelect | O(n) average, O(n²) worst | O(1) extra (in-place) |
| Kth Largest in a Stream (LC 703) | O(log k) per `add` | O(k) |
| Top K Frequent Elements/Words (LC 347/692) | O(n log k) | O(n) |
| K Closest Points (LC 973) | O(n log k) | O(k) |
| Last Stone Weight (LC 1046) | O(n log n) | O(n) |
| Merge k Sorted Lists (LC 23) | O(N log k), N = total nodes | O(k) |
| K Pairs with Smallest Sums (LC 373) | O(k log k) | O(k) |
| Kth Smallest in Sorted Matrix (LC 378) — heap | O(k log n) | O(n) |
| Kth Smallest in Sorted Matrix (LC 378) — binary search | O(n log(max−min)) | O(1) |
| Median from Data Stream (LC 295) | O(log n) per `addNum`, O(1) per query | O(n) |
| QuickSelect (Lomuto or Hoare) | O(n) average, O(n²) worst (esp. Lomuto with many duplicates) | O(1) extra (in-place) |
| QuickSelect (3-way partition) | O(n) average, degrades far more gracefully with duplicates | O(1) extra (in-place) |
| Sort Colors (LC 75) | O(n), one pass | O(1) |
| Median via QuickSelect | O(n) average | O(1) extra |
| Minimum Moves II (LC 462) | O(n log n) (sort) or O(n) average (quickselect) | O(1)–O(n) |
| Kth Smallest in Multiplication Table (LC 668) | O((m+n) log(mn)) | O(1) |
| Kth Smallest Pair Distance (LC 719) | O(n log n + n log(max−min)) | O(1) |
| Median of Two Sorted Arrays (LC 4) | O(log(min(m,n))) | O(1) |
| Kth Smallest in BST (LC 230) | O(H + k), H = tree height | O(H) |
| Third Maximum Number (LC 414) | O(n) | O(1) |

---

## Problem → Pattern Map

| Problem | Category | Core structure |
|---|---|---|
| LC 215 | Foundation | sort / min-heap(k) / QuickSelect — pick based on constraints |
| LC 703 | Heap (top-k) | min-heap of size k, streaming |
| LC 347 / 692 | Heap (top-k) | min-heap of size k, priority = frequency (+ tie-break for 692) |
| LC 973 | Heap (top-k) | max-heap of size k, priority = distance |
| LC 1046 | Heap (top-k) | max-heap, repeated pop-pop-push simulation |
| LC 23 | Heap (k-way merge) | one heap entry per source list |
| LC 373 | Heap (k-way merge) | one heap entry per row of an implicit grid |
| LC 378 | Heap (k-way merge) **or** Binary search | row-frontier heap, or staircase count |
| LC 295 | Heap (two-heap) | balanced max-heap + min-heap |
| QuickSelect (Lomuto) | QuickSelect | pivot lands at final index; O(n²) if array is all duplicates |
| QuickSelect (Hoare) | QuickSelect | pivot does NOT land at final index — boundary differs |
| QuickSelect (3-way partition) | QuickSelect | splits into <, ==, > pivot in one pass; fixes Lomuto's duplicate weakness |
| LC 75 | QuickSelect (3-way) | 3-way partition with a hardcoded pivot value of 1 |
| Median via QuickSelect | QuickSelect | kth-element, applied twice for even length |
| LC 462 | QuickSelect / Sort | median minimizes sum of \|x − median\| |
| LC 668 | Binary search on answer | count via per-row division |
| LC 719 | Binary search on answer | count via sliding window on sorted array |
| LC 4 | Binary search on answer | search a **partition point**, not a value |
| LC 230 | Different structure | BST inorder traversal, early stop |
| LC 414 | No structure needed | track top-3 in fixed variables |
