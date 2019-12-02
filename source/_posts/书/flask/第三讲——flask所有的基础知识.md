---
title: 第三讲——flask所有的基础知识
id: 4
date: 2019-11-21 20:00:00
tags: flask
toc: true
comment: true
---

## 内容回顾

1.什么是接口？

```python
#两个方面
- interface类型，Python没有，Java/C#语言才有。用于约束实现了该接口的类中必须有某些指定方法。
- api也可以成为一个接口。
```

2.抽象类和抽象方法

```
他既具有约束的功能又具有提供子类继承方法的功能，Python中通过abc实现。 
```

3.重载和重写？

```
重载就是根据方法中参数的不同调用不同的方法，实现不同的功能。
重写就是方法对父类方法的重写。
```

<!----more---->

4.flask和django的区别？

5.什么是数据库链接池？以及作用？

```
数据库连接池是创建一些和数据库的连接，共我们去调用，我们使用完之后再把该连接放回到数据库连接池中。
在没有数据库连接池的时候，我们每操作一次数据库就建立一个连接，这样就大大降低了效率。或者我们使用同一个连接，这又会造成数据库压力。数据连接池就很好的解决了上面的问题
```

6.sqlhelper

7.面向对象的上下文管理

8.上下文管理和SQLHelper

```python
import pymysql
from DBUtils.PooledDB import PooledDB

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

    def open(self):
        conn = self.pool.connection()
        cursor = conn.cursor()
        return conn,cursor

    def close(self,cursor,conn):
        cursor.close()
        conn.close()

    def fetchall(self,sql, *args):
        """ 获取所有数据 """
        conn,cursor = self.open()
        cursor.execute(sql, args)
        result = cursor.fetchall()
        self.close(conn,cursor)
        return result

    def fetchone(self,sql, *args):
        """ 获取所有数据 """
        conn, cursor = self.open()
        cursor.execute(sql, args)
        result = cursor.fetchone()
        self.close(conn, cursor)
        return result

    def __enter__(self):
        return self.open()[1]

    def __exit__(self, exc_type, exc_val, exc_tb):
        print(exc_type, exc_val, exc_tb)


db = SqlHelper()
```

## 今日概要

- wsgi
- 创建flask对象
  - 模板
  - 静态文件
- 路由系统
  - 路由的应用：装饰器（推荐）、方法
  - 动态路由
- 视图
  - FBV
  - CBV
- 模板
  - 继承
  - include
  - 自定义标签
- 特殊装饰器
  - before_request充当中间件角色



## 今日详细

### 1.wsgi 找源码的流程

```python
from werkzeug.serving import run_simple
from werkzeug.wrappers import BaseResponse

def func(environ, start_response):
    print('请求来了')
    response = BaseResponse('你好')
    return response(environ, start_response)


if __name__ == '__main__':
    run_simple('127.0.0.1', 5000, func)
```

```python
"""
    1.程序启动，等待用户请求到来
        app.run()
    2.用户请求到来 app()    
        app.__call__
"""

from flask import Flask

app = Flask(__name__)

@app.route('/index')
def index():
    return 'hello world'

if __name__ == '__main__':
    app.run()
```

### 2.flask对象

```python
from flask import Flask,render_template

app = Flask(__name__,template_folder='templates',static_folder='static')

@app.route('/index')
def index():
    return render_template('index.html')

if __name__ == '__main__':
    app.run()
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>首页</h1>
    <img src="/static/xx/xx/mm.jpg" />
    
    <!-- 建议 -->
    <img src="{{ url_for('static',filename='xx/xx/mm.jpg')}}" />
</body>
</html>
```

### 3.配置文件

#### 3.1 基于全局变量

![](http://9017499461.linshutu.top/flask%E5%85%A8%E5%B1%80%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%85%A8%E5%B1%80.JPG)

#### 3.2 基于类的方式

![](http://9017499461.linshutu.top/flask%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%9F%BA%E4%BA%8E%E7%B1%BB.JPG)

```python
flask中的配置文件是一个flask.config.Config对象（继承字典）,默认配置为：
    {
        'DEBUG':                                get_debug_flag(default=False),  是否开启Debug模式
        'TESTING':                              False,                          是否开启测试模式
        'PROPAGATE_EXCEPTIONS':                 None,                          
        'PRESERVE_CONTEXT_ON_EXCEPTION':        None,
        'SECRET_KEY':                           None,
        'PERMANENT_SESSION_LIFETIME':           timedelta(days=31),
        'USE_X_SENDFILE':                       False,
        'LOGGER_NAME':                          None,
        'LOGGER_HANDLER_POLICY':               'always',
        'SERVER_NAME':                          None,
        'APPLICATION_ROOT':                     None,
        'SESSION_COOKIE_NAME':                  'session',
        'SESSION_COOKIE_DOMAIN':                None,
        'SESSION_COOKIE_PATH':                  None,
        'SESSION_COOKIE_HTTPONLY':              True,
        'SESSION_COOKIE_SECURE':                False,
        'SESSION_REFRESH_EACH_REQUEST':         True,
        'MAX_CONTENT_LENGTH':                   None,
        'SEND_FILE_MAX_AGE_DEFAULT':            timedelta(hours=12),
        'TRAP_BAD_REQUEST_ERRORS':              False,
        'TRAP_HTTP_EXCEPTIONS':                 False,
        'EXPLAIN_TEMPLATE_LOADING':             False,
        'PREFERRED_URL_SCHEME':                 'http',
        'JSON_AS_ASCII':                        True,
        'JSON_SORT_KEYS':                       True,
        'JSONIFY_PRETTYPRINT_REGULAR':          True,
        'JSONIFY_MIMETYPE':                     'application/json',
        'TEMPLATES_AUTO_RELOAD':                None,
    }
 
方式一：
    app.config['DEBUG'] = True
 
    PS： 由于Config对象本质上是字典，所以还可以使用app.config.update(...)
 
方式二：
    app.config.from_pyfile("python文件名称")
        如：
            settings.py
                DEBUG = True
 
            app.config.from_pyfile("settings.py")
 
    app.config.from_envvar("环境变量名称")
        环境变量的值为python文件名称名称，内部调用from_pyfile方法
 
 
    app.config.from_json("json文件名称")
        JSON文件名称，必须是json格式，因为内部会执行json.loads
 
    app.config.from_mapping({'DEBUG':True})
        字典格式
 
    app.config.from_object("python类或类的路径")
 
        app.config.from_object('pro_flask.settings.TestingConfig')
 
        settings.py
 
            class Config(object):
                DEBUG = False
                TESTING = False
                DATABASE_URI = 'sqlite://:memory:'
 
            class ProductionConfig(Config):
                DATABASE_URI = 'mysql://user@localhost/foo'
 
            class DevelopmentConfig(Config):
                DEBUG = True
 
            class TestingConfig(Config):
                TESTING = True
 
        PS: 从sys.path中已经存在路径开始写
     
 
    PS: settings.py文件默认路径要放在程序root_path目录，如果instance_relative_config为True，则就是instance_path目录
```

### 4.路由系统

- 路由的两种写法

  ```python
  def index():
      return render_template('index.html')
  app.add_url_rule('/index', 'index', index)
  
  
  # 公司里一般用这种方式
  @app.route('/login')
  def login():
      return render_template('login.html')
  ```

- 路由加载的源码流程

  ```
  - 将url和函数打包成为 rule 对象
  - 将rule对象添加到map对象中。
  - app.url_map = map对象
  ```

- 动态路由

  ```python
  @app.route('/login')
  def login():
      return render_template('login.html')
      
  @app.route('/login/<name>')
  def login(name):
  	print(type(name))
      return render_template('login.html')
      
  @app.route('/login/<int:name>')
  def login(name):
  	print(type(name))
      return render_template('login.html')
  ```

- 支持正则表达式的路由

  ```python
  from flask import Flask,render_template
  
  app = Flask(__name__)
  
  
  from werkzeug.routing import BaseConverter
  class RegConverter(BaseConverter):
      def __init__(self, map, regex):
          super().__init__(map)
          self.regex = regex
  app.url_map.converters['regex'] = RegConverter
  
  @app.route('/index/<regex("\d+"):x1>')
  def index(x1):
      return render_template('index.html')
  
  if __name__ == '__main__':
      app.run()
  ```


### 5.视图

#### 5.1 FBV

```python
def index():
    return render_template('index.html')
app.add_url_rule('/index', 'index', index)


# 公司里一般用这种方式
@app.route('/login')
def login():
    return render_template('login.html')
```

#### 5.2 CBV

```python
from flask import Flask,render_template,views

app = Flask(__name__,)

def test1(func):
    def inner(*args,**kwargs):
        print('before1')
        result = func(*args,**kwargs)
        print('after1')
        return result
    return inner

def test2(func):
    def inner(*args,**kwargs):
        print('before2')
        result = func(*args,**kwargs)
        print('after2')
        return result
    return inner


class UserView(views.MethodView):
    methods = ['GET',"POST"]
	
    #装饰器
    decorators = [test1,test2]


    def get(self):
        print('get')
        return 'get'

    def post(self):
        print('post')
        return 'post'

app.add_url_rule('/user', view_func=UserView.as_view('user')) # endpoint

if __name__ == '__main__':
    app.run()
    
#注意：before1-->before2-->get-->after2-->after1
```

### 6.模板

Flask使用的是Jinja2模板，所以其语法与Django无差别。

#### 6.1 基本用法

flask比django更加接近Python。 

```python
from flask import Flask,render_template

app = Flask(__name__,)

def func(arg):
    return '你好' + arg

@app.route('/md')
def index():
    nums = [11,222,33]
    return render_template('md.html',nums=nums,f=func)


if __name__ == '__main__':
    app.run()
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>头</h1>
        {% block content %} {% endblock %}
    <h1>底</h1>
</body>
</html>
```

```html
<form action="">
    <input type="text">
    <input type="text">
    <input type="text">
    <input type="text">
    <input type="text">
</form>
```

模板继承

```html
{% extends 'layout.html' %}


{% block content %}
    <h1>MD</h1>
    {% include 'form.html' %}
    {{ f("pl") }}
{% endblock %}
```

#### 6.2 定义全局模板方法

```python
from flask import Flask,render_template

app = Flask(__name__,)

@app.template_global() #  {{ func("pl") }}
def func(arg):
    return 'pl' + arg

@app.template_filter() # {{ "pl"|x1("pl1") }}
def x1(arg,name):
    return 'pl' + arg + name

@app.route('/md/hg')
def index():
    return render_template('md_hg.html')

if __name__ == '__main__':
    app.run()
```

注意：在蓝图中注册时候，应用返回只有本蓝图。

### 7.特殊装饰器(中间件)

```python
from flask import Flask,render_template,request

app = Flask(__name__)

@app.before_request
def f1():
    if request.path == '/login':
        return
    print('f1')
    # return '123'

@app.after_request
def f10(response):
    print('f10')
    return response

@app.route('/index')
def index():
    print('index')
    return render_template('index.html')

if __name__ == '__main__':
    app.run()
```

多个装饰器

```python
from flask import Flask,render_template,request

app = Flask(__name__)

@app.before_request
def f1():
    print('f1')

@app.before_request
def f2():
    print('f2')

@app.after_request
def f10(response):
    print('f10')
    return response

@app.after_request
def f20(response):
    print('f20')
    return response

@app.route('/index')
def index():
    print('index')
    return render_template('index.html')

if __name__ == '__main__':
    app.run()
    app.__call__
    
#注意：f1-->f2-->index-->f20-->f10  请求按照顺序进来，回去的时候，将列表反转，所有顺序是反着回去
```

注意：before_after/request可以在蓝图中定义，在蓝图中定义的话，作用域只在本蓝图。

```python
from flask import Flask, flash, redirect, render_template, request
 
app = Flask(__name__)
app.secret_key = 'some_secret'
 
@app.route('/')
def index1():
    return render_template('index.html')
 
@app.route('/set')
def index2():
    v = request.args.get('p')
    flash(v)
    return 'ok'
 
class MiddleWare:
    def __init__(self,wsgi_app):
        self.wsgi_app = wsgi_app
 
    def __call__(self, *args, **kwargs):
 
        return self.wsgi_app(*args, **kwargs)
 
if __name__ == "__main__":
    app.wsgi_app = MiddleWare(app.wsgi_app)
    app.run(port=9999)
```

### 8.小细节

```python
from flask import Flask,render_template

app = Flask(__name__,)

@app.route('/index')
def index():
    return render_template('index.html')


@app.before_request
def func():
    print('xxx')

def x1():
    print('xxx')
app.before_request(x1)

if __name__ == '__main__':
    app.run()
```

## 赠送：threading.local

```python
import time
import threading

# 当每个线程在执行 val1.xx=1 ，在内部会为此线程开辟一个空间，来存储 xx=1
# val1.xx,找到此线程自己的内存地址去取自己存储 xx
val1 = threading.local()

def task(i):
    val1.num = i
    time.sleep(1)
    print(val1.num)

for i in range(4):
    t = threading.Thread(target=task,args=(i,))
    t.start()
```

## 补充

#### message

message是一个基于Session实现的用于保存数据的集合，其特点是：使用一次就删除。

```python
from flask import Flask, flash, redirect, render_template, request, get_flashed_messages

        app = Flask(__name__)
        app.secret_key = 'some_secret'

        @app.route('/')
        def index1():
            messages = get_flashed_messages()
            print(messages)
            return "Index1"

        @app.route('/set')
        def index2():
            v = request.args.get('p')
            flash(v)
            return 'ok'

        if __name__ == "__main__":
            app.run()
```

#### 请求扩展

```python
from flask import Flask, Request, render_template

app = Flask(__name__, template_folder='templates')
app.debug = True


@app.before_first_request
def before_first_request1():
    print('before_first_request1')


@app.before_first_request
def before_first_request2():
    print('before_first_request2')


@app.before_request
def before_request1():
    Request.nnn = 123
    print('before_request1')


@app.before_request
def before_request2():
    print('before_request2')


@app.after_request
def after_request1(response):
    print('before_request1', response)
    return response


@app.after_request
def after_request2(response):
    print('before_request2', response)
    return response


@app.errorhandler(404)
def page_not_found(error):
    return 'This page does not exist', 404


@app.template_global()
def sb(a1, a2):
    return a1 + a2


@app.template_filter()
def db(a1, a2, a3):
    return a1 + a2 + a3


@app.route('/')
def hello_world():
    return render_template('hello.html')


if __name__ == '__main__':
    app.run()
```













































































