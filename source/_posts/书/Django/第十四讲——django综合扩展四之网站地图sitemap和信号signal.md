---
title: 第十四讲——django综合扩展四之网站地图sitemap和信号signal
id: 14
date: 2019-10-16 20:30:00
tags: Django
comment: true
---

### 网站地图sitemap

网站地图是根据网站的结构、框架、内容，生成的导航网页，是一个网站所有链接的容器。很多网站的连接层次比较深，蜘蛛很难抓取到，网站地图可以方便搜索引擎或者网络蜘蛛抓取网站页面，了解网站的架构，为网络蜘蛛指路，增加网站内容页面的收录概率。网站地图一般存放在域名根目录下并命名为sitemap。

Django自带了一个高级的生成网站地图的框架，我们可以很容易地创建出XML格式的网站地图。创建网站地图，只需编写一个Sitemap类，并在URLconf中编写对应的访问路由。

<!----more---->

#### 安装

- 在INSTALLED_APPS设置中添加'django.contrib.sitemaps' .
- 确认settings.py中的`TEMPLATES`设置包含`DjangoTemplates`后端，并将`APP_DIRS`选项设置为True。其实，默认配置就是这样的，只有当你曾经修改过这些设置，才需要调整过来。
- 确认你已经安装sites框架. (注意: 网站地图APP并不需要在数据库中建立任何数据库表。修改`INSTALLED_APPS`的唯一原因是，以便`Loader()`模板加载器可以找到默认模板。）

#### 初始器

为了在网站上激活站点地图生成功能，请把以下代码添加到URLconf中:

```python
from django.contrib.sitemaps.views import sitemap

	url(r'^sitemap\.xml$', sitemap, {'sitemaps': sitemaps},
    name='django.contrib.sitemaps.views.sitemap')
```

sitemap视图需要一个额外的必需参数： `{'sitemaps': sitemaps}`。`sitemaps`应是一个字典，将部门的标签（例如news或blog）映射到其 Sitemap类（例如，NewsSitemap或BlogSitemap）。也可以映射到Sitemap类的实例（例如，BlogSitemap(some_var)）。

#### 实例

假设你有一个博客系统，拥有Entry模型，并且你希望站点地图包含指向每篇博客文章的所有链接。 以下是Sitemap类的写法：

```python
from django.contrib.sitemaps import Sitemap
from blog.models import Entry

class BlogSitemap(Sitemap):
    changefreq = "never"
    priority = 0.5

    def items(self):
        return Entry.objects.filter(is_draft=False)

    def lastmod(self, obj):
        return obj.pub_date
```

#### 快捷方式

sitemap框架提供了一个快捷类，帮助我们迅速生成网站地图：

```python
class GenericSitemap[source]
```

通过它，我们无需为sitemap编写单独的视图模块，直接在URLCONF中，获取对象，获取参数，传递参数，设置url，如下所示，一条龙服务：

```python
from django.conf.urls import url
from django.contrib.sitemaps import GenericSitemap
from django.contrib.sitemaps.views import sitemap
from blog.models import Entry

info_dict = {
    'queryset': Entry.objects.all(),
    'date_field': 'pub_date',
}

urlpatterns = [
    url(r'^sitemap\.xml$', sitemap,
        {'sitemaps': {'blog': GenericSitemap(info_dict, priority=0.6)}},
        name='django.contrib.sitemaps.views.sitemap'),
]
```

### 信号signal

django自带一套信号机制来帮助我们在框架的不同位置之间传递信息。也就是说，当某一事件发生时，信号系统可以允许一个或多个发送者（senders）将通知或信号（signals）发送给一组接受者（receivers）。

信号系统包含以下三要素：

- 发送者－信号的发出方
- 信号－信号本身
- 接收者－信号的接受者

Django内置了一整套信号，下面是一些比较常用的：

- django.db.models.signals.pre_save & django.db.models.signals.post_save

在ORM模型的save()方法调用之前或之后发送信号

- django.db.models.signals.pre_delete & django.db.models.signals.post_delete

在ORM模型或查询集的delete()方法调用之前或之后发送信号。

- django.db.models.signals.m2m_changed

当多对多字段被修改时发送信号。

- django.core.signals.request_started & django.core.signals.request_finished

当接收和关闭HTTP请求时发送信号。

#### 监听信号

要接收信号，请使用`Signal.connect()`方法注册一个接收器。当信号发送后，会调用这个接收器。

方法原型：

```python
Signal.connect(receiver, sender=None, weak=True, dispatch_uid=None)[source]
```

参数：

```python
receiver ：当前信号连接的回调函数，也就是处理信号的函数。 
sender ：指定从哪个发送方接收信号。 
weak ： 是否弱引用
dispatch_uid ：信号接收器的唯一标识符，以防信号多次发送。
```

下面以如何接收每次HTTP请求结束后发送的信号为例，连接到Django内置的现成的`request_finished`信号。

**1，编写接收器**

接收器其实就是一个Python函数或者方法：

```python
def my_callback(sender, **kwargs):
    print("Request finished!")
```

请注意，所有的接收器都必须接收一个sender参数和一个`**kwargs`通配符参数。

**2,连接接收器**

有两种方法可以连接接收器，一种是下面的手动方式：

```python
from django.core.signals import request_finished

request_finished.connect(my_callback)
```

另一种是使用receiver()装饰器：

```python
from django.core.signals import request_finished
from django.dispatch import receiver

@receiver(request_finished)
def my_callback(sender, **kwargs):
    print("Request finished!")
```

**3，接收特定发送者的信号**

一个信号接收器，通常不需要接收所有的信号，只需要接收特定发送者发来的信号，所以需要在sender参数中，指定发送方。下面的例子，只接收MyModel模型的实例保存前的信号。

```python
from django.db.models.signals import pre_save
from django.dispatch import receiver
from myapp.models import MyModel


@receiver(pre_save, sender=MyModel)
def my_handler(sender, **kwargs):
    ...
```

`my_handler`函数只在MyModel实例保存时被调用。

**4，防止重复信号**

为了防止重复信号，可以设置`dispatch_uid`参数来标识你的接收器，标识符通常是一个字符串，如下所示：

```python
from django.core.signals import request_finished

request_finished.connect(my_callback, dispatch_uid="my_unique_identifier")
```

最后的结果是，对于每个唯一的`dispatch_uid`值，你的接收器都只绑定到信号一次。

#### 自定义信号

除了Django为我们提供的内置信号（比如前面列举的那些），很多时候，我们需要自己定义信号。

类原型：class Signal(providing_args=list)[source]

所有的信号都是`django.dispatch.Signal`的实例。`providing_args`参数是一个列表，由信号将提供给监听者的参数的名称组成。可以在任何时候修改`providing_args`参数列表。

下面定义了一个新信号：

```python
import django.dispatch

pizza_done = django.dispatch.Signal(providing_args=["toppings", "size"])
```

上面的例子定义了`pizza_done`信号，它向接受者提供size和toppings 参数。

#### 发送信号

Django中有两种方法用于发送信号。

```python
Signal.send(sender, **kwargs)[source]


Signal.send_robust（sender，** kwargs）[source] 
```

必须提供sender参数（大部分情况下是一个类名），并且可以提供任意数量的其他关键字参数。

例如，这样来发送前面的`pizza_done`信号：

```python
class PizzaStore(object):
    ...

    def send_pizza(self, toppings, size):
        pizza_done.send(sender=self.__class__, toppings=toppings, size=size)
        ...
```

`send()`和`send_robust()`返回一个元组对的列表`[（receiver, response）， ... ]`，表示接收器和响应值二元元组的列表。

#### 断开信号

方法：

```python
Signal.disconnect(receiver=None, sender=None, dispatch_uid=None)[source]
```

`Signal.disconnect()`用来断开信号的接收器。和`Signal.connect()`中的参数相同。如果接收器成功断开，返回True，否则返回False。

#### 信号使用实例

信号可能不太好理解，下面我在Django内编写一个例子示范一下：

首先在根URLCONF中写一条路由：

```python
from django.conf.urls import url
from django.contrib import admin
from app1 import views

urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^signal/$', views.create_signal),
]
```

这个很好理解，我在项目里创建了一个app1应用，在它的views.py中创建了一个create_signal视图，通过`/signal/`可以访问这个视图。这些都不重要，随便配置，只要能正常工作就行。

然后在views.py中自定义一个信号，以及创建create_signal视图：

```python
from django.shortcuts import HttpResponse
import time
import django.dispatch
from django.dispatch import receiver

# 定义一个信号
work_done = django.dispatch.Signal(providing_args=['path', 'time'])


def create_signal(request):
    url_path = request.path
    print("我已经做完了工作。现在我发送一个信号出去，给那些指定的接收器。")

    # 发送信号，将请求的url地址和时间一并传递过去
    work_done.send(create_signal, path=url_path, time=time.strftime("%Y-%m-%d %H:%M:%S"))
    return HttpResponse("200,ok")
```

自定义的信号名叫`work_done`，它很简单，接收请求url地址和请求时间两个参数。

create_signal视图内，获取请求的url，生成请求的时间，作为参数，传递到send方法。

这样，我们就发送了一个信号。

然后，再写一个接收器：

```python
@receiver(work_done, sender=create_signal)
def my_callback(sender, **kwargs):
    print("我在%s时间收到来自%s的信号，请求url为%s" % (kwargs['time'], sender, kwargs["path"]))
```

通过装饰器注册为接收器。内部接收字典参数，并解析打印出来。

最终views.py文件如下：

```python
from django.shortcuts import HttpResponse
import time
import django.dispatch
from django.dispatch import receiver

# Create your views here.

# 定义一个信号
work_done = django.dispatch.Signal(providing_args=['path', 'time'])


def create_signal(request):
    url_path = request.path
    print("我已经做完了工作。现在我发送一个信号出去，给那些指定的接收器。")

    # 发送信号，将请求的IP地址和时间一并传递过去
    work_done.send(create_signal, path=url_path, time=time.strftime("%Y-%m-%d %H:%M:%S"))
    return HttpResponse("200,ok")


@receiver(work_done, sender=create_signal)
def my_callback(sender, **kwargs):
    print("我在%s时间收到来自%s的信号，请求url为%s" % (kwargs['time'], sender, kwargs["path"]))
```

现在可以来测试一下。python manage.py runserver启动服务器。浏览器中访问`http://127.0.0.1:8000/signal/`。重点不再浏览器的返回，而在后台返回的内容：

```python
我已经做完了工作。现在我发送一个信号出去，给那些指定的接收器。
我在2017-12-18 17:10:12时间收到来自<function create_signal at 0x0000000003AFF840>的信号，请求url为/signal/
```

这些提示信息，可以在Pycharm中看到，或者在命令行环境中看到。