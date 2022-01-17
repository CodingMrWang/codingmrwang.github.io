---
layout:     post
title:      Top K Problem System Design
subtitle:   Heavy Hitter
date:       2022-01-17
author:     CodingMrWang
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - System Design
    - Top K
    - MapReduce
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.


## Requirement
### Functional
- TopK(k, startTime, endTime)

### Non-Functional
- Scalable (scales out together with increasing amount of data: videos, tweets, posts, etc.)
- Highly available (survive hardware/network failure, no single point of failure)
- Highly performant
- Accurate (would be trade off between highly performant and accurate)
- Data persistence

## Estimation
### Request & data size
- 100M records per day
- 100B each record

### QPS
- 1157 qps

### Storage
- 10GB

## Service

Top K Service

### How to implement
1. HashMap + Min Heap

	- Use HashMap count frequency of each element
	- When an element comes, check if it exists in the map
 		- If yes, increase the frequency, compare with freq of min element in the heap, if it is larger than it, pop the least element in the heap and insert the new element		- If No, insert into map

 	- Time Complexity: O(N) 
 	- Space Complexity: O(N)
 	
	But if data size is large, we cannot save all data in a single machine.
	
2. Multiple machine HashMap + Min Heap

	- Shard the incoming data and save to multiple machine, in each machine, use a hashMap and a heap to maintain a max k frequent elements heap.
	- Send all k size heap to a single machine and combine into a single k-size heap

	Still cannot fit large size data. Memory size is limited.
	
3. Database + Heap
	- Shard the incoming data and save to database.
	- Sort each database to get top k items.
	- Combine k items from each database into a single heap and get top K

   Can save large amount of data, but if write qps is really large, need cache to save write queries, if machine crash, cache data will loss. So not good for large write requests.
   
4. Count-Min Sketch + Heap
   
   No really accurate but high performant.
   
   - A two dimension array
   		- Width is usually in thousands, depth is small (can be 5, 5 different hash function)
   	- Whenever an element comes, calculate 5 hash value base on 5 hash function, take the smallest value as its frequent.
   	- Update heap base on frequency of the new element.
	- Count-Min sketch is a fixed size data structure.

   - ![](https://drive.google.com/uc?id=1iIn-QAwxzXMbuZ0ABeWZjjXqGVjd18WV)

5. Map-Reduce
	- We can persist data in a distributed file system and do periodic Map-Reduce task to get most frequent k elements.
   - Map-Reduce is a time-consuming task, so cannot get update to date top k list, but it is accurate.
   - Need two Map-Reduce job, one to count frequency of each elements and get top k for each partition. one to get total top k frequent elements.

### Put all together

![](https://drive.google.com/uc?id=1y_dqL_OtQYGGEEmdjKtDUXDX7wpRl4rX)

1. User call API Gateway which will create a log.
2. Distributed Message system could be Kafka, Kinesis or SQS. Random partitioning.
3. Fast Processor
	- Creates count-min sketch and aggregates data for a short period of time
	- Memory is no longer a problem, no need to partition the data.
 	- Data replication is nice to have, but not strictly required.
4. Storage can be SQL/NoSQL
   - Builds final count-min sketch and stores a list of top k elements for a period of time.
   - Data replication is required.
5. Data Prtitioner
	- Parses batches of events into individual events
	- Hash partitioning
	- Deals with hot partitions
6. Parition Processor
	- Aggregates in-memory over the course of several minutes.
	- Generates files of specified size
7. Finally, save map reduce result to storage as well.

### Get top K
![](https://drive.google.com/uc?id=159fxvM0k22Z6P3yJN9OkG1qYOM_O2dV1)

