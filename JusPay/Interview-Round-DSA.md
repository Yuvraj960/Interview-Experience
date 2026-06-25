# Juspay Interview Round — DSA Questions

This round focused on problem solving and DSA fundamentals.

Questions asked:

1. K-th Smallest Sum in a Sorted Matrix
2. Maximum Subarray Sum

---

# Question 1: K-th Smallest Sum in Sorted Matrix

## Problem Statement

You are given a matrix of `m` rows and `n` columns where each row is sorted in non-decreasing order.

You need to choose exactly **one element from each row** and compute the sum.

Return the **k-th smallest sum** among all possible sums.

---

## Example

Input:

```text
matrix = [
    [1, 3, 11],
    [2, 4, 6]
]
k = 5
```

Output:

```text
7
```

---

## Explanation

Possible sums:

```text
1+2 = 3
1+4 = 5
3+2 = 5
1+6 = 7
3+4 = 7
3+6 = 9
11+2 = 13
11+4 = 15
11+6 = 17
```

Sorted sums:

```text
3, 5, 5, 7, 7, 9, 13, 15, 17
```

5th smallest = **7**

---

# Approach

Since brute force generates all combinations:

```text
O(n^m)
```

which is too expensive.

Optimized approach:
- Start with first row
- Merge row-by-row
- Keep only smallest `k` sums using Min Heap

This ensures we never keep unnecessary large sums.

---

# Complexity

```text
O(m × k × n × log k)
```

---

# Solution (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> mergeRows(vector<int>& a, vector<int>& b, int k) {
    priority_queue<int> pq;

    for (int x : a) {
        for (int y : b) {
            int sum = x + y;

            if ((int)pq.size() < k) {
                pq.push(sum);
            } else if (sum < pq.top()) {
                pq.pop();
                pq.push(sum);
            } else {
                break;
            }
        }
    }

    vector<int> result;
    while (!pq.empty()) {
        result.push_back(pq.top());
        pq.pop();
    }

    return result;
}

int kthSmallest(vector<vector<int>>& matrix, int k) {
    vector<int> sums = matrix[0];

    for (int i = 1; i < matrix.size(); i++) {
        sums = mergeRows(sums, matrix[i], k);
    }

    sort(sums.begin(), sums.end());

    return sums[k - 1];
}
```

---

# Question 2: Maximum Subarray Sum

## Problem Statement

Given an integer array `nums`, find the contiguous subarray with the largest sum.

Return that sum.

---

## Example 1

Input:

```text
nums = [-2,1,-3,4,-1,2,1,-5,4]
```

Output:

```text
6
```

Explanation:

Subarray:

```text
[4,-1,2,1]
```

Sum:

```text
6
```

---

## Example 2

Input:

```text
nums = [5,4,-1,7,8]
```

Output:

```text
23
```

Explanation:

Entire array gives maximum sum.

---

# Approach

Classic Kadane’s Algorithm.

Idea:
- Maintain current sum
- If current sum becomes negative, discard it
- Track maximum sum seen so far

---

# Complexity

```text
Time: O(N)
Space: O(1)
```

---

# Solution (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxSubArray(vector<int>& nums) {
    int currSum = nums[0];
    int maxSum = nums[0];

    for (int i = 1; i < nums.size(); i++) {
        currSum = max(nums[i], currSum + nums[i]);
        maxSum = max(maxSum, currSum);
    }

    return maxSum;
}
```

---

# Interview Notes

### Concepts Tested
- Heap / Priority Queue
- Matrix-based combinational optimization
- Dynamic Programming
- Kadane’s Algorithm
- Problem-solving under time pressure

### Difficulty
- Q1 → Medium-Hard
- Q2 → Easy-Medium

### Interview Focus
Juspay heavily emphasizes:
- Optimization
- Time complexity analysis
- Clean problem-solving approach
- DSA fundamentals