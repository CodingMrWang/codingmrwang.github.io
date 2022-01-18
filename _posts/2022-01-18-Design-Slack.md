---
layout:     post
title:      Design Slack
subtitle:   Chat App
date:       2022-01-18
author:     CodingMrWang
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - System Design
    - Slack
    - Chat
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.


## Requirement
### Functional
- Direct Message
- Channel: Group chat
- Multimedia msg
- Chat preview

### Non-Functional
- Highly available
- Highly reliable
- Consistent
- High performance: real time
- Highly Scalable

## Estimation
### Request & data size
- DAU: 10M
- 100 bytes / Msg

### QPS
- Peak Direct Message: 5 * 10M * 100 / 86400 = 50k/s

### Storage
- 50k * 100 = 5MB/s
- 5MB * 86400 = 5TB

## Service

Account service: manage user account info
Group service: manage group membership info
Media service: process multi-media
Message storage service: store message
Notification service: send notification
Channel service: as gateway
Message search service: manage messaging search index and lookup

## Diagram
![](https://drive.google.com/uc?id=1jcYKffK1z25vQ0hDnSXKBfWHV4iwF2F0)

## API
- GetChannels(user_id)
	- last viewed channels
	- visible channels
	- Show unread msg/last msg preview

- SendMessage(sender_id, receiver_id, message_type, message, media_id)
	- message type could be group message/direct message.
	- timestamp/expire time/STATE will be appended by channel service.
	- STATE could be [received, sent, viewed, notification_sent, deleted]
- updateMessage/deleteMessage(message_id, message)
- uploadFile
- LookupMessage

## Internal API
- Media service:
	- processMedia(media_id)
	  - Replicate media
	  - Post process
	  - distribute to CDN
- Notification service
	- notifyUser(user_id, sender_id, message_id, msg_preview)
	- Kafka
	- two phase commits to ensure exactly once.
## Storage
### Data Model

User table(SQL)

| Id(Shard Key) | name | status | timezone | team |
|---------------|------|--------|----------|------|

Chat table (Cassandra)

| user_id | chat_id | status | join date | last view time |
|---------|---------|--------|-----------|----------------|

last view time can be used to find last view msg.

Message table(DynamoDB)

| (chat_room_id)PartitionKey | timestamp-msg_id(Sort key) | Msg id | Sender id | message |
|----------------------------|----------------------------|--------|-----------|---------|

We don't find message by message id, we only find message by chat room. Get all messages from a room and retrieve only a certain period of time messages.

Channel Table(Redis)

Since we will have a lot of reads at each moment, using database could be slow, we can use redis to cache frequent read info

|key|value|
|-|-|
|user_id|David: {unread_msg: 10, last_msg_preview: hey, timestamp: 22343}|
|  |Company Chat: {unread_msg: 10, last_msg_preview: office, timestamp: 1234|
|-|-|


## Workflow

- User send message, load balancer will call channel service, channel service will call message storage service to save the message first, then send a message to kafka, notification service consume message and use socket to notify user. after message storage service save the message, it will update redis and send message to search service. elastic search will used to build index and search message.

- User read message, check redis, then call message storage service to get last n messages of the chat room. after that, update redis.

## Bonus

Emoji service
Emoji update really frequent for message, so create a single service for it. use redis to save emoji id and count.

Data is not that important, so use redis. Can back up in disk.

## Extra
Apple use APNS (Apple push Notification service) to send notification.
Web use web socket.

## Useful link:
[https://docs.google.com/document/d/1h3xoOvewFLf0KJGCNRhFV9HiK1W_ehHc6wG4bjUPTTc/edit](https://docs.google.com/document/d/1h3xoOvewFLf0KJGCNRhFV9HiK1W_ehHc6wG4bjUPTTc/edit)




