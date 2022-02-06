---
layout:     post
title:      Design Yelp
subtitle:   Proximity Server
date:       2022-02-05
author:     CodingMrWang
header-img: img/post-bg-time.jpeg
catalog: true
tags:
    - System Design
    - Yelp
---


>> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.

## What is Proximity Server
Proximity servers are used to discover nearby attractions like places,

## Requirements
### Functional requirements
- User should be able to add/delete/update places.
- Given their location (longitude/latitude), users should be able to find all nearby places within a given radius.
- Users should be able to add feedback/review about a place.

### Non Functional requirements
- Low latency
- Scability

## Estimation
### Traffic
500M DAU search three times/day
100K restaurant update/day
### data size
793B/restaurant metadata (locationId 8B, Name 256B, latitude 8B, longitude 8B, description 512B, category 1b)
### QPS
read: 500M * 3/86400 = 17360/s
write: 100K/86400=1.2/s
### Storage
500M restaurants in total
500M * 1KB = 500GB

## API
search(user_id, search_terms, user_location, radius_filter, maximum_results_to_return_category_filter, sort, page_token)
return: a json containing information about a list of businesses matching the search query. Each result entry will have the business name, address, category, rating and thumbnail.

## Diagram

![](https://drive.google.com/uc?id=1g8KvK83u_LOP-puxIKYm3WCjFRD7mRNg)

## Database
This will be a read heavy but write light application, users will search restaurants everyday but they won't always update restaurant info.

- Naive solution: Store all data in a relational database and use query like ```select * from places where latitude between X-D and X+D and longitude between Y-D and Y+D```
This query is really inefficient, for each condition, it will retrieve many records and performing the intersection on two lists will be slow.

- Grids: We can divide the whole map into smaller grids to group locations into smaller sets. Each grid will store all the places within the range of longitude and latitude.
We can use a NoSQL to store places like dynamoDB/cassandra, using region like zip code as hash key and gridId as range Key to build index.

- Keep index in memory: We can keep our index in a hash table where key is the grid number and value is the list of places contained in that grid. If we have 20 millions grids, we would need a four bytes number to uniquely identify each grid.
	(4 * 20M) + (8 * 500M) = 4GB
while this solution can run slow if grids have a lot of places since they may not uniformly distributed among grids.

- Dynamic size grids: We can limit the size of each grid to 500. Use a tree to store grids, if there are more than 500 places in the grid, split it into multiple children.

![](https://drive.google.com/uc?id=1q-hAs4dRqLFuvidS6qsiLyqRnspv_CoZ)

**How we build a QuadTree?** We start with root and break it down into four nodes and distribute locations among them, we keep repeating the process with each child node until there are no nodes left with more than 500 locations.
**How we find the grid for a given location?** We will start with the root node and search downward to find our required node. at each step, we will see if the current node we are visiting has children, if it has, we will move to the child node that contains our desired location and repeat this process. If the node doesn't have any children, then that is our desired node.
**How will we find neighboring grids of a given grid?** We can connect all leaf nodes with a double linked list. Then we can iterate forward and backward among neighboring leaf nodes to find out our desired locations.
**How much memory will be needed to store the QuadTree**
24 * 500M = 12GB
500M / 500 = 1M grids
This can be easily fit into a modern-day server.

## Data Partitioning
Share QuadTree into multiple machines

- Shard based on regions: may cause hot partition, can let load balancer to handle that.
- Share based on locationId: When we search, we need query all servers and send to a centralized server.

## Replication and fault tolerance
Having multiple replica of QuadTree. If loss all QuadTree, build it by scanning all data in database.

## Cache
Save hot places info in cache. can use LRU cache.

## Another solution
[GeoHash](https://en.wikipedia.org/wiki/Geohash)

## Useful link
- [https://www.educative.io/courses/grokking-the-system-design-interview/B8rpM8E16LQ](https://www.educative.io/courses/grokking-the-system-design-interview/B8rpM8E16LQ)
- [https://medium.com/double-pointer/system-design-interview-yelp-or-nearby-proximity-service-5258359c421c](https://medium.com/double-pointer/system-design-interview-yelp-or-nearby-proximity-service-5258359c421c)
- [https://en.wikipedia.org/wiki/Quadtree](https://en.wikipedia.org/wiki/Quadtree)