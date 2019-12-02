---
title: 第二讲——Django之URL路由
id: 2
date: 2019-9-25 20:00:00
tags: Django
comment: true
---

## 内容明晰

**路由的编写方式是Django2.0和1.11最大的区别所在。Django官方迫于压力和同行的影响，不得不将原来的正则匹配表达式，改为更加简单的path表达式，但依然通过re_path()方法保持对1.x版本的兼容。**

URL是Web服务的入口，用户通过浏览器发送过来的任何请求，都是发送到一个指定的URL地址，然后被响应。

在Django项目中编写路由，就是向外暴露我们接收哪些URL的请求，除此之外的任何URL都不被处理，也没有返回。通俗地理解，不恰当的形容，URL路由是你的Web服务对外暴露的API。

## Django如何处理请求

- **决定要使用的根URLconf模块。**通常，这是`ROOT_URLCONF`设置的值，但是如果传入的HttpRequest对象具有urlconf属性（由中间件设置），则其值将被用于代替`ROOT_URLCONF`设置。**通俗的讲，就是你可以自定义项目入口url是哪个文件！**
- **加载该 模块并寻找可用的urlpatterens。**
- **依次匹配每个URL模式，在与请求的URL相匹配的第一个模式停下**也就是说，url匹配是从上往下的短路操作，所以url在列表中的位置非常关键。
- 导入并调用匹配中给定的视图，该视图是一个简单的python函数，也就是我们常说的视图函数，或者基于类的视图。
- 如果没有匹配到任何的表达式，或者过程中抛出异常，将调用一个适合的错误处理视图。

## URL配置（URLConf）

解释：URL配置（URLconf）就像Django所支撑网站的目录。它的本质是URL与要为该URL调用的视图之间的映射表。

#### 基本格式

**Django版本1.x**

```python
from django.conf.urls import url
urlpatterns = [
     url(正则表达式, views，kwargs，name),
]
#参数说明：
1.正则表达式:当 Django 响应一个请求时，它会从 urlpatterns 的第一项开始，按顺序依次匹配列表中的项，直到找到匹配的项，然后执行该条目映射的视图函数或下级路由，其后的条目将不再继续匹配。因此，url路由的编写顺序非常重要
    
2.views：当Django匹配到某个路由条目时，自动将封装的HttpRequest对象作为第一个参数，被“捕获”的参数以关键字参数的形式，传递给该条目指定的视图view。

3.kwargs:任意数量的关键字参数可以作为一个字典传递给目标视图。
    
4.name:对你的URL进行命名，让你能够在Django的任意处，尤其是模板内显式地引用它。这是一个非常强大的功能，相当于给URL取了个全局变量名，不会将url匹配地址写死。
#注意
请求的URL被看做是一个普通的Python字符串，URLconf在其上查找并匹配。进行匹配时将不包括GET或POST请求方式的参数以及域名。

例如，在https://www.example.com/myapp/的请求中，URLconf将查找myapp/。

在https://www.example.com/myapp/?page=3的请求中，URLconf也将查找myapp/。
```

<!----more---->

**Django版本2.x**

```python
from django.urls import path

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<int:year>/', views.year_archive),
]
#Django 2.0版本中的路由系统已经替换成下面的写法，但是django2.0是向下兼容1.x版本的语法的
```

## 正则表达式详解

### 无名分组匹配（基本配置）

```python
#位置传参

from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^articles/2003/$', views.special_case_2003), #思考：如果用户想看2004、2005、2006....等，你要写一堆的url吗，是不是在articles后面写一个正则表达式/d{4}/就行啦，网址里面输入127.0.0.1:8000/articles/1999/试一下看看
    url(r'^articles/([0-9]{4})/$', views.year_archive), 
    url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive), #思考，如果你想拿到用户输入的什么年份，并通过这个年份去数据库里面匹配对应年份的文章，你怎么办？怎么获取用户输入的年份啊，分组/(\d{4})/，一个小括号搞定
    url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),
]
```

视图函数view.py

```python
#第一个参数必须是request，后面跟的三个参数是对应着上面分组正则匹配的每个参数的
url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),

def article_detail(request,year,month,day): 
    return HttpResponse(year+month+day)
#解释：从url匹配到的articles后面由3个参数，他们没一个都可以独立出来
```

**注意**

- urlpatterns中的元素按照书写顺序从上往下逐一匹配正则表达式，一旦匹配成功则不再继续，就直接break。

- 若要从URL中捕获一个值，只需要在它周围放置一对圆括号（分组匹配）。

- 不需要添加一个前导的反斜杠（也就是写在正则最前面的那个/），因为每个URL 都有。例如，应该是^articles 而不是 ^/articles。

- 每个正则表达式前面的'r' 是可选的但是建议加上，表示这是个路径。

- ^articles&  以什么结尾，以什么开头，严格限制路径。

- ```python
  #是否开启URL访问地址后面不为/跳转至带有/的路径的配置项
  APPEND_SLASH=True
  ```

### 分组命名匹配

解释：在Python的正则表达式中，分组命名正则表达式组的语法是**(?P<name>pattern)**，其中name是组的名称，pattern是要匹配的模式。

#### 基本配置（有名分组）

**重写上面的URL**

```python
#关键字传参

from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^articles/2003/$', views.special_case_2003), #注意正则匹配出来的内容是字符串，即便是你在url里面写的是2003数字，匹配出来之后也是字符串
　　 url(r'^articles/(\d{4})/$', views.year_archive),#year_archive(request,n),小括号为分组，有分组，那么这个分组得到的用户输入的内容，就会作为对应函数的位置参数传进去,别忘了形参要写两个了，明白了吗？
    url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),#某年的，(?P<year>[0-9]{4})这是命名参数（正则命名匹配还记得吗？），那么函数year_archive(request,year)，形参名称必须是year这个名字。而且注意如果你这个正则后面没有写$符号，即便是输入了月份路径，也会被它拦截下拉，因为它的正则也能匹配上
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),#某年某月的
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<day>[0-9]{2})/$', views.article_detail), #某年某月某日的
]
```

视图函数view.py

```python
#针对url /articles/2017/12/相当于按以下方式调用视图函数：
views.month_archive(request, year="2017", month="12")，#year和month的位置可以换，没所谓了，因为是按照名字来取数据的，还记得关键字参数吗？
```

#### 捕获的参数永远都是字符串

每个在URLconf中捕获的参数都作为一个普通的Python字符串传递给视图，无论正则表达式使用的是什么匹配方式。例如，下面这行URLconf 中：

```python
url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
```

 传递到视图函数**views.year_archive()** 中的`year` 参数永远是一个字符串类型,而非是一个数字类型。

#### 视图函数中指定默认值

```python
# urls.py中
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^blog/$', views.page),
    url(r'^blog/page(?P<num>[0-9]+)/$', views.page),
]

# views.py中，可以为num指定默认值
def page(request, num="1"):
    pass
```

在上面的例子中，两个URL模式指向相同的view - views.page - 但是第一个模式并没有从URL中捕获任何东西。

如果第一个模式匹配上了，page()函数将使用其默认参数num=“1”,如果第二个模式匹配，page()将使用正则表达式捕获到的num值。

#### 路由转发

include的背后是一种即插即用的思想。项目的根路由不会关心具体的app的路由策略，只管往指定的二级路由转发，实现了解耦。app所属的二级路由可以根据自己的需求随意编写，不会和其他的app路由发生冲突。

建议：除了admin路由外，尽量给每个app设计自己独立的二级路由。

```python
from django.conf.urls import include,url

urlpatterns = [
   url(r'^admin/', admin.site.urls),
   url(r'^blog/', include('blog.urls')),  # 可以包含其他的URLconfs文件
   url(r'^app01/',include('app01.urls')),  #别忘了要去app01这个应用下创建一个urls.py的文件，现在的意思是凡是以app01开头的路径请求，都让它去找app01下的urls文件中去找对应的视图函数，还要注意一点，此时这个文件里面的那个app01路径不能用$结尾，因为如果写了$，就没办法比配上app01/后面的路径了

]
```

app01的urls.py的内容：（其实就是将全局的urls.py里面的内容copy一下，放到你在app01文件夹下创建的那个urls.py文件中，把不是这个app01应用的url给删掉就行了）

```python
from django.conf.urls import url
#from django.contrib import admin
from app01 import views

urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^articles/2003/', views.special_case_2003,{'foo':'xxxxx'}), #{'foo':'xxxxx'}那么你的视图函数里面必须有个形参叫做foo来接收这种传参
    url(r'^articles/(\d{4})/(\d{2})/', views.year_archive),

]
```

### 命名URL和URL的反向解析和命名空间

人们强烈希望不要硬编码（其实就是在标签里面写死了路径，凡是写死了的代码就是硬编码）这些URL（费力、不可扩展且容易产生错误）或者设计一种与URLconf 毫不相关的专门的URL 生成机制，因为这样容易导致一定程度上产生过期的URL。

#### 反向解析

本质在view里面就是返回的一个路径，多用在redirect重定向，而且，这个参数还可以省略（哈哈哈.......）。

```python
#urls.py文件
	url(r'^index/',views.index,name="index")
#view.py文件
	from django.urls import reverse   #导入这个反向解析的模块
	def index(request):
		if reverse("index"):   #返回的肯定是True，其实它返回的就是一个路径，这个路径近视它里面写的index
			return xxx   #如果请求的url存在就返回数据
#html文件中
	格式：{% url "别名" %}
	实例：<a href="{{% url "index" %}}"></a>
```

#### 一个简单的栗子

**urls.py文件**

```python
url(r'^home', views.home, name='home'),  # 给我的url匹配模式起名（别名）为 home，别名不需要改，路径你就可以随便改了，别的地方使用这个路径，就用别名来搞
url(r'^index/(\d*)', views.index, name='index'),  # 给我的url匹配模式起名为index
```

**html文件**

```js
{% url 'home' %}  #模板渲染的时候，被django解析成了这个名字对应的那个url，这个过程叫做反向解析

<a href="{% url 'news-year-archive' 2012 %}">2012 Archive</a>

<ul>
{% for yearvar in year_list %}
<li><a href="{% url 'news-year-archive' yearvar %}">{{ yearvar }} Archive</a></li>
{% endfor %}
</ul>
```

**view.py文件**

```python
from django.urls import reverse
from django.shortcuts import redirect

def redirect_to_year(request):
    # ...
    year = 2006
    # ...
    return redirect(reverse('news-year-archive', args=(year,))) #或者直接return redirect('news-year-archive',year) redirect内部会自动调用reverse来进行反向解析
```

如果出于某种原因决定按年归档文章发布的URL应该调整一下，那么你将只需要修改URLconf 中的内容。但是，在某些场景中，一个视图是通用的，所以在URL 和视图之间存在多对一的关系。对于这些情况，当反查URL 时，只有视图的名字还不够。

注意：

为了完成上面例子中的URL 反查，你将需要使用命名的URL 模式。URL 的名称使用的字符串可以包含任何你喜欢的字符。不只限制在合法的Python 名称。

当命名你的URL 模式时，请确保使用的名称不会与其它应用中名称冲突。如果你的URL 模式叫做`comment`，而另外一个应用中也有一个同样的名称，当你在模板中使用这个名称的时候不能保证将插入哪个URL。

#### 命名空间模式

#### 路由分发include

**步骤流程**

```python
1.在每个app下创建urls.py文件，写上自己的app路径(就是总路由和分路由)
2.在项目目录在的urls.py文件中做一下路径分发：
	from django.conf.urls import url,include
	from django.contrib inport admin
	
	urlpatterns = [
		#url(r'^admin/',admin.site.urls),
		url(r'^app01/',include('app01.urls')),
		url(r'^app02/',include('app02.urls')),
	]
```

**实例**

总路由：urls.py

```python
from django.conf.urls import url,include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^app01/', include('app01.urls')),
    url(r'^app02/', include('app02.urls')),

]
```

分路由：app01下urls.py

```python
from django.conf.urls import url
from django.contrib import admin
from app01 import views
urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^index/', views.index,name='index'),
]
```

分路由：app02下urls.py

```python
from django.conf.urls import url
from django.contrib import admin
from app02 import views
urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^index/', views.index,name='index'),
]
```

app01下的views.py的写法

```python
from django.shortcuts import render,HttpResponse,redirect
from django.urls import reverse
# Create your views here.

def index(request):
    print(reverse('index'))
    return HttpResponse('ok')
```

app02下的views.py的写法

```python
from django.shortcuts import render,HttpResponse,redirect
from django.urls import reverse
# Create your views here.
def index(request):

    print(reverse('index'))
    return HttpResponse('ok2')
```

**你会发现，不管你是访问app01下的index还是app02下的index，打印的结果都是/app02/index/，也就是打印的是最后一个index别名对应的url路径。所以别名冲突了的话就需要我们的命名空间来保证别名对应的url的唯一性了。**

那么怎么去解决上面的问题呢，就是下面讲的命名空间

#### 命名空间namespace

**步骤流程**

第一种写法：

```python
from django.conf.urls import url,include
from django.contrib import admin
urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^app01/', include('app01.urls',namespace='app01')),#app01/home/
    url(r'^app02/', include('app02.urls',namespace='app02')),
	
]


使用:
	视图中使用:reverse('命名空间名称:别名') -- reverse('app01:home') 
        
	html文件中使用：hmtl:{% url '命名空间名称:别名' %}  -- {% url 'app01:home' %}
```

第二种写法：就是在每个app下的urls.py文件中指定app名称，同样也是命名空间.

总路由：urls.py

```python
from django.conf.urls import url,include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    # url(r'^app01/', include('app01.urls',namespace='app01')),
    url(r'^app01/', include('app01.urls')),
    # url(r'^app02/', include('app02.urls',namespace='app02')),
    url(r'^app02/', include('app02.urls')),

]
```

分路由:app01下urls.py

```python
from django.conf.urls import url
from django.contrib import admin
from app01 import views
app_name = 'app01'    #这个就是我们指定的命名空间
urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^index/', views.index,name='index'),
]
```

分路由:app02下urls.py

```python
from django.conf.urls import url
from django.contrib import admin
from app01 import views
app_name = 'app02'
urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^index/', views.index,name='index'),
]
```

第三种写法：

总路由：urls.py

```python
from django.conf.urls import url,include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    # url(r'^app01/', include('app01.urls',namespace='app01')),
    url(r'^app01/', include(('app01.urls','app01'),namespace='app01')),
    # url(r'^app02/', include('app02.urls',namespace='app02')),
    url(r'^app02/', include(('app02.urls','app02'),namespace='app02')),
```

**深入学习**：https://docs.djangoproject.com/en/1.11/topics/http/urls/

### 自定义错误页面

当Django找不到与请求匹配的URL时，或者当抛出一个异常时，将调用一个错误处理视图。Django默认的自带的错误视图包括400、403、404和500，分别表示请求错误、拒绝服务、页面不存在和服务器错误。它们分别位于：

- handler400 —— django.conf.urls.handler400。
- handler403 —— django.conf.urls.handler403。
- handler404 —— django.conf.urls.handler404。
- handler500 —— django.conf.urls.handler500。

但是他们非常的丑陋，我们怎么自定义呢？

**首先**：我们在根URLconf中额外增加下面的内容

```python
from django.contrib import admin
from django.urls import path,re_path
from app import views

urlpatterns = [
    re_path('admin/', admin.site.urls),
]

# 增加的条目
handler400 = views.bad_request
handler403 = views.permission_denied
handler404 = views.page_not_found
handler500 = views.error
```

**然后**：在我们的视图文件中增加和上面对应的4个视图函数

```python
def bad_request(request):
    return render(request, '400.html')

def permission_denied(request):
    return render(request, '403.html')

def page_not_found(request):
    return render(request, '404.html')

def error(request):
    return render(request, '500.html')
```

**最后**：我们在相应的.html文件中写我们自定义的内容就可以了

### 总结

**针对URL在模板文件中的命中为例，具体的url配置和视图的配置具体参考上面的**

无名分组和有名分组(硬编码)

```html
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

命名URL和反向解析(软编码)

```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

命名空间(解决多app的url命中问题)

```python
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

