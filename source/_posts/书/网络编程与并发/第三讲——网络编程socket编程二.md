---
title: 第三讲——socket编程二
id: 3
date: 2019-8-16 20:00:00
tags: socket和并发编程
comment: true
---

### 学习大纲

- 基于TCP连接的主机间的通讯
- 远端执行控制台命令
- 粘包
- 两种情况下会发生粘包
- 粘包解决的方案
- 高大上版解决粘包问题
- 基于UDP协议的socket通信

<!-----more----->

### 基于TCP连接的主机间的通讯

**单个客户端和服务端循环通信**

- server

  ```python
  import socket
  
  ss = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
  
  ss.bind(('127.0.0.1',8080))
  
  ss.listen(5)
  
  conn, client_addr = ss.accept()
  print(conn, client_addr, sep='\n')
  
  while 1:  # 循环收发消息
      try:
          from_client_data = conn.recv(1024)
          print(from_client_data.decode('utf-8'))
      
          conn.send(from_client_data + b'SB')
      
      except ConnectionResetError:
          break
  
  conn.close()
  ss.close()
  ```

- client

  ```python
  import socket
  
  cs = socket.socket(socket.AF_INET,socket.SOCK_STREAM) 
  
  cs.connect(('127.0.0.1',8080))  # 与客户端建立连接， 拨号
  
  while 1:  # 循环收发消息
      client_data = input('>>>')
      cs.send(client_data.encode('utf-8'))
      
      from_server_data = cs.recv(1024)
      
      print(from_server_data.decode('utf-8'))
  
  cs.close()  # 挂电话
  ```

**多个客户端和服务器循环连接通讯**

- server

  ```python
  import socket
  
  ss = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
  
  ss.bind(('127.0.0.1',8080))
  
  ss.listen(5)
  
  while 1 : # 循环连接客户端
      conn, client_addr = ss.accept()
      print(client_addr)
      
      while 1:
          try:
              from_client_data = conn.recv(1024)
              print(from_client_data.decode('utf-8'))
          
              conn.send(from_client_data + b'SB')
          
          except ConnectionResetError:
              break
  
  conn.close()
  ss.close()
  ```

- client

  ```python
  import socket
  
  cs = socket.socket(socket.AF_INET,socket.SOCK_STREAM)  # 买电话
  
  cs.connect(('127.0.0.1',8080))  # 与客户端建立连接， 拨号
  
  while 1:
      client_data = input('>>>')
      cs.send(client_data.encode('utf-8'))
      
      from_server_data = cs.recv(1024)
      
      print(from_server_data.decode('utf-8'))
  
  cs.close()  # 挂电话
  ```

**recv的工作原理**

```python
'''
源码解释：
Receive up to buffersize bytes from the socket.
接收来自socket缓冲区的字节数据，
For the optional flags argument, see the Unix manual.
对于这些设置的参数，可以查看Unix手册。
When no data is available, block untilat least one byte is available or until the remote end is closed.
当缓冲区没有数据可取时，recv会一直处于阻塞状态，直到缓冲区至少有一个字节数据可取，或者远程端关闭。
When the remote end is closed and all data is read, return the empty string.
关闭远程端并读取所有数据后，返回空字符串。
'''
----------服务端------------：
# 1，验证服务端缓冲区数据没有取完，又执行了recv执行，recv会继续取值。

import socket

phone =socket.socket(socket.AF_INET,socket.SOCK_STREAM)

phone.bind(('127.0.0.1',8080))

phone.listen(5)

conn, client_addr = phone.accept()
from_client_data1 = conn.recv(2)
print(from_client_data1)
from_client_data2 = conn.recv(2)
print(from_client_data2)
from_client_data3 = conn.recv(1)
print(from_client_data3)
conn.close()
phone.close()

# 2，验证服务端缓冲区取完了，又执行了recv执行，此时客户端20秒内不关闭的前提下，recv处于阻塞状态。

import socket

phone =socket.socket(socket.AF_INET,socket.SOCK_STREAM)

phone.bind(('127.0.0.1',8080))

phone.listen(5)

conn, client_addr = phone.accept()
from_client_data = conn.recv(1024)
print(from_client_data)
print(111)
conn.recv(1024) # 此时程序阻塞20秒左右，因为缓冲区的数据取完了，并且20秒内，客户端没有关闭。
print(222)

conn.close()
phone.close()


# 3 验证服务端缓冲区取完了，又执行了recv执行，此时客户端处于关闭状态，则recv会取到空字符串。

import socket

phone =socket.socket(socket.AF_INET,socket.SOCK_STREAM)

phone.bind(('127.0.0.1',8080))

phone.listen(5)

conn, client_addr = phone.accept()
from_client_data1 = conn.recv(1024)
print(from_client_data1)
from_client_data2 = conn.recv(1024)
print(from_client_data2)
from_client_data3 = conn.recv(1024)
print(from_client_data3)
conn.close()
phone.close()
------------客户端------------
# 1，验证服务端缓冲区数据没有取完，又执行了recv执行，recv会继续取值。
import socket
import time
phone = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
phone.connect(('127.0.0.1',8080))
phone.send('hello'.encode('utf-8'))
time.sleep(20)

phone.close()



# 2，验证服务端缓冲区取完了，又执行了recv执行，此时客户端20秒内不关闭的前提下，recv处于阻塞状态。
import socket
import time
phone = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
phone.connect(('127.0.0.1',8080))
phone.send('hello'.encode('utf-8'))
time.sleep(20)

phone.close()

# 3，验证服务端缓冲区取完了，又执行了recv执行，此时客户端处于关闭状态，则recv会取到空字符串。
import socket
import time
phone = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
phone.connect(('127.0.0.1',8080))
phone.send('hello'.encode('utf-8'))
phone.close()
```

### 远端执行控制台命令

**subprocess模块**

```python
import subprocess

def foo(cmd):
	obj = subprocess.Popen(cmd,
	                       shell=True,
	                       stdout=subprocess.PIPE,
	                       stderr=subprocess.PIPE,
	                       )

	print (obj.stdout.read().decode('gbk'))  # 正确命令
	print(obj.stderr.read().decode('gbk'))  # 错误命令
#subprocess下的Popen函数就是我们的和控制台交互的函数，这里面的的参数以及用法可以参考我的blog。https://pl1649947109.github.io/
```

**远程执行控制台命令**

- server

  ```python
  import socket
  import subprocess
  
  ss = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
  
  ss.bind(('127.0.0.1',8080))
  
  ss.listen(5)
  
  while 1 : # 循环连接客户端
      conn, client_addr = ss.accept()
      print(client_addr)
      
      while 1:
          try:
              cmd = conn.recv(1024)
              ret = subprocess.Popen(cmd.decode('utf-8'),shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
              correct_msg = ret.stdout.read()
              error_msg = ret.stderr.read()
              conn.send(correct_msg + error_msg)
          except ConnectionResetError:
              break
  
  conn.close()
  ss.close()
  ```

- client

  ```python
  import socket
  
  cs = socket.socket(socket.AF_INET,socket.SOCK_STREAM)  # 买电话
  
  cs.connect(('127.0.0.1',8080))  # 与客户端建立连接， 拨号
  
  
  while 1:
      cmd = input('>>>')
      cs.send(cmd.encode('utf-8'))
      
      from_server_data = cs.recv(1024)
      
      print(from_server_data.decode('gbk'))
  
  cs.close()  # 挂电话
  ```

### 粘包

**首先了解socket缓冲区问题**

![img](http://9017499461.linshutu.top/7.png)

```
每个 socket 被创建后，都会分配两个缓冲区，输入缓冲区和输出缓冲区。

write()/send() 并不立即向网络中传输数据，而是先将数据写入缓冲区中，再由TCP协议将数据从缓冲区发送到目标机器。一旦将数据写入到缓冲区，函数就可以成功返回，不管它们有没有到达目标机器，也不管它们何时被发送到网络，这些都是TCP协议负责的事情。

TCP协议独立于 write()/send() 函数，数据有可能刚被写入缓冲区就发送到网络，也可能在缓冲区中不断积压，多次写入的数据被一次性发送到网络，这取决于当时的网络情况、当前线程是否空闲等诸多因素，不由程序员控制。

read()/recv() 函数也是如此，也从输入缓冲区中读取数据，而不是直接从网络中读取。

这些I/O缓冲区特性可整理如下：

1.I/O缓冲区在每个TCP套接字中单独存在；
2.I/O缓冲区在创建套接字时自动生成；
3.即使关闭套接字也会继续传送输出缓冲区中遗留的数据；
4.关闭套接字将丢失输入缓冲区中的数据。

输入输出缓冲区的默认大小一般都是 8K。
```

代码查看缓冲区的大小

```python
import socket
server = socket.socket()
server.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1)  # 重用ip地址和端口
server.bind(('127.0.0.1',8010))
server.listen(3)
print(server.getsockopt(socket.SOL_SOCKET,socket.SO_SNDBUF))  # 输出缓冲区大小
print(server.getsockopt(socket.SOL_SOCKET,socket.SO_RCVBUF))  # 输入缓冲区大小
#65536本机是64k
#65536本机是64k
```

**只有TCP有粘包现象，UDP永远不会粘包！**

```
发送端可以是一K一K地发送数据，而接收端的应用程序可以两K两K地提走数据，当然也有可能一次提走3K或6K数据，或者一次只提走几个字节的数据，也就是说，应用程序所看到的数据是一个整体，或说是一个流（stream），一条消息有多少字节对应用程序是不可见的，因此TCP协议是面向流的协议，这也是容易出现粘包问题的原因。而UDP是面向消息的协议，每个UDP段都是一条消息，应用程序必须以消息为单位提取数据，不能一次提取任意字节的数据，这一点和TCP是很不同的。怎样定义消息呢？可以认为对方一次性write/send的数据为一个消息，需要明白的是当对方send一条信息的时候，无论底层怎样分段分片，TCP协议层会把构成整条消息的数据段排序完成后才呈现在内核缓冲区。

例如基于tcp的套接字客户端往服务端上传文件，发送时文件内容是按照一段一段的字节流发送的，在接收方看了，根本不知道该文件的字节流从何处开始，在何处结束

所谓粘包问题主要还是因为接收方不知道消息之间的界限，不知道一次性提取多少字节的数据所造成的。

此外，发送方引起的粘包是由TCP协议本身造成的，TCP为提高传输效率，发送方往往要收集到足够多的数据后才发送一个TCP段。若连续几次需要send的数据都很少，通常TCP会根据优化算法把这些数据合成一个TCP段后一次发送出去，这样接收方就收到了粘包数据。

TCP（transport control protocol，传输控制协议）是面向连接的，面向流的，提供高可靠性服务。收发两端（客户端和服务器端）都要有一一成对的socket，因此，发送端为了将多个发往接收端的包，更有效的发到对方，使用了优化方法（Nagle算法），将多次间隔较小且数据量小的数据，合并成一个大的数据块，然后进行封包。这样，接收端，就难于分辨出来了，必须提供科学的拆包机制。 即面向流的通信是无消息保护边界的。
UDP（user datagram protocol，用户数据报协议）是无连接的，面向消息的，提供高效率服务。不会使用块的合并优化算法，, 由于UDP支持的是一对多的模式，所以接收端的skbuff(套接字缓冲区）采用了链式结构来记录每一个到达的UDP包，在每个UDP包中就有了消息头（消息来源地址，端口等信息），这样，对于接收端来说，就容易进行区分处理了。 即面向消息的通信是有消息保护边界的。
tcp是基于数据流的，于是收发的消息不能为空，这就需要在客户端和服务端都添加空消息的处理机制，防止程序卡住，而udp是基于数据报的，即便是你输入的是空内容（直接回车），那也不是空消息，udp协议会帮你封装上消息头，实验略
udp的recvfrom是阻塞的，一个recvfrom(x)必须对唯一一个sendinto(y),收完了x个字节的数据就算完成,若是y>x数据就丢失，这意味着udp根本不会粘包，但是会丢数据，不可靠

tcp的协议数据不会丢，没有收完包，下次接收，会继续上次继续接收，己端总是在收到ack时才会清除缓冲区内容。数据是可靠的，但是会粘包。
```

### 两种情况下会发生粘包

一、自己的理解:**接收方没有及时接收到缓冲区的包，造成多个包接收（客户端发送了一段数据，服务端只接收了一小部分），服务端下次再接收的时候还是从缓冲区拿上次遗留下的数据，产生粘包。**

二、方便记忆：**接收方不知道发来的数据的分界线，不知道一次提取多少字节的数据，从而造成粘包**

- sever端

  ```python
  import socket
  import subprocess
  
  bind_ip = '127.0.0.1'
  bind_port = 9999
  Addr = (bind_ip,bind_port)
  
  ss = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
  ss.bind(Addr)
  ss.listen(5)
  
  while 1:
  	conn,addr = ss.accept()
  	print (addr)
  	while 1:
  		try:
  			cmd = conn.recv(1024)
  			ret = subprocess.Popen(cmd.decode('utf-8'), shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
  			correct_msg = ret.stdout.read()
  			error_msg = ret.stderr.read()
  			conn.send(correct_msg+error_msg)
  		except Exception:
  			break
  	conn.close()
  ss.close()
  结果：
  ('127.0.0.1', 59175)
  388
  ('127.0.0.1', 59181)
  4158
  ```

- client端

  ```python
  import socket
  
  bind_ip = '127.0.0.1'
  bind_port = 9999
  Addr = (bind_ip,bind_port)
  
  cs = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
  cs.connect(Addr)
  
  while 1:
  	cmd = input(">>>")
  	cs.send(cmd.encode("utf-8"))
  	for_server_data = cs.recv(1024)
  	print (for_server_data.decode("gbk"))
  cs.close()
  结果：
  >>>dir
  388
  >>>help
  1024
  ```

一、自己的解释：**发送端需要等缓冲区满才发出去，造成粘包（发送数据时间间隔很短，数据量也很小，会合到一起，产生粘包）**

二、便于记忆：**连续多次send数据量较小的数据，这些数据会粘在一起。**

- sever端

  ```python
  import socket
  
  bind_ip = '127.0.0.1'
  bind_port = 9999
  Addr = (bind_ip,bind_port)
  
  ss = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
  ss.bind(Addr)
  ss.listen(5)
  
  conn,addr = ss.accept()
  frist_data = conn.recv(1024)
  print ("第一次:"+frist_data.decode('utf-8'))
  second_data = conn.recv(1024)
  print ("第二次:"+second_data.decode('utf-8'))
  
  conn.close()
  ss.close()
  结果：
  第一次:helloworld
  第二次:
  ```

- client端

  ```python
  import socket
  
  cs = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  
  cs.connect(('127.0.0.1', 9999))
  
  cs.send(b'hello')
  cs.send(b'world')
  
  cs.close()
  ```

### 粘包解决的方案

- 收发数据的本质：不一定是一收一发。

- 首先了解struct模块

  - 功能：把一个类型，如数字，转换成固定长度的bytes

    ![img](http://9017499461.linshutu.top/8.png)

  - 实例

    ```python
    import struct
    
    #将一个数字转换成等长度的bytes类型
    ret = struct.pack('i',123456)
    print (ret,type(ret),len(ret))
    #b'@\xe2\x01\x00' <class 'bytes'> 4
    
    #通过unpack帆姐回来
    ret1 = struct.unpack('i',ret)[0]
    print(ret1,type(ret1))
    #123456 <class 'int'>
    
    #注意：通过struct不能处理的数字太大,否则就会报错
    # ret2 = struct.pack('l',98433463666)
    # print (ret2)
    ```

  - 解决粘包方案一（low版）

    - 分析：我们发现问题的根源在于，接收端不知道发送端将要传送的字节流的长度，所以解决粘包的方法就是围绕，如何让发送端在发送数据前，把自己将要发送的字节流总长度按照固定字节发送给接收端后面跟上数据，然后接收端先接收固定字节的总字节流，再来一个死循环接收完所有的数据。

    - server

      ```python
      #__*__coding:utf-8__*__
      import struct
      import socket
      import subprocess
      
      bind_ip = '127.0.0.1'
      bind_port = 9999
      Addr = (bind_ip,bind_port)
      
      ss = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
      ss.bind(Addr)
      ss.listen(5)
      
      while 1:
      	conn,addr = ss.accept()
      	print (addr)
      	while 1:
      		try:
      			cmd = conn.recv(1024)
      			ret = subprocess.Popen(cmd.decode('utf-8'), shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
      			correct_msg = ret.stdout.read()
      			error_msg = ret.stderr.read()
      
      			#制作固定报头
      			totle_size = len(correct_msg) + len(error_msg)
      			header = struct.pack('i',totle_size)
      
      			#发送报头
      			conn.send(header)
      
      			#发送真实数据
      			conn.send(correct_msg)
      			conn.send(error_msg)
      			print (len(correct_msg+error_msg))
      		except Exception:
      			break
      	conn.close()
      ss.close()
      
      # 但是low版本有问题：
      # 1，报头不只有总数据大小，而是还应该有MD5数据，文件名等等一些数据。
      # 2，通过struct模块直接数据处理，不能处理太大。
      ```

    - client

      ```python
      #__*__coding:utf-8__*__
      import socket
      import struct
      
      bind_ip = '127.0.0.1'
      bind_port = 9999
      Addr = (bind_ip,bind_port)
      
      cs = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
      cs.connect(Addr)
      
      while 1:
      	cmd = input(">>>")
      	cs.send(cmd.encode("utf-8"))
      
      	#接收固定的报头
      	header = cs.recv(4)
      
      	#解析报头
      	total_size = struct.unpack('i',header)[0]
      
      	#根据消息头的信息，接收真是的数据
      	recv_size = 0
      	res = b''
      	while recv_size < total_size:
      		for_server_data = cs.recv(1024)
      		res += for_server_data
      		recv_size += len(for_server_data)
      	print (res.decode("gbk"))
      cs.close()
      ```

### 高大上版解决粘包问题

- 流程详解

  ```python
  整个流程的大致解释：
  我们可以把报头做成字典，字典里包含将要发送的真实数据的描述信息(大小啊之类的)，然后json序列化，然后用struck将序列化后的数据长度打包成4个字节。
  我们在网络上传输的所有数据 都叫做数据包，数据包里的所有数据都叫做报文，报文里面不止有你的数据，还有ip地址、mac地址、端口号等等，其实所有的报文都有报头，这个报头是协议规定的，看一下
  
  发送时：
  先发报头长度
  再编码报头内容然后发送
  最后发真实内容
  
  接收时：
  先手报头长度，用struct取出来
  根据取出的长度收取报头内容，然后解码，反序列化
  从反序列化的结果中取出待取数据的描述信息，然后去取真实的数据内容
  ```

- server

  ```python
  # FTP 应用层自定义协议
  '''
  1. 高大上版: 自定制报头
  dic = {'filename': XX, 'md5': 654654676576776, 'total_size': 26743}
  2. 高大上版:可以解决文件过大的问题.
  
  
  '''
  # import struct
  
  # ret = struct.pack('Q',21321432423544354365563543543543)
  # print(ret)
  #执行包括，因为struct的范围有限
  
  import socket
  import subprocess
  import struct
  import json
  phone = socket.socket()
  
  ss.bind(('127.0.0.1',8848))
  
  ss.listen(2)
  # listen: 2 允许有两个客户端加到半链接池，超过两个则会报错
  
  while 1:
      conn,addr = ss.accept()  # 等待客户端链接我,阻塞状态中
      # print(f'链接来了: {conn,addr}')
  
      while 1:
          try:
  
              from_client_data = conn.recv(1024)  # 接收命令
  
  
              if from_client_data.upper() == b'Q':
                  print('客户端正常退出聊天了')
                  break
  
              obj = subprocess.Popen(from_client_data.decode('utf-8'),
                                     shell=True,
                                     stdout=subprocess.PIPE,
                                     stderr=subprocess.PIPE,
  
                                     )
              result = obj.stdout.read() + obj.stderr.read()
              total_size = len(result)
  
              # 1. 自定义报头
              head_dic = {
                  'file_name': 'test1',
                  'md5': 6567657678678,
                  'total_size': total_size,
  
              }
              # 2. json形式的报头
              head_dic_json = json.dumps(head_dic)
  
              # 3. bytes形式报头
              head_dic_json_bytes = head_dic_json.encode('utf-8')
  
              # 4. 获取bytes形式的报头的总字节数
              len_head_dic_json_bytes = len(head_dic_json_bytes)
  
              # 5. 将不固定的int总字节数变成固定长度的4个字节
              four_head_bytes = struct.pack('i',len_head_dic_json_bytes)
  
              # 6. 发送固定的4个字节
              conn.send(four_head_bytes)
  
              # 7. 发送报头数据
              conn.send(head_dic_json_bytes)
  
              # 8. 发送总数据
              conn.send(result)
  
          except ConnectionResetError:
              print('客户端链接中断了')
              break
      conn.close()
  ss.close()
  ```

- client

  ```python
  import socket
  import struct
  import json
  cs = socket.socket()
  
  cs.connect(('127.0.0.1',8848))
  while 1:
      to_server_data = input('>>>输入q或者Q退出').strip().encode('utf-8')
      if not to_server_data:
          # 服务端如果接受到了空的内容，服务端就会一直阻塞中，所以无论哪一端发送内容时，都不能为空发送
          print('发送内容不能为空')
          continue
      cs.send(to_server_data)
      if to_server_data.upper() == b'Q':
          break
  
      # 1. 接收固定长度的4个字节
      head_bytes = cs.recv(4)
  
      # 2. 获得bytes类型字典的总字节数
      len_head_dic_json_bytes = struct.unpack('i',head_bytes)[0]
  
      # 3. 接收bytes形式的dic数据
      head_dic_json_bytes = phone.recv(len_head_dic_json_bytes)
  
      # 4. 转化成json类型dic
      head_dic_json = head_dic_json_bytes.decode('utf-8')
  
      # 5. 转化成字典形式的报头
      head_dic = json.loads(head_dic_json)
      '''
      head_dic = {
                  'file_name': 'test1',
                  'md5': 6567657678678,
                  'total_size': total_size,
  
              }
      '''
      total_data = b''
      while len(total_data) < head_dic['total_size']:
          total_data += cs.recv(1024)
  
      # print(len(total_data))
      print(total_data.decode('gbk'))
  
  cs.close()
  ```

### 基于UDP协议的socket通信

**UDP的通信例图**

![](http://9017499461.linshutu.top/UDP%E7%9A%84socket%E7%BC%96%E7%A8%8B.png)

- server

  ```python
  import socket
  ip_port=('127.0.0.1',8081)
  udp_server_sock=socket.socket(socket.AF_INET,socket.SOCK_DGRAM) #DGRAM:datagram 数据报文的意思，象征着UDP协议的通信方式
  udp_server_sock.bind(ip_port)#你对外提供服务的端口就是这一个，所有的客户端都是通过这个端口和你进行通信的
  
  while True:
      qq_msg,addr=udp_server_sock.recvfrom(1024)# 阻塞状态，等待接收消息
      print('来自[%s:%s]的一条消息:\033[1;44m%s\033[0m' %(addr[0],addr[1],qq_msg.decode('utf-8')))
      back_msg=input('回复消息: ').strip()
  
      udp_server_sock.sendto(back_msg.encode('utf-8'),addr)
  ```

- client

  ```python
  import socket
  BUFSIZE=1024
  udp_client_socket=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
  
  qq_name_dic={
      'taibai':('127.0.0.1',8081),
      'Jedan':('127.0.0.1',8081),
      'Jack':('127.0.0.1',8081),
      'John':('127.0.0.1',8081),
  }
  
  
  while True:
      qq_name=input('请选择聊天对象: ').strip()
      while True:
          msg=input('请输入消息,回车发送,输入q结束和他的聊天: ').strip()
          if msg == 'q':break
          if not msg or not qq_name or qq_name not in qq_name_dic:continue
          udp_client_socket.sendto(msg.encode('utf-8'),qq_name_dic[qq_name])# 必须带着自己的地址，这就是UDP不一样的地方，不需要建立连接，但是要带着自己的地址给服务端，否则服务端无法判断是谁给我发的消息，并且不知道该把消息回复到什么地方，因为我们之间没有建立连接通道
  
          back_msg,addr=udp_client_socket.recvfrom(BUFSIZE)# 同样也是阻塞状态，等待接收消息
          print('来自[%s:%s]的一条消息:\033[1;44m%s\033[0m' %(addr[0],addr[1],back_msg.decode('utf-8')))
  
  udp_client_socket.close()
  ```

  **实例**

  - 需求：接下来，给大家说一个真实的例子，也就是实际当中应用的，那么这是个什么例子呢？就是我们电脑系统上的时间，windows系统的时间是和微软的时间服务器上的时间同步的，而mac本是和苹果服务商的时间服务器同步的，这是怎么做的呢，首先他们的时间服务器上的时间是和国家同步的，你们用我的系统，那么你们的时间只要和我时间服务器上的时间同步就行了，对吧，我时间服务器是不是提供服务的啊，相当于一个服务端，我们的电脑就相当于客户端，就是通过UDP来搞的。

  - server

    ```python
    from socket import *
    from time import strftime
    import time
    ip_port = ('127.0.0.1', 9000)
    bufsize = 1024
    
    tcp_server = socket(AF_INET, SOCK_DGRAM)
    tcp_server.setsockopt(SOL_SOCKET,SO_REUSEADDR,1)
    tcp_server.bind(ip_port)
    
    while True:
        msg, addr = tcp_server.recvfrom(bufsize)
        print('===>', msg)
        stru_time = time.localtime()  #当前的结构化时间
        if not msg:
            time_fmt = '%Y-%m-%d %X'
        else:
            time_fmt = msg.decode('utf-8')
        back_msg = strftime(time_fmt,stru_time)
        print(back_msg,type(back_msg))
        tcp_server.sendto(back_msg.encode('utf-8'), addr)
    
    tcp_server.close()
    
    server端
    ```

  - client

    ```python
    from socket import *
    ip_port=('127.0.0.1',9000)
    bufsize=1024
    
    tcp_client=socket(AF_INET,SOCK_DGRAM)
    
    while True:
        msg=input('请输入时间格式(例%Y %m %d)>>: ').strip()
        tcp_client.sendto(msg.encode('utf-8'),ip_port)
    
        data=tcp_client.recv(bufsize)
        print('当前日期：',str(data,encoding='utf-8'))
    ```