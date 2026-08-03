# Tries — Complete Guide for Google L6

---

## What is a Trie?

A **Trie** (prefix tree) is a tree where each path from root to a node spells out a string prefix.

```
Insert: "apple", "app", "bat"

        root
       /    \
      a      b
      |      |
      p      a
      |      |
      p*     t*
      |
      l
      |
      e*

* = isEnd marker
```

**Why Trie over HashMap?**
- Prefix queries: "all words starting with 'app'" → O(prefix length), not O(total strings)
- Shared prefix storage: "apple", "apply" share "appl" nodes

---

## When to Use a Trie

| Signal | Pattern |
|---|---|
| "search words with prefix" | Standard trie |
| "autocomplete / suggestions" | Trie with DFS to collect |
| "wildcard search (. matches any)" | Trie DFS with branching |
| "find words in 2D board" | Trie + DFS on grid |
| "max XOR of two numbers" | Binary trie |
| "count words with prefix" | Trie with count at each node |

---

## Trie Node Structure

```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd = false;
}
```

For more general alphabets:
```java
class TrieNode {
    Map<Character, TrieNode> children = new HashMap<>();
    boolean isEnd = false;
}
```

---

## Core Operations — Template

```java
class Trie {
    TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null)
                curr.children[idx] = new TrieNode();
            curr = curr.children[idx];
        }
        curr.isEnd = true;
    }

    public boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isEnd;
    }

    public boolean startsWith(String prefix) {
        return find(prefix) != null;
    }

    private TrieNode find(String s) {
        TrieNode curr = root;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) return null;
            curr = curr.children[idx];
        }
        return curr;
    }
}
```

---

## How to Think About Trie Problems

```
1. What alphabet? 26 lowercase? digits? any char?
   → 26 lowercase → TrieNode[26]
   → arbitrary → HashMap

2. What do I store at each node?
   → isEnd (search)
   → count (prefix count)
   → word itself (Word Search II — avoids string rebuilding)

3. Do I need to traverse multiple branches?
   → Wildcard ('.') → try all 26 children
   → Find all with prefix → DFS from prefix end node

4. Grid + Trie?
   → Build trie from word list
   → DFS each cell, walk trie simultaneously
```

---

---

# Problems

### 1. Implement Trie (LC 208)
Implement a trie with `insert(word)`, `search(word)` (exact match), and `startsWith(prefix)` operations.

```
trie.insert("apple")
trie.search("apple")   → true
trie.search("app")     → false  (not inserted as a word)
trie.startsWith("app") → true
trie.insert("app")
trie.search("app")     → true
```

```java
class Trie {
    class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd;
    }
    TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int i = c - 'a';
            if (curr.children[i] == null) curr.children[i] = new TrieNode();
            curr = curr.children[i];
        }
        curr.isEnd = true;
    }

    public boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isEnd;
    }

    public boolean startsWith(String prefix) {
        return find(prefix) != null;
    }

    private TrieNode find(String s) {
        TrieNode curr = root;
        for (char c : s.toCharArray()) {
            int i = c - 'a';
            if (curr.children[i] == null) return null;
            curr = curr.children[i];
        }
        return curr;
    }
}
```

---

### 2. Add and Search Word (LC 211)
Design a data structure with `addWord(word)` and `search(word)` where `'.'` in search matches any single character.

```
dict.addWord("bad")
dict.addWord("dad")
dict.addWord("mad")
dict.search("pad") → false
dict.search("bad") → true
dict.search(".ad") → true   ('.' matches 'b','d','m')
dict.search("b..") → true
```

```java
class WordDictionary {
    class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd;
    }
    TrieNode root = new TrieNode();

    public void addWord(String word) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int i = c - 'a';
            if (curr.children[i] == null) curr.children[i] = new TrieNode();
            curr = curr.children[i];
        }
        curr.isEnd = true;
    }

    public boolean search(String word) {
        return dfs(word, 0, root);
    }

    private boolean dfs(String word, int idx, TrieNode node) {
        if (idx == word.length()) return node.isEnd;
        char c = word.charAt(idx);
        if (c == '.') {
            for (TrieNode child : node.children)
                if (child != null && dfs(word, idx + 1, child)) return true;
            return false;
        }
        TrieNode next = node.children[c - 'a'];
        return next != null && dfs(word, idx + 1, next);
    }
}
```

**Key**: wildcard '.' branches to ALL 26 children.

---

### 3. Word Search II (LC 212)
Given an `m x n` board of characters and a list of words, return all words on the board. Words must be formed by sequentially adjacent cells (horizontally/vertically), same cell not reused.

```
board = [["o","a","a","n"],
         ["e","t","a","e"],
         ["i","h","k","r"],
         ["i","f","l","v"]]
words = ["oath","pea","eat","rain"]

Output: ["eat","oath"]
```

```java
class Solution {
    class TrieNode {
        TrieNode[] children = new TrieNode[26];
        String word; // store word at end node (avoids path reconstruction)
    }

    public List<String> findWords(char[][] board, String[] words) {
        // Build trie
        TrieNode root = new TrieNode();
        for (String w : words) {
            TrieNode curr = root;
            for (char c : w.toCharArray()) {
                int i = c - 'a';
                if (curr.children[i] == null) curr.children[i] = new TrieNode();
                curr = curr.children[i];
            }
            curr.word = w;
        }

        List<String> result = new ArrayList<>();
        int m = board.length, n = board[0].length;

        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                dfs(board, i, j, root, result);

        return result;
    }

    void dfs(char[][] board, int r, int c, TrieNode node, List<String> result) {
        if (r < 0 || r >= board.length || c < 0 || c >= board[0].length) return;
        char ch = board[r][c];
        if (ch == '#' || node.children[ch - 'a'] == null) return;

        node = node.children[ch - 'a'];
        if (node.word != null) {
            result.add(node.word);
            node.word = null; // avoid duplicates
        }

        board[r][c] = '#'; // mark visited
        dfs(board, r+1, c, node, result);
        dfs(board, r-1, c, node, result);
        dfs(board, r, c+1, node, result);
        dfs(board, r, c-1, node, result);
        board[r][c] = ch; // restore
    }
}
```

**Key optimizations**:
1. Store word at end node → no path string needed
2. Set `node.word = null` after finding → avoid duplicates
3. Mark board with '#' during DFS → avoid revisiting

---

### 4. Replace Words (LC 648)
Given a list of roots and a sentence, replace each word in the sentence with its shortest root from the dictionary. If a word has multiple roots, use the shortest.

```
dictionary = ["cat","bat","rat"]
sentence   = "the cattle was rattled by the battery"
Output:    = "the cat was rat by the bat"
```

```java
class Solution {
    class TrieNode {
        TrieNode[] children = new TrieNode[26];
        String root;
    }

    public String replaceWords(List<String> dictionary, String sentence) {
        TrieNode trie = new TrieNode();
        for (String root : dictionary) {
            TrieNode curr = trie;
            for (char c : root.toCharArray()) {
                int i = c - 'a';
                if (curr.children[i] == null) curr.children[i] = new TrieNode();
                curr = curr.children[i];
            }
            curr.root = root;
        }

        StringBuilder sb = new StringBuilder();
        for (String word : sentence.split(" ")) {
            if (sb.length() > 0) sb.append(' ');
            TrieNode curr = trie;
            for (char c : word.toCharArray()) {
                int i = c - 'a';
                if (curr.children[i] == null || curr.root != null) break;
                curr = curr.children[i];
            }
            sb.append(curr.root != null ? curr.root : word);
        }
        return sb.toString();
    }
}
```

---

### 5. Design Search Autocomplete System (LC 642)
Design a search autocomplete system. Given historical sentences and their usage counts, for each character input return top 3 most frequent sentences with that prefix (tie-break: alphabetical order). `'#'` marks end of input and saves the sentence.

```
sentences=["i love you","island","ironman","i love leetcode"]
times    =[5, 3, 2, 2]

input('i') → ["i love you","island","i love leetcode"]
input(' ') → ["i love you","i love leetcode"]
input('a') → []
input('#') → saves "i a" with count 1
```

```java
class AutocompleteSystem {
    class TrieNode {
        Map<Character, TrieNode> children = new HashMap<>();
        Map<String, Integer> counts = new HashMap<>(); // sentences at this prefix
    }

    TrieNode root = new TrieNode();
    StringBuilder current = new StringBuilder();

    public AutocompleteSystem(String[] sentences, int[] times) {
        for (int i = 0; i < sentences.length; i++)
            insert(sentences[i], times[i]);
    }

    private void insert(String sentence, int count) {
        TrieNode curr = root;
        for (char c : sentence.toCharArray()) {
            curr.children.putIfAbsent(c, new TrieNode());
            curr = curr.children.get(c);
            curr.counts.merge(sentence, count, Integer::sum);
        }
    }

    public List<String> input(char c) {
        if (c == '#') {
            insert(current.toString(), 1);
            current.setLength(0);
            return new ArrayList<>();
        }
        current.append(c);
        TrieNode curr = root;
        for (char ch : current.toString().toCharArray()) {
            if (!curr.children.containsKey(ch)) return new ArrayList<>();
            curr = curr.children.get(ch);
        }
        // get top 3 by count, then alphabetical
        return curr.counts.entrySet().stream()
            .sorted((a, b) -> a.getValue().equals(b.getValue())
                ? a.getKey().compareTo(b.getKey())
                : b.getValue() - a.getValue())
            .limit(3)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }
}
```

---

### 6. Longest Word in Dictionary (LC 720)
Given array `words`, return the longest word that can be built one character at a time by other words in the array. If tie, return the lexicographically smallest.

```
Input:  words = ["w","wo","wor","worl","world"]
Output: "world"  → each prefix exists in the array

Input:  words = ["a","banana","app","appl","ap","apply","apple"]
Output: "apple"  → "apple" and "apply" are both valid, "apple" < "apply"
```

```java
class Solution {
    public String longestWord(String[] words) {
        Arrays.sort(words);
        Set<String> built = new HashSet<>();
        built.add("");
        String result = "";
        for (String w : words) {
            if (built.contains(w.substring(0, w.length() - 1))) {
                built.add(w);
                if (w.length() > result.length()) result = w;
            }
        }
        return result;
    }
}
```

*Also solvable with a trie — mark nodes and DFS for longest path where every prefix is a word.*

---

### 7. Count Prefix and Suffix Pairs (variant)
Given array `words`, count all pairs `(i, j)` where `i < j` and `words[i]` is a prefix of `words[j]`.

```
Input:  words = ["a","ab","abc","bc"]
Output: 3
        ("a","ab"), ("a","abc"), ("ab","abc")
```

```java
class Solution {
    public int countPrefixSuffixPairs(String[] words) {
        // Build trie from words
        class TrieNode { TrieNode[] c = new TrieNode[26]; int count = 0; }
        TrieNode root = new TrieNode();
        int result = 0;

        for (String word : words) {
            // query first (count words that are prefix of current word)
            TrieNode curr = root;
            for (int i = 0; i < word.length(); i++) {
                int idx = word.charAt(i) - 'a';
                if (curr.c[idx] == null) break;
                curr = curr.c[idx];
                if (i == word.length() - 1) result += curr.count;
            }
            // insert
            curr = root;
            for (char ch : word.toCharArray()) {
                int idx = ch - 'a';
                if (curr.c[idx] == null) curr.c[idx] = new TrieNode();
                curr = curr.c[idx];
            }
            curr.count++;
        }
        return result;
    }
}
```

---

### 8. Maximum XOR of Two Numbers in Array (LC 421)
Given integer array `nums`, return the maximum result of `nums[i] XOR nums[j]` where `0 <= i <= j < n`.

```
Input:  nums = [3,10,5,25,2,8]
Output: 28  → 5 XOR 25 = 0101 XOR 11001 = 11100 = 28

Input:  nums = [14,70,53,83,49,91,36,80,92,51,66,70]
Output: 127
```

```java
class Solution {
    class TrieNode {
        TrieNode[] children = new TrieNode[2]; // binary trie
    }

    public int findMaximumXOR(int[] nums) {
        TrieNode root = new TrieNode();
        for (int num : nums) {
            TrieNode curr = root;
            for (int i = 31; i >= 0; i--) {
                int bit = (num >> i) & 1;
                if (curr.children[bit] == null) curr.children[bit] = new TrieNode();
                curr = curr.children[bit];
            }
        }

        int max = 0;
        for (int num : nums) {
            TrieNode curr = root;
            int xor = 0;
            for (int i = 31; i >= 0; i--) {
                int bit = (num >> i) & 1;
                int want = 1 - bit; // want opposite for max XOR
                if (curr.children[want] != null) {
                    xor |= (1 << i);
                    curr = curr.children[want];
                } else {
                    curr = curr.children[bit];
                }
            }
            max = Math.max(max, xor);
        }
        return max;
    }
}
```

**Binary trie**: each level represents a bit; greedy pick opposite bit for max XOR.

---

### 9. Palindrome Pairs (LC 336)
Given array of unique strings `words`, return all pairs `[i, j]` such that `words[i] + words[j]` is a palindrome.

```
Input:  words = ["abcd","dcba","lls","s","sssll"]
Output: [[0,1],[1,0],[3,2],[2,4]]
        "abcd"+"dcba"="abcddcba" ✓
        "dcba"+"abcd"="dcbaabcd" ✓
        "s"+"lls"="slls" ✓
        "lls"+"sssll"="llssssll" ✓
```

```java
class Solution {
    public List<List<Integer>> palindromePairs(String[] words) {
        Map<String, Integer> wordMap = new HashMap<>();
        for (int i = 0; i < words.length; i++)
            wordMap.put(new StringBuilder(words[i]).reverse().toString(), i);

        List<List<Integer>> result = new ArrayList<>();
        for (int i = 0; i < words.length; i++) {
            String word = words[i];
            for (int j = 0; j <= word.length(); j++) {
                // check prefix
                if (isPalin(word, j, word.length() - 1)) {
                    String prefix = word.substring(0, j);
                    if (wordMap.containsKey(prefix) && wordMap.get(prefix) != i)
                        result.add(Arrays.asList(i, wordMap.get(prefix)));
                }
                // check suffix (avoid duplicate when j==word.length())
                if (j > 0 && isPalin(word, 0, j - 1)) {
                    String suffix = word.substring(j);
                    if (wordMap.containsKey(suffix) && wordMap.get(suffix) != i)
                        result.add(Arrays.asList(wordMap.get(suffix), i));
                }
            }
        }
        return result;
    }

    boolean isPalin(String s, int lo, int hi) {
        while (lo < hi) if (s.charAt(lo++) != s.charAt(hi--)) return false;
        return true;
    }
}
```

---

### 10. Word Break II (LC 140)
Given string `s` and dictionary `wordDict`, return all possible ways to segment `s` into space-separated dictionary words.

```
Input:  s = "catsanddog", wordDict = ["cat","cats","and","sand","dog"]
Output: ["cats and dog","cat sand dog"]

Input:  s = "pineapplepenapple"
        wordDict = ["apple","pen","applepen","pine","pineapple"]
Output: ["pine apple pen apple","pineapple pen apple","pine applepen apple"]
```

```java
class Solution {
    Map<Integer, List<String>> memo = new HashMap<>();

    public List<String> wordBreak(String s, List<String> wordDict) {
        Set<String> dict = new HashSet<>(wordDict);
        return backtrack(s, dict, 0);
    }

    List<String> backtrack(String s, Set<String> dict, int start) {
        if (memo.containsKey(start)) return memo.get(start);
        List<String> result = new ArrayList<>();
        if (start == s.length()) { result.add(""); return result; }

        for (int end = start + 1; end <= s.length(); end++) {
            String word = s.substring(start, end);
            if (dict.contains(word)) {
                List<String> rest = backtrack(s, dict, end);
                for (String r : rest)
                    result.add(word + (r.isEmpty() ? "" : " " + r));
            }
        }
        memo.put(start, result);
        return result;
    }
}
```

---

---

# Common Pitfalls

## 1. Not Resetting `isEnd` After Delete

```java
// When deleting a word, only clear isEnd at the final node
// Don't remove intermediate nodes unless they have no other children
node.isEnd = false;
```

---

## 2. Word Search II — Duplicate Results

```java
// Set node.word = null after adding to result
if (node.word != null) {
    result.add(node.word);
    node.word = null;  // prevent duplicates
}
```

---

## 3. Forgetting to Restore Board in DFS

```java
board[r][c] = '#';  // mark
dfs(...)
board[r][c] = ch;   // MUST restore — board is shared across DFS calls
```

---

## 4. Trie vs HashMap

```
HashMap<String, Integer>: O(1) lookup per word
Trie: O(L) per word, but supports:
  - Prefix queries
  - Wildcard matching
  - Space efficient for shared prefixes

Use Trie when prefix queries or wildcard are needed.
```

---

---

# Quick Reference Card

## Trie Operations

| Operation | Time | Notes |
|---|---|---|
| insert | O(L) | L = word length |
| search | O(L) | |
| startsWith | O(L) | |
| DFS collect | O(n * L) | n = words matching |
| Grid DFS + Trie | O(m * n * 4^L) | Early termination via trie |

## Pattern Map

| Problem | Trie Addition |
|---|---|
| Basic search/prefix | Standard trie |
| Wildcard '.' | DFS branching at '.' |
| Grid word search | Store word at end node + DFS grid |
| Autocomplete | Store counts at each node |
| Max XOR | Binary trie (0/1 per bit) |
| Shortest replacement | Stop at first end node |

## Java Snippet: TrieNode with count

```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd;
    int count;    // count of words passing through
    String word;  // word ending here (for Word Search II)
}
```
