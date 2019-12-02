---
title: 第二讲——cmdb资产管理之完整单例、日志、数据库连接池
id: 2
date: 2019-11-14 20:00:00
tags: cmdb
comment: true
---

## 今日概要

- 单例模式
- 资产采集补充
  - 参考命令
  - 日志处理（单利模式）
  - 支持agent模式（简单工厂模式）
- 入库
  - 表结构设计
  - api写入数据库

<!----more---->

## 今日详细

### 1. 单例模式

#### 1.2 “多例模式”

```python
class Foo(object):
	def __init__(self,name,age):
		self.name = name
		self.age = age
	
	def func(self):
		msg = "%s-%s" %(self.name,self.age)
		
		
obj1 = Foo("pl",18) # Foo的一个对象/Foo类的一个实例
obj1.func()

obj2 = Foo("pl1",19)
obj2.func()
```

#### 1.3 单例模式

```python
class Foo(object):
	instance = None
	def __init__(self,name,age):
		self.name = name
		self.age = age
	
	def __new__(cls,*arg,**kawrgs):
		if not cls.instance:
			cls.instance = object.__new__(cls)
		return cls.instance
		
```

```python
class Singleton(object):
    instance = None

    def __init__(self):
        self.name = None

    def __new__(cls, *arg, **kawrgs):
        if not cls.instance:
            cls.instance = object.__new__(cls)
        return cls.instance
```



应用场景：

- django配置文件，只要加载一次，以后使用都用同一份值。

  ```python
  class Singleton(object):
      instance = None
  
      def __init__(self):
          self.k0 = 0
          self.k1 = 1
          self.k2 = 2
          ...
  
      def __new__(cls, *arg, **kawrgs):
          if not cls.instance:
              cls.instance = object.__new__(cls)
          return cls.instance
      
  obj1 = Singleton()
  obj2 = Singleton()
  obj3 = Singleton()
  ```

- django的admin，在注册models中用，希望所有的model类注册到同一个列表中。

  

##### 1.3.1 单例模式是错误

```python
class Singleton(object):
    instance = None

    def __init__(self):
        self.name = None

    def __new__(cls, *arg, **kawrgs):
        if not cls.instance:
            cls.instance = object.__new__(cls)
        return cls.instance
```

##### 1.3.1 单利模式 new（正确）

```python
import time
import threading

class Singleton(object):
    instance = None
    lock = threading.RLock()
    #摒弃了__init__
    
    def __new__(cls, *arg, **kwargs):
        if cls.instance:
            return cls.instance
        #加了线程锁，保证在多线程下出现不会因为出现阻塞而产生多个实例
        with cls.lock:
            if not cls.instance:
                cls.instance = object.__new__(cls)
            return cls.instance
        
obj1 = Singleton()
obj2 = Singleton()
```

##### 1.3.2 文件导入（正确），在源码中的应用

```python
# xx.py

class Site(object):
    def __init__(self):
        self.names = []

    def xx(self):
        pass
site = Site()
```

```python
import xx

print(xx.site)
```

- 实例

  ```python
  # xx.py 
  
  class Singleton(object):
  
      def __init__(self):
          self._registry = []
  
      def register(self,model_class):
          self._registry.append(model_class)
  
  site = Singleton()
  ```

  ```python
  import xx
  xx.site
  ```

#### 总结

- 基于 `__new__`实现单例模式（***）

  ```python
  import time
  import threading
  
  class Singleton(object):
      instance = None
      lock = threading.RLock()
      
      def __new__(cls, *arg, **kwargs):
          if cls.instance:
              #当有程序阻塞的时候，就算1秒，我们的线程锁就会阻塞掉，这个时候当有10000个实例，不可能让程序等10000秒吧，所以在这里判断一下，如果存在实例就去返回存在的实例
              return cls.instance
          with cls.lock:
              if not cls.instance:
                  cls.instance = object.__new__(cls)
              return cls.instance
          
  obj = Singleton()
  print(obj)
  ```

- 基于文件导入实现单利模式

  ```python
  # sg.py（这种情况还是很常见的，我们的配置文件就是这样导入的）
  class Singleton(object):
      pass
  
  obj = Singleton()
  ```

  ```python
  import sg
  print(sg.obj)
  ```

  

- 应用场景：

  - django中settings配置文件
  - django的admin内部使用，将所有model类注册到了一个字典中。

#### 疑问

基于`__new__`实现单利模式，在init中不设置值。

- 是单例，但数据会被覆盖

  ```python
  class Singleton(object):
      instance = None
  
      def __init__(self):
          self.registry = []
  
      def __new__(cls, *arg, **kawrgs):
          if not cls.instance:
              cls.instance = object.__new__(cls)
          return cls.instance
  
      def register(self, val):
          self.registry.append(val)
  
  # {registry：[]}
  obj1 = Singleton()
  # {registry：[x1]}
  obj1.register('x1')
  
  # instance = {registry：[x1]}
  # instance = {registry：[]}
  obj2 = Singleton()
  # instance = {registry：[x2]}
  obj2.register('x2')
  
  print(obj2.registry)
  print(obj1.registry)
  ```

- 单例模式

  ```python
  class Singleton(object):
      instance = None
      registry = []
  
      def __new__(cls, *arg, **kawrgs):
          if not cls.instance:
              cls.instance = object.__new__(cls)
          return cls.instance
  
      def register(self, val):
          self.registry.append(val)
  
  
  obj1 = Singleton()
  obj1.register('x1')
  
  obj2 = Singleton()
  obj2.register('x2')
  
  print(obj2.registry)
  print(obj1.registry)
  ```

#### 面试题

1.手写单利模式（new+锁）？

2.其他单例模式（五种）？

```python
1、new
#实现__new__方法
#并在将一个类的实例绑定到类变量_instance上,  
#如果cls._instance为None说明该类还没有实例化过,实例化该类,并返回  
#如果cls._instance不为None,直接返回cls._instance  
class Singleton(object):  
    def __new__(cls, *args, **kw):  
        if not hasattr(cls, '_instance'):  
            orig = super(Singleton, cls)  
            cls._instance = orig.__new__(cls, *args, **kw)  
        return cls._instance  
  
class MyClass(Singleton):  
    a = 1  
  
one = MyClass()  
two = MyClass()  

# 以下方法测试类比方法一
two.a = 3  
print one.a  
#3  
#one和two完全相同,可以用id(), ==, is检测  
print id(one)  
#29097904  
print id(two)  
#29097904  
print one == two  
#True  
print one is two  
#True 

2、共享属性
#所谓单例就是所有引用(实例、对象)拥有相同的状态(属性)和行为(方法)  
#同一个类的所有实例天然拥有相同的行为(方法),  
#只需要保证同一个类的所有实例具有相同的状态(属性)即可  
#所有实例共享属性的最简单最直接的方法就是__dict__属性指向(引用)同一个字典(dict)  
#可参看:http://code.activestate.com/recipes/66531/  
class Borg(object):  
    _state = {}  
    def __new__(cls, *args, **kw):  
        ob = super(Borg, cls).__new__(cls, *args, **kw)  
        ob.__dict__ = cls._state  
        return ob  
  
class MyClass2(Borg):  
    a = 1  
  
one = MyClass2()  
two = MyClass2()  
  
#one和two是两个不同的对象,id, ==, is对比结果可看出  

3、装饰器
#也是方法1的升级（高级）版本,  
#使用装饰器(decorator),  
#这是一种更pythonic,更elegant的方法,  
#单例类本身根本不知道自己是单例的,因为他本身(自己的代码)并不是单例的  
def singleton(cls, *args, **kw):  
    instances = {}  
    def _singleton():  
        if cls not in instances:  
            instances[cls] = cls(*args, **kw)  
        return instances[cls]  
    return _singleton  
 
@singleton  
class MyClass4(object):  
    a = 1  
    def __init__(self, x=0):  
        self.x = x  
  
one = MyClass4()  
two = MyClass4() 

4、import方法
# mysingleton .py
class MyClass(object):
    def __init__(self):
        self.a = 1

s_myclass = MyClass()

# to use
from mysingleton import s_myclass
s_myclass.a

5.metaclass（元类 ）
#本质上是方法1的升级（或者说高级）版  
#使用__metaclass__（元类）的高级python用法  
class Singleton2(type):  
    def __init__(cls, name, bases, dict):  
        super(Singleton2, cls).__init__(name, bases, dict)  
        cls._instance = None  
    def __call__(cls, *args, **kw):  
        if cls._instance is None:  
            cls._instance = super(Singleton2, cls).__call__(*args, **kw)  
        return cls._instance  
  
class MyClass3(object):  
    __metaclass__ = Singleton2  
  
one = MyClass3()  
two = MyClass3()  
```

3.`__new__`方法返回的是什么？ 新创建的对象，内部没有数据，需要经过init来进行初始化。

4.单利模式应用场景：

- django的配置文件
- django的admin
- 数据库链接池（单例模式，补充）
- 写日志

### 2. 获取日志堆栈信息

```python
import traceback

def run():
    try:
        int('asdf')
    except Exception as e:
        #traceback.format_exc()就是获取堆栈的信息，当程序出现错误的时候，我们可以读取详细的信息
        print(traceback.format_exc())
        
if __name__ == '__main__':
    run()
```

### 3. 获取资产的命令

- 内存信息

  ```shell
  sudo dmidecode  -q -t 17 2>/dev/null
  
  注意：linux上要提前安装 yum install dmidecode
  ```

- 硬盘（安装MegaCli）

  ```shell
  sudo MegaCli  -PDList -aALL
  ```

- 网卡

  ```shell
  sudo ip link show
  sudo ip addr show
  ```

- 主板

  ```shell
  sudo dmidecode -t1
  ```

- CPU

  ```shell
  cat /proc/cpuinfo
  ```


### 4.数据封装

```python
class BaseResponse(object):
    def __init__(self):
        self.status = True
        self.data = None
        self.error = None
    @property
    def dict(self):
        #__dict__封装的是字典数据
        return self.__dict__

def process():
    info = BaseResponse()
    try:
        info.status = True
        info.data = "nice"
    except Exception:
        pass
    return info.dict #{'status':True,'data':'nice','error':None}

result = process()
print(result)
```

