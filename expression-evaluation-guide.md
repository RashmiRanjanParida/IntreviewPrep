# Expression Evaluation & Notation Conversion — Complete Guide for Google L6

---

## What Is Infix / Postfix / Prefix?

```
Infix   (human-natural):  a + b            operator BETWEEN operands
                                            needs precedence rules + parens to disambiguate

Postfix (Reverse Polish): a b +            operator AFTER its operands
Prefix  (Polish):         + a b            operator BEFORE its operands

Both postfix and prefix are UNAMBIGUOUS without any precedence table or parentheses —
the position of each token in the string already fully determines evaluation order.
That's why calculators, compilers, and stack machines use them internally.
```

Example of why infix is ambiguous and postfix isn't:

```
Infix:    a + b * c        -- is this (a+b)*c or a+(b*c)? Depends on precedence rules you must know.
Postfix:  a b c * +        -- unambiguous: multiply b,c first (they're adjacent to the operator
                               that appears first), then add a to that result. No rules needed.
```

---

## The Core Mechanic — Everything Here Is One of Three Stack Shapes

```
1. OPERAND STACK ONLY (postfix/prefix evaluation, build tree from postfix/prefix)
   → order is already fixed by the notation itself, so you never need to look ahead
     or compare precedence. Push operands, and when you see an operator, pop what
     you need and push the result back.

2. OPERATOR STACK + OPERAND STACK (infix parsing: conversion, calculators, build tree from infix)
   → you don't know evaluation order yet, so operators wait on a stack until you know
     whether to apply them now or let a higher-precedence operator go first.

3. RECURSION / EXPLICIT INDEX POINTER (nested structure: parens, Lisp scoping, chemical formulas)
   → the "stack" becomes the call stack itself. Whenever you see the start of a nested
     region, recurse; the recursive call consumes up to its own matching close and
     reports back both a value and how far it advanced the shared position pointer.
```

---

## How to Identify Which One You're Facing

| Signal in the problem | Pattern | Shape |
|---|---|---|
| "Evaluate Reverse Polish Notation" / tokens already postfix | Operand-only stack | 1 |
| "Convert infix to postfix/prefix" | Precedence-aware stack | 2 |
| "Evaluate this expression string" with `+-*/`, maybe parens | Calculator family | 2 or 3 |
| "Nested brackets", `k[...]` repeats, chemical formula, nested function calls | Recursion / index pointer | 3 |
| "Build a tree from a ___fix expression" | Same as its shape, but push tree nodes instead of values | 1 or 2 |
| "How many ways to parenthesize / insert operators" | Divide-and-conquer or backtracking — **not a stack problem at all** | — |

---

## The Three Skeletons

**Skeleton 1 — postfix/prefix evaluation (operand stack only):**
```java
// postfix: scan LEFT to RIGHT
for (char c : postfix.toCharArray()) {
    if (isOperand(c)) stack.push(value(c));
    else {
        int b = stack.pop(), a = stack.pop();   // b first: it was pushed later
        stack.push(apply(c, a, b));
    }
}
return stack.pop();
// prefix: same idea, scan RIGHT to LEFT, and pop order becomes (a first, then b)
```

**Skeleton 2 — infix with precedence (operator stack + output/operand stack):**
```java
for (char c : infix.toCharArray()) {
    if (isOperand(c)) output.push(c);
    else if (c == '(') ops.push(c);
    else if (c == ')') {
        while (ops.peek() != '(') resolve(ops.pop());   // drain back to matching '('
        ops.pop();
    } else {
        while (!ops.isEmpty() && ops.peek() != '(' && precedence(ops.peek()) >= precedence(c)) {
            resolve(ops.pop());
        }
        ops.push(c);
    }
}
while (!ops.isEmpty()) resolve(ops.pop());
```

**Skeleton 3 — recursive descent with a shared position pointer (parens / scoping / nested formulas):**
```java
static int parse(String s, int[] pos) {
    // pos is a 1-element array purely so the recursive call can advance
    // the caller's cursor — Java has no pass-by-reference for primitives.
    ...
    if (s.charAt(pos[0]) == '(') {
        pos[0]++;                      // consume '('
        int inner = parse(s, pos);     // recursive call consumes up through its own ')'
        ...
    }
    ...
}
```

---

## Questions to Ask Before Coding

```
1. Is the expression ALREADY postfix/prefix, or is it infix?
   → already postfix/prefix: Skeleton 1, no precedence logic needed at all
   → infix: Skeleton 2 or 3

2. Do operators have different precedence (+- vs */)?
   → yes: need a precedence() function, or split into two eager passes (Basic Calculator II style)
   → no (single operator type, or explicit parens dictate everything): simpler, skip the table

3. Are there parentheses / nested structure?
   → yes: recursion (Skeleton 3), or push saved state onto a stack on '(' and restore on ')'
   → no: a single linear pass suffices

4. Single-character operands, or multi-character (numbers, variable names, element symbols)?
   → multi-character: you need an explicit accumulation loop / tokenizer, not just charAt(i)

5. Do you need the VALUE, or the STRUCTURE (a tree)?
   → value: collapse eagerly, push numbers
   → structure: push tree nodes instead of numbers — same control flow otherwise
```

---
---

# Problems

## Foundation

### 1. Valid Parentheses (LC 20)
**Description:** Given a string of `()[]{}`, determine if the brackets are properly matched and nested.

**Example:**
- Input: `"()[]{}"`  Output: `true`
- Input: `"([)]"`  Output: `false` (crosses instead of nesting)

```java
static boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stack.isEmpty();
}
```
**Pattern**: push on open, pop-and-compare on close. The prerequisite skill for every other problem in this guide.

---

### 2. Longest Valid Parentheses (LC 32)
**Description:** Given a string of only `(` and `)`, find the length of the longest valid (well-formed) substring.

**Example:**
- Input: `")()())"`  Output: `4` (`"()()"`)

```java
static int longestValidParentheses(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(-1);   // sentinel: boundary just before the current run
    int max = 0;
    for (int i = 0; i < s.length(); i++) {
        if (s.charAt(i) == '(') {
            stack.push(i);
        } else {
            stack.pop();
            if (stack.isEmpty()) {
                stack.push(i);                          // this ')' is unmatched — new boundary
            } else {
                max = Math.max(max, i - stack.peek());  // length of the run ending here
            }
        }
    }
    return max;
}
```
**Pattern**: stack of *indices*, not characters — what's left on top after popping tells you where the current valid run began. (Full write-up with DP and O(1)-space alternatives covered earlier in this session.)

---

## Notation Conversion
*Classic CS-fundamentals exercises — not individually numbered on LeetCode, but a very standard direct interview ask. Assumes single-character operands for clarity; multi-character operands just need a small tokenizer bolted on.*

### 3. Infix → Postfix
**Description:** Convert `a+b*c` into `abc*+`.

**Example:** `"a+b*c"` → `"abc*+"`, `"(a+b)*c"` → `"ab+c*"`

```java
static int precedence(char op) {
    switch (op) {
        case '+': case '-': return 1;
        case '*': case '/': return 2;
        case '^': return 3;
    }
    return -1;
}

static String infixToPostfix(String expr) {
    StringBuilder output = new StringBuilder();
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : expr.toCharArray()) {
        if (c == ' ') continue;
        if (Character.isLetterOrDigit(c)) {
            output.append(c);
        } else if (c == '(') {
            stack.push(c);
        } else if (c == ')') {
            while (stack.peek() != '(') output.append(stack.pop());
            stack.pop();
        } else {
            while (!stack.isEmpty() && stack.peek() != '(' &&
                   (precedence(stack.peek()) > precedence(c) ||
                    (precedence(stack.peek()) == precedence(c) && c != '^'))) {
                output.append(stack.pop());
            }
            stack.push(c);
        }
    }
    while (!stack.isEmpty()) output.append(stack.pop());
    return output.toString();
}
```
**Pattern**: Skeleton 2 exactly — an operator only gets pushed once everything of *higher-or-equal* precedence ahead of it has been flushed to output. The `c != '^'` clause keeps `^` right-associative (`2^3^2` groups as `2^(3^2)`); every other operator here is left-associative.

---

### 4. Infix → Prefix
**Description:** Convert `a-b-c` into `--abc`.

**Example:** `"a-b-c"` → `"--abc"`, `"a+b*c"` → `"+a*bc"`

```java
static String infixToPrefix(String expr) {
    StringBuilder rev = new StringBuilder();
    for (int i = expr.length() - 1; i >= 0; i--) {
        char c = expr.charAt(i);
        rev.append(c == '(' ? ')' : c == ')' ? '(' : c);   // reverse + swap parens
    }
    StringBuilder output = new StringBuilder();
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : rev.toString().toCharArray()) {
        if (c == ' ') continue;
        if (Character.isLetterOrDigit(c)) {
            output.append(c);
        } else if (c == '(') {
            stack.push(c);
        } else if (c == ')') {
            while (stack.peek() != '(') output.append(stack.pop());
            stack.pop();
        } else {
            // strict '>' here (not '>=') — this is the one change vs. infix-to-postfix
            while (!stack.isEmpty() && stack.peek() != '(' && precedence(stack.peek()) > precedence(c)) {
                output.append(stack.pop());
            }
            stack.push(c);
        }
    }
    while (!stack.isEmpty()) output.append(stack.pop());
    return output.reverse().toString();
}
```
**Pattern**: reverse the string (swapping bracket direction), run a *slightly* modified infix-to-postfix on it, reverse the result. The modification: equal-precedence operators must **not** pop during this pass (strict `>` instead of `>=`), because processing right-to-left flips which side is "first" — without this tweak, left-associative chains like `a-b-c` come out wrong. (Caveat: this simple version treats every operator as left-associative; making `^` right-associative *within* infix-to-prefix needs one more special case, skipped here for clarity.)

---

### 5. Postfix → Infix
**Description:** Convert `abc*+` back into `(a+(b*c))`.

```java
static String postfixToInfix(String postfix) {
    Deque<String> stack = new ArrayDeque<>();
    for (char c : postfix.toCharArray()) {
        if (Character.isLetterOrDigit(c)) {
            stack.push(String.valueOf(c));
        } else {
            String b = stack.pop();   // pushed later → right operand
            String a = stack.pop();   // right operand
            stack.push("(" + a + c + b + ")");
        }
    }
    return stack.pop();
}
```
**Pattern**: Skeleton 1, but the stack holds partially-built *strings* instead of numbers. Every operator wraps its two most-recent operands in parens — this is exactly how you'd reconstruct explicit grouping.

---

### 6. Prefix → Infix
**Description:** Convert `--abc` back into `((a-b)-c)`.

```java
static String prefixToInfix(String prefix) {
    Deque<String> stack = new ArrayDeque<>();
    for (int i = prefix.length() - 1; i >= 0; i--) {
        char c = prefix.charAt(i);
        if (Character.isLetterOrDigit(c)) {
            stack.push(String.valueOf(c));
        } else {
            String a = stack.pop();   // scanning right-to-left: first pop = LEFT operand
            String b = stack.pop();
            stack.push("(" + a + c + b + ")");
        }
    }
    return stack.pop();
}
```
**Pattern**: mirror image of postfix→infix — scan right-to-left, and the pop order flips (first pop is the left operand here, versus the right operand when scanning postfix left-to-right).

---

### 7. Postfix ↔ Prefix (direct, no infix detour)
**Description:** Convert directly between the two unambiguous notations without reconstructing infix in between.

**Example:** `"ab-"` (postfix) ↔ `"-ab"` (prefix), both meaning `a - b`.

```java
static String postfixToPrefix(String postfix) {
    Deque<String> stack = new ArrayDeque<>();
    for (char c : postfix.toCharArray()) {
        if (Character.isLetterOrDigit(c)) {
            stack.push(String.valueOf(c));
        } else {
            String b = stack.pop();
            String a = stack.pop();
            stack.push(c + a + b);        // just concatenation, no parens needed
        }
    }
    return stack.pop();
}

static String prefixToPostfix(String prefix) {
    Deque<String> stack = new ArrayDeque<>();
    for (int i = prefix.length() - 1; i >= 0; i--) {
        char c = prefix.charAt(i);
        if (Character.isLetterOrDigit(c)) {
            stack.push(String.valueOf(c));
        } else {
            String a = stack.pop();
            String b = stack.pop();
            stack.push(a + b + c);
        }
    }
    return stack.pop();
}
```
**Pattern**: identical to problems 5 and 6, except you never need parentheses — you're not reconstructing human-readable grouping, just relocating the operator token relative to its two already-resolved operand strings. Good interview signal: if someone only knows the "go through infix" trick, they'll over-complicate this.

---

## Direct Evaluation (Calculator Family)

### 8. Evaluate Reverse Polish Notation (LC 150)
**Description:** Evaluate a postfix expression given as tokens.

**Example:** `["2","1","+","3","*"]` → `9`  *((2+1)\*3)*

```java
static int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String t : tokens) {
        switch (t) {
            case "+": { int b = stack.pop(), a = stack.pop(); stack.push(a + b); break; }
            case "-": { int b = stack.pop(), a = stack.pop(); stack.push(a - b); break; }
            case "*": { int b = stack.pop(), a = stack.pop(); stack.push(a * b); break; }
            case "/": { int b = stack.pop(), a = stack.pop(); stack.push(a / b); break; }
            default:  stack.push(Integer.parseInt(t));
        }
    }
    return stack.pop();
}
```
**Pattern**: Skeleton 1, textbook pure form. This is the single most common "expression" interview question — everything else in this guide is a variation on top of it.

---

### 9. Basic Calculator (LC 224)
**Description:** Evaluate an infix string with `+ -` and nested parentheses only (no `* /`).

**Example:** `"(1+(4+5+2)-3)+(6+8)"` → `23`

```java
static int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int result = 0, sign = 1, num = 0;
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (Character.isDigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '+') {
            result += sign * num; num = 0; sign = 1;
        } else if (c == '-') {
            result += sign * num; num = 0; sign = -1;
        } else if (c == '(') {
            stack.push(result); stack.push(sign);   // save context, start fresh inside the parens
            result = 0; sign = 1;
        } else if (c == ')') {
            result += sign * num; num = 0;
            result *= stack.pop();   // sign that applied to the whole parenthesized group
            result += stack.pop();   // result accumulated before the group started
        }
    }
    result += sign * num;
    return result;
}
```
**Pattern**: since only `+ -` exist, precedence never matters — the only thing parentheses complicate is *sign propagation*. Push `(runningResult, currentSign)` on `(`, restore and combine on `)`. Spaces fall through every `if`/`else if` untouched, which is why there's no explicit space-handling branch.

---

### 10. Basic Calculator II (LC 227)
**Description:** Evaluate an infix string with `+ - * /` and standard precedence, no parentheses.

**Example:** `"3+2*2"` → `7`,  `" 3/2 "` → `1`

```java
static int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int num = 0;
    char op = '+';
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (Character.isDigit(c)) num = num * 10 + (c - '0');
        if ((!Character.isDigit(c) && c != ' ') || i == s.length() - 1) {
            switch (op) {
                case '+': stack.push(num); break;
                case '-': stack.push(-num); break;
                case '*': stack.push(stack.pop() * num); break;
                case '/': stack.push(stack.pop() / num); break;
            }
            op = c;
            num = 0;
        }
    }
    int result = 0;
    for (int n : stack) result += n;
    return result;
}
```
**Pattern**: resolve `* /` **immediately** (pop-multiply-push back), but defer `+ -` by pushing signed values onto the stack and summing everything at the end. This sidesteps precedence entirely — by the time you sum the stack, every high-precedence operation has already collapsed into a single signed number.

---

### 11. Basic Calculator III (LC 772)
**Description:** The union of the previous two — `+ - * /`, precedence, **and** parentheses together.

**Example:** `"2*(5+5*2)/3+(6/2)"` → `13`

```java
static int calculate(String s) {
    int[] pos = {0};
    return helper(s, pos);
}

static int helper(String s, int[] pos) {
    Deque<Integer> stack = new ArrayDeque<>();
    char op = '+';
    while (pos[0] < s.length()) {
        char c = s.charAt(pos[0]);
        if (c == ' ') { pos[0]++; continue; }
        if (c == ')') { pos[0]++; break; }        // return control to the caller

        int num;
        if (c == '(') {
            pos[0]++;
            num = helper(s, pos);                  // recursively evaluate the whole group as one "operand"
        } else {
            num = 0;
            while (pos[0] < s.length() && Character.isDigit(s.charAt(pos[0]))) {
                num = num * 10 + (s.charAt(pos[0]) - '0');
                pos[0]++;
            }
        }
        switch (op) {                               // same eager-resolve trick as LC 227
            case '+': stack.push(num); break;
            case '-': stack.push(-num); break;
            case '*': stack.push(stack.pop() * num); break;
            case '/': stack.push(stack.pop() / num); break;
        }
        while (pos[0] < s.length() && s.charAt(pos[0]) == ' ') pos[0]++;
        if (pos[0] < s.length() && s.charAt(pos[0]) != ')') {
            op = s.charAt(pos[0]);
            pos[0]++;
        }
    }
    int result = 0;
    for (int n : stack) result += n;
    return result;
}
```
**Pattern**: Skeleton 3 layered on top of LC 227's eager `* /` resolution. The key move: a parenthesized group is evaluated by a **recursive call that returns a single number**, then fed into the exact same "apply pending operator" logic as a plain digit would be — from the outer loop's point of view, `(...)` and a literal number are indistinguishable once the recursive call returns.

---

### 12. Parsing A Boolean Expression (LC 1106)
**Description:** Evaluate `t`/`f`/`!(expr)`/`&(expr1,expr2,...)`/`|(expr1,expr2,...)`.

**Example:** `"|(&(t,f,t),!(t))"` → `false`

```java
static boolean parseBoolExpr(String expression) {
    int[] pos = {0};
    return parse(expression, pos);
}

static boolean parse(String s, int[] pos) {
    char c = s.charAt(pos[0]++);
    if (c == 't') return true;
    if (c == 'f') return false;
    if (c == '!') {
        pos[0]++;                       // skip '('
        boolean val = !parse(s, pos);
        pos[0]++;                       // skip ')'
        return val;
    }
    pos[0]++;                           // skip '(' after '&' or '|'
    List<Boolean> vals = new ArrayList<>();
    vals.add(parse(s, pos));
    while (s.charAt(pos[0]) == ',') {
        pos[0]++;
        vals.add(parse(s, pos));
    }
    pos[0]++;                           // skip ')'
    boolean result = (c == '&');
    for (boolean v : vals) result = (c == '&') ? (result && v) : (result || v);
    return result;
}
```
**Pattern**: same Skeleton 3 recursive-descent shape as LC 772, just over a boolean grammar with **variable arity** (`&`/`|` can take 2+ operands, unlike arithmetic's fixed 2). The comma-separated operand list is what makes this a small step up in complexity from a fixed-arity parser.

---

## Expression Trees

```java
static class TreeNode {
    char val;
    TreeNode left, right;
    TreeNode(char val) { this.val = val; }
}
```

### 13. Build Expression Tree from Postfix
**Example:** `"23+"` → tree for `(2+3)`

```java
static TreeNode buildFromPostfix(String postfix) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    for (char c : postfix.toCharArray()) {
        TreeNode node = new TreeNode(c);
        if (isOperator(c)) {
            node.right = stack.pop();   // pushed later → right child
            node.left = stack.pop();
        }
        stack.push(node);
    }
    return stack.pop();
}
```
**Pattern**: Skeleton 1, but push **nodes** instead of collapsing to a value — same control flow as postfix evaluation, right down to which pop becomes the right child.

---

### 14. Build Expression Tree from Prefix
**Example:** `"+23"` → tree for `(2+3)`

```java
static TreeNode buildFromPrefix(String prefix) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    for (int i = prefix.length() - 1; i >= 0; i--) {
        char c = prefix.charAt(i);
        TreeNode node = new TreeNode(c);
        if (isOperator(c)) {
            node.left = stack.pop();    // scanning right-to-left: first pop = left child
            node.right = stack.pop();
        }
        stack.push(node);
    }
    return stack.pop();
}
```
**Pattern**: mirror of #13, same right-to-left / pop-order-flip relationship as prefix→infix.

---

### 15. Build Expression Tree from Infix
**Example:** `"a+b*c"` → tree where the root is `+`, its right child is `*` (with leaves `b`, `c`) — **not** a left-to-right chain, because precedence must be respected.

```java
static TreeNode combine(char op, Deque<TreeNode> nodes) {
    TreeNode node = new TreeNode(op);
    node.right = nodes.pop();
    node.left = nodes.pop();
    return node;
}

static TreeNode buildFromInfix(String expr) {
    Deque<TreeNode> nodes = new ArrayDeque<>();
    Deque<Character> ops = new ArrayDeque<>();
    for (char c : expr.toCharArray()) {
        if (c == ' ') continue;
        if (Character.isLetterOrDigit(c)) {
            nodes.push(new TreeNode(c));
        } else if (c == '(') {
            ops.push(c);
        } else if (c == ')') {
            while (ops.peek() != '(') nodes.push(combine(ops.pop(), nodes));
            ops.pop();
        } else {
            while (!ops.isEmpty() && ops.peek() != '(' && precedence(ops.peek()) >= precedence(c)) {
                nodes.push(combine(ops.pop(), nodes));
            }
            ops.push(c);
        }
    }
    while (!ops.isEmpty()) nodes.push(combine(ops.pop(), nodes));
    return nodes.pop();
}
```
**Pattern**: literally infix-to-postfix (#3) with every `output.append(...)` replaced by `nodes.push(combine(...))`. If you understand #3, this is a 5-minute transformation, not a new algorithm — a good thing to say out loud in an interview to show you see the connection.

---

### 16. Evaluate Expression Tree (LC 1628 style — *Design an Expression Tree With Evaluate Function*)
**Description:** Given the root of an expression tree, compute its value.

```java
static int evaluateTree(TreeNode root) {
    if (root.left == null && root.right == null) return root.val - '0';   // leaf = operand
    int left = evaluateTree(root.left);
    int right = evaluateTree(root.right);
    switch (root.val) {
        case '+': return left + right;
        case '-': return left - right;
        case '*': return left * right;
        case '/': return left / right;
    }
    throw new IllegalArgumentException("bad operator " + root.val);
}
```
**Pattern**: plain post-order traversal — evaluate both children, then apply the operator at the current node. This is the "obvious" recursive shape everything else in this section is secretly building toward.

---

### 17. Check If Two Expression Trees Are Equivalent (LC 1612)
**Description:** Two expression trees (LC 1612 restricts internal nodes to `+` only, so the operator is commutative and associative) are equivalent if they compute the same value for **every** variable assignment.

**Example:** `a+(b+c)` and `(a+b)+c` are equivalent (different shapes, same leaves); `a+(b+c)` and `a+(b+d)` are not.

```java
static boolean areEquivalent(TreeNode root1, TreeNode root2) {
    int[] count1 = new int[26], count2 = new int[26];
    collectLeaves(root1, count1);
    collectLeaves(root2, count2);
    return Arrays.equals(count1, count2);
}

static void collectLeaves(TreeNode node, int[] count) {
    if (node == null) return;
    if (node.left == null && node.right == null) { count[node.val - 'a']++; return; }
    collectLeaves(node.left, count);
    collectLeaves(node.right, count);
}
```
**Pattern**: because `+` is commutative and associative, tree *shape* is irrelevant — only the **multiset of leaf variables** matters. Don't overthink this into a structural tree-diff; that's solving a harder problem than the one that was actually asked. (If the operator set included `-` or non-commutative operators, you'd need actual symbolic evaluation instead — worth saying out loud if an interviewer changes the constraint.)

---

## Adjacent Parsing Problems
*Same stack/recursion mechanics, different payloads — commonly grouped with this topic in interview prep.*

### 18. Decode String (LC 394)
**Description:** Expand `k[encoded_string]` patterns, which can nest.

**Example:** `"3[a2[c]]"` → `"accaccacc"`

```java
static String decodeString(String s) {
    Deque<Integer> countStack = new ArrayDeque<>();
    Deque<StringBuilder> stringStack = new ArrayDeque<>();
    StringBuilder current = new StringBuilder();
    int k = 0;
    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            k = k * 10 + (c - '0');
        } else if (c == '[') {
            countStack.push(k);
            stringStack.push(current);
            current = new StringBuilder();
            k = 0;
        } else if (c == ']') {
            StringBuilder decoded = stringStack.pop();
            int count = countStack.pop();
            for (int i = 0; i < count; i++) decoded.append(current);
            current = decoded;
        } else {
            current.append(c);
        }
    }
    return current.toString();
}
```
**Pattern**: on `[`, save *both* the repeat count and the string-so-far, then start building fresh; on `]`, repeat the just-finished inner string and glue it onto what was saved. Two parallel stacks instead of recursion — an explicit-stack rewrite of Skeleton 3 that avoids actual recursive calls.

---

### 19. Number of Atoms (LC 726)
**Description:** Parse a chemical formula with nested parens and multiplier suffixes into a sorted atom-count string.

**Example:** `"K4(ON(SO3)2)2"` → `"K4N2O14S4"`

```java
static String countOfAtoms(String formula) {
    int[] pos = {0};
    TreeMap<String, Integer> counts = parseFormula(formula, pos);   // TreeMap = free alphabetical sort
    StringBuilder sb = new StringBuilder();
    for (Map.Entry<String, Integer> e : counts.entrySet()) {
        sb.append(e.getKey());
        if (e.getValue() > 1) sb.append(e.getValue());
    }
    return sb.toString();
}

static TreeMap<String, Integer> parseFormula(String s, int[] pos) {
    TreeMap<String, Integer> counts = new TreeMap<>();
    while (pos[0] < s.length() && s.charAt(pos[0]) != ')') {
        if (s.charAt(pos[0]) == '(') {
            pos[0]++;
            Map<String, Integer> inner = parseFormula(s, pos);   // recurse on the group
            pos[0]++;                                             // skip ')'
            int mult = parseNum(s, pos);                          // multiplier AFTER the group
            for (Map.Entry<String, Integer> e : inner.entrySet()) {
                counts.merge(e.getKey(), e.getValue() * mult, Integer::sum);
            }
        } else {
            String name = parseName(s, pos);
            int mult = parseNum(s, pos);
            counts.merge(name, mult, Integer::sum);
        }
    }
    return counts;
}

static String parseName(String s, int[] pos) {
    int start = pos[0];
    pos[0]++;                                                     // first char always uppercase
    while (pos[0] < s.length() && Character.isLowerCase(s.charAt(pos[0]))) pos[0]++;
    return s.substring(start, pos[0]);
}

static int parseNum(String s, int[] pos) {
    int start = pos[0];
    while (pos[0] < s.length() && Character.isDigit(s.charAt(pos[0]))) pos[0]++;
    return start == pos[0] ? 1 : Integer.parseInt(s.substring(start, pos[0]));
}
```
**Pattern**: Skeleton 3 with **two** helper tokenizers (element name, trailing number) instead of one — the recursion itself only handles the nesting; parsing "what is one token" is deliberately factored out so the recursive function stays readable. `TreeMap` quietly does the "output sorted by name" requirement for free.

---

### 20. Parse Lisp Expression (LC 736)
**Description:** Evaluate `(let ...)` / `(add ...)` / `(mult ...)` expressions with variable scoping — `let` bindings can shadow outer variables and reference earlier bindings within the same `let`.

**Example:** `"(let x 2 (mult x (let x 3 y 4 (add x y))))"` → `14`

```java
static int evaluate(String expression) {
    return eval(expression, new ArrayDeque<>());
}

static int eval(String expr, Deque<Map<String, Integer>> scopes) {
    if (expr.charAt(0) != '(') {
        if (Character.isDigit(expr.charAt(0)) || expr.charAt(0) == '-') {
            return Integer.parseInt(expr);
        }
        for (Map<String, Integer> scope : scopes) {          // search innermost to outermost
            if (scope.containsKey(expr)) return scope.get(expr);
        }
        throw new IllegalArgumentException("undefined variable: " + expr);
    }
    String inner = expr.substring(1, expr.length() - 1);
    List<String> tokens = tokenize(inner);                    // top-level split, respecting nested parens
    String op = tokens.get(0);
    Map<String, Integer> newScope = new HashMap<>();
    scopes.push(newScope);
    int result;
    if (op.equals("let")) {
        int i = 1;
        while (i <= tokens.size() - 3) {     // stop once only the final expr remains
            newScope.put(tokens.get(i), eval(tokens.get(i + 1), scopes));
            i += 2;
        }
        result = eval(tokens.get(tokens.size() - 1), scopes);
    } else if (op.equals("add")) {
        result = eval(tokens.get(1), scopes) + eval(tokens.get(2), scopes);
    } else {
        result = eval(tokens.get(1), scopes) * eval(tokens.get(2), scopes);
    }
    scopes.pop();
    return result;
}

static List<String> tokenize(String s) {
    List<String> tokens = new ArrayList<>();
    int depth = 0;
    StringBuilder cur = new StringBuilder();
    for (char c : s.toCharArray()) {
        if (c == '(') depth++;
        if (c == ')') depth--;
        if (c == ' ' && depth == 0) {           // only split on TOP-level spaces
            tokens.add(cur.toString());
            cur = new StringBuilder();
        } else {
            cur.append(c);
        }
    }
    if (cur.length() > 0) tokens.add(cur.toString());
    return tokens;
}
```
**Pattern**: this is Skeleton 3 plus an explicit **environment stack** (`Deque<Map<String,Integer>>`) for scoping — the closest thing in this guide to writing a tiny interpreter. Two easy-to-miss details: (1) `newScope` is populated incrementally *while* evaluating later bindings in the same `let`, which is exactly what makes `(let x 1 y (add x 1) ...)` correctly see `x`'s value when computing `y`; (2) the loop bound `i <= tokens.size() - 3` is what correctly separates "variable/expression pairs" from "the trailing expression to actually evaluate and return" — get this off by one and you'll either drop the final expression or misread it as a variable name.

---

### 21. Different Ways to Add Parentheses (LC 241)
**Description:** Given a string of digits and `+ - *`, return every possible result from every way of parenthesizing it.

**Example:** `"2-1-1"` → `[0, 2]`  *((2-1)-1=0, 2-(1-1)=2)*

```java
static List<Integer> diffWaysToCompute(String expression) {
    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < expression.length(); i++) {
        char c = expression.charAt(i);
        if (c == '+' || c == '-' || c == '*') {
            List<Integer> left = diffWaysToCompute(expression.substring(0, i));
            List<Integer> right = diffWaysToCompute(expression.substring(i + 1));
            for (int l : left) {
                for (int r : right) {
                    switch (c) {
                        case '+': result.add(l + r); break;
                        case '-': result.add(l - r); break;
                        case '*': result.add(l * r); break;
                    }
                }
            }
        }
    }
    if (result.isEmpty()) result.add(Integer.parseInt(expression));  // base case: no operator = pure number
    return result;
}
```
**Pattern**: **not** a stack problem — pure divide-and-conquer. Every operator in the string is a candidate "last operation performed"; recursively solve both sides of it and combine every left-result with every right-result. Add memoization (keyed on the substring) if asked to optimize — repeated substrings recur often on longer inputs.

---

### 22. Expression Add Operators (LC 282)
**Description:** Given a digit string and a target, insert `+ - *` between digits (digits can also be grouped into multi-digit numbers) so the resulting expression evaluates to target.

**Example:** `num="123"`, `target=6` → `["1+2+3", "1*2*3"]`

```java
static List<String> addOperators(String num, int target) {
    List<String> result = new ArrayList<>();
    backtrack(num, target, 0, "", 0, 0, result);
    return result;
}

static void backtrack(String num, int target, int pos, String expr,
                       long prevValue, long currentTotal, List<String> result) {
    if (pos == num.length()) {
        if (currentTotal == target) result.add(expr);
        return;
    }
    for (int i = pos; i < num.length(); i++) {
        if (i > pos && num.charAt(pos) == '0') break;   // no numbers with a leading zero (e.g. "05")
        String numStr = num.substring(pos, i + 1);
        long val = Long.parseLong(numStr);
        if (pos == 0) {
            backtrack(num, target, i + 1, numStr, val, val, result);
        } else {
            backtrack(num, target, i + 1, expr + "+" + numStr, val, currentTotal + val, result);
            backtrack(num, target, i + 1, expr + "-" + numStr, -val, currentTotal - val, result);
            // '*' has higher precedence than the '+'/'-' already committed to currentTotal, so
            // undo the last term's contribution and redo it multiplied by the new number:
            backtrack(num, target, i + 1, expr + "*" + numStr, prevValue * val,
                      currentTotal - prevValue + prevValue * val, result);
        }
    }
}
```
**Pattern**: backtracking over where to "cut" the digit string, plus one genuinely subtle trick — since `*` binds tighter than `+`/`-`, applying it to a running left-to-right total requires **undoing** the previous term (`currentTotal - prevValue`) before redoing it multiplied by the new number (`+ prevValue * val`). This is the same "fix up eager evaluation for higher precedence" idea as Basic Calculator II, just expressed as arithmetic on the running total instead of a stack.

---
---

## Complexity

| Problem | Time | Space |
|---|---|---|
| Valid Parentheses (LC 20) | O(n) | O(n) |
| Longest Valid Parentheses (LC 32) | O(n) | O(n) |
| Notation conversions (all 6) | O(n) | O(n) |
| Evaluate RPN (LC 150) | O(n) | O(n) |
| Basic Calculator I / II (LC 224 / 227) | O(n) | O(n) |
| Basic Calculator III (LC 772) | O(n) | O(n) — recursion depth = paren nesting depth |
| Parsing A Boolean Expression (LC 1106) | O(n) | O(n) — recursion depth |
| Build / evaluate / compare expression trees | O(n) | O(n) |
| Decode String (LC 394) | O(n · maxRepeat) — output can be larger than input | O(n) |
| Number of Atoms (LC 726) | O(n) | O(n) |
| Parse Lisp Expression (LC 736) | O(n²) worst case (variable lookups walk the scope stack) | O(n) |
| Different Ways to Add Parentheses (LC 241) | Exponential (Catalan-number-ish) without memo | O(same) |
| Expression Add Operators (LC 282) | O(n · 3ⁿ) backtracking | O(n) recursion depth |

---

## Problem → Pattern Map

| Problem | Category | Core structure |
|---|---|---|
| LC 20 | Foundation | stack of brackets |
| LC 32 | Foundation | stack of indices, `-1` sentinel |
| Infix→Postfix | Conversion | operator stack + precedence (Skeleton 2) |
| Infix→Prefix | Conversion | reverse + swapped precedence check + reverse back |
| Postfix→Infix / Prefix→Infix | Conversion | stack of strings, wrap in parens |
| Postfix↔Prefix direct | Conversion | stack of strings, no parens needed |
| LC 150 | Evaluation | operand-only stack (Skeleton 1) |
| LC 224 | Evaluation | sign-tracking stack, no precedence needed |
| LC 227 | Evaluation | eager `*//` resolve, defer `+-` to a final sum |
| LC 772 | Evaluation | LC 227's trick + recursion for parens (Skeleton 3) |
| LC 1106 | Evaluation | recursive descent, variable-arity operators |
| Build tree from postfix/prefix | Tree | Skeleton 1, push nodes not values |
| Build tree from infix | Tree | Skeleton 2, push nodes not values |
| LC 1628-style evaluate | Tree | plain post-order traversal |
| LC 1612 | Tree | leaf multiset comparison (relies on `+` being commutative) |
| LC 394 | Adjacent | two parallel stacks (count, partial string) |
| LC 726 | Adjacent | recursion + two token-level helper parsers |
| LC 736 | Adjacent | recursion + explicit scope stack (environment) |
| LC 241 | Adjacent | divide and conquer, not a stack problem |
| LC 282 | Adjacent | backtracking + "undo eager eval" trick for `*` |
