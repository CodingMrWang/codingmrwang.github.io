---
layout:     post
title:      Blocking Problem of queue in multithreading
subtitle:   A problem troubled me for a long time
date:       2018-07-31
author:     CodingMrWang
header-img: img/post_bg_mountain.jpg
catalog: true
tags:
    - Queue
    - Multithread
---


> This post is first created by [CodingMrWang](http://codingmrwang.github.io), 作者 [@Zexian Wang](http://github.com/codingmrwang) ,please keep the original link if you want to repost it.

When I use multithread with queue like this, my program always cannot quit gracefully.

```python
class Test(object):
	def __init__(self):
		self.queue = Queue.Queue()
	def get_relationship(self):
		while self.queue.qsize():
			data = queue.get()
			self.deal_with_data(data)
	def run(self):
		threads = [threading.Thread(target=self.get_relationship) for _ in range(10)]
		[u.start() for u in threads]
		[u.join() for u in threads]
if __name__ == '__main__':
	test = Test()
	for data in all_data:
		test.queue.put(data)
	test.run()
``` 

This is because of the blocking mechanism of queue. When queue is empty, if you use queue.get(), it will keep waiting until there is data come into the queue.
In multithread, if many threads come to the condition test, there is one element in the queue, all of them will enter the while loop, however, only one of them can get the last element, other threads would kept waiting and the thread will not quit.
There are two methods to solve this problem.

1. sleep for a random time when the qsize of queue is 1.

	```python
	def get_relationship(self):
		if q.qsize() == 1:
			sleep(randint(2, 4))
		while q.qsize():
			data = q.get()
			...
	```
	Then, when there is only one element in the queue, the thread first finish the sleep will enter the while loop and empty the queue, then other thread would quit.

2. Set block = Flase
		
	```python
	def get_relationship(self):
		while q.qsize():
			try:
				data = q.get(block=False)
			except:
				break
	```
	The default setting of get() function is block = True, so if you set block = False and the queue is empty, it will throw an exception, you need to catch the exception. Also, you can use get(0) which is almost the same with the above method.

I prefer the second method, if there are lots of threads, some of them may sleep at the same time and sleep for the same random number would also cause the problem. So using get(block=False) or get(0) with an exception catching is a better method.