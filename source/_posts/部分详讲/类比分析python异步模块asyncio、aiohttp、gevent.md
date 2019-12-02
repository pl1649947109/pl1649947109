---
title: 类比分析python异步模块asyncio、aiohttp、gevent
id: 4
date: 2019-8-5 20:00:00
tags: python部分内容详讲
comment: true
---

**简述（讲的很好）**

在Python3.6后，可以通过**关键词async def来定义一个coroutine协程，协程就相当于未来需要完成的任务，多个协程就是多个需要完成的任务，多个协程可以进一步封装到一个task对象中，task就是一个储存任务的盒子。**此时，**装在盒子里的任务并没有真正的运行，需要把它接入到一个监视器中使它运行，同时监视器还要持续不断的盯着盒子里的任务运行到了哪一步，这个持续不断的监视器就用一个循环对象loop来实现**。单来说在一个线程里，先后执行 AB 两个任务，但是当A遇到耗时操作（网络等待、文件读写等），这个时候 gevent 会让 A 继续执行，但是同时也会开始执行B任务，如果B在遇到耗时操作同时A又执行完了耗时操作。

#### asyncio示例

```python
import asyncio
import time

#定义第1个协程，协程就是将要具体完成的任务，该任务耗时3秒，完成后显示任务完成
async def to_do_something(i):
    print('第{}个任务：任务启动...'.format(i))
    #遇到耗时的操作，await就会使任务挂起，继续去完成下一个任务
    await asyncio.sleep(i)
    print('第{}个任务：任务完成！'.format(i))
#定义第2个协程，用于通知任务进行状态
async def mission_running():
    print('任务正在执行...')

start = time.time()
#创建一个循环
loop = asyncio.get_event_loop()
#创建一个任务盒子tasks，包含了3个需要完成的任务
tasks = [asyncio.ensure_future(to_do_something(1)),
         asyncio.ensure_future(to_do_something(2)),
         asyncio.ensure_future(mission_running())]
#tasks接入loop中开始运行
loop.run_until_complete(asyncio.wait(tasks))
end = time.time()
print(end-start)
```

说明：

- **启动入口 asyncio.run()**
- **并发运行asyncio任务 asyncio.create_task()**，这里使用future(),效果是一样的，不同的就是task是loop创建的
- **等待对象 await，await用于挂起阻塞的异步调用接口。**
- **休眠 asyncio.sleep() 挂起当前任务，允许允许其他任务**

**实例**：https://chpl.top/2019/09/16/%E7%88%AC%E8%99%AB/%E7%AC%AC%E4%B8%89%E8%AE%B2%E2%80%94%E2%80%94%E9%AB%98%E6%80%A7%E8%83%BD%E5%BC%82%E6%AD%A5%E7%88%AC%E8%99%AB/

**理论**：https://chpl.top/2019/08/22/%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A8%A1%E5%9D%97/asyncio%E2%80%94%E2%80%94%E5%8D%8F%E7%A8%8B%E6%A8%A1%E5%9D%97/

#### aoihttp网络访问

**实例**

```python
import asyncio
from aiohttp import request
from aiomultiprocess import Process

async def put(url, params):
    async with request("PUT", url, params=params) as response:
        pass

async def main():
    p = Process(target=put, args=("https://jreese.sh", ))
    await p

asyncio.run(main())
If you want to get results back from that coroutine, Worker makes that available:

import asyncio
from aiohttp import request
from aiomultiprocess import Worker

async def get(url):
    async with request("GET", url) as response:
        return await response.text("utf-8")

async def main():
    p = Worker(target=get, args=("https://jreese.sh", ))
    response = await p

asyncio.run(main())
If you want a managed pool of worker processes, then use Pool:

import asyncio
from aiohttp import request
from aiomultiprocess import Pool

async def get(url):
    async with request("GET", url) as response:
        return await response.text("utf-8")

async def main():
    urls = ["https://jreese.sh", ...]
    async with Pool() as pool:
        result = await pool.map(get, urls)

asyncio.run(main())
```

外链：https://www.jianshu.com/p/5f41d9fb6b12

#### 同类型的gevent模块

说明：python程序实现的一种单线程下的多任务执行调度器，**简单来说在一个线程里，先后执行 AB 两个任务，但是当A遇到耗时操作（网络等待、文件读写等），这个时候 gevent 会让 A 继续执行，但是同时也会开始执行B任务，如果B在遇到耗时操作同时A又执行完了耗时操作，gevent 又继续执行 A。**

**实例**：模拟通信

服务端

```python
from gevent import monkey, spawn; monkey.patch_all()  # 使用gevent必须 调用  
import socket                                                          
def communicate(conn):                                 
    while True:                                         
        try:                                           
            data = conn.recv(1024)                     
            if not data:break                           
            conn.send(data.upper())                     
        except ConnectionResetError:                   
            break                                       
    conn.close()                                                       


def server(ip, port):                                   
    soc = socket.socket(socket.AF_INET, socket.SOCK_STREAM)            
    soc.bind((ip,port))                                 
    soc.listen(5)                                       
    while True:                                         
        conn, der = soc.accept()                       
        spawn(communicate, conn)  # 协程开启处理IO问题     
    soc.close()                                                        

if __name__ == '__main__':                             
    g = spawn(server, '127.0.0.1', 8080)  # 开启协程, 异步提交                 
    g.join()            
```

客户端

```python
import socket
from threading import Thread,currentThread

def client():
    soc = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    soc.connect(('127.0.0.1', 8080))

    while True:
        soc.send(('%s hello' % currentThread().getName()).encode('utf-8'))
        data = soc.recv(1024)
        print(data.decode('utf-8'))
    soc.close()

if __name__ == '__main__':
    for i in range(500):  # 模拟500个 用户交互
        t = Thread(target=client)  # 线程开启
        t.start()    
```

