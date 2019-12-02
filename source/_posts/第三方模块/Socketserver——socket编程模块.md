---
title: socketserver——socket编程模块
id: 11
date: 2019-8-28 20:30:00
tags: 第三方模块
comments: true
---

**介绍**：这是标准库中的一个高级模块，它的目标就是简化多样性样板代码。

**用途**：我们之前讲的tcp协议的socket一次只能和一个客户端通信，如果用socketserver可以实现和多个客户端通信。它是在socket的基础上进行的一层封装，也就是说底层还是调用的socket，并发实现和多个客户端进行通信。

Socketserver的5个服务类：

- BaseServer：基类，所有服务器对象的超类，定义了接口。
- TCPServer
- UDPServer
- UnixStreamServer
- UnixDatagramServer

**实例代码**

```python
server端
import socketserver

class MyServer(socketserver.BaseRequestHandler):
	#重写父类方法，必须是这个名字
    def handle(self):
        """
        我们拿到了每个客服端的管道，那么我们自己在这个方法里面就写我们接收消息发送消息的逻辑就可以了。
        """
        #self.request相当于一个conn
		self.request.recv(1024)
		msg = input(">>>")
		self.request.send(bytes(msg,encoding='utf-8'))
        #关闭连接
		self.request.close()
if __name__ == '__main__':
    #使用socketserver的ThreadingTCPServer这个类，将IP和端口的元组传进去，还需要将我们上面定义的类传进去，实例化得到server这个对象的时候初始化__init__自动执行里面的代码，里面把所有的事情都干了，相当于我们通过它进行了bind，listen。
	server = socketserver.ThreadingTCPServer(('127.0.0.1',8888))
    #使用上面的这个类的对象来执行serve_forever方法，它的作用就是我们的服务一直开启，serve_forever帮我们进行accept
	server.serve_forever()
```

```python
上述的socketservr内部做的操作解读：
在整个socketserver这个模块中，其实就干了两件事情：
1、一个是循环建立链接的部分，每个客户链接都可以连接成功  
2、一个通讯循环的部分，就是每个客户端链接成功之后，要循环的和客户端进行通信。
看代码中的：server=socketserver.ThreadingTCPServer(('127.0.0.1',8090),MyServer)

还记得面向对象的继承吗？来，大家自己尝试着看看源码：

查找属性的顺序:ThreadingTCPServer
			->ThreadingMixIn
    		->TCPServer
			->BaseServer

实例化得到server，先找ThreadMinxIn中的__init__方法，发现没有init方法，然后找类ThreadingTCPServer的__init__,在TCPServer中找到，在里面创建了socket对象，进而执行server_bind（相当于bind）,server_active（点进去看执行了listen）

找server下的serve_forever,在BaseServer中找到，进而执行self._handle_request_noblock()，该方法同样是在BaseServer中
执行self._handle_request_noblock()进而执行request, client_address = self.get_request()（就是TCPServer中的self.socket.accept()），然后执行self.process_request(request, client_address)
在ThreadingMixIn中找到process_request，开启多线程应对并发，进而执行process_request_thread，执行self.finish_request(request, client_address)

上述四部分完成了链接循环，本部分开始进入处理通讯部分，在BaseServer中找到finish_request,触发我们自己定义的类的实例化，去找__init__方法，而我们自己定义的类没有该方法，则去它的父类也就是BaseRequestHandler中找....
源码分析总结：

基于tcp的socketserver我们自己定义的类中的

　　self.server即套接字对象
　　self.request即一个链接
　　self.client_address即客户端地址
基于udp的socketserver我们自己定义的类中的

　　self.request是一个元组（第一个元素是客户端发来的数据，第二部分是服务端的udp套接字对象），如(b'adsf', <socket.socket fd=200, family=AddressFamily.AF_INET, type=SocketKind.SOCK_DGRAM, proto=0, laddr=('127.0.0.1', 8080)>)
　　self.client_address即客户端地址
```

**Server Objects**：

BaseServer.fileno:返回整数表示那个服务器正在监听，常用。

select.select()：表示允许监听多个相同处理过程的服务。

BaseServer.handle_request:处理单一请求。

BaseServer.server_forever:处理请求直到明确shutdown()请求。

BaseServer.RequestHandlerClass:用户请求处理程序类，为每一个请求创建这个类的实例。

**Twisted框架**

介绍：Twisted是一个完整的事件驱动的网络框架，利用它既能使用也可以开发完整的异步网络应用程序和协议。它提供了大量的支持建立完整的系统，包括网络协议，线程，安全性和身份验证，聊天，DBM，web，电子邮件，命令行等。如果我们有这方面的需求的时候可以详细的了解：

[**http://gashero.yeax.com/?cat=7**](http://gashero.yeax.com/?cat=7)

[https://blog.csdn.net/qq_24861509/article/details/45192067](https://blog.csdn.net/qq_24861509/article/details/45192067)

