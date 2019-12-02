---
title: 第九讲——Django之ajax
id: 9
date: 2019-10-11 20:00:00
tags: Django
comment: true
---

### AJAX

#### Ajax简介

AJAX（Asynchronous Javascript And XML）翻译成中文就是“异步的Javascript和XML”。

AJAX的特点:

- 异步请求：客户端发出一个请求后，无需等待服务器响应结束，就可以发出第二个请求
- 浏览器页面局部刷新

<!----more---->

#### 实例

需求：模拟用户登录，提交数据之后：用户名或密码错误返回数据ajax局部显示“用户名或密码错误”，输入成功之后，返回主页页面。

login.html

```html
{% load static %}  #引入自己在app里面创建的静态文件夹
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    用户名: <input type="text" id="username">
    密码: <input type="password" id="password">
    <button id="sub">提交</button>
    <span class="error"></span>

</body>

<!--引入静态文件夹的jquery文件-->
<script src="{% static 'js/jquery.js' %}"></script>
<script>

    $('#sub').click(function () {  #点击登录按钮动作

        var uname = $('#username').val();
        var pwd = $('#password').val();

        $.ajax({
            url:'{% url "login" %}',  #指定url
            type:'post',    #指定发送数据格式
            data:{username:uname,password:pwd}, 
            success:function (res) {  #成功接收后台返回的数据
                if (res === '1'){ #后台返回字符串‘1’
                    location.href = '/home/'; // http://127.0.0.1:8000/home/ 登录成功打开主页页面

                }else{
                    $('.error').text('用户名密码错误!');
                }
            }
        })
    })
</script>
</html>
```

views.py

```python
from django.shortcuts import render,HttpResponse,redirect

def login(request):
    if request.method == 'GET':
        return render(request,'login.html')
    else:
        uname = request.POST.get('username')
        pwd = request.POST.get('password')

        if uname == 'dui' and pwd == 'gang':
            return HttpResponse('1')
        else:
            return HttpResponse('0')

def home(request):
    return render(request,'home.html')
```

#### ajax常见应用场景

- 搜索引擎根据用户输入的关键字，自动提示检索关键字
- 注册时候的用户名的查重（这样，整个过程没有页面的刷新，只是在输入框后下面进行提示，用户体验就非常好）

![](http://9017499461.linshutu.top/ajax-register.png)

#### ajax的参数

**请求参数**：

```python
######################------------data---------################

       data: 当前ajax请求要携带的数据，是一个json的object对象，ajax方法就会默认地把它编码成某种格式
             (urlencoded:?a=1&b=2)发送给服务端；此外，ajax默认以get方式发送请求。

             function testData() {
               $.ajax("/test",{     //此时的data是一个json形式的对象
                  data:{
                    a:1,
                    b:2
                  }
               });                   //?a=1&b=2
######################------------processData---------################

processData：声明当前的data数据是否进行转码或预处理，默认为true，即预处理；if为false，
             那么对data：{a:1,b:2}会调用json对象的toString()方法，即{a:1,b:2}.toString()
             ,最后得到一个［object，Object］形式的结果。
            
######################------------contentType---------################

contentType：默认值: "application/x-www-form-urlencoded"。发送信息至服务器时内容编码类型。
             用来指明当前请求的数据编码格式；urlencoded:?a=1&b=2；如果想以其他方式提交数据，
             比如contentType:"application/json"，即向服务器发送一个json字符串：
               $.ajax("/ajax_get",{
             
                  data:JSON.stringify({
                       a:22,
                       b:33
                   }),
                   contentType:"application/json",
                   type:"POST",
             
               });                          //{a: 22, b: 33}

             注意：contentType:"application/json"一旦设定，data必须是json字符串，不能是json对象

             views.py:   json.loads(request.body.decode("utf8"))


######################------------traditional---------################

traditional：一般是我们的data数据有数组时会用到 ：data:{a:22,b:33,c:["x","y"]},
              traditional为false会对数据进行深层次迭代；
```

**补充**

```
username
类型：String
用于响应 HTTP 访问认证请求的用户名。

password
类型：String
用于响应 HTTP 访问认证请求的密码

url
类型：String
默认值: 当前页地址。发送请求的地址。

type
类型：String
默认值: "GET")。请求方式 ("POST" 或 "GET")， 默认为 "GET"。注意：其它 HTTP 请求方法，如 PUT 和 DELETE 也可以使用，但仅部分浏览器支持。

timeout
类型：Number
设置请求超时时间（毫秒）。此设置将覆盖全局设置。

scriptCharset
类型：String
只有当请求时 dataType 为 "jsonp" 或 "script"，并且 type 是 "GET" 才会用于强制修改 charset。通常只在本地和远程的内容编码不同时使用。

global
类型：Boolean
是否触发全局 AJAX 事件。默认值: true。设置为 false 将不会触发全局 AJAX 事件，如 ajaxStart 或 ajaxStop 可用于控制不同的 Ajax 事件。

dataType
类型：String
预期服务器返回的数据类型。


```

**响应参数**：

```python
如果要处理 $.ajax() 得到的数据，则需要使用回调函数：beforeSend、error、dataFilter、success、complete。

success
当请求之后调用。传入返回后的数据，以及包含成功代码的字符串。

error
在请求出错时调用。传入 XMLHttpRequest 对象，描述错误类型的字符串以及一个异常对象（如果有的话）

complete
当请求完成之后调用这个函数，无论成功或失败。传入 XMLHttpRequest 对象，以及一个包含成

beforeSend
在发送请求之前调用，并且传入一个 XMLHttpRequest 作为参数。

dataFilter
在请求成功之后调用。传入返回的数据以及 "dataType" 参数的值。并且必须返回新的数据（可能是处理过的）传递给 success 回调函数。

context
类型：Object
这个对象用于设置 Ajax 相关回调函数的上下文。
$.ajax({ url: "test.html", 
        context: document.body, 
        success: function(){
        	$(this).addClass("done");
      }});

complete(XHR, TS)
类型：Function
请求完成后回调函数 (请求成功或失败之后均调用)。

cache
类型：Boolean
默认值: true，dataType 为 script 和 jsonp 时默认为 false。设置为 false 将不缓存此页面。
    
async
类型：Boolean
默认值: true。默认设置下，所有请求均为异步请求。如果需要发送同步请求，请将此选项设置为 false。
注意，同步请求将锁住浏览器，用户其它操作必须等待请求完成才可以执行。
```

### csrf

### csrf_token

简介：CSRF（Cross-site request forgery），中文名称：跨站请求伪造，缩写为：CSRF/XSRF。**攻击者通过HTTP请求将数据传送到服务器，从而盗取会话的cookie。盗取回话cookie之后，攻击者不仅可以获取用户的信息，还可以修改该cookie关联的账户信息。**

![](http://9017499461.linshutu.top/csrf%E5%8E%9F%E7%90%86.png)

所以解决csrf攻击的最直接的办法就是生成一个随机的csrf_token值，保存在用户的页面上，每次请求都带着这个值过来完成校验。

#### form表单设置csrf_token

```html
<form action="" method="post">
    {% csrf_token %}  <!--在这里设置-->
    用户名: <input type="text" name="username">
    密码: <input type="password" name="password">
    <input type="submit">
</form>
```

#### ajax设置csrf认证

```html
<script>
<!--方式1-->
    $.ajax({
      url: "/cookie_ajax/",
      type: "POST",
      data: {
        "username": "pl",
        "password": 123456,
        "csrfmiddlewaretoken": $("[name = 'csrfmiddlewaretoken']").val()  #通过获取隐藏的input标签中的csrfmiddlewaretoken值，放置在data中发送。
      },
      success: function (data) {
        console.log(data);
      }
    })

<!--方式2:就是修改data里面的数据，其他的不变-->
	$.ajax({
        data: {csrfmiddlewaretoken: '{{ csrf_token }}' },
    });
    
<!--方式3：把csrf放在请求头里面，推荐使用-->
	$.ajax({
 
		headers:{"X-CSRFToken":$.cookie('csrftoken')}, 　#通过获取返回的cookie中的字符串 放置在请求头中发送。
 
})   
</script>
```

jquery操作cookie: https://www.cnblogs.com/clschao/articles/10480029.html

#### django中csrf认证的原理

token字符串的前32位是salt， 后面是加密后的token， 通过salt能解密出唯一的secret。
django会验证表单中的token和cookie中token是否能解出同样的secret，secret一样则本次请求合法。
同样也不难解释，为什么ajax请求时，需要从cookie中拿取token添加到请求头中。

个人的解释：client访问服务器，服务器给cilent一个csrf令牌（一个字符串，加密的）以及一个cookie随机码。下一次客户端再来的时候，dj就解密cookie和csrf中的密码是不是一样，一样就是合法的，不一样就是不合法的。

更多细节详见：[Djagno官方文档中关于CSRF的内容](https://docs.djangoproject.com/en/1.11/ref/csrf/)

#### csrf后台免除认证

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

### 文件上传

#### form表单上传文件

xx.html

```html
<form action="" method="post" enctype="multipart/form-data">  别忘了enctype
    {% csrf_token %}
    用户名: <input type="text" name="username">
    密码: <input type="password" name="password">
    <!--删除头像相关的文件，自动有一个选择按钮-->
    头像: <input type="file" name="file">  

    <input type="submit">
</form>
```

view.py

```python
def upload(request):

    if request.method == 'GET':
        return render(request,'upload.html')

    else:
        print(request.POST)
        print(request.FILES)
        uname = request.POST.get('username')
        pwd = request.POST.get('password')

        file_obj = request.FILES.get('file')  #获取上传来的文件对象

        print(file_obj.name) #开班典礼.pptx,文件名称
		
        with open(file_obj.name,'wb') as f:
            #遍历文件对象的句柄，保存文件
            # for i in file_obj:
            #     f.write(i)
            #这种方式的利用django里面的自带的方式，确保每次取得数据都是固定的少量的，上面的那种方式只要不遇到\r或者\n就会一直读取数据并一次性把数据取出来，这样做是不对的。
            for chunk in file_obj.chunks(): #默认每次取65536字节
                f.write(chunk)
        return HttpResponse('ok')
```

####  ajax上传文件

views.py文件和上面是一样的

xx.html

```html
<script>
$('#sub').click(function () {
  		#创建一个form对象，就是为了马上把下面传输的数据包裹在里面。因为ajax内部没有content-type=mutil...的方法，那么传输的数据就是urlencoded编码方式的数据，这样传输的数据在后端解析非常麻烦。
        var formdata = new FormData();
	   
        var uname = $('#username').val();
        var pwd = $('#password').val();

        var file_obj = $('[type=file]')[0].files[0]; // js获取文件对象

    	#将页面的数据封装到form对象中
        formdata.append('username',uname);
        formdata.append('password',pwd);
        formdata.append('file',file_obj);

        $.ajax({
            url:'{% url "upload" %}',
            type:'post',
			#现在data传输form数据就可以了
            data:formdata,
            #固定的，必须写
            processData:false,  // 必须写
            contentType:false,  // 必须写

            headers:{
                "X-CSRFToken":$.cookie('csrftoken'),
            },
            success:function (res) {
                console.log(res);
                if (res === '1'){
                    // $('.error').text('登录成功');
                    location.href = '/home/'; // http://127.0.0.1:8000/home/

                }else{
                    $('.error').text('用户名密码错误!');
                }
            }
        })
    })
</script>
```

### 关于json

```html
<!--接口之间的数据交互-->
<script>
$.ajax({
            url:'{% url "jsontest" %}',
            type:'post',
            // data:{username:uname,password:pwd,csrfmiddlewaretoken:csrf},
 //data:JSON.stringify({username:uname,password:pwd}),
            data:{username:uname,password:pwd},
            headers:{
                // contentType:'application/json',
                "X-CSRFToken":$.cookie('csrftoken'),
            },
            success:function (res) {
                {#console.log(res,typeof res); // statusmsg {"status": 1001, "msg": "登录失败"}#}
                {#var res = JSON.parse(res);  //-- json.loads()#}
                console.log(res,typeof res);  //直接就是反序列化之后的了
                //JSON.stringify()  -- json.dumps
                if (res.status === 1000){
                    // $('.error').text('登录成功');
                    location.href = '/home/'; // http://127.0.0.1:8000/home/

                }else{
                    $('.error').text(res.msg);
                }
            }
        })
</script>
```

### jsonResponse

之前学的从视图里面返回给前端的数据和网页的方法有4中，其中有一种就是JsonResponse，但是我们当时它的应用场景是什么，下面我们有使用一个例子简化上面的json数据的交换（注意：Django没有自带的json数据解析器）

#### 实例

view.py

```python
from django.http import JsonResponse


        username = request.POST.get('username')
        pwd = request.POST.get('password')
        ret_data = {'status':None,'msg':None}
        print('>>>>>',request.POST)
        #<QueryDict: {'{"username":"123","password":"123"}': ['']}>
        if username == 'chao' and pwd == '123':
            ret_data['status'] = 1000  # 状态码
            ret_data['msg'] = '登录成功'
        else:
            ret_data['status'] = 1001  # 状态码
            ret_data['msg'] = '登录失败'
        # ret_data_json = json.dumps(ret_data,ensure_ascii=False)
        # return HttpResponse(ret_data_json,content_type='application/json')
         
        return JsonResponse(ret_data)  #非常的简洁
    	#对于传输非字典类型数据，加一个参数就可以
    	ret = [1,2,3]
        return JsonResponse(ret_data,safe=False)
    	
```

html

```html
<script>
 $.ajax({
            url:'{% url "jsontest" %}',
            type:'post',
            data:{username:uname,password:pwd},
            headers:{
                // contentType:'application/json',
                "X-CSRFToken":$.cookie('csrftoken'),
            },
            success:function (res) {
                {#console.log(res,typeof res); statusmsg {"status": 1001, "msg": "登录失败"}#}
                {#var res = JSON.parse(res);  //-- json.loads()#}
                console.log(res,typeof res);  //直接就是反序列化之后的了
                //JSON.stringify()  -- json.dumps
                if (res.status === 1000){
                    #从字典中解析出来的数据
                    location.href = '/home/'; // http://127.0.0.1:8000/home/
                }else{
                    $('.error').text(res.msg);
                }
            }
        })
</script>
```

