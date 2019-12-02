---
title: 第一讲——cmdb资产管理之项目框架
id: 1
date: 2019-11-13 20:00:00
tags: cmdb
comment: true
---

### 1.项目概述

- 背景

  - 大目标：运维自动化
  - 小目标：资产管理（以前用的excel）

  ```
  大公司：物理机1200 > 6000
  中：物理机70台
  小：阿里云（虚拟机）
  ```

  系统开发出来时候，可以自动化采集资产信息，并且给其他系统提供数据支持。

- 运维：
  - 业务
  - IDC
  - 桌面
  - 监控
- 物理机和虚拟机
  - 物理机：托管在兆维、世纪互联机房。
  - 虚拟机（云服务器）：阿里云、腾讯云、aws

<!----more---->

### 2.项目架构

![](http://9017499461.linshutu.top/cmdb1.png)

项目有三部分组成：

- 资产采集，用于远程连接服务器并获取服务器的资产信息，然后将资产信息汇报API。
- API，负责将资产信息写入数据库，并且要做资产变更记录，为了以后搭建运维自动化平台，可以为其他系统提供restful接口做数据支持。
- 资产管控平台，为用户提供数据展示和报表。

采集资产的方式：

- 基于SSH（paramiko），70台物理机
  ![](http://9017499461.linshutu.top/cmdb2.JPG)

- ansible管理工具（saltstack、puppet）

  ![](http://9017499461.linshutu.top/cmdb3.JPG)

- 基于agent（客户端），1200台物理机
  ![](http://9017499461.linshutu.top/cmdb4.JPG)



### 3.项目实现（资产采集、API）

项目目录：

![](http://9017499461.linshutu.top/cmdb5.JPG)

项目的流程：app是程序的入口

```python
# __*__coding:utf-8__*__
import paramiko
import requests
from concurrent.futures import ThreadPoolExecutor

import settings
from lib.plugins import get_server_info


def ssh(hostname, command):
	"""
	利用公私钥从服务器采集信息的脚本
	:param hostname: 主机的ip
	:param command: 执行的命令
	:return: 
	"""
	private_key = paramiko.RSAKey.from_private_key_file(settings.SSH_PRIVATE_KEY_PATH)
	ssh = paramiko.SSHClient()
	ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
	ssh.connect(hostname=hostname, port=settings.SSH_PORT, username=settings.SSH_USER, pkey=private_key)
	stdin, stdout, stderr = ssh.exec_command(command)
	result = stdout.read()
	ssh.close()
	return result.decode('utf-8')



def task(hostname):
	"""
	一个采用线程池的任务
	:param hostname: 
	:return: 
	"""
	info = get_server_info(hostname, ssh)
	requests.post(
		url="http://127.0.0.1:8000/api/v1/server/",
		json=info
	)


def run():
	"""
	真正的主函数
	:return: 
	"""
	response = requests.get(url="http://127.0.0.1:8000/api/v1/server/")
	host_list = response.json()
	pool = ThreadPoolExecutor(settings.THREAD_POOL_SIZE)
	for host in host_list:
		pool.submit(task, host)


if __name__ == "__main__":
	run()

```

从上面run()进入主函数，去api接口请求得到需求采集信息的主机的ip。

并考虑多服务器采集的时候需要并发的时候需要使用线程池来提高脚本的的效率。里面的静态参数都是放在了settings配置文件里面，显示开放封闭原则。

然后，把task任务发布到线程池，并把我们从api请求到的主机列表传递给task任务。

我们进入task函数，进而执行了get_server_info()函数，并把主机ip和ssh（和服务器交互执行脚本并返回结果），这个函数是我们从from lib.plugins import get_server_info导入的，纳闷这个函数在哪里呢？它其实是在plugins下的`__init__`文件里面，导入包时，自动加载 `__init__.py`。

```python
from settings import PLUGIN_DICT

def get_server_info(hostname, ssh_func):
	"""

	:param hostname: 远程操作的主机名
	:param ssh_fuc: 执行远程操作的方法
	:return:
	"""
	info_dict = {}

	for key, path in PLUGIN_DICT.items():
		# 从右边的第一个.切割字符串
		module_path, class_name = path.rsplit('.', maxsplit=1)

		# 根据字符串的形式去导入模块，"lib.plugins.board"
		import importlib
		module = importlib.import_module(module_path)

		# 利用反射的方式去到对应模块中找类名
		cls = getattr(module, class_name)

		# 找到类之后，实例化该对象
		obj = cls()

		# 执行实例中的process方法，把ssh_func传进去
		result = obj.process(hostname, ssh_func)

		info_dict[key] = result

	return info_dict
```

```python
PLUGIN_DICT = {
	'board':'lib.plugins.board.Board',
	'disk':'lib.plugins.disk.Disk',
	'nic':'lib.plugins.nic.Nic',
	'cpu':'lib.plugins.cpu.Cpu',
	'memory':'lib.plugins.memory.Memory',
}
```

我们可以从上面的代码中看出，settins里面的PLUGIN_DICT储存的是我们每一个插件（采集服务器每一个部分信息的模块）的路径，那么，这个怎么使用呢？这里就涉及到了一个新的知识点，这里使用rspilt把这个路径进行了切割，前面一部分是我们插件的文件路径，后面的一部分是插件文件里面的类。然后使用importlib.import_module(module_path)找到模块的路径，并利用反射的原理去该模块里面找前面切割的字符串（类名），然后实例化，并执行该模块里面的process()方法

```python
from .base import Base

class Disk(Base):

	def process(self,hostname,ssh_func):
		"""

		:return:
		"""
		result = ssh_func(hostname,'df')
		return result
```

在执行该方法接收的一个是主机的ip，另一个就是我们前面说的ssh函数，是它和我们的服务器之间进行交互的

```python
def ssh(hostname, command):
	"""
	利用公私钥从服务器采集信息的脚本
	:param hostname: 主机的ip
	:param command: 执行的命令
	:return: 
	"""
	private_key = paramiko.RSAKey.from_private_key_file(settings.SSH_PRIVATE_KEY_PATH)
	ssh = paramiko.SSHClient()
	ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
	ssh.connect(hostname=hostname, port=settings.SSH_PORT, username=settings.SSH_USER, pkey=private_key)
	stdin, stdout, stderr = ssh.exec_command(command)
	result = stdout.read()
	ssh.close()
	return result.decode('utf-8')
```

然后，这个函数接收了两个参数，一个是执行主机的ip，另外一个是我们输入的命令，然后，采得数据并返回给process，它又返回给get_server_info，它又返回给task，进而在task里面使用requests.post把我们采集的数据传递给了api接口。

其中，我们又一个base文件，它的功能就是一个约束的类

```python
class Base(object):

	def process(self,hostname,ssh_func):

		raise NotImplementedError(f"子类 {self.__class__.__name__} 必须实现process方法")
```

我们的模块都需要去继承该Base类，约束每一个子类实现process方法。

### 4.相关知识点

- 约束，面向对象对于子类的约束。

- 反射，根据字符串的形式去对象中取值（根据字符串的形式导入模块）

- 结合反射和配置文件，开发封闭原则。（设计模式：工厂模式）

  ```
  开放：配置文件开放。
  封闭：对源码修改封闭。
  ```

- 我们插件是可扩展。

## 总结

1. 为什么要开发CMDB？

   ```
   公司以后想要搭建自动化运维平台，CMDB是搭建平台的基石。
   目前而言，公司资产信息不够准确，因为都维护在excel中，维护主要人，通过cmdb可以自动采集资产信息以及做资产变更记录。
   ```

2. 你们公司有多少台服务器？物理机？虚拟机？

   ```
   100台左右物理机。
   ```

3. 你的CMDB是如何实现的？

   ```
   cmdb是由三部分组成，其中包含：资产采集的中控机、API、资产管控平台。
   
   -对于资产采集部分，通过paramiko远程操作服务器（本质SSH）并采集资产信息，然后将资产信息汇报到API，在资产采集部分还继承了可扩展的功能，让我们定制插件时可以更加方便，实现起来也比较简答，参考django中间件的原理、开发封闭原则、工厂模式实现可插拔式的插件。 
   
   -api，基于restful规范和drf组件来实现完成，主要做资产入库以及资产变更处理。
   
   -资产管控平台，对资产数据进行数据呈现和报表的处理。
   ```

4. 你的程序有什么Bug？难以忘记的经历？

   ```
   对于bug没有太多印象，主要在资产采集可扩展性方面,开始写的时候没考虑太多，只管实现功能；后来随着需求的增加，发现这种方式就不是很适合，因此，我就开始考虑程序的扩展性，经过一些努力并实现了这种功能。
   ```

5. 技术点

   - 类的约束

   - 通过字符串的形式导入一个模块

     ```python
     import importlib
     module = importlib.import_module("xxx.xx.xx.csss")
     ```

   - 反射，通过字符串形式去操作对象中属性

     ```
     getattr
     setattr
     delattr
     hasattr
     ```

   - 线程池的应用

   - 导入包时，自动加载 `__init__.py`

   - requests模块的应用

     ```python
     requests.get(url='...')
     
     requests.post(url="...",data={})
     
     requests.post(url="...",json={})
     ```

   - 项目中容易被修改的值，可以放在配置文件中。

   - 开放封闭原则

     ```
     配置开放
     源码封闭
     ```

   - 工厂模式：简单工厂

     ```
     mode = "email"
     ```

     ```
     class Email(object):
     	def send(self):
     		pass
     
     class Wechat(object):
     	def send(self):
     		pass
     		
     class Message(object):
     	def send(self):
     		pass
     ```

     ```
     def run():
     	instance = None 
     	if mode == 'email':
     		instance = Email()
     	elif mode == 'wechat':
     		instance = Wechat()
     	elif mode == 'message':
     		insance = Message()
     		
     	if not instance:
     		print('配置异常')
     	instance.send()
     	
     run()
     ```

   - 工厂模式：工厂方法/抽象工厂

     ```
     mode = "xxx.xx.Email"
     ```

     ```python
     class Email(object):
     	def send(self):
     		pass
     
     class Wechat(object):
     	def send(self):
     		pass
     		
     class Message(object):
     	def send(self):
     		pass
     ```

     ```python
     def run():
     	根据反射到类，并实例化。
     	instance.send()
     run()
     ```





































































































































、



