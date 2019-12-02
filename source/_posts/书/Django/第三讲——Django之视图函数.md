---
title: 第三讲——Django之视图层
id: 3
date: 2019-9-26 20:00:00
tags: Django
comment: true
---

### 视图层

视图层对外接收用户的请求，对内调度模型层和模板层，连接数据库和前端，最后根据业务逻辑，将处理好的数据，与前端结合，返回给用户。

Django的视图层包含的内容：

- URL路由
- 视图函数
- 快捷方式
- 装饰器
- 请求与响应
- 类视图
- 文件上传
- CSV、PDF生成
- 内置中间件

### 视图函数View

说明：一个视图函数（类），简称视图，是一个简单的Python 函数（类），它接受Web请求并且返回Web响应。响应可以是一张网页的HTML内容，一个重定向，一个404错误，一个XML文档，或者一张图片。

注意：每个视图至少做两件事情：返回一个包含请求页面的HTPResponse对象或者弹出一个类似http404的异常。其他的则随便，你爱干嘛干嘛。

### HTTPRequest对象祥讲

说明：当一个页面被请求时，Django就会创建一个包含本次请求原信息的HttpRequest对象。Django会将这个对象自动传递给响应的视图函数，一般视图函数约定俗成地使用 request 参数承接这个对象。

超链接：https://chpl.top/2019/08/22/%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A8%A1%E5%9D%97/requests%E2%80%94%E2%80%94%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E6%A8%A1%E5%9D%97/

### HTTPResponse对象详讲

说明：与由Django自动创建的HttpRequest对象相比，HttpResponse对象是我们的职责范围了。我们写的每个视图都需要实例化，填充和返回一个HttpResponse。HttpResponse类位于django.http模块中。

属性：

HttpResponse.content：响应内容

HttpResponse.charset：响应内容的编码

HttpResponse.status_code：响应的状态码

<!----more---->

Django在`django.shortcuts`模块中，为我们提供了很多快捷方便的类和方法，它们都很重要，使用频率很高。

#### **HttpResponse()**

括号内直接跟一个具体的字符串作为响应体。

render(request, template_name, context=None, content_type=None, status=None, using=None):回复html页面，结合一个给定的模板和一个给定的上下文字典，并返回一个渲染后的 HttpResponse 对象。

```python
参数说明：
request:用于生成响应的请求对象
template_name:要使用的模板的完整名称，可选的参数，如html文件
context=None:添加到模板上下文的一个字典。默认是一个空字典。如果字典中的某个值是可调用的，视图将在渲染模板之前调用它。
content_type=None:生成的文档要使用的MIME类型。默认为 DEFAULT_CONTENT_TYPE 设置的值。默认为'text/html'
status=None:响应的状态码。默认为200
using=None:用于加载模板的模板引擎的名称
实例：
from django.shortcuts import render
def my_view(request):
    return render(request,'my.html',{'foo':'bar'})

#使用HttpResponse来重写上面的代码
from django.http import HTTPResponse
from django.template import loader
def my_view(request):
    t = loader.get_template('my.html')
    c = {'foo':'bar'}
    return HttpResponse(t.render(c,request))
```

#### **render()**

在实际运用中，加载模板、传递参数，返回HttpResponse对象是一整套再常用不过的操作了，为了节省力气，Django提供了一个快捷方式：render函数，一步到位！

```python
格式：
return render(request, template_name, context=None, content_type=None, status=None, using=None)[source]
#必须参数：
request：视图函数处理的当前请求，封装了请求头的所有数据，其实就是视图参数request。
template_name：要使用的模板的完整名称或者模板名称的列表。如果是一个列表，将使用其中能够查找到的第一个模板。
#可选参数：
：添加到模板上下文的一个数据字典。默认是一个空字典。可以将认可需要提供给模板的数据以字典的格式添加进去。这里有个小技巧，使用Python内置的locals()方法，可以方便的将函数作用于内的所有变量一次性添加。
content_type：用于生成的文档的MIME类型。 默认为DEFAULT_CONTENT_TYPE设置的值。
status：响应的状态代码。 默认为200。
using：用于加载模板使用的模板引擎的NAME。
```

#### **redirect()**

默认返回一个临时的重定向，传递`permanent=True` 可以返回一个永久的重定向。

```python
格式：
return redirect(to, permanent=False, args, *kwargs)[source]
参数to可以是：
1. 一个模型：将调用模型的get_absolute_url() 函数，反向解析出目的url；
2.一个视图，可以带有参数：将使用urlresolvers.reverse 来反向解析名称
3.一个绝对的或相对的URL：将原封不动的作为重定向的位置。
#实例：
1.传递一个ORM对象：
from django.shortcuts import redirect
 
def my_view(request):
    object = MyModel.objects.get(...)
    return redirect(object)
2.传递一个视图的名称
def my_view(request):
    return redirect('some-view-name', foo='bar')
3.传递要重定向到的一个具体的网址
def my_view(request):
    return redirect('/some/url/')
4.一个完整的网址
def my_view(request):
    return redirect('http://example.com/')
```

重定向登录跳转的实例：

```python
#urls.py
from django.conf.urls import url
from django.contrib import admin
from app01 import views
urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^index/', views.index),
    url(r'^login/', views.login,name='xxx'),
]

#views.py
from django.shortcuts import render,HttpResponse,redirect

def index(request):
    return render(request,'index.html',{'name':'chao'})
def login(request):
    method = request.method
    if method == 'GET':
        return render(request,'login.html')
    else:
        username = request.POST.get('username')
        password = request.POST.get('password')
        if username == 'chao' and password == '123':
            return redirect('/index/') #重定向到/index/路径，这也是发送了一个请求，别忘了在上面引入这个redirect类，和render、Httpresponse在一个地方引入
            # return render(request,'index.html')#如果直接用render来返回页面，是一次响应就返回了页面，两者是有区别的，并且如果你用render返回index.html页面，那么这个页面里面的模板渲染语言里面需要的数据你怎么搞，如果这些数据就是人家index那个函数里面独有的呢，你怎么搞，有人可能就响了，我把所有的数据都拿过来不就行了吗，首先如果数据量很大的话，是不是都重复了，并且你想想如果用户登陆完成之后，你们有进行跳转，那么如果网速不太好，卡一下，你想刷新一下你的页面，你是不是相当于又发送了一个login请求，你刷新完之后，是不是还要让你输入用户名和密码，你想想是不是，所有咱们一般在登陆之后都做跳转。
            # 并且大家注意一个问题昂：redirect('/login/')如果你重定向到你当前这个函数对应的路径下，你想想是什么想过，一直重定向自己的这个网址，浏览器会报错，当然这个注册登陆页面不会出现这个报错的情况，因为需要你用户点击提交才发送请求。你可以试试那个index函数，里面返回一个redirect('/index/')
            #redirect本质上也是一个HttpResponse的操作，看看源码就知道了
            # return HttpResponse('success')
        else:
            return HttpResponse('失败')
```

关于301和302

```python
1）301和302的区别。
　　301和302状态码都表示重定向，就是说浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址，这个地址可以从响应的Location首部中获取
  （用户看到的效果就是他输入的地址A瞬间变成了另一个地址B）——这是它们的共同点。

　　他们的不同在于。301表示旧地址A的资源已经被永久地移除了（这个资源不可访问了），搜索引擎在抓取新内容的同时也将旧的网址交换为重定向之后的网址；
　　302表示旧地址A的资源还在（仍然可以访问），这个重定向只是临时地从旧地址A跳转到地址B，搜索引擎会抓取新的内容而保存旧的网址。 SEO302好于301

2）重定向原因：
（1）网站调整（如改变网页目录结构）；
（2）网页被移到一个新地址；
（3）网页扩展名改变(如应用需要把.php改成.Html或.shtml)。
        这种情况下，如果不做重定向，则用户收藏夹或搜索引擎数据库中旧地址只能让访问客户得到一个404页面错误信息，访问流量白白丧失；再者某些注册了多个域名的
    网站，也需要通过重定向让访问这些域名的用户自动跳转到主站点等。
    
临时重定向（响应状态码：302）和永久重定向（响应状态码：301）对普通用户来说是没什么区别的，它主要面向的是搜索引擎的机器人。

A页面临时重定向到B页面，那搜索引擎收录的就是A页面。
A页面永久重定向到B页面，那搜索引擎收录的就是B页面。
```

**JsonResponse**对象：JsonResponse是HttpResponse的子类，专门用来生成JSON编码的响应。

```python
实例：
from django.http import JsonResponse
	response = JsonResponse({'foo':'bar'})
    print(response.content)
结果：
b'{"foo":"bar"}'
```

#### get_object_or_404()

```python
格式：
return get_object_or_404(klass, args, *kwargs)[source]
#这个方法，非常有用，请一定熟记。常用于查询某个对象，找到了则进行下一步处理，如果未找到则给用户返回404页面。
在后台，Django其实是调用了模型管理器的get()方法，只会返回一个对象。不同的是，如果get()发生异常，会引发Http404异常，从而返回404页面，而不是模型的DoesNotExist异常。#就是get()在模型中，get()方法找不到对象不返回异常而是返回404页面

必须参数：
klass：要获取的对象的Model类名或者Queryset等；
**kwargs:查询的参数，格式应该可以被get()接受。
#实例一：从MyModel中使用主键1来获取对象
from django.shortcuts import get_object_or_404
def my_view(request):
    my_object = get_object_or_404(MyModel, pk=1)
#实例二：除了传递Model名称，还可以传递一个QuerySet实例
get_object_or_404(Book, title__startswith='M', pk=1)
#实例三：还可以使用Manager
get_object_or_404(Book.dahl_objects, title='Matilda')
```

更多属性和方法参见：http://www.liujiangblog.com/course/django/140

### 通用视图

视图的代码非常相似，有点冗余，这是一个程序猿不能忍受的。他们都具有类似的业务逻辑，实现类似的功能：通过从URL传递过来的参数去数据库查询数据，加载一个模板，利用刚才的数据渲染模板，返回这个模板。由于这个过程是如此的常见，Django很善解人意的帮你想办法偷懒，于是它提供了一种快捷方式，名为“通用视图”。

### CBV和FBV

CBV**(function base views)**:在视图里面使用类处理请求。

FBV**(class base views)**:在视图里面使用函数处理请求。

Python是一个面向对象的编程语言，如果只用函数来开发，有很多面向对象的优点就错失了（继承、封装、多态）。所以Django在后来加入了Class-Based-View。可以让我们用类写View。这样做的优点主要下面两种：

- 提高了代码的复用性，可以使用面向对象的技术，比如Mixin（多继承）

- 可以用不同的函数针对不同的HTTP方法处理，而不是通过很多if判断，提高代码可读性

对比：

```python
#写一个处理GET方法的view，用函数写的话是下面这样
from django.http import HttpResponse
  
def my_view(request):
     if request.method == 'GET':
            return HttpResponse('OK')

#如果用class-based view写的话，就是下面这样
from django.http import HttpResponse
from django.views import View
  
class MyView(View):

      def get(self, request):
            return HttpResponse('OK')
解释：
Django的url是将一个请求分配给可调用的函数的，而不是一个class。针对这个问题，class based view提供了一个as_view()静态方法（也就是类方法），调用这个方法，会创建一个类的实例，然后通过实例调用dispatch()方法，dispatch()方法会根据request的method的不同调用相应的方法来处理request（如get() , post()等）。到这里，这些方法和function-based view差不多了，要接收request，得到一个response返回。如果方法没有定义，会抛出HttpResponseNotAllowed异常。
#对应的urls.py文件的对应修改
from django.conf.urls import url
from myapp.views import MyView #引入我们在views.py里面创建的类
  
urlpatterns = [
     url(r'^index/$', MyView.as_view()),
]

#执行流程:调用View里面的as_view()静态方法，这个方法里面会执行一个view方法，这个方法去调用执行dispacth()方法，在这个方法里面通过反射的方式查找request的请求，与View函数中定义的一个列表中的方法进行匹配，如果有就可以自动执行自己类中定义的对应的方法，否则就会报错。

#现在让我们来看看这个View实现的源码：源码中通过dispatch反射来处理的
class View:
    #这一行源码就是我们可以在类中定义的方法，View帮我们做好了封装我们直接以该名称命名，路由就会自动找到对应的类中的函数的
    http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']

    def __init__(self, **kwargs):
        #遍历关键字参数，然后将它们的值保存到
        for key, value in kwargs.items():
            setattr(self, key, value)

    @classonlymethod
    def as_view(cls, **initkwargs):
        """这个就是我们在urls中调用的方法，在这个方法中，传入的参数是cls，这个是类对象，那么传进来的就是我们的类名，也就是我们写在view中的类名。后面的关键字参数就是我们view视图中类下的方法名"""
        #判断view下的类中是否存在以上的这些方法名
        for key in initkwargs:
            if key in cls.http_method_names:
                raise TypeError("You tried to pass in the %s method name as a "
                                "keyword argument to %s(). Don't do that."
                                % (key, cls.__name__))
            if not hasattr(cls, key):
                raise TypeError("%s() received an invalid keyword %r. as_view "
                                "only accepts arguments that are already "
                                "attributes of the class." % (cls.__name__, key))
		#这个方法就是我们的主要逻辑实现方法
        def view(request, *args, **kwargs):
            self = cls(**initkwargs)
            if hasattr(self, 'get') and not hasattr(self, 'head'):
                self.head = self.get
            self.setup(request, *args, **kwargs)
            if not hasattr(self, 'request'):
                raise AttributeError(
                    "%s instance has no 'request' attribute. Did you override "
                    "setup() and forget to call super()?" % cls.__name__
                )
            
            return self.dispatch(request, *args, **kwargs)
        view.view_class = cls
        view.view_initkwargs = initkwargs

        # take name and docstring from class
        update_wrapper(view, cls, updated=())

        # and possible attributes set by decorators
        # like csrf_exempt from dispatch
        update_wrapper(view, cls.dispatch, assigned=())
        return view

    def setup(self, request, *args, **kwargs):
        """Initialize attributes shared by all view methods."""
        self.request = request
        self.args = args
        self.kwargs = kwargs

    #这个就是我们通过反射判断view中类是否存在上面列表中的方法名
    def dispatch(self, request, *args, **kwargs):
        # Try to dispatch to the right method; if a method doesn't exist,
        # defer to the error handler. Also defer to the error handler if the
        # request method isn't on the approved list.
        if request.method.lower() in self.http_method_names:
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed
        return handler(request, *args, **kwargs)

    def http_method_not_allowed(self, request, *args, **kwargs):
        logger.warning(
            'Method Not Allowed (%s): %s', request.method, request.path,
            extra={'status_code': 405, 'request': request}
        )
        return HttpResponseNotAllowed(self._allowed_methods())

    def options(self, request, *args, **kwargs):
        """Handle responding to requests for the OPTIONS HTTP verb."""
        response = HttpResponse()
        response['Allow'] = ', '.join(self._allowed_methods())
        response['Content-Length'] = '0'
        return response

    def _allowed_methods(self):
        return [m.upper() for m in self.http_method_names if hasattr(self, m)]
```

### 给视图加装饰器

#### 使用装饰器装饰FBV

```python
def wrapper(func):
    def inner(*args, **kwargs):
        start_time = time.time()
        ret = func(*args, **kwargs)
        end_time = time.time()
        print("used:", end_time-start_time)
        return ret
    return inner


# FBV版添加班级
@wrapper
def add_class(request):
    if request.method == "POST":
        class_name = request.POST.get("class_name")
        models.Classes.objects.create(name=class_name)
        return redirect("/class_list/")
    return render(request, "add_class.html")
```

#### 使用装饰器装饰CBV

注意：类中的方法与独立函数不完全相同，因此不能直接将函数装饰器应用于类中的方法 ，我们需要先将其转换为方法装饰器。

Django中提供了method_decorator装饰器用于将函数装饰器转换为方法装饰器。

```python
from django.shortcuts import render,redirect,HttpResponse
from django.urls import reverse
from django.utils.decorators import method_decorator
from django.views import View
def wrapper(fn):
    def inner(request,*args,**kwargs):
        print('xxxxx')
        ret = fn(request)
        print('xsssss')
        return ret
    return inner

# @method_decorator(wrapper,name='get')#CBV版装饰器方式一:直接添加在类上，后面的name表示只给get添加装饰器
class BookList(View):
    
    @method_decorator(wrapper) #CBV版装饰器方式二:直接添加在dispatch里面，这样每个函数都会执行
    def dispatch(self, request, *args, **kwargs):
        print('请求内容处理开始')
        res = super().dispatch(request, *args, **kwargs)  #这里调用父类的，也就是View下的dispacth方法，同时我们也在他的外面定义一个dispacth用来加上我们的一些逻辑
        print('处理结束')
        return res
    def get(self,request):
        print('get内容')
        # all_books = models.Book.objects.all()
        return render(request,'login.html')
    
    @method_decorator(wrapper) #CBV版装饰器方式三:添加在每一个函数中
    def post(self,request):
        print('post内容')
        return redirect(reverse('book_list'))
# @wrapper
def book_list(request):
    
    return HttpResponse('aaa')
```

总结：

- 添加装饰器前必须导入from django.utils.decorators import method_decorator
- 添加装饰器的格式必须为@method_decorator()，括号里面为装饰器的函数名
- 给类添加是必须声明name
- 注意csrf-token装饰器的特殊性，在CBV模式下它只能加在dispatch上面(后面再说)

下面这是csrf_token的装饰器：

- @csrf_protect，为当前函数强制设置防跨站请求伪造功能，即便settings中没有设置csrfToken全局中间件。

- @csrf_exempt，取消当前函数防跨站请求伪造功能，即便settings中设置了全局中间件。
- from django.views.decorators.csrf import csrf_exempt,csrf_protect

### 文件上传（待）

详细见老师的总结。

### 文件下载(待)

详细见老师的总结。

### 动态生成CSV文件

CSV (Comma Separated Values)，以纯文本形式存储数字和文本数据的存储方式。纯文本意味着该文件是一个字符序列，不含必须像二进制数字那样的数据。CSV文件由任意数目的记录组成，记录间以某种换行符分隔；每条记录由字段组成，字段间的分隔符是其它字符或字符串，最常见的是逗号或制表符。通常，所有记录都有完全相同的字段序列。

```python
#实例：
from django.http import HttpResponse

def some_view(request):
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'

    writer = csv.writer(response)
    writer.writerow(['First row', 'Foo', 'Bar', 'Baz'])
    writer.writerow(['Second row', 'A', 'B', 'C', '"Testing"', "Here's a quote"])

    return response
```

- 响应对象的MIME类型设置为`text/csv`，告诉浏览器，返回的是一个CSV文件而不是HTML文件。
- 响应对象设置了附加的`Content-Disposition`协议头，含有CSV文件的名称。文件名随便取，浏览器会在“另存为...”对话框等环境中使用它。
- 要在生成CSV的API中使用钩子非常简单：只需要把response作为第一个参数传递给`csv.writer`。`csv.writer`方法接受一个类似于文件的对象，而HttpResponse对象正好就是这么个东西。
- 对于CSV文件的每一行，调用`writer.writerow`，向它传递一个可迭代的对象比如列表或者元组。
- CSV模板会为你处理各种引用，不用担心没有转义字符串中的引号或者逗号。只需要向writerow()传递你的原始字符串，它就会执行正确的操作。

