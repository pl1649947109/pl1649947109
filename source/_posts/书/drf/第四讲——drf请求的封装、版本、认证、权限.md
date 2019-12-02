---
title: 第四讲——drf请求的封装、版本、认证、权限
id: 5
date: 2019-11-8 20:00:00
tags: drf
comment: true
---

### 回顾和补充

1.restful规范

```python
1. 建议用https代替http

2. 在URL中体现api，添加api标识
        https://www.cnblogs.com/xwgblog/p/11812244.html   # 错误
        https://www.cnblogs.com/api/xwgblog/p/11812244.html  # 正确（根域名）
        https://api.cnblogs.com/xwgblog/p/11812244.html # 正确（子域方式）

        建议使用：https://www.cnblogs.com/api/...
        
3. 在URL中要体现版本（版本可以体在在多个地方，一般就体现在url中）
	https://www.cnblogs.com/api/v1/userinfo/
	https://www.cnblogs.com/api/v2/userinfo/
	
4. URL中一般用名词： 
	http://www.luffycity.com/api/v1/article/ (面向资源编程，网络上东西都视为资源)

5. 筛选条件，在URL参数中进行传递：
	http://www.luffycity.com/article/page=1&category=1
	
6. 根据请求不同做不同操作：GET/POST/PUT/DELTE/PATCH

7. 一般传输的数据格式都是JSON
```

<!----more---->

2.drf组件为我们提供了哪些功能

```
首先，drf是dajngo框架为我们提供的一个组件，它的本质是一个app，它可以依据restful规范快速开发为我们开发一套应用程序。
然后，他为我么做了这些：用户发来请求，请求走我们的url，在进入视图先执行dispacth，这里我们进行了版本控制、用户认证以及权限认证和频率限制。然后，drf为我们提供了解析器、序列化、分页、筛选、视图、渲染器的一些功能。其中，我们常用的是序列化、分页、筛选和视图的功能，其他的我们只需要一次配置就可以了。
```

3.类继承关系（各个组件之间的继承关系）

```python

class View(object):
	def dipatch(self):
		print(123)
	
class APIView(View):
    """
    该模块是都是一些公共的功能模块
    """
    #这里定义的都是一些变量，这里变量都是链接到settings里面的，我们的功能配置参数搜可以来这里查
    version_class = settings.xxx 
	parser_class = settings.sxx
	permision_classes = []
    
	def dipatch(self):

        #在执行么的反射之前先执行这个方法
        self.initial()
        
        """和我们之前的View一样，在执行class里面的函数之前，先执行该函数，在这里面根据我们的请求通过反射找方法并执行
        """
		method = getattr(self,"get")
		return method()
    
    def initial(self):
        #这个方法里面其实执行的就是版本控制和认证和权限的功能
		self.version_class()
		self.parser_class()
		for item in self.permision_classes:
		 	item()
	
class GenericAPIView(APIView):
    """
    这个class里面实现的是一些方法和放置一些默认的变量
    它也是一个放置公共模块和变量的类，但是它提供给的都是
    和增删改查相关的类，我们下面的增删改查的类都会继承该类
    """
	queryset = None
	serilizer_class = None 
	def get_queryset(self):
		return self.queryset
		
	def get_serilizer(self,*arg,**kwargs):
		cls = self.get_serilizer_class()
		return cls(*arg,**kwargs)
	
	def get_serilizer_class(self):
		return self.serilizer_class
    
class ListModelMixin(object):
    """
    这个是我们的和增删改查相关的功能类，我们的下面的每一个功能类
    都会继承每一个对应的该类，通过他们去找 最下面的子类里面的变量或者GenericAPIView变量和
    最下面的子类里面的方法或者GenericAPIVie里面的方法，然后返回数据给请求者
    """
	def list(self):
		queryset = self.get_queryset()
		ser = self.get_serilizer(queryset,many=True)
		return Reponse(ser.data)
		
class ListAPIView(ListModelMixin,GenericAPIView):
    """
    我们的功能类，每一个类对应一个增删查改的功能，这就是个入口，通过它去执行
    xxxxModelMixin里面的方法
    """
	def get(self):
		return self.list(...)
	
	
class TagView(ListAPIView):
    """
    我们的实例，在这里我们配置相关的变量就可以执行一系列的操作，我们也可以通过重构父类方法
    来自定制我们需要的功能，（记好：父类中所有的查找都是重这个类开始的）
    """
	queryset = models.User.object.all()
	serilizer_class = TagSerilizer
	
obj = TagView()
x = obj.dispatch()
```

### 请求的封装

```python
class HttpRequest(object):
	def __init__(self):
		pass
	
	@propery
	def GET(self):
		pass
	
	@propery
	def POST(self):
		pass
	
	@propery
	def body(self):
		pass

class Request(object):
	def __init__(self,request):
		self._request = request
	
	def data(self):
        """
        封装了request.POST=data
        """
		if content-type == "application/json"
			reutrn json.loads(self._request.body.decode('urf-8'))
		elif content-type == "x-www-...":
			return self._request.POST
		
	def query_params(self):
        """
        封装了request.GET=query_params
        """
		return self._reqeust.GET
```

视图

```python
class View(object):
	@classonlymethod
    def as_view(cls, **initkwargs):
        def view(request, *args, **kwargs):
            return self.dispatch(request, *args, **kwargs)
        return view
	
class APIView(View):
	
	@classmethod
    def as_view(cls, **initkwargs):
        view = super().as_view(**initkwargs)
        return csrf_exempt(view)
    
	def dispatch(self, request, *args, **kwargs):
        # 新request内部包含老request(_reuqest=老request)
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        
        self.initial(request, *args, **kwargs)
        
        # 通过反射执行“get”方法，并传入新的request
        handler = getattr(self, request.method.lower())
        response = handler(request, *args, **kwargs) # get(requst)
        return self.response
    
class OrderView(APIView):
	
    def get(self,request,*args,**kwargs):
        return Response('成功')
```

![](http://9017499461.linshutu.top/%E8%AF%B7%E6%B1%82%E5%B0%81%E8%A3%85.JPG)

### 版本

#### 源码

```python
class APIView(View):
    versioning_class = api_settings.DEFAULT_VERSIONING_CLASS
    
	def dispatch(self, request, *args, **kwargs):
       
        # ###################### 第一步 ###########################
        """
        request,是django的request，它的内部有：request.GET/request.POST/request.method
        args,kwargs是在路由中匹配到的参数，如：
            url(r'^order/(\d+)/(?P<version>\w+)/$', views.OrderView.as_view()),
            http://www.xxx.com/order/1/v2/
        """
        self.args = args
        self.kwargs = kwargs

        """
        request = 生成了一个新的request对象，此对象的内部封装了一些值。
        request = Request(request)
            - 内部封装了 _request = 老的request
        """
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request

        self.headers = self.default_response_headers  # deprecate?

        try:
            # ###################### 第二步 ###########################
            self.initial(request, *args, **kwargs)

            执行视图函数。。
	
	def initial(self, request, *args, **kwargs):
       
        # ############### 2.1 处理drf的版本 ##############
        version, scheme = self.determine_version(request, *args, **kwargs)
        request.version, request.versioning_scheme = version, scheme
		...
        
    def determine_version(self, request, *args, **kwargs):
        if self.versioning_class is None:
            return (None, None)
        scheme = self.versioning_class() # obj = XXXXXXXXXXXX()
        return (scheme.determine_version(request, *args, **kwargs), scheme)
        
class OrderView(APIView):
    versioning_class = URLPathVersioning
    def get(self,request,*args,**kwargs):
        print(request.version)
        print(request.versioning_scheme)
        return Response('...')

    def post(self,request,*args,**kwargs):
        return Response('post')
```

```python
class URLPathVersioning(BaseVersioning):
    """
    urlpatterns = [
        url(r'^(?P<version>[v1|v2]+)/users/$', users_list, name='users-list'),
       
    ]
    """
    invalid_version_message = _('Invalid version in URL path.')

    def determine_version(self, request, *args, **kwargs):
        version = kwargs.get(self.version_param, self.default_version)
        if version is None:
            version = self.default_version

        if not self.is_allowed_version(version):
            raise exceptions.NotFound(self.invalid_version_message)
        return version

```

#### 使用（局部）

- url中写version

  ```python
  url(r'^(?P<version>[v1|v2]+)/users/$', users_list, name='users-list'),
  ```

- 在视图中应用

  ```python
  from rest_framework.views import APIView
  from rest_framework.response import Response
  from rest_framework.request import Request
  from rest_framework.versioning import URLPathVersioning
  
  
  class OrderView(APIView):
  	#在需要的版本类里面定义该变量
      versioning_class = URLPathVersioning
      
      def get(self,request,*args,**kwargs):
          print(request.version)
          print(request.versioning_scheme)
          return Response('...')
  
      def post(self,request,*args,**kwargs):
          return Response('post')
  ```

- 在settings中配置

  ```python
  REST_FRAMEWORK = {
      "PAGE_SIZE":2,
      "DEFAULT_PAGINATION_CLASS":"rest_framework.pagination.PageNumberPagination",
      #配置项，设置版本
      "ALLOWED_VERSIONS":['v1','v2'],
      'VERSION_PARAM':'version'
  }
  ```

#### 使用（全局）推荐

- url中写version

  ```python
  url(r'^(?P<version>[v1|v2]+)/users/$', users_list, name='users-list'),
  
  url(r'^(?P<version>\w+)/users/$', users_list, name='users-list'),
  ```

- 在视图中应用

  ```python
  from rest_framework.views import APIView
  from rest_framework.response import Response
  from rest_framework.request import Request
  from rest_framework.versioning import URLPathVersioning
  
  
  class OrderView(APIView):
      def get(self,request,*args,**kwargs):
          print(request.version)
          print(request.versioning_scheme)
          return Response('...')
  
      def post(self,request,*args,**kwargs):
          return Response('post')
  ```

- 在settings中配置

  ```python
  REST_FRAMEWORK = {
      "PAGE_SIZE":2,
      "DEFAULT_PAGINATION_CLASS":"rest_framework.pagination.PageNumberPagination",
     
   #配置项，应用全局   "DEFAULT_VERSIONING_CLASS":"rest_framework.versioning.URLPathVersioning",
      "ALLOWED_VERSIONS":['v1','v2'],
      'VERSION_PARAM':'version'
  }
  ```

### 认证（***）

#### 源码

```python
class Request:

    def __init__(self, request,authenticators=None):
        self._request = request
        self.authenticators = authenticators or ()
        
	@property
    def user(self):
        """
        Returns the user associated with the current request, as authenticated
        by the authentication classes provided to the request.
        """
        if not hasattr(self, '_user'):
            with wrap_attributeerrors():
                self._authenticate()
        return self._user
    
    def _authenticate(self):
        """
        Attempt to authenticate the request using each authentication instance
        in turn.
        """
        for authenticator in self.authenticators:
            try:
                user_auth_tuple = authenticator.authenticate(self)
            except exceptions.APIException:
                self._not_authenticated()
                raise

            if user_auth_tuple is not None:
                self._authenticator = authenticator
                self.user, self.auth = user_auth_tuple
                return

        self._not_authenticated()
        
	@user.setter
    def user(self, value):
        """
        Sets the user on the current request. This is necessary to maintain
        compatibility with django.contrib.auth where the user property is
        set in the login and logout functions.

        Note that we also set the user on Django's underlying `HttpRequest`
        instance, ensuring that it is available to any middleware in the stack.
        """
        self._user = value
        self._request.user = value
```

使用继承

```python

class APIView(View):
    authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES
    
	def dispatch(self, request, *args, **kwargs):
        """
        `.dispatch()` is pretty much the same as Django's regular dispatch,
        but with extra hooks for startup, finalize, and exception handling.
        """
        # ###################### 第一步 ###########################
        """
        request,是django的request，它的内部有：request.GET/request.POST/request.method
        args,kwargs是在路由中匹配到的参数，如：
            url(r'^order/(\d+)/(?P<version>\w+)/$', views.OrderView.as_view()),
            http://www.xxx.com/order/1/v2/
        """
        self.args = args
        self.kwargs = kwargs


        """
        request = 生成了一个新的request对象，此对象的内部封装了一些值。
        request = Request(request)
            - 内部封装了 _request = 老的request
            - 内部封装了 authenticators = [MyAuthentication(), ]
        """
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
	
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

    def get_authenticators(self):
        """
        Instantiates and returns the list of authenticators that this view can use.
        """
        return [ auth() for auth in self.authentication_classes ]
    
class LoginView(APIView):
    authentication_classes = []
    def post(self,request,*args,**kwargs):
        user_object = models.UserInfo.objects.filter(**request.data).first()
        if not user_object:
            return Response('登录失败')
        random_string = str(uuid.uuid4())
        user_object.token = random_string
        user_object.save()
        return Response(random_string)

class OrderView(APIView):
    # authentication_classes = [TokenAuthentication, ]
    def get(self,request,*args,**kwargs):
        print(request.user)
        print(request.auth)
        if request.user:
            return Response('order')
        return Response('滚')

class UserView(APIView):
    同上
```

#### 使用

定义认证方法

```python
class MyAuthentication:
    def authenticate(self, request):
        """
        Authenticate the request and return a two-tuple of (user, token).
        """
        token = request.query_params.get('token')
        user_object = models.UserInfo.objects.filter(token=token).first()
        if user_object:
            return (user_object,token)
        return (None,None)
```

视图

```python
class LoginView(APIView):

    def post(self,request,*args,**kwargs):
        user_object = models.UserInfo.objects.filter(**request.data).first()
        if not user_object:
            return Response('登录失败')
        random_string = str(uuid.uuid4())
        user_object.token = random_string
        user_object.save()
        return Response(random_string)
```

全局使用

```python
"DEFAULT_AUTHENTICATION_CLASSES":["api.auth.authentications.MyAuthentication",],
```

局部使用

```python
class LoginView(APIView):
	#在使用的类里面设置空列表
	"DEFAULT_AUTHENTICATION_CLASSES" = []

    def post(self,request,*args,**kwargs):
        user_object = models.UserInfo.objects.filter(**request.data).first()
        if not user_object:
            return Response('登录失败')
        random_string = str(uuid.uuid4())
        user_object.token = random_string
        user_object.save()
        return Response(random_string)
```

### 权限

#### 源码

```python
class APIView(View):
    permission_classes = api_settings.DEFAULT_PERMISSION_CLASSES
    
    def dispatch(self, request, *args, **kwargs):
        封装request对象
        self.initial(request, *args, **kwargs)
        通过反射执行视图中的方法

	def initial(self, request, *args, **kwargs):
        版本的处理
        # 认证
        self.perform_authentication(request)
		
        # 权限判断
        self.check_permissions(request)
        
        self.check_throttles(request)
        
    def perform_authentication(self, request):
        request.user
	
    def check_permissions(self, request):
        # [对象,对象，]
        for permission in self.get_permissions():
            if not permission.has_permission(request, self):
                self.permission_denied(request, message=getattr(permission, 'message', None))
    def permission_denied(self, request, message=None):
        if request.authenticators and not request.successful_authenticator:
            raise exceptions.NotAuthenticated()
        raise exceptions.PermissionDenied(detail=message)
        
    def get_permissions(self):
        return [permission() for permission in self.permission_classes]
    
class UserView(APIView):
    permission_classes = [MyPermission, ]
    
    def get(self,request,*args,**kwargs):
        return Response('user')
```

#### 使用

```python
from rest_framework.permissions import BasePermission
from rest_framework import exceptions

class MyPermission(BasePermission):
    #自定义错误信息
    message = {'code': 10001, 'error': '你没权限'}
    def has_permission(self, request, view):
        """
        Return `True` if permission is granted, `False` otherwise.
        """
        if request.user:
            return True

        # raise exceptions.PermissionDenied({'code': 10001, 'error': '你没权限'})
        return False

    def has_object_permission(self, request, view, obj):
        """
        Return `True` if permission is granted, `False` otherwise.
        """
        return False
```

```python
class OrderView(APIView):
    #配置变量
    permission_classes = [MyPermission,]
    def get(self,request,*args,**kwargs):
        return Response('order')

class UserView(APIView):
    permission_classes = [MyPermission, ]
    def get(self,request,*args,**kwargs):
        return Response('user')

```

```python
REST_FRAMEWORK = {
    "PAGE_SIZE":2,
    "DEFAULT_PAGINATION_CLASS":"rest_framework.pagination.PageNumberPagination",
    "DEFAULT_VERSIONING_CLASS":"rest_framework.versioning.URLPathVersioning",
    "ALLOWED_VERSIONS":['v1','v2'],
    'VERSION_PARAM':'version',
    #权限配置
    "DEFAULT_AUTHENTICATION_CLASSES":["kka.auth.TokenAuthentication",]
}
```



#### 

