---
title: 第五部分——Django
id: 5
date: 2019-11-15 20:00:00
tags: 小绿本
toc: true
comment: true
---

### 1.简述http协议以及常用的请求头

```
User-Agent：标识浏览器
Content_Type：用来标记请求体的数据的格式，服务端根据这个对数据进行解析
Accept：指定客户端能够接收的内容类型
Accept-Encoding:指定浏览器可以支持的web服务器返回内容压缩编码类型
Accept-Language:浏览器可接受的语言
Content-Length:请求的内容长度
Date：请求发送的日期和时间
```

### 2.列举常见请求方法：

```
GET
POST
```

### 3.列举常见的状态码

```
200：请求成功
301：永久重定向
302：临时重定向
403：没有通过跨站请求伪造
404：请求的页面不存在
500：服务器错误
```

### 4.简述websocket协议及实现原理

```
协议：WebSocket 是一种标准协议，用于在客户端和服务端之间进行双向数据传输。但它跟 HTTP 没什么关系，它是基于 TCP 的一种独立实现。用来弥补http协议在持久通信能力上的不足。

原理：WebSocket是HTML5下一种新的协议。它实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯的目的。它与HTTP一样通过已建立的TCP连接来传输数据，但是它和HTTP最大不同是：
WebSocket是一种双向通信协议。在建立连接后，WebSocket服务器端和客户端都能主动向对方发送或接收数据，就像Socket一样；
WebSocket需要像TCP一样，先建立连接，连接成功后才能相互通信。

相比HTTP长连接，WebSocket有以下特点：
1，是真正的全双工方式，建立连接后客户端与服务器端是完全平等的，可以互相主动请求。
2，Websocket协议通过第一个request建立了TCP连接之后，之后交换的数据都不需要发送 HTTP header就能交换数据
3，此外还有 multiplexing、不同的URL可以复用同一个WebSocket连接等功能

Wesocket协议的优点：
Websocket协议一旦建立后，互相沟通所消耗的请求头是很小的
服务端可以向客户端推送消息了
Wesocket协议的缺点：
少部分浏览器不支持，浏览器支持的成都与方式有区别

Wesocket协议的应用场景：
即时聊天通信
多玩家游戏
在线协同编辑
实时数据流的拉取与推送
体育实况
实时地图位置
```

### 5.Python web开发中，跨域问题的解决思路是什么

```
首先，什么是跨域？
同源策略/SOP（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击。

SOP要求两个通讯地址的协议、域名、端口号必须相同，否则两个地址的通讯将被浏览器视为不安全的，并被block下来。比如“http页面”和“https页面”属于不同协议；“qq.com”、“www.qq.com”、“a.qq.com”都属于不同域名（或主机）；“a.com”和“a.com:8000”属于不同端口号。这三种情况常规都是无法直接进行通讯的。

解决办法：
目前业界流行的解决方案有三种：服务器代理、JSONP、CORS（不展开）
```

### 6.简述http的缓存机制

```
HTTP的缓存可以分为两大类：强制缓存和协商缓存。强制缓存的优先级高于比较缓存。
强制缓存（状态码200）：服务端通知浏览器一个缓存时间，在缓存期间内，下次请求，直接使用浏览器中缓存的数据，不在时间内，执行比较缓存策略。
比较缓存（状态码：304）：对于 比较缓存而言，将缓存信息中的Etag（浏览器当前资源在服务器的唯一标识）和Last-Modified（浏览器资源的最后修改时间）通过请求发送给服务器，由服务器校验，比较成功，返回304状态，浏览器直接使用缓存。
```

![](http://9017499461.linshutu.top/Http%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6.webp)

### 7.谈谈你知道的python web框架：

```
Django
flask
Tornado
Twisted
```

### 8.django中model的SulgFiled类型字段有什么用途

```
slug是一个新闻行业的术语。一个slug就是一个某种东西的简短标签，包含字母、数字、下划线或者连接线，通常用于URLs中。可以设置max_length参数，默认为50。
```

### 9.django常见的线上部署方式有哪几种

```
Nginx+uwsgi+Django+mysql（目前我用到的）
```

### 10.django中使用memcached作为缓存的具体方法？优缺点说明？

```
1，settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211', #多个使用列表
    }
}

2，views.py
from django.http import  HttpResponse
# 使用cache来操作memcached
from django.core.cache import cache

# Create your views here.
def index(request):
    cache.set('username','zhiliao',12000)
    username = cache.get('username')
    print(username)
    return HttpResponse('index')
我认为：
优点：对于一些访问量高的文件放在缓存中，可以提高访问的效率
缺点：缓存的缺点即使数据断电丢失，这点memcached没有好的办法处理
```

memcached的具体操作：https://blog.csdn.net/xujin0/article/details/84311862

<!----more---->

### 11.django、flask、tornado框架的比较？

```
-django，大而全的框架它的内部组件比较多，内部提供：ORM、Admin、中间件、Form、ModelForm、Session、缓存、信号、CSRF；功能也都挺完善的

- flask，微型框架，内部组件就比较少了，但是有很多第三方组件来扩展它，　　比如说有那个wtform（与django的modelform类似，表单验证）、flask-sqlalchemy（操作数据库的）、　　flask-session、flask-migrate、flask-script、blinker可扩展强，第三方组件丰富。所以对他本身来说有那种短小精悍的感觉

- tornado，异步非阻塞。
#是一个轻量级的Web框架，异步非阻塞+内置WebSocket功能。
'目标'：通过一个线程处理N个并发请求(处理IO)。
'内部组件
    #内部自己实现socket
    #路由系统
    #视图
    #模板
　　 #cookie
    #csrf

相同点：
django和flask的共同点就是，他们2个框架都没有写socket，所以他们都是利用第三方模块wsgi。
不同点：
1，但是内部使用的wsgi也是有些不同的：django本身运行起来使用wsgiref，而flask使用werkzeug wsgi,
2，还有一个区别就是他们的请求管理不太一样：django是通过将请求封装成request对象，再通过参数传递，而flask是通过上下文管理机制

另一种说法：
1. Django 、Flask、Tornado的对比
答案:
    1.Django走的是大而全的方向,开发效率高。它的MTV框架,自带的ORM,admin后台管理,自带的sqlite数据库和开发测试用的服务器
    给开发者提高了超高的开发效率
    2.Flask是轻量级的框架,自由,灵活,可扩展性很强,核心基于Werkzeug WSGI工具和jinja2模板引擎
    3.Tornado走的是少而精的方向,性能优越。它最出名的是异步非阻塞的设计方式
    Tornado的两大核心模块：
        1.iostraem：对非阻塞式的socket进行简单的封装
        2.ioloop：对I/O多路复用的封装，它实现了一个单例
```

### 12.什么是wsgi？

```
是web服务网关接口，是一套协议。
是通过以下模块实现了wsgi协议：
    - wsgiref
    - werkzurg
    - uwsgi   关于部署
以上模块本质：编写socket服务端，用于监听请求，当有请求到来，则将请求数据进行封装，然后交给web框架处理。
```

### 13.django请求的生命周期？

```
用户请求进来先走到  wsgi   然后将请求交给  jango的中间件   穿过django中间件（方法是process_request）  接着就是   路由匹配   路由匹配成功之后就执行相应的    视图函数   在视图函数中可以调用orm做数据库操作  再从模板路径   将模板拿到   然后在后台进行模板渲染   模板渲染完成之后就变成一个字符串     再把这个字符串经过所有中间件（方法：process_response）  和wsgi 返回给用户
```

![img](https://images2018.cnblogs.com/blog/1258691/201806/1258691-20180601114444453-1347550743.png)



### 14.列举django的内置组件？

```
form 组件
- 对用户请求的数据进行校验
- 生成HTML标签

PS：
- form对象是一个可迭代对象。
- 问题：choice的数据如果从数据库获取可能会造成数据无法实时更新
        - 重写构造方法，在构造方法中重新去数据库获取值。
        - ModelChoiceField字段
            from django.forms import Form
            from django.forms import fields
            from django.forms.models import ModelChoiceField
            class UserForm(Form):
                name = fields.CharField(label='用户名',max_length=32)
                email = fields.EmailField(label='邮箱')
                ut_id = ModelChoiceField(queryset=models.UserType.objects.all())    
            
            依赖：
                class UserType(models.Model):
                    title = models.CharField(max_length=32)

                    def __str__(self):
                        return self.title
```

```
信号、
django的信号其实就是django内部为开发者预留的一些自定制功能的钩子。
只要在某个信号中注册了函数，那么django内部执行的过程中就会自动触发注册在信号中的函数。
如： 
pre_init # django的modal执行其构造方法前，自动触发
post_init # django的modal执行其构造方法后，自动触发
pre_save # django的modal对象保存前，自动触发
post_save # django的modal对象保存后，自动触发
场景:
在数据库某些表中添加数据时，可以进行日志记录。


CSRF、
目标：防止用户直接向服务端发起POST请求。对所有的post请求做验证/ 将jango生成的一串字符串发送给我们，一种是从请求体发过来，一种是放在隐藏的标签里面用的是process_view　
方案：先发送GET请求时，将token保存到：cookie、Form表单中（隐藏的input标签），以后再发送请求时只要携带过来即可。 ContentType contenttype是django的一个组件（app），为我们找到django程序中所有app中的所有表并添加到记录中。 可以使用他再加上表中的两个字段实现：一张表和N张表创建FK关系。 - 字段：表名称 - 字段：数据行ID 应用：路飞表结构优惠券和专题课和学位课关联。javascript:void(0);)
```

```
中间件
对所有的请求进行批量处理，在视图函数执行前后进行自定义操作。应用：用户登录校验问题：为甚么不使用装饰器？如果不使用中间件，就需要给每个视图函数添加装饰器，太繁琐
权限组件:
用户登录后，将权限放到session中，然后再每次请求进来在中间件里，根据当前的url去session中匹配，判断当前用户是否有权限访问当前url,有权限就继续访问，没有就返回，
 （检查的东西就可以放到中间件中进行统一处理）在process_request方法里面做的，
　我们的中间件是放在session后面，因为中间件需要到session里面取数据
```

```
session
cookie与session区别
（a）cookie是保存在浏览器端的键值对，而session是保存的服务器端的键值对，但是依赖cookie。（也可以不依赖cookie，可以放在url，或请求头但是cookie比较方便）
（b）以登录为例，cookie为通过登录成功后，设置明文的键值对，并将键值对发送客户端存，明文信息可能存在泄漏，不安全；　　session则是生成随机字符串，发给用户，并写到浏览器的cookie中，同时服务器自己也会保存一份。
（c）在登录验证时，cookie：根据浏览器发送请求时附带的cookie的键值对进行判断，如果存在，则验证通过；　　session：在请求用户的cookie中获取随机字符串，根据随机字符串在session中获取其对应的值进行验证
```

```
cors跨域（场景：前后端分离时，本地测试开发时使用）
如果网站之间存在跨域，域名不同，端口不同会导致出现跨域，但凡出现跨域，浏览器就会出现同源策略的限制
解决：在我们的服务端给我们响应数据，加上响应头---> 在中间件加的
```

```
缓存/   常用的数据放在缓存里面，就不用走视图函数，请求进来通过所有的process_request,会到缓存里面查数据，有就直接拿，　　　　　　　　没有就走视图函数　　　　　　关键点：1：执行完所有的process_request才去缓存取数据　　　　　　　　　　2：执行完所有的process_response才将数据放到缓存
关于缓存问题
1:为什么放在最后一个process_request才去缓存
因为需要验证完用户的请求，才能返回数据

2:什么时候将数据放到缓存中
第一次走中间件，缓存没有数据，会走视图函数，取数据库里面取数据，
当走完process_response,才将数据放到缓存里，因为，走process_response的时候可能给我们的响应加处理
```

```
为什么使用缓存
将常用且不太频繁修改的数据放入缓存。
以后用户再来访问，先去缓存查看是否存在，如果有就返回
否则，去数据库中获取并返回给用户（再加入到缓存，以便下次访问）
```

### 15.列举django中间件的5个方法？以及django中间件的应用场景？

```
process_request(self,request)  先走request 通过路由匹配返回
process_view(self, request, callback, callback_args, callback_kwargs) 再返回执行view
process_template_response(self,request,response)   当视图函数的返回值
process_exception(self, request, exception)  当视图函数的返回值对象中有render方法时，该方法才会被调用
process_response(self, request, response)
```

执行流程

![img](https://images2018.cnblogs.com/blog/1258691/201806/1258691-20180601133706615-2118245019.png)

### 16.简述什么是FBV和CBV？

```
FBV 基于函数
# FBV 写法
# urls.py
 url(r'^login/$',views.login, name="login"),

# views.py
def login(request):
    if request.method == "POST":
        print(request.POST)

    return render(request,"login.html")

# HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>登录页面</title>
</head>
<body>
<form action="{% url 'login' %}" method="post" enctype="multipart/form-data">
    <input type="text" name="user2">
    <input type="file" name="file">
    <input type="submit" value="提交">


</form>
</body>
</html>

CBV 基于类
# urls.py    
url(r'^login/$',views.Login.as_view(), name="login"), 

# views.py
from django.views import View
class Login(View):   # 类首字母大写
    def get(self,request):
        return render(request,"login.html")
    def post(self,request):
        print(request.POST)
        return HttpResponse("OK")

加装饰器

=================================
class IndexView(View):
    
    # 如果是crsf相关，必须放在此处
    def dispach(self,request):
        # 通过反射执行post/get 
    
    @method_decoretor(装饰器函数)
    def get(self,request):
        pass
        
    def post(self,request):
        pass 
路由：IndexView.as_view()
```

### 17.FBV与CBV的区别

```
- 没什么区别，因为他们的本质都是函数。CBV的.as_view()返回的view函数，view函数中调用类的dispatch方法，
在dispatch方法中通过反射执行get/post/delete/put等方法。D

非要说区别的话：
- CBV比较简洁，GET/POST等业务功能分别放在不同get/post函数中。FBV自己做判断进行区分。
```

### 18.django的request对象是在什么时候创建的？

```
当请求一个页面时, Django会建立一个包含请求元数据的 HttpRequest 对象. 当Django 加载对应的视图时, HttpRequest对象将作为视图函数的第一个参数. 每个视图会返回一个HttpResponse对象.
```

### 19.如何给CBV的程序添加装饰器？

```
添加装饰器
方式一：
from django.views import View
from django.utils.decorators import method_decorator  ---> 需要引入memethod_decorator

def auth(func):
    def inner(*args,**kwargs):
        return func(*args,**kwargs)
    return inner

class UserView(View):
    @method_decorator(auth)
    def get(self,request,*args,**kwargs):
        return HttpResponse('...')    

方式二：
- csrf的装饰器要加到dispath前面
from django.views import View
from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt,csrf_protect   ---> 需要引入 csrf_exempt


class UserView(View):
    @method_decorator(csrf_exempt)
    def dispatch(self, request, *args, **kwargs):
        return HttpResponse('...')

或者：
from django.views import View
from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt,csrf_protect

@method_decorator(csrf_exempt,name='dispatch')  --->  指定名字
class UserView(View):
    def dispatch(self, request, *args, **kwargs):
        return HttpResponse('...')
```

### 20.列举django orm 中所有的方法（QuerySet对象的所有方法）

```
返回QuerySet对象的方法有：
      all()
      filter()
      exclude()
      order_by()
      reverse()
      distinct()
  特殊的QuerySet：
      values()       返回一个可迭代的字典序列
      values_list() 返回一个可迭代的元组序列
  返回具体对象的：
      get()
      first()
      last()
  返回布尔值的方法有：
      exists()
  返回数字的方法有：
      count() 
```

### 21.only和defer的区别？

```
 def defer(self, *fields):
    models.UserInfo.objects.defer('username','id')
    或
    models.UserInfo.objects.filter(...).defer('username','id')
    #映射中排除某列数据

 def only(self, *fields):
    #仅取某个表中的数据
     models.UserInfo.objects.only('username','id')
     或
     models.UserInfo.objects.filter(...).only('username','id')
```

### 22.select_related和prefetch_related的区别？

```
# 他俩都用于连表查询，减少SQL查询次数
\select_related
select_related主要针一对一和多对一关系进行优化，通过多表join关联查询，一次性获得所有数据，
存放在内存中，但如果关联的表太多，会严重影响数据库性能。
def index(request):
    obj = Book.objects.all().select_related("publisher")
    return render(request, "index.html", locals())
\prefetch_related
prefetch_related是通过分表，先获取各个表的数据，存放在内存中，然后通过Python处理他们之间的关联。
def index(request):
    obj = Book.objects.all().prefetch_related("publisher")
    return render(request, "index.html", locals())
```

```
def select_related(self, *fields)
     性能相关：表之间进行join连表操作，一次性获取关联的数据。
     model.tb.objects.all().select_related()
     model.tb.objects.all().select_related('外键字段')
     model.tb.objects.all().select_related('外键字段__外键字段')

def prefetch_related(self, *lookups)
    性能相关：多表连表操作时速度会慢，使用其执行多次SQL查询在Python代码中实现连表操作。
            # 获取所有用户表
            # 获取用户类型表where id in (用户表中的查到的所有用户ID)
            models.UserInfo.objects.prefetch_related('外键字段')

            from django.db.models import Count, Case, When, IntegerField
            Article.objects.annotate(
                numviews=Count(Case(
                    When(readership__what_time__lt=treshold, then=1),
                    output_field=CharField(),
                ))
            )

            students = Student.objects.all().annotate(num_excused_absences=models.Sum(
                models.Case(
                    models.When(absence__type='Excused', then=1),
                default=0,
                output_field=models.IntegerField()
            )))
```

```
# 1次SQL
# select * from userinfo
objs = UserInfo.obejcts.all()
for item in objs:
    print(item.name)
    
# n+1次SQL
# select * from userinfo
objs = UserInfo.obejcts.all()
for item in objs:
    # select * from usertype where id = item.id 
    print(item.name,item.ut.title)
    
select_related（）
# 1次SQL
# select * from userinfo inner join usertype on userinfo.ut_id = usertype.id 
objs = UserInfo.obejcts.all().select_related('ut')  连表查询
for item in objs:
    print(item.name,item.ut.title)
            
.prefetch_related()
    # select * from userinfo where id <= 8
    # 计算：[1,2]
    # select * from usertype where id in [1,2]
    objs = UserInfo.obejcts.filter(id__lte=8).prefetch_related('ut')
    for obj in objs:
        print(obj.name,obj.ut.title)
```

### 23.filter和exclude的区别？

```
  def filter(self, *args, **kwargs)
      # 条件查询(符合条件)
       # 查出符合条件
      # 条件可以是：参数，字典，Q

  def exclude(self, *args, **kwargs)
      # 条件查询(排除条件)
      # 排除不想要的
      # 条件可以是：参数，字典，Q
```

### 24.列举django orm中三种能写sql语句的方法。

```
原生SQL --->  connection
from django.db import connection, connections
cursor = connection.cursor()  # cursor = connections['default'].cursor()
cursor.execute("""SELECT * from auth_user where id = %s""", [1])
row = cursor.fetchone() # fetchall()/fetchmany(..)
靠近原生SQL-->extra\raw
extra
- extra
    def extra(self, select=None, where=None, params=None, tables=None, order_by=None, 
select_params=None)
        # 构造额外的查询条件或者映射，如：子查询
        Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"},
 select_params=(1,))
        Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
        Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
        Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, s
elect_params=(1,), order_by=['-nid'])

- raw 
def raw(self, raw_query, params=None, translations=None, using=None):
    # 执行原生SQL
    models.UserInfo.objects.raw('select * from userinfo')
    # 如果SQL是其他表时，必须将名字设置为当前UserInfo对象的主键列名
    models.UserInfo.objects.raw('select id as nid,name as title  from 其他表')
    # 为原生SQL设置参数
    models.UserInfo.objects.raw('select id as nid from userinfo where nid>%s', params=[12,])
    # 将获取的到列名转换为指定列名
    name_map = {'first': 'first_name', 'last': 'last_name', 'bd': 'birth_date', 'pk': 'id'}
    Person.objects.raw('SELECT * FROM some_other_table', translations=name_map)
    # 指定数据库
    models.UserInfo.objects.raw('select * from userinfo', using="default")
```

### 25.django orm 中如何设置读写分离？

```
 方式一：手动使用queryset的using方法
from django.shortcuts import render,HttpResponse
from app01 import models
def index(request):

    models.UserType.objects.using('db1').create(title='普通用户')
　　# 手动指定去某个数据库取数据
    result = models.UserType.objects.all().using('db1')
    print(result)

    return HttpResponse('...')

方式二：写配置文件
class Router1:
　　#  指定到某个数据库取数据
    def db_for_read(self, model, **hints):
        """
        Attempts to read auth models go to auth_db.
        """
        if model._meta.model_name == 'usertype':
            return 'db1'
        else:
            return 'default'
　　　# 指定到某个数据库存数据
    def db_for_write(self, model, **hints):
        """
        Attempts to write auth models go to auth_db.
        """
        return 'default'
再写到配置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    },
    'db1': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
DATABASE_ROUTERS = ['db_router.Router1',]
```

### 26.F和Q的作用?

```
  F:主要用来获取原数据进行计算。
  Django 支持 F() 对象之间以及 F() 对象和常数之间的加减乘除和取模的操作。
  修改操作也可以使用F函数,比如将每件商品的价格都在原价格的基础上增加10
from django.db.models import F
from app01.models import Goods
 
Goods.objects.update(price=F("price")+10)  # 对于goods表中每件商品的价格都在原价格的基础上增加10元F查询专门对对象中某列值的操作，不可使用__双下划线！
Q:用来进行复杂查询    Q查询可以组合使用 “&”, “|” 操作符，当一个操作符是用于两个Q的对象,它产生一个新的Q对象，　　Q对象可以用 “~” 操作符放在前面表示否定，也可允许否定与不否定形式的组合。　　Q对象可以与关键字参数查询一起使用，不过一定要把Q对象放在关键字参数查询的前面。
  Q(条件1) | Q(条件2) 或
  Q(条件1) & Q(条件2) 且
  Q(条件1) & ~
  Q(条件2) 非
```

扩展:

```
11.Django中查询queryset时什么情况下用Q?
    关键字参数查询 - 输入filter()等 - 是“AND”编辑在一起。如果需要执行更复杂的查询（例如，带OR语句的查询），则可以使用。Q objects
    例如需要进行复合条件的查询的SQL语句如下：
    select * from goods where name like '%好看%' or title like '%好看%'; 

    使用Q就可以写成：
    from django.db.models import Q
    obs = Goods.objects.filter(Q(name__contains='好看')|Q(title__contains='好看'))
```



### 27.values和values_list的区别？

```
def values(self, *fields):
    # 获取每行数据为字典格式

def values_list(self, *fields, **kwargs):
    # 获取每行数据为元祖
```

### 28.如何使用django orm批量创建数据？

```
def bulk_create(self, objs, batch_size=None):
    # 批量插入
    # batch_size表示一次插入的个数
    objs = [
        models.DDD(name='r11'),
        models.DDD(name='r22')
    ]
    models.DDD.objects.bulk_create(objs, 10)
```

### 29.django的Form和ModeForm的作用？

```
 - 作用：
      - 对用户请求数据格式进行校验
      - 自动生成HTML标签
  - 区别：
      - Form，字段需要自己手写。
          class Form(Form):
              xx = fields.CharField(.)
              xx = fields.CharField(.)
              xx = fields.CharField(.)
              xx = fields.CharField(.)
      - ModelForm，可以通过Meta进行定义
          class MForm(ModelForm):
              class Meta:
                  fields = "__all__"
                  model = UserInfo                            
  - 应用：只要是客户端向服务端发送表单数据时，都可以进行使用，如：用户登录注册
```

### 30.django的Form组件中，如果字段中包含choices参数，请使用两种方式实现数据源实时更新。

```
 方式一:重写构造方法，在构造方法中重新去数据库获取值
  class UserForm(Form):
      name = fields.CharField(label='用户名',max_length=32)
      email = fields.EmailField(label='邮箱')
      ut_id = fields.ChoiceField(
          # choices=[(1,'普通用户'),(2,'IP用户')]
          choices=[]
      )

      def __init__(self,*args,**kwargs):
          super(UserForm,self).__init__(*args,**kwargs)

          self.fields['ut_id'].choices = models.UserType.objects.all().values_list('id','title')
  方式二: ModelChoiceField字段
  from django.forms import Form
  from django.forms import fields
  from django.forms.models import ModelChoiceField
  class UserForm(Form):
      name = fields.CharField(label='用户名',max_length=32)
      email = fields.EmailField(label='邮箱')
      ut_id = ModelChoiceField(queryset=models.UserType.objects.all())    

  依赖：
      class UserType(models.Model):
          title = models.CharField(max_length=32)

          def __str__(self):
              return self.title
```

### 31.django的Model中的ForeignKey字段中的on_delete参数有什么作用？

```
在django2.0后，定义外键和一对一关系的时候需要加on_delete选项，此参数为了避免两个表里的数据不一致问题，不然会报错：

TypeError: __init__() missing 1 required positional argument: 'on_delete'

 举例说明：

user=models.OneToOneField(User)

owner=models.ForeignKey(UserProfile)

需要改成：

user=models.OneToOneField(User,on_delete=models.CASCADE)          --在老版本这个参数（models.CASCADE）是默认值

owner=models.ForeignKey(UserProfile,on_delete=models.CASCADE)    --在老版本这个参数（models.CASCADE）是默认值
参数说明：

on_delete有CASCADE、PROTECT、SET_NULL、SET_DEFAULT、SET()五个可选择的值

CASCADE：此值设置，是级联删除。
PROTECT：此值设置，是会报完整性错误。
SET_NULL：此值设置，会把外键设置为null，前提是允许为null。
SET_DEFAULT：此值设置，会把设置为外键的默认值。
SET()：此值设置，会调用外面的值，可以是一个函数。
一般情况下使用CASCADE就可以了。
```

### 32.django中csrf的实现机制？

```
目的：防止用户直接向服务端发起POST请求
- 用户先发送GET获取csrf token: Form表单中一个隐藏的标签 + token
- 发起POST请求时，需要携带之前发送给用户的csrf token；
- 在中间件的process_view方法中进行校验。

在html中添加{%csrf_token%}标签

第一步：django第一次响应来自某个客户端的请求时,后端随机产生一个token值，把这个token保存在SESSION状态中;同时,后端把这个token放到cookie中交给前端	  页面；
    第二步：下次前端需要发起请求（比如发帖）的时候把这个token值加入到请求数据或者头信息中,一起传给后端；Cookies:{csrftoken:xxxxx}
    第三步：后端校验前端请求带过来的token和SESSION里的token是否一致。
```

### 33.django如何实现websocket？

```
django中可以通过channel实现websocket
```

### 34.基于django使用ajax发送post请求时，都可以使用哪种方法携带csrf token？

```
//方式一给每个ajax都加上上请求头
    function Do1(){
        $.ajax({
            url:"/index/",
            data:{id:1},
            type:'POST',
　　　　　　　data:{csrfmiddlewaretoken:'{{ csrf_token }}',name:'alex'}
            success:function(data){
                console.log(data);
            }
        });
    }

方式二：需要先下载jQuery-cookie，才能去cookie中获取token
        function Do1(){
        $.ajax({
            url:"/index/",
            data:{id:1},
            type:'POST',
            headers:{
              'X-CSRFToken':$.cookie('csrftoken')  // 去cookie中获取
            },
            success:function(data){
                console.log(data);
            }
        });
    }

方式三：搞个函数ajaxSetup，当有多的ajax请求，即会执行这个函数
        $.ajaxSetup({
           beforeSend:function (xhr,settings) {
               xhr.setRequestHeader("X-CSRFToken",$.cookie('csrftoken'))
           } 
        });

函数版本
<body>
<input type="button" onclick="Do1();"  value="Do it"/>
<input type="button" onclick="Do2();"  value="Do it"/>
<input type="button" onclick="Do3();"  value="Do it"/>

<script src="/static/jquery-3.3.1.min.js"></script>
<script src="/static/jquery.cookie.js"></script>
<script>
    $.ajaxSetup({
        beforeSend: function(xhr, settings) {
            xhr.setRequestHeader("X-CSRFToken", $.cookie('csrftoken'));
        }
    });

     function Do1(){
        $.ajax({
            url:"/index/",
            data:{id:1},
            type:'POST',
            success:function(data){
                console.log(data);
            }
        });
    }

     function Do2(){
        $.ajax({
            url:"/index/",
            data:{id:1},
            type:'POST',
            success:function(data){
                console.log(data);
            }
        });
    }

     function Do3(){
        $.ajax({
            url:"/index/",
            data:{id:1},
            type:'POST',
            success:function(data){
                console.log(data);
            }
        });
    }
</script>
</body>
```

### 35.django中如何实现orm表中添加数据时创建一条日志记录。

给信号注册函数

```
使用django的信号机制，可以在添加、删除数据前后设置日志记录
pre_init  # Django中的model对象执行其构造方法前,自动触发
post_init  # Django中的model对象执行其构造方法后,自动触发
pre_save  # Django中的model对象保存前,自动触发
post_save  # Django中的model对象保存后,自动触发
pre_delete  # Django中的model对象删除前,自动触发
post_delete  # Django中的model对象删除后,自动触发
```

### 36.django缓存如何设置？

```
jango中提供了6种缓存方式：
　　开发调试（不加缓存）
　　内存
　　文件
　　数据库
　　Memcache缓存（python-memcached模块）
　　Memcache缓存（pylibmc模块）

安装第三方组件支持redis：
　　django-redis组件


设置缓存
# 全站缓存（中间件）
MIDDLEWARE_CLASSES = (
    ‘django.middleware.cache.UpdateCacheMiddleware’, #第一
    'django.middleware.common.CommonMiddleware',
    ‘django.middleware.cache.FetchFromCacheMiddleware’, #最后
)
 
# 视图缓存
from django.views.decorators.cache import cache_page
import time
  
@cache_page(15)          #超时时间为15秒
def index(request):
   t=time.time()      #获取当前时间
   return render(request,"index.html",locals())
 
# 模板缓存
{% load cache %}
 <h3 style="color: green">不缓存:-----{{ t }}</h3>
  
{% cache 2 'name' %} # 存的key
 <h3>缓存:-----:{{ t }}</h3>
{% endcache %}
```

### 37.django的缓存能使用redis吗？如果可以的话，如何配置？

```
  pip install django-redis  
  apt-get install redis-serv

在setting添加配置文件
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache", # 缓存类型
        "LOCATION": "127.0.0.1:6379", # ip端口
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",  #
            "CONNECTION_POOL_KWARGS": {"max_connections": 100} # 连接池最大连接数
            # "PASSWORD": "密码",
        }
    }
}


使用
from django.shortcuts import render,HttpResponse
from django_redis import get_redis_connection
  
def index(request):
# 根据名字去连接池中获取连接
conn = get_redis_connection("default")
    conn.hset('n1','k1','v1') # 存数据
    return HttpResponse('...')
```

### 38.django路由系统中name的作用？

```
反向解析路由字符串
路由系统中name的作用：反向解析
url(r'^home', views.home, name='home')在模板中使用：{ % url 'home' %}在视图中使用：reverse(“home”）
```

### 39.django的模板中filter和simple_tag的区别？

```
filter : 类似管道,只能接受两个参数第一个参数是|前的数据

simple_tag : 类似函数1、模板继承：{% extends 'layouts.html' %}2、自定义方法
  	 'filter'：只能传递两个参数，可以在if、for语句中使用
  	 'simple_tag'：可以无线传参，不能在if for中使用
  	 'inclusion_tags'：可以使用模板和后端数据
3、防xss攻击： '|safe'、'mark_safe'
```

### 40.django-debug-toolbar的作用？

```
一、查看访问的速度、数据库的行为、cache命中等信息。 
二、尤其在Mysql访问等的分析上大有用处(sql查询速度)
```

https://blog.csdn.net/weixin_39198406/article/details/78821677

### 41.django中如何实现单元测试？

```
对于每一个测试方法都会将setUp()和tearDown()方法执行一遍
会单独新建一个测试数据库来进行数据库的操作方面的测试，默认在测试完成后销毁。
在测试方法中对数据库进行增删操作，最后都会被清除。也就是说，在test_add中插入的数据，在test_add测试结束后插入的数据会被清除。
django单元测试时为了模拟生产环境，会修改settings中的变量，例如, 把DEBUG变量修改为True, 把ALLOWED_HOSTS修改为[*]。
```

### 42.解释orm中 db first 和 code first的含义？

```
db first: 先创建数据库，再更新表模型
code first：先写表模型，再更新数据库
```

https://www.cnblogs.com/jassin-du/p/8988897.html

### 43.django中如何根据数据库表生成model中的类？

```
1、修改seting文件，在setting里面设置要连接的数据库类型和名称、地址
2、运行下面代码可以自动生成models模型文件
       - python manage.py inspectdb
3、创建一个app执行下下面代码：
       - python manage.py inspectdb > app/models.py 
```

### 44.使用orm和原生sql的优缺点？

```
SQL：
# 优点：
执行速度快
# 缺点：
编写复杂，开发效率不高
---------------------------------------------------------------------------
ORM：
# 优点：
让用户不再写SQL语句，提高开发效率
可以很方便地引入数据缓存之类的附加功能
# 缺点：
在处理多表联查、where条件复杂查询时，ORM的语法会变得复杂。
没有原生SQL速度快
```

### 45.简述MVC和MTV

```
16.MVC的核心思想 
程序解耦，让不同的代码块之间降低耦合，增强代码的可扩展和可移植性，实现向后兼容。

MVC软件系统分为三个基本部分：模型(Model)、视图(View)和控制器(Controller)
    Model：负责业务对象与数据库的映射(ORM)
    View：负责与用户的交互
    Control：接受用户的输入调用模型和视图完成用户的请求
Django框架的MTV设计模式借鉴了MVC框架的思想,三部分为：Model、Template和View
    Model(模型)：负责业务对象与数据库的对象(ORM)
    Template(模版)：负责如何把页面展示给用户
    View(视图)：负责业务逻辑，并在适当的时候调用Model和Template
    此外,Django还有一个urls分发器,
    它将一个个URL的页面请求分发给不同的view处理,view再调用相应的Model和Template
```

### 46.django的contenttype组件的作用？

```
contenttype是django的一个组件(app)，它可以将django下所有app下的表记录下来
可以使用他再加上表中的两个字段,实现一张表和N张表动态创建FK关系。
   - 字段：表名称
   - 字段：数据行ID
应用：路飞表结构优惠券和专题课和学位课关联
```

### 47.谈谈你对restfull 规范的认识？

```
restful其实就是一套编写接口的'协议'，规定如何编写以及如何设置返回值、状态码等信息。
# 最显著的特点：
# 用restful: 
    给用户一个url，根据method不同在后端做不同的处理
    比如：post创建数据、get获取数据、put和patch修改数据、delete删除数据。
# 不用restful: 
    给调用者很多url，每个url代表一个功能，比如：add_user/delte_user/edit_user/
# 当然，还有协议其他的，比如：
    '版本'来控制让程序有多个版本共存的情况，版本可以放在 url、请求头（accept/自定义）、GET参数
    '状态码'200/300/400/500
    'url中尽量使用名词'restful也可以称为“面向资源编程”
    'api标示'
        api.luffycity.com
        www.luffycity.com/api/
```

### 48.接口的幂等性是什么意思？

```
'一个接口通过1次相同的访问，再对该接口进行N次相同的访问时，对资源不造影响就认为接口具有幂等性。'
    GET，  #第一次获取结果、第二次也是获取结果对资源都不会造成影响，幂等。
    POST， #第一次新增数据，第二次也会再次新增，非幂等。
    PUT，  #第一次更新数据，第二次不会再次更新，幂等。
    PATCH，#第一次更新数据，第二次不会再次更新，非幂等。
    DELTE，#第一次删除数据，第二次不在再删除，幂等。
```

### 49.什么是RPC？

```
'远程过程调用协议'
是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。
进化的顺序: 现有的RPC,然后有的RESTful规范
```

### 50.Http和Https的区别？

```
#Http: 80端口
#https: 443端口
# http信息是明文传输，https则是具有安全性的ssl加密传输协议。
#- 自定义证书 
    - 服务端：创建一对证书
    - 客户端：必须携带证书
#- 购买证书
    - 服务端： 创建一对证书，。。。。
    - 客户端： 去机构获取证书，数据加密后发给咱们的服务单
    - 证书机构:公钥给改机构
```

### 51.为什么要使用django rest framework框架？

```
# 在编写接口时可以不使用django rest framework框架，
# 不使用：也可以做，可以用django的CBV来实现，开发者编写的代码会更多一些。
# 使用：内部帮助我们提供了很多方便的组件，我们通过配置就可以完成相应操作，如：
    '序列化'可以做用户请求数据校验+queryset对象的序列化称为json
    '解析器'获取用户请求数据request.data，会自动根据content-type请求头的不能对数据进行解析
    '分页'将从数据库获取到的数据在页面进行分页显示。
     # 还有其他组件：
         '认证'、'权限'、'访问频率控制 
```

### 52.django rest framework框架中都有那些组件？

```
#- 路由，自动帮助开发者快速为一个视图创建4个url
        www.oldboyedu.com/api/v1/student/$
        www.oldboyedu.com/api/v1/student(?P<format>\w+)$
        www.oldboyedu.com/api/v1/student/(?P<pk>\d+)/$
        www.oldboyedu.com/api/v1/student/(?P<pk>\d+)(?P<format>\w+)$
#- 版本处理
    - 问题：版本都可以放在那里？
            - url
            - GET 
            - 请求头 
#- 认证 
    - 问题：认证流程？
#- 权限 
    - 权限是否可以放在中间件中？以及为什么？
#- 访问频率的控制
    匿名用户可以真正的防止？无法做到真正的访问频率控制，只能把小白拒之门外。
    如果要封IP，使用防火墙来做。
    登录用户可以通过用户名作为唯一标示进行控制，如果有人注册很多账号，则无法防止。
#- 视图
#- 解析器 ，根据Content-Type请求头对请求体中的数据格式进行处理。request.data 
#- 分页
#- 序列化
    - 序列化
        - source
        - 定义方法
    - 请求数据格式校验
#- 渲染器 
```

### 53.django rest framework框架中的视图都可以继承哪些类

```
a. 继承APIView（最原始）但定制性比较强
    这个类属于rest framework中的顶层类，内部帮助我们实现了只是基本功能：认证、权限、频率控制，
但凡是数据库、分页等操作都需要手动去完成，比较原始。
    class GenericAPIView(APIView)
    def post(...):
          pass 

b.继承GenericViewSet（ViewSetMixin，generics.GenericAPIView）
　　首先他的路由就发生变化
    如果继承它之后，路由中的as_view需要填写对应关系
　　在内部也帮助我们提供了一些方便的方法：
　　get_queryset
　　get_object
　　get_serializer
　　get_serializer_class
　　get_serializer_context
　　filter_queryset
注意：要设置queryset字段，否则会抛出断言的异常。

代码
只提供增加功能 只继承GenericViewSet

class TestView(GenericViewSet):
　　serialazer_class = xxx
　　def creat(self,*args,**kwargs):
　　　　pass  # 获取数据并对数据

c. 继承  modelviewset  --> 快速快发
　　　　-ModelViewSet(增删改查全有+数据库操作)
　　　　-mixins.CreateModelMixin（只有增）,GenericViewSet
　　　　-mixins.CreateModelMixin,DestroyModelMixin,GenericViewSet
　　对数据库和分页等操作不用我们在编写，只需要继承相关类即可。
　　
示例：只提供增加功能
class TestView(mixins.CreateModelMixin,GenericViewSet):
　　　　serializer_class = XXXXXXX
*** 
　　modelviewset --> 快速开发，复杂点的genericview、apiview
```

![img](https://images2018.cnblogs.com/blog/1258691/201806/1258691-20180604131035243-475790859.png)

### 54.简述 django rest framework框架的认证流程。

```
- 如何编写？写类并实现authenticators
　　请求进来认证需要编写一个类，类里面有一个authenticators方法，我们可以自定义这个方法，可以定制3类返回值。
　　成功返回元组，返回none为匿名用户，抛出异常为认证失败。

源码流程：请求进来先走dispatch方法，然后封装的request对象会执行user方法，由user触发authenticators认证流程
- 方法中可以定义三种返回值：
    - （user,auth），认证成功
    - None , 匿名用户
    - 异常 ，认证失败
- 流程：
    - dispatch 
    - 再去request中进行认证处理
```

### 55.django rest framework如何实现的用户访问频率控制？ 

```
# 对匿名用户，根据用户IP或代理IP作为标识进行记录，为每个用户在redis中建一个列表
    {
        throttle_1.1.1.1:[1526868876.497521,152686885.497521...]，
        throttle_1.1.1.2:[1526868876.497521,152686885.497521...]，
        throttle_1.1.1.3:[1526868876.497521,152686885.497521...]，
    } 
 每个用户再来访问时，需先去记录中剔除过期记录，再根据列表的长度判断是否可以继续访问。
 '如何封IP'：在防火墙中进行设置
--------------------------------------------------------------------------
# 对注册用户，根据用户名或邮箱进行判断。
    {
        throttle_xxxx1:[1526868876.497521,152686885.497521...]，
        throttle_xxxx2:[1526868876.497521,152686885.497521...]，
        throttle_xxxx3:[1526868876.497521,152686885.497521...]，
    }
每个用户再来访问时，需先去记录中剔除过期记录，再根据列表的长度判断是否可以继续访问。
\如1分钟：40次，列表长度限制在40，超过40则不可访问
```

### 56.简述Django和Flask有什么区别？

```
8.Django和Flask有什么区别？

答案示例:
	Flask
    ·轻量级web框架，默认依赖两个外部库：jinja2和Werkzeug WSGI工具
    ·适用于做小型网站以及web服务的API，开发大型网站无压力，但架构需要自己设计
    ·与关系型数据库的结合不弱于Django，而与非关系型数据库的结合远远优于Django
	Django
    ·重量级web框架，功能齐全，提供一站式解决的思路，能让开发者不用在选择上花费大量时间。
    ·自带ORM(Object-Relational Mapping 对象关系映射)和模板引擎，支持jinja等非官方模板引擎。
    ·自带ORM使Django和关系型数据库耦合度高，如果要使用非关系型数据库，需要使用第三方库
    ·自带数据库管理app
    ·成熟，稳定，开发效率高，相对于Flask，Django的整体封闭性比较好，适合做企业级网站的开发。
    ·python web框架的先驱，第三方库丰富
```

### 57.Django中想验证表单提交是否格式正确需要用到Form中的哪个函数？

```
答案: is_valid()
```

### 58.Flask-WTF是什么，有什么特点？

```
答案: Flask-wtf是一个用于表单处理,校验并提供csrf验证的功能的扩展库 Flask-wtf能把正表单免受CSRF<跨站请求伪造>的攻击
```

### 59.基于django使用ajax发送post请求时，都可以使用哪种方法携带csrf token？

```
    1.后端将csrftoken传到前端，发送post请求时携带这个值发送
    data: {
            csrfmiddlewaretoken: '{{ csrf_token }}'
      },
    2.获取form中隐藏标签的csrftoken值，加入到请求数据中传给后端
    data: {
              csrfmiddlewaretoken:$('[name="csrfmiddlewaretoken"]').val()
         },
    3.cookie中存在csrftoken,将csrftoken值放到请求头中
    headers:{ "X-CSRFtoken":$.cookie("csrftoken")}
```

### 60.列举django的orm的查询方法

外链：https://www.cnblogs.com/lpdeboke/p/11275714.html

### 61.django如何连接多个数据库？

外链：https://www.jianshu.com/p/1d4442b683e6

### 62.django的ORM的懒加载是干嘛的？

外链：https://blog.csdn.net/orangleliu/article/details/57088557

### 63.dj的selete_related、prefetch_related的作用？

```
有外键存在时，可以很好的减少数据库请求的次数,提高性能

select_related通过多表join关联查询,一次性获得所有数据,只执行一次SQL查询

prefetch_related分别查询每个表,然后根据它们之间的关系进行处理,执行两次查询
```

### 64.描述Python GIL的概念， 以及它对python多线程的影响？一个单线程抓取网页的程序，与一个多线程抓取网页的程序哪个性能更高，并解释原因

```
1.GIL，全局解释器锁(global interpreter lock)，它是cpython解析器的特性，不是python的特性 ，它要求线程在执行前，需要获取GIL锁，

2.由于GIL的存在，会影响多线程不能利用多核CPU资源(原因是一个进程只存在一把gil锁，当在执行多个线程时，内部会争抢gil锁，这会造成当某一个线程没有抢到锁的时候会让cpu等待，进而不能合理利用多核cpu资源)，通过多进程方式可利用多个CPU资源

3.线程释放GIL锁的情况：
    1.在IO操作等可能会引起阻塞的system call之前,可以暂时释放GIL,但在执行完毕后,必须重新获取GIL
    2.Python 3x使用计时器（执行时间达到阈值后，当前线程释放GIL）

4.多线程爬取比单线程性能有提升，因为遇到IO阻塞会自动释放GIL锁，这样在线程阻塞情况下，可以执行其他线程中的代码 
```

### 65.GIL

```
1.GIL是什么？
GIL全称Global Interpreter Lock，即全局解释器锁。 作用就是，限制多线程同时执行，保证同一时间内只有一个线程在执行。
GIL并不是Python的特性，它是在实现Python解析器(CPython)时所引入的一个概念。python 与 python解释器是两个概念，切不可混为一谈，也就是说，GIL只存在于使用C语言编写的解释器CPython中。
通俗地说，就是如果你不用Python官方推荐的CPython解释器，而使用其他语言编写的Python解释器（比如  JPython: 运行在Java上的解释器，直接把python代码编译成Java字节码执行 ），就不会有GIL问题。然而因为CPython是大部分环境下默认的Python执行环境。所以在很多人的概念里CPython就是Python，也就想当然的把GIL归结为Python语言的缺陷。所以这里要先明确一点：GIL并不是Python的特性，Python完全可以不依赖于GIL。
 
2.GIL有什么作用？
为了更有效的利用多核处理器的性能，就出现了多线程的编程方式，而随之带来的就是线程间数据的一致性和状态同步的完整性。  python为了利用多核，开始支持多线程，但线程是非独立的，所以同一进程里线程是数据共享，当各个线程访问数据资源时会出现竞状态，即数据可能会同时被多个线程占用，造成数据混乱，这就是线程的不安全。而解决多线程之间数据完整性和状态同步最简单的方式就是加锁。GIL能限制多线程同时执行，保证同一时间内只有一个线程在执行。
 
3.GIL有什么影响？
GIL无疑就是一把全局排他锁。毫无疑问全局锁的存在会对多线程的效率有不小影响。甚至就几乎等于Python是个单线程的程序。
 
4.如何避免GIL带来的影响？
方法一：用进程+协程 代替 多线程的方式
在多进程中，由于每个进程都是独立的存在，所以每个进程内的线程都拥有独立的GIL锁，互不影响。但是，由于进程之间是独立的存在，所以进程间通信就需要通过队列的方式来实现。
 
方法二：更换解释器
像JPython和IronPython这样的解析器由于实现语言的特性，他们不需要GIL的帮助。然而由于用了Java/C#用于解析器实现，他们也失去了利用社区众多C语言模块有用特性的机会。所以这些解析器也因此一直都比较小众。
```

