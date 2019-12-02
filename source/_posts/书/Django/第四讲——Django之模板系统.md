---
title: 第四讲——Django之模板系统
id: 4
date: 2019-9-27 20:00:00
tags: Django
comment: true
---

内容大全：https://docs.djangoproject.com/en/1.11/ref/templates/builtins/#std:templatetag-for

### 什么是系统模板

每一个Web框架都需要一种很便利的方法用于动态生成HTML页面，最常见的做法就是使用模板。系统模板就是如何网HTML文件中填入动态内容的系统。Django自带一个称为DTL（Django Template Language ）的模板语言，以及另外一种流行的Jinja2语言(需要提前安装，pip install Jinja2）。Django为加载和渲染模板定义了一套标准的API，与具体的后台无关。加载指的是，根据给定的模版名称找到的模板然后预处理，通常会将它编译好放在内存中。渲染则表示，使用Context数据对模板插值并返回生成的字符串。

### 配置引擎

模板引擎通过settings中的`TEMPLATES`设置来配置。这是一个列表，与引擎一一对应，每个元素都是一个引擎配置字典。由startproject命令生成的`settings.py`会自定定义如下的值：

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
'django.template.context_processors.debug',
'django.template.context_processors.request',
'django.contrib.auth.context_processors.auth',
'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

#解释每个字段的含义
BACKEND:后端。内置的后端有：
    #原生的DTL 
    django.template.backends.django.DjangoTemplates  
    #流行的Jinja2
    django.template.backends.jinjia2.Jinja2
OPTIONS:包含了具体的后端设置
    由于绝大多数引擎都是从文件加载模板的，所有每种模板引擎都包含两项通用设置：
    DIRS:定义了一个目录列表，模板引擎按列表顺序搜索这些目录以查找模板源文件。
    APP_DIRS:告诉模板引擎是否应该进入每个已经安装的应用中查找模板。通查过请将该项保持为True
DTL引擎的OPTIONS配置项中接受一下参数：
    'autoescape'：一个布尔值，用于控制是否启用HTML自动转义功能。默认为True。
    'context_processors': 以"."为分隔符的Python调用路径的列表。默认是个空列表。
    'debug'：打开/关闭模板调试模式的布尔值。默认和setting中的DEBUG有相同的值。
    'loaders'：模板加载器类的虚拟Python路径列表。默认值取决于DIRS和APP_DIRS的值。
    'string_if_invalid'：非法变量时输出的字符串。默认为空字符串。
    'file_charset'：用于读取磁盘上的模板文件的字符集编码。默认为FILE_CHARSET的值。
    'libraries'：用于注册模板引擎。 这可以用于添加新的库或为现有库添加备用标签。
    'builtins'：以圆点分隔的Python路径的列表。
         
#注意：每种模板引擎后端都定义了一个惯用的名称作为应用内部存放模板的子目录名称。（例如Django为它自己的模板引擎指定的是 ‘templates’，为jinja2指定的名字是‘jinja2’）。尤其是，django允许你有多个模板引擎后端实例，且每个实例有不同的配置选项。 在这种情况下你必须为每个配置指定一个唯一的NAME .
```

### 模板

模板时纯文本文件，可以生成任何基于文本的文件格式，比如HTML，XML，CSV等。

**一个简单的小模板**

```js
{% extends "base_generic.html" %}    #继承母版

{% block title %}{{ section.title }}{% endblock %}  #填充自己页面的内容

{% block content %}
<h1>{{ section.title }}</h1>

{% for story in story_list %}
<h2>
  <a href="{{ story.get_absolute_url }}">
    {{ story.headline|upper }}
  </a>
</h2>
<p>{{ story.tease|truncatewords:"100" }}</p>
{% endfor %}
{% endblock %}
```

### 基本语法

关于模板渲染你只需要记两种特殊符号（语法）：

```js
{{  }}和 {% %}
变量相关的用{{}}，逻辑相关的用{%%}。
```

<!----more---->

### 变量

```js
在Django的模板语言中按此语法使用：:{{ 变量名 }}
#解释：当模板引擎遇到一个变量，它将从上下文context中获取这个变量的值，然后用值替换掉它本身。

万能的点".":功能
	字典查询
    属性或者方法查询
    数字索引查询
如果我们使用的变量不存在，模板系统就会插入string_if_invaild选项的值，默认设置为空字符串
```

```python
#views.py
def index(request):
    import datetime
    s = "hello"
    l = [111, 222, 333]  # 列表
    dic = {"name": "yuan", "age": 18}  # 字典
    date = datetime.date(1993, 5, 2)  # 日期对象

    class Person(object):
        def __init__(self, name):
            self.name = name
        def dream(self):
            return 'dreamer'
    person_yuan = Person("chao")  # 自定义类对象
    person_egon = Person("yantao")
    person_alex = Person("jinxin")

    person_list = [person_yuan, person_egon, person_alex]

    return render(request, "index.html", {"l": l, "dic": dic, "date": date, "person_list": person_list})
    # return render(request,'index.html',locals())
    #locals()获取函数内容所有的变量，然后通过render方法给了index.html文件进行模板渲染，如果你图省事，你可以用它，但是很多多余的变量也被传进去了，效率低
    
#html
<h4>{{s}}</h4>
<h4>列表:{{ l.0 }}</h4>
<h4>列表:{{ l.2 }}</h4>
<h4>字典:{{ dic.name }}</h4>
<h4>日期:{{ date.year }}</h4>

<!--取列表的第1个对象的name属性的值-->
<h4>类对象列表:{{ person_list.0.name }}</h4>
<!--取列表的第1个对象的dream方法的返回值，如果没有返回值，拿到的是none-->
<h4>类对象列表:{{ person_list.0.dream }}</h4>
注意：
    调用对象里面的方法的时候，不需要写括号来执行，并且只能执行不需要传参数的方法，如果你的这个方法需要传参数，那么模板语言不支持，不能帮你渲染
```

### 过滤器

在Django的模板语言中，通过使用 过滤器 来改变变量的显示。

**语法**：

```js
{{ value|filter_name:参数 }}
```

注意：

1.过滤器支持“链式”操作。即一个过滤器的输出作为另一个过滤器的输入。

```js
例如：
{{ text|escape|linebreaks }}    #就是一个常用的过滤器链，它首先转移文本内容，然后把文本行转成<p>标签。
```

2.过滤器可以接受参数

```js
例如：
{{ sss|truncatewords:30 }}  #这将显示sss的前30个词
```

3.过滤器参数包含空格的话，必须用引号包裹起来。比如使用逗号和空格去连接一个列表中的元素，如：

```js
{{ list|join:', ' }}
```

4.'|'左右没有空格没有空格没有空格

| 过滤器             | 说明                                                  |
| ------------------ | ----------------------------------------------------- |
| add                | 加法                                                  |
| addslashes         | 添加斜杠                                              |
| capfirst           | 首字母大写                                            |
| center             | 文本居中                                              |
| cut                | 切除字符                                              |
| date               | 日期格式化                                            |
| default            | 设置默认值                                            |
| default_if_none    | 为None设置默认值                                      |
| dictsort           | 字典排序                                              |
| dictsortreversed   | 字典反向排序                                          |
| divisibleby        | 整除判断                                              |
| escape             | 转义                                                  |
| escapejs           | 转义js代码                                            |
| filesizeformat     | 文件尺寸人性化显示                                    |
| first              | 第一个元素                                            |
| floatformat        | 浮点数格式化                                          |
| force_escape       | 强制立刻转义                                          |
| get_digit          | 获取数字                                              |
| iriencode          | 转换IRI                                               |
| join               | 字符列表链接                                          |
| last               | 最后一个                                              |
| length             | 长度                                                  |
| length_is          | 长度等于                                              |
| linebreaks         | 行转换                                                |
| linebreaksbr       | 行转换                                                |
| linenumbers        | 行号                                                  |
| ljust              | 左对齐                                                |
| lower              | 小写                                                  |
| make_list          | 分割成字符列表                                        |
| phone2numeric      | 电话号码                                              |
| pluralize          | 复数形式                                              |
| pprint             | 调试                                                  |
| random             | 随机获取                                              |
| rjust              | 右对齐                                                |
| safe               | 安全确认                                              |
| safeseq            | 列表安全确认                                          |
| slice              | 切片                                                  |
| slugify            | 转换成ASCII                                           |
| stringformat       | 字符串格式化                                          |
| striptags          | 去除HTML中的标签                                      |
| time               | 时间格式化                                            |
| timesince          | 从何时开始                                            |
| timeuntil          | 到何时多久                                            |
| title              | 所有单词首字母大写                                    |
| truncatechars      | 截断字符                                              |
| truncatechars_html | 截断字符                                              |
| truncatewords      | 截断单词                                              |
| truncatewords_html | 截断单词                                              |
| unordered_list     | 无序列表                                              |
| upper              | 大写                                                  |
| urlencode          | 转义url                                               |
| urlize             | url转成可点击的链接                                   |
| urlizetrunc        | urlize的截断方式                                      |
| wordcount          | 单词计数                                              |
| wordwrap           | 单词包裹                                              |
| yesno              | 将True，False和None，映射成字符串‘yes’，‘no’，‘maybe’ |

**过滤器**

#### default

如果一个变量是false或者为空，使用给定的默认值。 否则，使用变量的值

```js
{{ value|default:"nothing"}}  #如果value没有传值或者值为空的话就显示nothing
```

#### length

返回值的长度，作用于字符串和列表

```js
{{ value|length }} #返回value的长度，如 value=['a', 'b', 'c', 'd']的话，就显示4.
```

#### filesizeformat

将值格式化为一个 “人类可读的” 文件尺寸 （例如 `'13 KB'`, `'4.1 MB'`, `'102 bytes'`, 等等）。例如：

```js
{{ value|filesizeformat }}   #如果 value 是 123456789，输出将会是 117.7 MB。
```

#### slice

切片,如果 value="hello world",还有其他可切片的数据类型

```js
{{value|slice:"2:-1"}}
```

#### date

格式化,如果 value=datetime.datetime.now()

```
{{ value|date:"Y-m-d H:i:s"}}
```

关于时间日期的可用的参数(除了Y,m,d等等)还有很多，有兴趣的可以去查查看看。

#### safe

Django的模板中在进行模板渲染的时候会对HTML标签和JS等语法标签进行自动转义，原因显而易见，这样是为了安全，django担心这是用户添加的数据，比如**如果有人给你评论的时候写了一段js代码，这个评论一提交，js代码就执行啦，这样你是不是可以搞一些坏事儿了，写个弹窗的死循环，那浏览器还能用吗，是不是会一直弹窗啊，这叫做xss攻击**，所以浏览器不让你这么搞，给你转义了。但是有的时候我们可能不希望这些HTML元素被转义，比如我们做一个内容管理系统，后台添加的文章中是经过修饰的，这些修饰可能是通过一个类似于FCKeditor编辑加注了HTML修饰符的文本，如果自动转义的话显示的就是保护HTML标签的源文件。为了在Django中关闭HTML的自动转义有两种方式，如果是一个单独的变量我们可以通过过滤器“|safe”的方式告诉Django这段代码是安全的不必转义。

```
{{ value|safe}}
```

很多网站，都会对你提交的内容进行过滤，一些敏感词汇、特殊字符、标签、黄赌毒词汇等等，你一提交内容，人家就会检测你提交的内容，如果包含这些词汇，就不让你提交，其实这也是解决xss攻击的根本途径。

#### truncatechars

如果字符串字符多于指定的字符数量，那么会被截断。截断的字符串将以可翻译的省略号序列（“...”）结尾

参数：截断的字符数

```js
{{ value|truncatechars:9}}  #注意：最后那三个省略号也是9个字符里面的，也就是这个9截断出来的是6个字符+3个省略号，有人会说，怎么展开啊，配合前端的点击事件就行啦
```

#### truncatewords

在一定数量的字后截断字符串，是截多少个单词。

```js
例如：‘hello girl hi baby yue ma’,
{{ value|truncatewords:3}}  #上面例子得到的结果是 'hello girl h1...'
```

#### cut

移除value中所有的与给出的变量相同的字符串

```js
{{ value|cut:' ' }}    #如果value为'i love you'，那么将输出'iloveyou'.
```

#### join

使用字符串连接列表

```python
{{ list|join:', ' }}    #就像Python的str.join(list)
```

#### timesince

将日期格式设为自该日期起的时间（例如，“4天，6小时”）。

采用一个可选参数，它是一个包含用作比较点的日期的变量（不带参数，比较点为现在）。 例如，如果blog_date是表示2006年6月1日午夜的日期实例，并且comment_date是2006年6月1日08:00的日期实例，则以下将返回“8小时”：

```python
{{ blog_date|timesince:comment_date }}    #分钟是所使用的最小单位，对于相对于比较点的未来的任何日期，将返回“0分钟”。
```

详细讲解：http://www.liujiangblog.com/course/django/147

#### 注意：

- 过滤器支持“链式”操作，也就是一个过滤器的输出作为另一个过滤器的输入
- 过滤器参数包含空格的话，必须使用引号包裹起来
- |左右没有空格，切记

#### 支持的写法

```javascript
- {# 取l中的第一个参数 #}`

  `{{ l.0 }}`

- `{# 取字典中key的值 #}`

  `{{ d.name }}`

- `{# 取对象的name属性 #}`

  `{{ person_list.0.name }}`

- `{# .操作只能调用不带参数的方法 #}`

  `{{ person_list.0.dream }}`
```

### 标签Tags

格式：

```js
标签看起来像是这样的： {% tag %}
#标签比变量更加复杂：一些在输出中创建文本，一些通过循环或逻辑来控制流程，一些加载其后的变量将使用到的额外信息到模版中，一些标签需要开始和结束标签 
（例如{% tag %} ...标签 内容 ... {% endtag %}）
```

| Django内置标签 |           说明            |
| :------------: | :-----------------------: |
|   autoescape   |       自动转移开关        |
|     block      |          块引用           |
|    comment     |           注释            |
|   csrf_token   |         CSRF令牌          |
|     cycle      |       循环对象的值        |
|     debug      |         调式模式          |
|    extends     |         继承模板          |
|     filter     |         过滤功能          |
|    firstof     | 输出第一个不为false的参数 |
|      for       |         循环对象          |
|  for...empty   |     带empty说明的循环     |
|       if       |         条件判断          |
|    ifequal     |         如果等于          |
|   ifnotequal   |        如果不等于         |
|    include     |     导入子模板的内容      |
|      load      |     加载标签和过滤器      |
|      now       |         当前时间          |
|   resetcycle   |         重置循环          |
|   spaceless    |         去除空白          |
|      url       |       获取url字符串       |
|    verbatim    |       禁用模板引擎        |
|   widthradio   |         宽度比例          |
|      with      |       上下文管理器        |

#### cycle

每当这个标签被访问,返回它的下一个元素。第一次访问返回第一个元素,第二次访问返回第二个参数,以此类推. 一旦所有的变量都被访问过了，就会回到最开始的地方，重复下去。

```python
{% for o in some_list %}
    <tr class="{% cycle 'row1' 'row2'%}">
        ...
    </tr>
{% endfor %}
#第一次迭代产生的HTML引用了row1类，第二次则是row2类，第三次又是row1 类，如此类推。
```

#### filter

通过一个或多个过滤器对内容过滤。需要结束标签endfilter。

```python
{% filter force_escape|lower %}
    This text will be HTML-escaped, and will appear in all lowercase.
{% endfilter %}
```

#### for标签

遍历每一个元素：  写个for，然后 tab键自动生成for循环的结构，循环很基础，就这么简单的用，没有什么break之类的，复杂一些的功能，你要通过js

```js
{% for person in person_list %}
    <p>{{ person.name }}</p>  <!--凡是变量都要用两个大括号括起来-->
{% endfor %}
```

**反向循环**：

```js
可以利用  {% for obj in list reversed %}  反向完成循环。
```

**遍历一个字典**：

```js
{% for key,val in dic.items %}
    <p>{{ key }}:{{ val }}</p>
{% endfor %}
```

注：**循环序号可以通过**

```js
｛｛forloop｝｝
```

显示，必须在循环内部用　　

```
forloop.counter            当前循环的索引值(从1开始)，forloop是循环器，通过点来使用功能
forloop.counter0           当前循环的索引值（从0开始）
forloop.revcounter         当前循环的倒序索引值（从1开始）
forloop.revcounter0        当前循环的倒序索引值（从0开始）
forloop.first              当前循环是不是第一次循环（布尔值）
forloop.last               当前循环是不是最后一次循环（布尔值）
forloop.parentloop         本层循环的外层循环的对象，再通过上面的几个属性来显示外层循环的计数等
```

#### for ... empty

`　　　　for` 标签带有一个可选的

```
{% empty %}
```

 从句，以便在给出的组是空的或者没有被找到时，可以有所操作。

```js
{% for person in person_list %}
    <p>{{ person.name }}</p>

{% empty %}
    <p>sorry,no person here</p>
{% endfor %}
```

#### if 标签

```
{% if %}
```

会对一个变量求值，如果它的值是“True”（存在、不为空、且不是boolean类型的false值），对应的内容块会输出。

```js
{% if num > 100 or num < 0 %}
    <p>无效</p>  <!--不满足条件，不会生成这个标签-->
{% elif num > 80 and num < 100 %}
    <p>优秀</p>
{% else %}  <!--也是在if标签结构里面的-->
    <p>凑活吧</p>
{% endif %}
```

当然也可以只有if和else

```js
{% if user_list|length > 5 %}  <!--结合过滤器来使用-->
  七座豪华SUV
{% else %}
    黄包车
{% endif %}
```

if语句支持 and 、or、==、>、<、!=、<=、>=、in、not in、is、is not判断，注意条件两边都有空格。

#### spaceless

删除HTML标签之间的空白，包括制表符和换行。

```python
{% spaceless %}
    <p>
        <a href="foo/">Foo</a>
    </p>
{% endspaceless %}
#这个示例将返回下面的HTML：
<p><a href="foo/">Foo</a></p>
```

#### with

使用一个简单地名字缓存一个复杂的变量，多用于给一个复杂的变量起别名，当你需要使用一个“昂贵的”方法（比如访问数据库）很多次的时候是非常有用的

例如：

注意等号左右不要加空格。

```js
{% with total=business.employees.count %}
    {{ total }} <!--只能在with语句体内用-->
{% endwith %}
```

或

```js
{% with business.employees.count as total %}
    {{ total }}
{% endwith %}
```

#### csrf_token

我们以post方式提交表单的时候，会报错，还记得我们在settings里面的中间件配置里面把一个csrf的防御机制给注销了啊，本身不应该注销的，而是应该学会怎么使用它，并且不让自己的操作被forbiden，通过这个东西就能搞定。

　　　　这个标签用于跨站请求伪造保护，

　　　　在页面的form表单里面（注意是在form表单里面）任何位置写上

```js
{% csrf_token %}  这个东西模板渲染的时候替换成了隐藏的，
```

```html
<input type="hidden" name="csrfmiddlewaretoken" value="8J4z1wiUEXt0gJSN59dLMnktrXFW0hv7m4d40Mtl37D7vJZfrxLir9L3jSTDjtG8">
```

这个标签的值是个随机字符串，提交的时候，这个东西也被提交了，首先这个东西是我们后端渲染的时候给页面加上的，那么当你通过我给你的form表单提交数据的时候，你带着这个内容我就认识你，不带着，我就禁止你，因为后台我们django也存着这个东西，和你这个值相同的一个值，可以做对应验证是不是我给你的token，存储这个值的东西我们后面再学，你先知道一下就行了，就像一个我们后台给这个用户的一个通行证，如果你用户没有按照我给你的这个正常的页面来post提交表单数据，或者说你没有先去请求我这个登陆页面，而是直接模拟请求来提交数据，那么我就能知道，你这个请求是非法的，反爬虫或者恶意攻击我的网站，以后将中间件的时候我们在细说这个东西，但是现在你要明白怎么回事，明白为什么django会加这一套防御。

爬虫发送post请求简单模拟：

```python
import requests

res = requests.post('http://127.0.0.1:8000/login/',data={
    'username':'pl',
    'password':'123'
})
print(res.text)　　
```

#### 注释

```python
{# ... #}  #单行注释
{% comment %}和{% endcomment %}之间的内容会被忽略，作为注释。    #多行注释
```

#### 注意事项

1. Django的模板语言不支持连续判断，即**不支持**以下写法：

```js
{% if a > b > c %}
...
{% endif %}
```

2. Django的模板语言中**属性的优先级大于方法**（了解）

```python
def xx(request):
    d = {"a": 1, "b": 2, "c": 3, "items": "100"}
    return render(request, "xx.html", {"data": d})
```

如上，我们在使用render方法渲染一个页面的时候，传的字典d有一个key是items并且还有默认的 d.items() 方法，此时在模板语言中:

```js
{{ data.items }}   默认会取d的items key的值。
```

详细解析：http://www.liujiangblog.com/course/django/147

### 其他标签和过滤器

Django附带了一些其他模板标签，必须在`INSTALLED_APPS`设置中显式启用，并在模板中启用标记。

**static**

static标签用于链接保存在`STATIC_ROOT`中的静态文件。例如：

```html
用法一：
{% load static %}
<img src="{% static "images/hi.jpg" %}" alt="Hi!" />
用法二：
{% load static %}
{% static "images/hi.jpg" as myphoto %}
<img src="{{ myphoto }}"></img>
用法三：
{% load static %}
<link rel="stylesheet" href="{% static user_stylesheet %}" type="text/css" media="screen" />
```

### 人类可读性

```
一些Django的‘奇技淫巧’就存在于这些不起眼的地方。

为了提高模板系统对人类的友好性，Django在`django.contrib.humanize`中提供了一系列的模板过滤器，有助于为数据展示添加“人文关怀”。

需要把`django.contrib.humanize`添加到`INSTALLED_APPS`设置中来激活这些过滤器。然后在模板中使用`{% load humanize %}`标签，就可以使用下面的过滤器了。
```

**apnumber**

```
对于数字1~9，返回英文单词，否则返回数字本身。 这遵循了出版图书的格式。

例如：

1 会变成one。
2 会变成 two。
10 会变成 10。
可以传递整数，或者整数的字符串形式。
```

**intcomma**

```
将整数或浮点数（或两者的字符串表示形式）转换为每隔三位数字包含逗号的字符串。这在财务报表中很有用。

例如：

4500 会变成 4,500。
4500.2变为4,500.2。
45000 会变成 45,000
450000 会变成 450,000。
4500000 会变成 4,500,000。
如果启动了Format localization，还将遵循用户本地国家标准。例如，在德语（'de'）中：

45000 会变成 '45.000'。
450000 会变成 '450.000'。
```

**intword**

```
将大整数（或整数的字符串表示形式）转换为友好的文本表示形式。适用于超过一百万的数字。

例如：

1000000 会变成 1.0 million。
1200000 会变成 1.2 million。
1200000000 会变成 1.2 billion。
支持高达10的100次方 (Googol) 的整数。

如果启动了Format localization，还将遵循用户本地国家标准。例如，在德语（'de'）中：

1000000 会变成 '1,0 Million'。
1200000 会变成 '1,2 Million'。
1200000000 会变成 '1,2 Milliarden'。
```

**naturalday**

```
对于当天或者一天之内的日期，返回“today”,“tomorrow”或者“yesterday”的表示形式，视情况而定。否则，使用传进来的格式字符串进行日期格式化。

例如（“今天”是2007年2月17日）：

16 Feb 2007 会变成 yesterday。
17 Feb 2007 会变成 today。
18 Feb 2007 会变成 tomorrow。
其它的日期，还是按照传统的方法展示。
```

**naturaltime**

```
对于日期时间的值，返回一个字符串来表示多少秒、分钟或者小时之前。如果超过一天之前，则回退为使用timesince格式。如果是未来的日期时间，返回值会自动使用合适的文字表述。

例如（“现在”是2007年2月17日16时30分0秒）：

17 Feb 2007 16:30:00 会变成 now。
17 Feb 2007 16:29:31 会变成 29 seconds ago。
17 Feb 2007 16:29:00 会变成 a minute ago。
17 Feb 2007 16:25:35 会变成 4 minutes ago。
17 Feb 2007 15:30:29 会变成 59 minutes ago。
17 Feb 2007 15:30:01 会变成 59 minutes ago。
17 Feb 2007 15:30:00 会变成 an hour ago。
17 Feb 2007 13:31:29 会变成 2 hours ago。
16 Feb 2007 13:31:29 会变成 1 day, 2 hours ago。
16 Feb 2007 13:30:01 会变成 1 day, 2 hours ago。
16 Feb 2007 13:30:00 会变成 1 day, 3 hours ago。
17 Feb 2007 16:30:30 会变成 30 seconds from now。
17 Feb 2007 16:30:29 会变成 29 seconds from now。
17 Feb 2007 16:31:00 会变成 a minute from now。
17 Feb 2007 16:34:35 会变成 4 minutes from now。
17 Feb 2007 17:30:29 会变成 an hour from now。
17 Feb 2007 18:31:29 会变成 2 hours from now。
18 Feb 2007 16:31:29 会变成 1 day from now。
26 Feb 2007 18:31:29 会变成 1 week, 2 days from now。
```

**ordinal**

```
将一个整数转化为它的序数词字符串。
例如：
1 会变成 1st。
2 会变成 2nd。
3 会变成 3rd。
```

### 方法调用

大多数对象上的方法调用同样可用在模板中。这就意味着模板能够访问到的不仅仅是对象的属性和视图传入的变量，还可以执行对象的方法。

```python
#all()
{% for comment in task.comment_set.all %}
    {{ comment }}
{% endfor %}
#count()
{{ task.comment_set.all.count }}
#或者视图中自定义的方法
models.py
class Task(models.Model):
    def foo(self):
        return "bar"
.html
{{ task.foo }}
```

注意：**由于Django有意限制模板语言中的处理逻辑，不能够在模板中传递参数来调用方法**。数据应该在视图中处理，然后传递给模板用于展示。所以说每个部分有每个部分的功能，我们不能意淫。

### 模板继承

Django模板引擎中最强大也是最复杂的部分就是模板继承了。模板继承可以让我们创建一个基本的“骨架”模板，它包含您站点中的全部元素，并且可以定义能够被子模板覆盖的blocks。

**步骤流程**：

```html
1.创建一个base.html页面（作为额母板，其他的页面都来继承它使用）

2.在母板中定义block块（可以定义多个，整个页面任意的位置）
	{% block content %}  <!-- 预留的钩子,共其他需要继承它的html,自定义自己的内容 -->

	{% endblock %}

3.其他的页面来继承母板的写法
	{% extends 'base.html' %}  <!--必须放在页面开头-->

4.页面中写和母板中名字相同的block块，从而显示自定义的内容
	{% block content %}  <!-- 预留的钩子,共其他需要继承它的html,自定义自己的内容 -->
        {{ block.super }}  <!--这是显示继承的母版中的content这个快中的内容-->
        这是xx1
    {% endblock %}

实例：
<!--母板文件-->
<!DOCTYPE html>
<html lang="en">
<head>
    <link rel="stylesheet" href="style.css" />
    <title>{% block title %}My amazing site{%/span> endblock %}</title>     <!--预留的钩子1-->
</head>

<body>
    <div id="sidebar">
        {% block sidebar %}
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/blog/">Blog</a></li>
        </ul>
        {% endblock %}    <!--预留的钩子2-->
    </div>

    <div id="content">
        {% block content %}{% endblock %}   <!--预留的钩子3-->
    </div>
</body>
</html>

<!--子母板继承-->
{% extends "base.html" %}   <!--字模板继承母模板之前先导入-->
 
{% block title %}My amazing blog{% endblock %}    <!---->
 
{% block content %}    <!--填钩子3-->
{% for entry in blog_entries %}
    <h2>{{ entry.title }}</h2>
    <p>{{ entry.body }}</p>
{% endfor %}
{% endblock %}
```

注意：extends标签是关键。它告诉模板引擎，这个模板继承另一个模板。当模板系统处理这个模板时，首先，它将定位母板的的位置。

不能在一个模板中定义多个相同名字的block标签。

### 组件

可以将常用的页面内容如导航条，页尾信息等组件保存在单独的文件中，然后在需要使用的地方，文件的任意位置按如下语法导入即可。

**语法**

```js
{% include 'navbar.html' %}
```

**实例**

```html
<!--组件html-->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .c1{
            background-color: red;
            height: 40px;
        }
    </style>
</head>
<body>

<div class="c1">
    <div>
        <a href="">我是组件</a>
        <a href="">我也是组件</a>
    </div>
</div>

</body>
</html>

<!--被嵌入的html文件-->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
{% include 'nav.html' %}    <!--现在我们上面的文件就被嵌入在了该界面的开头-->
<h1>我才是这个界面的内容呀</h1>
</body>
</html>
```

**组件和插件的区别**

- 组件提供某一完整功能的模块，如编辑器组件、qq空间提供的关注组件
- 插件更加倾向于封闭某一功能方法的函数
- 这两者的区别在于JavaScript里面区别很小的，组件这个名词用的不多，**一般统称插件**

### 自定义标签和过滤器

**步骤流程**

```js
1.在app应用下创建一个叫做templatetags的文件夹(只能使用这个名字),在里面创建一个py文件。例如：xx.py

2.在xx.py文件中引用django提供的template类：
	form django import template
	regisiter = template.Library()  #这个register的变量名称也不能修改
	
	#自定义过滤器：
	@register.fiflter  
	def xx(v1,v2):    #函数里面的参数最多只能有3个
		return pl
	#使用过滤器
	{% load xx %}
	{{ name|xx:"penglin" }}
	
	#自定义标签
	@register.simple_tag
	def pltag(n1,n2):   #函数里面的参数没有个数的限制
		 '''
        :param n1:  变量的值 管道前面的
        :param n2:  传的参数 管道后面的,如果不需要传参,就不要添加这个参数
        :return:
        '''
        return n1+n2
     #使用标签
     {% pltag 1 1 %} #2
     
   	 #inclusion_tag  多用于返回html片段的标签
   	 @register.inclusion_tag("result.html")
   	 def res(n1):      #n1:["aa","bb","cc"]
   	 	return {"li":n1}
   	 #使用标签
   	 {%res a%}
```

**实例**

在app应用下创建的templatetags下创建一个.py文件：如my_tag.py

```python
from django import template
from django.utils.safestring import mark_safe
 
register = template.Library()   #register的名字是固定的,不可改变
 
 
@register.filter     #自定义过滤器
def filter_multi(v1,v2):
    return  v1 * v2

@register.simple_tag  #自定义标签，和自定义filter类似，只不过接收更灵活的参数，没有个数限制。
def simple_tag_multi(v1,v2):
    return  v1 * v2

@register.simple_tag
def my_input(id,arg):
    result = "<input type='text' id='%s' class='%s' />" %(id,arg,)
    return mark_safe(result)
```

现在，创建好了过滤器和标签，剩下的就是怎么去使用了：**在使用自定义simple_tag和filter的html文件中导入之前创建的 my_tags.py**

```js
{% load my_tags %}   #把我们自定义的py文件导入进html文件

num=12
{{ num|filter_multi:2 }} #24
 
{{ num|filter_multi:"[22,333,4444]" }}
 
{% simple_tag_multi 2 5 %} # 参数不限,但不能放在if for语句中
{% simple_tag_multi num 5 %}
```

**inclusion_tag**:多用于返回html代码片段

实例：

定义：templatetags/my_inclusion.py

```python
from django import template

register = template.Library()


@register.inclusion_tag('result.html')  #将result.html里面的内容用下面函数的返回值渲染，然后作为一个组件一样，加载到使用这个函数的html文件里面
def show_results(n): #参数可以传多个进来
    n = 1 if n < 1 else int(n)
    data = ["第{}项".format(i) for i in range(1, n+1)]
    return {"data": data}#这里可以穿多个值，和render的感觉是一样的{'data1':data1,'data2':data2....}
```

借助第三方html渲染：templates/snippets/result.html

```html
<ul>
  {% for choice in data %}
    <li>{{ choice }}</li>
  {% endfor %}
</ul>
```

使用：templates/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="x-ua-compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>inclusion_tag test</title>
</head>
<body>

{% load inclusion_tag_test %}

{% show_results 10 %}  
</body>
</html>
```

### 静态文件相关

**步骤流程**

```python
1.项目目录下创建一个文件夹，就是和templates目录同级，例如起一个名字为jingtaiwenjian，将所有的静态文件都放在这个文件夹里面。同样的，我们也可以在templates下面创建static文件夹。
'''
注意：
目录的别名也是一种安全机制，浏览器上通过调试台你能够看到的是别名的名字，这样别人就不能知道你静态文件夹的名字了，不然别人就能通过这个文件夹路径进行攻击，所以最好不要使用static这样的名字，不安全。
'''

2.settings.py里面进行下面相关的配置：
	#配置静态文件相关的
	STATIC_URL = "/static/"  #别名：这个是创建项目的时候系统自动为我们创建的名称，这个名称是可以更换的，就是我们取得别名，我们可以利用这个别名去找到指定的静态文件。
	#配置自创建的文件相关的
	STATICFILES_DIRS = [
		os.path.join(BASE_DIR,"jingtaiwenjian")
	]
	或者
	STATICFILES_DIRS = [
		os.path.join(BASE_DIR,"templates/static")
	]
	
3.网页中引入静态文件（2种方式）
方式一：利用别名去引入
<link rel="stylesheet" href="/static/xxx.css">  #注意我们这里导入的文件名字是和settings下的STATIC_URL相关连的，所以，它里面取得名字是什么我们在这里就怎么去导入。

方式二：下面的是load组件的形式去引入（最好使用这个方式，别问为什么）
```

![](http://9017499461.linshutu.top/dj%E6%A8%A1%E6%9D%BF%E7%B3%BB%E7%BB%9F%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6%E7%9B%B8%E5%85%B3.png)

```js
#{% static %}
{% load static %}
<img src="{% static "images/hi.jpg" %}" alt="Hi!" />
    
#引入js文件时使用
{% load static %}
<script src="{% static "mytest.js" %}"></script>
 
#某个文件多处被用到可以存为一个变量
{% load static %}
{% static "images/hi.jpg" as myphoto %}
<img src="{{ myphoto }}"></img>
```

```js
#{% get_static_prefix %}
{% load static %}
<img src="{% get_static_prefix %}images/hi.jpg" alt="Hi!" />

{% load static %}
{% get_static_prefix as STATIC_PREFIX %}

<img src="{{ STATIC_PREFIX }}images/hi.jpg" alt="Hi!" />
<img src="{{ STATIC_PREFIX }}images/hi2.jpg" alt="Hello!" />
```

注意：

```html
<form action="/login/"></form>
<img src="/static/1.jpg" alt="">
等标签需要写路径的地方，如果写的是相对路径，那么前置的/这个斜杠必须写上，不然这个请求会拼接当前网页的路径来发送请求，就不能匹配我们的后端路径了
还要注意一点就是，使用绝对路径的形式（或者相对路径的方式）去引入文件是不可以的，指点一定注意
<link rel="stylesheet" href="../jingtaiwenjian/xxx.css">
```

**PS：良好的目录结构是每个应用都应该创建自己的urls、forms、views、models、templates和static，每个templates包含一个与应用同名的子目录，每个static也包含一个与应用同名的子目录。**

