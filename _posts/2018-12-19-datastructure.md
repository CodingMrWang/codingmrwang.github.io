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