# Juspay OA — Question 2  
# Parking Lot Arrangement

## Problem Statement

You are managing a parking lot with `n` parking spaces.

Cars are parked at scattered positions.

Goal:
- Rearrange cars so all cars occupy consecutive parking spaces.

Rule:
- In one move, one car can be moved to any arbitrary position.

Find minimum moves required.

---

# Input Format

- First line contains integer `n`
- Next `n` lines contain positions of parked cars

---

# Constraints

- `1 ≤ n ≤ 10^5`
- `1 ≤ a[i] ≤ 10^9`

---

# Output

Return minimum moves required.

---

# Approach

Key Insight:
We want maximum number of cars already fitting inside some window of size `n`.

Cars outside that window must be moved.

Use:
- Sorting
- Sliding Window

---

# Complexity

- Sorting → `O(N log N)`
- Sliding Window → `O(N)`

Total:
```text
O(N log N)
```

---

# Solution (Java)

```java
Import java.io.*;
import java.util.*;

public class Solution {

    public static int solve(int n, List<Integer> a) {
        List<Integer> sorted = new ArrayList<>(a);
        Collections.sort(sorted);

        int maxInWindow = 1;
        int left = 0;

        for (int right = 0; right < n; right++) {
            while (sorted.get(right) - sorted.get(left) >= n) {
                left++;
            }
            maxInWindow = Math.max(maxInWindow, right - left + 1);
        }

        return n - maxInWindow;
    }

    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);

        int n = Integer.parseInt(scan.nextLine().trim());

        List<Integer> a = new ArrayList<>(n);
        for (int j = 0; j < n; j++) {
            a.add(Integer.parseInt(scan.nextLine().trim()));
        }

        int result = solve(n, a);
        System.out.println(result);
    }
}
```