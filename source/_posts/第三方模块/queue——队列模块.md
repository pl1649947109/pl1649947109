---
title: queue——队列模块
id: 7
date: 2019-8-22 20:30:00
tags: 第三方模块
comment: true
---

介绍：队列是线程中最常用的数据交换的形式。在python中，多个线程之间的数据是共享的，多个线程进行数据交换的时候，不能够保证数据的安全性和一致性，这个时候队列就出现了，它完美的解决了线程间数据的交换，安全和一致性的问题。

<!-----more----->

##### 队列对象的创建

```python
import queue
q = queue.Queue(maxsize = 10)
queue.queue类即是一个队列的同步实现。队列长度可为无限或者有限。可通过queue的构造函数的可选参数maxsize来设定队列长度。如果maxsize小于1的整数就表示队列长度无限。
```

##### Python queue模块有三种队列及构造函数

- Python queue模块的FIFO队列先进先出。

  class  Queue(maxsize)   

- LIFO类似于堆，即先进后出。

  class LifoQueue(maxsize)                      

- 还有一种是优先级队列级别越低越先出来。

  class PriorityqQueue(maxsize)

##### 常用的方法

q = queue.Queue()

- q.qsize() 返回队列的大小。
- q.empty() 如果队列为空，返回True,反之False。
- q.full() 如果队列满了，返回True,反之False。
- q.full 与 maxsize 大小对应。
- q.get([block[, timeout]]) 获取队列，timeout等待时间。
- q.get_nowait() 相当q.get(False)非阻塞 。
- q.put(item) 写入队列，timeout等待时间。
- q.put_nowait(item) 相当q.put(item, False)。
- q.task_done() 在完成一项工作之后，q.task_done() 函数向任务已经完成的队列发送一个信号。
- q.join() 实际上意味着等到队列为空，再执行别的操作。
- q.clear()清空队列。