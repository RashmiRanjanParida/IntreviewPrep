# Top 30 Linked List Problems — Java Solutions

## Node Definition (used throughout)
```java
public class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}
```

---

## Category 1: Reverse Problems

### L1. Reverse Linked List (LC 206) — Easy
Reverse a singly linked list.

**Example:** `1→2→3→4→5` → `5→4→3→2→1`

**Key Insight:** Keep track of `prev`, `curr`, `next`. Iterative is O(1) space; recursive is O(n) stack space.

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L2. Reverse Linked List II (LC 92) — Medium
Reverse the sublist from position `left` to `right`.

**Example:** `1→2→3→4→5`, left=2, right=4 → `1→4→3→2→5`

**Key Insight:** Use a dummy node. Walk to `(left-1)`th node, then reverse the next `(right-left+1)` nodes in place by repeatedly moving `curr.next` to just after `prev`.

```java
public ListNode reverseBetween(ListNode head, int left, int right) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode prev = dummy;

    for (int i = 1; i < left; i++) prev = prev.next;

    ListNode curr = prev.next;
    for (int i = 0; i < right - left; i++) {
        ListNode next = curr.next;
        curr.next = next.next;
        next.next = prev.next;
        prev.next = next;
    }
    return dummy.next;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L3. Reverse Nodes in k-Group (LC 25) — Hard
Reverse every k consecutive nodes. Leave remaining nodes as-is if fewer than k.

**Example:** `1→2→3→4→5`, k=2 → `2→1→4→3→5`

**Key Insight:** Check if k nodes remain before reversing. Use a dummy node and track the tail of the previous group to reconnect after each reversal.

```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode groupPrev = dummy;

    while (true) {
        ListNode kth = getKth(groupPrev, k);
        if (kth == null) break;

        ListNode groupNext = kth.next;
        ListNode prev = groupNext, curr = groupPrev.next;
        while (curr != groupNext) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        ListNode tmp = groupPrev.next;
        groupPrev.next = kth;
        groupPrev = tmp;
    }
    return dummy.next;
}

private ListNode getKth(ListNode node, int k) {
    while (node != null && k > 0) { node = node.next; k--; }
    return node;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L4. Swap Nodes in Pairs (LC 24) — Medium
Swap every two adjacent nodes (k=2 special case of LC 25).

**Example:** `1→2→3→4` → `2→1→4→3`

**Key Insight:** Dummy node + track the node before each pair. Reconnect: `prev → second → first → next pair`.

```java
public ListNode swapPairs(ListNode head) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode prev = dummy;

    while (prev.next != null && prev.next.next != null) {
        ListNode first = prev.next;
        ListNode second = prev.next.next;
        first.next = second.next;
        second.next = first;
        prev.next = second;
        prev = first;
    }
    return dummy.next;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

## Category 2: Fast & Slow Pointers

### L5. Middle of Linked List (LC 876) — Easy
Find the middle node. If two middles, return the second.

**Example:** `1→2→3→4→5` → node `3`; `1→2→3→4` → node `3`

**Key Insight:** Fast pointer moves 2 steps, slow moves 1. When fast reaches end, slow is at middle.

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L6. Linked List Cycle (LC 141) — Easy
Detect if a cycle exists.

**Key Insight:** Floyd's algorithm — fast and slow pointers eventually meet inside the cycle if one exists.

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L7. Linked List Cycle II (LC 142) — Medium
Find the node where the cycle begins.

**Example:** `3→1→2→` (cycle back to 1) → return node `1`

**Key Insight:** After fast/slow meet, reset one pointer to `head`. Move both one step at a time — they meet at the cycle entry. Mathematical proof: distance from head to cycle entry equals distance from meeting point to cycle entry.

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            slow = head;
            while (slow != fast) {
                slow = slow.next;
                fast = fast.next;
            }
            return slow;
        }
    }
    return null;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L8. Remove Nth Node From End (LC 19) — Medium
Remove the nth node from the end in one pass.

**Example:** `1→2→3→4→5`, n=2 → `1→2→3→5`

**Key Insight:** Two pointers — advance fast by n steps first. Then move both until fast reaches end. Slow is now at the node before the target.

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode fast = dummy, slow = dummy;

    for (int i = 0; i <= n; i++) fast = fast.next;

    while (fast != null) {
        slow = slow.next;
        fast = fast.next;
    }
    slow.next = slow.next.next;
    return dummy.next;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L9. Palindrome Linked List (LC 234) — Easy
Check if a linked list is a palindrome.

**Example:** `1→2→2→1` → true; `1→2` → false

**Key Insight:** Find middle, reverse second half, compare with first half, restore (optional).

```java
public boolean isPalindrome(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    // reverse second half
    ListNode prev = null, curr = slow;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    // compare
    ListNode left = head, right = prev;
    while (right != null) {
        if (left.val != right.val) return false;
        left = left.next;
        right = right.next;
    }
    return true;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L10. Intersection of Two Linked Lists (LC 160) — Easy
Find the node at which two lists intersect.

**Key Insight:** Two pointers — when one reaches the end, redirect it to the other list's head. They meet at the intersection after traversing `lenA + lenB` total steps each.

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode a = headA, b = headB;
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    return a;
}
```
**Complexity:** Time: O(m+n) | Space: O(1)

---

## Category 3: Merge & Sort

### L11. Merge Two Sorted Lists (LC 21) — Easy
Merge two sorted linked lists into one sorted list.

**Example:** `1→2→4` and `1→3→4` → `1→1→2→3→4→4`

**Key Insight:** Dummy head simplifies edge cases. Always attach the smaller node.

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), curr = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else                  { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```
**Complexity:** Time: O(m+n) | Space: O(1)

---

### L12. Merge K Sorted Lists (LC 23) — Hard
Merge k sorted linked lists into one sorted list.

**Example:** `[1→4→5, 1→3→4, 2→6]` → `1→1→2→3→4→4→5→6`

**Key Insight:** Min-heap of size k — always extract the minimum node, then push that node's next into the heap.

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>(Comparator.comparingInt(n -> n.val));
    for (ListNode node : lists) if (node != null) pq.offer(node);

    ListNode dummy = new ListNode(0), curr = dummy;
    while (!pq.isEmpty()) {
        ListNode node = pq.poll();
        curr.next = node;
        curr = curr.next;
        if (node.next != null) pq.offer(node.next);
    }
    return dummy.next;
}
```
**Complexity:** Time: O(n log k) | Space: O(k)

---

### L13. Sort List (LC 148) — Medium
Sort a linked list in O(n log n) time and O(1) space.

**Key Insight:** Bottom-up merge sort on linked list avoids O(log n) recursion stack. Find middle, split, merge sorted halves.

```java
public ListNode sortList(ListNode head) {
    if (head == null || head.next == null) return head;

    ListNode mid = getMid(head);
    ListNode right = mid.next;
    mid.next = null;

    return merge(sortList(head), sortList(right));
}

private ListNode getMid(ListNode head) {
    ListNode slow = head, fast = head.next;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}

private ListNode merge(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), curr = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else                  { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```
**Complexity:** Time: O(n log n) | Space: O(log n) recursion stack (O(1) for iterative bottom-up)

---

### L14. Reorder List (LC 143) — Medium
Reorder `L0→L1→...→Ln` to `L0→Ln→L1→Ln-1→L2→Ln-2→...`

**Example:** `1→2→3→4→5` → `1→5→2→4→3`

**Key Insight:** Three steps — find middle, reverse second half, merge two halves alternately.

```java
public void reorderList(ListNode head) {
    // 1. find middle
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    // 2. reverse second half
    ListNode prev = null, curr = slow.next;
    slow.next = null;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    // 3. merge alternately
    ListNode first = head, second = prev;
    while (second != null) {
        ListNode tmp1 = first.next, tmp2 = second.next;
        first.next = second;
        second.next = tmp1;
        first = tmp1;
        second = tmp2;
    }
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

## Category 4: Remove / Delete Nodes

### L15. Remove Linked List Elements (LC 203) — Easy
Remove all nodes with value equal to `val`.

**Example:** `1→2→6→3→4→5→6`, val=6 → `1→2→3→4→5`

**Key Insight:** Dummy head handles the case where head itself needs removal.

```java
public ListNode removeElements(ListNode head, int val) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode curr = dummy;
    while (curr.next != null) {
        if (curr.next.val == val) curr.next = curr.next.next;
        else curr = curr.next;
    }
    return dummy.next;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L16. Remove Duplicates from Sorted List (LC 83) — Easy
Remove all duplicates, keep one copy.

**Example:** `1→1→2→3→3` → `1→2→3`

```java
public ListNode deleteDuplicates(ListNode head) {
    ListNode curr = head;
    while (curr != null && curr.next != null) {
        if (curr.val == curr.next.val) curr.next = curr.next.next;
        else curr = curr.next;
    }
    return head;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L17. Remove Duplicates from Sorted List II (LC 82) — Medium
Remove ALL nodes that have duplicates (keep none).

**Example:** `1→2→3→3→4→4→5` → `1→2→5`

**Key Insight:** Dummy head + check if `curr.next` starts a run of duplicates. Skip the entire run.

```java
public ListNode deleteDuplicates(ListNode head) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode prev = dummy;

    while (prev.next != null) {
        ListNode curr = prev.next;
        if (curr.next != null && curr.val == curr.next.val) {
            int dupVal = curr.val;
            while (prev.next != null && prev.next.val == dupVal)
                prev.next = prev.next.next;
        } else {
            prev = prev.next;
        }
    }
    return dummy.next;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L18. Delete Node in a Linked List (LC 237) — Medium
Delete a node given only access to that node (not head).

**Key Insight:** Copy the value of the next node into the current node, then skip the next node. You can't truly delete `node` itself, so delete `node.next` instead.

```java
public void deleteNode(ListNode node) {
    node.val = node.next.val;
    node.next = node.next.next;
}
```
**Complexity:** Time: O(1) | Space: O(1)

---

## Category 5: Partition & Rearrange

### L19. Partition List (LC 86) — Medium
Partition list so all nodes with val < x come before nodes with val >= x.

**Example:** `1→4→3→2→5→2`, x=3 → `1→2→2→4→3→5`

**Key Insight:** Two separate lists — `less` for nodes < x, `greater` for nodes >= x. Merge at end. Relative order preserved.

```java
public ListNode partition(ListNode head, int x) {
    ListNode lessHead = new ListNode(0), greaterHead = new ListNode(0);
    ListNode less = lessHead, greater = greaterHead;

    while (head != null) {
        if (head.val < x) { less.next = head; less = less.next; }
        else              { greater.next = head; greater = greater.next; }
        head = head.next;
    }
    greater.next = null;
    less.next = greaterHead.next;
    return lessHead.next;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L20. Odd Even Linked List (LC 328) — Medium
Group all odd-indexed nodes first, then even-indexed nodes.

**Example:** `1→2→3→4→5` → `1→3→5→2→4`

**Key Insight:** Two pointers for odd and even chains. Connect even chain's head to the start of odd chain at the end.

```java
public ListNode oddEvenList(ListNode head) {
    if (head == null) return null;
    ListNode odd = head, even = head.next, evenHead = even;
    while (even != null && even.next != null) {
        odd.next = even.next;
        odd = odd.next;
        even.next = odd.next;
        even = even.next;
    }
    odd.next = evenHead;
    return head;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L21. Rotate List (LC 61) — Medium
Rotate list to the right by k places.

**Example:** `1→2→3→4→5`, k=2 → `4→5→1→2→3`

**Key Insight:** Form a circular list, then break it at position `n - k % n - 1`.

```java
public ListNode rotateRight(ListNode head, int k) {
    if (head == null || head.next == null || k == 0) return head;

    ListNode tail = head;
    int n = 1;
    while (tail.next != null) { tail = tail.next; n++; }

    int steps = n - k % n;
    if (steps == n) return head;

    tail.next = head; // make circular
    ListNode newTail = head;
    for (int i = 1; i < steps; i++) newTail = newTail.next;

    ListNode newHead = newTail.next;
    newTail.next = null;
    return newHead;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L22. Split Linked List in Parts (LC 725) — Medium
Split list into k consecutive parts as equally as possible.

**Example:** `1→2→3→4→5→6→7→8→9→10`, k=3 → `[1→2→3→4, 5→6→7, 8→9→10]`

**Key Insight:** `base = n/k` nodes per part, first `n%k` parts get one extra.

```java
public ListNode[] splitListToParts(ListNode head, int k) {
    int n = 0;
    ListNode curr = head;
    while (curr != null) { n++; curr = curr.next; }

    int base = n / k, extra = n % k;
    ListNode[] result = new ListNode[k];
    curr = head;
    for (int i = 0; i < k && curr != null; i++) {
        result[i] = curr;
        int size = base + (i < extra ? 1 : 0);
        for (int j = 1; j < size; j++) curr = curr.next;
        ListNode next = curr.next;
        curr.next = null;
        curr = next;
    }
    return result;
}
```
**Complexity:** Time: O(n + k) | Space: O(k)

---

## Category 6: Math on Linked Lists

### L23. Add Two Numbers (LC 2) — Medium
Two numbers stored in reverse order as linked lists. Return their sum as a linked list.

**Example:** `(2→4→3) + (5→6→4)` → `7→0→8` (342 + 465 = 807)

**Key Insight:** Simulate addition digit by digit with a carry. Process until both lists are exhausted and carry is 0.

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), curr = dummy;
    int carry = 0;
    while (l1 != null || l2 != null || carry != 0) {
        int sum = carry;
        if (l1 != null) { sum += l1.val; l1 = l1.next; }
        if (l2 != null) { sum += l2.val; l2 = l2.next; }
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
    }
    return dummy.next;
}
```
**Complexity:** Time: O(max(m,n)) | Space: O(max(m,n))

---

### L24. Add Two Numbers II (LC 445) — Medium
Numbers stored in forward order (most significant digit first).

**Example:** `(7→2→4→3) + (5→6→4)` → `7→8→0→7` (7243 + 564 = 7807)

**Key Insight:** Use two stacks to reverse the digit order, then add from least significant digit. Or reverse both lists, add, reverse result.

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    Deque<Integer> s1 = new ArrayDeque<>(), s2 = new ArrayDeque<>();
    while (l1 != null) { s1.push(l1.val); l1 = l1.next; }
    while (l2 != null) { s2.push(l2.val); l2 = l2.next; }

    int carry = 0;
    ListNode curr = null;
    while (!s1.isEmpty() || !s2.isEmpty() || carry != 0) {
        int sum = carry;
        if (!s1.isEmpty()) sum += s1.pop();
        if (!s2.isEmpty()) sum += s2.pop();
        carry = sum / 10;
        ListNode node = new ListNode(sum % 10);
        node.next = curr;
        curr = node;
    }
    return curr;
}
```
**Complexity:** Time: O(m+n) | Space: O(m+n)

---

### L25. Convert Binary Number in Linked List to Integer (LC 1290) — Easy
Binary number represented as linked list (MSB first). Convert to decimal.

**Example:** `1→0→1` → 5

**Key Insight:** Shift result left and OR with current bit.

```java
public int getDecimalValue(ListNode head) {
    int result = 0;
    while (head != null) {
        result = result * 2 + head.val;
        head = head.next;
    }
    return result;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

## Category 7: Copy & Clone

### L26. Copy List with Random Pointer (LC 138) — Medium
Each node has `next` and `random` pointer. Deep copy the list.

**Key Insight:** Three passes — (1) interleave cloned nodes: `A→A'→B→B'→C→C'`, (2) set random pointers: `A'.random = A.random.next`, (3) separate the two lists.

```java
public Node copyRandomList(Node head) {
    if (head == null) return null;

    // Step 1: interleave clones
    Node curr = head;
    while (curr != null) {
        Node clone = new Node(curr.val);
        clone.next = curr.next;
        curr.next = clone;
        curr = clone.next;
    }
    // Step 2: set random pointers
    curr = head;
    while (curr != null) {
        if (curr.random != null)
            curr.next.random = curr.random.next;
        curr = curr.next.next;
    }
    // Step 3: separate lists
    Node dummy = new Node(0), cloneCurr = dummy;
    curr = head;
    while (curr != null) {
        cloneCurr.next = curr.next;
        curr.next = curr.next.next;
        cloneCurr = cloneCurr.next;
        curr = curr.next;
    }
    return dummy.next;
}
```
**Complexity:** Time: O(n) | Space: O(1) (not counting output)

---

## Category 8: Advanced

### L27. Flatten a Multilevel Doubly Linked List (LC 430) — Medium
Flatten a doubly linked list where nodes may have a `child` pointer to another doubly linked list.

**Key Insight:** When a node has a child, find the tail of the child list, then connect: `node ↔ child_head ... child_tail ↔ node.next`.

```java
public Node flatten(Node head) {
    Node curr = head;
    while (curr != null) {
        if (curr.child != null) {
            Node child = curr.child;
            Node next = curr.next;

            curr.next = child;
            child.prev = curr;
            curr.child = null;

            Node tail = child;
            while (tail.next != null) tail = tail.next;

            tail.next = next;
            if (next != null) next.prev = tail;
        }
        curr = curr.next;
    }
    return head;
}
```
**Complexity:** Time: O(n) | Space: O(1)

---

### L28. Convert Sorted List to Binary Search Tree (LC 109) — Medium
Convert a sorted linked list to a height-balanced BST.

**Key Insight:** Find the middle (slow/fast pointers) — it becomes the root. Recursively build left subtree from left half and right subtree from right half.

```java
public TreeNode sortedListToBST(ListNode head) {
    if (head == null) return null;
    if (head.next == null) return new TreeNode(head.val);

    ListNode prev = null, slow = head, fast = head;
    while (fast != null && fast.next != null) {
        prev = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    prev.next = null; // split left half

    TreeNode root = new TreeNode(slow.val);
    root.left = sortedListToBST(head);
    root.right = sortedListToBST(slow.next);
    return root;
}
```
**Complexity:** Time: O(n log n) | Space: O(log n)

---

### L29. LRU Cache (LC 146) — Medium
Design a data structure with O(1) get and put.

**Key Insight:** HashMap + Doubly Linked List. Map stores key→node for O(1) lookup. DLL maintains access order — MRU at head, LRU at tail. Sentinel dummy nodes simplify edge cases.

```java
class LRUCache {
    private static class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }

    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0); // dummy
    private final Node tail = new Node(0, 0); // dummy

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        remove(node);
        addToHead(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) remove(map.get(key));
        Node node = new Node(key, value);
        addToHead(node);
        map.put(key, node);
        if (map.size() > capacity) {
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);
        }
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void addToHead(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
```
**Complexity:** Time: O(1) get and put | Space: O(capacity)

---

### L30. Design Linked List (LC 707) — Medium
Implement a singly/doubly linked list with: `get(index)`, `addAtHead`, `addAtTail`, `addAtIndex`, `deleteAtIndex`.

**Key Insight:** Use a dummy head node so all insertions and deletions work uniformly (no special-casing for index 0).

```java
class MyLinkedList {
    private static class Node {
        int val;
        Node next;
        Node(int val) { this.val = val; }
    }

    private Node dummy = new Node(0);
    private int size = 0;

    public int get(int index) {
        if (index < 0 || index >= size) return -1;
        Node curr = dummy.next;
        for (int i = 0; i < index; i++) curr = curr.next;
        return curr.val;
    }

    public void addAtHead(int val) { addAtIndex(0, val); }
    public void addAtTail(int val) { addAtIndex(size, val); }

    public void addAtIndex(int index, int val) {
        if (index < 0 || index > size) return;
        Node prev = dummy;
        for (int i = 0; i < index; i++) prev = prev.next;
        Node node = new Node(val);
        node.next = prev.next;
        prev.next = node;
        size++;
    }

    public void deleteAtIndex(int index) {
        if (index < 0 || index >= size) return;
        Node prev = dummy;
        for (int i = 0; i < index; i++) prev = prev.next;
        prev.next = prev.next.next;
        size--;
    }
}
```
**Complexity:** Time: O(n) per operation | Space: O(n)

---

## Quick Reference

| # | Problem | Pattern | Time | Space |
|---|---------|---------|------|-------|
| L1 | Reverse Linked List | Iterative reverse | O(n) | O(1) |
| L2 | Reverse Linked List II | Dummy + in-place reverse | O(n) | O(1) |
| L3 | Reverse Nodes in k-Group | Group pointer + reverse | O(n) | O(1) |
| L4 | Swap Nodes in Pairs | Dummy + pair swap | O(n) | O(1) |
| L5 | Middle of Linked List | Fast/slow pointers | O(n) | O(1) |
| L6 | Linked List Cycle | Floyd's algorithm | O(n) | O(1) |
| L7 | Linked List Cycle II | Floyd's + reset | O(n) | O(1) |
| L8 | Remove Nth from End | Two pointers, n-gap | O(n) | O(1) |
| L9 | Palindrome Linked List | Find mid + reverse half | O(n) | O(1) |
| L10 | Intersection of Lists | Two pointer cross-walk | O(m+n) | O(1) |
| L11 | Merge Two Sorted Lists | Dummy + merge | O(m+n) | O(1) |
| L12 | Merge K Sorted Lists | Min-heap | O(n log k) | O(k) |
| L13 | Sort List | Merge sort | O(n log n) | O(log n) |
| L14 | Reorder List | Mid + reverse + merge | O(n) | O(1) |
| L15 | Remove Elements | Dummy + skip | O(n) | O(1) |
| L16 | Remove Duplicates I | Overwrite next | O(n) | O(1) |
| L17 | Remove Duplicates II | Dummy + skip runs | O(n) | O(1) |
| L18 | Delete Node | Copy-then-skip | O(1) | O(1) |
| L19 | Partition List | Two chains | O(n) | O(1) |
| L20 | Odd Even List | Odd/even chains | O(n) | O(1) |
| L21 | Rotate List | Circular + break | O(n) | O(1) |
| L22 | Split List in Parts | n/k + extras | O(n+k) | O(k) |
| L23 | Add Two Numbers | Digit sim + carry | O(max(m,n)) | O(max(m,n)) |
| L24 | Add Two Numbers II | Stacks + carry | O(m+n) | O(m+n) |
| L25 | Binary List to Int | Shift + OR | O(n) | O(1) |
| L26 | Copy List Random Ptr | Interleave + separate | O(n) | O(1) |
| L27 | Flatten Multilevel DLL | Child → inline | O(n) | O(1) |
| L28 | Sorted List to BST | Mid as root, recurse | O(n log n) | O(log n) |
| L29 | LRU Cache | HashMap + DLL | O(1) | O(capacity) |
| L30 | Design Linked List | Dummy head | O(n) | O(n) |

---

## Key Patterns Summary

| Pattern | When to use |
|---------|-------------|
| **Dummy head** | Any insertion/deletion that might affect the head node |
| **Fast/slow pointers** | Find middle, detect cycle, find cycle start, nth from end |
| **Two pointer cross-walk** | Intersection of two lists (redirect to other list's head) |
| **Reverse in place** | Reverse whole or partial list, palindrome check, reorder |
| **Two chains** | Partition, odd/even separation, split |
| **Stack** | Forward-order operations (add two numbers II, palindrome) |
| **Min-heap** | Merge k sorted lists |
| **Interleave + separate** | Deep copy with random pointers |
