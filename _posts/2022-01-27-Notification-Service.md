---
layout:     post
title:      Notification Service
subtitle:   Producer Consumer
date:       2022-01-26
author:     CodingMrWang
header-img: img/post-bg-calendar.png
catalog: true
tags:
    - System Design
    - Notification Service
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.


## Requirement
### Functional
- CreateTopic
- Publish message to topic
- subscribe a topic and receive message

### Non-Functional
- Scalable (supports an arbitaraily large number of topics, publishers and subscribers)
- Highly available (Survives hardware/network failures, no single point of failure)
- Highly performant (end-to-end latency as low as possible)
- Durable (messages won't loss, subscriber receive message at least once)

## Estimation
### Traffic
- 1k topics
- 10k messages/topic per day
- 2k subscribers

### data size
- 200B/topic
- 200B/subscriber
- 200B/message

### QPS
- 1k*10k / 86400 = 120/s

### Storage
- user: 10M * 200B = 2GB/day

## API
### External
- createTopic(topicName)
- publish(topicName, message)
- subscribe(topicName, endpoint)

## Service
FrontEnd Service
- ![](https://drive.google.com/uc?id=1NOpnY9BDDeVhjoXIZ0KkrBy_qpMa9pKH)

MetaData Service
- A caching layer between the FrontEnd and a persistent storage.
- Many reads, little writes.

Temporary Storage
- Fast, highly-available and scalable web service
- Has to store messages for serveral days to handle unavailability of a subscriber

Sender
- ![](https://drive.google.com/uc?id=1z8F_wlpGgvg2vbBW3Wb1TC5S9fdp0meI)

### Database
SQL vs NoSQL
1. don't need transaction.
2. No complexy queries
3. Not for warehouse or analytics
NoSQL wins. (Column/key-value databases like DynamoDB, Cassendra)

In memory storages: Redis
Message queues: SQS/Kafka

## Something more

How to handle spammer: register subscriber, sign agreement.
Duplicate messages: Frontend remove duplicates

## Useful link:
- [https://www.youtube.com/watch?v=TYO849M8Q5U](https://www.youtube.com/watch?v=TYO849M8Q5U)



