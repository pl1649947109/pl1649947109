---
title: 第六讲——drf写视图的方法、呼啦圈最终项目版本
id: 7
date: 2019-11-12 20:00:00
tags: drf
comment: true
---

### 内容回顾补充

1.drf组件认证的实现过程？

```
路由进入我们的视图首先进入as_view()类，在该类里面首先执行dispacth方法，接着执行initial_request对request进行封装，在这个封装里面会封装一个get_authenticators，它会循环所有的authentication对象并对其进行实例化封装成一个列表；接着执行initial方法，它里面会执行perform_authentication，该方法里面执行request.user，它就会循环执行我们的认证类，它会返回3种情况，它返回一个元祖（user,auth）表示认证成功或者NOne（接着进行下一个认证）或者抛出异常；我们根据返回的情况来判断认证。
```

2.drf组件中权限的实现过程？

```
permission_classes = api_settings.DEFAULT_PERMISSION_CLASSES

路由进入我们的视图首先进入as_view()类，在该类里面首先执行dispacth方法，在这个方法里面接着会去执行initial方法，在这个方法里面会执行一个check_permissions方法，
该方法里面接着会去执行get_permissions，这个方法只执行
return [permission() for permission in self.permission_classes]，在这里面会去找我们自己是否有定义的permission_classes，这是和列表，里面我们可以重写，去执行里面的类，如果该方法没有has_permission方法，就会执行permission_denied去触发异常。
```

<!----more---->

3.drf组件中节流的实现方式？

```python
throttle_classes = api_settings.DEFAULT_THROTTLE_CLASSES

路由进入我们的视图首先进入as_view()类，在该类里面首先执行dispacth方法，在这个方法里面接着会去执行initial方法，接着执行check_throttles，这这个方法里面定义了throttle_durations=[]这样一个列表，这里面[throttle() for throttle in self.throttle_classes]先遍历throttle_classes，

然后业务就是这样子的：
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

4.什么是jwt？优势？

```python
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

### 写视图的方法（三种）

#### 第一种：原始APIView

```python
url(r'^login/$',account.LoginView.as_view())
```

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework_jwt.settings import api_settings
from rest_framework.throttling import AnonRateThrottle
from api import models


class LoginView(APIView):
    authentication_classes = []
    def post(self,request,*args,**kwargs):
        # 1.根据用户名和密码检测用户是否可以登录
        user = models.UserInfo.objects.filter(username=request.data.get('username'),password=request.data.get('password')).first()
        if not user:
            return Response({'code':10001,'error':'用户名或密码错误'})

        # 2. 根据user对象生成payload（中间值的数据）
        jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
        payload = jwt_payload_handler(user)

        # 3. 构造前面数据，base64加密；中间数据base64加密；前两段拼接然后做hs256加密（加盐），再做base64加密。生成token
        jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
        token = jwt_encode_handler(payload)
        return Response({'code': 10000, 'data': token})
```

#### 第二种：ListApiView等

```python
url(r'^article/$',article.ArticleView.as_view()),
url(r'^article/(?P<pk>\d+)/$',article.ArticleDetailView.as_view()),
```

```python
from rest_framework.throttling import AnonRateThrottle
from rest_framework.response import Response
from rest_framework.generics import ListAPIView,RetrieveAPIView
from api import models
from api.serializer.article import ArticleSerializer,ArticleDetailSerializer

class ArticleView(ListAPIView):
    authentication_classes = []
    # throttle_classes = [AnonRateThrottle,]

    queryset = models.Article.objects.all()
    serializer_class = ArticleSerializer

class ArticleDetailView(RetrieveAPIView):
    authentication_classes = []
    queryset = models.Article.objects.all()
    serializer_class = ArticleDetailSerializer
```

#### 第三种：ListModelMixin等

```python
url(r'^article/$',article.ArticleView.as_view({"get":'list','post':'create'})),
    url(r'^article/(?P<pk>\d+)/$',article.ArticleView.as_view({'get':'retrieve','put':'update','patch':'partial_update','delete':'destroy'}))
```

```python
from rest_framework.viewsets import GenericViewSet
from rest_framework.mixins import ListModelMixin,RetrieveModelMixin,CreateModelMixin,UpdateModelMixin,DestroyModelMixin
from api.serializer.article import ArticleSerializer,ArticleDetailSerializer

class ArticleView(GenericViewSet,ListModelMixin,RetrieveModelMixin,CreateModelMixin,UpdateModelMixin,DestroyModelMixin):
    authentication_classes = []
    throttle_classes = [AnonRateThrottle,]

    queryset = models.Article.objects.all()
    serializer_class = None

    def get_serializer_class(self):
        pk = self.kwargs.get('pk')
        if pk:
            return ArticleDetailSerializer
        return ArticleSerializer
```

### 最终版本项目

项目文件目录

![](http://9017499461.linshutu.top/drf.png)

项目思路：

models.py

```python
from django.db import models


class UserInfo(models.Model):
    """ 用户表 """
    username = models.CharField(verbose_name='用户名',max_length=32)
    password = models.CharField(verbose_name='密码',max_length=64)


class Article(models.Model):
    """ 文章表 """
    category_choices = (
        (1,'咨询'),
        (2,'公司动态'),
        (3,'分享'),
        (4,'答疑'),
        (5,'其他'),
    )
    category = models.IntegerField(verbose_name='分类',choices=category_choices)
    title = models.CharField(verbose_name='标题',max_length=32)
    image = models.CharField(verbose_name='图片路径',max_length=128) # /media/upload/....
    summary = models.CharField(verbose_name='简介',max_length=255)

    comment_count = models.IntegerField(verbose_name='评论数',default=0)
    read_count = models.IntegerField(verbose_name='浏览数',default=0)

    author = models.ForeignKey(verbose_name='作者',to='UserInfo')
    date = models.DateTimeField(verbose_name='创建时间',auto_now_add=True)


class ArticleDetail(models.Model):
    article = models.OneToOneField(verbose_name='文章表',to='Article')
    content = models.TextField(verbose_name='内容')


class Comment(models.Model):
    """ 评论表 """
    article = models.ForeignKey(verbose_name='文章',to='Article')
    content = models.TextField(verbose_name='评论')
    user = models.ForeignKey(verbose_name='评论者',to='UserInfo')
```

我一般喜欢先配置settings文件：

#### 1.软件注册

我们这里肯定会使用到rest_framework和jwt，所以先把他们注册了先：

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'api.apps.ApiConfig',
    #注册rest_framework组件
    'rest_framework',
    #注册rest_framework_jwt组件
    'rest_framework_jwt',
]
```

#### 2.路由分配

然后我们因为分路由了，那么我们再去配置路由：

```python
from django.conf.urls import url,include

urlpatterns = [
    #主路由这里使用了api表示和版本控制，后面会说到
    url(r'^api/(?P<version>\w+)/', include('api.urls')),
]
```

```python
from django.conf.urls import url,include
from django.contrib import admin
from .views import account
from .views import order
from .views import comment
from .views import article
from django.views.decorators.csrf import csrf_exempt
from django.shortcuts import HttpResponse

urlpatterns = [
    #登陆路由
    url(r'^login/$',account.LoginView.as_view()),
	#平论路由
    url(r'^comment/$',comment.CommentView.as_view()),
	#文章路由
    url(r'^article/$',article.ArticleView.as_view()),
    url(r'^article/(?P<pk>\d+)/$',article.ArticleDetailView.as_view()),
	#这个也是文章路由，不过这种采用的视图是listModelmixin类型的，所以url也需要修改，我们后面会一点点的解析的
    # url(r'^article/$',article.ArticleView.as_view({"get":'list'})),
    # url(r'^article/(?P<pk>\d+)/$',article.ArticleView.as_view({'get':'retrieve'}))
]
```

#### 3.登录认证

路由就是我们程序的门户，那么，我们下面就按照路由一条条地去分析我们的代码，那我们就先拿url(r'^login/$',account.LoginView.as_view())下手，根据路由，我们进入account文件：

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework_jwt.settings import api_settings
from rest_framework.throttling import AnonRateThrottle
from api import models


class LoginView(APIView):
    authentication_classes = []
    def post(self,request,*args,**kwargs):
        # 1.根据用户名和密码检测用户是否可以登录
        user = models.UserInfo.objects.filter(username=request.data.get('username'),password=request.data.get('password')).first()
        if not user:
            return Response({'code':10001,'error':'用户名或密码错误'})

        # 2. 根据user对象生成payload（中间的数据，xx.xx.xx）
        jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
        payload = jwt_payload_handler(user)

        # 3. 构造前面数据，base64加密；中间数据base64加密；前两段拼接然后做hs256加密（加盐），再做base64加密。生成token
        jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
        token = jwt_encode_handler(payload)
        return Response({'code': 10000, 'data': token})
```

分析：登陆相关的视图，只有一个post的方法向我们的后台提交数据，因为只有一个简单的提交数据的处理，所以我们就不使用复杂的View，而是继承最简单的，最靠近底层的APIView；

在函数的最开是的地方有以一个authentication_classes（这个是认证相关的，因为这里的业务是登陆就不存在什么认证相关的，所以在这里把该变量设置为空列表，就不会触发认证了，后面会讲到的）；

因为request在执行我们该视图类之前在initial里面进行了封装，所以，我们可以通过request.data取得前端发送过来的数据 和数据库进行查找判断；

接着，api_settings.JWT_PAYLOAD_HANDLER，它是干什么的呢？我大致看了一下，里面就是一个配置文件，写的是一些静态的参数，估计就是为了实现jwt的配置，然后，我们把从数据库找到的账户对象给了它，用来生成jwt三段数据中的中间那段的数据；

然后下面的就先构造第一段数据，并结合第二段数据做hash并加盐，最后使用baseurl加密，生成最终的token，并返回给前端，客户端，登陆成功。

#### 4.评论模块

接下来，我们先跳过文章，先走这个评论的路由：url(r'^comment/$',comment.CommentView.as_view()),在呼啦圈这个项目中，每篇文章的内容是匿名可见的，但是想要进行评论就必须登陆了，这个是我们的业务需求。

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.generics import ListAPIView,CreateAPIView
from rest_framework.filters import BaseFilterBackend

from api import models
from api.serializer.comment import CommentListSerializer,CommentCreateSerializer

class CommentFilterBackend(BaseFilterBackend):
    
    def filter_queryset(self, request, queryset, view):
        """
        Return a filtered queryset.
        """
        article_id = request.query_params.get('article',None)
        if not article_id:
            return queryset.none()
        return queryset.filter(article_id=article_id)

class CommentView(ListAPIView,CreateAPIView):
    """ 评论相关接口 """
    
    #必序定义的变量
    queryset = models.Comment.objects.all()

    serializer_class = None

    filter_backends = [CommentFilterBackend,]

    def get_authenticators(self):

        if self.request.method == "GET":
            return []
        elif self.request.method == 'POST':
            return super().get_authenticators()

    def perform_create(self, serializer):
        serializer.save(user=self.request.user)

    def get_serializer_class(self):
        """
        根据不同的需求，应用不同的序列化类，实现数据的自定制
        """
        if self.request.method == 'GET':
            return CommentListSerializer
        return CommentCreateSerializer
```

分析：先看这个CommentView类视图，因为该视图的业务设计数据的查看和添加以及删除，所以，为了方便，我们使用了进一步封装的ListAPICView等类。他里面封装了get(),post(),put(),delete()等方法就不许要我们再手写了。

他的原理就是，通过反url请求方法的不同使用反射的方式去执行继承的各种API类，进而执行xxModelMixin里面的方法并返回数据

```python
class ListAPIView(mixins.ListModelMixin,
                  GenericAPIView):
    """
    Concrete view for listing a queryset.
    """
    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
    
class ListModelMixin:
    """
    List a queryset.
    """
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
    
class GenericAPIView(views.APIView):

	# 你需要设置这些属性，或重写 `get_queryset()`/`get_serializer_class()`，如果要重写视图方法，调用不是直接访问“queryset”属性，而是“get_queryset（）”，因为“queryset”将只计算一次，并	且这些结果将被缓存所有后续请求。
    queryset = None
    serializer_class = None

	# 如果要使用pk以外的对象查找，请设置“lookup_field”。对于更复杂的查找要求，
    # 请重写“get_object()”。
    lookup_field = 'pk'
    lookup_url_kwarg = None

    # 用于queryset筛选的筛选器后端类
    filter_backends = api_settings.DEFAULT_FILTER_BACKENDS

    # 用于queryset分页的样式。
    pagination_class = api_settings.DEFAULT_PAGINATION_CLASS


    def get_serializer(self, *args, **kwargs):
        """
		返回应用于验证和反序列化输入和序列化输出。
        """
        serializer_class = self.get_serializer_class()
        kwargs['context'] = self.get_serializer_context()
        return serializer_class(*args, **kwargs)

    def get_serializer_class(self):
        """
		返回要用于序列化程序的类。默认使用“self.serializer_class`”类，如果需要提供不同的序列化			类，根据传入请求进行序列化。（例如，管理员获得完全序列化，其他人获得基本序列化）
        """
        assert self.serializer_class is not None, (
            "'%s' should either include a `serializer_class` attribute, "
            "or override the `get_serializer_class()` method."
            % self.__class__.__name__
        )

        return self.serializer_class

```

这里面的评论就很简单，只要获得是那一篇文章就可以展示出该篇文章的相关的评论；

由于ListAPIView继承ListModelMixin和GenericAPIView视图类，根据GenericAPIView视图类的定义，我们需要指定  queryset = None   serializer_class = None这两个变量，前面的是我们的queryset 对象，就是我们从数据库中取得的数据，后面的一个变量是我们使用的序列化类，但是我们这里的get和post使用的不是同一个序列化类，所以，我们就需要重写get_serializer_class()方法进行判断来指定序列化类的使用

```python
from rest_framework import serializers
from api import models


class CommentListSerializer(serializers.ModelSerializer):
    user = serializers.CharField(source='user.username')
    class Meta:
        model = models.Comment
        fields = "__all__"

class CommentCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Comment
        exclude = ['user']
```

从上面的两个不同的序列化类也可以看出来，他们序列化的内容是不一样的所以序要写不同的序列化类应用到不同的请求。

那么，现在有一个问题就是他的哪一篇文章的评论怎么去绑定呢？这就使用到了过滤，

```python
class CommentFilterBackend(BaseFilterBackend):
    def filter_queryset(self, request, queryset, view):
        """
        Return a filtered queryset.
        """
        article_id = request.query_params.get('article',None)
        if not article_id:
            #这种方式返回的是就是一个空的queryset对象
            return queryset.none()
        return queryset.filter(article_id=article_id)
    
    
class BaseFilterBackend:
    """
	所有筛选器子类都应从中继承的基类。
    """

    def filter_queryset(self, request, queryset, view):
        """
        返回一个过滤后的queryset
        """
        raise NotImplementedError(".filter_queryset() must be overridden.")

    def get_schema_fields(self, view):
        assert coreapi is not None, 'coreapi must be installed to use `get_schema_fields()`'
        assert coreschema is not None, 'coreschema must be installed to use `get_schema_fields()`'
        return []

class GenericAPIView(views.APIView):
        # 用于queryset筛选的筛选器后端类
    filter_backends = api_settings.DEFAULT_FILTER_BACKENDS
    
api_settings.py   
	'DEFAULT_FILTER_BACKENDS': [],
```

我们看上面的一段代码，在GenericAPIView里面，filter_backends会过滤类，没有的会就去自己的settings里面找，最终找到一个空列表，也就是说不对数据进行过滤，因此，我们在这里写一个过滤类，并实现BaseFilterBackend里面的filter_queryset过滤方法（这里的过滤条件是根据前端返回的url后面的参数article的id来进行过滤判断的条件，如果没有就返回一个空的queryset对象，否则就把从后台取的所有的评论按照article_id的条件进行过滤，并返回一个queryset对象），实现过滤。

现在，我们的评论数据展示完成，只有我们的登陆之后创建和删除评论了，那么，咱们第一步就得先做登陆认证吧，

```python
authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES

class APIView(View):
	def get_authenticators(self):
        """
		实例化并返回此视图可以使用的身份验证程序列表。
        """
        return [auth() for auth in self.authentication_classes]
    
    
#我们的settings
    # 认证配置
    "DEFAULT_AUTHENTICATION_CLASSES": ["api.extensions.auth.HulaQueryParamAuthentication",],
```

我们在这里可以看出，从写get_authenticators方法进行get和post的区分，咱们的类视图里面重构的该方法针对get返回的是一个空的列表，针对post请求，他就去重构父类的get_authenticators方法，在父类里面他就会去调用我们自己setting配置文件里面的DEFAULT_AUTHENTICATION_CLASSES，从而执行auth.HulaQueryParamAuthentication对用户进行认证，现在，我们看看，在他里面都做了哪些事情：

```python
import jwt
from rest_framework import exceptions
from rest_framework.authentication import BaseAuthentication
from rest_framework_jwt.settings import api_settings
from api import models


class HulaQueryParamAuthentication(BaseAuthentication):
	def authenticate(self, request):
		"""
		# raise Exception(), 不在继续往下执行，直接返回给用户。
		# return None ,本次认证完成，执行下一个认证
		# return ('x',"x")，认证成功，不需要再继续执行其他认证了，继续往后权限、节流、视图函数
		"""
		token = request.query_params.get('token')
		if not token:
			raise exceptions.AuthenticationFailed({'code': 10002, 'error': "登录成功之后才能操作"})

		jwt_decode_handler = api_settings.JWT_DECODE_HANDLER
		try:
			payload = jwt_decode_handler(token)
        #token过期
		except jwt.ExpiredSignature:
			raise exceptions.AuthenticationFailed({'code': 10003, 'error': "token已过期"})
        #token值被修改
		except jwt.DecodeError:
			raise exceptions.AuthenticationFailed({'code': 10004, 'error': "token格式错误"})
        #认证失败
		except jwt.InvalidTokenError:
			raise exceptions.AuthenticationFailed({'code': 10005, 'error': "认证失败"})

		jwt_get_username_from_payload = api_settings.JWT_PAYLOAD_GET_USERNAME_HANDLER
		username = jwt_get_username_from_payload(payload)
		user_object = models.UserInfo.objects.filter(username=username).first()
		return (user_object, token)

    
class BaseAuthentication:
    """
    所有身份验证类都应扩展BaseAuthentication。
    """

    def authenticate(self, request):
        """
        验证请求并返回两个参数的元组（user，token）。
        """
        raise NotImplementedError(".authenticate() must be overridden.")

    def authenticate_header(self, request):
        """
        Return a string to be used as the value of the `WWW-Authenticate`
        header in a `401 Unauthenticated` response, or `None` if the
        authentication scheme should return `403 Permission Denied` responses.
        """
        pass

```

我们之前的认证都是基于uuid随机产生的一个字符串，现在，我们不再使用那种low的方式，我们使用jwt的方式，我们获取前端返回的toekn（这个token就是我们之前在登陆的时候产生的jwt的字符串并传给前端的），现在判断，如果他不存在，那么肯定就没有登陆，我们自定义错误抛出异常。

接着，我们对这个字符串进行反解析，确定各种错误之后，我们在考虑我们的返回值，(user,auth)，user自然就是我们的用户对象，auth就是这个token值。

总结上面的认证就是返回三种情况，返回None，没有完成认证接着完成下一个认证，抛出异常结束，正常返回元组，认证完成。

最后，我们发现，评论视图里面只剩下一个

```python
  def perform_create(self, serializer):
        serializer.save(user=self.request.user)
```

，这个其实就是保存我们的数据的，源码里面也就只有这save()这一行代码，在这里从写的含义就是我们想在保存数据的时候再多加一点内容，那就是用户名，知此，我们的评论的创建和删除也完成了。

#### 5.文章模块

文章展示的逻辑主要是分为两大块的内容，一个是主页的信息的展示；一个是详情页信息的展示。那么我们跟着路由来做。

我们先看路由

```python
    url(r'^article/$',article.ArticleView.as_view()),
```

这是个不带参数的路由，我们先进入走进这个ArticleView视图类

```python
from api.serializer.article import ArticleSerializer

class ArticleView(ListAPIView):
	authentication_classes = []
	queryset = models.Article.objects.all()
	serializer_class = ArticleSerializer
```

```python
class ArticleSerializer(serializers.ModelSerializer):
    author = serializers.CharField(source='author.username')
    category = serializers.CharField(source='get_category_display')
    date = serializers.SerializerMethodField()
    class Meta:
        model = models.Article
        fields = "__all__"

    def get_date(self,obj):
        return obj.date.strftime("%Y-%m-%d %H:%M:%S") if obj.date else ""
```

看见这个视图类发现里面并没有什么，首先我们先看到的是authentication_classes=[],看这个变量的名字是不是很熟悉，它就是和认证相关的，但是，从我们的路由来看它是一个没有参数的，所以这是首页展示的页面。那么这个页面是所有人都可见的，所以就不需要认证，所以，我们把上面的列表设置为空就可以不使用认证（因为认证在全局配置了）

接着就是我们很熟悉的取数据库的信息，然后定义一个序列化类来展示需要的数据。在序列化类里面，我们需要反向查表取得文章作者的信息（使用小写的表名），数据类型的自定制I也可以写一个SerializerMethodField，定义一个钩子来写自己想要的格式（比如时间信息的展示）

现在，首页的内容算是gank完了（但是分类的没有处理，先欠着），接着就是我们带参数的路由

```python
url(r'^article/(?P<pk>\d+)/$',article.ArticleDetailView.as_view())
```

套路还是一样，看我们的视图类

```python
from django.db.models import F
from api import models
from api.serializer.article import ArticleDetailSerializer


class ArticleDetailView(RetrieveAPIView):
	authentication_classes = []
	queryset = models.Article.objects.all()
	serializer_class = ArticleDetailSerializer

	def get(self, request, *args, **kwargs):
		# 1.获取单挑数据再做序列化
		result = super().get(request, *args, **kwargs)
		# 2.对浏览器进行自加1
		pk = kwargs.get('pk')
		models.Article.objects.filter(pk=pk).update(read_count=F('read_count') + 1)

		return result
    
class ArticleDetailSerializer(serializers.ModelSerializer):
    author = serializers.CharField(source='author.username')
    category = serializers.CharField(source='get_category_display')
    date = serializers.SerializerMethodField()
    content = serializers.CharField(source='articledetail.content')
    class Meta:
        model = models.Article
        fields = "__all__"

    def get_date(self,obj):
        return obj.date.strftime("%Y-%m-%d %H:%M:%S") if obj.date else ""
```

对于单篇文章的业务逻辑就是，就是展示页面详情的时候，对文章的阅读数进行增加1的操作。看到这个我们很清楚的就知道了这是跳过认证；

下面的取数据和和序列化和我们前面的首页的展示是一样的，不在赘述；

这里，我们重写了get方法，因为，我们需要取部分的数据，首先我们就重构了父类的get()方法（切记，重写源码的时候一定记得重构），因为我们继承的是RetrieveAPIView，它是一个展示单页数据的类，所以，这里就不需要我们做什么了；

下面对文章的浏览量进行I加1操作，就是从我们的url里面传来的参数，使用F（一个对数据库数据进行加减*/运算的方法）进行数据库数据的+1，完成

#### 6.扩展（文章模块使用ListModelMixin实现）

路由

```python
 	url(r'^article/$',article.ArticleView.as_view({"get":'list'})),
    url(r'^article/(?P<pk>\d+)/$',article.ArticleView.as_view({'get':'retrieve'}))

```

我们发现，路由有所变化，在as_view()里面有参数了，它是一个字典，键就是我们的方法，值是我们锁需要做的操作（list：展示所有的数据，retrieve：展示单页的数据，create：新建数据.......）,那接下来看看我们的视图是怎么操作的(我们直接在代码上面分析)

```python
from rest_framework.viewsets import GenericViewSet
#继承的类也是不一样的
from rest_framework.mixins import ListModelMixin,RetrieveModelMixin,CreateModelMixin,UpdateModelMixin,DestroyModelMixin
from api.serializer.article import ArticleSerializer,ArticleDetailSerializer

class ArticleView(GenericViewSet,ListModelMixin,RetrieveModelMixin):
    authentication_classes = []
	#需要写的两个变量
    queryset = models.Article.objects.all()
    serializer_class = None

    def get_serializer_class(self):
        #上面的serializer_class无效，自定制使用序列化类，这里需要对数据进行筛选
        pk = self.kwargs.get('pk')
        if pk:
            #这里就很巧妙了，我们前面使用的那种方式在展示首页和详情页的数据的时候需要写两个不同的视图类，这里就需要写一个，那么怎么区分？这里就使用url里面是否有pk值来进行不同的序列化
            return ArticleDetailSerializer
        return ArticleSerializer

    def create(self,request,*args,**kwargs):
        pass
```

#### setting.py

```python
REST_FRAMEWORK = {
    # 版本配置
    "DEFAULT_VERSIONING_CLASS":"rest_framework.versioning.URLPathVersioning",
    "ALLOWED_VERSIONS":['v1','v2'],

    # 认证配置
    "DEFAULT_AUTHENTICATION_CLASSES": ["api.extensions.auth.HulaQueryParamAuthentication",],
    "UNAUTHENTICATED_USER":None,
    "UNAUTHENTICATED_TOKEN":None,

    # 节流配置
    "DEFAULT_THROTTLE_RATES": {
            "anon": "10/m",
        },
    # 分页配置
    "DEFAULT_PAGINATION_CLASS":"rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE":10
}
import datetime
JWT_AUTH = {
    "JWT_EXPIRATION_DELTA":datetime.timedelta(minutes=10)
}
```

