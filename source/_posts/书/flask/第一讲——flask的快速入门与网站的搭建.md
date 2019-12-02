---
title: 第一讲——flask的快速入门与网站的搭建
id: 2
date: 2019-11-19 20:00:00
tags: flask
toc: true
comment: true
---

django是个大而全的框架，flask是一个轻量级的框架。

django内部为我们提供了非常多的组件：orm / session / cookie / admin / form / modelform / 路由 / 视图 / 模板 /  中间件 / 分页 / auth / contenttype  / 缓存 / 信号 / 多数据库连接 

flask框架本身没有太多的功能：路由/视图/模板(jinja2)/session/中间件 ，第三方组件非常齐全。 
注意：**django的请求处理是逐一封装和传递； flask的请求是利用上下文管理来实现的**。 

<!----more---->

## 内容回顾

1. 什么是jwt？
2. cmdb的实现原理？
3. 都用到了那些命令？
4. 遇到过哪些bug？
5. 什么是开封封闭原则？

## 今日概要

- flask的快速使用
- 实现一个xx管理系统
- 蓝图

## 今日详细

### 1.flask快速使用

安装

```
pip3 install flask
```

#### 1.1 依赖wsgi Werkzeug

```python
from werkzeug.serving import run_simple

def func(environ, start_response):
    print('请求来了')
    pass

if __name__ == '__main__':
    run_simple('127.0.0.1', 5000, func)
```

```python
from werkzeug.serving import run_simple

class Flask(object):
    
    def __call__(self,environ, start_response):
        return "xx"
    
app = Flask()

if __name__ == '__main__':
    run_simple('127.0.0.1', 5000, app)
    
```

```python
from werkzeug.serving import run_simple

class Flask(object):
    
    def __call__(self,environ, start_response):
        return "xx"
    
    def run(self):
        run_simple('127.0.0.1', 5000, self)
        
app = Flask()

if __name__ == '__main__':
    app.run()
```

#### 1.2 快速使用flask

```python
from flask import Flask

# 创建flask对象
app = Flask(__name__)

@app.route('/index')
def index():
    return 'hello world'


@app.route('/login')
def login():
    return 'login'

if __name__ == '__main__':
    app.run()
```

总结：

- flask框架是基于werkzeug的wsgi实现，flask自己没有wsgi。 
- 用户请求一旦到来，就会之 `app.__call__ `方法 。 
- 写flaks标准流程

#### 1.3 用户登录&用户管理

```python
from flask import Flask, render_template, jsonify,request,redirect,url_for

app = Flask(__name__)

DATA_DICT = {
    1: {'name':'陈硕',"age":73},
    2: {'name':'汪洋',"age":84},
}

@app.route('/login',methods=['GET','POST'])
def login():
    if request.method == 'GET':
        # return '登录' # HttpResponse
        # return render_template('login.html') # render
        # return jsonify({'code':1000,'data':[1,2,3]}) # JsonResponse
        return render_template('login.html')
    user = request.form.get('user')
    pwd = request.form.get('pwd')
    if user == 'changxin' and pwd == "dsb":
        return redirect('/index')
    error = '用户名或密码错误'
    # return render_template('login.html',**{'error':error})
    return render_template('login.html',error=error)

@app.route('/index',endpoint='idx')
def index():
    data_dict = DATA_DICT
    return render_template('index.html',data_dict=data_dict)

@app.route('/edit',methods=['GET','POST'])
def edit():
    nid = request.args.get('nid')
    nid = int(nid)

    if request.method == "GET":
        info = DATA_DICT[nid]
        return render_template('edit.html',info=info)

    user = request.form.get('user')
    age = request.form.get('age')
    DATA_DICT[nid]['name'] = user
    DATA_DICT[nid]['age'] = age
    return redirect(url_for('idx'))

@app.route('/del/<int:nid>')
def delete(nid):
    del DATA_DICT[nid]
    # return redirect('/index')
    return redirect(url_for("idx"))

if __name__ == '__main__':
    app.run()
```

#### 总结

1. flask路由

   ```python
   @app.route('/login',methods=['GET','POST'])
   def login():
   	pass
   ```

2. 路由的参数

   ```python
   @app.route('/login',methods=['GET','POST'],endpoint="login")
   def login():
   	pass
   	
   # 注意：endpoint不能重名
   ```

3. 动态路由

   ```python
   @app.route('/index')
   def login():
   	pass
   	
   @app.route('/index/<name>')
   def login(name):
   	pass
   	
   @app.route('/index/<int:nid>')
   def login(nid):
   	pass
   ```

4. 获取提交的数据

   ```python
   from flask import request
   
   @app.route('/index')
   def login():
   	request.args # GET形式传递的参数
   	request.form # POST形式提交的参数
   ```

5. 返回数据

   ```python
   @app.route('/index')
   def login():
   	return render_template('模板文件')
   	return jsonify()
   	reutrn redirect('/index/') # reutrn redirect(url_for('idx'))
   	return "...."
   ```

6. 模板处理

   ```python
   {{ x }}
   
   {% for item in list %}
   	{{item}}
   {% endfor %}
   ```
   

#### 1.4 保存用户会话信息

```python
from flask import request,render_template,redirect,session
@app.route('/login', methods=["GET", "POST"])
	def login():
		if request.method == "GET":
			return render_template('login.html')
		else:
			username = request.form.get("username")
			password = request.form.get("password")

			# 执行sql
			with mysql() as cursor:
				row_count = cursor.execute("select * from userinfo")
				for count in range(row_count):
					userinfo_obj = cursor.fetchone()
					user = userinfo_obj.get('username')
					pwd = userinfo_obj.get('password')
					if username == user and password == pwd:
                        #登录成功，设置session，但是session不是封装在request里面，而是需要我们						  导入
						session['is_login'] = True
						return redirect("/listbook")

			error = "用户名或密码失败"
			return render_template('login.html', error=error)
```

### 2. 蓝图（blue print)

构建业务功能可拆分的目录结构。

面试题：django的app和flask的蓝图有什么区别？（一样）

总路由：

```python
from .views.book import books

def create_app():
	app = Flask(__name__)
	#蓝图注册
	app.register_blueprint(books)
```

分路由（蓝图）：

```python
from flask import  Blueprint
from flask import request,render_template,redirect,url_for

#注册app
books = Blueprint('book',__name__)

@books.route('/listbook',methods = ["GET","POST"])
def listbook():
    pass
```

**注意：这里面是最容易出现重名问题的，所以在创建的时候一定要检查**

## 总结

1. flask和django的区别？
2. 其他
3. flask的session是保存在：加密的形式保存在浏览器的cookie上。
4. 装饰器相关
   - 编写装饰器，记得加functools
   - 多个装饰器的应用
5. 蓝图



## 扩展

### 配置文件

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

### 路由系统

- @app.route('/login', methods=['GET', 'POST'])
- @app.route('/user/`<username>`')
- @app.route('/post/`<int:post_id>`')
- @app.route('/post/`<float:post_id>`')
- @app.route('/post/`<path:path>`')

常用路由系统有以上的五种，所有的路由系统都是基于以下对应关系来处理：

```python
DEFAULT_CONVERTERS = {
    'default':          UnicodeConverter,
    'string':           UnicodeConverter,
    'any':              AnyConverter,
    'path':             PathConverter,
    'int':              IntegerConverter,
    'float':            FloatConverter,
    'uuid':             UUIDConverter,
}

```

### 请求与相响应

```python
from flask import Flask
from flask import request
from flask import render_template
from flask import redirect
from flask import make_response

app = Flask(__name__)


@app.route('/login.html', methods=['GET', "POST"])
def login():

    # 请求相关信息
    # request.method
    # request.args
    # request.form
    # request.values
    # request.cookies
    # request.headers
    # request.path
    # request.full_path
    # request.script_root
    # request.url
    # request.base_url
    # request.url_root
    # request.host_url
    # request.host
    # request.files
    # obj = request.files['the_file_name']
    # obj.save('/var/www/uploads/' + secure_filename(f.filename))

    # 响应相关信息
    # return "字符串"
    # return render_template('html模板路径',**{})
    # return redirect('/index.html')

    # response = make_response(render_template('index.html'))
    # response是flask.wrappers.Response类型
    # response.delete_cookie('key')
    # response.set_cookie('key', 'value')
    # response.headers['X-Something'] = 'A value'
    # return response


    return "内容"

if __name__ == '__main__':
    app.run()
```

### 蓝图

- 蓝图URL前缀：xxx = Blueprint('account', __name__,url_prefix='/xxx')
- 蓝图子域名：xxx = Blueprint('account', __name__,subdomain='admin')

### 请求扩展

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

































































