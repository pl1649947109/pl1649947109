---
title: 第二十讲——Django综合扩展十之跨域和CORS
id: 20
date: 2019-10-22 20:00:00
tags: Django
comment: true
---

### 跨域

同源策略（Same origin policy）是一种约定，**它是浏览器最核心也最基本的安全功能**，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。　　　

同源策略，它是由Netscape提出的一个著名的安全策略。现在所有支持JavaScript 的浏览器都会使用这个策略。**所谓同源是指，域名，协议，端口相同。**当一个浏览器的两个tab页中分别打开来 百度和谷歌的页面当浏览器的百度tab页执行一个脚本的时候会检查这个脚本是属于哪个页面的，即检查是否同源，只有和百度同源的脚本才会被执行。如果非同源，那么在请求数据时，浏览器会在控制台中报一个异常，提示拒绝访问。

**使用自己的或表述就是：我现在有两个项目，其中一个项目的地址是127.0.0.1:8000,另一个项目的地址是127.0.0.1:8001。很明显，这两个项目的port是不一样的，所以他们就不是同源的。现在我们8000下面有一个函数想去取8001下面的数据，这就产生了跨域，这样留啦你就会把消息拦截下来，并返回不同源错误信息。**

现在我们有这样一个需求，a和b是不同源的两个项目，但是他们是有相互关系的项目，我们就是想跨域交互数据，怎么办？？？下面的CORS就给出了解决办法

### CORS

CORS需要浏览器和服务器同时支持。目前，所有的浏览器都支持这个功能。注意：整个CORS通信过程，都是浏览器自动完成，不需要用户参与。**对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。**

浏览器将CORS请求分为两类：**简单请求**和**非简单请求**

#### 简单请求

只要同时满足以下两大条件，就属于**简单请求**。

```
（1） 请求方法是以下三种方法之一：（也就是说如果你的请求方法是什么put、delete等肯定是非简单请求）
HEAD
GET
POST
（2）HTTP的头信息不超出以下几种字段：（如果比这些请求头多，那么一定是非简单请求）
Accept
Accept-Language
Content-Language
Last-Event-ID
Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain，也就是说，如果你发送的application/json格式的数据，那么肯定是非简单请求，vue的axios默认的请求体信息格式是json的，ajax默认是urlencoded的。
```

#### 非简单请求

凡是不同时满足上面两个条件，就属于**非简单请求**

实例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h2>s1的首页</h2>
<button id="btn">Ajax请求</button>
<script src="https://cdn.bootcss.com/jquery/3.4.0/jquery.js"></script>
<script>
    $('#btn').click(function () {
        $.ajax({
            url:'http://127.0.0.1:8001/books/',
            type:'post',
            contentType:'application/json',//非简单请求，会报错：Request header field content-type is not allowed by Access-Control-Allow-Headers in preflight response.
            data:JSON.stringify({
               a:'1'
            }),
            success:function (response) {
                console.log(response);
            }
        })
    })
</script>
</body>
</html>
```

#### 简单请求和非简单请求的区别

```
简单请求：一次请求
非简单请求：两次请求，在发送数据之前会先发一次请求用于做“预检”，只有“预检”通过后才再发送一次请求用于数据传输。

关于“预检”
- 请求方式：OPTIONS
- “预检”其实做检查，检查如果通过则允许传输数据，检查不通过则不再发送真正想要发送的消息
- 如何“预检”
     => 如果复杂请求是PUT等请求，则服务端需要设置允许某请求，否则“预检”不通过
        Access-Control-Request-Method
     => 如果复杂请求设置了请求头，则服务端需要设置允许某请求头，否则“预检”不通过
        Access-Control-Request-Headers
```

#### 服务器怎么处理

在我们额views.py下：

```python
from django.shortcuts import render
from django.http import JsonResponse

def books(request):
    obj = JsonResponse(['西游记2','三国演义2','水浒传2'],safe=False)
    #支持简单请求
    # obj["Access-Control-Allow-Origin"] = "*"    #支持所有的ip访问
    obj["Access-Control-Allow-Origin"] = "http://127.0.0.1:8000"
    #处理预检的options请求，这个预检的响应，我们需要在响应头里面加上下面的内容
    
    #支持非简单请求
    if request.method == 'OPTIONS':
        obj['Access-Control-Allow-Headers'] = "content-type" #"Content-Type",首字母小写也行。这个content-type的意思是，什么样的请求体类型数据都可以，我们前面说了content-type等于application/json时，是复杂请求，复杂请求先进行预检，预检的响应中我们加上这个，就是告诉浏览器，不要拦截
        obj['Access-Control-Allow-Methods'] = "DELETE,PUT"  #通过预检的请求方法设置
    return obj
```

**小结**

支持跨域，简单请求（一步操作）

```html
服务器设置响应头：Access-Control-Allow-Origin = '域名' 或 '*'
```

支持跨域，非简单请求（两步操作）

```
由于复杂请求时，首先会发送“预检”请求，如果“预检”成功，则发送真实数据。
	--　“预检”请求时，允许请求方式则需服务器设置响应头：Access-Control-Request-Method
	--“预检”请求时，允许请求头则需服务器设置响应头：Access-Control-Request-Headers
```

说白了，就是请求的时候，浏览器报错，显示什么错误我们就去在后台返回的时候加上什么样的消息，告诉浏览器，我让这条消息跨域。