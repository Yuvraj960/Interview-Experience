# Juspay OA — Question 1  
# Smart City Traffic Toll System

## Problem Statement

In a smart city, there are infinite traffic control hubs numbered with positive integers starting from 1.

Each hub is connected in a tree-like structure:

- There is a direct road between hub `i` and `2i`
- There is another direct road between hub `i` and `2i + 1`

Thus, the city forms an infinite binary tree.

Initially, all roads are toll-free.

Traffic authority introduces two types of events:

### 1) Toll Fee Update
An update described by integers:

```text
1 x y t
```

Meaning:
- Add toll fee `t` to all roads along the shortest path from hub `x` to hub `y`.

---

### 2) Travel Cost Calculation
A travel event described by:

```text
2 x y 0
```

Meaning:
- A commuter travels from hub `x` to hub `y`
- Calculate total toll fees on the shortest path

---

# Input Format

- First line contains integer `q` → number of events
- Next `q` lines each contain 4 integers

---

# Constraints

- `1 ≤ q ≤ 10^5`
- `1 ≤ a[i][j] ≤ 10^5`

---

# Output

For each travel event, calculate total travel cost and return the sum.

---

# Approach

Key idea:
- Tree is implicit → no need to build entire tree
- Parent of node `i = i/2`
- Use:
  - LCA (Lowest Common Ancestor)
  - HashMap to store edge tolls

### Steps:
1. Find LCA of `x` and `y`
2. Update toll on path using parent traversal
3. Query cost similarly

---

# Complexity Analysis

- Update: `O(log N)`
- Query: `O(log N)`

---

# Solution (Java)

```java
Import java.io.*;
import java.util.*;
import java.util.stream.Collectors;

public class Solution {

    // Edge cost stored at child node: edge (child -> child/2) has cost edgeCost[child]
    static Map<Long, Long> edgeCost = new HashMap<>();

    // Find LCA of two nodes in binary tree (parent of i = i/2)
    static long lca(long a, long b) {
        while (a != b) {
            if (a > b) a >>= 1;
            else       b >>= 1;
        }
        return a;
    }

    // Add toll t to all edges on path from node 'node' up to (but not including) 'ancestor'
    static void addCostToPath(long node, long ancestor, long t) {
        while (node != ancestor) {
            edgeCost.merge(node, t, Long::sum);
            node >>= 1;
        }
    }

    // Sum toll on path from node up to (but not including) ancestor
    static long queryCostToAncestor(long node, long ancestor) {
        long cost = 0;
        while (node != ancestor) {
            cost += edgeCost.getOrDefault(node, 0L);
            node >>= 1;
        }
        return cost;
    }

    public static int solve(int q, List<List<Integer>> a) {
        edgeCost.clear();
        long totalSum = 0;

        for (List<Integer> event : a) {
            int type = event.get(0);
            long x    = event.get(1);
            long y    = event.get(2);
            long t    = event.get(3);

            long lcaNode = lca(x, y);

            if (type == 1) {
                addCostToPath(x, lcaNode, t);
                addCostToPath(y, lcaNode, t);

            } else {
                totalSum += queryCostToAncestor(x, lcaNode);
                totalSum += queryCostToAncestor(y, lcaNode);
            }
        }

        return (int) totalSum;
    }

    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);

        int q = Integer.parseInt(scan.nextLine().trim());

        List<List<Integer>> a = new ArrayList<>(q);
        for (int i = 0; i < q; i++) {
            a.add(
                Arrays.asList(scan.nextLine().trim().split(" "))
                      .stream()
                      .map(Integer::parseInt)
                      .collect(Collectors.toList())
            );
        }

        int result = solve(q, a);
        System.out.println(result);
    }
}
```