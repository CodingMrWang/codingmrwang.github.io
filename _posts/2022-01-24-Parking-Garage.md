---
layout:     post
title:      Parking Garage
subtitle:   Reservation System
date:       2022-01-24
author:     CodingMrWang
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - System Design
    - Parking Garage
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.


## Requirement
### Functional
- Users should be able to reserve a parking lot for a type of vehical.
- Users should be able to pay for the reservation.


### Non-Functional
- Consistency (no two user got reservered for the same parking lot)

### Assumption
- flat rate

## Estimation
### Traffic
- Make reservation 20000/day
- Check reservation 40000/day

### data size
100B/record

### QPS
- 1200000/86400 = 2/s


### Storage
- 146MB / 20 years


## Service

- Reservation Service
- Payment Service

## API
### External
- makeReservation(garageId, userId, startTime, endTime, verhicalType(Optional)) [reservationId, spotId]
- makePayment(reservationId)
- cancelReservation(garageId)
- login/signup (Optional)

### Internal
- getAvailableSpots(garageId, startTime, endTime, verhicalType)
- calculatePrice(reservationId)
- allocateSpot(garageId, time, spotId)

## Data Storage
- We want high consistency, data is not large and qps is not high, postgre could be a good choice.

Tradeoff made here, we want strong consistency higher than latency. One user won't make many reservations per day.

### Schema

|Reservation||
|---|---|
|id|primary key|
|garage_id|foreign key, int|
|spot_id|foreign key, int|
|start|timestamp|
|end|timestamp|
|paid|boolean|


|Garage||
|---|---|
|id|primary key|
|zip_code|varchar|
|compact rate|decimal|
|large rate|decimal|
|electric rate|decimal|

|Spot||
|---|---|
|id|primary key|
|level|int|
|location|varchar|
|type|enum|

|User||
|---|---|
|id|primary key|
|email|varchar|


## Diagram
![](https://drive.google.com/uc?id=1jhXc6u_n9HVmE7FvClRE4GD6o1IVpBwU)


## Useful link:
- [https://www.youtube.com/watch?v=NtMvNh0WFVM](https://www.youtube.com/watch?v=NtMvNh0WFVM)



