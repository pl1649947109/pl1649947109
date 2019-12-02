---
title: async——源码解读
id: 6
date: 2019-11-18 20:00:00
tags: 知识扩展
comment: true
---

### 内容解读

PEP 492 - 具有异步和等待语法的协同程序

此PEP假定异步任务类似于stdlib模块*asyncio.events.AbstracEventLoop*的事件循环调度和协调。虽然PEP不依赖任何特定的事件循环实现，但它仅与使用yield作为调度器程序信号的协程类型相关，表明协同程序将等待直到事件完成。（如IO）

新的协程声明语法：

```python
async def read_data(db):
	pass
```

**协程的关键属性**

- async def函数总是协程，及时他们不包含awai表达式。
- 从异步函数中的表达式获得yield或yield是一个SystaxError。
- 在内部：
  - CO_CORUTTNE用于标记本机协同程序。
  - CO_ITERABLE_COROUNTINE用于使基于生成器的协同程序与本机协同程序兼容。
- 常规生成器在被调用时返回一个生成器对象；同样，协同程序返回一个协程对象。
- StopIteration异常不会从协程传播出去，而是被RuntimeError替换。
- 当垃圾手机本机协程时，如果从未等待过，则会引发RuntimeWarning

<!----more---->

**等待表达**

```
async def read_data(db):
	data = await db.fetch('SELECT...')
```

await，**类似于yield from **,暂停执行的read_data协程，直到db.fetch等待完成并返回结果数据。

**更新运算符优先级表**

| Operator                                                     | Description                                                 |
| :----------------------------------------------------------- | :---------------------------------------------------------- |
| `yield` `x`, `yield from` `x`                                | Yield expression                                            |
| `lambda`                                                     | Lambda expression                                           |
| `if` -- `else`                                               | Conditional expression                                      |
| `or`                                                         | Boolean OR                                                  |
| `and`                                                        | Boolean AND                                                 |
| `not` `x`                                                    | Boolean NOT                                                 |
| `in`, `not in`, `is`, `is not`, `<`, `<=`, `>`, `>=`,`!=`, `==` | Comparisons, including membership tests and identity tests  |
| `|`                                                          | Bitwise OR                                                  |
| `^`                                                          | Bitwise XOR                                                 |
| `&`                                                          | Bitwise AND                                                 |
| `<<`, `>>`                                                   | Shifts                                                      |
| `+`, `-`                                                     | Addition and subtraction                                    |
| `*`, `@`, `/`, `//`, `%`                                     | Multiplication, matrix multiplication, division, remainder  |
| `+x`, `-x`, `~x`                                             | Positive, negative, bitwise NOT                             |
| `**`                                                         | Exponentiation                                              |
| `await` `x`                                                  | Await expression                                            |
| `x[index]`, `x[index:index]`,`x(arguments...)`, `x.attribute` | Subscription, slicing, call, attribute reference            |
| `(expressions...)`,`[expressions...]`, `{key:value...}`, `{expressions...}` | Binding or tuple display, list display, dictionary display, |

**等待表达式的实例**

| `if await fut: pass`           | `if (await fut): pass`            |
| ------------------------------ | --------------------------------- |
| `if await fut + 1: pass`       | `if (await fut) + 1: pass`        |
| `pair = await fut, 'spam'`     | `pair = (await fut), 'spam'`      |
| `with await fut, open(): pass` | `with (await fut), open(): pass`  |
| `await foo()['spam'].baz()()`  | `await ( foo()['spam'].baz()() )` |
| `return await coro()`          | `return ( await coro() )`         |
| `res = await coro() ** 2`      | `res = (await coro()) ** 2`       |
| `func(a1=await coro(), a2=0)`  | `func(a1=(await coro()), a2=0)`   |
| `await foo() + await bar()`    | `(await foo()) + (await bar())`   |
| `-await foo()`                 | `-(await foo())`                  |

**异步上下文管理器和异步**

一个异步上下文管理器是一个上下文管理器，它能够暂停其执行进入和退出的方法。为了实现这一点，提出了一种异步上下文管理器的新协议。添加了两个新的魔法：__aenter__和__aexit__。两者都必须返回等待。

实例

```python
class AsyncCotextManager:
	async def __aenter__(self):
		await log("进入上下文")
	async def __aexit__(self,exc_type,exc,tb):
		await log("退出上下文")
```

**新语法**

```python
async with EXPR as VAR:
	BLOCK
```

上述的新语法就相当于下面的代码：

```python
mgr = (EXPR)
aexit = type(mgr).__aexit__
aenter = type(mgr).__aenter__(mgr)

VAR = await aenter
try:
	BLOCK 
except:
	 if not await aexit(mgr, *sys.exc_info()):
        raise
else:
	await aexit(mgr, None, None, None)
```

解释：与常规的with语句一样，可以在单个async with 语句中指定多个上下文管理器。将没有__aenter__和__aexit__方法的常规上下文管理器传递给异步是错误的，在异步def函数之外使用async是一个SyntaxError。

### 注意事项

为什么异步和等待关键字？

很久之前C#就有了，还有很多其他的语言都有。这是一个巨大的好处，因为一些用户已经具有异步/等待的经验，并且因为它使用得在一个项目中使用多种语言更加容易。

async关键字的重要性？

虽然可以实现await表达式并将至少一个await处理所有函数作为协程程序处理，但这种方法使得API设计，代码重构和长时间支持变得更加困难

```python
假装python只有await关键字：
def func():
	await log(...)
def important():
	await fun()
```

如果func()函数被重构并且有人重中删除所有await表达式，它将称为常规python函数，并且依赖它的所有代码，都将被破坏。为了缓解这个问题，必须引入类似于@asyncio.coroutine的装饰器。

为什么异步def?

对于一些人来说哦，async func()：传递语法可能看起来比async def name()更加有吸引力。但是另一方面，它打破了async def,async之间的对称性，其中async是一个修饰符，声明该语句是异步的他也与现在的语法更加一致。

为什么不重用现有的for和with语句？

现在基于生成器的协程和此前提议背后的愿景是让用户可以轻松查看代码可能被挂起的位置。使现在for和with语句识别异步迭代器和上下文管理器不可避免地创建隐式挂起点，这使得更加难推理代码。

**对比异步函数和生成器之间的性能差异**

```python
import sys
import time

def binary(n):
	if n <= 0:
		return 1
	l = yield from bindary(n - 1)
	r = yield fron bindary(n - 1)
	return l + 1 +r
	
async def abinary(n):
	if n <= 0:
		return 1
	l = await abinary(n - 1)
	r = await abinary(n - 1)
	return l + 1 + r
	
def timeit(func,depth,repeat):
	 t0 = time.time()
	 for _ in range(repeat):
	 	o = func(depth)
	 try:
	 	while True:
	 		o.send(None)
	 except StoIteration:
	 	pass
	 t1 = time.time()
	 print ('{}({}) * {}: total {:.3f}s'.format(
        func.__name__, depth, repeat, t1-t0))
结果：
binary(19) * 30: total 53.321s
abinary(19) * 30: total 55.073s

binary(19) * 30: total 53.361s
abinary(19) * 30: total 51.360s

binary(19) * 30: total 49.438s
abinary(19) * 30: total 51.047s
```

**工作实例**

```python
import asyncio

async def echo_server():
    print('Serving on localhost:8000')
    await asyncio.start_server(handle_connection,
                               'localhost', 8000)

async def handle_connection(reader, writer):
    print('New connection...')

    while True:
        data = await reader.read(8192)

        if not data:
            break

        print('Sending {:.10}... back'.format(repr(data)))
        writer.write(data)

loop = asyncio.get_event_loop()
loop.run_until_complete(echo_server())
try:
    loop.run_forever()
finally:
    loop.close()
```

### 补充

```
之前使用Python的人往往纠缠在多线程，多进程，评判哪个效率更高？
其实，相对于别家的协程和异步，不管多线程还是多进程效率都要被吊打，多线程之间切换耗费cpu寄存器资源，OS 调度的不太可控，多进程间通信不便的问题。
后来Python改进了语法，引入了yiled from充当协程调度，后来有人根据这个新特性开发了第三方协程框架，Tornado，Gevent等。
在这场效率之争里，Python这么受欢迎的语言，官方怎么能默不出声？所以Python之父深入简出3年，苦心钻研自家的协程，async/await和asyncio库，并放到Python3.5后成为远程原生的协程，
对于类似爬虫这种延时的IO操作，协程是个大利器，优点很多，他可以在一个阻塞发生时，挂起当前程序，跑去执行其他程序，把事件注册到循环中，实现多程序并发，据说超越了10k限制，不过我没有试验过极限。

作者：予岁月以文明
链接：https://www.jianshu.com/p/7690edfe9ba5
来源：简书
```

协程一次发起100个请求（其实也是一个一个的发），不同的是协程发送一个请求，挂起，再发送下一个请求，再挂起，发起100个，挂起100个。然后等待100个返回，效率提升了100倍。可以理解为同时做了100件事情，相当于多线程，做到了自己调度而不是交给CPU，程序流程可控，节省资源，效率极大提升。

```python
async def get(url):
	async with aiohttp.ClientSession() as session:
		async with session.get(url) as response:
			return await response.text()
#await是挂起命令，挂起当前，执行response.text()，response.text()执行完后重新激活当前函数运行，返回。如果response迟迟不回，程序不会死等，而是去我们定义的任务循环中寻找另一个任务，如果没有循环任务，那就只能死等，毕竟总是要有返回结果的。如下图：
```

![](http://9017499461.linshutu.top/await.webp)

可以从上图中看出：任务一直在跑，每到一个地方await一次，然后返回await，直到最终全部返回。主程序结束。

**调用协程**

协程不能直接运行，组要把协程注册到事件循环（loop）.asyncio.get_event_loop方法可以创建一个事件循环，然后run_until_complete将协程注册到事件循环，并启动事件循环。

```python
import time
import asyncio

now = lambda: time.time()
async def do_some_work(x)：
	print("Waiting:",x)
start = now()
coroutine = do_some_work(2)
loop = asyncio.get_event_loop()
loop.run_until_complete(coroutine)
print ("TIME:",now() - start)
结果：
Waiting: 2
TIME: 0.004952669143676758
```

**关于task**

协程对象不能直接运行，在注册事件循环的时候，其实run_until_complete方法将协程包装成一个任务（task）对象。这个task保存了协程运行后的状态，用来未来获取协程的结果。

```python
import asyncio
import time 
now = lambda:time.time()
async def do_some_work(x):
	print ("Waiting:",x)
start = now()
coroutine = do_some_work(2)
loop = asyncio.get_event_loop()

task = loop.create_task(coroutine)
print (task)
loop.run_until_complete(task)
print (task)
print ("TIME:",now() - start)
结果：
<Task pending coro=<do_some_work()>
Waiting: 2
<Task finished coro=<do_some_work() done>
TIME: 0.004949092864990234
```

**从上面的代码可以看出来，创建task后，task在加入事件循环之前是pending状态，当loop事件循环开始，所有的pending状态的task都开始执行到await那一步（函数体内不是这样），不管loop里面是否调用。**

结论：asyncio.ensure_future(coroutine) 和 loop.create_task(coroutine)都可以创建一个task，run_until_complete的参数是一个futrue对象。当传入一个协程，其内部会自动封装成task，task是Future的子类。isinstance(task, asyncio.Future)将会输出True。
**两个例子**

```python
import asyncio
import time

start = time.time()


def tic():
    return 'at %1.1f seconds' % (time.time() - start)


async def gr1():
    # Busy waits for a second, but we don't want to stick around...
    print('gr1 started work: {}'.format(tic()))
    # 暂停两秒，但不阻塞时间循环，下同
    await asyncio.sleep(2)
    print('gr1 ended work: {}'.format(tic()))


async def gr2():
    # Busy waits for a second, but we don't want to stick around...
    print('gr2 started work: {}'.format(tic()))
    await asyncio.sleep(2)
    print('gr2 Ended work: {}'.format(tic()))


async def gr3():
    print("Let's do some stuff while the coroutines are blocked, {}".format(tic()))
    await asyncio.sleep(1)
    print("Done!")

# 事件循环
loop = asyncio.get_event_loop()

# tasks中也可以使用asyncio.ensure_future(gr1())..
#tasks = [
#    loop.create_task(gr1()),
#    loop.create_task(gr2()),
#    loop.create_task(gr3())
#]
tasks = [gr1(), gr2(), gr3() ]  #简便的写法
loop.run_until_complete(asyncio.wait(tasks))
loop.close()

结果：
gr1 started work: at 0.0 seconds
gr2 started work: at 0.0 seconds
Let's do some stuff while the coroutines are blocked, at 0.0 seconds
Done!
gr2 Ended work: at 2.0 seconds
gr1 ended work: at 2.0 seconds
解释：
asyncio.wait(...)协程的参数是一个由future或协程构成的可迭代对象，wait会分别把各个协程包装进Task对象。
```

```python
生产者消费者模型
import time
import asyncio
from asyncio import Queue


def now(): return time.time()


async def worker(q):#工作者消费队列
    print('Start worker')

    while 1:#无限循环
        start = now()
        task = await q.get()#开始消费
        if not task:
            await asyncio.sleep(1)
            continue
        print('working on ', int(task))
        await asyncio.sleep(int(task))
        q.task_done()#队列通知
        print('Job Done for ', task, now() - start)


async def generate_run(q):#生成worker线程函数
    asyncio.ensure_future(worker(q))
    asyncio.ensure_future(worker(q))#先弄了两个worker去跑
    await q.join()主线程挂起等待队列完成通知
    jobs = asyncio.Task.all_tasks()完成后收集所有线程，这里是3个，算上自己
    print('是否已经关闭任务', asyncio.gather(*jobs).cancel())#关闭线程方法，返回True


def main():

    loop = asyncio.get_event_loop()
    q = Queue()
    for i in range(3):
        q.put_nowait(str(i))#一定要放入字符，数字0是空，队列一直不会结束。
    loop.run_until_complete(generate_run(q))#启动生成函数

    loop.close()


if __name__ == '__main__':
    main()
```

