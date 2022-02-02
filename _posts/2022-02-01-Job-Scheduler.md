---
layout:     post
title:      Distributed Job Scheduler
subtitle:   Task scheduling
date:       2022-02-01
author:     CodingMrWang
header-img: img/post-bg-time.jpeg
catalog: true
tags:
    - System Design
    - Job Scheduler
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.


## Requirement
### Functional
- Clients can schedule tasks to execute at a specified time, it can be scheduled for one time or multiple executions.
- Tasks can be associated with a priority. Tasks with higher priority should get executed before tasks with a lower priority once they are ready for execution.
- Client can query the status of a scheduled task.

### Non-Functional
- Reliability: At least one task execution
- Scalability: thousands or even millions of jobs can be scheduled and run per day
- Availability: It should always be possible to schedule and execute jobs

### Restrictions
- Idempotence: a single task of a job can be executed multiple times, developers should ensure that job logic and correctness of task execution in clients are not affected by this.
- Resiliency: Worker processes which execute tasks might die at any point during task execution. The system may retry abruptly interrupted tasks, which could also be retried on different hosts. Job owners should design their job such that retries on different host don't affect job correctness.

## Service

**Frontend**
Client use frontend to schedule job requests.

**Task Store**
Task store stores all job information and task scheduling information.

**Task consumer**
This service will periodically poll tasks from task store to find tasks that are ready for execution and pushes them onto right queues.

```
repeat every second
1. poll tasks ready for execution from task store
2. push tasks onto right queues
3. update task statuses to be enqueued
```

The task consumer poll tasks that failed in earlier execution attempts, this helps with at-least-once guarantee.

**Queue**
Use AWS SQS to queue tasks internally. These queues act as a buffer between the store consumer and controllers. Each job has a dedicated SQS queue.

**Controller**
Worker hosts are physically hosts dedicated for task execution. Each worker host has one controller process responsible for polling tasks from SQS queues in a background thread, and then pushing them onto process local buffered queues. The controller is only aware of the jobs it is serving.
Multiple executor will poll tasks from controller, controller maintain a priorityQueue and return tasks to executor base on priorities.

**Executor**
The executor is a process with multiple threads, responsible for actual task execution. Each thread within an executor process follows the loop:
```
while True:
	w = get_next_work()
	do_work(w)
```

**Status controller**
Status controller is responsible for updating task status in task store. After controller pull a task from SQS queue, it will call Status controller to update job status as claimed and set next trigger timestamp for task, then after the task is finished by executor, executor will call status controller as well to update job status to be finished/failed/retriable.

![](https://drive.google.com/uc?id=1J6lJqOQZOPktPqp_jyVBLaZjg2XhsWa5)

## Storage

The database will have low write requests but high read requests. We don't create task really often, but task consumer will scan the database quite often. We can use dynamodb and have index on job id.

### Data model

Job: job id, job info, status, createdTime, createdBy, frequency, queue
Task: job id, next execution time, status

|Job status|Task status|next trigger timestamp in task| comment|
|---|---|---|---|
|new|new|scheduled_timestamp|pick up new tasks that are ready|
|enqueued|started|enqueued_timestamp + enqueue_timeout|re-enqueue task if it has been in enqueued state for too long. This can happen if queue losses data or controller goes down after polling the queue.|
|claimed|started|claimed_timestamp + claim_timeout|re-enqueue if task is claimed but never transfered to processing. This can happen if controller is down after claiming a task.|
|processing|started|heartbeat_timestamp+heartbeat_timeout|Re-enqueue if task hasn't sent heartbeat for too long, this can happen if executor is down. Task status is changed to enqueued after re-enequeue|
|retriable failure|started|compute next_timestamp according to backoff logic|exponential backoff for tasks with retriable failure|
|success|completed/new|N/A/scheduled timestamp|when task finished, get ready for next execution|
|fatal_failure|completed|N/A||

The task consumer polls for tasks base on the following query:

```
task_status = && next_timestamp <= time.now();
```

## Achieving guarantees

**At-least-one task execution**
This is guaranteed by task retrying. If task was dropped by any part of system, it can always be picked again and retried. 

**Isolation**
Isolcation of jobs is achieved through dedicated worker clusters, dedicated queues and dedicated per-job scheduling quotas.

**Delivery latency**
This services doesn't require ultra-low task delivery latencies. Task delivery latencies on the order of a couple of seconds are acceptable. Tasks ready for execution are periodically polled by task consumer and this period of polling largely controls the task delivery latency.


## Useful link:
- [https://dropbox.tech/infrastructure/asynchronous-task-scheduling-at-dropbox](https://dropbox.tech/infrastructure/asynchronous-task-scheduling-at-dropbox)
- [https://www.1point3acres.com/bbs/thread-161803-1-1.html](https://www.1point3acres.com/bbs/thread-161803-1-1.html)
- [https://leetcode.com/discuss/general-discussion/1082786/System-Design%3A-Designing-a-distributed-Job-Scheduler-or-Many-interesting-concepts-to-learn](https://leetcode.com/discuss/general-discussion/1082786/System-Design%3A-Designing-a-distributed-Job-Scheduler-or-Many-interesting-concepts-to-learn)


