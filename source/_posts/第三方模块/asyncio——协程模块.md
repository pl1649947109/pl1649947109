---
title: asyncio——协程模块
id: 2
date: 2019-8-22 20:30:00
tags: 第三方模块
comments: true
---

前言：python由于GIL（全局锁）的存在，不能发挥多核的优势，其性能一直饱受诟病。然而在IO密集型（阻塞）的网络编程中，异步处理比同步处理能提升成百上千倍的效率，弥补了python新跟性能方面的短板。

### asyncio能干什么

- 异步网络操作
- 并发
- 协程

### 同步和异步

**概念：**

同步：指的是完成事务的逻辑，先执行一个事务，如果阻塞了会一直的等待，直到事务完成，再去执行第二个事务，顺序执行........

异步：和同步时相对的，异步在处理调用事务的时候，不会等待这个事务的处理结果，直接就去处理第二个事务，通过状态、通知、回调来处理结果。

<!-----more----->

**同步代码**

```python
import time

def hello():
    time.sleep(1)

def run():
    for i in range(5):
        hello()
        print('Hello World:%s' % time.time())  # 任何伟大的代码都是从Hello World 开始的！
if __name__ == '__main__':
    run()
```

**异步代码**

```python
import time
import asyncio

# 定义异步函数
async def hello():
    asyncio.sleep(1)
    print('Hello World:%s' % time.time())

def run():
    for i in range(5):
        loop.run_until_complete(hello())  
#把异步的任务丢到循环方法.run_until_complete中
loop = asyncio.get_event_loop()   
#主线程
if __name__ =='__main__':
run()
```

### asyncio的关键字

- event_loop （事件循环）：程序开启一个无限循环，把一些函数注册到事件循环上，当满足事件发生的时候，调用相应的协程函数。
- coroutine （协程）：协程对象，指一个使用async关键字定义的函数，它的调用不会立即执行函数，而是会返回一个协程对象。协程对象需要注册到事件循环，由事件循环调用。
- task （任务）：一个协程对象就是一个原生可以挂起的函数，任务则是对协程进一步封装，其中包含了任务的各种状态。
- future: 代表将来执行或没有执行的任务的结果。它和task上没有本质上的区别。
- async/await 关键字：python3.5用于定义协程的关键字，async定义一个协程，await用于挂起阻塞的异步调用接口。

### 开启协程学习篇章

**定义一个协程**

```python
import time
import asyncio

now = lambda : time.time()

async def do_some_work(x):
    print("waiting:", x)
start = now()
# 这里是一个协程对象，这个时候do_some_work函数并没有执行
coroutine = do_some_work(2)
print(coroutine)
#  创建一个事件loop
loop = asyncio.get_event_loop()
# 将协程加入到事件循环loop 
loop.run_until_complete(coroutine)
print("Time:",now()-start)
```

**创建一个task**

说明：协程对象不能直接运行，在注册事件循环的时候，其实是run_until_complete方法将协程包装成为了一个任务（task）对象. task对象是Future类的子类，保存了协程运行后的状态，用于未来获取协程的结果。

```python
import asyncio
import time

now = lambda: time.time()

async def do_some_work(x):
    print("waiting:", x)
    
start = now()

coroutine = do_some_work(2)

loop = asyncio.get_event_loop()

task = loop.create_task(coroutine)
print(task)

loop.run_until_complete(task)
print(task)

print("Time:",now()-start)
```

**绑定回调**

说明：在task执行完成的时候可以获取执行的结果，回调的最后一个参数future对象，通过该对象可以获取协程的返回值。

```python
import time
import asyncio

now = lambda : time.time()

async def do_some_work(x):
    print("waiting:",x)
    return "Done after {}s".format(x)

def callback(future):
    print("callback:",future.result())
    
start = now()

coroutine = do_some_work(2)

loop = asyncio.get_event_loop()

task = asyncio.ensure_future(coroutine)
print(task)

task.add_done_callback(callback)
print(task)

loop.run_until_complete(task)
print("Time:", now()-start)
```

**阻塞和await**

说明：使用async可以定义协程对象，使用await可以针对耗时的操作进行挂起，就像生成器里的yield一样，函数让出控制权。协程遇到await，事件循环将会挂起该协程，执行别的协程，直到其他的协程也挂起或者执行完毕，再进行下一个协程的执行。

耗时的操作一般是一些IO操作，例如网络请求，文件读取等。我们使用asyncio.sleep函数来模拟IO操作。协程的目的也是让这些IO操作异步化。

```python
import asyncio
import time

now = lambda :time.time()

async def do_some_work(x):
    print("waiting:",x)
    # await 后面就是调用耗时的操作
    await asyncio.sleep(x)#模拟了阻塞或者耗时的操作，这个时候就会让出控制权。
    return "Done after {}s".format(x)

start = now()

coroutine = do_some_work(2)
loop = asyncio.get_event_loop()
task = asyncio.ensure_future(coroutine)
loop.run_until_complete(task)

print("Task ret:", task.result())
print("Time:", now() - start)
```

### 并行和并发

```python
import asyncio
import time

now = lambda :time.time()

async def do_some_work(x):
    print("Waiting:",x)
    await asyncio.sleep(x)
    return "Done after {}s".format(x)

start = now()

coroutine1 = do_some_work(1)
coroutine2 = do_some_work(2)
coroutine3 = do_some_work(4)

tasks = [
    asyncio.ensure_future(coroutine1),
    asyncio.ensure_future(coroutine2),
    asyncio.ensure_future(coroutine3)
]

loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.wait(tasks))
for task in tasks:
    print("Task ret:",task.result())
print("Time:",now()-start)
```

### 协程的嵌套

```python
import asyncio
import time

now = lambda: time.time()

async def do_some_work(x):
    print("waiting:",x)
    await asyncio.sleep(x)
    return "Done after {}s".format(x)

async def main():
    coroutine1 = do_some_work(1)
    coroutine2 = do_some_work(2)
    coroutine3 = do_some_work(4)
    tasks = [
        asyncio.ensure_future(coroutine1),
        asyncio.ensure_future(coroutine2),
        asyncio.ensure_future(coroutine3)
    ]
    for task in asyncio.as_completed(tasks):
        result = await task
        print("Task ret: {}".format(result))

start = now()

loop = asyncio.get_event_loop()
loop.run_until_complete(main())print("Time:", now()-start)
```

### 协程停止

实现结束task有两种方式：关闭单个task、关闭loop，涉及主要函数：

- asyncio.Task.all_tasks():获取事件循环任务列表
- KeyboardInterrupt:捕获停止异常（Ctrl+C）
- loop.stop():停止任务循环
- task.cancel():取消单个任务
- loop.run_forever()
- loop.close():关闭事件循环，不然会重启

### future对象的几个状态

- Pending
- Running
- Done
- Cacelled

解释说明：创建future的时候，task为pending，事件循环调用执行的时候当然就是running，调用完毕自然就是done，如果需要停止事件循环，就需要先把task取消。可以使用asyncio.Task获取事件循环的task。

### 处理和网页有关

介绍：当网页发送http请求的时候，通常使用的时requests，这是同步的库，如果想用异步的话需要引入aiohttp，这里引入一个类，from aiohttp import ClientSession，首先要建立一个session对象，然后用session对象去打开网页。session可以进行多项操作，比如post, get, put, head等。

**基本用法：**

async with ClientSession() as session:

async with session.get(url) as response:

**Aiohttp异步实现**

```python
import asynciofrom aiohttp import ClientSession

tasks = []
url = "https://www.baidu.com/{}"
async def hello(url):    #定义一个异步函数
async with ClientSession() as session:
        async with session.get(url) as response:
            response = await response.read()#await关键字加载需要等待的操作前面，response.read(）等待request响应
            print(response)
if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(hello(url))
```

**多链接异步访问**

```python
import time
import asyncio
from aiohttp import ClientSession

tasks = []
url = "https://www.baidu.com/{}"
async def hello(url):
    async with ClientSession() as session:
        async with session.get(url) as response:
            response = await response.read()#            print(response)
            print('Hello World:%s' % time.time())
def run():
    for i in range(5):   #请求多个URL时，同步就需要就在一个for循环里面，但是在异步里面，我需要把hello函数包装
#在asyncio的Future对象中，然后将Future对象列表作为任务传递给事件循环
        task = asyncio.ensure_future(hello(url.format(i)))
        tasks.append(task)

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    run()
    loop.run_until_complete(asyncio.wait(tasks))
```

**收集http响应**

说明：上面我们实现了访问不同连接的异步实现方式，我们发出了请求，如何把相应一一收集到一个列表中，最后打印出来呢？可以通过asyncio.gather(*tasks)将相应全部收集起来。

```python
import time
import asyncio
from aiohttp import ClientSession

tasks = []
url = "https://www.baidu.com/{}"
async def hello(url):
    async with ClientSession() as session:
        async with session.get(url) as response:#            print(response)
            print('Hello World:%s' % time.time())
            return await response.read()
def run():
    for i in range(5):
        task = asyncio.ensure_future(hello(url.format(i)))
        tasks.append(task)
    result = loop.run_until_complete(asyncio.gather(*tasks))
    print(result)
if __name__ == '__main__':
    loop = asyncio.get_event_loop()
```

**异常解决**

说明：当我们的并发是2000个，程序就会报错，这个是由于操作系统的限制，linux打开文件最大的默认数是1024个，windows默认的是509，超过了这个值，程序开始报错。我们的解决方案是：限制并发数量、使用回调方式、修改操作系统的打开最大文件数的限制。

```python
import time
import asyncio
from aiohttp import ClientSession

tasks = []
url = "https://www.baidu.com/{}"
async def hello(url):
    async with ClientSession() as session:
        async with session.get(url) as response:#            print(response)
            print('Hello World:%s' % time.time())
            return await response.read()
def run():
    for i in range(5):
        task = asyncio.ensure_future(hello(url.format(i)))
        tasks.append(task)
    result = loop.run_until_complete(asyncio.gather(*tasks))
    print(result)
if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    run()
异常解决：
当我们的并发是2000个，程序就会报错，这个是由于操作系统的限制，linux打开文件最大的默认数是1024个，windows默认的是509，超过了这个值，程序开始报错。我们的解决方案是：限制并发数量、使用回调方式、修改操作系统的打开最大文件数的限制。
在这里我们限制并发的数量：
import time,asyncio,aiohttp


url = 'https://www.baidu.com/'
async def hello(url,semaphore):
    async with semaphore:
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as response:
                return await response.read()


async def run():
    semaphore = asyncio.Semaphore(500) # 限制并发量为500
to_get = [hello(url.format(),semaphore) for _ in range(1000)] #总共1000任务    
await asyncio.wait(to_get)    #等待执行，有相当于回到了同步一样

if __name__ == '__main__':#    now=lambda :time.time()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(run())
    loop.close()
```

