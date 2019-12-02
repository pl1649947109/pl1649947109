---
title: 第十五讲——django综合扩展五之序列化和消息框架(模板)
id: 15
date: 2019-10-17 20:30:00
tags: Django
comment: true
---

### 序列化serializers

Django的序列化工具让你可以将Django的模型‘翻译’成其它格式的数据。通常情况下，这种其它格式的数据是基于文本的，并且用于数据交换\传输过程。

<!----more---->

#### 一、序列化数据

Django为我们提供了一个强大的序列化工具serializers。使用它也很简单，如下所示：

```
from django.core import serializers
data = serializers.serialize("xml", SomeModel.objects.all())
```

首先，从djang.core导入它，然后调用它的serialize方法，这个方法至少接收两个参数，第一个是你要序列化成为的数据格式，这里是‘xml’，第二个是要序列化的数据对象，数据通常是ORM模型的QuerySet，一个可迭代的对象。

就是这么简单！！

还有一种比较复杂，但钩子更多的使用方法，如下所示：

```
XMLSerializer = serializers.get_serializer("xml")
xml_serializer = XMLSerializer()
xml_serializer.serialize(queryset)
data = xml_serializer.getvalue()
```

主要是使用了serializers的get_serializer()和getvalue()方法。

当你需要将序列化的数据保存到一个文件对象中的时候，上面的方式就非常有用，例如：

```
with open("file.xml", "w") as out:
    xml_serializer.serialize(SomeModel.objects.all(), stream=out)
```

##### 1. 序列化指定字段

如果你不想序列化模型对象所有字段的内容，只想序列化某些指定的字段，可以使用fields参数，如下所示：

```
from django.core import serializers
data = serializers.serialize('xml', SomeModel.objects.all(), fields=('name','size'))
```

这样，只有name和size字段会被序列化。但是，有一个例外，模型的主键pk被隐式输出了，虽然它并不包含在fields参数中。

##### 2. 序列化继承模型

考虑下面的模型继承：

```
class Place(models.Model):
    name = models.CharField(max_length=50)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
```

如果你只序列化餐厅模型：

```
data = serializers.serialize('xml', Restaurant.objects.all())
```

序列化输出上的字段将只包含`serves_hot_dogs`属性。基类的`name`属性并不会一起序列化。

为了完全序列化Restaurant实例，还需要将Place模型序列化，如下所示：

```
all_objects = list(Restaurant.objects.all()) + list(Place.objects.all())
data = serializers.serialize('xml', all_objects)
```

#### 二、反序列化数据

反序列化也相当简单，如下所示：

```
for obj in serializers.deserialize("xml", data):
    do_something_with(obj)
```

其中的data是我们以前序列化后生成的数据（一个字符串或者数据流）。deserialize()方法返回的是一个迭代器，通过for循环，拿到它内部的每个元素。

在这里有区别的是，deserialize返回的迭代器对象不是简单的Django模型的对象。而是特殊的DeserializedObject实例。调用DeserializedObject.save()方法可以将对象保存到数据库。

PS：如果序列化数据中的pk属性不存在或为null，则新实例将保存到数据库。

将DeserializedObject保存到数据库，可以如下操作：

```
for deserialized_object in serializers.deserialize("xml", data):
    if object_should_be_saved(deserialized_object):
        deserialized_object.save()
```

在这么做之前，你必须保证你的序列化数据是合乎你本地ORM模型属性的，否则保存的过程中会出现各种让你挠头的错误，这是很显然的。

#### 三、可序列化的格式

Djanggo支持三种序列化格式，其中的一些可能需要安装第三方库支持：

- xml
- json
- yaml

##### 1. XML

xml格式相当简单：

```
<?xml version="1.0" encoding="utf-8"?>
<django-objects version="1.0">
    <object pk="123" model="sessions.session">
        <field type="DateTimeField" name="expire_date">2013-01-16T08:16:59.844560+00:00</field>
        <!-- ... -->
    </object>
</django-objects>
```

外键字段被序列化成下面的格式：

```
<object pk="27" model="auth.permission">
    <!-- ... -->
    <field to="contenttypes.contenttype" name="content_type" rel="ManyToOneRel">9</field>
    <!-- ... -->
</object>
```

多对多字段被序列化成下面的样子：

```
<object pk="1" model="auth.user">
    <!-- ... -->
    <field to="auth.permission" name="user_permissions" rel="ManyToManyRel">
        <object pk="46"></object>
        <object pk="47"></object>
    </field>
</object>
```

##### 2. JSON

序列化成json格式后，看起来是下面的样子：

```
[
    {
        "pk": "4b678b301dfd8a4e0dad910de3ae245b",
        "model": "sessions.session",
        "fields": {
            "expire_date": "2013-01-16T08:16:59.844Z",
            ...
        }
    }
]
```

需要注意的是，如果你的ORM模型具有自定义的字段，那么Django给我们提供的序列化工具，就无法正常工作，你必须自己编写相应部分的序列化代码，下面是一个参考的例子：

```
from django.utils.encoding import force_text
from django.core.serializers.json import DjangoJSONEncoder

class LazyEncoder(DjangoJSONEncoder):
    def default(self, obj):
        if isinstance(obj, YourCustomType):
            return force_text(obj)
        return super(LazyEncoder, self).default(obj)
```

上面编写了一个LazyEncoder类，用来实现你的序列化方法，使用下面的方法调用它：

```
from django.core.serializers import serialize

serialize('json', SomeModel.objects.all(), cls=LazyEncoder)
```

PS：Python本身不支持序列化`类`到json格式，Django帮我们实现了它自己的模型类序列化为json的方法，但也仅限于此，如果你在Django内写了一个别的自定义类，一样无法序列化为json格式，除非你自己实现，像上面的例子所示。

##### 3. YAML

yaml的格式和json很像，如下所示：

```
-   fields: {expire_date: !!timestamp '2013-01-16 08:16:59.844560+00:00'}
    model: sessions.session
    pk: 4b678b301dfd8a4e0dad910de3ae245b
```

### 消息框架message

在网页应用中，我们经常需要在处理完表单或其它类型的用户输入后，显示一个通知信息给用户。

对于这个需求，Django提供了基于Cookie或者会话的消息框架messages，无论是匿名用户还是认证的用户。这个消息框架允许你临时将消息存储在请求中，并在接下来的请求（通常就是下一个请求）中提取它们并显示。每个消息都带有一个特定的level标签，表示其优先级（例如info、 warning或error）。

#### 启用消息框架

Django的messages消息框架的实现，依赖messages中间件和对应的context processor。

通过`django-admin startproject xxx`命令创建工程时，已经默认在settings.py中开启了消息框架功能需要的所有的设置：

- INSTALLED_APPS中注册的'django.contrib.messages'。
- MIDDLEWARE中添加'django.contrib.sessions.middleware.SessionMiddleware'和'django.contrib.messages.middleware.MessageMiddleware'。Django的messages框架默认使用的存储后端为sessions。所以Session中间件必须被启用，并出现在Message中间件之前。
- TEMPLATES设置中的DjangoTemplates选项包含的'context_processors'配置项要包含'django.contrib.messages.context_processors.messages'。

#### 配置消息引擎

通常我们使用默认的就好，可以跳过这节，但如果真有需要，也可以配置：http://www.liujiangblog.com/course/django/172

#### 使用消息框架

##### 1.添加消息

方法原型：add_message(request, level, message, extra_tags='', fail_silently=False)[source]

新增一条消息：

```python
from django.contrib import messages
messages.add_message(request, messages.INFO, 'Hello world.')
```

提供请求对象request（直接用就行），消息级别、消息内容字符串三个参数即可。

或者使用下面的快捷方式

```python
messages.debug(request, '%s SQL statements were executed.' % count)
messages.info(request, 'Three credits remain in your account.')
messages.success(request, 'Profile details updated.')
messages.warning(request, 'Your account expires in three days.')
messages.error(request, 'Document deleted.')
```

##### 2.显示消息

方法原型：get_messages(request)[source]

在你的模板文件中，像下面这样使用：

```html
{% if messages %}
<ul class="messages">
    {% for message in messages %}
    <li{% if message.tags %} class="{{ message.tags }}"{% endif %}>{{ message }}</li>
    {% endfor %}
</ul>
{% endif %}
```

相关说明：

- 通过if判断是否有消息；
- messages是一个列表，必须用for标签循环它；
- 即使你知道只有一条消息，也要迭代messages列表，否则下个请求中，上个请求的消息不会被清除。
- 可以通过message.tags拿到每个消息的CSS样式

有一个`DEFAULT_MESSAGE_LEVELS`变量，它映射消息级别的名称到它们的数值：

```html
{% if messages %}
<ul class="messages">
    {% for message in messages %}
    <li{% if message.tags %} class="{{ message.tags }}"{% endif %}>
        {% if message.level == DEFAULT_MESSAGE_LEVELS.ERROR %}Important: {% endif %}
        {{ message }}
    </li>
    {% endfor %}
</ul>
{% endif %}
```

说明：

- 可以通过message.level拿到当前消息的级别数值；
- 将它与DEFAULT_MESSAGE_LEVELS.ERROR进行对比；
- 如果一样，就说明当前消息级别为ERROR，需要显示到页面上。

在模板的外面，比如视图中，可以使用`get_messages()`方法获取消息：

```python
from django.contrib.messages import get_messages

storage = get_messages(request)
for message in storage:
    do_something_with_the_message(message)
```

说明：

- get_messages()返回的是存储后端的一个实例。
- 循环这个实例，可以获得每条消息

对于每一个消息实例，都包含下面的属性，可以在模版或视图中调用：

- message: 消息的实际内容文本。不要使用message.message，直接message。
- level: 消息级别，一个整数。
- tags: 一个字符串，由该消息的所有标签(extra_tags和tags)组合而成，组合时用空格分割开这些标签。
- `extra_tags`: 一个字符串，由该消息的定制标签组合而成，并用空格分割。默认为空。
- `level_tag`: 当前消息级别对应的CSS字符串，前面介绍过。

##### 3.自定义笑消息级别

消息级别只是一个整数常量，所以，可以定义自己的级别常量，例如：

```python
CRITICAL = 50

def my_view(request):
    messages.add_message(request, CRITICAL, 'A serious error occurred.')
```

在自定义消息级别时，应小心避免覆盖现有级别。内置级别的值为：

| 级别    | 对应整数值 |
| ------- | ---------- |
| DEBUG   | 10         |
| INFO    | 20         |
| SUCCESS | 25         |
| WARNING | 30         |
| ERROR   | 40         |

如果你需要在HTML或CSS中使用自定义级别，则需要通过`MESSAGE_TAGS`设置提供相应的映射关系。

##### 4.自定义每个请求的最小记录级别

每个请求都可以通过`set_level()`方法设置最小记录级别，如下所示:

```python
from django.contrib import messages

# 修改最小级别为DEBUG
messages.set_level(request, messages.DEBUG)
messages.debug(request, 'Test message...')

# 在另外一个视图中修改最小级别为WARNING
messages.set_level(request, messages.WARNING)
messages.success(request, 'Your profile was updated.') # 被忽略，不记录
messages.warning(request, 'Your account is about to expire.') # 记录

# 将最小级别恢复到默认值
messages.set_level(request, None)
```

`set_level()`方法接收request为第一参数，消息级别为第二参数。

类似的，当前有效的记录级别可以用`get_level()`方法获取:

```python
from django.contrib import messages
current_level = messages.get_level(request)
```

##### 5. 添加额外的消息CSS样式

要添加自定义的消息CSS样式，可以通过extra_tags参数：

```python
messages.add_message(request, messages.INFO, 'Over 9000!', extra_tags='dragonball')
messages.error(request, 'Email box full', extra_tags='email')
```

#### 消息过期机制

默认情况下，如果包含消息的迭代器完成迭代后，当前请求中的消息都将被删除。

如果你不想这么做，想保留这些消息，那么需要显式的指定used参数为False，如下所示：

```python
storage = messages.get_messages(request)
for message in storage:
    do_something_with(message)
storage.used = False
```