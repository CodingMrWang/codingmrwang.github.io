---
layout:     post
title:      Design Donation App
subtitle:   System Design
date:       2022-02-06
author:     CodingMrWang
header-img: img/post-bg-facebook.jpg
catalog: true
tags:
    - System Design
    - Donation
---


>> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.

## Background

- You are required to design a donation app for a 3 day charity event across the US where you are expecting 3 million donations.
- You should accept details like customer name, credit card details etc.
- No pay specific knowledge is required (presume that you’d integrate with a 3rd party pay provider like Stripe or Braintree).


## Requirements
### Functional requirements
- Register/Login
- User can make donation to a charity
- User can view past donation history

### Non Functional requirements
- Availability (be able to donate anytime, need data replication)
- Scability (partition)
- Reliability (data consistent, no duplicate donation and no duplicate charge)
- Latency (not that important here)

## Estimation
### Traffic
- 3M donations over 3 days
- 10RPS (peak could be 100rps)

## API
submitDonation(user_id, charity_id, amount, payment_method)
viewDonations(user_id)

## Service

User Service: register, login
Payment Method Service: get/update payment methods
Donation Service: submit and view donations
PaymentAttempt Service: submit payment and handle failed payments
Notification Service: send notification to customers

## Data Model
|User||
|---|---|
|user_id|pk|
|email|string|
|name|string|

|Payment method||
|---|---|
|id|pk|
|user_id|fk|
|card_number|string|
|cridential|string|

|Donation||
|---|---|
|donation_id|pk|
|user_id|fk|
|charity_id|fk|
|amount|int|
|status|string|

|Payment Attempt||
|---|---|
|attempt_id|pk|
|donation_id|fk|
|payment_method_id|fk|
|amount|int|
|status|string|

We can use relational database and put donation and payment attempt into the same db to enforce transaction, but according to single responsibility, we use two micro services to handle those data. we can use dynamodb or relational db. But since our query is simple, we can use dynamodb.

## Diagram
![](https://drive.google.com/thumbnail?id=17VeeB7X0tadcwiICsNBRxtt8Mp2JLX1L&sz=w1000)

## Workflow

The happy path would be user create donation, donation service create donation data record with pending status and then call payment attempt service, payment succeeded and return donation 200, donation update status to SUCCEEDED and return user 200.

While third party payment may fail anytime, we can enqueue the failed task to the queue and retry with a exp backoff, if it failed 5 times, we can update the status to be failed and send notification to user, if it succeeded, we will also send notification to user. For sending notification and update donation status, we do it async, send a message to sqs and let donation service and notification service to consume it.

## Failure handling

1. No duplicate donation: when making a donation, check if there is any donation with same user, charity, amount within one minute, if there is, ask user if they really want to make the donation.

2. No duplication payment: check if payment app if idempontent. also can use a status at donation to ensure no duplicate payment.

3. if data is not consistent between payment attempt and donation, run a cron job to sync them.

4. Read replica for data, if master die, select one slave to be master.

## Scale

Data partition shard by user id. for payment attempt, we can shard by donation id.


## Useful link
- [https://chivagarg.medium.com/system-design-for-dummies-part-3-design-a-donation-app-c9f2720222ce](https://chivagarg.medium.com/system-design-for-dummies-part-3-design-a-donation-app-c9f2720222ce)
