---
layout:     post
title:      Data Structure
subtitle:   This post is used to record some data structures and Algorithms I met.
date:       2018-12-19
author:     CodingMrWang
header-img: img/ds.png
catalog: true
tags:
    - Data Structure
    - Java
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.

### Segment Tree

Build

```java
public SegmentTreeNode build(int[] A) {
    if (A == null || A.length == 0) {
        return null;
    }
    return buildHelper(A, 0, A.length - 1);
}
private SegmentTreeNode buildHelper(int[] A, int start, int end) {
	//leaf
    if (start == end) {
        return new SegmentTreeNode(start, start, A[start]);
    }
    // recursively build the segment Tree
    SegmentTreeNode node = new SegmentTreeNode(start, end, Integer.MIN_VALUE);
    node.left = buildHelper(A, start, start + (end - start) / 2);
    node.right = buildHelper(A, start + (end - start) / 2 + 1, end);
    node.max = Math.max(node.left.max, node.right.max);
    return node;
}
```

Query

```java
public int query(SegmentTreeNode root, int start, int end) {
    // write your code here
    if (start > end || start > root.end || end < root.start) {
        return Integer.MIN_VALUE;
    }
    if (start == root.start && end == root.end) {
        return root.max;
    }
    int mid = root.start + (root.end - root.start) / 2;
    int leftmax = query(root.left, start, Math.min(end, mid));
    int rightmax = query(root.right, Math.max(start, mid + 1), end);
    return Math.max(leftmax, rightmax);
}
```

Modify

```java
public void modify(SegmentTreeNode root, int index, int value) {
    if (root.start == index && root.end == index) {
        root.max = value;
        return;
    }
    int mid = root.start + (root.end - root.start) / 2;
    if (index <= mid) {
        modify(root.left, index, value);
    } else {
        modify(root.right, index, value);
    }
    root.max = Math.max(root.left.max, root.right.max);
}
```

Time complexity

```
RangeSum: O(logN)
Range MAX/MIN: O(logN)
MAX/MIN: O(1) update: O(logN)
Min value larger than a number: O(logN)
Max value smaller than a number: O(logN)
```

### Maximum SubArray/Matrix

1. Compute prefix sum
2. Use for loop to iterate the array, keep a min of sum, use max to record sum - min to get max.

Maximum SubMatrix

```java
public int maxSubmatrix(int[][] matrix) {
    if (matrix == null || matrix.length == 0 || matrix[0] == null || matrix[0].length == 0) {
        return 0;
    }
    int max = Integer.MIN_VALUE;
	 //compute prefix sum of the matrix
    int[][] prefixSum = getPrefix(matrix);
	 // i is the upper bound, j is the lower bound.
	 // compute the sum of the matrix in this j - i, then use maximum subarray to get maximum submatrix.
    for (int i = 0; i < matrix.length; i++) {
        for (int j = i; j < matrix[0].length; j++) {
            int[] arr = compress(prefixSum, i, j);
            int min = 0;
            int sum = 0;
            for (int k = 0; k < arr.length; k++) {
                sum += arr[k];
                max = Math.max(max, sum - min);
                min = Math.min(sum, min);
            }
        }
    }
    return max;
}
private int[] compress(int[][] prefix, int i, int j) {
    int[] arr = new int[prefix[0].length];
    for (int k = 0; k < prefix[0].length; k++) {
        arr[k] = prefix[j + 1][k] - prefix[i][k];
    }
    return arr;
}
private int[][] getPrefix(int[][] matrix) {
    int[][] prefix = new int[matrix.length + 1][matrix[0].length];
    for (int i = 0; i < matrix.length; i++) {
        for (int j = 0; j < matrix[0].length; j++) {
            prefix[i + 1][j] = prefix[i][j] + matrix[i][j];
        }
    }
    return prefix;
}
```


### Longest Palindromic Substring

Take each element as the middle point of the palindromic
Time complexity: O(N^2)

```java
public String longestPalindrome(String s) {
    // write your code here
    if (s == null || s.length() == 0) {
        return s;
    }
    int longest = 1;
    int start = 0;
    int len = 0;
    for (int i = 0; i < s.length() - 1; i++) {
        len = helper(s, i, i);
        if (longest < len) {
            longest = len;
            start = i - len / 2;
        }
        len = helper(s, i, i + 1);
        if (longest < len) {
            longest = len;
            start = i - len / 2 + 1;
        }
    }
    return s.substring(start, start + longest);
}
    
private int helper(String s, int left, int right) {
    int len = 0;
    while (left >= 0 && right < s.length()) {
        if (s.charAt(left) == s.charAt(right)) {
            len++;
            if (left != right) {
                len++;
            }
            left--;
            right++;
        } else {
            return len;
        }
    }
    return len;
}
```

### Permutation without recursion

```java
public List<List<Integer>> permute(int[] nums) {
    // write your code here
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> stack = new ArrayList<>();
    if (nums == null) {
        return result;
    }
    if (nums.length == 0) {
        result.add(stack);
        return result;
    }
    stack.add(-1);
    int n = nums.length;
    while (stack.size() != 0) {
        Integer last = stack.get(stack.size() - 1);
        //back tracking
        stack.remove(stack.size() - 1);
        int next = -1;
        //[0, 1, 2], remove 2, remove 1, add 2. [0, 2]
        for (int i = last + 1; i < n; i++) {
            if (!stack.contains(i)) {
                next = i;
                break;
            }
        }
        if (next == -1) {
            continue;
        }
        stack.add(next);
        //[0, 2, 1]
        for (int i = 0; i < n; i++) {
            if (!stack.contains(i)) {
                stack.add(i);
            }
        }
        List<Integer> step = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            step.add(nums[stack.get(i)]);
        }
        result.add(step);
    }
    return result;
}
```

### Median of k sorted arrays

```java
public double findMedian(int[][] nums) {
    // write your code here
    int totalL = 0;
    for (int i = 0; i < nums.length; i++) {
        totalL += nums[i].length;
    }
    if (totalL == 0) {
        return 0;
    }
    if (totalL % 2 != 0) {
        return findKth(nums, totalL / 2 + 1);
    } else {
        return findKth(nums, totalL / 2) * 1.0 / 2 + findKth(nums, totalL / 2 + 1) * 1.0 / 2;
    }
}
private int findKth(int[][] nums, int k) {
    int start = 0;
    int end = Integer.MAX_VALUE;
    while (start + 1 < end) {
        int mid = start + (end - start) / 2;
        // this make sure the mid is in the nums array
        if (let(nums, mid) >= k) {
            end = mid;
        } else {
            start = mid;
        }
    }
    if (let(nums, start) == k) {
        return start;
    }
    return end;
}
    
private int let(int[][] nums, int val) {
    int sum = 0;
    for (int i = 0; i < nums.length; i++) {
        sum += let(nums[i], val);
    }
    return sum;
}
// get how many number in num smaller or equal to val
private int let(int[] num, int val) {
    if (num.length == 0) {
        return 0;
    }
    int start = 0;
    int end = num.length - 1;
    while (start + 1 < end) {
        int mid = start + (end - start) / 2;
        if (num[mid] <= val) {
            start = mid;
        } else {
            end = mid;
        }
    }
    if (num[end] <= val) {
        return end + 1;
    }
    if (num[start] <= val) {
        return start + 1;
    }
    return 0;
}
```