# Trees — Advanced Guide for Google L6

---

## Topics Covered

1. **LCA (Lowest Common Ancestor)** — binary trees and BSTs
2. **Serialize / Deserialize** — binary tree to string and back
3. **Morris Traversal** — O(1) space inorder traversal
4. **Tree Construction** — from preorder + inorder, etc.
5. **Binary Search Tree** — validation, search, insert, kth smallest

---

## How to Identify

| Signal | Topic |
|---|---|
| "find ancestor of two nodes" | LCA |
| "encode/decode tree" | Serialize/Deserialize |
| "O(1) space traversal" | Morris |
| "rebuild tree from traversals" | Tree Construction |
| "BST k-th element, range sum" | BST properties |
| "path between two nodes" | LCA + path sum |

---

## How to Think About Tree Problems

```
1. Where does the answer live?
   → At a node: return it upward
   → Along a path: track max/min as you go
   → Across root: left + root + right

2. What do I return from each recursive call?
   → Often: (answer-so-far, value-to-return-upward)

3. Post-order or Pre-order?
   → Post-order: need children's info before processing current node (LCA, diameter)
   → Pre-order: pass info downward (path sum, level-order)

4. Null check first?
   → Almost always: if (root == null) return base;
```

---

---

# LCA — Lowest Common Ancestor

### Core Algorithm (Binary Tree, LC 236)

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) return root;

        TreeNode left  = lowestCommonAncestor(root.left,  p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        if (left != null && right != null) return root;  // p and q on different sides
        return left != null ? left : right;              // both on same side
    }
}
```

**Logic**:
- If root IS p or q → return root (this is a potential ancestor)
- If p and q found in different subtrees → root is LCA
- Otherwise pass non-null result upward

Time: O(n) | Space: O(h)

---

### LCA of BST (LC 235)

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        while (root != null) {
            if (p.val < root.val && q.val < root.val)
                root = root.left;
            else if (p.val > root.val && q.val > root.val)
                root = root.right;
            else
                return root;  // split point or one of them IS the root
        }
        return null;
    }
}
```

**BST property**: if both p and q are smaller → go left; both larger → go right; otherwise root is LCA.

---

### Problem 1: LCA of Binary Tree (LC 236)
Given a binary tree and two nodes `p` and `q`, find their lowest common ancestor (the lowest node that has both as descendants; a node can be a descendant of itself).

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4

p=5, q=4 → Output: 5  (5 is ancestor of 4)
p=5, q=1 → Output: 3  (split at root)
```

(See core algorithm above.)

---

### Problem 2: LCA of BST (LC 235)
Same as LC 236 but on a BST — use BST property to avoid full traversal.

```
BST:    6
       / \
      2   8
     / \ / \
    0  4 7  9
      / \
     3   5

p=2, q=8 → Output: 6
p=2, q=4 → Output: 2
```

(See BST variant above.)

---

### Problem 3: LCA of Deepest Leaves (LC 1123)
Return the LCA of the deepest leaves of a binary tree.

```
      1
     / \
    2   3
   / \
  4   5

Deepest leaves: 4 and 5 (depth 3)
Output: 2  (LCA of 4 and 5)

      1
     / \
    2   3
         \
          4

Deepest leaf: 4 only
Output: 4
```

```java
class Solution {
    public TreeNode lcaDeepestLeaves(TreeNode root) {
        return dfs(root).node;
    }

    int[] dfs2(TreeNode node) { return new int[]{depth(node.left), depth(node.right)}; }

    // returns [lca, depth]
    Pair<TreeNode, Integer> dfs(TreeNode node) {
        if (node == null) return new Pair<>(null, 0);
        Pair<TreeNode, Integer> left  = dfs(node.left);
        Pair<TreeNode, Integer> right = dfs(node.right);
        if (left.getValue() > right.getValue())
            return new Pair<>(left.getKey(), left.getValue() + 1);
        if (right.getValue() > left.getValue())
            return new Pair<>(right.getKey(), right.getValue() + 1);
        return new Pair<>(node, left.getValue() + 1); // equal depths → current node is LCA
    }
}
```

---

### Problem 4: Diameter of Binary Tree (LC 543)
Given a binary tree, return the length of its diameter (longest path between any two nodes, measured in number of edges). The path may not pass through the root.

```
    1
   / \
  2   3
 / \
4   5

Output: 3  → path [4,2,1,3] or [5,2,1,3]

    1
   /
  2

Output: 1
```

```java
class Solution {
    int max = 0;
    public int diameterOfBinaryTree(TreeNode root) {
        dfs(root);
        return max;
    }
    int dfs(TreeNode node) {
        if (node == null) return 0;
        int left = dfs(node.left), right = dfs(node.right);
        max = Math.max(max, left + right);  // path through this node
        return 1 + Math.max(left, right);   // height returned upward
    }
}
```

---

### Problem 5: Binary Tree Maximum Path Sum (LC 124)
Given a binary tree, find the path with the maximum sum. The path can start and end at any node (does not need to go through root).

```
    1
   / \
  2   3

Output: 6  → path 2→1→3

   -10
   /  \
  9   20
     /  \
    15   7

Output: 42  → path 15→20→7
```

```java
class Solution {
    int max = Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        dfs(root);
        return max;
    }
    int dfs(TreeNode node) {
        if (node == null) return 0;
        int left  = Math.max(0, dfs(node.left));   // ignore negative paths
        int right = Math.max(0, dfs(node.right));
        max = Math.max(max, left + node.val + right);
        return node.val + Math.max(left, right);    // only one side returned upward
    }
}
```

---

---

# Serialize / Deserialize

### Core Algorithm (LC 297)

**Key insight**: Preorder traversal with null markers lets us reconstruct exactly.

```java
public class Codec {
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        serializeHelper(root, sb);
        return sb.toString();
    }

    private void serializeHelper(TreeNode node, StringBuilder sb) {
        if (node == null) { sb.append("null,"); return; }
        sb.append(node.val).append(",");
        serializeHelper(node.left, sb);
        serializeHelper(node.right, sb);
    }

    public TreeNode deserialize(String data) {
        Queue<String> queue = new LinkedList<>(Arrays.asList(data.split(",")));
        return deserializeHelper(queue);
    }

    private TreeNode deserializeHelper(Queue<String> queue) {
        String val = queue.poll();
        if (val.equals("null")) return null;
        TreeNode node = new TreeNode(Integer.parseInt(val));
        node.left  = deserializeHelper(queue);
        node.right = deserializeHelper(queue);
        return node;
    }
}
```

**Preorder** works because we process root before children → we know what's "current" before building subtrees.

---

### Problem 6: Serialize and Deserialize Binary Tree (LC 297)
Design an algorithm to serialize a binary tree to a string and deserialize that string back to the original tree.

```
    1
   / \
  2   3
     / \
    4   5

Serialize  → "1,2,null,null,3,4,null,null,5,null,null"
Deserialize → restores original tree
```

(See core algorithm above.)

---

### Problem 7: Serialize and Deserialize BST (LC 449)
Same as LC 297 but for a BST — can use a more compact encoding since BST structure is recoverable from preorder traversal alone (no null markers needed).

```
BST:  2
     / \
    1   3

Serialize  → "2,1,3"  (no nulls needed!)
Deserialize → use BST bounds to reconstruct
```

```java
public class Codec {
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        preorder(root, sb);
        return sb.toString();
    }

    private void preorder(TreeNode node, StringBuilder sb) {
        if (node == null) return;
        sb.append(node.val).append(",");
        preorder(node.left, sb);
        preorder(node.right, sb);
    }

    public TreeNode deserialize(String data) {
        if (data.isEmpty()) return null;
        int[] vals = Arrays.stream(data.split(",")).mapToInt(Integer::parseInt).toArray();
        return build(vals, new int[]{0}, Integer.MIN_VALUE, Integer.MAX_VALUE);
    }

    private TreeNode build(int[] vals, int[] idx, int min, int max) {
        if (idx[0] == vals.length || vals[idx[0]] < min || vals[idx[0]] > max) return null;
        int val = vals[idx[0]++];
        TreeNode node = new TreeNode(val);
        node.left  = build(vals, idx, min, val);
        node.right = build(vals, idx, val, max);
        return node;
    }
}
```

---

### Problem 8: Find Duplicate Subtrees (LC 652)
Given the root of a binary tree, return all duplicate subtrees (same structure and node values). Return the root of each duplicate subtree (only one copy per duplicate group).

```
        1
       / \
      2   3
     /   / \
    4   2   4
       /
      4

Output: [[2,4],[4]]  → subtrees rooted at 2 and 4 each appear twice
```

```java
class Solution {
    Map<String, Integer> count = new HashMap<>();
    List<TreeNode> result = new ArrayList<>();

    public List<TreeNode> findDuplicateSubtrees(TreeNode root) {
        serialize(root);
        return result;
    }

    private String serialize(TreeNode node) {
        if (node == null) return "#";
        String serial = node.val + "," + serialize(node.left) + "," + serialize(node.right);
        count.merge(serial, 1, Integer::sum);
        if (count.get(serial) == 2) result.add(node);
        return serial;
    }
}
```

---

---

# Tree Construction

### Problem 9: Construct from Preorder + Inorder (LC 105)
Given `preorder` and `inorder` traversal arrays of a binary tree, construct and return the binary tree.

```
preorder = [3,9,20,15,7]
inorder  = [9,3,15,20,7]

Output:
    3
   / \
  9  20
    /  \
   15   7

Key: preorder[0]=3 is root; in inorder, 3 splits [9] (left) and [15,20,7] (right)
```

```java
class Solution {
    Map<Integer, Integer> inMap = new HashMap<>();
    int[] preorder;

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        this.preorder = preorder;
        for (int i = 0; i < inorder.length; i++) inMap.put(inorder[i], i);
        return build(0, 0, inorder.length - 1);
    }

    private TreeNode build(int preIdx, int inLeft, int inRight) {
        if (inLeft > inRight) return null;
        int rootVal = preorder[preIdx];
        TreeNode root = new TreeNode(rootVal);
        int inIdx = inMap.get(rootVal);
        int leftSize = inIdx - inLeft;

        root.left  = build(preIdx + 1, inLeft, inIdx - 1);
        root.right = build(preIdx + leftSize + 1, inIdx + 1, inRight);
        return root;
    }
}
```

**Key**: in preorder, root comes first; in inorder, root splits left and right subtrees.

---

### Problem 10: Construct from Inorder + Postorder (LC 106)
Given `inorder` and `postorder` traversal arrays, construct and return the binary tree.

```
inorder   = [9,3,15,20,7]
postorder = [9,15,7,20,3]

Output:
    3
   / \
  9  20
    /  \
   15   7

Key: postorder last element = root; find in inorder to split left/right
```

```java
class Solution {
    Map<Integer, Integer> inMap = new HashMap<>();
    int[] postorder;
    int postIdx;

    public TreeNode buildTree(int[] inorder, int[] postorder) {
        this.postorder = postorder;
        this.postIdx = postorder.length - 1;
        for (int i = 0; i < inorder.length; i++) inMap.put(inorder[i], i);
        return build(0, inorder.length - 1);
    }

    private TreeNode build(int inLeft, int inRight) {
        if (inLeft > inRight) return null;
        int rootVal = postorder[postIdx--];
        TreeNode root = new TreeNode(rootVal);
        int inIdx = inMap.get(rootVal);

        root.right = build(inIdx + 1, inRight);  // right BEFORE left (postorder reversed)
        root.left  = build(inLeft, inIdx - 1);
        return root;
    }
}
```

---

---

# BST Problems

### Problem 11: Validate BST (LC 98)
Given the root of a binary tree, determine if it is a valid binary search tree (left subtree strictly less, right subtree strictly greater, for all nodes).

```
    2
   / \
  1   3
→ true

    5
   / \
  1   4
     / \
    3   6
→ false  (4 is in right subtree of 5 but 4 < 5)
```

```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }
    boolean validate(TreeNode node, long min, long max) {
        if (node == null) return true;
        if (node.val <= min || node.val >= max) return false;
        return validate(node.left, min, node.val) && validate(node.right, node.val, max);
    }
}
```

---

### Problem 12: Kth Smallest Element in BST (LC 230)
Given the root of a BST and integer `k`, return the kth smallest value (1-indexed).

```
BST:  3
     / \
    1   4
     \
      2

k=1 → Output: 1
k=3 → Output: 3

Inorder traversal of BST gives sorted order: [1,2,3,4]
```

```java
class Solution {
    int count = 0, result = 0;
    public int kthSmallest(TreeNode root, int k) {
        inorder(root, k);
        return result;
    }
    void inorder(TreeNode node, int k) {
        if (node == null) return;
        inorder(node.left, k);
        if (++count == k) { result = node.val; return; }
        inorder(node.right, k);
    }
}
```

---

### Problem 13: Range Sum of BST (LC 938)
Given the root of a BST and integers `low` and `high`, return the sum of values of all nodes with values in the range `[low, high]`.

```
BST:    10
       /  \
      5   15
     / \    \
    3   7   18

low=7, high=15 → Output: 32  → 7+10+15=32
low=6, high=10 → Output: 17  → 7+10=17
```

```java
class Solution {
    public int rangeSumBST(TreeNode root, int low, int high) {
        if (root == null) return 0;
        int sum = 0;
        if (root.val >= low && root.val <= high) sum += root.val;
        if (root.val > low)  sum += rangeSumBST(root.left,  low, high);
        if (root.val < high) sum += rangeSumBST(root.right, low, high);
        return sum;
    }
}
```

---

---

# Morris Traversal (O(1) Space Inorder)

### Algorithm Explanation

```
Normal inorder uses O(h) stack space for recursion.
Morris Traversal: use the right pointers of nodes that have no right child 
as "threads" to return to the parent → O(1) extra space.

For each node:
  1. If no left child: visit current, move to right
  2. If left child exists:
     a. Find inorder predecessor (rightmost node of left subtree)
     b. If predecessor.right == null: thread it to current, go left
     c. If predecessor.right == current: remove thread, visit current, go right
```

### Problem 14: Binary Tree Inorder Traversal (Morris, LC 94)
Return inorder traversal of a binary tree. Solve it with O(1) extra space (no recursion or explicit stack) using Morris Traversal.

```
    1
     \
      2
     /
    3

Output: [1,3,2]

    4
   / \
  2   5
 / \
1   3

Output: [1,2,3,4,5]
```

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        TreeNode curr = root;
        while (curr != null) {
            if (curr.left == null) {
                result.add(curr.val);  // no left → visit
                curr = curr.right;
            } else {
                // find inorder predecessor
                TreeNode prev = curr.left;
                while (prev.right != null && prev.right != curr)
                    prev = prev.right;

                if (prev.right == null) {
                    prev.right = curr;  // thread
                    curr = curr.left;
                } else {
                    prev.right = null;  // remove thread
                    result.add(curr.val);
                    curr = curr.right;
                }
            }
        }
        return result;
    }
}
```

Time: O(n) — each edge traversed at most twice | Space: O(1)

---

### Problem 15: Recover BST (LC 99)
Two nodes of a BST are swapped by mistake. Recover the tree without changing its structure.

```
Incorrect BST:
    3
   / \
  1   4
     /
    2

Correct BST:
    2
   / \
  1   4
     /
    3

Nodes 2 and 3 were swapped. Inorder of incorrect: [1,3,2,4] — spot the two inversions.
```

```java
class Solution {
    TreeNode first = null, second = null, prev = null;

    public void recoverTree(TreeNode root) {
        inorder(root);
        int tmp = first.val; first.val = second.val; second.val = tmp;
    }

    void inorder(TreeNode node) {
        if (node == null) return;
        inorder(node.left);
        if (prev != null && prev.val > node.val) {
            if (first == null) first = prev;
            second = node;
        }
        prev = node;
        inorder(node.right);
    }
}
```

---

---

# Common Pitfalls

## 1. LCA — What to Return When Node Not Found

```java
// If only p exists in tree, still returns p
// That's OK — LCA guarantees both exist per problem statement
if (root == null || root == p || root == q) return root;
```

---

## 2. Diameter vs Path Sum — What to Return Upward

```java
// Diameter: return HEIGHT upward, track max diameter globally
return 1 + Math.max(left, right);

// Max Path Sum: return max gain (one side only) upward
return node.val + Math.max(left, right);
// Can't go down both sides and return upward!
```

---

## 3. Serialize — Order Matters for Deserialize

```java
// Serialize: preorder (root, left, right)
// Deserialize: must consume in same order
// BFS serialize: level-by-level, use queue
```

---

## 4. BST Validation — Use Long Boundaries

```java
// Integer.MIN_VALUE node.val will fail with int bounds
validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
```

---

## 5. Construct Tree — Right Before Left in Postorder

```java
// In postorder: last element is root
// Process right subtree before left (since we consume from end)
root.right = build(inIdx + 1, inRight);
root.left  = build(inLeft, inIdx - 1);
```

---

---

# Quick Reference Card

## Problem → Algorithm Map

| Problem | Algorithm | Key insight |
|---|---|---|
| LCA Binary Tree | Post-order DFS | Return node when found; merge at split |
| LCA BST | Iterative | BST property narrows direction |
| Diameter | Post-order DFS | Height up, diameter = left+right |
| Max Path Sum | Post-order DFS | Gain = max(0, subtree); path = left+val+right |
| Serialize | Preorder + null | Unique reconstruction from preorder+nulls |
| Construct (pre+in) | Preorder root + inorder split | inMap for O(1) lookup |
| Kth Smallest BST | Inorder (BST sorted) | Count during inorder |
| Validate BST | Bounds passing | Pass (min, max) down |
| Morris Inorder | Thread right pointers | O(1) space |

## Return Value Patterns

```
Height          → 1 + max(left, right)
Path through    → left + val + right (can't return this, only one branch up)
Count of nodes  → 1 + left + right
Is balanced?    → return -1 for unbalanced, height otherwise
LCA             → return node if found, null if not
```
