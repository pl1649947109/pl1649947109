---
title: 第七讲——扩展：drf中版本、认证、权限、频率限制源码分析
id: 8
date: 2019-11-19 20:00:00
tags: drf
comment: true
---

文章导读：在这里我们将分为两部分来讲解:第一部分，我们将随着程序执行的顺序一步步的去分析源码(看源码不可跳读，必须一步步的走)；第二部分，我们使用一个实例完整的把这个使用流程展示出来。

第一部分：源码的讲解

路由

```python
from .views import account
urlpatterns = [
    url(r'^login/$',account.LoginView.as_view()),
```

视图类

```python
from rest_framework.views import APIView

class LoginView(APIView):
	"""
	CBV视图类
	"""
    pass
```

下面，我们将开始分析，**login路由**进入查找到我们的**account.LoginView**，接着执行LoginView视图类里面的**as_view()**视图类，在该类里面找不到，接着找他的父类，如后就到**APIView**父类里面查找，

<!----more---->

```python
class APIView(View):
	@classmethod
    def as_view(cls, **initkwargs):
        view = super().as_view(**initkwargs)

        return csrf_exempt(view)
```

我们发现，在该视图类里面，它重构了**父类View**类的**as_view()**方法，我们接着找它父类的该方法，看看里面做了什么，

```python
class View(object):
	@classonlymethod
    def as_view(cls, **initkwargs):
        def view(request, *args, **kwargs):
            return self.dispatch(request, *args, **kwargs)
        return view
```

在View视图里面，我们发现它使用了开发过封闭原则执行了**dispatch()**方法，接着，我们开始从最开始的LoginView类往上找，最后我们在它的亲类APIView里面找到了，我们看看它里面都执行了什么

```python
    def dispatch(self, request, *args, **kwargs):
        """
        它里面的钩子用于启动、完成和异常处理。
        """
        self.args = args
        self.kwargs = kwargs
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        self.headers = self.default_response_headers

        try:
            self.initial(request, *args, **kwargs)

            # Get the appropriate handler method
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed

            response = handler(request, *args, **kwargs)

        except Exception as exc:
            response = self.handle_exception(exc)

        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response
```

执行上面的方法，首先是对**request进行封装**，执行**initialize_request**，在initialize_request里面做了什么，我们来看看

```python
def initialize_request(self, request, *args, **kwargs):

    parser_context = self.get_parser_context(request)

    return Request(
        request,
        parsers=self.get_parsers(),
        #按这个名字是不是和我们的认证有关系
        authenticators=self.get_authenticators(),
        negotiator=self.get_content_negotiator(),
        parser_context=parser_context
    )
    
def get_authenticators(self):

    return [auth() for auth in self.authentication_classes]

#需要我们自己设置的，否则就使用默认的None值
authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES
```

从上面的方法里面，我们发现了，新的request里面封装了更多的内容，其中一项就是我们的**实例化认证类列表**，我们后面会讲到，接下来，我们执行try里面的**initial**

```python
def initial(self, request, *args, **kwargs):

    # 执行版本相关的方法
    version, scheme = self.determine_version(request, *args, **kwargs)
    request.version, request.versioning_scheme = version, scheme

    # 认证相关
    self.perform_authentication(request)
    # 权限相关
    self.check_permissions(request)
    # 频率限制相关
    self.check_throttles(request)
```

在这个函数里，就是我们真正需要关注的内容了，我在里面进行了简单的标注，下面，我们将一个个的分析它们：

版本：

执行**determine_version**

```python
def determine_version(self, request, *args, **kwargs):
    """
    If versioning is being used, then determine any API version for the
    incoming request. Returns a two-tuple of (version, versioning_scheme)
    """
    if self.versioning_class is None:
        return (None, None)
    scheme = self.versioning_class()
    return (scheme.determine_version(request, *args, **kwargs), scheme)

#需要我们自己设置的，否则就使用默认的None值
versioning_class = api_settings.DEFAULT_VERSIONING_CLASS
```

如果我们的**settings**里面没有设置这个**versioning_class**的值，那么程序就会去**api_settings**里面找，返回的就是一个**(None,None)**的元组，否则就会把我们定义的实例化类赋值给**scheme**，并返回一个包含我们定义的实例化类的方法和该类**(selfversioning_class().determine_version,self.versioning_class())**

```python
request.version, request.versioning_scheme = version, scheme
```

最后，把版本的信息封装进了**request**

认证：

执行**perform_authentication**

```python
def perform_authentication(self, request):
    request.user
```

执行**request**里面封装的**user**（这个看着user是一个属性对不对，但是既然这里什么没有return，那么它就肯定不是一个属性，而是一个方法，只是伪装成了属性而已），

```python
@property
def auth(self):
    if not hasattr(self, '_user'):
        with wrap_attributeerrors():
            self._authenticate()
    return self._user

def _authenticate(self):
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
```

这里的**_authenticate()**就是重点，它里面执行的就是我们之前在**initialize_request**里面封装的我们自定义的功能类，这里执行并返回一个元组，这里返回的是三种情况，抛出异常，认证失败，程序退出；返回None，接着执行下一个认证；返回一个包含用户信息和token值的元组，表示认证成功。

权限：

执行**check_permissions**

```python
def check_permissions(self, request):
    for permission in self.get_permissions():
        if not permission.has_permission(request, self):
            self.permission_denied(
                request, message=getattr(permission, 'message', None)
            )
def get_permissions(self):
    return [permission() for permission in self.permission_classes]

#需要我们自己设置的，否则就使用默认的None值
permission_classes = api_settings.DEFAULT_PERMISSION_CLASSES
```

在权限里面，首先循环了**get_permissions()**方法，这个方法就去配置文件里面找我们自定义的**实例化的权限类**，并进行封装到一个列表里面，接着，循环我们自定义的实例化类，执行实例化里面的**has_permission()**方法，这就是我们需要在自定义类里面写的方法，他返回的是一个True（表示拥有访问的权限），该方法执行完成。

否则，执行**permission_denied()**方法

```python
def permission_denied(self, request, message=None):

    if request.authenticators and not request.successful_authenticator:
        raise exceptions.NotAuthenticated()
        raise exceptions.PermissionDenied(detail=message)
```

我们就可以很清楚的看出来，当权限不足，就是没有访问的权限的时候就会抛出异常。

频率限制：

在讲频率限制之前，我们先说一下它实现的原理，这样会帮助我们去理解源码，

```
DRF中的频率控制基本原理是基于访问次数和时间的，当然我们可以通过自己定义的方法来实现。

当我们请求进来，走到我们频率组件的时候，DRF内部会有一个字典来记录访问者的IP，或者是登录用户的相关信息

这里，当匿名用户的IP为key，value为一个列表，存放访问者每次访问的时间，

{  IP1: [第三次访问时间，第二次访问时间，第一次访问时间]，}

把每次访问最新时间放入列表的最前面，记录这样一个数据结构后，通过什么方式限流呢~~

如果我们设置的是10秒内只能访问5次，

　　-- 1，判断访问者的IP是否在这个请求IP的字典里（如果是登录用户，我们可以根据用户名等信息）

　　-- 2，保证这个列表里都是最近10秒内的访问的时间

　　　　　　判断当前请求时间和列表里最早的(也就是最后的一个的)请求时间的差

　　　　　　如果差大于10秒，说明请求已经不是最近10秒内的，删除掉，

　　　　　　继续判断倒数第二个，直到差值小于10秒

　　-- 3，判断列表的长度(即访问次数)，是否大于我们设置的5次，

　　　　　　如果大于就限流，否则放行，并把时间放入列表的最前面。
第二步：
我们以一个列表来说明:
	s = {'IP':[1:20'09'',1:20'08'',1:20'07'',1:20'06'',1:20'02'',1:19'50'']}
现在，我们又这个IP又发来一次请求，时间为1:20'10'',
我们使用1:20'10''- 1:19'50''>10,发现，这个时间过期了，我们就s['IP'].pop()
	现在s = {'IP':[1:20'09'',1:20'08'',1:20'07'',1:20'06'',1:20'02'']}
我们再比较1:20'10'' - 1:20'02''<10,说明这个时间没有过期，好我们就把1:20'10''这个时间插入到上面的这个列表里面
	于是s = {'IP':[1:20'10'',1:20'09'',1:20'07'',1:20'02'']}

第三步：
现在，我们再来判断上面的列表的长度是不是>5，满足表示需要限流了，就不让它访问，否则它就可以访问。最后发现它还可以访问，就是这样的原理流程
```

执行**check_throttles**

```python
def check_throttles(self, request):

    throttle_durations = []
    for throttle in self.get_throttles():
        if not throttle.allow_request(request, self):
            throttle_durations.append(throttle.wait())

            if throttle_durations:
                durations = [
                    duration for duration in throttle_durations
                    if duration is not None
                ]

                duration = max(durations, default=None)
                self.throttled(request, duration)
                
def get_throttles(self):
	return [throttle() for throttle in self.throttle_classes]

#需要我们自己设置的，否则就使用默认的None值
throttle_classes = api_settings.DEFAULT_THROTTLE_CLASSES
```

和前面的几种情况一样，第一步都是去生成一个**实例化限流类的列表**，并循环执行，接着就执行实例化类里面的**allow_request()**方法，这里我们就拿**SimpleRateThrottle**里面的该方法来说明

```python
def allow_request(self, request, view):
	
    #判断请求的方式是不是字符串的形式，确定频率限制定义的格式
    if self.rate is None:
        return True
	
    #这里就是上面步骤的第一步
    #必须在我们自己定义类里面重写该方法，如果限制的话，判断用户请求（返回匿名用户IP，登录用户的user之类的）是否满足返回之，否则就不限制，直接返回Non（这里是None返回的是True）
    self.key = self.get_cache_key(request, view)
    if self.key is None:
        return True

    #拿缓存中的数据，就是我们第二步里面的列表，当缓存中没有的话默认设置为空列表
    self.history = self.cache.get(self.key, [])
    #timer() = datetime.datetime()获取当前时间
    self.now = self.timer()

    #这里就是上面步骤的第二步
    #self.duration获取的是我们设置的时间间隔（10/minute），self.duration=60
    #剔除列表中过期的时间
    while self.history and self.history[-1] <= self.now - self.duration:
        self.history.pop()
        
    #num_requests我们的次数，这里是10
    #判断这个列表的长度是否超过10
    if len(self.history) >= self.num_requests:
        #长度超过了，执行该方法返回False
        return self.throttle_failure()
    #长度没有超过，表示没有限制，执行该方法返回True
    return self.throttle_success()


def get_cache_key(self, request, view):
    """
	应返回可用于限制的唯一缓存密钥。
	必须重写。

	如果不应限制请求，则可以返回“None”。
    """
    raise NotImplementedError('.get_cache_key() must be overridden')
```

执行完我们的方法，接着执行**throttle_durations.append(throttle.wait())**

```python
def wait(self):
        """
        返回下次建议的请求时间
        """
    if self.history:
        remaining_duration = self.duration - (self.now - self.history[-1])
    else:
        remaining_duration = self.duration

    available_requests = self.num_requests - len(self.history) + 1
    #表示num_requests长度为空,返回None
    if available_requests <= 0:
        return None
	
    #这个列表间隔时间/列表长度 = x次/秒（最终返回的是一个int）
    return remaining_duration / float(available_requests)
```

上面的返回的是建议我们下次请求的时间，接着就剩下下面的内容没执行了

```python
if throttle_durations:
    durations = [
        duration for duration in throttle_durations
        if duration is not None
    ]

    duration = max(durations, default=None)
    self.throttled(request, duration)
```

我们分析，当我们定义的列表不为空的时候，我们循环遍历该列表**throttle_durations**剔除**None**值

，这种取出**durations**中最大的值，当作参数传递给**throttled()**并执行

```python
def throttled(self, request, wait):
    """
    如果请求被限制，请确定引发哪种异常。
    """
    raise exceptions.Throttled(wait)
    
    
class Throttled(APIException):
    status_code = status.HTTP_429_TOO_MANY_REQUESTS
    default_detail = _('Request was throttled.')
    extra_detail_singular = _('Expected available in {wait} second.')
    extra_detail_plural = _('Expected available in {wait} seconds.')
    default_code = 'throttled'

    def __init__(self, wait=None, detail=None, code=None):
        if detail is None:
            detail = force_str(self.default_detail)
        if wait is not None:
            wait = math.ceil(wait)
            detail = ' '.join((
                detail,
                force_str(ngettext(self.extra_detail_singular.format(wait=wait),
                                   self.extra_detail_plural.format(wait=wait),
                                   wait))))
        self.wait = wait
        super().__init__(detail, code)
```

至此，我们的截流也说完了。

第二部分：应用实例

配置文件部分settings.py

```python
REST_FRAMEWORK = {
    #版本控制
    "DEFAULT_VERSIONING_CLASS":"rest_framework.versioning.URLPathVersioning",
    "ALLOWED_VERSIONS":['v1','v2'],

    #认证
    "DEFAULT_AUTHENTICATION_CLASSES":["api.extensions.auth.HulaQueryParamAuthentication",],
    #"UNAUTHENTICATED_USER":None,
    #"UNAUTHENTICATED_TOKEN":None,

    #权限
	"DEFAULT_AUTHENTICATION_CLASSES":["kka.auth.TokenAuthentication",]
    
    #频率限制
    "DEFAULT_THROTTLE_RATES":{
        'anon': '5/minute',  #匿名用户
        'user': '10/minute', #登录用户
    },
}
```

自定义功能类

**版本**

不需要自定义类，再setting里面配置好就可以了

**认证**

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
            raise exceptions.AuthenticationFailed({'code':10002,'error':"登录成功之后才能操作"})

        jwt_decode_handler = api_settings.JWT_DECODE_HANDLER
        try:
            payload = jwt_decode_handler(token)
        except jwt.ExpiredSignature:
            raise exceptions.AuthenticationFailed({'code':10003,'error':"token已过期"})
        except jwt.DecodeError:
            raise exceptions.AuthenticationFailed({'code':10004,'error':"token格式错误"})
        except jwt.InvalidTokenError:
            raise exceptions.AuthenticationFailed({'code':10005,'error':"认证失败"})

        jwt_get_username_from_payload = api_settings.JWT_PAYLOAD_GET_USERNAME_HANDLER
        username = jwt_get_username_from_payload(payload)
        user_object = models.UserInfo.objects.filter(username=username).first()
        return (user_object,token)
```

**权限**

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

**截流**

```python
from rest_framework.throttling import SimpleRateThrottle

class AnonThrottle(SimpleRateThrottle):
    """
    截流：匿名用户使用IP
    """
	scope = 'anon'  # 相当于设置了最大的访问次数和时间

	def get_cache_key(self, request, view):
		if request.user:
			return None  # 返回None表示我不限制，登录用户我不管
			# 匿名用户
		return self.get_ident(request)  # 返回一个唯一标识IP


class UserThrottle(SimpleRateThrottle):
    """
    截流：登录用户使用用户名
    """
	scope = 'user'

	def get_cache_key(self, request, view):
		# 登录用户
		if request.user:
			return request.user
		return None  # 返回NOne表示匿名用户我不管
```

