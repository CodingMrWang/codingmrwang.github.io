---
layout:     post
title:      Design Uber
subtitle:   ride-sharing application
date:       2022-02-06
author:     CodingMrWang
header-img: img/post-bg-uber.jpeg
catalog: true
tags:
    - System Design
    - Uber
---


>> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.

## What is a Uber
Uber enables its customers to book drivers for taxi rides. Both customers and drivers communicate with each other through their smartphones using the Uber app.

## Requirements
### Functional requirements
- Drivers report location
- Rider request uber, match a driver with rider
- Driver deny/accept a request
- Driver cancel a matched request
- Rider cancel a request
- Driver pick up a ride / start a trip
- Driver drop off a rider / end a trip
### Non Functional requirements
- Availability: Customers can request ride anytime and drivers can receive requests and report location at anytime.
- Scability: Can suport large amount of customers and drivers.
- Reliability: data should be consistent.

## Estimation
### Traffic
- 300M cusomters and 1M drivers with 1M daily active customers and 500K daily active drivers.
- 1M daily rides
- Drivers will report their current location every three seconds.
### data size
- 20B/driver location
- 100B/rides
### QPS
- 12 rides/s
- 167k/s
### Storage
customer info: 300M * 100B = 30GB
driver info: 1M * 100B = 100MB
ride info: 1M * 100B * 365 = 40GB/year
location: 1M * 86400 / 3 * 20B = 576GB
## Data Model
```
Trip {
	tripId,
	driverId,
	riderId,
	startLatitude,
	startlongitude,
	endLatitude,
	endLongitude,
	status,
	createdAt
}

Location {
	driverId,
	latitude,
	longitude,
	updatedAt
}
```

## Database Design
It is ineffecient to range search on both latitude and longitude. Use Geohash instead. GeoHash turn latitude and longitude into a string, and places close to each other share the same prefix. The longer the common prefix is, the closer the two points are.

- NoSQL - Cassandra
	- use geohash as column key
	- do range query on geohash
- NoSQL - Redis
	- save driver location by level, if driver location is 9q9hvt, save it in 9q9hvt, 9q9hv, 9q9h these 3 keys.
	- 6 common geohash places are within 1km
	- 4 common geohash places are within 20km
	- key = 9q9hvt, value = set of drivers in this location

## Workflow
### Rider
- rider send request, search drivers within the location range
	- (lat,lng) -> geohash -> [driver1, driver2]
	- search 6 prefix geohash
	- if there is no driver, search 5 prefix
	- if still no driver, search 4 prefix
- If match driver successfully, user search dirver location.

### Driver
- driver report his lat, lng and geohash
	- search his original geohash
	- if there is a change, update redis to new driver set
	- update location in driver table.
- driver accept ride request
	- update trip status
	- update driver table to be unavailable
- driver finish a trip
	- update trip table to be completed
	- update driver table to be available

### Whole solution
1. Rider send ride request, sever create a trip
	- server return trip id back to user
	- user check trip status every 3 seconds
2. Sever find matched driver, update trip and update driver status
3. Driver report his location
	- send message to geo service
	- pull assigned trip every 3 seconds
	- if there is a matched trip, return trip to the driver
4. Driver accept ride request
	- update driver table and trip table
	- rider receive matched trip
5. Driver deny ride request
	- update driver table and trip info
	- re match a new driver


## Diagram
![](https://drive.google.com/uc?id=h1N1opDmqL6gwmozQyCBCatbjW1NFas49b)

## Data sharding
- Shard by the first 4 chars of geoHash
- Shard by city

## Data replication
- Redis Master Slave

## Useful link
- [https://www.cnblogs.com/LBSer/p/3310455.html](https://www.cnblogs.com/LBSer/p/3310455.html)
- [https://en.wikipedia.org/wiki/Geohash](https://en.wikipedia.org/wiki/Geohash)
- [https://www.educative.io/courses/grokking-the-system-design-interview/YQVkjp548NM](https://www.educative.io/courses/grokking-the-system-design-interview/YQVkjp548NM)