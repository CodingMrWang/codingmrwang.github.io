---
layout:     post
title:      Google Calender
subtitle:   Reservation System
date:       2022-01-26
author:     CodingMrWang
header-img: img/post-bg-calendar.png
catalog: true
tags:
    - System Design
    - Google Calendar
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.


## Requirement
### Functional
- Create an event on the calendar
- Send invite to people for the meeting
- Look up all meetings on the calendar
- Send reminder before 30 mins of the meeting
- Cancel the meeting
- Modify the meeting
- Check availability of other members (Out of scope)

### Non-Functional
- High availability
- Consistency
- Latency
- Reliability

## Estimation
### Traffic
- 100M users
- 20M daily active users
- on avg, 2 meetings creation per day -> 40M
- on avg, a user will receive 4 meetings invites -> 80M
- on avg, check calendar twice per day -> 80M

### data size
- 100B/user
- 100B/invites

### QPS
- 40M/86400 = 500/s
- 80M/86400 = 1000/s

### Storage
- user: 100B * 100M = 10GB
- invites: 80 * 365 * 100B = 3000GB
- in total 3TB

## Service

- UserAccount Service
- CalendarEvent Service
- Notification Service

## API
### External
- createMeeting(name, description, startTime, endTime, occuranceFrequency, inviteUsers)
- cancelMeeting(meetingId)
- updateMeeting(meetingId, description, startTime, endTime, occuranceFrequency, inviteUsers)
- checkEvents(userId)

### Internal
- sendInvite(meetingId, user)
- createEvent(name, description, startTime, endTime, user)

## Data Storage
- Since we have high qps and data size, also we don't have really complex sql needed, noSql can be a good choice here.

### Schema
|Meeting||
|---|---|
|id|Hash key|
|create user|string|
|description|string|
|meeting link|string|
|occurance frequency|string|

|Event||
|---|---|
|id|hash key|
|user|global index|
|start time|timestamp|
|end time|timestamp|

|User||
|---|---|
|id|hash key|
|name|string|
|email|string|


## Diagram
![](https://drive.google.com/uc?id=1ofKs9DwFSTaYEl5WKAKrtLiCWDkHkyai)

## Topics
- How to handle reoccuring meeting
	- Create all future meetings ahead / create meetings within a time period, have the scheduler to create future ones.

## Workflow
- User create meeting -> calender service create meeting and event for each user, and send messages to message queue, notification send notification to user for invitation.
- Scheduler check db and send 30 min reminder messages to queue.
- User update meeting -> calender service update events and meeting for each user, send messages to message queue for new invitations.

## Something more
We can cache user events using Redis for quick calendar check, especially for users who has a lot of events.

## Useful link:
- [https://www.youtube.com/watch?v=TYO849M8Q5U](https://www.youtube.com/watch?v=TYO849M8Q5U)



