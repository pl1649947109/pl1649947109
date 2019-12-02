---
title: epoll——线程相关模块
id: 4
date: 2019-8-22 20:30:00
tags: 第三方模块
comment: true
---

简介：对比select(),poll(),epoll()三个模块。他们都是IO多路复用的具体的实现。

**Select**：对多次调用不友好，内核开销大，监听连接数目有限，不是线程安全。

**Poll**：对其进行了改进，但是还有很多的问题epoll 可以说是I/O 多路复用最新的一个实现。

**epoll **：修复了poll 和select绝大部分问题。 

<!-----more----->

比如：

- 对于每次需要将FD从用户态拷贝至内核态，epoll的解决方案在epoll_ctl函数中                                。每次注册新的事件到epoll句柄中时（在epoll_ctl中指定EPOLL_CTL_ADD），会把所有的fd拷贝进内核，而不是在epoll_wait的时候重复拷贝。epoll保证了每个fd在整个过程中只会拷贝一次。
- 同样epoll也没有1024的连接数限制
- epoll 现在是线程安全的。
- epoll 现在不仅告诉你sock组里面数据，还会告诉你具体哪个sock有数据，你不用自己去找了。 
- epoll的解决方案不像select或poll一样每次都把current轮流加入fd对应的设备等待队列中，而只在epoll_ctl时把current挂一遍（这一遍必不可少）并为每个fd指定一个回调函数，当设备就绪，唤醒等待队列上的等待者时，就会调用这个回调函数，而这个回调函数会把就绪的fd加入一个就绪链表）。epoll_wait的工作实际上就是在这个就绪链表中查看有没有就绪的fd（利用schedule_timeout()实现睡一会，判断一会的效果，和select实现中的第7步是类似的）。
- 于是epoll就出现了。

**总结**

一、select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而epoll其实也需要调用epoll_wait不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在epoll_wait中进入睡眠的进程。虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间。这就是回调机制带来的性能提升。 
二、select，poll每次调用都要把fd集合从用户态往内核态拷贝一次，并且要把current往设备等待队列中挂一次，而epoll只要一次拷贝，而且把current往等待队列上挂也只挂一次（在epoll_wait的开始，注意这里的等待队列并不是设备等待队列，只是一个epoll内部定义的等待队列）。这也能节省不少的开销。

 

**Epoll**：只适用于unix和linux操作系统

原理图：

![img](https://github.com/pl1649947109/pl1649947109.github.io/blob/master/img/epoll.png?raw=true) 

**点集：**

Select.EPOLLIN对应1

select.EPOLLOUT对应4

Select.EPOLLHUP对应16

 

**用法：**

Import select

导入select模块

Epoll = select.epoll()

创建epoll对象

epoll.register(fileno,event)

注册要监控的文件句柄到epoll相关事件ID进行监听

event_type：

Select.EPOLLIN监听读事件

select.EPOLLOUT监听写事件

Select.EPOLLHUP监听断开事件

Select.EPOLLERR 监听错误事件

Epoll.Unregister(fileno)

销毁文件描述符，不再进行监听

Epoll.poll(-1)

当文件描述符发生改变则会以列表的形式主动报告给用户进程，timeout为获取结果的时间单位为秒，当timeout等于-1的时候表示永远等待知道文件描述符发生改变，如果指定为1那么epoll每一秒进行汇报一次当前文件描述符的状态，哪怕是没有文件描述符发生改变是个空值。

Epoll.fileno()

返回epoll的控制文件描述符

Epoll.modfiy(fileno,event)

Fileno：文件描述符

Event：格式的epoll的常数点集

将当前文件描述符转移到指定的一个点集

Epoll.fromfd(fileno)

创建一个epoll fileno文件描述符，从一个给定的控制对象

Epoll.close()

关闭epoll控制，对象将引发异常



**Select模块：**

int select(int maxfdpl, fd_set * readset, fd_set *writeset, fd_set *exceptset, const struct timeval * tiomeout)

第一个是最大的文件描述符长度

第二个是监听的可读集合

第三个是监听的可写集合

第四个是监听的异常集合

第五个是时间限制

以上是C的实现，但是在python中就比较简单了，因为py对它进行了封装，

因此我们的调用就可以这样写：

can_read，can_write，_ = select.select(inputs,outputs,None,None)

参数一：监听可读的套接字

参数二：监听可写的套接字

参数三：监听异常的套接字

参数四：时间限制

**server**

```python

import socket

import select

#创建socket文件句柄

server=socket.socket(socket.AF_INET,socket.SOCK_STREAM)

#设置socket为ip address复用

server.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1)

#将socket句柄绑定到本机的1234端口

server.bind(('0.0.0.0',1234))

#监听最大个数10个

server.listen(10)

#设置非阻碍模式

server.setblocking(0)

#创建一个epoll对象

epoll=select.epoll()

#注册socket文件句柄到 select.EPOLLIN点集

epoll.register(server.fileno(),select.EPOLLIN)

#测试代码不用鸟他

print 'server filno:',server.fileno()

#创建字典来存储 连接 请求

connections={};requests={};responses={}

while True:

#将epoll将发生改变的文件句柄回调到用户进程 用events暂存此数据

events=epoll.poll()

#将文件句柄 和 时间状态从events数据提出来 events = [(文件句柄,时间状态ID)]

for fileno,event in events:

#判断是否为本地socket文件句柄 如果是说明第一次连接

if fileno == server.fileno():

#等待客户端连接

connection,addr=server.accept()

#取客户端的socket句柄整形比如 1 15 212

connFd=connection.fileno()

#设置客户端socket为非阻碍模式

connection.setblocking(0)

#将客户端socket句柄注册到EPOLLIN

epoll.register(connFd,select.EPOLLIN)

#将客户端socket句柄整形 和 客户端socket句柄放到connections字典暂存

connections[connFd]=connection

#判断客户端是否断开连接

elif event & select.EPOLLHUP:

print('close')

#销毁已断开的客户端socket句柄整形的哦

epoll.unregister(fileno)

#关闭客户端socket连接

connections[fileno].close()

#删除客户端暂存数据

del connections[fileno]

#判断客户端是否为收

elif event & select.EPOLLIN:

#接受数据 并strip掉没用的 比如：空格

requests[fileno]=connections[fileno].recv(1024).strip()

#将客户端文件句柄整形的 从EPOLLIN中转移到 EPOLLOUT

epoll.modify(fileno,select.EPOLLOUT)

#判断客户端是否为发

elif event & select.EPOLLOUT:

#发送数据

connections[fileno].send(requests[fileno])

#将客户端文件句柄整形的 从EPOLLOUT中转移到 EPOLLIN

epoll.modify(fileno,select.EPOLLIN)
```

 **client**

```
import socket
client_socket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
host_address = ('192.168.17.128',1234)
client_socket.connect(host_address)
#client_socket.setblocking(0)
while True:
data = raw_input('please input:')
client_socket.sendall(data)
server_data = client_socket.recv(1024)
print server_data
client_socket.close()
```

 

 

 

 

 

 

 

 

 

 