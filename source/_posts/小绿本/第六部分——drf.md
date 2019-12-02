---
title: 第六部分——drf
id: 6
date: 2019-11-16 20:00:00
tags: 小绿本
toc: true
comment: true
---

#### 1.装饰器

```python
def outer(func):
    def inner(*args,**kwargs):
        return func(*args,**kwargs)
    return inner

@outer
def index(a1):
    pass

index()
```

```python
def outer(func):
    def inner(*args,**kwargs):
        return func(*args,**kwargs)
    return inner

def index(a1):
    pass

#这种方式执行的和装饰器的效果是一样的，但是这种方法要看懂，因为源码里面有这样写的
index = outer(index)
#现在index = inner

index()
```

<!----more--->

#### 2.django中可以免除csrftoken认证

```python
from django.views.decorators.csrf import csrf_exempt
from django.shortcuts import HttpResponse

@csrf_exempt
def index(request):
    return HttpResponse('...')

# index = csrf_exempt(index)

urlpatterns = [
    url(r'^index/$',index),
]
```

在drf中的应用

```python
urlpatterns = [
    url(r'^login/$',account.LoginView.as_view()),
]

class APIView(View):
    @classmethod
    def as_view(cls, **initkwargs):
        view = super().as_view(**initkwargs)
        view.cls = cls
        view.initkwargs = initkwargs

        # 注意：基于会话的身份验证是明确的CSRF验证，
        # 所有其他身份验证都是CSRF豁免的。
        return csrf_exempt(view)
```

#### 3.面向对象中基于继承+异常处理来做的约束

```python
class BaseVersioning:
    #父类约束
    def determine_version(self, request, *args, **kwargs):
        raise NotImplementedError("must be implemented")
        
class URLPathVersioning(BaseVersioning):
    #子类执行父类约束方法
	def determine_version(self, request, *args, **kwargs):
        version = kwargs.get(self.version_param, self.default_version)
        if version is None:
            version = self.default_version

        if not self.is_allowed_version(version):
            raise exceptions.NotFound(self.invalid_version_message)
        return version
```

#### 4.面向对象封装

```python
class Foo(object):
	def __init__(self,name,age):
		self.name = name
		self.age = age 
		
obj = Foo('pl',18)
```

```python
class APIView(View):
    def dispatch(self, request, *args, **kwargs):
		#封装request
        self.args = args
        self.kwargs = kwargs
        request = self.initialize_request(request, *args, **kwargs)
        #返回封装的request，但是现在的request里面包含了更多的内容
        self.request = request
		...
        
	def initialize_request(self, request, *args, **kwargs):
        """
        Returns the initial request object.
        """
        parser_context = self.get_parser_context(request)

        return Request(
            request,
            parsers=self.get_parsers(),
            authenticators=self.get_authenticators(), # [MyAuthentication(),]
            negotiator=self.get_content_negotiator(),
            parser_context=parser_context
        )
```

#### 5.面向对象继承

```python
class View(object):
   	pass

class APIView(View):
    def dispatch(self):
        method = getattr(self,'get')
        method()

class GenericAPIView(APIView):
    serilizer_class = None
    
    def get_seriliser_class(self):
        return self.serilizer_class

class ListModelMixin(object):
    def get(self):
        ser_class = self.get_seriliser_class()
        print(ser_class)

class ListAPIView(ListModelMixin,GenericAPIView):
    pass

class UserInfoView(ListAPIView):
    
    def get_seriliser_class(self):
        return "咩咩"

view = UserInfoView()
view.dispatch()
```

#### 6.反射

```python
class View(object):
	def dispatch(self, request, *args, **kwargs):
        # 通过反射的方式去执行method方法
        if request.method.lower() in self.http_method_names:
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed
        return handler(request, *args, **kwargs)
```

#### 7.发送ajax请求

```
$.ajax({
	url:'地址',
	type:'GET',
	data:{...},
	success:function(arg){
		console.log(arg);
	}
})
```

#### 8.浏览器具有 "同源策略的限制"，导致 `发送ajax请求` + `跨域` 存在无法获取数据。

- 3个不同：协议、域名、端口号
- 简单请求，发送一次请求。
- 复杂请求，先options请求做预检，然后再发送真正请求

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>常鑫的网站</h1>
    <p>
        <input type="button" value="点我" onclick="sendMsg()">
    </p>
    <p>
        <input type="button" value="点他" onclick="sendRemoteMsg()">
    </p>

  
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
    <script>
        function sendMsg() {
            $.ajax({
                url:'/msg/',
                type:'GET',
                success:function (arg) {
                    console.log(arg);
                }
            })
        }
        function sendRemoteMsg() {
            $.ajax({
                url:'http://127.0.0.1:8002/json/',
                type:'GET',
                success:function (arg) {
                    console.log(arg);
                }
            })

        }
    </script>
</body>
</html>
```

#### 9.如何解决ajax+跨域？

```
CORS，跨站资源共享，本质：设置响应头。
```

#### 10.常见的Http请求方法

```python
get
post
put
patch
delete
options:对于跨域时的预检
```

#### 11.http请求中Content-type请求头

```python
情况一：
    content-type:x-www-form-urlencode
    name=alex&age=19&xx=10
	
	request.POST和request.body中均有值。
	
情况二：
	content-type:application/json
    {"name":"ALex","Age":19}
    
    request.POST没值
    request.body有值。
```

#### 12.django中获取空Queryset

```python
models.User.object.all().none()
```

#### 13.基于django的fbv和cbv都能实现遵循restful规范的接口

```python
def user(request):
    if request.metho == 'GET':
        pass
    
    
class UserView(View):
    def get()...
    
    def post...
```

#### 14.基于django rest framework框架实现restful api的开发

```
- 免除csrf认证
- 视图（三种继承：APIView、ListAPIView、ListModelMinx）
- 版本
- 认证
- 权限
- 节流
- 解析器
- 筛选器
- 分页
- 序列化
- 渲染器
```

#### 15.简述drf中认证流程？

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

#### 16.简述drf中节流的实现原理以及过程？匿名用户/非匿名用户 如何实现频率限制？

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
如1分钟：40次，列表长度限制在40，超过40则不可访问
```

#### 17.GenericAPIView视图类的作用？

```python
他提供了一些规则，例如：

class GenericAPIView(APIView):
    serializer_class = None
    queryset = None
    lookup_field = 'pk'
    
    filter_backends = api_settings.DEFAULT_FILTER_BACKENDS
    pagination_class = api_settings.DEFAULT_PAGINATION_CLASS
    
    def get_queryset(self):
        return self.queryset
    
    def get_serializer_class(self):
        return self.serializer_class
    
	def filter_queryset(self, queryset):
        for backend in list(self.filter_backends):
            queryset = backend().filter_queryset(self.request, queryset, self)
        return queryset
    
    @property
    def paginator(self):
        if not hasattr(self, '_paginator'):
            if self.pagination_class is None:
                self._paginator = None
            else:
                self._paginator = self.pagination_class()
        return self._paginator
    
他相当于提供了一些规则，建议子类中使用固定的方式获取数据，例如：
class ArticleView(GenericAPIView):
    queryset = models.User.objects.all()
    
    def get(self,request,*args,**kwargs):
        query = self.get_queryset()

我们可以自己继承GenericAPIView来实现具体操作，但是一般不会，因为更加麻烦。
而GenericAPIView主要是提供给drf内部的 ListAPIView、Create....
class ListModelMixin:
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
    
class ListAPIView(mixins.ListModelMixin,GenericAPIView):
    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

class MyView(ListAPIView):
    queryset = xxxx 
    ser...
```

```
总结：GenericAPIView主要为drf内部帮助我们提供增删改查的类LIstAPIView、CreateAPIView、UpdateAPIView、提供了执行流程和功能，我们在使用drf内置类做CURD时，就可以通过自定义 静态字段（类变量）或重写方法（get_queryset、get_serializer_class）来进行更高级的定制。
```

#### 18.jwt以及其优势。

```
一般在前后端分离时，用于做用户认证（登录）使用的技术。
jwt的实现原理：
	- 用户登录成功之后，会给前端返回一段token。
	- token是由.分割的三段组成。
		- 第一段：类型和算法信心
		- 第二段：用户信息+超时时间
		- 第三段：hs256（前两段拼接）加密 + base64url
	- 以后前端再次发来信息时
		- 超时验证
		- token合法性校验
优势：
	- token只在前端保存，后端只负责校验。
	- 内部集成了超时时间，后端可以根据时间进行校验是否超时。
	- 由于内部存在hash256加密，所以用户不可以修改token，只要一修改就认证失败。
```

#### 19.序列化时many=True和many=False的区别？

```
在序列化的时候，many=True可以针对的是多条数据
many=False针对的是单条数据而言
```

#### 20.应用DRF中的功能进行项目开发

```
*****
	解析器:request.query_parmas/request.data
	视图
	序列化
	渲染器：Response

****
	request对象封装
	版本处理
	分页处理
***
	认证
	权限
	节流
```

- 基于APIView实现呼啦圈
- 继承ListAPIView+ GenericViewSet,ListModelMixin实现呼啦圈

#### 21.接口的幂等性是什么意思？

```
首先，我们来看一下接口存在的问题：
	-现如今我们的系统大多拆分为分布式SOA，或者微服务，一套系统中包含了多个子系统服务，而一个子系统服务往往会去调用另一个服务，而服务调用服务无非就是使用RPC通信或者restful，既然是通信，那么就有可能在服务器处理完毕后返回结果的时候挂掉，这个时候用户端发现很久没有反应，那么就会多次点击按钮，这样请求有多次，那么处理数据的结果是否要统一呢？那是肯定的！尤其在支付场景。
	
那么，什么是接口的幂等性？
	-接口幂等性就是用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生副作用。

什么情况下需要保证接口的幂等性？
	-在增删查改中，尤其注意增加和修改
	新增：比如支付时候的重复提交事件
	修改：比如A字段增加1，这种操作就不是幂等的

那么，如何设计接口才能做到幂等呢？
	-通过代码逻辑判断实现
	-使用token机制是实现
```

#### 22.什么是RPC？

```
'远程过程调用协议'
是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。
进化的顺序: 先有的RPC,然后有的RESTful规范
```

#### 23.Http和Https的区别？

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

#### 24.为什么要使用django rest framework框架？

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

#### 25.django rest framework框架中的视图都可以继承哪些类

```python
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

#### 26.drf框架如何对Queryset进行序列化？

```
queryset = Book.objects.all()
#定义一个序列化类
serializer = BookSerializer(queryset, many=True)
```