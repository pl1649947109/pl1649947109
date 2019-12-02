---
title: 第十讲——django之models锁和事务与中间件
id: 10
date: 2019-10-12 20:30:00
tags: Django
comment: true
---

### 锁和事务

#### 锁

所有匹配的行将被锁定，直到事务结束。这意味着可以通过锁防止数据被其它事务修改。

​		一般情况下如果其他事务锁定了相关行，那么本查询将被阻塞，直到锁被释放。 如果这不想要使查询阻塞的话，使用select_for_update(nowait=True)。 如果其它事务持有冲突的锁，互斥锁, 那么查询将引发 DatabaseError 异常。你也可以使用select_for_update(skip_locked=True)忽略锁定的行。 nowait和　　skip_locked是互斥的，同时设置会导致ValueError。

<!----more---->

​		目前，postgresql，oracle和mysql数据库后端支select_for_update()。 但是，MySQL不支持nowait和skip_locked参数。

原生的mysql代码：

```mysql
select * from book where id=1 for update;
```

在django里面我们这样设置：

```python
models.Book.objects.select_for_update().filter(id=1)
```

#### 事务

原生的mysql代码：

```mysql
begin;  
start transaction;
	select * from t1 where id=1 for update;
commit
rollback;
```

在dajngo中，我们有3种设置方式：

**全局开启：不推荐使用**

它是这样工作的：当有请求过来时，Django会在调用视图方法前开启一个事务。如果请求却正确处理并正确返回了结果，Django就会提交该事务。否则，Django会回滚该事务。

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mxshop',
        'HOST': '127.0.0.1',
        'PORT': '3306',
        'USER': 'root',
        'PASSWORD': '123',
        'OPTIONS': {
            "init_command": "SET default_storage_engine='INNODB'",
　　　　　　　#'init_command': "SET sql_mode='STRICT_TRANS_TABLES'", #配置开启严格sql模式


        }
        "ATOMIC_REQUESTS": True, #全局开启事务，绑定的是http请求响应整个过程
        "AUTOCOMMIT":False, #全局取消自动提交，慎用
    }，
　　'other':{
　　　　'ENGINE': 'django.db.backends.mysql', 
            ......
　　} #还可以配置其他数据库
}
```

**局部使用事务**

使用atomic，我们就可以创建一个具备原子性的代码块。一旦代码块正常运行完毕，所有的修改会被提交到数据库。反之，如果有异常，更改会被回滚。

用法一：给函数做装饰器来使用

```python
from django.db import transaction

@transaction.atomic
def viewfunc(request):
    # This code executes inside a transaction.
    do_stuff()
```

用法二：作为上下文管理器来使用，就是设置事务的保存点

```python
from django.db import transaction

def viewfunc(request):
    # This code executes in autocommit mode (Django's default).
    do_stuff()

    with transaction.atomic():   #保存点
        # This code executes inside a transaction.
        do_more_stuff()

    do_other_stuff()
```

### 中间件

用途：我们学的给视图加装饰器来判断用户是否登陆与否，在站内进行跳转。那么我们后面加入的视图函数每个前面都需要加一个装饰器，这样是不是很麻烦。那么我们今天学的就是解决这个问题。

#### 中间件介绍

中间件顾名思义，是介于request与response处理之间的一道处理过程，相对比较轻量级，并且在全局上改变django的输入与输出。因为改变的是全局，所以需要谨慎实用，用不好会影响到性能。和之前学习爬虫的那个中间件是一样的道理。

我们一直都在使用中间件，只是我们没有注意而已,如我们的settings文件里面的MIDDLEWARE下面都是django为我们配置的中间件：

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

内置中间件详解：https://my.oschina.net/liuyuantao/blog/756778

那现在先让我们看一看，中间件在我们请求生命周期的那个位置吧：

**完整的请求生命周期**：

![](http://9017499461.linshutu.top/%E5%AE%8C%E6%95%B4%E7%9A%84%E8%AF%B7%E6%B1%82%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

#### 自定义中间件

中间件可以定义五个方法：

```python
process_request(self,request)    ****
process_view(self, request, view_func, view_args, view_kwargs)        **
process_template_response(self,request,response)  **
process_exception(self, request, exception)
process_response(self, request, response)    ****
```

以上方法的返回值可以是None或一个HttpResponse对象，如果是None，则继续按照django定义的规则向后继续执行，如果是HttpResponse对象，则直接将该对象返回给用户。

**当用户发起请求的时候会依次经过所有的的中间件，这个时候的请求时process_request,最后到达views的函数中，views函数处理后，在依次穿过中间件，这个时候是process_response,最后返回给请求者。**

##### process_request(请求中间件)

process_request有一个参数，就是request，这个request和视图函数中的request是一样的。

```python
from django.utils.deprecation import MiddlewareMixin

class MD1(MiddlewareMixin):
    def process_request(self, request):
        print("MD1里面的 process_request")

class MD2(MiddlewareMixin):
    def process_request(self, request):
        print("MD2里面的 process_request")
        pass
'''
访问视图，返回结果：
MD1里面的 process_request
MD2里面的 process_request
app01 中的 index视图
把MD1和MD2的位置调换一下，再访问一个视图，会发现终端中打印的内容如下：
MD2里面的 process_request
MD1里面的 process_request
app01 中的 index视图
'''
```

在settings.py的MIDDLEWARE配置项中注册上述两个自定义中间件：

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'app.middlewares.MD1',  # 自定义中间件MD1，这个写的是你项目路径下的一个路径，例如，如果你放在项目下，文件夹名成为utils，那么这里应该写utils.middlewares.MD1
    'middlewares.MD2'  # 自定义中间件MD2
]
```

小结：

- 中间件的process_request方法是在执行视图函数之前执行的
- 当配置多个中间件时，会按照MIDDLEWARE中的注册顺序，也就是列表的索引值，从前到后依次执行的
- 不同中间件之间传递的request都是同一个对象

##### process_response（响应中间件）

它有两个参数，一个是request，一个是response，request就是上述例子中一样的对象，response是视图函数返回的HttpResponse对象。

给上述的M1和M2加上process_response方法：

```python
from django.utils.deprecation import MiddlewareMixin


class MD1(MiddlewareMixin):

    def process_request(self, request):
        print("MD1里面的 process_request")
        #不必须写return值
    def process_response(self, request, response):#request和response两个参数必须有，名字随便取
        print("MD1里面的 process_response")
　　　　 return response  #必须有返回值，写return response，这个response就像一个接力棒一样
        #return HttpResponse('瞎搞') ,如果你写了这个，那么你视图返回过来的内容就被它给替代了

class MD2(MiddlewareMixin):
    def process_request(self, request):
        print("MD2里面的 process_request")
        pass
    def process_response(self, request, response): 
        print("MD2里面的 process_response") 
        return response  #必须返回response，不然你上层的中间件就没有拿到httpresponse对象，就会报错
"""
访问视图结果：
MD1里面的 process_request
MD2里面的 process_request
app01 中的 index视图
MD2里面的 process_response
MD1里面的 process_response
"""
```

process_response方法是在视图函数之后执行的，并且顺序是MD1比MD2先执行。(此时settings.py中 MD2比MD1先注册)

**多个中间件中的process_response方法是按照MIDDLEWARE中的注册顺序倒序执行的，也就是说第一个中间件的process_request方法首先执行，而它的process_response方法最后执行，最后一个中间件的process_request方法最后一个执行，它的process_response方法是最先执行。**

##### process_view（视图只能关键见）

process_view(self, request, view_func, view_args, view_kwargs)

该方法有四个参数:

- request是HttpRequest对象。

- view_func是Django即将使用的视图函数。 （它是实际的函数对象，而不是函数的名称作为字符串。）

- view_args是将传递给视图的位置参数的列表.

- view_kwargs是将传递给视图的关键字参数的字典。 view_args和view_kwargs都不包含第一个视图参数（request）。

　给MD1和MD2添加process_view方法:

```python
from django.utils.deprecation import MiddlewareMixin


class MD1(MiddlewareMixin):

    def process_request(self, request):
        print("MD1里面的 process_request")

    def process_response(self, request, response):
        print("MD1里面的 process_response")
        return response

    def process_view(self, request, view_func, view_args, view_kwargs):
        print("-" * 80)
        print("MD1 中的process_view")
        print(view_func, view_func.__name__) #就是url映射到的那个视图函数，也就是说每个中间件的这个process_view已经提前拿到了要执行的那个视图函数
        #ret = view_func(request) #提前执行视图函数，不用到了上图的试图函数的位置再执行，如果你视图函数有参数的话，可以这么写 view_func(request,view_args,view_kwargs) 
        #return ret  #直接就在MD1中间件这里这个类的process_response给返回了，就不会去找到视图函数里面的这个函数去执行了。

class MD2(MiddlewareMixin):
    def process_request(self, request):
        print("MD2里面的 process_request")
        pass

    def process_response(self, request, response):
        print("MD2里面的 process_response")
        return response

    def process_view(self, request, view_func, view_args, view_kwargs):
        print("-" * 80)
        print("MD2 中的process_view")
        print(view_func, view_func.__name__)
'''
请求访问结果：
MD1里面的 process_request
MD2里面的 process_request
--------------------------------------------------------------------------------
MD1 中的process_view
<function index at 0x000001DE68317488> index
--------------------------------------------------------------------------------
MD2 中的process_view
<function index at 0x000001DE68317488> index
app01 中的 index视图
MD2里面的 process_response
MD1里面的 process_response
'''
```

**Django会在调用视图函数之前调用process_view方法**

##### process_exception（异常中间件）

process_exception(self, request, exception)

该方法两个参数:

- 一个HttpRequest对象

- 一个exception是视图函数异常产生的Exception对象。

**这个方法只有在视图函数中出现异常了才执行，它返回的值可以是一个None也可以是一个HttpResponse对象。如果是HttpResponse对象，Django将调用模板和中间件中的process_response方法，并返回给浏览器，否则将默认处理异常。如果返回一个None，则交给下一个中间件的process_exception方法来处理异常。它的执行顺序也是按照中间件注册顺序的倒序执行。**

给MD1和MD2添加上这个方法：

```python
from django.utils.deprecation import MiddlewareMixin


class MD1(MiddlewareMixin):

    def process_request(self, request):
        print("MD1里面的 process_request")

    def process_response(self, request, response):
        print("MD1里面的 process_response")
        return response

    def process_view(self, request, view_func, view_args, view_kwargs):
        print("-" * 80)
        print("MD1 中的process_view")
        print(view_func, view_func.__name__)

    def process_exception(self, request, exception):
        print(exception)
        print("MD1 中的process_exception")
        return HttpResponse(str(exception))  # 返回一个响应对象


class MD2(MiddlewareMixin):
    def process_request(self, request):
        print("MD2里面的 process_request")
        pass

    def process_response(self, request, response):
        print("MD2里面的 process_response")
        return response

    def process_view(self, request, view_func, view_args, view_kwargs):
        print("-" * 80)
        print("MD2 中的process_view")
        print(view_func, view_func.__name__)

    def process_exception(self, request, exception):
        print(exception)
        print("MD2 中的process_exception")
"""
请求访问结果：
MD1里面的 process_request
MD2里面的 process_request
--------------------------------------------------------------------------------
MD1 中的process_view
<function index at 0x0000022C09727488> index
--------------------------------------------------------------------------------
MD2 中的process_view
<function index at 0x0000022C09727488> index
app01 中的 index视图
呵呵    #这个就是视图触发异常时给我们返回的数据
MD1 中的process_exception
MD2里面的 process_response
MD1里面的 process_response
"""
```

小结：

- process_request和process_view是正向执行；process_exception和process_response是逆向执行。
- 他们都可以返回None和HttpRequest或者HttpResponse，返回None的时候表示没有问题，继续向下执行，否则就是我们触发的事件的发生。

##### process_template_response（模板中间件）

用的比较少，这里不做讲解。

### 中间件的执行流程

第一种理解：

\1. 应突时请求中间件，处理传入请求。如果请求中间件方法process_request返回的response非空，则终止处理过程，执行步骤7。

\2. url匹配，查找视图函数。

3，应用视图中间件，处理传入请求视图与视图参数。如果视图中间件方法process_view返回的response非空，则终止处理过程，执行步骤7。

4，调用视图函数。

5， 如果视图函数抛出异常 ，应用异常中间件，处理传入请求与异常。如果异常中间件方法process_exception回的response非空，则终止处理过程。无论是否终止过程，都会跳到步骤7.

6， 如果response支持延迟渲染，应用模板中间件。执行步骤7。

7，应用响应中间件，处理传入请求与中间件返回的response。

图示：

![](http://9017499461.linshutu.top/%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B4.png)

第二种理解：

请求到达中间件之后，先按照正序执行每个注册中间件process_request方法，process_request方法返回的值是None，就依次执行，如果返回的值是HttpResponse对象，不再执行后面的process_request方法，而是执行当前对应中间件的process_response方法，将HttpResponse对象返回给浏览器。也就是说：如果MIDDLEWARE中注册了6个中间件，执行过程中，第3个中间件返回了一个HttpResponse对象，那么第4,5,6中间件的process_request和process_response方法都不执行，顺序执行3,2,1中间件的process_response方法。

![](http://9017499461.linshutu.top/%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B1.png)

process_request方法都执行完后，匹配路由，找到要执行的视图函数，先不执行视图函数，先执行中间件中的process_view方法process_view方法返回None，继续按顺序执行，所有process_view方法执行完后执行视图函数。加入中间件3 的process_view方法返回了HttpResponse对象，则4,5,6的process_view以及视图函数都不执行，直接从最后一个中间件，也就是中间件6的process_response方法开始倒序执行。

![](http://9017499461.linshutu.top/%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B2.png)

process_template_response和process_exception两个方法的触发是有条件的，执行顺序也是倒序。总结所有的执行流程如下：

![](http://9017499461.linshutu.top/%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B3.png)

### 中间件应用场景

- 做IP访问频率限制

  某些IP访问服务器的频率过高，进行拦截，比如限制每分钟不能超过20次。

- url过滤

  - 如果用户访问的是login视图（放过）

  - 如果访问其他视图，需要检测是不是有session认证，已经有了放行，没有返回login，这样就省得在多个视图函数上写装饰器了！

### 中间件版登陆认证

url.py

```python
from django.conf.url import url
from app01 import views

urlpatterents = [
    url(r"^index/$",views.index),
    url(r"^login/$",views.login,name="login"),
]
```

views.py

```python
from django.shortcuts import render,redirect,HttpResponse

def login(request):
    if request.method == "POST":
        user = request.POST.get("user")
        pwd = request.POST.get("ped")
        if user == "pl" and pwd == "123":
            #设置session
            request.session["user"] = user
            #获取跳到登陆页面之前的URL
            next_url = request.GET.get("next")
            if next_url:
                return redirect(next_url)
            else:
                return redirect("/index/")
    return render(request,'login.html')
def index(request):
    return HttpResponse("This is index")
```

login.html

```html
html<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="x-ua-compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>登录页面</title>
</head>
	<body>
        <form action="{% url "login" %}">
            <p>
                <label for="user">用户名：</label>
                <input type="text" name="user" id="user">
            </p>
                   <p>
                <label for="pwd">密码：</label>
                <input type="text" name="pwd" id="pwd">
            </p>
            <input type="submit" value="登陆">
        </form>
    </body>
</html>
```

middlewares.py

```python
class AuthMD(MiddlewareMixin):
    white_list = ['/login/',]  #白名单，通过中间件的策略
    black_list = ['/black/',]  #黑名单，阻止url通过
    
    def process_request(self,request):
        from dajngo.shortcuts import redirect,HttpResponse
        next_url = request.path_info
        if next_url in self.white_list or request.session.get("user"):
            return 
        elif next_url in self.black_list:
            return HttpResponse("This is a illegal URL")
        else:
            return rediect('/login/?next={}'.format(next_url))
```

在settings.py中注册

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'middlewares.AuthMD',
]
```

　　　　AuthMD中间件注册后，所有的请求都要走AuthMD的process_request方法。

　　　　访问的URL在白名单内或者session中有user用户名，则不做阻拦走正常流程；

　　　　如果URL在黑名单中，则返回This is an illegal URL的字符串；

　　　　正常的URL但是需要登录后访问，让浏览器跳转到登录页面。