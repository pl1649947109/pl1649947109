---
title: threading——多线程模块
id: 9
date: 2019-8-22 20:30:00
tags: 第三方模块
comment: true
---

threading模块提供的类(组件):**Thread、Lock、Rlock、Event、Timer、ocal**

### threading之Thread

介绍：Thread类代表一个分离的控制线程活动。当线程的实例被创建，通过调用实例start()方法闯进活动，它将在一个分离的线程控制器中运行run()方法。

##### 构造器：

**threading.Thread(group=None,target=None,name=None,args=(),kwargs={},\*,daemon=None)**

参数说明：

```
Group：保留扩展

Target：可调用对象，可以被run()方法调用

Name：线程的名字

Args:参数，调用的元组

Kwagrs：参数，调用的关键字字典
```

<!-----more----->

##### 方法：

- start():开启线程。
- run():代表线程的活动，在子类中可以进行run方法重写，也可以作为target参数传入。
- join(timeout=None):等待直到该线程终止。
- name():标记目的线程的名字。
- deamon:一个bool，标记这个线程是否为守护线程，必须在start()之前设置。
- is_alive():当前线程是否存活。

### threading之Lock/Rlock

介绍：一个原始锁就是一种原始的同步方法，当上锁时，不会被特殊的线程获取，在python中，他是目前最低级别的原始同步方法，通过_thread线程扩展模块实现。就是将并发变成串行，从而保护数据的安全。

##### 方法：

- acquire()：获取一个锁。
- release():释放一个锁。

参见：https://chpl.top/2019/08/23/%E4%B9%A6/%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E4%B8%8E%E5%B9%B6%E5%8F%91/%E7%AC%AC%E4%B8%89%E5%8D%81%E5%9B%9B%E8%AE%B2%E2%80%94%E2%80%94%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E4%B9%8B%E7%BA%BF%E7%A8%8B%E4%BA%8C/

### threading之Event

　Event（事件）是最简单的线程通信机制之一：一个线程通知事件，其他线程等待事件。Event内置了一个初始为False的标志，当调用set()时设为True，调用clear()时重置为 False。wait()将阻塞线程至等待阻塞状态。

　　Event其实就是一个简化版的 Condition。Event没有锁，无法使线程进入同步阻塞状态。

**实例方法**

- is_set(): 当内置标志为True时返回True。 
- set(): 将标志设为True，并通知所有处于等待阻塞状态的线程恢复运行状态。 
- clear(): 将标志设为False。 
- wait([timeout]): 如果标志为True将立即返回，否则阻塞线程至等待阻塞状态，等待其他线程调用set()。

### Threading模块提供的方法：

- threading.active_count():返回存活的Thread对象数量。 ***

- threading.current_thread():返回当前线程，对应调用者的控制线程。***

- threading.enumerate:返回当前存活线程列表，这个列表包括守护线程，虚拟线程，主线程。

- threading.main_thread():返回主线程，一般就是python解释器的启动线程。

- threading.stack_size([size]):当新的线程创建时，返回线程栈的大小

- threading.TIMEOUT_MAX:支队功能锁方法（Lock.acquire(),RLock.acquire(),Condition.wait(),e)，允许设置timeout的最大值，设置超过此值的timeout将会引起报错,就是设置全局的超时时间。