---
title: 第十七讲——django综合扩展七之发送邮件与日志记录
id: 17
date: 2019-10-19 20:30:00
tags: Django
comment: true
---

## 发送邮件

在Python中已经内置了一个smtp邮件发送模块，Django在此基础上进行了简单地封装，让我们在Django环境中可以更方便更灵活的发送邮件。

所有的功能都在django.core.mail中。

### 快速上手

两行就可以搞定一封邮件：

```python
from django.core.mail import send_mail

send_mail(
    'Subject here',
    'Here is the message.',
    'from@example.com',
    ['to@example.com'],
    fail_silently=False,
)
```

导入功能模块，然后发送邮件，so easy！

默认情况下，使用配置文件中的`EMAIL_HOST`和`EMAIL_PORT`设置SMTP服务器主机和端口，`EMAIL_HOST_USER`和`EMAIL_HOST_PASSWORD`是用户名和密码。如果设置了`EMAIL_USE_TLS`和`EMAIL_USE_SSL`，它们将控制是否使用相应的加密链接。

<!----more---->

### 单发 send_mail()

方法原型：send_mail(subject, message, from_email, recipient_list, fail_silently=False, auth_user=None, auth_password=None, connection=None, html_message=None)[source]

让我们来了解一下`send_mail()`方法，它接收一系列参数，其中的subject、message、`from_email`和`recipient_list`参数是必须的，其它的可选。

- subject：邮件主题。字符串。
- message：邮件具体内容。字符串。
- from_email：邮件发送者。字符串。
- `recipient_list`：收件人。一个由邮箱地址组成的字符串列表。`recipient_list`中的每一个成员都会在邮件信息的“To:”区域看到其它成员。
- `fail_silently`: 一个布尔值。如果它是False，`send_mail`发送失败时，将会引发一个smtplib.SMTPException异常。
- `auth_user`: 可选的用户名用来验证SMTP服务器，如果你要特别指定使用哪个邮箱帐号，就指定这个参数。如果没有提供这个值，Django将会使用settings中`EMAIL_HOST_USER`的值。如果两者都不提供，那你还发什么？？？
- `auth_password`: 可选的密码用来验证SMTP服务器。如果没有提供这个值，Django 将会使用settings中`EMAIL_HOST_PASSWORD`的值。和上面那个参数是一家的。
- connection: 可选的用来发送邮件的电子邮件后端。
- `html_message`: 如果提供了`html_message`，可以发送带HTML代码的邮件。

`send_mail()`方法返回值将是成功发送出去的邮件数量（只会是0或1，因为它只能发送一封邮件）。

### 群发 send_mass_mail()

方法原型：send_mass_mail（datatuple，fail_silently = False，auth_user = None，auth_password = None ，connection = None）[source]

`send_mass_mail()`用来处理大批量邮件任务，也就是所谓的群发。

它的参数中，datatuple是必需参数，接收一个元组，元组的每个元素的格式如下：

```
(subject, message, from_email, recipient_list)
```

上面四个字段的意义与`send_mail()`中的相同。

例如，以下代码将向两组不同的收件人发送两个不同的消息；但是，只能打开一个到邮件服务器的连接：

```python
message1 = ('Subject here', 'Here is the message', 'from@example.com', ['first@example.com', 'other@example.com'])

message2 = ('Another Subject', 'Here is another message', 'from@example.com', ['second@test.com'])

send_mass_mail((message1, message2), fail_silently=False)
```

`send_mass_mail()`方法的返回值是成功发送的邮件数量。

使用`send_mail()`方法时，每调用一次，它会和SMTP服务器建立一次连接，也就是发一次连一次，效率很低。而`send_mass_mail()`，则只建立一次链接，就将所有的邮件都发送出去，效率比较高。

### 防止头部注入攻击

有时候，我们要根据用户表单的输入来构造电子邮件，这就存在头部注入攻击的风险，Django给我们提供了一定的防范能力，但是更多时候，还需要你自己编写安全防范代码。

下面是一个例子，接收用户输入的主题、邮件内容和发送方，将邮件发送到系统管理员：

```python
from django.core.mail import send_mail, BadHeaderError
from django.http import HttpResponse, HttpResponseRedirect

def send_email(request):
    subject = request.POST.get('subject', '')
    message = request.POST.get('message', '')
    from_email = request.POST.get('from_email', '')
    if subject and message and from_email:
        try:
            send_mail(subject, message, from_email, ['admin@example.com'])
        except BadHeaderError:
            return HttpResponse('Invalid header found.')
        return HttpResponseRedirect('/contact/thanks/')
    else:
        # In reality we'd use a form class
        # to get proper validation errors.
        return HttpResponse('Make sure all fields are entered and valid.')
```

如果检查到用户的输入带有头部注入攻击的可能性，会弹出BadHeaderError异常。

### 发送多媒体邮件

默认情况下，发送的邮件都是纯文本格式的。但有时候我们希望能在邮件里带一些超级链接、图片，甚至视频和JS动作。

Django为我们提供了一个EmailMultiAlternatives类，可以同时发送文本和HTML内容，下面是个范例，我们照着写就行：

```python
from django.core.mail import EmailMultiAlternatives

subject, from_email, to = 'hello', 'from@example.com', 'to@example.com'
text_content = 'This is an important message.'
html_content = '<p>This is an <strong>important</strong> message.</p>'
msg = EmailMultiAlternatives(subject, text_content, from_email, [to])
msg.attach_alternative(html_content, "text/html")
msg.send()
```

需要提醒的是，接收方的邮件服务商不一定支持多媒体邮件，也许是为了安全，也许是别的原因。为了保证你的邮件内容能被阅读，请务必同时发送纯文本邮件。

邮件的实例在项目的登录注册的实现里面。

## Dajngo日志

在Python的logging模块中，主要包含下面四大金刚：

- Loggers： 记录器
- Handlers：处理器
- Filters： 过滤器
- Formatters： 格式化器

### 在Django视图中使用logging

```python
# 导入logging库
import logging

# 获取一个logger对象
logger = logging.getLogger(__name__)

def my_view(request, arg1, arg):
    ...
    if bad_mojo:
        # 记录一个错误日志
        logger.error('Something went wrong!')
```

每满足`bad_mojo`条件一次，就写入一条错误日志。

### 在Django中配置logging

通常，只是像上面的例子那样简单的使用logging模块是远远不够的，我们一般都要对logging的四大金刚进行一定的配置。

Python的logging模块提供了好几种配置方式。默认情况下，Django使用dictConfig format。也就是字典方式。

**例一：**

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'DEBUG',
            'class': 'logging.FileHandler',
            'filename': '/path/to/django/debug.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}
```

**例二：**

下面这个示例配置，让Django将日志打印到控制台，通常用做开发期间的信息展示。

```python
import os

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['console'],
            'level': os.getenv('DJANGO_LOG_LEVEL', 'INFO'),
        },
    },
}
```

### Django对logging模块的扩展

Django对logging模块进行了一定的扩展，用来满足Web服务器专门的日志记录需求。

#### 记录器Loggers

Django额外提供了几个其内建的logger。

- django： 不要使用这个记录器，用下面的。这是一个被供起来的记录器，^-^
- django.request： 记录与处理请求相关的消息。5XX错误被记录为ERROR消息；4XX错误记录为WARNING消息。接收额外参数：status_code和request
- django.server： 记录开发服务器下处理请求相关的消息。只用于开发阶段。
- django.template: 记录与渲染模板相关的日志。
- django.db.backends: 与数据库交互的代码相关的消息。
- django.security： 记录任何与安全相关的错误。
- django.security.csrf： 记录CSRF验证失败日志。
- django.db.backends.schema： 记录查询导致数据库修改的日志。

#### 处理器Handlers

Django额外提供了一个handler，AdminEmailHandler。这个处理器将它收到的每个日志信息用邮件发送给站点管理员。

#### 过滤器Filters

Django还额外提供两个过滤器。

- CallbackFilter(callback)[source]：这个过滤器接受一个回调函数，并对每个传递给过滤器的记录调用它。如果回调函数返回False，将不会进行记录的处理。
- RequireDebugFalse[source]： 这个过滤器只会在settings.DEBUG==False时传递。