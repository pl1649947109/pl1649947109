---
title: 第八讲——django知识回顾及cookie和session
id: 8
date: 2019-10-10 20:30:00
tags: Django
comment: true
---

今日大纲

- 知识回顾
- cookie
- session

<!----more---->

### 知识回顾

知识回顾一：最简单的装饰器

```python
def wrapper(f):
	def inner(*args,**kwargs):
		return f(*args,**kwargs)
	return inner
扩展：
import functools
def wrapper(f):
    @functools.wraps(f)  #保留元数据
	def inner(*args,**kwargs):
		return f(*args,**kwargs)
	return inner

@wrapper
def index(a,b)
	return a+b
print (index.__name__)  #没有加@functools.wraps(f)之前打印的是inner，但是我们想要保留员信息怎么办，那就加这个装饰器就行了，打印的就是index。
print (index.__doc__)
```

知识点回顾二：什么是HTTP协议？

```python
超文本传输协议
关于连接：一次请求一次响应之后断开连接（无状态、短链接）
关于格式：
	请求：请求头+请求体(http://www.baidu.com/index/?a=123)
	send("GET /index/?a=123 http1.1\r\nuser-agent:xxx\r\nhpst:www.baidu.com\r\n\r\n")
    send("POST /index/ http1.1\r\nuser-agent:xxx\r\n\r\nname=pl&password=123")
	响应：响应头+响应体
    Content-Encoding:gzip\r\nCache-Control:
private\r\n\r\n网页看到的HTML内容
#注意：回答的时候就按照上面的三点内容回答

'''	
\r\n:请求头和请求头之间的分隔
\r\n\r\n:请求头和请求体之前的分隔
'''
扩展一：常见的请求头有哪些？
	-user-agent：浏览器标识
	-content-type：用来标记请求体的格式是什么样子的，服务器根据的格式的要求进行解析
    
HTTP头字段总结
最后我总结下HTTP协议的头部字段。

1、 Accept：告诉WEB服务器自己接受什么介质类型，*/* 表示任何类型，type/* 表示该类型下的所有子类型，type/sub-type。
2、 Accept-Charset： 浏览器申明自己接收的字符集 
Accept-Encoding： 浏览器申明自己接收的编码方法，通常指定压缩方法，是否支持压缩，支持什么压缩方法（gzip，deflate） 
Accept-Language：浏览器申明自己接收的语言 
语言跟字符集的区别：中文是语言，中文有多种字符集，比如big5，gb2312，gbk等等。
3、 Accept-Ranges：WEB服务器表明自己是否接受获取其某个实体的一部分（比如文件的一部分）的请求。bytes：表示接受，none：表示不接受。
4、 Age：当代理服务器用自己缓存的实体去响应请求时，用该头部表明该实体从产生到现在经过多长时间了。
5、 Authorization：当客户端接收到来自WEB服务器的 WWW-Authenticate 响应时，用该头部来回应自己的身份验证信息给WEB服务器。
6、 Cache-Control：请求：no-cache（不要缓存的实体，要求现在从WEB服务器去取） 
max-age：（只接受 Age 值小于 max-age 值，并且没有过期的对象） 
max-stale：（可以接受过去的对象，但是过期时间必须小于 max-stale 值） 
min-fresh：（接受其新鲜生命期大于其当前 Age 跟 min-fresh 值之和的缓存对象） 
响应：public(可以用 Cached 内容回应任何用户) 
private（只能用缓存内容回应先前请求该内容的那个用户） 
no-cache（可以缓存，但是只有在跟WEB服务器验证了其有效后，才能返回给客户端） 
max-age：（本响应包含的对象的过期时间） 
ALL: no-store（不允许缓存）
7、 Connection：请求：close（告诉WEB服务器或者代理服务器，在完成本次请求的响应后，断开连接，不要等待本次连接的后续请求了）。 
keepalive（告诉WEB服务器或者代理服务器，在完成本次请求的响应后，保持连接，等待本次连接的后续请求）。 
响应：close（连接已经关闭）。 
keepalive（连接保持着，在等待本次连接的后续请求）。 
Keep-Alive：如果浏览器请求保持连接，则该头部表明希望 WEB 服务器保持连接多长时间（秒）。例如：Keep-Alive：300
8、 Content-Encoding：WEB服务器表明自己使用了什么压缩方法（gzip，deflate）压缩响应中的对象。例如：Content-Encoding：gzip
9、Content-Language：WEB 服务器告诉浏览器自己响应的对象的语言。
10、Content-Length： WEB 服务器告诉浏览器自己响应的对象的长度。例如：Content-Length: 26012
11、Content-Range： WEB 服务器表明该响应包含的部分对象为整个对象的哪个部分。例如：Content-Range: bytes 21010-47021/47022
12、Content-Type： WEB 服务器告诉浏览器自己响应的对象的类型。例如：Content-Type：application/xml
13、ETag：就是一个对象（比如URL）的标志值，就一个对象而言，比如一个 html 文件，如果被修改了，其 Etag 也会别修改，所以ETag 的作用跟 Last-Modified 的作用差不多，主要供 WEB 服务器判断一个对象是否改变了。比如前一次请求某个 html 文件时，获得了其 ETag，当这次又请求这个文件时，浏览器就会把先前获得的 ETag 值发送给WEB 服务器，然后 WEB 服务器会把这个 ETag 跟该文件的当前 ETag 进行对比，然后就知道这个文件有没有改变了。
14、 Expired：WEB服务器表明该实体将在什么时候过期，对于过期了的对象，只有在跟WEB服务器验证了其有效性后，才能用来响应客户请求。是 HTTP/1.0 的头部。例如：Expires：Sat, 23 May 2009 10:02:12 GMT
15、 Host：客户端指定自己想访问的WEB服务器的域名/IP 地址和端口号。例如：Host：rss.sina.com.cn
16、 If-Match：如果对象的 ETag 没有改变，其实也就意味著对象没有改变，才执行请求的动作。
17、 If-None-Match：如果对象的 ETag 改变了，其实也就意味著对象也改变了，才执行请求的动作。
18、 If-Modified-Since：如果请求的对象在该头部指定的时间之后修改了，才执行请求的动作（比如返回对象），否则返回代码304，告诉浏览器该对象没有修改。例如：If-Modified-Since：Thu, 10 Apr 2008 09:14:42 GMT
19、 If-Unmodified-Since：如果请求的对象在该头部指定的时间之后没修改过，才执行请求的动作（比如返回对象）。
20、 If-Range：浏览器告诉 WEB 服务器，如果我请求的对象没有改变，就把我缺少的部分给我，如果对象改变了，就把整个对象给我。浏览器通过发送请求对象的 ETag 或者 自己所知道的最后修改时间给 WEB 服务器，让其判断对象是否改变了。总是跟 Range 头部一起使用。
21、 Last-Modified：WEB 服务器认为对象的最后修改时间，比如文件的最后修改时间，动态页面的最后产生时间等等。例如：Last-Modified：Tue, 06 May 2008 02:42:43 GMT
22、 Location：WEB 服务器告诉浏览器，试图访问的对象已经被移到别的位置了，到该头部指定的位置去取。例如：Location：http://i0.sinaimg.cn/dy/deco/2008/0528/sinahome_0803_ws_005_text_0.gif
23、 Pramga：主要使用 Pramga: no-cache，相当于 Cache-Control： no-cache。例如：Pragma：no-cache
24、 Proxy-Authenticate： 代理服务器响应浏览器，要求其提供代理身份验证信息。Proxy-Authorization：浏览器响应代理服务器的身份验证请求，提供自己的身份信息。
25、 Range：浏览器（比如 Flashget 多线程下载时）告诉 WEB 服务器自己想取对象的哪部分。例如：Range: bytes=1173546-
26、 Referer：浏览器向 WEB 服务器表明自己是从哪个 网页/URL 获得/点击 当前请求中的网址/URL。例如：Referer：http://www.sina.com/
27、 Server: WEB 服务器表明自己是什么软件及版本等信息。例如：Server：Apache/2.0.61 (Unix)
28、 User-Agent: 浏览器表明自己的身份（是哪种浏览器）。例如：User-Agent：Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.8.1.14) Gecko/20080404 Firefox/2、0、0、14
29、 Transfer-Encoding: WEB 服务器表明自己对本响应消息体（不是消息体里面的对象）作了怎样的编码，比如是否分块（chunked）。例如：Transfer-Encoding: chunked
30、 Vary: WEB服务器用该头部的内容告诉 Cache 服务器，在什么条件下才能用本响应所返回的对象响应后续的请求。假如源WEB服务器在接到第一个请求消息时，其响应消息的头部为：Content-Encoding: gzip; Vary: Content-Encoding那么 Cache 服务器会分析后续请求消息的头部，检查其 Accept-Encoding，是否跟先前响应的 Vary 头部值一致，即是否使用相同的内容编码方法，这样就可以防止 Cache 服务器用自己 Cache 里面压缩后的实体响应给不具备解压能力的浏览器。例如：Vary：Accept-Encoding
31、 Via： 列出从客户端到 OCS 或者相反方向的响应经过了哪些代理服务器，他们用什么协议（和版本）发送的请求。当客户端请求到达第一个代理服务器时，该服务器会在自己发出的请求里面添加 Via 头部，并填上自己的相关信息，当下一个代理服务器收到第一个代理服务器的请求时，会在自己发出的请求里面复制前一个代理服务器的请求的Via 头部，并把自己的相关信息加到后面，以此类推，当 OCS 收到最后一个代理服务器的请求时，检查 Via 头部，就知道该请求所经过的路由。例如：Via：1.0 236.D0707195.sina.com.cn:80 (squid/2.6.STABLE13)
```

知识点回顾三：django的生命周期/浏览器上输入http://www.xxx.com请求到达服务器都发生了什么？

```
扩展：wsgi协议（本质就是socket）：
	-wsgiref：测试使用
	-uwsgi：部署使用
浏览器发送一个请求给服务端-->服务端的wsgi捕获到请求将它转发给服务端的内部-->服务端内部将它转发到路由系统，并进行路由匹配-->匹配到视图函数之后进入视图进行逻辑操作，这中间可能发生和数据库数据的交换，这涉及到了我们的ORM，同时我们的将我们的模板进行渲染（模板渲染本质就是字符串的替换）之后生成一个字符串-->将生成的字符串通过wsgi发送给客户端浏览器

扩展：WSGI（web server geteway interface）web服务器网关协议：是一种规范，它定义了使用python编写web app与web server之间接口格式，实现web app和web server之间的解耦。
```

图解：

![](http://9017499461.linshutu.top/django%E8%AF%B7%E6%B1%82%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

### 课前知识

**什么是会话跟踪**

会话就是客户端和服务器之间的一次会晤，在一次会晤中可能包含多次请求和响应。

在一个会话的多个请求中共享数据，这就是会话跟踪技术。例如在一个会话中的请求如下： 

- 请求银行主页； 
- 请求登录（请求参数是用户名和密码）；
- 请求转账（请求参数与转账相关的数据）； 
- 请求信誉卡还款（请求参数与还款相关的数据）。  

在这上面的会话中当前用户信息必须在这个会话中共享，因为登录的是张三，那么在转账和还款时一定是相对张三的转账和还款。这就说明我们必须在一个会话过程中有共享数据的能力。那么，在web中这种能力的实现就是要依靠cookie和session来实现的。

### 案例带入：blog系统

**cookie的由来**：HTTP协议是无状态的，对服务器来说，每次的请求都是全新的。状态可以理解为客户端和服务器在某次会话中产生的数据不会被浏览器保留下来。

有一个实际的案列就是：我们在登录一个网站的时候，我们没办法确定我们是不是登录了，如果我们登录了，还要保证登录了的用户不需要再重复登录，就能够访问我们网站的其他网址的页面。但是我们的http是无状态的，那怎么办？这个时候cookie就很巧妙的解决了这个问题。

**cookie的原理**：保存在用户浏览器端的键值对，向服务端发请求时会自动携带。（展开说：**cookie是浏览器技术**浏览器访问服务器，会带着空的cookie，然后由服务器产生内容，浏览器收到相应的内容后保存在本地；当浏览器再次访问服务器的时候，浏览器就会自动的携带cookie，这样服务器就可以判断这个发来消息的浏览器是“谁”了）

**session介绍**：session是一项服务器技术，利用这个技术，服务器在运行时为每一个用户的浏览器创建一个单独的session对象，也就是一个独立的空间，所以用户在访问服务器的web资源时 ，可以把各自的数据放在各自的session中，当用户再去访问该服务器中的其它web资源时，其它web资源再从用户各自的session中 取出数据为用户服务。

![](http://9017499461.linshutu.top/session.png)

##### 问题：用户未登录就不能访问指定页面

#### 基于cookie实现（操作）

**Ctrl + Shift + del三个键来清除页面缓存和cookie，将来这个操作你会用的很多。**

urls.py

```python
from django.conf.urls import url
from django.contrib import admin
from app01 import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^login/', views.login), #登录页面
    url(r'^index/', views.index), #后台页面
]
```

models.py

```python
from django.db import models

class UserInfo(models.Model):
    """
    用户表
    """
    username = models.CharField(verbose_name='用户名',max_length=32)
    password = models.CharField(verbose_name='密码',max_length=64)

class Blog(models.Model):
    """
    博客表
    """
    title = models.CharField(verbose_name='标题',max_length=32)
    content = models.TextField(verbose_name='博客内容')
    author = models.ForeignKey(verbose_name='作者',to='UserInfo')
```

views.py

```python
from django.shortcuts import render,redirect
from app01 import models
# Create your views here.
def login(request):
	"""
	用户登录
	:param request:
	:return:
	"""
	if request.method == "GET":
		return render(request,'login.html')
	#获取用户名和密码
	user = request.POST.get("user")
	pwd = request.POST.get("pwd")
	# 去数据库检查用户名密码是否正确
	# user_object = models.UserInfo.objects.filter(username=user,password=pwd).first()  #获取的是对象，存在就返回对象，不存在就返回None
	# user_object = models.UserInfo.objects.filter(username=user, password=pwd).exists() #返回的是布尔值，存在True
	user_object = models.UserInfo.objects.filter(username=user, password=pwd).exists()
	if user_object:
		result =  redirect('/index/')
		result.set_cookie('pl',user)
		return result
	#用户名或密码错误
	return render(request, "login.html", {"error": "用户名或密码错误"})

def index(request):
	"""
	后台博客首页
	:param request:
	:return:
	"""
	user = request.COOKIES.get("pl")
	if not user:
		return redirect('/login/')
	return render(request, "index.html",{"user":user})
```

##### cookie拓展

**cookie规范**

- cookie上限的大小是4k
- 一个服务器最多在客户端浏览器上保存20个cookie
-  一个浏览器最多保存300个Cookie，因为一个浏览器可以访问多个服务器。

注意：**上面的内容只是一种规范而已，不是标准**，不同浏览器之间是不共享Cookie的。也就是说在你使用IE访问服务器时，服务器会把Cookie发给IE，然后由IE保存起来，当你在使用FireFox访问服务器时，不可能把IE保存的Cookie发送给服务器。

**cookie的覆盖**

如果服务器端发送重复的Cookie那么会覆盖原有的Cookie，**例如客户端的第一个请求服务器端发送的Cookie是：Set-Cookie: a=A；第二请求服务器端发送的是：Set-Cookie: a=AA，那么客户端只留下一个Cookie，即：a=AA。 **

#### 基于session实现（操作）

index.html

```html
获取用户名
<h1>欢迎{{ request.session.user_name  }}登录后台管理系统</h1>
```

views.py

```python
from django.shortcuts import render,redirect
from app01 import models
# Create your views here.
def login(request):
	"""
	用户登录
	:param request:
	:return:
	"""
	if request.method == "GET":
		return render(request,'login.html')
	#获取用户名和密码
	user = request.POST.get("user")
	pwd = request.POST.get("pwd")
	# 去数据库检查用户名密码是否正确
	# user_object = models.UserInfo.objects.filter(username=user,password=pwd).first()  #获取的是对象，存在就返回对象，不存在就返回None
	# user_object = models.UserInfo.objects.filter(username=user, password=pwd).exists() #返回的是布尔值，存在True
	user_object = models.UserInfo.objects.filter(username=user, password=pwd).first()
	if user_object:
		# result =  redirect('/index/')
		# result.set_cookie('pl',user)
		# return result

		request.session["user_name"] = user_object.username
		request.session['user_id'] = user_object.pk
		return redirect('/index/')
	#用户名或密码错误
	return render(request, "login.html", {"error": "用户名或密码错误"})

def auth(func):
	def inner(request,*args,**kwargs):
		name = request.session.get('user_name')
		if not name:
			return redirect('/login/')
		return func(request,*args,**kwargs)
	return inner

@auth
def index(request):
	"""
	后台博客首页
	:param request:
	:return:
	"""
	return render(request,'index.html')
```

##### session拓展

Session就是在服务器端的‘Cookie’，将用户数据保存在服务器端，远比保存在用户端要安全、方便和快捷得多。

```python
原理：
session是一种存储数据的方式,依赖于cookie,实现本质:
	用户向服务端发送请求,服务端做两件事:生成随机字符串;为此用户开辟一个独立的空间来存放当前用户独有的值.
	在空间中如何想要设置值:
        request.session['x1'] = 123
        request.session['x2'] = 456
	在空间中取值:
        request.session['x2'] 
        request.session.get('x2')  #推荐使用
	视图函数中的业务操作处理完毕,给用户响应,在响应时会将随机字符串存储到用户浏览器的cookie中
```

问题：

```python
cookie和session的区别?（从原理的角度来分析）
答: cookie是存储在客户端浏览器上的键值对,发送请求时浏览器会自动携带. session是一种存储数据方式,基于cookie实现,将数据存储在服务端(django默认存储到数据库).其本质是:
	用户向服务端发送请求,服务端做两件事:生成随机字符串;为此用户开辟一个独立的空间来存放当前用户独有的值.
	在空间中如何想要设置值:
        request.session['x1'] = 123
        request.session['x2'] = 456
	在空间中取值:
        request.session['x2']
        request.session.get('x2')
	视图函数中的业务操作处理完毕,给用户响应,在响应时会将随机字符串存储到用户浏览器的cookie中.
```

django中的session相关的配置

```python
1. 数据库Session
SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）

2. 缓存Session
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置

3. 文件Session
SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir() 

4. 缓存+数据库
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎

5. 加密Cookie Session
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎

#公用设置
SESSION_COOKIE_NAME = "sessionid" #Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串
SESSION_COOKIE_DOMAIN = None #api.baidu.com /www.baidu.com/ xxx.baidu.com
SESSION_COOKIE_PATH = "/" # Session的cookie保存的路径
SESSION_COOKIE_HTTPONLY = True # 是否Session的cookie只支持http传输
SESSION_COOKIE_AGE = 1209600 # Session的cookie失效日期（2周）
SESSION_EXPIRE_AT_BROWSER_CLOSE = False #是否关闭浏览器使得Session过期
SESSION_SAVE_EVERY_REQUEST = False # 是否每次请求都保存Session，默认修改之后才保存
```

django中的session如何设置过期时间?

```python
SESSION_COOKIE_AGE = 1209600 # Session的cookie失效日期（2周）
```

django的session默认存储在数据库,可以放在其他地方吗?

```python
小系统:默认放在数据库即可.
大系统:缓存(redis)

#session保存在文件
SESSION_ENGINE =
'django.contrib.sessions.backends.file'
SESSION_FILE_PATH = '/sssss/'

#session保存在缓存(内存)
SESSION_ENGINE =
'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
CACHES = {
	'default': {
		'BACKEND':
'django.core.cache.backends.locmem.LocMem
Cache',
		'LOCATION': 'unique-snowflake',
	}
}

#session保存在缓存(redis)
SESSION_ENGINE =
'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
CACHES = {
	"default": {
		"BACKEND":
"django_redis.cache.RedisCache",
		"LOCATION":
"redis://127.0.0.1:6379",
	"OPTIONS": {
		"CLIENT_CLASS":
"django_redis.client.DefaultClient",
		"CONNECTION_POOL_KWARGS":
{"max_connections": 100}
		# "PASSWORD": "密码",
		}
	}
}
```

操作session

```python
# 设置(添加&修改)
request.session['x1'] = 123
request.session['x2'] = 456
request.session.setdefault('k1',123) #存在则不设置
# 读取
request.session['xx']
request.session.get('xx')
# 删除
del request.session['xx']
#删除所有
request.session.delete()
内容总结
1. 装饰器要加入functools.wrap装饰
2. orm字段中的verbose_name
3. 路由系统中记得加入终止符 $
4. 用户名和密码检测
5. 模板查找顺序
#查所有的键、值、键值对
request.session.keys()
request.session.values()
request.session.items()
#设置会话Session和Cookie的超时时间
request.session.set_expiry(value)
	* 如果value是个整数，session会在些秒数后失效。
    * 如果value是个datatime或timedelta，session就会在这个时间后失效。
    * 如果value是0,用户关闭浏览器session就会失效。
    * 如果value是None,session会依赖全局session失效策略。
#将过期的删除
request.session.clear_expired()
#获取会话session的key
request.session.session_key
#检查会话session的key在数据库中是否存在
request.session.exists("session_key")
# 删除当前的会话数据并删除会话的Cookie
request.session.flush()   '''常用，清空所有cookie---删除session表里的这个会话的记录，这用于确保前面的会话数据不可以再次被用户的浏览器访问例如，django.contrib.auth.logout() 函数中就会调用它。'''

```

扩展：http://www.liujiangblog.com/course/django/168

### 总结

- 装饰器要加入functools.wrap装饰

```
保留函数的元数据(函数名/注释)
```

- orm字段中的verbose_name

```
目前当注释用.
以后:在model form和form中使用.
```

- 路由系统中记得加入终止符 $

- 用户名和密码检测

```
xxxx.first() # 返回对象或None
xxxx.exists() # 返回布尔值
```

- 模板查找顺序

```python
首先在根目录templates找，如果没有就根据app注册顺序去每个app里面找自己定义tempaltes里面的文件

#注意:
app里面的文件的名称可以是任意的名称，为了安全，我们最好不要使用templates这个名称,你如设定statics。那么怎么引入他们呢？
在settings.py里面配置：
STATICFIELS_DIRS = [
    os.path.join(BASE_DIRS,'/statics/')
]
这样，我们在html中就可以引用了。（两种引用方式）
方式一：和原来一样
方式二：
{% load static%}   #固定写法
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>主页</title>
</head>
<body>
<h1 style="background: greenyellow">{{ request.session.user_name }}!欢迎小主回家！</h1>
</body>
</html>
```

-  cookie

  操作

```python
def login(request):
    # return HttpResponse('...')
    # return render('...')
    # return redirect('...')
    
    # 设置cookie
    data = redirect('...')
    data.set_cookie()
    
    # 读取cookie
    request.COOKIES.get('xx')
    
    #删除cookie
    data = redirect("...")
    data.delete_cookie("xx")
必须会背cookie的下面三个参数:
key, value='', max_age=None
其他参数：
expires=None, 超时时间
                   
path='/', Cookie生效的路径，/ 表示根路径，特殊的：根路径的cookie可以被任何url的页面访问

domain=None, Cookie生效的域名

secure=False, https传输

httponly=False 只能http协议传输，无法被JavaScript获取（不是绝对，底层抓包可以获取到也可以被覆盖）

#应用场景:
    用户认证
    投票
    每页默认显示数据
#注意：cookie在设置的时候不允许出现中文，但是也是有办法解决的，就是编码，解码呗。
```

jQuery操作cookie：https://www.cnblogs.com/clschao/articles/10480029.html

-  session

```python
配置
数据存储位置
	数据库(django默认)
	文件
	缓存(内存/redis)

#应用场景
    用户认证
    短信验证过期
    权限管理
```

强调

```python
session中的数据是根据用户相互隔离.
# 示例
def login(request):
    # 获取用户提交的用户名和密码
    user = request.POST.get('user')
    request.session['user_name'] = user
def index(request):
	print(request.session['user_name'])
```

- 通过js设置cookie

```python
document.cookie = 'k1=wy222;path=/'
$.cookie('k1','wy222',{path:'/'})
注意:path不同会导致设置不同.
```

- path的作用

  ```python
  / , 当前网站中所有的URL都能读取到此值.
  "",只能在当前页面访问的到此数据.
  /index/ ,只能在/index/xxx 的网页中查看.
  
  from django.shortcuts import render,HttpResponse
  
  # Create your views here.
  
  def index(request):
      data = HttpResponse('相应')
  
      data.set_cookie('a1','1',path='/')
      data.set_cookie('a2','2',path='')
      data.set_cookie('a3','3',path='/test/')
  
      return data
  
  
  def home(request):
      return HttpResponse('HOME')  #a1,a2
  
  def test(request):
      return HttpResponse('TEST')  #a1,a3
  
  def test_api(request):
      return HttpResponse('TEST API')  #a1,a3
  ```

- django的路由的url路径后面加'/'和不加'/'的区别：
  - 不加"/",系统会自动进行301的重定向，把url变成末尾加”/“的地址。
  - 如而request.getParameter("")仅有一次生命周期，经过两次跳转后，前系统传的值失效了，取值会出现问题。
  - 加'/'速度将会更快，当然这种速度是感觉不到的，但优化应该“尽可能”，减少浪费的时间。