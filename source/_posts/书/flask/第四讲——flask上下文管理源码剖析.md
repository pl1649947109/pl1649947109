---
title: 第四讲——flask上下文管理源码剖析
id: 5
date: 2019-11-24 20:00:00
tags: flask
toc: true

comment: true
---

## 内容回顾

1.django和flask的区别

```
- 概括的区别
- django中提供功能列举
- 请求处理机制不同，django是通过传参的形式，flask是通过上下文管理的方式实现。 
```

2.wsgi

```
django和flask内部都没有实现socket，而是wsgi实现。
wsgi是web服务网管接口，他是一个协议，实现它的协议的有：wsgiref/werkzurg/uwsgi
```

```python
# django之前
from wsgiref.simple_server import make_server
 
def run(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return [bytes('<h1>Hello, web!</h1>', encoding='utf-8'), ]
 
if __name__ == '__main__':
    httpd = make_server('127.0.0.1', 8000, run)
    httpd.serve_forever()
```

```python
# flask之前
from werkzeug.serving import run_simple
from werkzeug.wrappers import BaseResponse


def func(environ, start_response):
    print('请求来了')
    response = BaseResponse('你好')
    return response(environ, start_response)


if __name__ == '__main__':
    run_simple('127.0.0.1', 5000, func)
```

<!----more---->

3.web框架都有的功能：路由、视图、模板

4.before_request/after_request

```
相当于django的中间件，对所有的请求定制功能。 
```

5.tempalte_global / template_filter

```
定制在所有模板中都可以使用函数
```

6.路由系统处理本质 @app.route

```
将url和函数打包成rule，添加到map对象，map再放到app中。
```

7.路由

- 装饰器实现 / add_url_rule
- endpoint
- 动态路由
- 如果给视图加装饰器：放route下面 、 functools

8.视图

- FBV
- CBV（返回一个view函数，闭包的应用场景）
- 应用到的功能都是通过导入方式：request/session

9.flask中支持session

```
默认将session加密，然后保存在浏览器的cookie中。 
```

10.模板比django方便一点

```
支持python原生的语法
```

11.蓝图

```
帮助我们可以对很多的业务功能做拆分，创建多个py文件，把各个功能放置到各个蓝图，最后再将蓝图注册到flask对象中。 

帮助我们做目录结构的拆分。
```

12.threading.local对象

```
自动为每个线程开辟空间，让你进行存取值。
```

13.数据库链接池 DBUtils （SQLHelper）

14.面向对象上下文管理（with)

## 概要

- flask上下文源码
- flask的扩展

## 内容详细

### 1. 栈

后进先出，通过列表可以实现一个栈。

```python
v = [11,22,33]
v.append(44)
v.pop()
```

应用场景：

- 节流

### 2. 面向对象

```python
class Foo(object):

    def __setattr__(self, key, value):
        print(key,value)
    
    def __getattr__(self, item):
        print(item)

obj = Foo()  #触发__setattr__
obj.x = 123 
obj.x    #触发__getattr__
```

- drf中request


```python
request.data
request.query_params
request._request
request._request.POST
request._request.GET
```

```python
class Local(object):
    def __init__(self):
        # self.storage = {}
        object.__setattr__(self,"storage",{})

    def __setattr__(self, key, value):
        self.storage[key] = value

    def __getattr__(self, item):    #一直在这循环
        return self.storage.get(item)

local = Local()
local.x1 = 123
print(local.x1)
```

### 3.线程唯一标识

```python
import threading
from threading import get_ident

def task():
    ident = get_ident()  #获取线程ID
    print(ident)
for i in range(20):
    t = threading.Thread(target=task)
    t.start()
```

### 4.自定义threading.local

```python
import threading
"""
storage = {
    1111:{'x1':[0,1,2,3]},  #每一个线程，里面键是固定名称，值是一个列表（这就是那个神奇的地方）
    1112:{'x1':1}
    1113:{'x1':2}
    1114:{'x1':3}
    1115:{'x1':4}
}
"""
class Local(object):
    def __init__(self):
        #程序就来触发它的执行
        object.__setattr__(self,'storage',{})

    def __setattr__(self, key, value):
        #ident是线程ID
        ident = threading.get_ident()
        #判断线程该线程是否在storage字典里面，有就把传来的值存里面，没有就新建一条数据
        if ident in self.storage:
            self.storage[ident][key] = value
        else:
            self.storage[ident] = {key:value}

    def __getattr__(self, item):
        #取对象值得时候触发，获取线程ID
        ident = threading.get_ident()
        #有就返回值没有就返回None
        if ident not in self.storage:
            return
        return self.storage[ident].get(item)

local = Local()

def task(arg):
    local.x1 = arg
    print(local.x1)

for i in range(5):
    t = threading.Thread(target=task,args=(i,))
    t.start()
```

### 5.加强版threading.local

```python
import threading
"""
storage = {
    1111:{'x1':[]},
    1112:{'x1':[]}
    1113:{'x1':[]}
    1114:{'x1':[]}
    1115:{'x1':[]},
    1116:{'x1':[]}
}
"""
class Local(object):
    def __init__(self):
        object.__setattr__(self,'storage',{})

    def __setattr__(self, key, value):
        ident = threading.get_ident()
        #这里使用了栈的功能，把值添加到列表的尾部
        if ident in self.storage:
            self.storage[ident][key].append(value)
        else:
            self.storage[ident] = {key:[value,]}

    def __getattr__(self, item):
        ident = threading.get_ident()
        #取值得时候取最后进入列表额的
        if ident not in self.storage:
            return
        return self.storage[ident][item][-1]

local = Local()

def task(arg):
    local.x1 = arg
    print(local.x1)

for i in range(5):
    t = threading.Thread(target=task,args=(i,))
    t.start()
```

### 6.flask源码关于local的实现

```python
try:
    # 协程
    from greenlet import getcurrent as get_ident
except ImportError:
    try:
        from thread import get_ident
    except ImportError:
        from _thread import get_ident
"""
__storage__ = {
    1111:{"stack":[pl] }
}
"""
class Local(object):

    def __init__(self):
        # self.__storage__ = {}
        # self.__ident_func__ = get_ident
        object.__setattr__(self, "__storage__", {})
        object.__setattr__(self, "__ident_func__", get_ident)

    def __iter__(self):
        return iter(self.__storage__.items())

    def __release_local__(self):
        self.__storage__.pop(self.__ident_func__(), None)

    def __getattr__(self, name):
        try:
            return self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)

    def __setattr__(self, name, value):
        ident = self.__ident_func__() # 1111
        storage = self.__storage__
        try:
            storage[ident][name] = value
        except KeyError:
            storage[ident] = {name: value}

    def __delattr__(self, name):
        try:
            del self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)

class LocalStack(object):
    def __init__(self):
        self._local = Local()
    def push(self, obj):
        """Pushes a new item to the stack"""
        # self._local.stack == getattr
        # rv = None
        rv = getattr(self._local, "stack", None)
        if rv is None:
            self._local.stack = rv = []
        rv.append(obj)
        return rv

    def pop(self):
        stack = getattr(self._local, "stack", None)
        if stack is None:
            return None
        elif len(stack) == 1:
            # release_local(self._local)
            # del __storage__[1111]
            return stack[-1]
        else:
            return stack.pop()

    @property
    def top(self):
        try:
            return self._local.stack[-1]
        except (AttributeError, IndexError):
            return None

obj = LocalStack()
obj.push('汪洋')
obj.push('成说')

print(obj.top)

obj.pop()
obj.pop()
```

总结：

```python
在flask中有个local类，他和threading.local的功能一样，为每个线程开辟空间进行存取数据，他们两个的内部实现机制，内部维护一个字典，以线程(协程)ID为key，进行数据隔离，如：
    __storage__ = {
		1211:{'k1':123}
    }
    
    obj = Local()
    obj.k1 = 123
    
在flask中还有一个LocalStack的类，他内部会依赖local对象，local对象负责存储数据，localstack对象用于将local中的值维护成一个栈。
	__storage__ = {
		1211:{'stack':['k1',]}
    }

	obj= LocalStack()
    obj.push('k1')
    obj.top
    obj.pop()
```

7.flask源码中总共有2个localstack对象

```python
# context locals
__storage__ = {
	1111:{'stack':[RequestContext(reqeust,session),]},
    1123:{'stack':[RequestContext(reqeust,session),]},
}
_request_ctx_stack = LocalStack()


__storage__ = {
	1111:{'stack':[AppContenxt(app,g),]}
    1123:{'stack':[AppContenxt(app,g),]},
}
_app_ctx_stack = LocalStack()
```

```python
_request_ctx_stack.push('小魔方')
_app_ctx_stack.push('大魔方')
```

- 上下文管理
  - 请求上下文管理
  - 应用上下文管理

![](http://9017499461.linshutu.top/%E4%B8%8A%E4%B8%8B%E6%96%87%E7%AE%A1%E7%90%86.JPG)

### 7.源码初识

#### 7.1 项目启动

- 实例化Flask对象

  ```
  app = Flask(__name__)
  ```

  ```python
  1. 对app对象封装一些初始化的值。
  	app.static_url_path
  	app.static_folder
  	app.template_folder
  	app.view_functions = {}
  2. 添加静态文件的路由
      self.add_url_rule(
          self.static_url_path + "/<path:filename>",
          endpoint="static",
          host=static_host,
          view_func=self.send_static_file,
          )
          
  3. 实例化了url_map的对象，以后在map对象中放 【/index/ 函数的对象别名】
      class Flask(object):
          url_rule_class = Rule
          url_map_class = Map
  
          def __init__(self...):
              self.static_url_path
              self.static_folder
              self.template_folder
              self.view_functions = {}
              self.url_map = self.url_map_class()
      app = Flask()
      app.view_functions
  app.url_rule_class
  ```

- 加载配置文件（给app的config进行赋值）

  ```python
  from flask import Flask
  
  app = Flask(__name__,static_url_path='/xx')
  
  app.config.from_object('xx.xx')
  ```

  ```python
  1. 读取配置文件中的所有键值对，并将键值对全都放到Config对象。（Config是一个字典）
  2. 把包含所有配置文件的Config对象，赋值给 app.config
  
  ```

- 添加路由映射

  ```python
  from flask import Flask
  
  app = Flask(__name__,static_url_path='/xx')
  
  @app.route('/index')
  def index():
      return 'hello world'
  ```

  ```python
  1. 将 url = /index  和  methods = [GET,POST]  和 endpoint = "index"封装到Rule对象
  
  2. 将Rule对象添加到 app.url_map中。
  
  3. 把endpoint和函数的对应关系放到 app.view_functions中。
  ```

- 截止目前

  ```
  app.config
  app.url_map
  app.view_functions
  ```

- 运行flask

  ```python
  from flask import Flask
  
  app = Flask(__name__,static_url_path='/xx')
  
  @app.route('/index')
  def index():
      return 'hello world'
  
  if __name__ == '__main__':
      app.run()
  ```

  ```python
  1. 内部调用werkzeug的run_simple，内部创建socket，监听IP和端口，等待用户请求到来。
  
  2. 一旦有用户请求，执行app.__call__方法。
  
  	class Flask(object):
          def __call__(self,envion,start_response):
              pass
          def run(self):
  			run_simple(host, port, self, **options)
  
      if __name__ == '__main__':
          app.run()
  ```

#### 7.2 有用户请求到来

- 创建ctx = RequestContext对象，其内部封装了 Request对象和session数据。 

- 创建app_ctx = AppContext对象，其内部封装了App和g。 

- 然后ctx.push触发将 ctx 和 app_ctx 分别通过自己的LocalStack对象将其放入到Local中，Local的本质是以线程ID为key，以{“stack”:[]}为value的字典。

  ```
  {
  	1111:{“stack”:[ctx,]}
  }
  
  {
  	1111:{“stack”:[app_ctx,]}
  }
  ```

  注意：以后再想要获取 request/session / app / g时，都需要去local中获取。 

- 执行所有的before_request函数

- 执行视图函数

- 执行所有after_request函数（session加密放到cookie中）

- 销毁ctx和app_ctx



### 8.了解源码流程之后，使用：session、request、app、g

- 偏函数

  ```python
  import functools
  
  # 偏函数
  """
  def func(a1,a2):
      print(a1,a2)
  
  new_func = functools.partial(func,123)
  new_func(2)
  """
  ```

- 私有成员

  ```python
  class Foo:
      def __init__(self):
          self.name = 'alex'
          self.__age = 123
  
  
  obj = Foo()
  
  print(obj.name)
  print(obj._Foo__age)
  ```

- setattr

- setitem

```python
# session, request, current_app, g 全部都是LocalProxy对象。
"""
session['x'] = 123     ctx.session['x'] = 123
request.method         ctx.request.method
current_app.config    app_ctx.app.config
g.x1                  app_ctx.g.x1
"""
```

### 9.g到底是个什么鬼？

```
在一次请求请求的周期，可以在g中设置值，在本次的请求周期中都可以读取或复制。
相当于是一次请求周期的全局变量。 
```

```python
from flask import Flask,g

app = Flask(__name__,static_url_path='/xx')

@app.before_request
def f1():
    g.x1 = 123

@app.route('/index')
def index():
    print(g.x1)
    return 'hello world'

if __name__ == '__main__':
    app.run()
```



## 总结

- 第一阶段：启动flask程序，加载特殊装饰器、路由，把他们封装  app= Flask对象中。 
- 第二阶段：请求到来
  - 创建上下文对象：应用上下文、请求上下文。
  - 执行before / 视图 / after 
  - 销毁上下文对象 

### flask请求完整的生命周期

![](http://9017499461.linshutu.top/flask%E5%AE%8C%E6%95%B4%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

启动先执行manage.py 中的    app.run()

```python
class Flask(_PackageBoundObject):　　 
    def run(self, host=None, port=None, debug=None, **options):
    　　from werkzeug.serving import run_simple
    　　try:
        　　#run_simple 是werkzeug 提供的方法，会执行第三个参数 self()
        　　run_simple(host, port, self, **options)
```

执行app()，对象()表示调用对象的`__call__`方法

```python
class Flask(_PackageBoundObject):　　 
    def __call__(self, environ, start_response):
        return self.wsgi_app(environ, start_response)
```

又调用了app.wsgi_app方法

```python
class Flask(_PackageBoundObject):　　 
    def wsgi_app(self, environ, start_response):
        #1.
        ctx = self.request_context(environ)　　　　       
        self.request_context
        #2.
        ctx.push()　　 　　
        try:
            try:　　　　　　　　　 
                #3.执行视图函数
                response = self.full_dispatch_request()
            except Exception as e:
                error = e　　　　　　　　　 
                #4.
                response = self.handle_exception(e)
            except:
                error = sys.exc_info()[1]
                raise
            return response(environ, start_response)
        finally:　　　　　　　
            #5.
            ctx.auto_pop(error)
```

第1步：执行app.request_context方法，把请求的相关信息传进去了

```python
class Flask(_PackageBoundObject):　　 
    def request_context(self, environ):
        return RequestContext(self, environ)
```

返回了一个RequestContext类的实例对象

```python
class RequestContext(object):　　 
    def __init__(self, app, environ, request=None):
        self.app = app
        if request is None:
            request = app.request_class(environ) 　　　　　　  
            app.request_class = Request       
        self.request = request
        self.session = None
```

在init构造方法中注意app又调用了request_class方法，也就是Request 实例一个对象，

那么第1步我们知道：

```
ctx是一个RequestContext对象，这个对象里面封装了两个主要的属性，一个是self.request = Request实例的对象，Request对象里面封装了请求进来的所有数据；另外一个是self.session = None就可以了
```

第2步：执行ctx.push()方法

因为ctx是RequestContext类的对象，那我们就要去RequestContext类中找push方法

```python
class RequestContext(object):　　 
    def push(self):
　　　　 #2.1.
        app_ctx = _app_ctx_stack.top
        if app_ctx is None or app_ctx.app != self.app:
            app_ctx = self.app.app_context()　　　　　　　　　　　　
            # self.app.app_context = app.app_context = AppContext(app)
            app_ctx.push()　　　　　
            #2.2.　　　　  
            _request_ctx_stack.push(self)　　　　　　　　
            #_request_ctx_stack = LocalStack()　　　　 
            #2.3.        
            self.session = self.app.open_session(self.request)        
            #判断没有 secret_key时：        
            if self.session is None:            
                self.session = self.app.make_null_session()            
                raise RuntimeError('The session is unavailable because no secret ''key was set.)
```

第2.1步：到_app_ctx_stack这个栈中取最后一个数据，如果未取到或者取到的不是当前的app，就调用app.app_context()方法，就是新实例一个上下文app_ctx对象，再执行app_ctx.push()方法     （在这再次强调，因为app_ctx是AppContext对象，就要先去AppContext类中找push方法），

```python
class AppContext(object):　　 
    def push(self):
        _app_ctx_stack.push(self)     #把新创建的app_ctx上下文app对象添加到了_app_ctx_stack这个栈中
        appcontext_pushed.send(self.app)   #在这里遇到了第一个信号，请求app上下文push时执行
```

第2.2步：LocalStack类的对象调用push方法

```python
class LocalStack(object):　　
    def push(self, obj):
        rv = getattr(self._local, 'stack', None)       #self._local = Local()　　　　      #第一次的时候rv肯定是None
        if rv is None:
            self._local.stack = rv = []      #Local对象 .stack = rv = [] 就执行了对象的 __setattr__方法
        rv.append(obj)                       #把 ctx对象添加到Local类的列表中
        return rv
```

```python
try:
    from greenlet import getcurrent as get_ident
except ImportError:
    try:
        from thread import get_ident
    except ImportError:
        from _thread import get_ident

class Local(object):        
    def __init__(self):        
        object.__setattr__(self, '__storage__', {})             #这里为什么用object.__setattr__  而不是直接用self.__storage__={}        
        object.__setattr__(self, '__ident_func__', get_ident)   #如果用self的方式设置属性，就会触发self的__setattr__方法，就会无限的循环　　
    def __setattr__(self, name, value):
        ident = self.__ident_func__()
        storage = self.__storage__
        try:
            storage[ident][name] = value    # {"唯一标识1"：{"stack":[]}，"唯一标识2"：{"stack":[]}}   和本地线程类似
        except KeyError:
            storage[ident] = {name: value}
```

 第2.3步：给ctx.session赋值，执行app.open_session(ctx.request)

```python
class Flask(_PackageBoundObject):　　 
    def open_session(self, request):
        return self.session_interface.open_session(self, request)　　　　　#return SecureCookieSessionInterface().open_session(app, request)　　　　　#所以就要去SecureCookieSessionInterface类找open_session方法
```

```python
class SecureCookieSessionInterface(SessionInterface):　　 
    def open_session(self, app, request):
        # 查看 是否有secret_key
        s = self.get_signing_serializer(app)
        if s is None:
            return None
        val = request.cookies.get(app.session_cookie_name)
        # 请求第一次来的时候取不到值
        if not val:
            return self.session_class()
            #返回了一个 类似字典
        max_age = total_seconds(app.permanent_session_lifetime)
        try:
            data = s.loads(val, max_age=max_age)  #loads 作用是： 反序列化+解析乱码
            return self.session_class(data)       ##返回了一个 类似字典对象，对象里面有data
        except BadSignature:
            return self.session_class()
```

那么第2步我们知道：

```
1.把app_ctx上下文对象添加到了_app_ctx_stack这个栈中2.把 ctx请求对象添加到Local类的列表中3.执行open_session方法，把session加载到内
```

 第3步：app.full_dispatch_request()   执行视图函数 

```python
class Flask(_PackageBoundObject):
    def full_dispatch_request(self):
        #3.1 
        self.try_trigger_before_first_request_functions()
        try:
            request_started.send(self)     # 信号 - 请求到来前执行
            # 3.2 
            rv = self.preprocess_request()
            if rv is None:
                # 3.3 如果所有的中间件都通过了， 执行视图函数
                rv = self.dispatch_request()
　　　　 #3.4 
        return self.finalize_request(rv)
```

第3.1步：找到所有的 执行一次的 伪中间件 执行

```python
class Flask(_PackageBoundObject):
    def try_trigger_before_first_request_functions(self):

        with self._before_request_lock:
            for func in self.before_first_request_funcs:
                func()
```

第3.2步：找到所有的 伪中间件的执行

```python
class Flask(_PackageBoundObject):
    def preprocess_request(self):

        funcs = self.before_request_funcs.get(None, ())
        for func in funcs:
            rv = func()
            if rv is not None:
                return rv
```

第3.3步：

```python
class Flask(_PackageBoundObject):
    def dispatch_request(self):
        #获取请求的ctx对象中的request数据
        req = _request_ctx_stack.top.request
        #获取请求的url
        rule = req.url_rule
        #执行视图函数
        return self.view_functions[rule.endpoint](**req.view_args)
```

第3.4步：

```python
class Flask(_PackageBoundObject):
    def finalize_request(self, rv, from_error_handler=False):
        response = self.make_response(rv)   #通过make_response方法后就可以对返回值进行设置响应头等数据了
        try:　　　　　　　#3.4.1
            response = self.process_response(response)
            request_finished.send(self, response=response)  #信号 -  请求结束后执行
        return response
```

第3.4.1步：

```python
class Flask(_PackageBoundObject):
    def process_response(self, response):
        ctx = _request_ctx_stack.top
        #找到所有的 after_request 伪中间件执行
        funcs = ctx._after_request_functions
        for handler in funcs:
            response = handler(response)
        # 3.4.1.1 如果有session就执行self.save_session方法
        if not self.session_interface.is_null_session(ctx.session):　　　　　     			self.session_interface = SecureCookieSessionInterface() 　　　　　　   				#3.4.1.2　　　　 　　 
        self.save_session(ctx.session, response) return response
```

第3.4.1.1步： 到SecureCookieSessionInterface类中找is_null_session方法，发现没有，就去它基类SessionInterface中找

```python
class SessionInterface(object):
    def is_null_session(self, obj):
        #判断ctx.session 是不是 self.null_session_class = NullSession 类或者它派生类的对象
        return isinstance(obj, self.null_session_class)
```

第3.4.1.2步：执行了SecureCookieSessionInterface类的save_session方法

```python
class Flask(_PackageBoundObject):
    def save_session(self, session, response):
        return self.session_interface.save_session(self, session, response)
        # return SecureCookieSessionInterface().save_session(self, session, response)
class SecureCookieSessionInterface(SessionInterface):
    def save_session(self, app, session, response):
        #给响应设置cookie
        response.set_cookie(app.session_cookie_name, val,
                            expires=expires, httponly=httponly,
                            domain=domain, path=path, secure=secure)
```

补充：自定义session

第4步：

```python
class Flask(_PackageBoundObject):
    def handle_exception(self, e):
        got_request_exception.send(self, exception=e)    #信号 - 请求执行出现异常时执行
```

第5步： 执行了RequestContext 的 pop 方法

```python
class RequestContext(object):
    def auto_pop(self, exc):
        else:
            self.pop(exc)
```

```python
class RequestContext(object):
    def pop(self, exc=_sentinel):
　　　　 try:
        　　if not self._implicit_app_ctx_stack:
　　　　　　　　  #5.1
            　　self.app.do_teardown_request(exc)
        finally:
　　　　　　  # 请求结束时  request上下文的栈中就把请求pop掉
            rv = _request_ctx_stack.pop()
        　　 if app_ctx is not None:
　　　　　　　　　　#5.2
        　　 　　 app_ctx.pop(exc)
```

第5.1步： 执行  app.do_teardown_request方法

```python
class Flask(_PackageBoundObject):
    def do_teardown_request(self, exc=_sentinel):
　　　　 # 信号 - 请求执行完毕后自动执行（无论成功与否）
        request_tearing_down.send(self, exc=exc)
```

第5.2步：

```python
class AppContext(object):
    def pop(self, exc=_sentinel):        
        try:            
            if self._refcnt <= 0:　　　　　　　　　 
                #5.2.1                
                self.app.do_teardown_appcontext(exc)　　　　 
                # 信号 - 请求上下文pop时执行
        appcontext_popped.send(self.app)
```

第5.2.1步：

```python
class Flask(_PackageBoundObject):
    def do_teardown_appcontext(self, exc=_sentinel):
        # 信号 - 请求上下文执行完毕后自动执行（无论成功与否）
        appcontext_tearing_down.send(self, exc=exc)
```

## 扩展

```python
class Foo:
     # 

obj = Foo()
obj() # __call__

obj[x1] = 123 # __setitem__
obj[x2]  # __getitem__

obj.x1 = 123 # __setattr__
obj.x2  # __getattr__
```

SQLhelper

- 方式一

  ```python
  import pymysql
  import threading
  from DBUtils.PooledDB import PooledDB
  
  """
  storage = {
      1111:{'stack':[]}
  }
  """
  
  class SqlHelper(object):
      def __init__(self):
          self.pool = PooledDB(
              creator=pymysql,  # 使用链接数据库的模块
              maxconnections=6,  # 连接池允许的最大连接数，0和None表示不限制连接数
              mincached=2,  # 初始化时，链接池中至少创建的链接，0表示不创建
              blocking=True,  # 连接池中如果没有可用连接后，是否阻塞等待。True，等待；False，不等待然后报错
              ping=0,
              # ping MySQL服务端，检查是否服务可用。# 如：0 = None = never, 1 = default = whenever it is requested, 2 = when a cursor is created, 4 = when a query is executed, 7 = always
              host='127.0.0.1',
              port=3306,
              user='root',
              password='222',
              database='cmdb',
              charset='utf8'
          )
          self.local = threading.local()
  
      def open(self):
          conn = self.pool.connection()
          cursor = conn.cursor()
          return conn, cursor
  
      def close(self, cursor, conn):
          cursor.close()
          conn.close()
  
      def fetchall(self, sql, *args):
          """ 获取所有数据 """
          conn, cursor = self.open()
          cursor.execute(sql, args)
          result = cursor.fetchall()
          self.close(conn, cursor)
          return result
  
      def fetchone(self, sql, *args):
          """ 获取所有数据 """
          conn, cursor = self.open()
          cursor.execute(sql, args)
          result = cursor.fetchone()
          self.close(conn, cursor)
          return result
  
      def __enter__(self):
          conn,cursor = self.open()
          rv = getattr(self.local,'stack',None)
          if not rv:
              self.local.stack = [(conn,cursor),]
          else:
              rv.append((conn,cursor))
              self.local.stack = rv
          return cursor
  
      def __exit__(self, exc_type, exc_val, exc_tb):
          rv = getattr(self.local,'stack',None)
          if not rv:
              # del self.local.stack
              return
          conn,cursor = self.local.stack.pop()
          cursor.close()
          conn.close()
  
  db = SqlHelper()
  ```

  ```python
  from sqlhelper import db
  
  
  # db.fetchall(...)
  # db.fetchone(...)
  
  with db as c1:
      c1.execute('select 1')
      with db as c2:
          c1.execute('select 2')
      print(123)
  
  
  ```

- 方式二

  ```python
  import pymysql
  import threading
  from DBUtils.PooledDB import PooledDB
  
  POOL = PooledDB(
              creator=pymysql,  # 使用链接数据库的模块
              maxconnections=6,  # 连接池允许的最大连接数，0和None表示不限制连接数
              mincached=2,  # 初始化时，链接池中至少创建的链接，0表示不创建
              blocking=True,  # 连接池中如果没有可用连接后，是否阻塞等待。True，等待；False，不等待然后报错
              ping=0,
              # ping MySQL服务端，检查是否服务可用。# 如：0 = None = never, 1 = default = whenever it is requested, 2 = when a cursor is created, 4 = when a query is executed, 7 = always
              host='127.0.0.1',
              port=3306,
              user='root',
              password='222',
              database='cmdb',
              charset='utf8'
          )
  
  class SqlHelper(object):
      def __init__(self):
          self.conn = None
          self.cursor = None
  
      def open(self):
          conn = POOL.connection()
          cursor = conn.cursor()
          return conn, cursor
  
      def close(self):
          self.cursor.close()
          self.conn.close()
  
      def __enter__(self):
          self.conn,self.cursor = self.open()
          return self.cursor
  
      def __exit__(self, exc_type, exc_val, exc_tb):
          self.close()
  ```

  ```python
  
  # ################## 使用 ##################
  
  with SqlHelper() as c1:
      c1.execute('select 1')
      with SqlHelper() as c2:
          c2.execute('select 2')
      print(666)
  
  with SqlHelper() as cursor:
      cursor.execute('select 1')
  
  with SqlHelper() as cursor:
      cursor.execute('select 1')
  ```


### 1.drf源码分析系列

- 01 restful规范
- 02 从cbv到drf的视图 / 快速了解drf
- 03 视图
- 04 版本
- 05 认证
- 06 权限
- 07 节流
- 08 jwt
- 持续更新中...

### 2.flask源码分析系列

- 01 werkzurg 了解wsgi
- 02 快速使用
- 03 threading.local和高级
- 04 LocalStack和Local对象实现栈的管理
- 05 Flask源码之：配置加载
- 06 Flask源码之：路由加载
- 持续更新中...

印象笔记 、 有道云笔记









































