---
title: requests和Django中的request对象
id: 8
date: 2019-8-22 20:30:00
tags: 第三方模块
comment: true
---

### Django request对象

介绍：服务器接收到http协议的请求之后，会根据包文创建HttpRequest对象，这个对象不需要我们创建，直接使用服务器构造好的对象就可以。视图的第一个参数必须是HttpRequest对象，再djando.http模块中定义了HttpRequest对象的API。

### request的对象

这个对象也就是视图的第一个参数def  func(request):

#### **属性**

- path：请求页面的相对路径，但是不包括域名：

  ```python
  def func(request):
  	print (retuest.path)
      print (request.get_ful_path())
  结果：
  "/music/renran/"（这个路径就是文件再服务器上的地址）
  "music/renren?a=1&b=2"
  
  扩展：
  获取相对路径:request.get_info
  获取全路径:request.get_ful_path()
  获取绝对url:request.build_absolute_uri()
  ```

- method:提交请求使用的HTTP方法。总是大写：

  ```python
  if request.method ==‘GET/POST’:
  	do_something
  ```

- GET:一个类字典对象，包含所有的HTTP的get参数信息。

  ```python
  获取get方式表单中或url提交的数据
  request.POST["username"]
  request.POST.get("username")
  
  #注意：虽然上面的两种方式都可以取得username的值，但是如果POST对象中没有username，那么使用request.POST["username"]就会报错，而request.POST.get("username")不会报错，而是返回一个None。
  其实上面的这个问题也很好解决：
  request.POST["username",None]，加个None就很好的解决了
  ```

- POST：一个类字典对象，包涵所有的HTTP的POST参数的信息（提交表单，但是可以是空，不包含文件的上传）。

  ```python
  #获取post中表单键值数据
  request.POST  #<QueryDict: {"username":"pl"}>  
  request.POST.dict()   #{"username":"pl"}
  request.POST.get("username")   #"pl"
  
  #总结：
  request.POST适合接收表单数据，表单数据一般比较简单，面对复杂的数据时，即使是一个简单的list接受后都会变形(元素变成字符串),处理起来不方便。
  
  request.body可以方便的提交复杂的结构化数据，特别适合RESTful的接口
  
  #实例对比POST和body的数据
  print ("body:",request.body)
  print(request.body.decode('utf-8'))
  print ("--------------")
  print ("POST:",request.POST)
  print(request.POST.dict())
  #结果（input的name=pl,value=pl）
  body: b'pl=pl'	#body的原始数据是bytes类型，所以进行json转换的时候非常的容易转换和处理
  pl=pl    #解码之后直接就拿到了字符串数据
  --------------
  POST: <QueryDict: {'pl': ['pl']}>   #POST拿到请求字典形式的数据，它和字典类似又不一样，获取它里面的数据的详细信息在下面的知识扩展里面
  {'pl': 'pl'}	#这个方法就是获取它的字典属性
  #注意：提交数据的时候，input中的name千万别忘记写了，不然是收不到数据的
  这里有一个建议，当你成功处理POST数据后，应当保持一个良好的习惯，始终返回一个HttpResponseRedirect。这不仅仅是对Django而言，它是一个良好的WEB开发习惯。
  ```

- COOKIES：一个标准的python字典，包涵所有的cookie(键值都是字符串)。

<!-----more----->

- META:一个标准的python字典，包含所有的有效的HTTP头信息，有效的信息设计关于server和client之间的，里面有 很多的参数：

  ```
  CONTENT_LENGTH —— 请求的正文的长度（是一个字符串）。
  
  CONTENT_TYPE —— 请求的正文的MIME 类型。
  
  HTTP_ACCEPT —— 响应可接收的Content-Type。
  
  HTTP_ACCEPT_ENCODING —— 响应可接收的编码。
  
  HTTP_ACCEPT_LANGUAGE —— 响应可接收的语言。
  
  HTTP_HOST —— 客服端发送的HTTP Host 头部。
  
  HTTP_REFERER —— Referring 页面。
  
  HTTP_USER_AGENT —— 客户端的user-agent 字符串。
  
  QUERY_STRING —— 单个字符串形式的查询字符串（未解析过的形式）。
  
  REMOTE_ADDR —— 客户端的IP 地址。
  
  REMOTE_HOST —— 客户端的主机名。
  
  REMOTE_USER —— 服务器认证后的用户。
  
  REQUEST_METHOD —— 一个字符串，例如"GET" 或"POST"。
  
  SERVER_NAME —— 服务器的主机名。
  
  SERVER_PORT —— 服务器的端口（是一个字符串）
  ```

- files：一个类字典对象，包涵所有的上传文件。

  ```python
  自身包涵3个key：
      filename:上传的文件名。
      Content-type:上传文件的内容类型。
      Content:上传文件的原始类容。
  
  注意：FILES 只在请求的方法是 POST，并且提交的 <form> 包含enctype="multipart/form-data"时才包含数据。否则， FILES 只是一个空的类字典对象。
  ```

- body：请求的主体，返回的是一个字符串,get请求body里面没有数据，它拿到的数据就是post请求的原始数据部分

- data：请求的数据部分，返回的是一个字典对象（除此之外，与request.body是很类似的）

- scheme:请求的方式，要么是http或者https

- encoding：请求提交的数据的编码方式

- user(中间件技术)：一个AUUTH_USER_MODEL类型的对象表示当前登陆的用户，user 仅当Django激活 AuthenticationMiddleware时有效。就是关于认证和用户的完整细节。

- session(中间件技术)：一个类字典的对象，表示当前的会话，常用来保**存一些数据来实现会话跟踪技术。既可以读又可写的类似于字典的对象。仅当Django激活session支持有效。**

#### 方法

- __getitem__(key):请求所给键的GET/POST值，先查找POST，然后查找GET。

- has_key():返回True/False，表示request.GET   or request.POST是否包含所给的键。

- is_secure():如果请求时安全的，就返回True,也就是说请求时按HTTPS的形式提交的。

- is_ajax():如果请求是通过XMLHttpRequest 发起的，则返回True。

- get_host():返回请求的源主机(IP+PORT)

- get_port():返回请求主机的源端口号

- get_full_path():返回相全路径，并包含附加的查询信息

- build_absolute_uri(location):返回location的绝对uri，location默认为request.get_full_path()。

- get_signed_cookie(key, default=RAISE_ERROR, salt=’’, max_age=None):返回签名过的Cookie 对应的值，如果签名不再合法则返回django.core.signing.BadSignature。如果提供 default 参数，将不会引发异常并返回 default 的值。

  可选参数salt 可以用来对安全密钥强力攻击提供额外的保护。max_age 参数用于检查Cookie 对应的时间戳以确保Cookie 的时间不会超过max_age 秒。

#### 知识扩展（QueryDicts）

介绍：queryDict是一个类似字典的一种对象，它是python中字典的子类。专门用来处理用一个键的多值。当处理一些HTML表单中的元素，特别是

```html
<select multiple='multiple'>
```

之类传递同一key的多值的元素时，就需要这个类。

QueryDict实现了所有标准字典的方法，因为它就是字典的一个子类，但是**其与标准字典之间的区别是：**

__getitem__:与一个字典一样，当一个键多值时，返回最后一个值

__setitem__：将所给键的值设为一个只有一个值的列表

get():如果一个键多个值，返回最后一个值

update():这个区别是在这里是增加值而不是替换

items():和标准字典一样，不同的是它返回最后一个值

values():都是返回最后一个值

pop():删除某个键值对

dict():返回QueryDict的字典的表现形式

**QueryDict非字典的方法：**

copy():返回一个副本，使用的是copy.deepcopy()。值可变，就是软拷贝

getlist(key):以python列表的形式返回所请求的数据，若键不存在则返回空列表

setlist(key,list_):将所给的键的键值设为list_

appendlist(key,item):在key相关的list上增加item

setlistfault(key,l):和setduault一样，但是第二个元素时一个列表

lists():和items一样，但是它以一个列表的形式返回字典每一个成员的所有值

urlencode():返回一个请求字符串格式的数据字符串

### 爬虫的requests模块

```python
response = requests.get("www.baidu.com")
```

response = request.get()取得返回的数据

response.text  获取整个网页的源数据

response.history  拿出正常返回的数据   # 和text相同

response.content  #二进制内容（rar文件、图片、视频等）

response.iter_content #如果是一个视频文件数量就非常的大，一次取值的话就会把内存撑爆,这个就相当于for循环接收保存

response.status_code #用来获取状态码 200/404/500等

response.headers #获取响应头（以字典的形式返回）

response.cookies #返回cookies

response.cookies.get_dict()  #返回cookies，以字典的形式返回

response.cookies.items()   和字典的items同理

response.url #拿出要重定向的地址

response.encoding #返回数据的编码格式

response.apparent_encoding  #返回当前页码的编码格式

```python
#HTTP请求类型
#get类型
r = requests.get('https://github.com/timeline.json')
#post类型
r = requests.post("http://m.ctrip.com/post")
#put类型
r = requests.put("http://m.ctrip.com/put")
#delete类型
r = requests.delete("http://m.ctrip.com/delete")
#head类型
r = requests.head("http://m.ctrip.com/head")
#options类型
r = requests.options("http://m.ctrip.com/get")

#获取响应内容
print r.content #以字节的方式去显示，中文显示为字符
print r.text #以文本的方式去显示

#URL传递参数
payload = {'keyword': '日本', 'salecityid': '2'}
r = requests.get("http://m.ctrip.com/webapp/tourvisa/visa_list", params=payload) 
print r.url #示例为http://m.ctrip.com/webapp/tourvisa/visa_list?salecityid=2&keyword=日本

#获取/修改网页编码
r = requests.get('https://github.com/timeline.json')
print r.encoding
r.encoding = 'utf-8'

#json处理
r = requests.get('https://github.com/timeline.json')
print r.json() #需要先import json    

#定制请求头
url = 'http://m.ctrip.com'
headers = {'User-Agent' : 'Mozilla/5.0 (Linux; Android 4.2.1; en-us; Nexus 4 Build/JOP40D) AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.166 Mobile Safari/535.19'}
r = requests.post(url, headers=headers)
print r.headers

#复杂post请求
url = 'http://m.ctrip.com'
payload = {'some': 'data'}
r = requests.post(url, data=json.dumps(payload)) #如果传递的payload是string而不是dict，需要先调用dumps方法格式化一下

#post多部分编码文件
url = 'http://m.ctrip.com'
files = {'file': open('report.xls', 'rb')}
r = requests.post(url, files=files)

#响应状态码
r = requests.get('http://m.ctrip.com')
print r.status_code
    
#响应头
r = requests.get('http://m.ctrip.com')
print r.headers
print r.headers['Content-Type']
print r.headers.get('content-type') #访问响应头部分内容的两种方式
    
#Cookies
url = 'http://example.com/some/cookie/setting/url'
r = requests.get(url)
r.cookies['example_cookie_name']    #读取cookies
    
url = 'http://m.ctrip.com/cookies'
cookies = dict(cookies_are='working')
r = requests.get(url, cookies=cookies) #发送cookies

#设置超时时间
r = requests.get('http://m.ctrip.com', timeout=0.001)

#设置访问代理
proxies = {
           "http": "http://10.10.10.10:8888",
           "https": "http://10.10.10.100:4444",
          }
r = requests.get('http://m.ctrip.com', proxies=proxies)
```

