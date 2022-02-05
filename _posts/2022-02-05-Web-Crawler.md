---
layout:     post
title:      Design Web Crawler
subtitle:   web spiders
date:       2022-02-05
author:     CodingMrWang
header-img: img/post-bg-time.jpeg
catalog: true
tags:
    - System Design
    - Web Crawler
---


>> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.

## What is a Web Crawler
A web crawler is a software program that browses the websites in a methodical and automated manner. It collects documents by recursively fetching links from a set of start pages(seeds). Many sites, particularly search engines, use web crawling as a means of providing up-to-date data. Search engines download all the pages to create an index on them to perform faster searches.

## Requirements
### Functional requirements
- Crawl web pages
### Non Functional requirements
- Scalability: Be able to crawl a large amount of web pages.
- Reliability: Not blocked.
- Duarbility: Persist data
### Questions for interviewer
- Is the crawler for HTML pages only? do we need to crawl media, sound files, images as well.
- What is the expected number of pages we will crawl.

## Estimation
### Traffic
15 billion pages per month
### data size
100KB/web page
### QPS
15B/(4 * 7 * 86400) = 6200 pages/s
### Storage
15B * 100KB = 1.5 petabytes

## Diagram
![](https://drive.google.com/uc?id=1-up7370TNlZx106m0MFsLOrWmP7Uq_9y)

**Domain name resolution**: Before contacting a Web server, a Web crawler must use the Domain Name Service (DNS) to map the Web server’s hostname into an IP address. DNS name resolution will be a big bottleneck for our crawlers given the huge number of URLs we will be working with. To avoid repeated requests, we can start caching DNS results by building our local DNS server.

**Web Crawler**: Cralwer to send HTTP requests to urls and retrieve web pages. We should use multi process & multi threads to crawl data. Too many context switch will cause CPU utilization go down, we have have multiple server and each server start 100 processes.

**Document Checksum store**: In order to not save duplicate document, we can take checksum of the document, then check document checksum store to see if we save this document before, if not, we can save the document to DFS and extract the document. Then we can use extractor to urls from the html, then save url and domain to the url store.

**Url Store**: It should be a key value store which support high read throughput. We need to save domain name and url in the store. If we crawl a single website too frequently, our IP will be blocked, each domain has its own protocal, so we can save domain and crawl rate in a domain table, for each url, we can set a next crawl time. This db can be replicated with read replica, and we can shard by domain name.

**Job Scheduler**: This scheduler will monitor the url store and schedule jobs for every five minutes (can be shorter or longer). Find all urls which have next crawl time smaller than now, then send them to sqs.

**SQS**: In order to decouple producer and consumer, we use sqs so that crawler can crawl with its own speed.

### Workflow
1. Web crawler will start with some seeds websites, crawl web pages and save documents in DFS and urls in kv store.
2. Job scheduler will monitor url store, if it reaches next crawl time, job scheduler will send the url to sqs and update their next crawl time.
3. Web crawler will handle sqs messages and crawl web pages.
4. Extractor will check if we crawl same document before by checking its checksum with checksum store, if not, we will extract urls and save urls into url store if they are not in the store.
5. When web crawler crawl websites, it can directly get DNS from DNS resovler if DNS cache the IP address.
6. We can also cache frequently used url so that we don't need to check url store each time whether the url has been crawled before.

**Useful link**
- [https://www.educative.io/courses/grokking-the-system-design-interview/NE5LpPrWrKv](https://www.educative.io/courses/grokking-the-system-design-interview/NE5LpPrWrKv)
