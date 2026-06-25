# Juspay OA — Question 3  
# Corporate Restructuring

## Problem Statement

Techify Inc. has a hierarchical employee structure represented as a tree.

- CEO is root
- Employees have IDs from `1` to `N`

Employee states:
- `0` → Compliant
- `1` → Non-compliant

A non-compliant employee can be removed if:
1. Employee is non-compliant
2. Employee has no compliant subordinates
3. If multiple employees satisfy condition, remove smallest ID

After removal:
- Direct reports get reassigned to removed employee’s supervisor

Repeat until no removable employees remain.

---

# Input Format

- First line contains integer `N`
- Next `N` lines contain:
```text
Pi Ci
```

Where:
- `Pi` = Parent ID (`-1` for CEO)
- `Ci` = Compliance status

---

# Constraints

- `1 ≤ N ≤ 10^5`

---

# Output

Return removal order of employees.

If none removable:
```text
-1
```

---

# Approach

Use:
- Parent array
- Children adjacency list
- Compliance count in subtrees
- Min Heap for smallest valid removable employee

Steps:
1. Build tree
2. Compute compliant counts in subtree
3. Push removable nodes into min-heap
4. Remove smallest valid employee repeatedly

---

# Complexity

Approx:
```text
O(N log N)
```

---

# Solution (Java)

```java
Import java.io.*;
import java.util.*;
import java.util.stream.Collectors;

public class Solution {

    public static List<Integer> isCompliant(int N, List<List<Integer>> Arr) {
        // parent[i], compliant[i] for employee i (1-indexed)
        int[] parent    = new int[N + 1];
        int[] compliant = new int[N + 1]; // 0=compliant, 1=non-compliant

        // children set for each node
        // Use TreeSet so children are ordered (helps with re-evaluation)
        List<Set<Integer>> children = new ArrayList<>();
        for (int i = 0; i <= N; i++) children.add(new HashSet<>());

        int root = -1;

        for (int i = 0; i < N; i++) {
            int id = i + 1; // employee IDs are 1-indexed
            int p  = Arr.get(i).get(0);
            int c  = Arr.get(i).get(1);
            parent[id]    = p;
            compliant[id] = c;
            if (p == -1) {
                root = id;
            } else {
                children.get(p).add(id);
            }
        }

        // compliantDescendants[i] = number of compliant employees in subtree of i
        // (including i itself if compliant)
        int[] compliantCount = new int[N + 1];

        // Compute compliantCount via post-order DFS
        // compliantCount[i] = (compliant[i]==0 ? 1 : 0) + sum of children
        Deque<int[]> stack = new ArrayDeque<>(); // [node, phase]
        int[] visitOrder  = new int[N + 1];
        // Iterative post-order
        Deque<Integer> dfsStack = new ArrayDeque<>();
        List<Integer> postOrder = new ArrayList<>();
        dfsStack.push(root);
        while (!dfsStack.isEmpty()) {
            int node = dfsStack.pop();
            postOrder.add(node);
            for (int child : children.get(node)) {
                dfsStack.push(child);
            }
        }
        // postOrder is reverse post-order (pre-order actually), reverse it
        Collections.reverse(postOrder);

        for (int node : postOrder) {
            compliantCount[node] = (compliant[node] == 0) ? 1 : 0;
            for (int child : children.get(node)) {
                compliantCount[node] += compliantCount[child];
            }
        }

        // A node is "removable" if:
        // 1. non-compliant (compliant[node] == 1)
        // 2. compliantCount[node] == 0  (no compliant in its subtree including itself)
        // Since node itself is non-compliant, compliantCount[node] == 0
        // means no compliant descendants either

        // Min-heap by ID
        PriorityQueue<Integer> pq = new PriorityQueue<>();
        for (int id = 1; id <= N; id++) {
            if (compliant[id] == 1 && compliantCount[id] == 0) {
                pq.offer(id);
            }
        }

        List<Integer> result = new ArrayList<>();

        while (!pq.isEmpty()) {
            int emp = pq.poll();
            result.add(emp);

            int par = parent[emp];

            // Reassign emp's children to emp's parent
            for (int child : children.get(emp)) {
                parent[child] = par;
                if (par != -1) {
                    children.get(par).add(child);
                }
            }
            children.get(emp).clear();

            // Remove emp from parent's children
            if (par != -1) {
                children.get(par).remove(emp);

                // Update compliantCount up the ancestor chain
                // emp's subtree compliantCount was 0, so no change needed
                // But we need to re-check if par is now removable
                // Walk up and re-check removability
                int cur = par;
                while (cur != -1) {
                    // Recompute compliantCount[cur] from scratch
                    // Only cur's own value + children changed
                    int oldCount = compliantCount[cur];
                    int newCount = (compliant[cur] == 0) ? 1 : 0;
                    for (int child : children.get(cur)) {
                        newCount += compliantCount[child];
                    }
                    compliantCount[cur] = newCount;

                    // Check removability of cur
                    if (compliant[cur] == 1 && compliantCount[cur] == 0
                            && !pq.contains(cur)) {
                        pq.offer(cur);
                    }

                    if (newCount == oldCount) break; // no change propagates further
                    cur = parent[cur];
                }
            }
        }

        if (result.isEmpty()) {
            result.add(-1);
        }

        return result;
    }

    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        int N = Integer.parseInt(scan.nextLine().trim());

        List<List<Integer>> Arr = new ArrayList<>(N);
        for (int i = 0; i < N; i++) {
            Arr.add(
                Arrays.asList(scan.nextLine().trim().split(" "))
                      .stream()
                      .map(Integer::parseInt)
                      .collect(Collectors.toList())
            );
        }

        List<Integer> result = isCompliant(N, Arr);
        for (int j = 0; j < result.size(); j++) {
            System.out.println(result.get(j));
        }
    }
}
```