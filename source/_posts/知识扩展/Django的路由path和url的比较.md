---
title: Django的路由path和url的比较
id: 2
date: 2019-10-6 20:30:00
tags: 知识扩展
comment: true
---

### Django的路由path和url的比较

我们知道，Django1.x版本和Django2.0版本的最大的区别就是路由系统的不同。在Django2.0之前一直使用的是正则匹配的方式；之后使用的是path转换器方式匹配。当然，为了兼容性，re_path支持正则方式匹配。

#### 实例

views.py

```python
def articles(seq,year):
    year = int(year)
    return HttpResponse(year)
```

路由使用正则匹配的方式：

```python
url(r"^articles/(?P<year>[0-9]{4})/$",views.articles)  #采用有名分组的方式进行路由匹配
```

路由使用path的方式：

```python
path(r"articles/<int:teay>/",views.articles)
```

<!----more---->

#### 语法

```python
path(route,view,name,**kwargs)
```

path最大的差别就是这个route参数：

- 要从URL捕获值，使用尖括号
- 捕获的值可以选择包括转换器类型。例如：<int:name>捕获整数参数。如果未包含转换器/，则匹配字符之外的任何字符串
- 没有必要添加前导斜杠，因为每一个URL都有

默认的情况下，以下路径转换器可用：

- str-匹配除路径分隔符之外的任何非空字符串。
- int-匹配0或者任何整数。
- slug-匹配由ASCII字母或者数字组成的任何slug字符串，以及连接符和下划线字符。例如：buliding-your-lst-django-site。
- uuid-匹配格式化的UUID。要防止多个URL映射到同一页面，必须包含短划线并且字母必须是小写。例如：075194d3-6885-417e-a8a8-6c931e272f00。
- path-匹配任何非空字符串，包含路径分隔符'/'。这使我们可以匹配完整的URL路径，而不仅仅是URL路径中的一部分str。

#### path转换器实例

```python
urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<int:year>/', views.year_archive),
    path('articles/<int:year>/<int:month>/', views.month_archive),
    path('articles/<int:year>/<int:month>/<slug>/', views.article_detail),
]
```

上面将匹配到：

| 请求URL                                   | 匹配项 | 视图函数调用形式                                             |
| ----------------------------------------- | ------ | ------------------------------------------------------------ |
| /articles/2005/03/                        | 第3个  | views.month_archive(request, year=2005, month=3)             |
| /articles/2003/                           | 第1个  | views.special_case_2003(request)                             |
| /articles/2003                            | 无     | -                                                            |
| /articles/2003/03/building-a-django-site/ | 第4个  | views.article_detail(request, year=2003, month=3, slug=”building-a-django-site”) |

总结：

路由（url）到视图（View）的流程可以概括为四个步骤：

- url匹配
- 正则捕获
- 视图调用

Django2.0之后的path相比较之前的多了一步变量类型转化：

- url匹配
- 正则捕获
- 变量类型转化
- 视图调用

新的path语法可以解决一下的几个场景:

- 类型自动转化
- 公用正则表达式