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