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

### Longest Substring Without Repeating Characters
O(N)

```java
public int lengthOfLongestSubstring(String s) {
    // write your code here
    int[] hash = new int[256];
    int max = 0;
    int j = 0;
    for (int i = 0; i < s.length(); i++) {
        while (j < s.length() && hash[s.charAt(j)] == 0) {
            hash[s.charAt(j)] += 1;
            max = Math.max(max, j - i + 1);
            j++;
        }
        hash[s.charAt(i)] -= 1;
    }
    return max;
}
```

### Union Find

```java
int[] f;
public ConnectingGraph2(int n) {
    // do intialization if necessary
    f = new int[n + 1];
    for (int i = 0; i <= n; i++) {
        f[i] = -1;
    }
}

/*
 * @param a: An integer
 * @param b: An integer
 * @return: nothing
 */
public void union(int a, int b) {
    // write your code here
    int a_root = find(a);
    int b_root = find(b);
    if (a_root == b_root) {
        return;
    }
    int a_count = -f[a_root];
    int b_count = -f[b_root];
    f[a_root] = -(a_count + b_count);
    f[b_root] = a_root;
}
private int find(int a) {
    if (f[a] < 0) {
        return a;
    }
    return f[a] = find(f[a]);
}
/*
 * @param a: An integer
 * @return: An integer
 */
public int query(int a) {
    // write your code here
    int a_root = find(a);
    return -f[a_root];
}
```

### Monotonous stack
When we want to get the first numbers smaller than on the left and right side of each element, we can use stack.

Description

Given a 2D boolean matrix filled with False and True, find the largest rectangle containing all True and return its area.

Example

Given a matrix:

```
[
  [1, 1, 0, 0, 1],
  [0, 1, 0, 0, 1],
  [0, 0, 1, 1, 1],
  [0, 0, 1, 1, 1],
  [0, 0, 0, 0, 1]
]
```
return 6.

Transfer into 1D array.
![](https://ws4.sinaimg.cn/large/006tNc79ly1fyz1rwn9efj30du0buq35.jpg)

```java
public int maximalRectangle(boolean[][] matrix) {
    // write your code here
    if (matrix == null || matrix.length == 0) {
        return 0;
    }
    int[][] height = new int[matrix.length][matrix[0].length];
    int max = Integer.MIN_VALUE;
    for (int i = 0; i < matrix.length; i++) {
        Stack<Integer> stack = new Stack<>();
        for (int j = 0; j <= matrix[0].length; j++) {
            if (j < matrix[0].length) {
                height[i][j] = matrix[i][j] ? 1 : 0;
                if (i > 0 && height[i][j] == 1) {
                    height[i][j] += height[i - 1][j];
                }
            }
            int curr = j < matrix[0].length ? height[i][j] : Integer.MIN_VALUE;
            while (!stack.isEmpty() && curr <= height[i][stack.peek()]) {
                int temp = stack.pop();
                int w = stack.isEmpty() ? -1 : stack.peek();
                max = Math.max(max, height[i][temp] * (j - w - 1));
            }
            stack.push(j);
        }
    }
    return max;
}
```

### Expression evaluation
if is num, enter num Stack directly. if is operator, see operator stack, pop all operator with higher or equal priority and do operations. For example, if current operator is +, then pop /, *, -, +. If current operator is "/", only pop * or /. If is (, just enter operator stack, if is ), pop until (.

```java
public int evaluateExpression(String[] expression) {
    // write your code here
    if (expression == null || expression.length == 0) {
        return 0;
    }
    Map<String, Integer> operators = new HashMap<>();
    operators.put("+", 0);
    operators.put("-", 0);
    operators.put("*", 1);
    operators.put("/", 1);
    operators.put("(", -1);
    operators.put(")", -1);
    Stack<Integer> numStack = new Stack<>();
    Stack<String> operStack = new Stack<>();
    for (int i = 0; i < expression.length; i++) {
        if (!operators.containsKey(expression[i])) {
            numStack.push(Integer.parseInt(expression[i]));
        } else {
            if (expression[i].equals("(")) {
                operStack.push("(");
            } else if (expression[i].equals(")")) {
                while (!operStack.peek().equals("(")) {
                    String oper = operStack.pop();
                    Integer a = numStack.pop();
                    Integer b = numStack.pop();
                    getResult(a, b, oper, numStack);
                }
                operStack.pop();
            } else {
                while (!operStack.isEmpty() && operators.get(expression[i]) <= operators.get(operStack.peek())) {
                    String oper = operStack.pop();
                    getResult(numStack.pop(), numStack.pop(), oper, numStack);
                }
                operStack.push(expression[i]);
            }
        }
    }
    while (!operStack.isEmpty()) {
        getResult(numStack.pop(), numStack.pop(), operStack.pop(), numStack);
    }
    if (numStack.isEmpty()) {
        return 0;
    }
    return numStack.pop();
}
private void getResult(int a, int b, String oper, Stack<Integer> numStack) {
    if (oper.equals("+")) {
        numStack.push(a + b);
    } else if (oper.equals("-")) {
        numStack.push(b - a);
    } else if (oper.equals("*")) {
        numStack.push(a * b);
    } else {
        numStack.push(b / a);
    }
}
```

# Sweep Line

Divide [start, end] into [start, true] [end, false], then sort and walk through each point. record a count at the same time.

### The Skyline Problem

Using Sweep line, divid [start, end, height] into [start, 0, height] and [end, 1, height]. Sort the list. Also using tree map to record largest height. current height is always the largest height in the treemap. If current height is different from previous height, it means a new outline is required to be recorded. So use current index and previous index and previous height to record the previous outline.

```java
public List<List<Integer>> buildingOutline(int[][] buildings) {
    // write your code here
    if (buildings == null || buildings.length == 0 || buildings[0].length == 0) {
        return null;
    }
    List<Point> list = new ArrayList<>();
    for (int i = 0; i < buildings.length; i++) {
        list.add(new Point(buildings[i][0], buildings[i][2], 0));
        list.add(new Point(buildings[i][1], buildings[i][2], 1));
    }
    Collections.sort(list, new Comparator<Point>() {
        public int compare(Point p1, Point p2) {
            if (p1.x == p2.x) {
                if (p1.up == p2.up) {
                    return p2.height - p1.height;
                }
                return p1.up - p2.up;
            }
            return p1.x - p2.x;
        }
    });
    TreeMap<Integer, Integer> tm = new TreeMap<>();
    int preheight = 0;
    int preindex = 0;
    List<List<Integer>> result = new ArrayList<>();
    for (Point p : list) {
        if (p.up == 0) {
            if(!tm.containsKey(p.height)) {
               tm.put(p.height, 0); 
            }
            tm.put(p.height, tm.get(p.height) + 1);
        } else {
            tm.put(p.height, tm.get(p.height) - 1);
            if (tm.get(p.height) == 0) {
                tm.remove(p.height);
            }
        }
        
        int currheight = tm.isEmpty() ? 0 : tm.lastKey();
        if (preheight != currheight) {
            if (preindex != p.x && preheight != 0) {
                List<Integer> step = new ArrayList<>();
                step.add(preindex);
                step.add(p.x);
                step.add(preheight);
                result.add(step);
            }
            preheight = currheight;
            preindex = p.x;
        }
    }
    return result;
}
private static class Point {
    int x, height, up;
    Point(int x, int height, int up) {
        this.x = x;
        this.height = height;
        this.up = up;
    }
}
```

# Deque

### Sliding Window Maximum
when enter the queue, if current is larger than last element, polllast because the last is not maximum for sure. For deque, if the first element is equal to nums[i - k + 1], poll it.

```java
public List<Integer> maxSlidingWindow(int[] nums, int k) {
    // write your code here
    if (nums == null || nums.length == 0) {
        return null;
    }
    List<Integer> result = new ArrayList<>();
    Deque<Integer> dq = new ArrayDeque<>();
    for (int i = 0; i < k - 1; i++) {
        enqueue(dq, nums[i]);
    }
    for (int i = k - 1; i < nums.length; i++) {
        enqueue(dq, nums[i]);
        result.add(dq.peekFirst());
        dequeue(dq, nums[i - k + 1]);
    }
    return result;
}
private void enqueue(Deque<Integer> dq, int v) {
    while (!dq.isEmpty() && v > dq.peekLast()) {
        dq.pollLast();
    }
    dq.offer(v);
}
private void dequeue(Deque<Integer> dq, int v) {
    if (dq.peekFirst() == v) {
        dq.pollFirst();
    }
}
```

### Submatrix Sum

use presum, `int[][] presum = new int[n + 1][m + 1]`

`sum[i to j] = presum[j + 1] - presum[i]`

```java
public int[][] submatrixSum(int[][] matrix) {
    int n = matrix.length;
    int m = matrix[0].length;
    int[][] presum = new int[n + 1][m + 1];
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            presum[i + 1][j + 1] = matrix[i][j] + presum[i][j + 1] + presum[i + 1][j] - presum[i][j];
        }
    }
    int[][] result = new int[2][2];
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j <= n; j++) {
            Map<Integer, Integer> map = new HashMap<>();
            for (int k = 0; k < m + 1; k++) {
                int diff = presum[j][k] - presum[i][k];
                if (map.containsKey(diff)) {
                    result[0][0] = i;
                    result[0][1] = map.get(diff);
                    result[1][0] = j - 1;
                    result[1][1] = k - 1;
                    return result;
                } else {
                    map.put(diff, k);
                }
            }
        }
    }
    return result;
}
```

### BackPack
Use Value as index.

dp[i - 1][j - current value] || dp[i - 1][j]

- left: take current value
- right: do not take current value

```java
public int backPack(int m, int[] A) {
    // write your code here
    if (A == null || A.length == 0) {
        return 0;
    }
    boolean[][] dp = new boolean[A.length + 1][m + 1];
    for (int i = 0; i < dp.length; i++) {
        dp[i][0] = true;
    }
    for (int i = 1; i < dp.length; i++) {
        for (int j = 0; j < dp[0].length; j++) {
            dp[i][j] = dp[i - 1][j];
            if (j - A[i - 1] >= 0) {
                dp[i][j] = dp[i][j] || dp[i - 1][j - A[i - 1]];
            }
        }
    }
    for (int i = dp[0].length - 1; i >= 0; i--) {
        if (dp[dp.length - 1][i] == true) {
            return i;
        }
    }
    return 0;
}
```

###  Maximum Gap

```
Description
Given an unsorted array, find the maximum difference between
the successive elements in its sorted form.
Return 0 if the array contains less than 2 elements.
```

Idea:

此题用bucket sort的原理： 最大的gap必定大于等于平均gap！所以，用平均gap作为bucket size，在同一个bucket里面的元素之间的gap必定小于average gap不予考虑，只考虑相邻有元素的bucket。 最大的gap一定是某一对bucket后一个bucket的最小减去前一个bucket最大。
因为第一个bucket和最后一个bucket一定存在min和max，所以pre可以设为0， 然后看存在min的idx，如何存在min，一定存在max，这样就可以遍历所有存在max和min的了。

```java
public int maximumGap(int[] nums) {
    // write your code here
    int max = Integer.MIN_VALUE;
    int min = Integer.MAX_VALUE;
    for (int i : nums) {
        max = Math.max(max, i);
        min = Math.min(min, i);
    }
    int size = (max - min) / nums.length + 1;
    int num = (max - min) / size + 1;
    int[] bucketMax = new int[num];
    int[] bucketMin = new int[num];
    for (int i = 0; i < num; i++) {
        bucketMin[i] = -1;
        bucketMax[i] = -1;
    }
    for (int i = 0; i < nums.length; i++) {
        int idx = (nums[i] - min) / size;
        bucketMin[idx] = bucketMin[idx] == -1 ? nums[i] : Math.min(bucketMin[idx], nums[i]);
        bucketMax[idx] = bucketMax[idx] == -1 ? nums[i] : Math.max(bucketMax[idx], nums[i]);
    }
    int pre = 0;
    int result = 0;
    for (int i = 0; i < num; i++) {
        if (bucketMin[i] == -1) {
            continue;
        }
        result = Math.max(result, bucketMin[i] - bucketMax[pre]);
        pre = i;
    }
    return result;
}
```

### Build Post Office

idea:

一组数组，求他们所有元素距离一个已知target距离之和

比如：
[1, 3, 6, 9] 距离 8之和

Step 1. 求数组presum [1, 4, 10, 19]

Step 2, 二分法找到8的位置，这里8位置是2，那么presum(2)是10，那么左边距离总和为 3 * 8 - 10， 右边presum为 19 - 10，距离总和为 9 - 8，这样总体和为14 + 1 = 15。

首先初始化求presum，之后每一次求距离总和只需要logn时间

```java
public int shortestDistance(int[][] grid) {
    // write your code here
    List<Integer> x = new ArrayList<>();
    List<Integer> y = new ArrayList<>();
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == 1) {
                x.add(i);
                y.add(j);
            }
        }
    }
    Collections.sort(x);
    Collections.sort(y);
    List<Integer> sumx = new ArrayList<>();
    List<Integer> sumy = new ArrayList<>();
    sumx.add(0);
    sumy.add(0);
    for (int i = 1; i <= x.size(); i++) {
        sumx.add(sumx.get(i - 1) + x.get(i - 1));
        sumy.add(sumy.get(i - 1) + y.get(i - 1));
    }
    int result = Integer.MAX_VALUE;
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == 0) {
                int xCost = getCost(x, sumx, i);
                int yCost = getCost(y, sumy, j);
                result = Math.min(result, xCost + yCost);
            }
        }
    }
    return result;
}
private int getCost(List<Integer> x, List<Integer> sum, int pos) {
    int left = 0, right = x.size() - 1;
    while (left + 1 < right) {
        int mid = left + (right - left) / 2;
        if (x.get(mid) <= pos) {
            left = mid;
        } else {
            right = mid;
        }
    }
    int idx = 0;
    if (x.get(left) <= pos) {
        idx = left;
    } else {
        idx = right;
    }
    return (idx + 1) * pos - sum.get(idx + 1) - (x.size() - idx - 1) * pos + sum.get(x.size()) - sum.get(idx + 1);
}
```

### [Number of Longest Increasing Subsequence](https://www.lintcode.com/problem/number-of-longest-increasing-subsequence)

A really great dp question, use two array, one keep the lis, one keep the count to achieve this lis.


if lis[i] == lis[j] + 1, count[i] += count[j]

if lis[i] < lis[j] + 1, count[i] = count[j]

```java
public int findNumberOfLIS(int[] nums) {
    // Write your code here
    if (nums == null || nums.length == 0) {
        return 0;
    }
    int[] dp = new int[nums.length];
    dp[0] = 1;
    int[] count = new int[nums.length];
    count[0] = 1;
    int max = 1;
    for (int i = 1; i < dp.length; i++) {
        dp[i] = 1;
        count[i] = 1;
        for (int j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                if (dp[i] == dp[j] + 1) {
                    count[i] = count[i] + count[j];
                } else if (dp[i] < dp[j] + 1) {
                    count[i] = count[j];
                    dp[i] = dp[j] + 1;
                }
                if (dp[i] >= max) {
                    max = dp[i];
                }
            }
        }
    }
    int result = 0;
    for (int i = 0; i < dp.length; i++) {
        if (dp[i] == max) {
            result += count[i];
        }
    }
    return result;
}
```


### Search in Rotated Sorted Array II

If there are duplicate numbers in the array

1. Compare nums[mid], nums[left], if nums[left] < nums[mid], left is sorted. if target >= nums[left] && target < nums[mid], search(nums, target, left, mid - 1). else search(nums, target, mid + 1, right).
2. If nums[mid] < nums[left], right is sorted. if target > nums[mid] and target <= nums[right], search(nums, target, mid + 1, right). else search(nums, target, left, mid - 1).
3. If nums[mid] == nums[left], we compare nums[mid] and nums[right], if nums[mid] != nums[right], we search right side, if nums[mid] == nums[right], we need to search both side.


### Construct Binary Tree from Preorder and Inorder Traversal

Idea:

- [3, 9, 20, 15, 7]
- [9, 3, 15, 20, 7]

3 will be the root, then we search in the inorder array. 3 is the second elements. So the first two elements in the inorder array are left sub tree. Also, the first two elements in the preorder array are left subtree. So 20 is the first element in the right sub tree. Then 20 is the root of right subtree.

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if (preorder == null || inorder == null || preorder.length == 0 || inorder.length == 0) {
            return null;
        }
        return build(preorder, inorder, 0, 0, inorder.length - 1);
    }
    private TreeNode build(int[] preorder,
                           int[] inorder,
                           int pIdx,
                           int iIdx,
                           int iEnd) {
        if (pIdx >= preorder.length) {
            return null;
        }
        if (iIdx == iEnd) {
            return new TreeNode(preorder[pIdx]);
        }
        TreeNode node = new TreeNode(preorder[pIdx]);
        int i = 0;
        for (; i + iIdx < iEnd; i++) {
            if (inorder[iIdx + i] == preorder[pIdx]) {
                break;
            }
        }
        if (i > 0) {
            node.left = build(preorder, inorder, pIdx + 1, iIdx, iIdx + i - 1);
        }
        if (i + iIdx < iEnd) {
            node.right = build(preorder, inorder, pIdx + i + 1, iIdx + i + 1, iEnd);
        }
        return node;
    }
}
```

### Shortest Subarray with Sum at Least K

idea:

- Using deque

```java
while (!deque.isEmpty() && pre[i] - pre[deque.getFirst()] >= K) {
    min = Math.min(i - deque.pollFirst(), min);
}
//保持最小的值在第一位
while (!deque.isEmpty() && pre[i] <= pre[deque.getLast()]) {
    deque.pollLast();
}
```

```
class Solution {
    public int shortestSubarray(int[] A, int K) {
        Deque<Integer> deque = new LinkedList<>();
        int[] pre = new int[A.length + 1];
        for (int i = 1; i < pre.length; i++) {
            pre[i] = pre[i - 1] + A[i - 1];
        }
        int min = Integer.MAX_VALUE;
        for (int i = 0; i < pre.length; i++) {
            while (!deque.isEmpty() && pre[i] - pre[deque.getFirst()] >= K) {
                min = Math.min(i - deque.pollFirst(), min);
            }
            while (!deque.isEmpty() && pre[i] <= pre[deque.getLast()]) {
                deque.pollLast();
            }
            deque.offer(i);
        }
        if (min == Integer.MAX_VALUE) {
            return -1;
        }
        return min;
    }
}
```