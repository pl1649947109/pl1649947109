---
title: mutiprocessing——多进程管理包
id: 5
date: 2019-8-22 20:30:00
tags: 第三方模块
comment: true
---

multiprocessing支持子进程、通信和数据共享、执行不同形式的同步。提供了**Process、Queue、Pipe、Lock**等组件

### Process类

构造方法：

__init__(self, group=None, target=None, name=None, args=(), kwargs={})

**参数说明**

group：扩展参数

target：表示调用对象任务。

name：给子进程取别名。

args：表示条用对象的位置参数元组。

kwargs：表示调用对象的字典。

<!-----more----->

**属性**

deamon:父进程终止后自动终止，且自己不能产生新进程，必须在start()之前设置。

exitcode:进程在运行时为None。

name:进程的名字，自定义。

pid:每个进程都有一个唯一的PID编号。

**方法**

is_alive():判断线程是否存活。

join([timeout])：子进程结束再执行下一步,串行运行。

run():如果创建Process时不指定target，那么就会默认执行Process的run()方法。

start():启动进程。

terminate():终止进程（使用psutil包会更好一些）。

### Pool类

说明：线程池，Pool类可以提供指定数量的进程供用户使用，进程池内维护一个进程序列，当使用时，则去进程池中获取一个进程，如果进程池序列没有提供使用的进程，那么程序就会等待，直到进程池中有可用的进程为止。

**方法**

apply(func[,args[,kwargs]]):同步进程池。

apply_async(func[, args[, kwds[, callback[, error_callback]]]]):异步进程池。

map(func,iterable[]):阻塞 进程知道返回结果。

close():关闭进程池，阻止更多的任务提交到pool，待任务完成后，工作进程会退出。

terminate():结束工作进程，不再处理未完成的任务。

join():wait工作线程的退出，再调用join()前，必须调用close()或则terminate()，这样是因为被终止的进程需要被父进程调用wait(join等价wait),否则进程就会变成僵尸进程。

**Lock**

用的特别的少，一般在线程中使用。因为进程间的数据出现锁的情况非常的低，因为进程之间的本质就是数据不共享。所以就基本不会出现锁的情况。

