---
layout:     post
title:      Rate Limiter
subtitle:   Local and distributed
date:       2022-01-21
author:     CodingMrWang
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - System Design
    - Rate Limiter
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.


## Requirement
### Functional
- allowRequest(request)
  - Limit the number of requests an entity can send to an API within a time window, e.g., 15 requests per second.

### Non-Functional
- Low latency (should not introduce substantial latencies affecting the user experience)
- Accurate
- Scable
- Available (maybe not high priority, we can do more than less)

## Estimation
### data size

- Data size 20 bytes per user if only user id & timestamp & count

### QPS
- 1000/s - 1M/s

### Storage
- 20B * 1M = 20MB

## Service

- Client Identifier Builder: (build unique identifier for each request user)
- Rate Limiter: decide whether we want to throttle the request.
- Throttle Rules Cache


## API
- allowRequest(userId, count)

## Single host solution
![](https://drive.google.com/uc?id=1guIbwYoXpjWb5BtGkK3JdfLLj2aqCjIY)

Data can be saved in memory as the size is not really large. Rules can also be cached in memory, we can cache the rules in another server as a single service or put into a single server, client identifier builder as well.

## Algorithms

### Fixed Window Algorithm
![](https://drive.google.com/uc?id=1yj5ZTmCRD2fL9L53lK3q7sljJ1fFl-yR)
We can save data in a hashtable:
Key: Value
UserId: {Count, StartTime}

- If the ‘UserID’ is not present in the hash-table, insert it, set the ‘Count’ to 1, set ‘StartTime’ to the current time (normalized to a minute), and allow the request.

- Otherwise, find the record of the ‘UserID’ and if CurrentTime – StartTime >= 1 min, set the ‘StartTime’ to the current time, ‘Count’ to 1, and allow the request.

- If CurrentTime - StartTime <= 1 min and

	- If ‘Count < 3’, increment the Count and allow the request.
	- If ‘Count >= 3’, reject the request.
#### Problem: 
	- It can allow twice of number of requests per time period.
	
### Sliding Window algorithm

Store timestamp of each request in a sorted set
Key: Value
UserId: {1499818000, 1499818500, 1499818600}

Let’s assume our rate limiter is allowing three requests per minute per user, so, whenever a new request comes in, the Rate Limiter will perform following steps:

- Remove all the timestamps from the Sorted Set that are older than “CurrentTime - 1 minute”.

- Count the total number of elements in the sorted set. Reject the request if this count is greater than our throttling limit of “3”.

- Insert the current time in the sorted set and accept the request.

#### Problem
- Much more memory needed, need to save all timestamp.

### Token Bucket Algorithm
- Bucket has max capacity
- Current tokens <= max capacity
- tokens arrive at fixed rate

```java
public class TokenBucket {
	private final long maxBucketSize;
	private final long refillRate;
	
	private double currentBucketSize;
	private long lastRefillTimestamp;
	
	public synchronized boolean allowRequest(int token) {
		refill();
		if (currBucketSize > tokens) {
			currBucketSize -= tokens;
			return true;
		}
		return false;
	}
	private void refill() {
		long now = System.nanoTime();
		double tokensToAdd = (now - lastRefillTimestamp) * refillRate / 1e9;
		currentBucketSize = Math.min(tokensToAdd + currentBucketSize, maxBucketSize);
		lastRefillTimestamp = now;
	}
}

```

### Interface & Classes
![](https://drive.google.com/uc?id=1V5a4i3cyVbn6XL7a6RUDNaOSmZNvYiUf)

### Distrubuted System

When come to distributed system, we may create bucket for the same user in different hosts. Need a way to broadcast number of token consumed in each host.
![](https://drive.google.com/uc?id=1w8mHitLYeB76lzwgRFZ-0gGlhfLR-HZ5)

### Something else

- If there are millions of users, we need to keep millions of buckets in our memory?

	We can delete userId which haven't made request recently


## Useful link:
- [https://www.youtube.com/watch?v=FU4WlwfS3G0](https://www.youtube.com/watch?v=FU4WlwfS3G0)
- [https://www.educative.io/courses/grokking-the-system-design-interview/3jYKmrVAPGQ#1.-What-is-a-Rate%C2%A0Limiter?](https://www.educative.io/courses/grokking-the-system-design-interview/3jYKmrVAPGQ#1.-What-is-a-Rate%C2%A0Limiter?)




