---
title: 第五讲——drf请求的跨域、频率限制、jwt
id: 6
date: 2019-11-11 20:00:00
tags: drf
comment: true
---

## 内容回顾

1.restful规范？

<!----more---->

```
第一步：整体说restful规范是什么？

第二步：再详细说restful建议
	1. https代替http,保证数据传输时安全。
	2. 在url中一般要体现api标识，这样看到url就知道他是一个api。
		http://www.luffycity.com/api/....（建议，因为他不会存在跨域的问题）
		http://api.luffycity.com/....
		假设：
			前段：https://www.luffycity.com/home
			后端：https://www.luffycity.com/api/
	3. 在接口中要体现版本
		http://www.luffycity.com/api/v1....（建议，因为他不会存在跨域的问题）
		注意：版本还可以放在请求头中
			http://www.luffycity.com/api/
			accept: ...
			
	4. restful也称为面向资源编程，视网络上的一切都是资源，对资源可以进行操作，所以一般资源都用名词。
		http://www.luffycity.com/api/user/
		
	5. 如果要加入一些筛选条件，可以添加在url中	
		http://www.luffycity.com/api/user/?page=1&type=9

	6. 根据method不同做不同操作。

下面的可以不说但是得记住：

	7. 返回给用户状态码
		- 200，成功
		- 300，301永久 /302临时
		- 400，403拒绝 /404找不到
		- 500，服务端代码错误
		
		很多公司：
                def get(self,request,*args,**kwargs):
                    result = {'code':1000,'data':None,'error':None}
                    try:
                        val = int('你好')
                    except Exception as e:
                        result['code'] = 10001
                        result['error'] = '数据转换错误'

                    return Response(result)
        
	8. 返回值
		GET http://www.luffycity.com/api/user/
			[
				{'id':1,'name':'alex','age':19},
				{'id':1,'name':'alex','age':19},
			]
		POST http://www.luffycity.com/api/user/
			{'id':1,'name':'alex','age':19}
			
		GET http://www.luffycity.com/api/user/2/
			{'id':2,'name':'alex','age':19}
			
		PUT http://www.luffycity.com/api/user/2/
			{'id':2,'name':'alex','age':19}
		
		PATCH https//www.luffycity.com/api/user/2/
			{'id':2,'name':'alex','age':19}
			
		DELETE https//www.luffycity.com/api/user/2/
			空
	9. 操作异常时，要返回错误信息
	
		{
            error: "Invalid API key"
        }
	10. 对于下一个请求要返回一些接口：Hypermedia AP
		{
			'id':2,
			'name':'alex',
			'age':19,
			'depart': "http://www.luffycity.com/api/user/30/"
		}
```

2.drf框架

```
记忆：请求到来之后，先执行视图的dispatch方法。

1. 视图
2. 版本处理
3. 认证
4. 权限
5. 节流（频率限制）
6. 解析器
7. 筛选器
8. 分页
9. 序列化
10. 渲染
```

### 1.跨域

链接：https://chpl.top/2019/10/05/%E4%B9%A6/Django/%E7%AC%AC%E4%BA%8C%E5%8D%81%E8%AE%B2%E2%80%94%E2%80%94Django%E7%BB%BC%E5%90%88%E6%89%A9%E5%B1%95%E5%8D%81%E4%B9%8B%E8%B7%A8%E5%9F%9F%E5%92%8CCORS/

```
由于浏览器具有“同源策略”的限制。
如果在同一个域下发送ajax请求，浏览器的同源策略不会阻止。
如果在不同域下发送ajax，浏览器的同源策略会阻止。
```

#### 总结

- 域相同，永远不会存在跨域。
  - crm，非前后端分离，没有跨域。
  - 路飞学城，前后端分离，没有跨域（之前有，现在没有）。
- 域不同时，才会存在跨域。
  - l拉勾网，前后端分离，存在跨域（设置响应头解决跨域)

#### 解决跨域：CORS

```python
本质在数据返回值设置响应头

from django.shortcuts import render,HttpResponse

def json(request):
    response = HttpResponse("JSONasdfasdf")
    response['Access-Control-Allow-Origin'] = "*"
    return response
    
```

#### 跨域时，发送了2次请求？

在跨域时，发送的请求会分为两种：

- 简单请求，发一次请求。

  ```python
  设置响应头就可以解决
  from django.shortcuts import render,HttpResponse
  
  def json(request):
      response = HttpResponse("JSONasdfasdf")
      response['Access-Control-Allow-Origin'] = "*"
      return response
  ```
  
- 复杂请求，发两次请求。

  - 预检
  - 请求

```python

@csrf_exempt
def put_json(request):
    response = HttpResponse("JSON复杂请求")
    if request.method == 'OPTIONS':
        # 处理预检
        response['Access-Control-Allow-Origin'] = "*"
        response['Access-Control-Allow-Methods'] = "PUT"
        return response
    elif request.method == "PUT":
        return response
```

```python
条件：
    1、请求方式：HEAD、GET、POST
    2、请求头信息：
        Accept
        Accept-Language
        Content-Language
        Last-Event-ID
        Content-Type 对应的值是以下三个中的任意一个
                                application/x-www-form-urlencoded
                                multipart/form-data
                                text/plain
 
注意：同时满足以上两个条件时，则是简单请求，否则为复杂请求
```

#### 总结

1.由于浏览器具有“同源策略”的限制，所以在浏览器上跨域发送	    	Ajax请求时，会被浏览器阻止。

2.解决跨域

- CORS（跨站资源共享，本质是设置响应头来解决）。
  - 简单请求：发送一次请求
  - 复杂请求：发送两次请求

### 2.drf的访问频率限制

- 频率限制在版本、认证、权限之后

- 知识点

  ```python
  {
  	throttle_anon_1.1.1.1:[100121340,],
  	1.1.1.2:[100121251,100120450,]
  }
  
  限制：60s能访问3次
  来访问时：
  	1.获取当前时间 100121280
  	2.100121280-60 = 100121220，小于100121220所有记录删除
  	3.判断1分钟以内已经访问多少次了？ 4 
  	4.无法访问
  停一会
  来访问时：
  	1.获取当前时间 100121340
  	2.100121340-60 = 100121280,小于100121280所有记录删除
  	3.判断1分钟以内已经访问多少次了？ 0
  	4.可以访问
  ```

#### 源码

重写方法

```python
from rest_framework.throttling import SimpleRateThrottle


class AnonThrottle(SimpleRateThrottle):
	scope = 'anon'  # 相当于设置了最大的访问次数和时间

	def get_cache_key(self, request, view):
		if request.user:
			return None  # 返回None表示我不限制，登录用户我不管
			# 匿名用户
		return self.get_ident(request)  # 返回一个唯一标识IP


class UserThrottle(SimpleRateThrottle):
	scope = 'user'

	def get_cache_key(self, request, view):
		# 登录用户
		if request.user:
			return request.user
		return None  # 返回NOne表示匿名用户我不管
```

在视图中使用

```python
from rest_framework.views import APIView
from rest_framework.response import Response

from rest_framework.throttling import AnonRateThrottle,BaseThrottle

class ArticleView(APIView):
    throttle_classes = [AnonRateThrottle,]
    def get(self,request,*args,**kwargs):
        return Response('文章列表')
    
    	def throttled(self, request, wait):
		'''可定制方法设置中文错误'''

		# raise exceptions.Throttled(wait)
		class MyThrottle(exceptions.Throttled):
			default_detail = '请求被限制'
			extra_detail_singular = 'Expected available in {wait} second.'
			extra_detail_plural = 'Expected available in {wait} seconds.'
			default_code = '还需要再等{wait}秒'

		raise MyThrottle(wait)

class ArticleDetailView(APIView):
    def get(self,request,*args,**kwargs):
        return Response('文章列表')
```

##### dispacth

```python
def dispatch(self, request, *args, **kwargs):
        """
        `.dispatch()` is pretty much the same as Django's regular dispatch,
        but with extra hooks for startup, finalize, and exception handling.
        """
        self.args = args
        self.kwargs = kwargs
        # 1. 对request进行加工
        # request封装了
        """
        request,
        parsers=self.get_parsers(),
        authenticators=self.get_authenticators(),
        negotiator=self.get_content_negotiator(),
        parser_context=parser_context
        """
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        self.headers = self.default_response_headers  # deprecate?

        try:
            # 初始化request
            # 确定request版本,用户认证，权限控制，用户访问频率限制
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
        # 6. 二次加工request
        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response
```

##### initial

```python
def initial(self, request, *args, **kwargs):
        """
        Runs anything that needs to occur prior to calling the method handler.
        """
        self.format_kwarg = self.get_format_suffix(**kwargs)

        # Perform content negotiation and store the accepted info on the request
        neg = self.perform_content_negotiation(request)
        request.accepted_renderer, request.accepted_media_type = neg

        # Determine the API version, if versioning is in use.
        # 2. 确定request版本信息
        version, scheme = self.determine_version(request, *args, **kwargs)
        request.version, request.versioning_scheme = version, scheme

        # Ensure that the incoming request is permitted
        # 3. 用户认证
        self.perform_authentication(request)
        # 4. 权限控制
        self.check_permissions(request)
        # 5. 用户访问频率限制
        self.check_throttles(request)
```

##### check_throttles

```python
def check_throttles(self, request):
        """
        Check if request should be throttled.
        Raises an appropriate exception if the request is throttled.
        """
        for throttle in self.get_throttles():
            #循环每一个throttle对象，执行allow_request方法
            # allow_request:
                #返回False，说明限制访问频率
                #返回True，说明不限制，通行
            if not throttle.allow_request(request, self):
                self.throttled(request, throttle.wait())
                #throttle.wait()表示还要等多少秒就能访问了

check_throttles
```

##### get_throttles

```python
def get_throttles(self):
    """
    Instantiates and returns the list of throttles that this view uses.
    """
    return [throttle() for throttle in self.throttle_classes]
```

##### throttle_classes（可自定制）

```python
class APIView(View):

    # 以下策略可以设置为全局策略或按视图策略
    renderer_classes = api_settings.DEFAULT_RENDERER_CLASSES
    parser_classes = api_settings.DEFAULT_PARSER_CLASSES
    authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES
    #这个就是我们可以设置的频率限制
    throttle_classes = api_settings.DEFAULT_THROTTLE_CLASSES
    permission_classes = api_settings.DEFAULT_PERMISSION_CLASSES
    content_negotiation_class = api_settings.DEFAULT_CONTENT_NEGOTIATION_CLASS
    metadata_class = api_settings.DEFAULT_METADATA_CLASS
    versioning_class = api_settings.DEFAULT_VERSIONING_CLASS

    # 允许其他设置的依赖注入，以使测试更容易
    settings = api_settings

    schema = DefaultSchema()
```

#### 总结

如何实现的评率限制

```python
- 匿名用户，用IP作为用户唯一标记，但如果用户换代理IP，无法做到真正的限制（因为可以使用代理）。
- 登录用户，用用户名或用户ID做标识。
具体实现：
	在django的缓存中 = {
        throttle_anon_1.1.1.1:[100121340,],
        1.1.1.2:[100121251,100120450,]
    }


    限制：60s能访问3次
    来访问时：
        1.获取当前时间 100121280
        2.100121280-60 = 100121220，小于100121220所有记录删除
        3.判断1分钟以内已经访问多少次了？ 4 
        4.无法访问
    停一会
    来访问时：
        1.获取当前时间 100121340
        2.100121340-60 = 100121280,小于100121280所有记录删除
        3.判断1分钟以内已经访问多少次了？ 0
        4.可以访问
```

### 3.jwt

解释：JSON Web Token，它是一种认证技术。用在前后端分离项目中，实现和用户登录相关的操作。

在我们dajngo的时候使用的方式是session，在我们上一节的内容中也使用了认证，那种是使用uuid随机生成一个随机码给前端，后端储存一份，用户再来请求的时候携带该token和后端的token对比验证。

一般用户认证有2中方式：

- token

  ```
  用户登录成功之后，生成一个随机字符串，自己保留一分+给前端返回一份。
  
  以后前端再来发请求时，需要携带字符串。
  
  后端对字符串进行校验（是否超时，是否合法）。
  ```

- jwt

  ```python
  用户登录成功之后，生成一个随机字符串，给前端。
  	-格式：xx.xx.xx
  	- 生成随机字符串
  		{typ:"jwt","alg":'HS256'}     {id:1,username:'alx','exp':10}
  		98qow39df0lj980945lkdjflo.saueoja8979284sdfsdf.asiuokjd978928374
  		- token类型和短发通过base64url加密
  		- 用户传输的一些数据通过base64加密
  		- 前两个密文拼接后使用h256加密+加盐（该盐是一串字符串，jwt里面自动处理）再使用base64url进行加密
  		- 给前端返回
  		98qow39df0lj980945lkdjflo.saueoja8979284sdfsdf.asiuokjd978928375
  
  前端获取随机字符串之后，保留起来。
  以后再来发送请求时，携带98qow39df0lj980945lkdjflo.saueoja8979284sdfsdf.asiuokjd978928375。
  
  
  后端接受到之后，
  	1.先做时间判断
      2.字符串合法性校验。
      
  ```

#### 安装

```python
pip3 install djangorestframework-jwt
```

#### 案例

- app中注册

  ```python
  INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      'api.apps.ApiConfig',
      'rest_framework',
      #注册
      'rest_framework_jwt'
  ]
  ```

- 用户登录

  ```python
  import uuid
  from rest_framework.views import APIView
  from rest_framework.response import Response
  from rest_framework.versioning import URLPathVersioning
  from rest_framework import status
  
  from api import models
  
  class LoginView(APIView):
      """
      登录接口
      """
      def post(self,request,*args,**kwargs):
  
          # 基于jwt的认证
          # 1.去数据库获取用户信息
          from rest_framework_jwt.settings import api_settings
          jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
          jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
  
          user = models.UserInfo.objects.filter(**request.data).first()
          if not user:
              return Response({'code':1000,'error':'用户名或密码错误'})
  
          payload = jwt_payload_handler(user)
          token = jwt_encode_handler(payload)
          return Response({'code':1001,'data':token})
  
  ```

- 用户认证

  ```python
  from rest_framework.views import APIView
  from rest_framework.response import Response
  
  # from rest_framework.throttling import AnonRateThrottle,BaseThrottle
  
  
  class ArticleView(APIView):
      # throttle_classes = [AnonRateThrottle,]
  
      def get(self,request,*args,**kwargs):
          # 获取用户提交的token，进行一步一步校验
          import jwt
          from rest_framework import exceptions
          from rest_framework_jwt.settings import api_settings
          jwt_decode_handler = api_settings.JWT_DECODE_HANDLER
  
          jwt_value = request.query_params.get('token')
          try:
              payload = jwt_decode_handler(jwt_value)
          except jwt.ExpiredSignature:
              msg = '签名已过期'
              raise exceptions.AuthenticationFailed(msg)
          except jwt.DecodeError:
              msg = '认证失败'
              raise exceptions.AuthenticationFailed(msg)
          except jwt.InvalidTokenError:
              raise exceptions.AuthenticationFailed()
  
          return Response('文章列表')
  ```

#### 详细使用实例

##### falsk中使用

jwt_auth.py

```python
import jwt
import datetime
from jwt import exceptions

JWT_SALT = 'iv%x6xo7l7_u9bf_u!9#g#m*)*=ej@bek5)(@u3kh*72+unjv='


def create_token(payload, timeout=20):
    """
    :param payload:  例如：{'user_id':1,'username':'wupeiqi'}用户信息
    :param timeout: token的过期时间，默认20分钟
    :return:
    """
    headers = {
        'typ': 'jwt',
        'alg': 'HS256'
    }
    payload['exp'] = datetime.datetime.utcnow() + datetime.timedelta(minutes=timeout)
    result = jwt.encode(payload=payload, key=JWT_SALT, algorithm="HS256", headers=headers).decode('utf-8')
    return result


def parse_payload(token):
    """
    对token进行和发行校验并获取payload
    :param token:
    :return:
    """
    result = {'status': False, 'data': None, 'error': None}
    try:
        verified_payload = jwt.decode(token, JWT_SALT, True)
        result['status'] = True
        result['data'] = verified_payload
    except exceptions.ExpiredSignatureError:
        result['error'] = 'token已失效'
    except jwt.DecodeError:
        result['error'] = 'token认证失败'
    except jwt.InvalidTokenError:
        result['error'] = '非法的token'
    return result
```

manage.py

```python
from flask import Flask, request, jsonify, views, g
from utils.jwt_auth import create_token, parse_payload

app = Flask(__name__)


# 通过url传递token

@app.before_request
def jwt_query_params_auth():
    if request.path == '/login/':
        return

    token = request.args.get('token')
    result = parse_payload(token)
    if not result['status']:
        return jsonify(result)
    g.user_info = result['data']

@app.route('/login/', methods=['POST'])
def login():
    user = request.form.get('username')
    pwd = request.form.get('password')

    # 检测用户和密码是否正确，此处可以在数据进行校验。
    if user == 'wupeiqi' and pwd == '123':
        # 用户名和密码正确，给用户生成token并返回
        token = create_token({'username': 'wupeiqi'})
        return jsonify({'status': True, 'token': token})
    return jsonify({'status': False, 'error': '用户名或密码错误'})

@app.route('/order/', methods=['GET', "POST", "PUT", "DELETE"])
def order():
    print(g.user_info)
    if request.method == 'GET':
        return "订单列表"
    return "订单信息"


if __name__ == '__main__':
    app.run()
```

##### drf中使用

路由

```python
from django.conf.urls import url, include

urlpatterns = [
    url(r'^api/(?P<version>\w+)/', include('api.urls')),
]
```

```python
from django.conf.urls import url, include
from api import views

urlpatterns = [
    #登陆
    url('^login/$', views.LoginView.as_view()),
    #订单
    url('^order/$', views.OrderView.as_view()),
]

```

视图

```python
from rest_framework.views import APIView
from rest_framework.response import Response

from utils.jwt_auth import create_token
from extensions.auth import JwtQueryParamAuthentication, JwtAuthorizationAuthentication


class LoginView(APIView):
    def post(self, request, *args, **kwargs):
        """ 用户登录 """
        user = request.POST.get('username')
        pwd = request.POST.get('password')

        # 检测用户和密码是否正确，此处可以在数据进行校验。
        if user == 'wupeiqi' and pwd == '123':
            # 用户名和密码正确，给用户生成token并返回
            token = create_token({'username': 'wupeiqi'})
            return Response({'status': True, 'token': token})
        return Response({'status': False, 'error': '用户名或密码错误'})


class OrderView(APIView):
    # 通过url传递token
    authentication_classes = [JwtQueryParamAuthentication, ]

    # 通过Authorization请求头传递token
    # authentication_classes = [JwtAuthorizationAuthentication, ]

    def get(self, request, *args, **kwargs):
        print(request.user, request.auth)
        return Response({'data': '订单列表'})

    def post(self, request, *args, **kwargs):
        print(request.user, request.auth)
        return Response({'data': '添加订单'})

    def put(self, request, *args, **kwargs):
        print(request.user, request.auth)
        return Response({'data': '修改订单'})

    def delete(self, request, *args, **kwargs):
        print(request.user, request.auth)
        return Response({'data': '删除订单'})

```

from utils.jwt_auth import create_token

```python
import jwt
import datetime
from jwt import exceptions

JWT_SALT = 'iv%x6xo7l7_u9bf_u!9#g#m*)*=ej@bek5)(@u3kh*72+unjv='


def create_token(payload, timeout=20):
    """
    :param payload:  例如：{'user_id':1,'username':'wupeiqi'}用户信息
    :param timeout: token的过期时间，默认20分钟
    :return:
    """
    headers = {
        'typ': 'jwt',
        'alg': 'HS256'
    }
    payload['exp'] = datetime.datetime.utcnow() + datetime.timedelta(minutes=timeout)
    result = jwt.encode(payload=payload, key=JWT_SALT, algorithm="HS256", headers=headers).decode('utf-8')
    return result


def parse_payload(token):
    """
    对token进行和发行校验并获取payload
    :param token:
    :return:
    """
    result = {'status': False, 'data': None, 'error': None}
    try:
        verified_payload = jwt.decode(token, JWT_SALT, True)
        result['status'] = True
        result['data'] = verified_payload
    except exceptions.ExpiredSignatureError:
        result['error'] = 'token已失效'
    except jwt.DecodeError:
        result['error'] = 'token认证失败'
    except jwt.InvalidTokenError:
        result['error'] = '非法的token'
    return result

```

from extensions.auth import JwtQueryParamAuthentication, JwtAuthorizationAuthentication

```python
from rest_framework.authentication import BaseAuthentication
from rest_framework import exceptions
from utils.jwt_auth import parse_payload


class JwtQueryParamAuthentication(BaseAuthentication):
    """
    用户需要在url中通过参数进行传输token，例如：
    http://www.pythonav.com?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NzM1NTU1NzksInVzZXJuYW1lIjoid3VwZWlxaSIsInVzZXJfaWQiOjF9.xj-7qSts6Yg5Ui55-aUOHJS4KSaeLq5weXMui2IIEJU
    """

    def authenticate(self, request):
        token = request.query_params.get('token')
        payload = parse_payload(token)
        if not payload['status']:
            raise exceptions.AuthenticationFailed(payload)

        # 如果想要request.user等于用户对象，此处可以根据payload去数据库中获取用户对象。
        return (payload, token)


class JwtAuthorizationAuthentication(BaseAuthentication):
    """
    用户需要通过请求头的方式来进行传输token，例如：
    Authorization:jwt eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NzM1NTU1NzksInVzZXJuYW1lIjoid3VwZWlxaSIsInVzZXJfaWQiOjF9.xj-7qSts6Yg5Ui55-aUOHJS4KSaeLq5weXMui2IIEJU
    """

    def authenticate(self, request):

        # 非登录页面需要校验token
        authorization = request.META.get('HTTP_AUTHORIZATION', '')
        auth = authorization.split()
        if not auth:
            raise exceptions.AuthenticationFailed({'error': '未获取到Authorization请求头', 'status': False})
        if auth[0].lower() != 'jwt':
            raise exceptions.AuthenticationFailed({'error': 'Authorization请求头中认证方式错误', 'status': False})

        if len(auth) == 1:
            raise exceptions.AuthenticationFailed({'error': "非法Authorization请求头", 'status': False})
        elif len(auth) > 2:
            raise exceptions.AuthenticationFailed({'error': "非法Authorization请求头", 'status': False})

        token = auth[1]
        result = parse_payload(token)
        if not result['status']:
            raise exceptions.AuthenticationFailed(result)

        # 如果想要request.user等于用户对象，此处可以根据payload去数据库中获取用户对象。
        return (result, token)

```

##### django中间件中使用

配置文件

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # 通过url传递token
    'middlewares.jwt.JwtQueryParamMiddleware' 
    # 通过Authorization请求头传递token
    # 'middlewares.jwt.JwtAuthorizationMiddleware'  
]
```

路由

```python
from django.conf.urls import url
from api import views

urlpatterns = [
    url('login/', views.LoginView.as_view()),
    url('order/', views.OrderView.as_view()),
]
```

视图

```python
from django.http import JsonResponse
from django.views import View
from django.views.decorators.csrf import csrf_exempt
from django.utils.decorators import method_decorator

from utils.jwt_auth import create_token

#免除csrf认证
@method_decorator(csrf_exempt, name='dispatch')
class LoginView(View):
    def post(self, request, *args, **kwargs):
        """ 用户登录 """
        user = request.POST.get('username')
        pwd = request.POST.get('password')

        # 检测用户和密码是否正确，此处可以在数据进行校验。
        if user == 'wupeiqi' and pwd == '123':
            # 用户名和密码正确，给用户生成token并返回
            token = create_token({'username': 'wupeiqi'})
            return JsonResponse({'status': True, 'token': token})
        return JsonResponse({'status': False, 'error': '用户名或密码错误'})

#免除csrf认证
@method_decorator(csrf_exempt, name='dispatch')
class OrderView(View):

    def get(self, request, *args, **kwargs):
        print(request.user_info)
        return JsonResponse({'data': '订单列表'})

    def post(self, request, *args, **kwargs):
        print(request.user_info)
        return JsonResponse({'data': '添加订单'})

    def put(self, request, *args, **kwargs):
        print(request.user_info)
        return JsonResponse({'data': '修改订单'})

    def delete(self, request, *args, **kwargs):
        print(request.user_info)
        return JsonResponse({'data': '删除订单'})

```

from utils.jwt_auth import create_token

```python
import jwt
import datetime
from jwt import exceptions

JWT_SALT = 'iv%x6xo7l7_u9bf_u!9#g#m*)*=ej@bek5)(@u3kh*72+unjv='


def create_token(payload, timeout=20):
    """
    :param payload:  例如：{'user_id':1,'username':'wupeiqi'}用户信息
    :param timeout: token的过期时间，默认20分钟
    :return:
    """
    headers = {
        'typ': 'jwt',
        'alg': 'HS256'
    }
    payload['exp'] = datetime.datetime.utcnow() + datetime.timedelta(minutes=timeout)
    result = jwt.encode(payload=payload, key=JWT_SALT, algorithm="HS256", headers=headers).decode('utf-8')
    return result


def parse_payload(token):
    """
    对token进行和发行校验并获取payload
    :param token:
    :return:
    """
    result = {'status': False, 'data': None, 'error': None}
    try:
        verified_payload = jwt.decode(token, JWT_SALT, True)
        result['status'] = True
        result['data'] = verified_payload
    except exceptions.ExpiredSignatureError:
        result['error'] = 'token已失效'
    except jwt.DecodeError:
        result['error'] = 'token认证失败'
    except jwt.InvalidTokenError:
        result['error'] = '非法的token'
    return result

```

中间件

```python
from django.utils.deprecation import MiddlewareMixin
from utils.jwt_auth import parse_payload
from django.http import JsonResponse


class JwtQueryParamMiddleware(MiddlewareMixin):
    """
    用户需要在url中通过参数进行传输token，例如：
    http://www.pythonav.com?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NzM1NTU1NzksInVzZXJuYW1lIjoid3VwZWlxaSIsInVzZXJfaWQiOjF9.xj-7qSts6Yg5Ui55-aUOHJS4KSaeLq5weXMui2IIEJU
    """

    def process_request(self, request):
        if request.path_info == '/login/':
            return

        token = request.GET.get('token')
        result = parse_payload(token)
        if not result['status']:
            return JsonResponse(result)
        request.user_info = result['data']


class JwtAuthorizationMiddleware(MiddlewareMixin):
    """
    用户需要通过请求头的方式来进行传输token，例如：
    Authorization:jwt eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NzM1NTU1NzksInVzZXJuYW1lIjoid3VwZWlxaSIsInVzZXJfaWQiOjF9.xj-7qSts6Yg5Ui55-aUOHJS4KSaeLq5weXMui2IIEJU
    """

    def process_request(self, request):

        # 如果是登录页面，则通过
        if request.path_info == '/login/':
            return

        # 非登录页面需要校验token
        authorization = request.META.get('HTTP_AUTHORIZATION', '')
        auth = authorization.split()
        if not auth:
            return JsonResponse({'error': '未获取到Authorization请求头', 'status': False})
        if auth[0].lower() != 'jwt':
            return JsonResponse({'error': 'Authorization请求头中认证方式错误', 'status': False})
        if len(auth) == 1:
            return JsonResponse({'error': "非法Authorization请求头", 'status': False})
        elif len(auth) > 2:
            return JsonResponse({'error': "非法Authorization请求头", 'status': False})

        token = auth[1]
        result = parse_payload(token)
        if not result['status']:
            return JsonResponse(result)
        request.user_info = result['data']
```





























































































































