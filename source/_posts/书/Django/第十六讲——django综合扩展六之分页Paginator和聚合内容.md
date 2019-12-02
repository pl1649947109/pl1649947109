---
title: 第十六讲——django综合扩展六之分页Paginator和聚合内容
id: 16
date: 2019-10-18 20:30:00
tags: Django
comment: true
---

## 分页Paginator

分页功能是几乎所有的网站上都需要提供的功能，当你要展示的条目比较多时，必须进行分页，不但能减小数据库读取数据压力，也有利于用户浏览。

Django又很贴心的为我们提供了一个Paginator分页工具，但是不幸的是，这个工具功能差了点，不好添加CSS样式，所以前端的展示效果比较丑。如果你能力够，自己编写一个分页器，然后提交给Django官方吧，争取替代掉这个当前的分页器，我看好你哦！

但不管怎么样，当前的Paginator分页器，还是值得学一下用一下的。

<!----more---->

### 实例展示

```python
>> from django.core.paginator import Paginator
>>> objects = ['john', 'paul', 'george', 'ringo']
>>> p = Paginator(objects, 2)  # 对objects进行分页，虽然objects只是个字符串列表，但没关系，一样用。每页显示2条。
>>> p.count   # 对象个数
4
>>> p.num_pages  # 总共几页
2
>>> type(p.page_range)  # `<type 'rangeiterator'>` in Python 2.<class 'range_iterator'>
>>> p.page_range  # 分页范围
range(1, 3)
>>> page1 = p.page(1) # 获取第一页
>>> page1
<Page 1 of 2>
>>> page1.object_list # 获取第一页的对象
['john', 'paul']
>>> page2 = p.page(2)
>>> page2.object_list
['george', 'ringo']
>>> page2.has_next()  # 判断是否有下一页
False
>>> page2.has_previous()# 判断是否有上一页
True
>>> page2.has_other_pages() # 判断是否有其它页
True
>>> page2.next_page_number() # 获取下一页的页码
Traceback (most recent call last):
...
EmptyPage: That page contains no results
>>> page2.previous_page_number() # 获取上一页的页码
1
>>> page2.start_index() # 从1开始计数的当前页的第一个对象
3
>>> page2.end_index() # 从1开始计数的当前页最后1个对象
4
>>> p.page(0)  # 访问不存在的页面
Traceback (most recent call last):
...
EmptyPage: That page number is less than 1
>>> p.page(3) # 访问不存在的页面
Traceback (most recent call last):
...
EmptyPage: That page contains no results
```

### 在视图中使用Paginator

下面的例子假设你拥有一个已经导入的Contacts模型。

```python
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from django.shortcuts import render

def listing(request):
    contact_list = Contacts.objects.all()
    paginator = Paginator(contact_list, 25) # 每页显示25条

    page = request.GET.get('page')
    try:
        contacts = paginator.page(page)
    except PageNotAnInteger:
        # 如果请求的页数不是整数，返回第一页。
        contacts = paginator.page(1)
    except EmptyPage:
        # 如果请求的页数不在合法的页数范围内，返回结果的最后一页。
        contacts = paginator.page(paginator.num_pages)
    return render(request, 'list.html', {'contacts': contacts})
```

在`list.html`模板中，使用下面的范例来展示每个要显示的contact，以及最后的一个分页栏：

```html
{# 分页标签的HTML代码 #}
<div class="pagination">
    <span class="step-links">
        {% if contacts.has_previous %}
            <a href="?page={{ contacts.previous_page_number }}">previous</a>   #上一页
        {% endif %}

        <span class="current">
            Page {{ contacts.number }} of {{ contacts.paginator.num_pages }}.
        </span>

        {% if contacts.has_next %}
            <a href="?page={{ contacts.next_page_number }}">next</a>    //下一页
        {% endif %}
    </span>
</div>
```

### Paginator对象

Paginator类拥有以下方法和属性：

**方法：**

Paginator.page(number)[source]

返回指定页面的对象列表，比如第7页的所有内容，下标以1开始。如果提供的页码不存在，抛出InvalidPage异常。

**属性**：

- Paginator.count：所有页面的对象总数。
- Paginator.num_pages：页面总数。
- Paginator.page_range：基于1的页数范围迭代器。

### 处理异常

在实际使用中，可能恶意也可能不小心，用户请求的页面，可能千奇百怪。正常我们希望是个合法的1，2，3之类，但请求的可能是‘apple’，‘1000000’，‘#’，这就有可能导致异常，需要特别处理。Django为我们内置了下面几个，Paginator相关异常。

- exception InvalidPage[source]：异常的基类，当paginator传入一个无效的页码时抛出。
- exception PageNotAnInteger[source]：当向page()提供一个不是整数的值时抛出。
- exception EmptyPage[source]：当向page()提供一个有效值，但是那个页面上没有任何对象时抛出。

后面两个异常都是InvalidPage的子类，所以你可以通过简单的except InvalidPage来处理它们。

### Page对象

Paginator.page()将返回一个Page对象，我们主要的操作都是基于Page对象的，它具有下面的方法和属性：

**方法**：

- Page.has_next()[source]：如果有下一页，则返回True。
- Page.has_previous()[source]：如果有上一页，返回 True。
- Page.has_other_pages()[source]：如果有上一页或下一页，返回True。
- Page.next_page_number()[source]：返回下一页的页码。如果下一页不存在，抛出InvalidPage异常。
- Page.previous_page_number()[source]：返回上一页的页码。如果上一页不存在，抛出InvalidPage异常。
- `Page.start_index()[source]`：返回当前页上的第一个对象，相对于分页列表的所有对象的序号，从1开始计数。 比如，将五个对象的列表分为每页两个对象，第二页的`start_index()`会返回3。
- `Page.end_index()[source]`:返回当前页上的最后一个对象，相对于分页列表的所有对象的序号，从1开始。 比如，将五个对象的列表分为每页两个对象，第二页的`end_index()`会返回4。

**属性**:

- Page.object_list:当前页上所有对象的列表。
- Page.number:当前页的序号，从1开始计数。
- Page.paginator：当前Page对象所属的Paginator对象。

## 聚合内容 RSS/Atom

Django提供了一个高层次的聚合内容框架，让我们创建RSS/Atom变得简单，你需要做的只是编写一个简单的Python类。

### 范例

要创建一个feed，只需要编写一个Feed类，然后设置一条指向Feed实例的URLconf就可以了，非常简单，下面是一个示例，演示了某站点的最近五条新闻记录：

```python
from django.contrib.syndication.views import Feed
from django.urls import reverse
from policebeat.models import NewsItem

class LatestEntriesFeed(Feed):
    title = "Police beat site news"
    link = "/sitenews/"
    description = "Updates on changes and additions to police beat central."

    def items(self):
        return NewsItem.objects.order_by('-pub_date')[:5]

    def item_title(self, item):
        return item.title

    def item_description(self, item):
        return item.description

    # item_link is only needed if NewsItem has no get_absolute_url method.
    def item_link(self, item):
        return reverse('news-item', args=[item.pk])
```

要设置链接这个feed的URL，只需要将这个Feed类的实例，作为参数，加入URLconf，如下所示：

```python
from django.conf.urls import url
from myproject.feeds import LatestEntriesFeed

urlpatterns = [
    # ...
    url(r'^latest/feed/$', LatestEntriesFeed()),
    # ...
]
```

**注意:**

- 新建的Feed类继承于django.contrib.syndication.views.Feed。
- title、link和description属性分别对应标准RSS的`<title>`、`<link>`和`<description>`元素。
- items()方法简单地返回此Feed需要包含的对象，列表形式。
- 如果你要创建一个Atom feed，而不是RSS feed，使用subtitle属性替代description。

还有一件事要做。在一个 RSS feed中，每一个`<item>`都有一个`<title>`, `<link>` 和`<description>`， 我们需要告诉框架往这些对象里放入哪些数据。

- 对于`<title>`和`<description>`，Django将尝试调用Feed类中的`item_title()`和`item_description()`方法。 这两个方法都会被传入一个参数：item，也就是对象自己。
- 对于`<link>`，Django首先会尝试调用`item_link()`方法，如果该方法不存在，则使用对象的ORM模型中定义的`get_absolute_url()`方法。

### 指定feed类型

默认情况下，使用RSS 2.0类型，如果要指定类型，在Feed类中添加feed_type属性，如下所示：

```python
from django.utils.feedgenerator import Atom1Feed

class MyFeed(Feed):
    feed_type = Atom1Feed
```

目前可用的类型有下面三种：

- django.utils.feedgenerator.Rss201rev2Feed (RSS 2.01. Default.)
- django.utils.feedgenerator.RssUserland091Feed (RSS 0.91.)
- django.utils.feedgenerator.Atom1Feed (Atom 1.0.)

### 同时发布Atom和RSS feeds

要同时发布这两者，很简单，为你的Feed类创建一个子类，并且将其feed_type设置为你需要的类型，最后添加一条URLconf就可以了，如下所示：

```python
from django.contrib.syndication.views import Feed
from policebeat.models import NewsItem
from django.utils.feedgenerator import Atom1Feed

class RssSiteNewsFeed(Feed):
    title = "Police beat site news"
    link = "/sitenews/"
    description = "Updates on changes and additions to police beat central."

    def items(self):
        return NewsItem.objects.order_by('-pub_date')[:5]

# 增加下面的子类
class AtomSiteNewsFeed(RssSiteNewsFeed):
    feed_type = Atom1Feed # 修改类型
    subtitle = RssSiteNewsFeed.description
```

增加路由：

```python
from django.conf.urls import url
from myproject.feeds import RssSiteNewsFeed, AtomSiteNewsFeed

urlpatterns = [
    # ...
    url(r'^sitenews/rss/$', RssSiteNewsFeed()),
    url(r'^sitenews/atom/$', AtomSiteNewsFeed()),
    # ...
]
```