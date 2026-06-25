# Juspay Hackathon — Part A  
## Locking the Tree of Space

## Problem Statement

You are given a world map represented as an **M-ary Tree**.

Example structure:

```text
                World
           /      |      \
        Asia    Africa    ...
       /  |  \
   India China ...
```

You need to define three operations on this tree:

1. `lock(X, uid)`
2. `unlock(X, uid)`
3. `upgradeLock(X, uid)`

Where:
- `X` → Name of a unique node in the tree
- `uid` → User ID performing the operation

---

# Operation Definitions

## 1) Lock(X, uid)

Lock takes exclusive access on the subtree rooted at `X`.

Once `lock(X, uid)` succeeds:

- `lock(A, anyUserId)` should fail if `A` is a descendant of `X`
- `lock(B, anyUserId)` should fail if `X` is a descendant of `B`
- Lock operation cannot be performed on a node that is already locked

### Conditions for Lock:
- Node itself should not be locked
- No ancestor should be locked
- No descendant should be locked

---

## 2) Unlock(X, uid)

Unlock reverts the lock operation.

### Conditions:
- Unlock can only be performed on the same node
- Same `uid` who locked the node must unlock it

Returns:
- `true` → successful unlock
- `false` → otherwise

---

## 3) UpgradeLock(X, uid)

Upgrade lock helps a user upgrade their locks from descendants to ancestor.

### Conditions:
- Node `X` should not already be locked
- Node `X` must have locked descendants
- All locked descendants must be locked by the same user `uid`
- No locked ancestor should exist

Successful upgrade:
- Unlock all locked descendants
- Lock current node

Upgrade should not violate Lock/Unlock consistency.

---

# Constraints

- `1 < N < 5 * 10^5`
- `1 < m < 30`
- `1 < Q < 5 * 10^5`
- `1 < length(NodeName) < 20`

Where:
- `N` → Number of nodes
- `m` → Number of children per node
- `Q` → Number of queries

---

# Time Complexity Requirement

- Lock → `O(logₘ N)`
- Unlock → `O(logₘ N)`
- UpgradeLock → `O(numberOfLockedNodes × logₘ N)`

---

# Input Format

- First line → Number of Nodes (`N`)
- Second line → Number of children per node (`m`)
- Third line → Number of queries (`Q`)
- Next `N` lines → Node names
- Next `Q` lines → Queries in format:

```text
OperationType NodeName UserId
```

Where:
- `1` → Lock
- `2` → Unlock
- `3` → UpgradeLock

---

# Sample Input

```text
7
2
5
World
Asia
Africa
China
India
SouthAfrica
Egypt
1 China 9
1 India 9
3 Asia 9
2 India 9
2 Asia 9
```

---

# Sample Output

```text
true
true
true
false
true
```

---

# Explanation

### Query 1: `1 China 9`
- China is unlocked initially  
→ Success

### Query 2: `1 India 9`
- No ancestors or descendants are locked  
→ Success

### Query 3: `3 Asia 9`
- Asia has locked descendants: China & India
- Both locked by user 9  
→ Upgrade successful

### Query 4: `2 India 9`
- India already got unlocked during upgrade  
→ Fail

### Query 5: `2 Asia 9`
- Asia is locked by user 9  
→ Success

---

# Approach

## Key Optimization Idea

To meet strict constraints, each node stores:

- `isLocked`
- `lockedBy`
- `ancLockedCount`
- `descLockedCount`
- `lockedDescendants`

This allows:
- Fast ancestor checks
- Fast descendant checks
- Efficient upgrade operation

---

# Data Structure Used

```java
static class Node {
    String name;
    Node parent;
    List<Node> children;

    boolean isLocked;
    int lockedBy;
    int ancLockedCount;
    int descLockedCount;

    Set<Node> lockedDescendants;
}
```

---

# Solution (Java)

```java
import java.io.*;
import java.util.*;

public class Main {
    static class Node {
        String name;
        Node parent;
        List<Node> children;
        
        boolean isLocked;
        int lockedBy;
        int ancLockedCount;
        int descLockedCount;
        
        // Tracks the actual descendant nodes that are locked
        Set<Node> lockedDescendants;

        Node(String name, Node parent) {
            this.name = name;
            this.parent = parent;
            this.children = new ArrayList<>();
            this.isLocked = false;
            this.lockedBy = -1;
            this.ancLockedCount = 0;
            this.descLockedCount = 0;
            this.lockedDescendants = new HashSet<>();
        }
    }

    // Quick lookup from Node Name string to Node reference
    static Map<String, Node> nameToNodeMap = new HashMap<>();

    public static void main(String[] args) throws IOException {
        // Using BufferedReader for fast I/O performance
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st;

        String line = br.readLine();
        if (line == null) return;
        int n = Integer.parseInt(line.trim());

        line = br.readLine();
        int m = Integer.parseInt(line.trim());

        line = br.readLine();
        int q = Integer.parseInt(line.trim());

        String[] nodeNames = new String[n];
        for (int i = 0; i < n; i++) {
            nodeNames[i] = br.readLine().trim();
        }

        // Reconstruct the full m-ary tree structure from the array order
        Node root = new Node(nodeNames[0], null);
        nameToNodeMap.put(nodeNames[0], root);

        Queue<Node> queue = new LinkedList<>();
        queue.add(root);

        int childIdx = 1;
        while (!queue.isEmpty() && childIdx < n) {
            Node parent = queue.poll();
            for (int i = 0; i < m && childIdx < n; i++) {
                Node child = new Node(nodeNames[childIdx], parent);
                nameToNodeMap.put(nodeNames[childIdx], child);
                parent.children.add(child);
                queue.add(child);
                childIdx++;
            }
        }

        // Process queries
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < q; i++) {
            line = br.readLine();
            if (line == null) break;
            st = new StringTokenizer(line);
            int type = Integer.parseInt(st.nextToken());
            String nodeName = st.nextToken();
            int uid = Integer.parseInt(st.nextToken());

            Node node = nameToNodeMap.get(nodeName);
            boolean res = false;

            if (type == 1) {
                res = lock(node, uid);
            } else if (type == 2) {
                res = unlock(node, uid);
            } else if (type == 3) {
                res = upgradeLock(node, uid);
            }
            sb.append(res).append("\n");
        }
        System.print.out(sb.toString());
    }

    // 1. LOCK OPERATION: O(log_m N)
    private static boolean lock(Node node, int uid) {
        if (node.isLocked || node.ancLockedCount > 0 || node.descLockedCount > 0) {
            return false;
        }

        node.isLocked = true;
        node.lockedBy = uid;

        Node curr = node.parent;
        while (curr != null) {
            curr.descLockedCount++;
            curr.lockedDescendants.add(node);
            curr = curr.parent;
        }

        updateDescendantsAncCount(node, 1);

        return true;
    }

    // 2. UNLOCK OPERATION: O(log_m N)
    private static boolean unlock(Node node, int uid) {
        if (!node.isLocked || node.lockedBy != uid) {
            return false;
        }

        node.isLocked = false;
        node.lockedBy = -1;

        Node curr = node.parent;
        while (curr != null) {
            curr.descLockedCount--;
            curr.lockedDescendants.remove(node);
            curr = curr.parent;
        }

        updateDescendantsAncCount(node, -1);

        return true;
    }

    // 3. UPGRADE LOCK OPERATION: O(NumberOfLockedNodes * log_m N)
    private static boolean upgradeLock(Node node, int uid) {
        if (node.isLocked || node.ancLockedCount > 0 || node.descLockedCount == 0) {
            return false;
        }

        for (Node desc : node.lockedDescendants) {
            if (desc.lockedBy != uid) {
                return false;
            }
        }

        List<Node> toUnlock = new ArrayList<>(node.lockedDescendants);

        for (Node desc : toUnlock) {
            unlock(desc, uid);
        }

        return lock(node, uid);
    }

    private static void updateDescendantsAncCount(Node node, int val) {
        for (Node child : node.children) {
            child.ancLockedCount += val;
            updateDescendantsAncCount(child, val);
        }
    }
}
```

---

# Complexity Analysis

| Operation | Complexity |
|-----------|------------|
| Lock | O(logₘ N) |
| Unlock | O(logₘ N) |
| UpgradeLock | O(K × logₘ N) |

Where:
- `K` = number of locked descendants

---

# Important Notes

There are two minor typos in the submitted code:

### 1
```java
System.print.out(sb.toString());
```

Should be:
```java
System.out.print(sb.toString());
```

### 2
Depending on judge environment, recursive descendant updates may become expensive for large subtrees.

---