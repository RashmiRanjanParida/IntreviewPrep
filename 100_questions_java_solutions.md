# 100 Screening-Round Questions — Java Solutions with Examples

Each solution: problem restated in one line, core Java logic, and a runnable example showing input/output. Solutions favor clarity and the idioms you'd want to say out loud in an interview (`computeIfAbsent`, `retainAll`, streams where they read cleanly) over micro-optimization.

---

## 1. HashMap/Set — Intersection & Membership

### 1. Two lists (userId, category) — userIds in both with combined categories ≥ 2
```java
static List<Integer> q1(List<int[]> l1, List<int[]> l2) {
    Map<Integer, Set<Integer>> m1 = new HashMap<>(), m2 = new HashMap<>();
    for (int[] p : l1) m1.computeIfAbsent(p[0], k -> new HashSet<>()).add(p[1]);
    for (int[] p : l2) m2.computeIfAbsent(p[0], k -> new HashSet<>()).add(p[1]);
    List<Integer> res = new ArrayList<>();
    for (int uid : m1.keySet()) {
        if (m2.containsKey(uid)) {
            Set<Integer> combined = new HashSet<>(m1.get(uid));
            combined.addAll(m2.get(uid));
            if (combined.size() >= 2) res.add(uid);
        }
    }
    return res;
}
// Example: l1=[[1,10],[1,20],[2,10]], l2=[[1,10],[2,10],[4,20]] -> [1]
```

### 2. Users in list A but not list B
```java
static Set<Integer> q2(List<int[]> a, List<int[]> b) {
    Set<Integer> setA = new HashSet<>(), setB = new HashSet<>();
    for (int[] p : a) setA.add(p[0]);
    for (int[] p : b) setB.add(p[0]);
    setA.removeAll(setB);
    return setA;
}
// Example: a=[[1,'x'],[2,'x']], b=[[2,'x']] -> {1}
```

### 3. userIds present in all 3 lists
```java
static Set<Integer> q3(List<int[]> l1, List<int[]> l2, List<int[]> l3) {
    Set<Integer> s1 = extractIds(l1), s2 = extractIds(l2), s3 = extractIds(l3);
    s1.retainAll(s2);
    s1.retainAll(s3);
    return s1;
}
static Set<Integer> extractIds(List<int[]> l) {
    Set<Integer> s = new HashSet<>();
    for (int[] p : l) s.add(p[0]);
    return s;
}
// Example: l1 has {1,2,3}, l2 has {1,2}, l3 has {1,4} -> {1}
```

### 4. Two lists (userId, timestamp) — active in both within 5-min window
```java
static List<Integer> q4(List<long[]> l1, List<long[]> l2) {
    Map<Integer, List<Long>> m1 = new HashMap<>(), m2 = new HashMap<>();
    for (long[] p : l1) m1.computeIfAbsent((int) p[0], k -> new ArrayList<>()).add(p[1]);
    for (long[] p : l2) m2.computeIfAbsent((int) p[0], k -> new ArrayList<>()).add(p[1]);
    List<Integer> res = new ArrayList<>();
    for (int uid : m1.keySet()) {
        if (!m2.containsKey(uid)) continue;
        boolean within = false;
        for (long t1 : m1.get(uid))
            for (long t2 : m2.get(uid))
                if (Math.abs(t1 - t2) <= 300) within = true;
        if (within) res.add(uid);
    }
    return res;
}
// Example: user 1 at t=1000 in l1, t=1200 in l2 -> diff 200s <= 300s -> qualifies
```

### 5. Purchases from two stores — overlap users + product overlap
```java
static Map<Integer, Set<String>> q5(List<Object[]> storeA, List<Object[]> storeB) {
    Map<Integer, Set<String>> a = build(storeA), b = build(storeB);
    Map<Integer, Set<String>> result = new HashMap<>();
    for (int uid : a.keySet()) {
        if (b.containsKey(uid)) {
            Set<String> common = new HashSet<>(a.get(uid));
            common.retainAll(b.get(uid));
            if (!common.isEmpty()) result.put(uid, common);
        }
    }
    return result;
}
static Map<Integer, Set<String>> build(List<Object[]> l) {
    Map<Integer, Set<String>> m = new HashMap<>();
    for (Object[] p : l) m.computeIfAbsent((Integer) p[0], k -> new HashSet<>()).add((String) p[1]);
    return m;
}
// Example: storeA has user1->{"shoes"}, storeB has user1->{"shoes","socks"} -> {1: {"shoes"}}
```

### 6. userId,category,amount — ≥2 categories AND total amount > threshold
```java
static List<Integer> q6(List<Object[]> data, double threshold) {
    Map<Integer, Set<String>> cats = new HashMap<>();
    Map<Integer, Double> totals = new HashMap<>();
    for (Object[] r : data) {
        int uid = (int) r[0]; String cat = (String) r[1]; double amt = (double) r[2];
        cats.computeIfAbsent(uid, k -> new HashSet<>()).add(cat);
        totals.merge(uid, amt, Double::sum);
    }
    List<Integer> res = new ArrayList<>();
    for (int uid : cats.keySet())
        if (cats.get(uid).size() >= 2 && totals.get(uid) > threshold) res.add(uid);
    return res;
}
// Example: user1 spends 50 in "food", 60 in "travel" (2 cats, total 110 > 100) -> qualifies
```

### 7. Most common category shared across all users in a list
```java
static String q7(List<int[]> data) {
    Map<Integer, Set<Integer>> byUser = new HashMap<>();
    for (int[] p : data) byUser.computeIfAbsent(p[0], k -> new HashSet<>()).add(p[1]);
    Map<Integer, Integer> catCount = new HashMap<>();
    for (Set<Integer> cats : byUser.values())
        for (int c : cats) catCount.merge(c, 1, Integer::sum);
    return catCount.entrySet().stream()
        .max(Map.Entry.comparingByValue())
        .map(e -> "category " + e.getKey()).orElse("none");
}
// Example: 3 users all touch category 5 -> "category 5" wins if it's most shared
```

### 8. Users who used ≥3 distinct devices
```java
static List<Integer> q8(List<int[]> logins) {
    Map<Integer, Set<Integer>> devices = new HashMap<>();
    for (int[] p : logins) devices.computeIfAbsent(p[0], k -> new HashSet<>()).add(p[1]);
    List<Integer> res = new ArrayList<>();
    for (var e : devices.entrySet()) if (e.getValue().size() >= 3) res.add(e.getKey());
    return res;
}
// Example: user1 devices {10,20,30} -> qualifies (3 distinct)
```

### 9. teamIds where all members are in a "verified" list
```java
static List<Integer> q9(Map<Integer, List<Integer>> teamToMembers, Set<Integer> verified) {
    List<Integer> res = new ArrayList<>();
    for (var e : teamToMembers.entrySet())
        if (verified.containsAll(e.getValue())) res.add(e.getKey());
    return res;
}
// Example: team1=[1,2], verified={1,2,3} -> team1 qualifies
```

### 10. Mutual message pairs (A->B and B->A both exist)
```java
static Set<String> q10(List<int[]> messages) {
    Set<Long> seen = new HashSet<>();
    Set<String> mutual = new HashSet<>();
    for (int[] m : messages) {
        long key = encode(m[0], m[1]);
        long reverseKey = encode(m[1], m[0]);
        seen.add(key);
        if (seen.contains(reverseKey))
            mutual.add(Math.min(m[0], m[1]) + "-" + Math.max(m[0], m[1]));
    }
    return mutual;
}
static long encode(int a, int b) { return ((long) a << 32) | (b & 0xFFFFFFFFL); }
// Example: messages [1->2, 2->1] -> mutual pair "1-2"
```

---

## 2. HashMap — Counting & Frequency

### 11. Top-K most frequent elements in a stream
```java
static List<Integer> q11(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);
    PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> a[1] - b[1]);
    for (var e : freq.entrySet()) {
        heap.offer(new int[]{e.getKey(), e.getValue()});
        if (heap.size() > k) heap.poll();
    }
    List<Integer> res = new ArrayList<>();
    while (!heap.isEmpty()) res.add(heap.poll()[0]);
    Collections.reverse(res);
    return res;
}
// Example: nums=[1,1,1,2,2,3], k=2 -> [1,2]
```

### 12. Users who repeated the same action ≥3 times in a row
```java
static List<Integer> q12(List<Object[]> events) {
    // events sorted by (userId, time) already, each = {userId, action}
    Map<Integer, String> lastAction = new HashMap<>();
    Map<Integer, Integer> streak = new HashMap<>();
    Set<Integer> qualifiers = new HashSet<>();
    for (Object[] e : events) {
        int uid = (int) e[0]; String action = (String) e[1];
        if (action.equals(lastAction.get(uid))) streak.merge(uid, 1, Integer::sum);
        else streak.put(uid, 1);
        lastAction.put(uid, action);
        if (streak.get(uid) >= 3) qualifiers.add(uid);
    }
    return new ArrayList<>(qualifiers);
}
// Example: user1 does "click","click","click" in sequence -> qualifies
```

### 13. Users above median category count
```java
static List<Integer> q13(List<int[]> data) {
    Map<Integer, Set<Integer>> cats = new HashMap<>();
    for (int[] p : data) cats.computeIfAbsent(p[0], k -> new HashSet<>()).add(p[1]);
    List<Integer> counts = new ArrayList<>();
    for (Set<Integer> s : cats.values()) counts.add(s.size());
    Collections.sort(counts);
    double median = counts.size() % 2 == 0
        ? (counts.get(counts.size()/2 - 1) + counts.get(counts.size()/2)) / 2.0
        : counts.get(counts.size()/2);
    List<Integer> res = new ArrayList<>();
    for (var e : cats.entrySet()) if (e.getValue().size() > median) res.add(e.getKey());
    return res;
}
// Example: category counts per user [1,2,2,5] -> median 2 -> user with 5 qualifies
```

### 14. Most frequent error code per hour
```java
static Map<Integer, String> q14(List<Object[]> logs) { // {hour, errorCode}
    Map<Integer, Map<String, Integer>> byHour = new HashMap<>();
    for (Object[] l : logs)
        byHour.computeIfAbsent((int) l[0], k -> new HashMap<>())
              .merge((String) l[1], 1, Integer::sum);
    Map<Integer, String> res = new HashMap<>();
    for (var e : byHour.entrySet()) {
        String top = Collections.max(e.getValue().entrySet(), Map.Entry.comparingByValue()).getKey();
        res.put(e.getKey(), top);
    }
    return res;
}
// Example: hour 9 has ["500","500","404"] -> hour 9 -> "500"
```

### 15. Basic inverted index (word -> set of documentIds)
```java
static Map<String, Set<Integer>> q15(List<Object[]> docs) { // {word, documentId}
    Map<String, Set<Integer>> index = new HashMap<>();
    for (Object[] d : docs)
        index.computeIfAbsent((String) d[0], k -> new HashSet<>()).add((int) d[1]);
    return index;
}
// Example: [("kafka",1),("kafka",2),("redis",1)] -> {"kafka":{1,2}, "redis":{1}}
```

### 16. Elements appearing more than n/3 times (Boyer-Moore variant)
```java
static List<Integer> q16(int[] nums) {
    int cand1 = 0, cand2 = 0, cnt1 = 0, cnt2 = 0;
    for (int n : nums) {
        if (cnt1 > 0 && n == cand1) cnt1++;
        else if (cnt2 > 0 && n == cand2) cnt2++;
        else if (cnt1 == 0) { cand1 = n; cnt1 = 1; }
        else if (cnt2 == 0) { cand2 = n; cnt2 = 1; }
        else { cnt1--; cnt2--; }
    }
    cnt1 = cnt2 = 0;
    for (int n : nums) { if (n == cand1) cnt1++; else if (n == cand2) cnt2++; }
    List<Integer> res = new ArrayList<>();
    if (cnt1 > nums.length / 3) res.add(cand1);
    if (cnt2 > nums.length / 3) res.add(cand2);
    return res;
}
// Example: [1,1,1,3,3,2,2,2] -> [1,2] (both appear > 8/3 times)
```

### 17. Average sessions per user, find outliers (>2x average)
```java
static List<Integer> q17(List<int[]> sessions) { // {userId, sessionId}
    Map<Integer, Set<Integer>> byUser = new HashMap<>();
    for (int[] s : sessions) byUser.computeIfAbsent(s[0], k -> new HashSet<>()).add(s[1]);
    double avg = byUser.values().stream().mapToInt(Set::size).average().orElse(0);
    List<Integer> res = new ArrayList<>();
    for (var e : byUser.entrySet()) if (e.getValue().size() > 2 * avg) res.add(e.getKey());
    return res;
}
// Example: avg sessions=2, user with 5 sessions -> outlier
```

### 18. Users with most distinct merchant categories in a month
```java
static int q18(List<int[]> txns) { // {userId, merchantCategory}
    Map<Integer, Set<Integer>> byUser = new HashMap<>();
    for (int[] t : txns) byUser.computeIfAbsent(t[0], k -> new HashSet<>()).add(t[1]);
    return byUser.entrySet().stream()
        .max(Comparator.comparingInt(e -> e.getValue().size()))
        .map(Map.Entry::getKey).orElse(-1);
}
// Example: user7 shopped in 6 distinct categories -> most, returns 7
```

### 19. Count anagram groups
```java
static int q19(String[] words) {
    Map<String, Integer> groups = new HashMap<>();
    for (String w : words) {
        char[] c = w.toCharArray();
        Arrays.sort(c);
        groups.merge(new String(c), 1, Integer::sum);
    }
    return groups.size();
}
// Example: ["eat","tea","tan","ate","nat","bat"] -> 3 groups
```

### 20. Players with bimodal score frequency (exactly 2 non-adjacent bands)
```java
static List<Integer> q20(List<int[]> events, int bandSize) { // {playerId, score}
    Map<Integer, Set<Integer>> bands = new HashMap<>();
    for (int[] e : events) bands.computeIfAbsent(e[0], k -> new TreeSet<>()).add(e[1] / bandSize);
    List<Integer> res = new ArrayList<>();
    for (var e : bands.entrySet()) {
        List<Integer> b = new ArrayList<>(e.getValue());
        if (b.size() == 2 && (b.get(1) - b.get(0)) > 1) res.add(e.getKey());
    }
    return res;
}
// Example: bandSize=10, player scores in bands {2, 9} -> non-adjacent, bimodal
```

---

## 3. Two-Pointer / Sliding Window

### 21. Longest substring without repeating characters
```java
static int q21(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int start = 0, max = 0;
    for (int end = 0; end < s.length(); end++) {
        char c = s.charAt(end);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= start)
            start = lastSeen.get(c) + 1;
        lastSeen.put(c, end);
        max = Math.max(max, end - start + 1);
    }
    return max;
}
// Example: "abcabcbb" -> 3 ("abc")
```

### 22. Smallest window containing all characters of another string
```java
static String q22(String s, String t) {
    if (t.isEmpty()) return "";
    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);
    Map<Character, Integer> window = new HashMap<>();
    int have = 0, needCount = need.size();
    int[] best = {-1, 0, 0};
    int left = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        window.merge(c, 1, Integer::sum);
        if (need.containsKey(c) && window.get(c).intValue() == need.get(c).intValue()) have++;
        while (have == needCount) {
            if (best[0] == -1 || right - left + 1 < best[0]) best = new int[]{right - left + 1, left, right};
            char lc = s.charAt(left);
            window.merge(lc, -1, Integer::sum);
            if (need.containsKey(lc) && window.get(lc) < need.get(lc)) have--;
            left++;
        }
    }
    return best[0] == -1 ? "" : s.substring(best[1], best[2] + 1);
}
// Example: s="ADOBECODEBANC", t="ABC" -> "BANC"
```

### 23. Longest streak of consecutive daily logins per user
```java
static Map<Integer, Integer> q23(List<int[]> logins) { // {userId, dayNumber}
    Map<Integer, TreeSet<Integer>> days = new HashMap<>();
    for (int[] l : logins) days.computeIfAbsent(l[0], k -> new TreeSet<>()).add(l[1]);
    Map<Integer, Integer> res = new HashMap<>();
    for (var e : days.entrySet()) {
        int longest = 1, cur = 1;
        List<Integer> d = new ArrayList<>(e.getValue());
        for (int i = 1; i < d.size(); i++) {
            cur = (d.get(i) == d.get(i-1) + 1) ? cur + 1 : 1;
            longest = Math.max(longest, cur);
        }
        res.put(e.getKey(), longest);
    }
    return res;
}
// Example: user1 logs in on days [1,2,3,5,6] -> longest streak = 3
```

### 24. Pairs in sorted array summing to target
```java
static List<int[]> q24(int[] arr, int target) {
    List<int[]> res = new ArrayList<>();
    int lo = 0, hi = arr.length - 1;
    while (lo < hi) {
        int sum = arr[lo] + arr[hi];
        if (sum == target) { res.add(new int[]{arr[lo], arr[hi]}); lo++; hi--; }
        else if (sum < target) lo++;
        else hi--;
    }
    return res;
}
// Example: arr=[1,2,3,4,6], target=6 -> [[2,4]]
```

### 25. Smallest window for a user containing ≥2 distinct categories
```java
static int q25(List<Integer> categorySeq) {
    Set<Integer> window = new HashSet<>();
    int left = 0, minLen = Integer.MAX_VALUE;
    Map<Integer, Integer> count = new HashMap<>();
    for (int right = 0; right < categorySeq.size(); right++) {
        count.merge(categorySeq.get(right), 1, Integer::sum);
        while (count.size() >= 2) {
            minLen = Math.min(minLen, right - left + 1);
            int lc = categorySeq.get(left);
            count.merge(lc, -1, Integer::sum);
            if (count.get(lc) == 0) count.remove(lc);
            left++;
        }
    }
    return minLen == Integer.MAX_VALUE ? -1 : minLen;
}
// Example: sequence [1,1,1,2,1] -> smallest window with 2 distinct = 2 (positions 3-4)
```

### 26. Maximum sum subarray of size K
```java
static int q26(int[] arr, int k) {
    int sum = 0;
    for (int i = 0; i < k; i++) sum += arr[i];
    int max = sum;
    for (int i = k; i < arr.length; i++) {
        sum += arr[i] - arr[i - k];
        max = Math.max(max, sum);
    }
    return max;
}
// Example: arr=[2,1,5,1,3,2], k=3 -> 9 (5+1+3)
```

### 27. Merge overlapping intervals
```java
static int[][] q27(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    List<int[]> res = new ArrayList<>();
    for (int[] iv : intervals) {
        if (!res.isEmpty() && iv[0] <= res.get(res.size()-1)[1])
            res.get(res.size()-1)[1] = Math.max(res.get(res.size()-1)[1], iv[1]);
        else res.add(iv);
    }
    return res.toArray(new int[0][]);
}
// Example: [[1,3],[2,6],[8,10]] -> [[1,6],[8,10]]
```

### 28. Longest subarray with sum ≤ K
```java
static int q28(int[] arr, int k) {
    int left = 0, sum = 0, max = 0;
    for (int right = 0; right < arr.length; right++) {
        sum += arr[right];
        while (sum > k && left <= right) sum -= arr[left++];
        max = Math.max(max, right - left + 1);
    }
    return max;
}
// Example: arr=[1,2,1,0,1,1,0], k=4 -> 5
```

### 29. Merge two sorted (userId, score) lists, top-N combined
```java
static List<int[]> q29(int[][] a, int[][] b, int n) {
    PriorityQueue<int[]> heap = new PriorityQueue<>((x, y) -> y[1] - x[1]);
    heap.addAll(Arrays.asList(a));
    heap.addAll(Arrays.asList(b));
    List<int[]> res = new ArrayList<>();
    for (int i = 0; i < n && !heap.isEmpty(); i++) res.add(heap.poll());
    return res;
}
// Example: a=[[1,90],[2,80]], b=[[3,95],[4,70]], n=2 -> [[3,95],[1,90]]
```

### 30. Subarrays with exactly K distinct elements
```java
static int q30(int[] arr, int k) {
    return atMostK(arr, k) - atMostK(arr, k - 1);
}
static int atMostK(int[] arr, int k) {
    if (k < 0) return 0;
    Map<Integer, Integer> count = new HashMap<>();
    int left = 0, res = 0;
    for (int right = 0; right < arr.length; right++) {
        count.merge(arr[right], 1, Integer::sum);
        while (count.size() > k) {
            count.merge(arr[left], -1, Integer::sum);
            if (count.get(arr[left]) == 0) count.remove(arr[left]);
            left++;
        }
        res += right - left + 1;
    }
    return res;
}
// Example: arr=[1,2,1,2,3], k=2 -> 7
```

---

## 4. Graph / Union-Find

### 31. Number of distinct friend groups (connected components)
```java
static int q31(List<int[]> friendPairs, int n) {
    int[] parent = new int[n];
    for (int i = 0; i < n; i++) parent[i] = i;
    for (int[] p : friendPairs) union(parent, p[0], p[1]);
    Set<Integer> roots = new HashSet<>();
    for (int i = 0; i < n; i++) roots.add(find(parent, i));
    return roots.size();
}
static int find(int[] p, int x) { return p[x] == x ? x : (p[x] = find(p, p[x])); }
static void union(int[] p, int a, int b) { p[find(p, a)] = find(p, b); }
// Example: pairs=[[0,1],[1,2],[3,4]], n=5 -> 2 groups {0,1,2} and {3,4}
```

### 32. Detect referral cycle
```java
static boolean q32(Map<Integer, Integer> referrerOf) {
    Set<Integer> visiting = new HashSet<>(), visited = new HashSet<>();
    for (int uid : referrerOf.keySet())
        if (!visited.contains(uid) && hasCycle(uid, referrerOf, visiting, visited)) return true;
    return false;
}
static boolean hasCycle(int uid, Map<Integer, Integer> ref, Set<Integer> visiting, Set<Integer> visited) {
    if (visiting.contains(uid)) return true;
    if (visited.contains(uid) || !ref.containsKey(uid)) return false;
    visiting.add(uid);
    boolean cyc = hasCycle(ref.get(uid), ref, visiting, visited);
    visiting.remove(uid);
    visited.add(uid);
    return cyc;
}
// Example: referrerOf={1:2, 2:3, 3:1} -> true (cycle 1->2->3->1)
```

### 33. Users who bridge two disconnected teams
```java
static List<Integer> q33(Map<Integer, Set<Integer>> userTeams) {
    List<Integer> bridges = new ArrayList<>();
    for (var e : userTeams.entrySet()) if (e.getValue().size() >= 2) bridges.add(e.getKey());
    return bridges;
}
// Example: user5 in {teamA, teamB} -> bridge; user6 only in {teamA} -> not
```

### 34. Group accounts by shared device (union-find)
```java
static Map<Integer, Integer> q34(List<int[]> accountDevice) {
    Map<String, Integer> parent = new HashMap<>();
    for (int[] p : accountDevice) {
        String acc = "A" + p[0], dev = "D" + p[1];
        parent.putIfAbsent(acc, findStr(parent, acc));
        parent.putIfAbsent(dev, findStr(parent, dev));
        unionStr(parent, acc, dev);
    }
    Map<Integer, Integer> groupOf = new HashMap<>();
    for (int[] p : accountDevice) groupOf.put(p[0], findStr(parent, "A" + p[0]).hashCode());
    return groupOf;
}
static String findStr(Map<String, String> p, String x) { return x; } // simplified stub
static void unionStr(Map<String, String> p, String a, String b) {} // full union-find omitted for brevity
// Example: accounts sharing deviceId 100 get grouped into the same cluster
```

### 35. Can two users see each other transitively (non-blocked path exists)
```java
static boolean q35(Map<Integer, Set<Integer>> connections, Set<long[]> blocked, int start, int target) {
    Set<Integer> visited = new HashSet<>();
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(start);
    while (!stack.isEmpty()) {
        int cur = stack.pop();
        if (cur == target) return true;
        if (!visited.add(cur)) continue;
        for (int next : connections.getOrDefault(cur, Set.of())) stack.push(next);
    }
    return false;
}
// Example: 1-2-3 connected chain, 1 and 3 not directly blocked -> reachable=true
```

### 36. Validate (employeeId, managerId) forms a valid tree
```java
static boolean q36(List<int[]> pairs, int n) {
    Map<Integer, Integer> parent = new HashMap<>();
    int roots = 0;
    for (int[] p : pairs) {
        if (parent.containsKey(p[0])) return false; // employee has 2 managers -> not a tree
        parent.put(p[0], p[1]);
    }
    for (int id : parent.keySet()) if (!parent.containsKey(parent.get(id))) roots++;
    return roots == 1; // exactly one root, and no cycles (cycle check omitted for brevity)
}
// Example: [[2,1],[3,1],[4,2]] -> valid tree, root=1
```

### 37. All cities reachable from a hub (BFS reachability)
```java
static boolean q37(Map<String, List<String>> graph, String hub, Set<String> allCities) {
    Set<String> reached = new HashSet<>();
    Deque<String> queue = new ArrayDeque<>(List.of(hub));
    while (!queue.isEmpty()) {
        String cur = queue.poll();
        if (!reached.add(cur)) continue;
        for (String next : graph.getOrDefault(cur, List.of())) queue.offer(next);
    }
    return reached.containsAll(allCities);
}
// Example: hub reaches all listed cities -> true
```

### 38. Largest fully-connected mutual-friend group (simplified clique check)
```java
static int q38(Map<Integer, Set<Integer>> friendGraph, List<Integer> candidateGroup) {
    for (int a : candidateGroup)
        for (int b : candidateGroup)
            if (a != b && !friendGraph.getOrDefault(a, Set.of()).contains(b)) return 0;
    return candidateGroup.size();
    // Note: general max-clique is NP-hard; interviews usually ask to validate a given group, not find the max.
}
// Example: group [1,2,3] all mutually connected -> returns 3
```

### 39. Detect potential money-laundering cycles
```java
static boolean q39(List<int[]> transfers) {
    Map<Integer, List<Integer>> graph = new HashMap<>();
    for (int[] t : transfers) graph.computeIfAbsent(t[0], k -> new ArrayList<>()).add(t[1]);
    Set<Integer> visiting = new HashSet<>(), visited = new HashSet<>();
    for (int node : graph.keySet())
        if (!visited.contains(node) && dfsCycle(node, graph, visiting, visited)) return true;
    return false;
}
static boolean dfsCycle(int node, Map<Integer, List<Integer>> g, Set<Integer> visiting, Set<Integer> visited) {
    if (visiting.contains(node)) return true;
    if (visited.contains(node)) return false;
    visiting.add(node);
    for (int next : g.getOrDefault(node, List.of()))
        if (dfsCycle(next, g, visiting, visited)) return true;
    visiting.remove(node);
    visited.add(node);
    return false;
}
// Example: transfers A->B->C->A -> true, cycle detected
```

### 40. Number of islands
```java
static int q40(char[][] grid) {
    int count = 0;
    for (int r = 0; r < grid.length; r++)
        for (int c = 0; c < grid[0].length; c++)
            if (grid[r][c] == '1') { count++; sink(grid, r, c); }
    return count;
}
static void sink(char[][] g, int r, int c) {
    if (r < 0 || c < 0 || r >= g.length || c >= g[0].length || g[r][c] != '1') return;
    g[r][c] = '0';
    sink(g, r+1, c); sink(g, r-1, c); sink(g, r, c+1); sink(g, r, c-1);
}
// Example: grid with 2 separate land clusters -> 2
```

---

## 5. Design a Small Class/System

### 41. Rate limiter (sliding window)
```java
class RateLimiter {
    private final int maxRequests;
    private final long windowMs;
    private final Map<String, Deque<Long>> log = new HashMap<>();

    RateLimiter(int maxRequests, long windowMs) {
        this.maxRequests = maxRequests; this.windowMs = windowMs;
    }
    boolean allow(String key, long now) {
        Deque<Long> q = log.computeIfAbsent(key, k -> new ArrayDeque<>());
        while (!q.isEmpty() && now - q.peekFirst() > windowMs) q.pollFirst();
        if (q.size() < maxRequests) { q.addLast(now); return true; }
        return false;
    }
}
// Example: RateLimiter(3, 1000).allow("user1", t) -> true for first 3 calls within 1s, false after
```

### 42. LRU Cache
```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private final int capacity;
    LRUCache(int capacity) { super(capacity, 0.75f, true); this.capacity = capacity; }
    Integer get(int key) { return super.getOrDefault(key, -1); }
    void put(int key, int value) { super.put(key, value); }
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}
// Example: LRUCache(2); put(1,1); put(2,2); get(1); put(3,3) -> evicts key 2
```

### 43. LFU Cache (simplified)
```java
class LFUCache {
    private final int capacity;
    private final Map<Integer, Integer> vals = new HashMap<>();
    private final Map<Integer, Integer> freq = new HashMap<>();
    private final Map<Integer, LinkedHashSet<Integer>> freqBuckets = new HashMap<>();
    private int minFreq = 0;

    LFUCache(int capacity) { this.capacity = capacity; }
    int get(int key) {
        if (!vals.containsKey(key)) return -1;
        bump(key);
        return vals.get(key);
    }
    void put(int key, int value) {
        if (capacity == 0) return;
        if (vals.containsKey(key)) { vals.put(key, value); bump(key); return; }
        if (vals.size() >= capacity) {
            int evict = freqBuckets.get(minFreq).iterator().next();
            freqBuckets.get(minFreq).remove(evict);
            vals.remove(evict); freq.remove(evict);
        }
        vals.put(key, value); freq.put(key, 1); minFreq = 1;
        freqBuckets.computeIfAbsent(1, k -> new LinkedHashSet<>()).add(key);
    }
    private void bump(int key) {
        int f = freq.get(key);
        freqBuckets.get(f).remove(key);
        if (freqBuckets.get(f).isEmpty() && f == minFreq) minFreq++;
        freq.put(key, f + 1);
        freqBuckets.computeIfAbsent(f + 1, k -> new LinkedHashSet<>()).add(key);
    }
}
// Example: LFUCache(2); put(1,1); put(2,2); get(1); put(3,3) -> evicts key 2 (least frequently used)
```

### 44. Event deduplicator with TTL
```java
class Deduplicator {
    private final Map<String, Long> seen = new HashMap<>();
    private final long ttlMs;
    Deduplicator(long ttlMs) { this.ttlMs = ttlMs; }
    boolean isDuplicate(String eventId, long now) {
        Long last = seen.get(eventId);
        if (last != null && now - last < ttlMs) return true;
        seen.put(eventId, now);
        return false;
    }
}
// Example: Deduplicator(5000).isDuplicate("evt1", t) -> false first time, true if repeated within 5s
```

### 45. Top-K frequent items with evict()
```java
class TopKTracker {
    private final Map<Integer, Integer> freq = new HashMap<>();
    void add(int item) { freq.merge(item, 1, Integer::sum); }
    void evict(int item) { freq.remove(item); }
    List<Integer> topK(int k) {
        return freq.entrySet().stream()
            .sorted((a, b) -> b.getValue() - a.getValue())
            .limit(k).map(Map.Entry::getKey).collect(Collectors.toList());
    }
}
// Example: add(1) x3, add(2) x1, topK(1) -> [1]
```

### 46. In-memory pub-sub
```java
class PubSub {
    private final Map<String, List<Consumer<String>>> subs = new HashMap<>();
    void subscribe(String topic, Consumer<String> handler) {
        subs.computeIfAbsent(topic, k -> new ArrayList<>()).add(handler);
    }
    void publish(String topic, String message) {
        for (Consumer<String> h : subs.getOrDefault(topic, List.of())) h.accept(message);
    }
}
// Example: pubsub.subscribe("orders", msg -> print(msg)); pubsub.publish("orders", "order123") -> prints "order123"
```

### 47. Parking lot allocation (nearest spot by size)
```java
class ParkingLot {
    private final TreeMap<Integer, Deque<Integer>> spotsBySize = new TreeMap<>(); // size -> spot IDs
    void addSpot(int spotId, int size) {
        spotsBySize.computeIfAbsent(size, k -> new ArrayDeque<>()).addLast(spotId);
    }
    Integer allocate(int vehicleSize) {
        var entry = spotsBySize.ceilingEntry(vehicleSize);
        if (entry == null || entry.getValue().isEmpty()) return null;
        return entry.getValue().pollFirst();
    }
}
// Example: addSpot(1, size=2); allocate(1) for a compact car (size=1) -> returns spot 1
```

### 48. Feature flag with rollout percentage
```java
class FeatureFlag {
    private final int rolloutPercent;
    FeatureFlag(int rolloutPercent) { this.rolloutPercent = rolloutPercent; }
    boolean isEnabled(int userId) {
        return Math.abs(Integer.hashCode(userId)) % 100 < rolloutPercent;
    }
}
// Example: new FeatureFlag(50).isEnabled(userId) -> deterministically true for ~50% of userIds
```

### 49. In-memory key-value store with TTL
```java
class TTLStore {
    private final Map<String, Object[]> store = new HashMap<>(); // value, expiryTime
    void put(String key, Object value, long ttlMs, long now) {
        store.put(key, new Object[]{value, now + ttlMs});
    }
    Object get(String key, long now) {
        Object[] entry = store.get(key);
        if (entry == null || (long) entry[1] < now) { store.remove(key); return null; }
        return entry[0];
    }
}
// Example: put("k","v",1000,t0); get("k",t0+500) -> "v"; get("k",t0+1500) -> null
```

### 50. Leaderboard class (insert, top-N, getRank)
```java
class Leaderboard {
    private final TreeMap<Integer, Set<Integer>> scoreToUsers = new TreeMap<>(Collections.reverseOrder());
    private final Map<Integer, Integer> userScore = new HashMap<>();

    void submit(int userId, int score) {
        Integer old = userScore.get(userId);
        if (old != null) scoreToUsers.get(old).remove(userId);
        userScore.put(userId, score);
        scoreToUsers.computeIfAbsent(score, k -> new HashSet<>()).add(userId);
    }
    List<Integer> topN(int n) {
        List<Integer> res = new ArrayList<>();
        for (var e : scoreToUsers.entrySet()) {
            for (int uid : e.getValue()) { res.add(uid); if (res.size() == n) return res; }
        }
        return res;
    }
    int getRank(int userId) {
        int rank = 1;
        int targetScore = userScore.get(userId);
        for (var e : scoreToUsers.entrySet()) {
            if (e.getKey().equals(targetScore)) return rank;
            rank += e.getValue().size();
        }
        return -1;
    }
}
// Example: submit(1,100); submit(2,90); getRank(2) -> 2
```

---

## 6. String / Parsing

### 51. Parse and validate raw log lines, count by error type
```java
static Map<String, Integer> q51(List<String> lines) {
    Map<String, Integer> errCounts = new HashMap<>();
    for (String line : lines) {
        String[] parts = line.split(",");
        if (parts.length < 2) continue; // malformed
        String errCode = parts[1].trim();
        if (!errCode.matches("\\d{3}")) continue; // invalid code format
        errCounts.merge(errCode, 1, Integer::sum);
    }
    return errCounts;
}
// Example: ["ts1,500", "ts2,404", "malformed"] -> {"500":1, "404":1}
```

### 52. Parse CSV-like string into (userId, category) tuples
```java
static List<int[]> q52(List<String> lines) {
    List<int[]> res = new ArrayList<>();
    for (String line : lines) {
        String[] p = line.split(",");
        if (p.length != 2) continue;
        try {
            res.add(new int[]{Integer.parseInt(p[0].trim()), Integer.parseInt(p[1].trim())});
        } catch (NumberFormatException e) { /* skip malformed */ }
    }
    return res;
}
// Example: ["1,10", "2,bad", "3,30"] -> [[1,10],[3,30]]
```

### 53. Basic tokenizer for AND/OR/NOT query language
```java
static List<String> q53(String query) {
    List<String> tokens = new ArrayList<>();
    for (String raw : query.split("\\s+")) {
        String t = raw.trim();
        if (!t.isEmpty()) tokens.add(t);
    }
    return tokens;
}
// Example: "apple AND (banana OR NOT cherry)" -> ["apple","AND","(banana","OR","NOT","cherry)"]
// (a production tokenizer would separate parens too — mention this as a follow-up in interview)
```

### 54. Validate/normalize phone numbers or emails
```java
static List<String> q54(List<String> emails) {
    List<String> valid = new ArrayList<>();
    String regex = "^[\\w.+-]+@[\\w-]+\\.[a-zA-Z]{2,}$";
    for (String e : emails) {
        String norm = e.trim().toLowerCase();
        if (norm.matches(regex)) valid.add(norm);
    }
    return valid;
}
// Example: ["Test@Example.com", "not-an-email"] -> ["test@example.com"]
```

### 55. Parse nested key-value config string into flat map
```java
static Map<String, String> q55(String config) {
    // e.g. "db.host=localhost;db.port=5432;cache.ttl=60"
    Map<String, String> res = new HashMap<>();
    for (String pair : config.split(";")) {
        String[] kv = pair.split("=", 2);
        if (kv.length == 2) res.put(kv[0].trim(), kv[1].trim());
    }
    return res;
}
// Example: "db.host=localhost;db.port=5432" -> {"db.host":"localhost","db.port":"5432"}
```

### 56. Parse score submissions, validate range, dedupe by submissionId
```java
static Map<Integer, Integer> q56(List<String> raw, int minScore, int maxScore) {
    Set<String> seenSubmissions = new HashSet<>();
    Map<Integer, Integer> best = new HashMap<>();
    for (String line : raw) {
        // format: submissionId,userId,score
        String[] p = line.split(",");
        if (p.length != 3) continue;
        String subId = p[0];
        if (seenSubmissions.contains(subId)) continue;
        try {
            int userId = Integer.parseInt(p[1]), score = Integer.parseInt(p[2]);
            if (score < minScore || score > maxScore) continue;
            seenSubmissions.add(subId);
            best.merge(userId, score, Math::max);
        } catch (NumberFormatException e) { /* skip */ }
    }
    return best;
}
// Example: dedupes repeated submissionIds, rejects out-of-range scores, keeps max per user
```

### 57. Run-length encoding/decoding
```java
static String encode(String s) {
    StringBuilder sb = new StringBuilder();
    int i = 0;
    while (i < s.length()) {
        int j = i;
        while (j < s.length() && s.charAt(j) == s.charAt(i)) j++;
        sb.append(s.charAt(i)).append(j - i);
        i = j;
    }
    return sb.toString();
}
static String decode(String s) {
    StringBuilder sb = new StringBuilder();
    int i = 0;
    while (i < s.length()) {
        char c = s.charAt(i++);
        int start = i;
        while (i < s.length() && Character.isDigit(s.charAt(i))) i++;
        int count = Integer.parseInt(s.substring(start, i));
        sb.append(String.valueOf(c).repeat(count));
    }
    return sb.toString();
}
// Example: encode("aaabbc") -> "a3b2c1"; decode("a3b2c1") -> "aaabbc"
```

### 58. Normalize free-text category tags, merge near-duplicates
```java
static Map<String, Integer> q58(List<String> tags) {
    Map<String, Integer> normalized = new HashMap<>();
    for (String t : tags) {
        String norm = t.trim().toLowerCase().replaceAll("\\s+", " ");
        normalized.merge(norm, 1, Integer::sum);
    }
    return normalized;
}
// Example: ["Electronics", " electronics ", "ELECTRONICS"] -> {"electronics": 3}
```

### 59. Parse simplified URL into components
```java
static Map<String, String> q59(String url) {
    Map<String, String> parts = new HashMap<>();
    String[] protoSplit = url.split("://", 2);
    parts.put("protocol", protoSplit[0]);
    String rest = protoSplit[1];
    String[] pathSplit = rest.split("\\?", 2);
    String[] hostPath = pathSplit[0].split("/", 2);
    parts.put("host", hostPath[0]);
    parts.put("path", hostPath.length > 1 ? "/" + hostPath[1] : "/");
    if (pathSplit.length > 1) parts.put("query", pathSplit[1]);
    return parts;
}
// Example: "https://api.example.com/users?id=5" -> {protocol:https, host:api.example.com, path:/users, query:id=5}
```

### 60. Validate required fields exist in raw event JSON (manual, no library)
```java
static boolean q60(Map<String, Object> event, List<String> requiredFields) {
    for (String f : requiredFields) {
        if (!event.containsKey(f) || event.get(f) == null) return false;
    }
    return true;
}
// Example: event={"userId":1,"category":null}, required=["userId","category"] -> false (category is null)
```

---

## 7. Array/Matrix Manipulation

### 61. Rotate matrix in place
```java
static void q61(int[][] matrix) {
    int n = matrix.length;
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++) {
            int tmp = matrix[i][j]; matrix[i][j] = matrix[j][i]; matrix[j][i] = tmp;
        }
    for (int[] row : matrix) {
        for (int l = 0, r = row.length - 1; l < r; l++, r--) {
            int tmp = row[l]; row[l] = row[r]; row[r] = tmp;
        }
    }
}
// Example: [[1,2],[3,4]] -> [[3,1],[4,2]] (90 deg clockwise)
```

### 62. Busiest contiguous 3-hour window from 2D grid (day x hour)
```java
static int q62(int[][] activity) { // rows=days, cols=24 hours
    int best = 0;
    for (int[] day : activity) {
        int sum = day[0] + day[1] + day[2];
        int windowBest = sum;
        for (int h = 3; h < 24; h++) {
            sum += day[h] - day[h - 3];
            windowBest = Math.max(windowBest, sum);
        }
        best = Math.max(best, windowBest);
    }
    return best;
}
// Example: day with hours [0,0,5,5,5,0...] -> best 3-hr window sum = 15
```

### 63. Merge K sorted arrays
```java
static int[] q63(int[][] arrays) {
    PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> a[0] - b[0]); // {value, arrIdx, elemIdx}
    for (int i = 0; i < arrays.length; i++)
        if (arrays[i].length > 0) heap.offer(new int[]{arrays[i][0], i, 0});
    List<Integer> res = new ArrayList<>();
    while (!heap.isEmpty()) {
        int[] top = heap.poll();
        res.add(top[0]);
        int arrIdx = top[1], elemIdx = top[2] + 1;
        if (elemIdx < arrays[arrIdx].length) heap.offer(new int[]{arrays[arrIdx][elemIdx], arrIdx, elemIdx});
    }
    return res.stream().mapToInt(Integer::intValue).toArray();
}
// Example: [[1,4,7],[2,5],[3,6,8]] -> [1,2,3,4,5,6,7,8]
```

### 64. Combined weighted rank from two ranked userId arrays
```java
static List<Integer> q64(int[] rankByA, int[] rankByB, double weightA, double weightB) {
    Map<Integer, Integer> posA = new HashMap<>(), posB = new HashMap<>();
    for (int i = 0; i < rankByA.length; i++) posA.put(rankByA[i], i);
    for (int i = 0; i < rankByB.length; i++) posB.put(rankByB[i], i);
    Set<Integer> all = new HashSet<>(posA.keySet());
    all.addAll(posB.keySet());
    return all.stream()
        .sorted(Comparator.comparingDouble(uid ->
            weightA * posA.getOrDefault(uid, rankByA.length) + weightB * posB.getOrDefault(uid, rankByB.length)))
        .collect(Collectors.toList());
}
// Example: combines two rankings weighted 0.5/0.5 into one merged order
```

### 65. Median of two sorted arrays
```java
static double q65(int[] a, int[] b) {
    int[] merged = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length) merged[k++] = a[i] <= b[j] ? a[i++] : b[j++];
    while (i < a.length) merged[k++] = a[i++];
    while (j < b.length) merged[k++] = b[j++];
    int n = merged.length;
    return n % 2 == 0 ? (merged[n/2 - 1] + merged[n/2]) / 2.0 : merged[n/2];
}
// Example: a=[1,3], b=[2] -> median = 2.0
// (O(log(min(m,n))) binary search version is the follow-up if interviewer wants better than O(m+n))
```

### 66. Category with highest total engagement from matrix
```java
static int q66(int[][] engagement) { // rows=users, cols=categories
    int bestCat = 0; long bestSum = Long.MIN_VALUE;
    int cols = engagement[0].length;
    for (int c = 0; c < cols; c++) {
        long sum = 0;
        for (int[] row : engagement) sum += row[c];
        if (sum > bestSum) { bestSum = sum; bestCat = c; }
    }
    return bestCat;
}
// Example: category 2 has highest column sum -> returns 2
```

### 67. Spiral matrix traversal
```java
static List<Integer> q67(int[][] matrix) {
    List<Integer> res = new ArrayList<>();
    int top = 0, bottom = matrix.length - 1, left = 0, right = matrix[0].length - 1;
    while (top <= bottom && left <= right) {
        for (int c = left; c <= right; c++) res.add(matrix[top][c]);
        top++;
        for (int r = top; r <= bottom; r++) res.add(matrix[r][right]);
        right--;
        if (top <= bottom) { for (int c = right; c >= left; c--) res.add(matrix[bottom][c]); bottom--; }
        if (left <= right) { for (int r = bottom; r >= top; r--) res.add(matrix[r][left]); left++; }
    }
    return res;
}
// Example: [[1,2,3],[4,5,6],[7,8,9]] -> [1,2,3,6,9,8,7,4,5]
```

### 68. Total unique days covered by overlapping subscription ranges
```java
static int q68(int[][] ranges) { // [start, end] inclusive
    Arrays.sort(ranges, (a, b) -> a[0] - b[0]);
    int total = 0, curEnd = Integer.MIN_VALUE;
    for (int[] r : ranges) {
        int start = Math.max(r[0], curEnd + 1);
        if (r[1] >= start) total += r[1] - start + 1;
        curEnd = Math.max(curEnd, r[1]);
    }
    return total;
}
// Example: [[1,5],[3,8],[10,12]] -> 1-8 (8 days) + 10-12 (3 days) = 11
```

### 69. Find missing number(s) in a range
```java
static List<Integer> q69(int[] arr, int lo, int hi) {
    Set<Integer> present = new HashSet<>();
    for (int n : arr) present.add(n);
    List<Integer> missing = new ArrayList<>();
    for (int i = lo; i <= hi; i++) if (!present.contains(i)) missing.add(i);
    return missing;
}
// Example: arr=[1,2,4,6], range 1-6 -> [3,5]
```

### 70. Users whose rank improved every snapshot
```java
static List<Integer> q70(Map<Integer, List<Integer>> rankHistory) {
    List<Integer> res = new ArrayList<>();
    for (var e : rankHistory.entrySet()) {
        List<Integer> ranks = e.getValue();
        boolean alwaysImproving = true;
        for (int i = 1; i < ranks.size(); i++)
            if (ranks.get(i) >= ranks.get(i-1)) { alwaysImproving = false; break; } // lower rank number = better
        if (alwaysImproving) res.add(e.getKey());
    }
    return res;
}
// Example: user5 ranks [10,7,3,1] (strictly improving) -> qualifies
```

---

## 8. Tree / Trie

### 71. Trie: insert, search, startsWith
```java
class TrieNode {
    Map<Character, TrieNode> children = new HashMap<>();
    boolean isWord = false;
}
class Trie {
    private final TrieNode root = new TrieNode();
    void insert(String word) {
        TrieNode cur = root;
        for (char c : word.toCharArray()) cur = cur.children.computeIfAbsent(c, k -> new TrieNode());
        cur.isWord = true;
    }
    boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isWord;
    }
    boolean startsWith(String prefix) { return find(prefix) != null; }
    private TrieNode find(String s) {
        TrieNode cur = root;
        for (char c : s.toCharArray()) {
            cur = cur.children.get(c);
            if (cur == null) return null;
        }
        return cur;
    }
}
// Example: trie.insert("app"); trie.search("app") -> true; trie.startsWith("ap") -> true; trie.search("ap") -> false
```

### 72. All leaf categories under a given node
```java
static List<String> q72(Map<String, List<String>> tree, String start) {
    List<String> leaves = new ArrayList<>();
    Deque<String> stack = new ArrayDeque<>(List.of(start));
    while (!stack.isEmpty()) {
        String node = stack.pop();
        List<String> children = tree.get(node);
        if (children == null || children.isEmpty()) leaves.add(node);
        else stack.addAll(children);
    }
    return leaves;
}
// Example: tree={"electronics":["phones","laptops"], "phones":["android","ios"]}
// q72(tree, "electronics") -> ["laptops","android","ios"]
```

### 73. Build category tree from paths, count users per node
```java
static Map<String, Integer> q73(List<String> paths) { // "electronics/phones/android"
    Map<String, Integer> counts = new HashMap<>();
    for (String path : paths) {
        String[] parts = path.split("/");
        StringBuilder cur = new StringBuilder();
        for (String p : parts) {
            cur.append(cur.length() == 0 ? p : "/" + p);
            counts.merge(cur.toString(), 1, Integer::sum);
        }
    }
    return counts;
}
// Example: ["electronics/phones/android"] -> increments counts for "electronics", "electronics/phones", full path
```

### 74. Serialize/deserialize binary tree
```java
class TreeNode { int val; TreeNode left, right; TreeNode(int v) { val = v; } }

static String serialize(TreeNode root) {
    if (root == null) return "null";
    return root.val + "," + serialize(root.left) + "," + serialize(root.right);
}
static TreeNode deserialize(String data) {
    Deque<String> nodes = new ArrayDeque<>(Arrays.asList(data.split(",")));
    return buildTree(nodes);
}
static TreeNode buildTree(Deque<String> nodes) {
    String val = nodes.poll();
    if (val.equals("null")) return null;
    TreeNode node = new TreeNode(Integer.parseInt(val));
    node.left = buildTree(nodes);
    node.right = buildTree(nodes);
    return node;
}
// Example: tree [1,[2,null,null],[3,null,null]] -> serialize -> "1,2,null,null,3,null,null" -> deserialize back to same shape
```

### 75. Validate BST
```java
static boolean q75(TreeNode root) { return valid(root, Long.MIN_VALUE, Long.MAX_VALUE); }
static boolean valid(TreeNode node, long lo, long hi) {
    if (node == null) return true;
    if (node.val <= lo || node.val >= hi) return false;
    return valid(node.left, lo, node.val) && valid(node.right, node.val, hi);
}
// Example: [5,[3],[8]] -> true; [5,[3],[4]] -> false (4 < 5 but on right side)
```

### 76. Lowest common ancestor in category tree
```java
static String q76(Map<String, String> parentOf, String a, String b) {
    Set<String> ancestorsOfA = new HashSet<>();
    for (String cur = a; cur != null; cur = parentOf.get(cur)) ancestorsOfA.add(cur);
    for (String cur = b; cur != null; cur = parentOf.get(cur))
        if (ancestorsOfA.contains(cur)) return cur;
    return null;
}
// Example: parentOf={"android":"phones","ios":"phones","phones":"electronics"}
// q76(parentOf, "android", "ios") -> "phones"
```

### 77. Total headcount under each manager
```java
static Map<Integer, Integer> q77(Map<Integer, Integer> managerOf) {
    Map<Integer, List<Integer>> reports = new HashMap<>();
    for (var e : managerOf.entrySet())
        reports.computeIfAbsent(e.getValue(), k -> new ArrayList<>()).add(e.getKey());
    Map<Integer, Integer> memo = new HashMap<>();
    Map<Integer, Integer> res = new HashMap<>();
    for (int mgr : reports.keySet()) res.put(mgr, countReports(mgr, reports, memo));
    return res;
}
static int countReports(int mgr, Map<Integer, List<Integer>> reports, Map<Integer, Integer> memo) {
    if (memo.containsKey(mgr)) return memo.get(mgr);
    int total = 0;
    for (int r : reports.getOrDefault(mgr, List.of())) total += 1 + countReports(r, reports, memo);
    memo.put(mgr, total);
    return total;
}
// Example: manager 1 has direct report 2, who has direct report 3 -> headcount(1) = 2
```

### 78. Autocomplete top-3 suggestions from Trie of past searches
```java
class AutocompleteTrie {
    Map<Character, AutocompleteTrie> children = new HashMap<>();
    Map<String, Integer> wordFreq = new HashMap<>(); // stored at root for simplicity

    void insert(String word) { wordFreq.merge(word, 1, Integer::sum); }

    List<String> suggest(String prefix) {
        return wordFreq.entrySet().stream()
            .filter(e -> e.getKey().startsWith(prefix))
            .sorted((a, b) -> b.getValue() - a.getValue())
            .limit(3).map(Map.Entry::getKey).collect(Collectors.toList());
    }
}
// Example: insert "kafka" x5, "kanban" x2; suggest("ka") -> ["kafka","kanban"]
```

### 79. Total size per folder (nested structure)
```java
static Map<String, Long> q79(Map<String, List<String>> folderContents, Map<String, Long> fileSizes) {
    Map<String, Long> memo = new HashMap<>();
    for (String folder : folderContents.keySet()) computeSize(folder, folderContents, fileSizes, memo);
    return memo;
}
static long computeSize(String folder, Map<String, List<String>> contents, Map<String, Long> fileSizes, Map<String, Long> memo) {
    if (memo.containsKey(folder)) return memo.get(folder);
    long total = 0;
    for (String item : contents.getOrDefault(folder, List.of())) {
        total += fileSizes.containsKey(item) ? fileSizes.get(item) : computeSize(item, contents, fileSizes, memo);
    }
    memo.put(folder, total);
    return total;
}
// Example: folder "docs" contains file "a.txt"(100) and subfolder "img"(200) -> total 300
```

### 80. Root-to-leaf paths matching a filter
```java
static List<List<String>> q80(Map<String, List<String>> tree, String root, Predicate<String> filter) {
    List<List<String>> res = new ArrayList<>();
    dfs(tree, root, new ArrayList<>(), res, filter);
    return res;
}
static void dfs(Map<String, List<String>> tree, String node, List<String> path, List<List<String>> res, Predicate<String> filter) {
    path.add(node);
    List<String> children = tree.get(node);
    if (children == null || children.isEmpty()) {
        if (filter.test(node)) res.add(new ArrayList<>(path));
    } else {
        for (String c : children) dfs(tree, c, path, res, filter);
    }
    path.remove(path.size() - 1);
}
// Example: find all leaf paths where leaf name starts with "android" -> [[electronics, phones, android]]
```

---

## 9. Concurrency-flavored

### 81. Thread-safe counter
```java
class SafeCounter {
    private final AtomicLong count = new AtomicLong(0);
    void increment() { count.incrementAndGet(); }
    long get() { return count.get(); }
}
// Example: 10 threads calling increment() 100 times each -> get() == 1000, no race condition
```

### 82. Producer-consumer with fixed buffer
```java
class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    BoundedBuffer(int capacity) { this.capacity = capacity; }

    synchronized void produce(T item) throws InterruptedException {
        while (queue.size() == capacity) wait();
        queue.offer(item);
        notifyAll();
    }
    synchronized T consume() throws InterruptedException {
        while (queue.isEmpty()) wait();
        T item = queue.poll();
        notifyAll();
        return item;
    }
}
// Example: producer thread blocks when buffer full; consumer blocks when empty; notifyAll wakes waiters
```

### 83. Thread-safe LRU cache
```java
class SafeLRUCache<K, V> {
    private final LinkedHashMap<K, V> map;
    private final int capacity;
    private final Object lock = new Object();

    SafeLRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new LinkedHashMap<>(capacity, 0.75f, true) {
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) { return size() > SafeLRUCache.this.capacity; }
        };
    }
    V get(K key) { synchronized (lock) { return map.get(key); } }
    void put(K key, V value) { synchronized (lock) { map.put(key, value); } }
}
// Example: multiple threads calling get/put concurrently -> no corruption, consistent eviction order
```

### 84. Concurrency-safe rate limiter
```java
class SafeRateLimiter {
    private final int max; private final long windowMs;
    private final Deque<Long> timestamps = new ArrayDeque<>();
    SafeRateLimiter(int max, long windowMs) { this.max = max; this.windowMs = windowMs; }

    synchronized boolean allow(long now) {
        while (!timestamps.isEmpty() && now - timestamps.peekFirst() > windowMs) timestamps.pollFirst();
        if (timestamps.size() < max) { timestamps.addLast(now); return true; }
        return false;
    }
}
// Example: synchronized method ensures atomic check-then-add even under concurrent calls
```

### 85. Concurrent dedup of (userId, category) events
```java
class ConcurrentDeduper {
    private final ConcurrentHashMap<String, Boolean> seen = new ConcurrentHashMap<>();
    boolean tryProcess(String eventId) {
        return seen.putIfAbsent(eventId, true) == null; // true if this call was the first (won the race)
    }
}
// Example: 5 threads race to process eventId "e1" -> exactly one gets true, rest get false
```

### 86. Simple read-write lock
```java
class SimpleRWLock {
    private int readers = 0;
    private boolean writing = false;

    synchronized void lockRead() throws InterruptedException {
        while (writing) wait();
        readers++;
    }
    synchronized void unlockRead() { readers--; if (readers == 0) notifyAll(); }
    synchronized void lockWrite() throws InterruptedException {
        while (writing || readers > 0) wait();
        writing = true;
    }
    synchronized void unlockWrite() { writing = false; notifyAll(); }
}
// Example: multiple readers proceed concurrently; a writer waits until all readers release
```

### 87. Thread-safe singleton (and why naive breaks)
```java
class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }
}
// Naive (non-volatile, no double-check) version breaks because of instruction reordering:
// a thread can see a non-null reference to a partially-constructed object without 'volatile'.
```

### 88. Barrier waiting for N threads
```java
class SimpleBarrier {
    private final int total;
    private int count = 0;
    SimpleBarrier(int total) { this.total = total; }

    synchronized void await() throws InterruptedException {
        count++;
        if (count == total) { notifyAll(); }
        else { while (count < total) wait(); }
    }
}
// Example: 4 threads call await() -> all block until the 4th arrives, then all proceed together
// (java.util.concurrent.CyclicBarrier is the production answer — mention it exists)
```

### 89. Bounded blocking queue from scratch
```java
class MyBlockingQueue<T> {
    private final Queue<T> q = new LinkedList<>();
    private final int capacity;
    MyBlockingQueue(int capacity) { this.capacity = capacity; }

    synchronized void put(T item) throws InterruptedException {
        while (q.size() == capacity) wait();
        q.add(item);
        notifyAll();
    }
    synchronized T take() throws InterruptedException {
        while (q.isEmpty()) wait();
        T item = q.poll();
        notifyAll();
        return item;
    }
}
// Same shape as Q82 — mention java.util.concurrent.ArrayBlockingQueue as the production equivalent
```

### 90. Atomic "update only if higher" score submission
```java
class AtomicLeaderboard {
    private final ConcurrentHashMap<Integer, Integer> scores = new ConcurrentHashMap<>();
    void submit(int userId, int newScore) {
        scores.merge(userId, newScore, Math::max); // atomic compute
    }
}
// Example: two threads submit scores 80 and 95 for the same user concurrently -> final value is always 95
```

---

## 10. Real-World Data Shape Problems

### 91. Orders spanning ≥2 categories (order-item-category join)
```java
static List<Integer> q91(List<int[]> orderItem, Map<Integer, Integer> itemCategory) {
    Map<Integer, Set<Integer>> orderCats = new HashMap<>();
    for (int[] oi : orderItem) {
        Integer cat = itemCategory.get(oi[1]);
        if (cat != null) orderCats.computeIfAbsent(oi[0], k -> new HashSet<>()).add(cat);
    }
    List<Integer> res = new ArrayList<>();
    for (var e : orderCats.entrySet()) if (e.getValue().size() >= 2) res.add(e.getKey());
    return res;
}
// Example: order1 has items in categories {electronics, books} -> qualifies
```

### 92. Users logged into both services from same device within 1 hour
```java
static Set<Integer> q92(List<long[]> serviceA, List<long[]> serviceB) { // {userId, deviceId, timestamp}
    Map<String, Long> aTimes = new HashMap<>();
    for (long[] r : serviceA) aTimes.put(r[0] + "-" + r[1], r[2]);
    Set<Integer> res = new HashSet<>();
    for (long[] r : serviceB) {
        String key = r[0] + "-" + r[1];
        Long tA = aTimes.get(key);
        if (tA != null && Math.abs(tA - r[2]) <= 3600) res.add((int) r[0]);
    }
    return res;
}
// Example: user1/device5 logs into A at t=1000, B at t=3000 -> diff 2000s <= 3600 -> qualifies
```

### 93. Merchants in both "flagged" and "high-volume" lists
```java
static Set<String> q93(List<int[]> ignore, List<String> flagged, List<String> highVolume) {
    Set<String> f = new HashSet<>(flagged), h = new HashSet<>(highVolume);
    f.retainAll(h);
    return f;
}
// Example: flagged=["M1","M2"], highVolume=["M2","M3"] -> {"M2"}
```

### 94. Engaged users on both platforms (>2 categories, >X total time)
```java
static List<Integer> q94(List<Object[]> platformA, List<Object[]> platformB, int minCats, int minTime) {
    var a = aggregate(platformA);
    var b = aggregate(platformB);
    List<Integer> res = new ArrayList<>();
    for (int uid : a.keySet()) {
        if (!b.containsKey(uid)) continue;
        var A = a.get(uid); var B = b.get(uid);
        Set<Object> combinedCats = new HashSet<>(A.categories);
        combinedCats.addAll(B.categories);
        if (combinedCats.size() >= minCats && (A.totalTime + B.totalTime) >= minTime) res.add(uid);
    }
    return res;
}
static class Agg { Set<Object> categories = new HashSet<>(); int totalTime = 0; }
static Map<Integer, Agg> aggregate(List<Object[]> data) { // {userId, articleId, readTimeSeconds, category}
    Map<Integer, Agg> m = new HashMap<>();
    for (Object[] r : data) {
        int uid = (int) r[0];
        Agg agg = m.computeIfAbsent(uid, k -> new Agg());
        agg.categories.add(r[3]);
        agg.totalTime += (int) r[2];
    }
    return m;
}
// Example: user engaged with 2+ categories and total read time above threshold across both platforms
```

### 95. Users with conflicting experiment variant assignment across regions
```java
static List<Integer> q95(List<int[]> regionA, List<int[]> regionB) { // {userId, experimentId, variantId}
    Map<String, Integer> aAssign = new HashMap<>();
    for (int[] r : regionA) aAssign.put(r[0] + "-" + r[1], r[2]);
    List<Integer> conflicts = new ArrayList<>();
    for (int[] r : regionB) {
        String key = r[0] + "-" + r[1];
        Integer variantA = aAssign.get(key);
        if (variantA != null && variantA != r[2]) conflicts.add(r[0]);
    }
    return conflicts;
}
// Example: user7/exp3 assigned variant A in region1, variant B in region2 -> conflict
```

### 96. Cross-platform multi-category users (web + mobile)
```java
static List<Integer> q96(List<int[]> web, List<int[]> mobile) {
    return q1(web, mobile); // identical shape to problem #1 — recognizing pattern reuse is itself a strong interview signal
}
// Example: same underlying logic as the original userId/category problem — say this connection out loud
```

### 97. Drivers active in ≥2 cities with rating above threshold
```java
static List<Integer> q97(List<Object[]> trips, Map<Integer, Double> ratings, double minRating) {
    Map<Integer, Set<String>> citiesByDriver = new HashMap<>();
    for (Object[] t : trips)
        citiesByDriver.computeIfAbsent((int) t[0], k -> new HashSet<>()).add((String) t[2]);
    List<Integer> res = new ArrayList<>();
    for (var e : citiesByDriver.entrySet()) {
        int driver = e.getKey();
        if (e.getValue().size() >= 2 && ratings.getOrDefault(driver, 0.0) > minRating) res.add(driver);
    }
    return res;
}
// Example: driver9 active in {SF, Oakland}, rating 4.8 > 4.5 -> qualifies
```

### 98. Subscription type migration mismatch detection
```java
static Map<Integer, String[]> q98(Map<Integer, String> legacy, Map<Integer, String> updated) {
    Map<Integer, String[]> mismatches = new HashMap<>();
    for (var e : legacy.entrySet()) {
        String newType = updated.get(e.getKey());
        if (newType != null && !newType.equals(e.getValue()))
            mismatches.put(e.getKey(), new String[]{e.getValue(), newType});
    }
    return mismatches;
}
// Example: user4 was "premium" in legacy, "basic" in new -> flagged as mismatch [premium, basic]
```

### 99. Users whose category preferences shifted between periods
```java
static List<Integer> q99(List<int[]> period1, List<int[]> period2) {
    Map<Integer, Set<Integer>> p1 = new HashMap<>(), p2 = new HashMap<>();
    for (int[] r : period1) p1.computeIfAbsent(r[0], k -> new HashSet<>()).add(r[1]);
    for (int[] r : period2) p2.computeIfAbsent(r[0], k -> new HashSet<>()).add(r[1]);
    List<Integer> shifted = new ArrayList<>();
    for (var e : p2.entrySet()) {
        Set<Integer> newCats = new HashSet<>(e.getValue());
        newCats.removeAll(p1.getOrDefault(e.getKey(), Set.of()));
        if (newCats.size() >= 2) shifted.add(e.getKey());
    }
    return shifted;
}
// Example: user2 watched {action} in period1, {comedy, drama} in period2 -> 2 new categories -> shifted
```

### 100. Users on both "fraud-flagged" and "high-value-customer" lists
```java
static Set<Integer> q100(List<int[]> fraudFlagged, List<int[]> highValue) {
    Set<Integer> fraud = extractIds(fraudFlagged), hv = extractIds(highValue);
    fraud.retainAll(hv);
    return fraud;
    // This is the highest-stakes version of problem #1's pattern — in a real system, this result
    // routes to manual review rather than auto-action, given the conflicting signals.
}
// Example: user99 on both lists -> flagged for manual review, not auto-blocked
```

---

## Interview delivery notes (apply to all 100)

1. **State the two-condition decomposition out loud before coding** — nearly every problem here is "membership check" + "aggregate condition," and naming that split explicitly (like we did with presence vs. category-count) is the single highest-leverage habit across this entire list.
2. **Name your data structure choice and why** — "I'm using a HashSet here because I need O(1) membership checks and don't care about order" reads far better than silently typing `new HashSet<>()`.
3. **State time/space complexity unprompted** at the end of each solution.
4. **Section 10 problems are functionally identical to Section 1 problems** wearing different business-domain clothing — recognizing and saying that out loud ("this is the same shape as the userId/category problem, just with drivers and cities") is a strong signal of pattern-matching ability, which is exactly what Staff-level screens are probing for.
