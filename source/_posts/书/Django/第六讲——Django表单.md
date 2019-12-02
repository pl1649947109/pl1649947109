---
title: 第六讲——Django之表单
id: 6
date: 2019-9-29 20:00:00
tags: Django
comment: true
---

### HTML表单概述

一个HTML表单必须指定两样东西：

- 目的地：用户数据发送的目的URL
- 方式：发送数据所使用的HTTP方法

Django的后台登陆源码

```html
<form action="/admin/login/?next=/admin/" method="post" id="login-form">

    <input type='hidden' name='csrfmiddlewaretoken'              value='NNHZaDVJGduajNMECXygKZkAt8vyEcw9HS2qm2Vdf7brDZrA0qK1R0I7M2p3TKcs' />

    <div class="form-row">
        <label class="required" for="id_username">用户名:</label> 
        <input type="text" name="username" autofocus maxlength="254" required id="id_username" />
    </div>

    <div class="form-row">
        <label class="required" for="id_password">密码:</label> 
        <input type="password" name="password" required id="id_password" />
        <input type="hidden" name="next" value="/admin/" />
    </div>

    <div class="submit-row">
        <label>&nbsp;</label><input type="submit" value="登录" />
    </div>
</form>
```

<!----more---->

**GET和POST**

GET方法将用户数据以`键=值`的形式，以‘&’符号组合在一起成为一个整体字符串，最后添加前缀“？”，将字符串拼接到url内，生成一个类似`https://docs.djangoproject.com/search/?q=forms&release=1`的URL。

而对于POST方法，浏览器会组合表单数据、对它们进行编码，然后打包将它们发送到服务器，数据不会出现在url中。

### Dajngo的form表单

通常情况下，我们需要手动在HTML页面中，编写form标签和其内部的其他元素。但这费时费力，而且有可能写的不太恰当，数据验证也比较麻烦。有鉴于此，Django在内部集成了一个表单模块，专门帮助我们快速处理表单相关的内容。Django的表单模块给我们提供了下面三个主要功能：

- 生成页面可用的HTML标签
- 对用户提交的数据进行校验
- 保留上次输入的内容

### 去使用表单

#### 编写表单类

我们可以通过Django提供的Form类来自用生成上面的表单，不再需要手动在HTML中编写。

首先，我们在当前的app内创建一个forms.py文件：

```python
from django import forms
class NameForm(forms.Form):
	your_name = forms.CharField(label="Your name"max_lehgth=128)
```

注意：**由于浏览器页面是可以被篡改、伪造、禁用、跳过的，所有的HTML手段的数据验证只能防止意外不能防止恶意行为，是没有安全保证的，破坏分子完全可以跳过浏览器的防御手段伪造发送请求！所以，在服务器后端，必须将前端当做“裸机”来对待，再次进行完全彻底的数据验证和安全防护！**

当我们将上面的表单渲染程真正的HTML元素，其内容如下：

```html
<label for="your_name">Your name: </label>
<input id="your_name" type="text" name="your_name" maxlength="100" required />
```

注意：渲染成的表单，它不包含form标签本身以及提交按钮，为什么这样做呢？就是为了方便我们自己控制表单动作和处理css，js以及其他类Bootstrap框架的嵌入。

#### 视图处理

在视图中，实例化我们编写好的表单类

```python
from django.shortcuts import render
from django.http import HttpResponseRedirect
from .forms import NameForm

def get_name(request):
    #如果form通过POST方法发送数据
    if request.method == "POST":
        #接受后request.POST参数构造form类实例
        form = NameForm(request.POST)
        #验证数据是否合法
        if form.is_valid():
            #重定向
            return HttpResponseRedirect('/thanks/')
    #如果通过GET方法请求的数据，返回空表单
    else:
        form = NameForm()
    return render(request,'name.html',{'form':form})
```

注意：

- 对于GET方法请求的页面数据，返回空的表单，让用户可以填入数据
- 对于POST方法，接收表单数据，并进行验证
- 如果数据是合法的，按照正常业务的逻辑继续执行下去
- 如果数据是不合法的，返回一个包含先前数据的表单给前端的页面，方便用户修改
- 我们还可以通过is_bound属性货值表单已经绑定了数据，还是一个控表

#### 模板处理

在Django模板中，我们只需要按照下面的方式进行处理，就可以得到完整的HTML页面：

```html
<form action="/your_name/" method="POST">
	{% csrf_token %}
	{{ form }}
	<input type="submit" value="Submit"/>
</form>
```

注意：

- form标签需要自己写
- 使用POST方法，必须添加csrf，用来处理csrf安全机制
- 中间的form，就是从后台传来的form就是Dj为我们生成的form标签元素。
- 提交按钮需要手动添加
- Dj支持HTML5表单验证的功能，比如邮箱地址验证、必填项目验证等等。

### 高级用法

上面的例子中，只是一个用户名输入框，太简单了。

```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField(widget=forms.Textarea)
    sender = forms.EmailField()
    cc_myself = forms.BooleanField(required=False)
```

这个例子就有4个框组了。实际上Django的表单模块为我们内置了许多表单字段。每一个表单字段类型都对应一种Widget类，每一种Widget类都对应了HMTL语言中的一种input元素类型，比如`<input type="text">`。需要在HTML中实际使用什么类型的input，就需要在Django的表单字段中选择相应的field。比如要一个`<input type="text">`，可以选择一个`CharField`。

一旦你的表单接收数据并验证通过了，那么就可以从`form.cleaned_data`字典中读取所有的表单数据，下面是一个例子：

```python
# views.py

from django.core.mail import send_mail

if form.is_valid():
    subject = form.cleaned_data['subject']
    message = form.cleaned_data['message']
    sender = form.cleaned_data['sender']
    cc_myself = form.cleaned_data['cc_myself']

    recipients = ['info@example.com']
    if cc_myself:
        recipients.append(sender)

    send_mail(subject, message, sender, recipients)
    return HttpResponseRedirect('/thanks/')
```

### 使用表单模板

#### 表单渲染格式

前面我们通过{{ form }}模板语言，简单地将表单渲染到HTML页面中了，实际上，有更多的方式：

- form.as_table将表单渲染成一个表格元素，每个输入框作为一个<tr>标签
- form.as_p 将表单的每个输入框包裹在一个`<p>`标签内 tags
- form.as_ul 将表单渲染成一个列表元素，每个输入框作为一个`<li>`标签

下面是将上面的ContactForm作为form.as_p 的例子：

```html
<p><label for="id_subject">Subject:</label>
    <input id="id_subject" type="text" name="subject" maxlength="100" required /></p>
<p><label for="id_message">Message:</label>
    <textarea name="message" id="id_message" required></textarea></p>
<p><label for="id_sender">Sender:</label>
    <input type="email" name="sender" id="id_sender" required /></p>
<p><label for="id_cc_myself">Cc myself:</label>
    <input type="checkbox" name="cc_myself" id="id_cc_myself" /></p>
```

注意：Django自动为每个input元素设置了一个id名称。

#### 手动渲染表单字段

直接form虽然好，啥都不用操心，但是往往并不是你想要的，比如你要使用CSS和JS，比如你要引入Bootstarps框架，这些都需要对表单内的input元素进行额外控制，那怎么办呢？手动渲染字段就可以了。

我们可以使用 form.name_of_field  获取每一个字段，然后分别渲染，如下例所示：

```html
{{ form.non_field_errors }}
<div class="fieldWrapper">
    {{ form.subject.errors }}
    <label for="{{ form.subject.id_for_label }}">Email subject:</label>
    {{ form.subject }}
</div>
<div class="fieldWrapper">
    {{ form.message.errors }}
    <label for="{{ form.message.id_for_label }}">Your message:</label>
    {{ form.message }}
</div>
<div class="fieldWrapper">
    {{ form.sender.errors }}
    <label for="{{ form.sender.id_for_label }}">Your email address:</label>
    {{ form.sender }}
</div>
<div class="fieldWrapper">
    {{ form.cc_myself.errors }}
    <label for="{{ form.cc_myself.id_for_label }}">CC yourself?</label>
    {{ form.cc_myself }}
</div>
```

其中的label标签甚至可以用label_tag()方法生成，于是就有下面的简写方式：

```html
<div class="fieldWrapper">
    {{ form.subject.errors }}
    {{ form.subject.label_tag }}
    {{ form.subject }}
</div>
```

这种方式就显的灵活了很多，但是代价就是我们需要多写代码。

#### 渲染表单错误信息

一切非字段的错误信息，比如表单的错误，隐藏字段的错误都保存在 form.non_field_errors  中，上面的例子，我们把它放在了表单的外围上面，它将被按下面的HTML和CSS格式渲染：

```html
<ul class="errorlist nonfield">
    <li>Generic validation error</li>
</ul>
```

#### 循环表单的字段

如果你的表单字段有相同格式的HMTL表现，那么完全可以循环生成，不必要手动的编写每个字段，减少冗余和重复代码，只需要使用模板语言中的 for 循环，如下所示：

```html
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
        {% if field.help_text %}
        <p class="help">{{ field.help_text|safe }}</p>
        {% endif %}
    </div>
{% endfor %}
```

其中，我们通过field可以获得一些信息：

| 属性                       | 说明                                                        |
| -------------------------- | ----------------------------------------------------------- |
| `{{ field.label }}`        | 字段对应的label信息                                         |
| `{{ field.label_tag }}`    | 自动生成字段的label标签，注意与`{{ field.label }}`的区别。  |
| `{{ field.id_for_label }}` | 自定义字段标签的id                                          |
| `{{ field.value }}`        | 当前字段的值，比如一个Email字段的值`someone@example.com`    |
| `{{ field.html_name }}`    | 指定字段生成的input标签中name属性的值                       |
| `{{ field.help_text }}`    | 字段的帮助信息                                              |
| `{{ field.errors }}`       | 包含错误信息的元素                                          |
| `{{ field.is_hidden }}`    | 用于判断当前字段是否为隐藏的字段，如果是，返回True          |
| `{{ field.field }}`        | 返回字段的参数列表。例如`{{ char_field.field.max_length }}` |

#### 不可见字段的特殊处理

就是出现错误的时候，让一些报错误的内容隐藏起来，Django提供了两种独立的方法，用于循环那些不可见的和可见的字段，`hidden_fields()`和`visible_fields()`。这里，我们可以稍微修改一下前面的例子：

```html
{# 循环那些不可见的字段 #}
{% for hidden in form.hidden_fields %}
{{ hidden }}
{% endfor %}
{# 循环可见的字段 #}
{% for field in form.visible_fields %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
    </div>
{% endfor %}
```

#### 重用表单模板

如果你在自己的HTML文件中，多次使用同一种表单模板，那么你完全可以把表单模板存成一个独立的HTML文件，然后在别的HTML文件中通过include模板语法将其包含进来，如下例所示：

```html
# 实际的页面文件中:
{% include "form_snippet.html" %}

-----------------------------------------------------

# 单独的表单模板文件form_snippet.html:
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
    </div>
{% endfor %}
```

### 表单API

#### 表单的绑定属性

如果我们需要区分绑定的表单和未绑定的表单，我们可以检查表单的is_bound属性值：

```python
>>> f = ContactForm()    #未绑定数据
>>> f.is_bound
False
>>> f = ContactForm({'subject': 'hello'})    #绑定了数据，即使传的是一个空字典返回的也是True
>>> f.is_bound
True
```

注意：Form实例的数据没有自读的功能，Form实例一旦创建，它的数据将不会改改变。

#### 使用表单验证数据

- Form.clean()

  如果我们自定义验证功能，那么就需要重新实现这个clean()方法。

- Form.is_valid()

  调用该方法来执行绑定表单的数据验证工作，并返回一个表示数据是否合法的布尔值。

  实例：

  ```python
  data = {
      'subject': '',   #subject为空（默认所有字段都是必需的）
  	'message': 'Hi there',
  	'sender': 'invalid email address',	#sender是一个不合法的邮件地址
  	'cc_myself': True}
  f = ContactForm(data)
  f.is_valid()
  >>>False  #返回的布尔值是False
  ```

- Form.errors

  表单errors属性保存了错误信息字典。

  ```python
  f = ContactForm(data)
  f.errors
  >>>{'sender': ['Enter a valid email address.'], 	'subject': ['This field is required.']}
  ```

  在上面的这个字典中，键为字典的名称，值为错误的信息列表。错误信息保存在列表中是因为字段可能有多个错误的信息。

- Form_errors.as_data()

  返回一个字典，它将映射到原始的ValidationError实例。

  ```python
  >>> f.errors.as_data()
  {'sender': [ValidationError(['Enter a valid email address.'])],
  'subject': [ValidationError(['This field is required.'])]}
  ```

- Form.errors.as_join(escape_html=False)

  返回JSON序列化后的错误信息字典。

  ```python
  >>> f.errors.as_json()
  {"sender": [{"message": "Enter a valid email address.", "code": "invalid"}],
  "subject": [{"message": "This field is required.", "code": "required"}]}
  ```

- Form.add_error(field,error)

  向表单特定字段添加错误信息。

  field参数为字段的名称，如果值为None，error将作为`Form.non_field_errors()`的一个非字段错误。

- Form.has_error(field,code=None)

  判断某个字段是否具有指定code的错误。当code为None时，如果字段有任何错误它都将返回True。

- Form.non_field_errors()

  返回Form.errors中不是与特定字段相关联的错误。

- 对于没有绑定数据的表单

  验证没有绑定数据的表单是没有意义的。因为返回的不是False就是空字典。

#### 检查表单数据是否被修改

- Form.has_changed()

  当我们需要检查表单的数据是否从初始数据发生改变时，可以使用该方法。

  ```python
  >>> f = ContactForm(request.POST, initial=data)
  #第一个参数是用户提交的数据，第二个参数是我们后台给表单初始的数据。
  >>> f.has_changed()    #进行判断
  False    #返回False说明数据没有修改，否则就修改了
  ```

- Form.chhangeed_data

  返回有变化的字段的列表

  ```python
  f = ContactForm(request.POST, initial=data)
  if f.has_changed():
  	print("The following fields changed: %s" % ", ".join(f.changed_data))
  ```

  **方法和属性结合使用。**

#### 访问表单中的字段

使用field属性访问

#### 访问cleaned_data

**Form类中的每个字段不仅负责验证数据，还负责将他们转换成正确的格式**。例如，DateField将输入转换为Python的datetime.date对象。无论你传递的是普通字符串'1994-07-15'、DateField格式的字符串、datetime.date对象、还是其它格式的数字，Django将始终把它们转换成datetime.date对象。

一旦我们创建一个Form实例并通过验证后，我们就可以通过它的cleaned_data属性访问干净的数据：

```python
data = {'subject': 'hello',
        'message': 'Hi there',
        'sender': 'foo@example.com',
        'cc_myself': True}
f = ContactForm(data)
f.is_valid()
>>>True
f.cleaned_data
>>>{'cc_myself': True, 'message': 'Hi there', 			'sender': 'foo@example.com', 'subject': 'hello'}
```

注意：如果我们的数据没有通过验证，cleaned_data字典中只能包含合法的字段。

#### 表单的HTML生成方式

Form的第二个任务就是将它渲染成HTML代码，默认的情况下，根据form类中字段的编写顺序，在HTML中以同样的顺序罗列。

```html
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> print(f)
<tr><th><label for="id_subject">Subject:</label></th><td><input id="id_subject" type="text" name="subject" maxlength="100" value="hello" required /></td></tr>
<tr><th><label for="id_message">Message:</label></th><td><input type="text" name="message" id="id_message" value="Hi there" required /></td></tr>
<tr><th><label for="id_sender">Sender:</label></th><td><input type="email" name="sender" id="id_sender" value="foo@example.com" required /></td></tr>
<tr><th><label for="id_cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="id_cc_myself" checked /></td></tr>
```

- 为了灵活性，输出不包含`<table>`和`</table>`、`<form>`和`</form>`以及`<input type="submit">`标签。 需要我们程序员手动添加它们。
- 每个字段类型都由一个默认的HTML标签展示。注意，这些只是默认的，可以使用Widget 特别指定。
- 每个HTML标签的name属性名直接从ContactForm类中获取。
- form使用HTML5语法，顶部需添加`<!DOCTYPE html>`说明。

**渲染成文字段落as_p()**

该方法将form渲染成一系列`<p>`标签，每个`<p>`标签包含一个字段；

```html
>>> f = ContactForm()
>>> print(f.as_p())
<p><label for="id_subject">Subject:</label> <input id="id_subject" type="text" name="subject" maxlength="100" required /></p>
<p><label for="id_message">Message:</label> <input type="text" name="message" id="id_message" required /></p>
<p><label for="id_sender">Sender:</label> <input type="email" name="sender" id="id_sender" required /></p>
<p><label for="id_cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="id_cc_myself" /></p>
```

**渲染成无序列表as_ul()**

```html
>>> f = ContactForm()
>>> print(f.as_ul())
<li><label for="id_subject">Subject:</label> <input id="id_subject" type="text" name="subject" maxlength="100" required /></li>
<li><label for="id_message">Message:</label> <input type="text" name="message" id="id_message" required /></li>
<li><label for="id_sender">Sender:</label> <input type="email" name="sender" id="id_sender" required /></li>
<li><label for="id_cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="id_cc_myself" /></li>
```

**渲染成表格as_table()**

```html
>>> f = ContactForm()
>>> print(f.as_table())
<tr><th><label for="id_subject">Subject:</label></th><td><input id="id_subject" type="text" name="subject" maxlength="100" required /></td></tr>
<tr><th><label for="id_message">Message:</label></th><td><input type="text" name="message" id="id_message" required /></td></tr>
<tr><th><label for="id_sender">Sender:</label></th><td><input type="email" name="sender" id="id_sender" required /></td></tr>
<tr><th><label for="id_cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="id_cc_myself" /></td></tr>
```

### Django表单字段汇总

**Field.clean(value)[source]**

每个Field的实例都有一个clean()方法，它接受一个参数，然后返回“清洁的”数据或者抛出一个`django.forms.ValidationError`异常：

```python
from django import forms
f = forms.EmailField()	#实例化field
f.clean('foo@example.com')
>>>'foo@example.com'	#验证通过，返回干净的数据
f.clean('invalid email address')
'''Traceback (most recent call last):    
...
ValidationError: ['Enter a valid email address.']'''#验证没有通过，抛出错误
```

注意：这个clean()方法经常被我们用来开发或者测试过程中对数据进行验证和测试使用的。

#### 核心字段参数

以下的参数是每个Field类都可以使用的。

- required

  给字段添加必填的属性，不能空着。

  ```python
  >>> from django import forms
  >>> f = forms.CharField()
  >>> f.clean('foo')
  'foo'
  >>> f.clean('')   #这就是空着的数据，报错
  Traceback (most recent call last):
  ...
  ValidationError: ['This field is required.']
  >>> f.clean(None)  #None也不行
  Traceback (most recent call last):
  ...
  ValidationError: ['This field is required.']
  >>> f.clean(' ')
  ' '
  >>> f.clean(0)
  '0'
  >>> f.clean(True)
  'True'
  >>> f.clean(False)
  'False'
  ```

  若要表示一个字段不是必需的，设置required=False：

  ```python
  >>> f = forms.CharField(required=False)
  >>> f.clean('foo')
  'foo'
  >>> f.clean('')    #这样就可以了
  ''
  >>> f.clean(None)   #None也不会报错
  ''
  >>> f.clean(0)
  '0'
  >>> f.clean(True)
  'True'
  >>> f.clean(False)
  'False'
  ```

- label

  label参数用来给字段添加‘人类友好’的提示信息。如果没有设置这个参数，那么就用字段的首字母大写名字。比如：

  下面的例子，前两个字段有，最后的comment没有label参数：

  ```python
  from django import forms
  class CommentForm(forms.Form):
      name = forms.CharField(label='Your name')
      url = forms.URLField(label='Your website', required=False)
      comment = forms.CharField()
  f = CommentForm(auto_id=False)
  print(f)
  <tr><th>Your name:</th><td><input type="text" name="name" required /></td></tr>    #自定义
  <tr><th>Your website:</th><td><input type="url" name="url" /></td></tr>    #自定义
  <tr><th>Comment:</th><td><input type="text" name="comment" required /></td></tr>    #默认，首字母大写
  ```

- label_suffix

  Django默认为上面的label参数后面加个冒号后缀，如果想自定义，可以使用`label_suffix`参数。比如下面的例子用“？”代替了冒号：

  ```python
  >>> class ContactForm(forms.Form):
  ...     age = forms.IntegerField()
  ...     nationality = forms.CharField()
  ...     captcha_answer = forms.IntegerField(label='2 + 2', label_suffix=' =')
  >>> f = ContactForm(label_suffix='?')
  >>> print(f.as_p())
  <p><label for="id_age">Age?</label> <input id="id_age" name="age" type="number" required /></p>
  <p><label for="id_nationality">Nationality?</label> <input id="id_nationality" name="nationality" type="text" required /></p>
  <p><label for="id_captcha_answer">2 + 2 =</label> <input id="id_captcha_answer" name="captcha_answer" type="number" required /></p>
  ```

- initial

  为HTML页面中表单元素定义初始值。也就是input元素的value参数的值。

  ```python
  >>> from django import forms
  >>> class CommentForm(forms.Form):
  ...     name = forms.CharField(initial='Your name')
  ...     url = forms.URLField(initial='http://')
  ...     comment = forms.CharField()
  >>> f = CommentForm(auto_id=False)
  >>> print(f)
  <tr><th>Name:</th><td><input type="text" name="name" value="Your name" required /></td></tr>
  <tr><th>Url:</th><td><input type="url" name="url" value="http://" required /></td></tr>
  <tr><th>Comment:</th><td><input type="text" name="comment" required /></td></tr>
  ```

  你可能会问为什么不在渲染表单的时候传递一个包含初始化值的字典给它，不是更方便？因为如果这么做，你将触发表单的验证过程，此时输出的HTML页面将包含验证中产生的错误

  注意：**如果提交表单时某个字段的值没有填写，initial的值不会作为“默认”的数据。initial值只用于原始表单的显示。**

  除了常量之外，你还可以传递一个可调用的对象：

  ```python
  >>> import datetime
  >>> class DateForm(forms.Form):
  ...     day = forms.DateField(initial=datetime.date.today)
  >>> print(DateForm())
  <tr><th>Day:</th><td><input type="text" name="day" value="12/23/2008" required /><td><tr>
  ```

- widget

  最重要的参数之一，指定渲染Widget时使用的widget类，也就是这个form字段在HTML页面中是显示为文本输入框？密码输入框？单选按钮？多选框？还是别的....后面讲

- help_text

  该参数用于设置字段的辅助描述文本。

  ```python
  >>> from django import forms
  >>> class HelpTextContactForm(forms.Form):
  ...     subject = forms.CharField(max_length=100, help_text='100 characters max.')
  ...     message = forms.CharField()
  ...     sender = forms.EmailField(help_text='A valid email address, please.')
  ...     cc_myself = forms.BooleanField(required=False)
  ```

- error_messages

  该参数允许你覆盖字段引发异常时的默认信息。 传递的是一个字典，其键为你想覆盖的错误信息。 

  下面是默认的错误信息：

  ```python
  >>> from django import forms
  >>> generic = forms.CharField()
  >>> generic.clean('')
  Traceback (most recent call last):
    ...
  ValidationError: ['This field is required.']
  ```

  下面是自定义的错误信息：

  ```python
  >>> name = forms.CharField(error_messages={'required': 'Please enter your name'})
  >>> name.clean('')
  Traceback (most recent call last):
    ...
  ValidationError: ['Please enter your name']
  ```

- validators

  指定一个列表，其中包含了为字段进行验证的函数。也就是说，如果你自定义了验证方法，不用Django内置的验证功能，那么要通过这个参数，将字段和自定义的验证方法链接起来。

- localize

  localize参数帮助实现表单数据输入的本地化。

- disabled

  设置有该属性的字段在前端页面中将显示为不可编辑状态。

  该参数接收布尔值，当设置为True时，使用HTML的disabled属性禁用表单域，以使用户无法编辑该字段。即使非法篡改了前端页面的属性，向服务器提交了该字段的值，也将依然被忽略。

#### Django表单内置的字段

就是和数据库的字段一样，具体的内容参见http://www.liujiangblog.com/course/django/154

```
Field
    required=True,               是否允许为空
    widget=None,                 HTML插件
    label=None,                  用于生成Label标签或显示内容
    initial=None,                初始值
    help_text='',                帮助信息(在标签旁边显示)
    error_messages=None,         错误信息 {'required': '不能为空', 'invalid': '格式错误'}
    validators=[],               自定义验证规则
    localize=False,              是否支持本地化
    disabled=False,              是否可以编辑
    label_suffix=None            Label内容后缀
 
 
CharField(Field)
    max_length=None,             最大长度
    min_length=None,             最小长度
    strip=True                   是否移除用户输入空白
 
IntegerField(Field)
    max_value=None,              最大值
    min_value=None,              最小值
 
FloatField(IntegerField)
    ...
 
DecimalField(IntegerField)
    max_value=None,              最大值
    min_value=None,              最小值
    max_digits=None,             总长度
    decimal_places=None,         小数位长度
 
BaseTemporalField(Field)
    input_formats=None          时间格式化   
 
DateField(BaseTemporalField)    格式：2015-09-01
TimeField(BaseTemporalField)    格式：11:12
DateTimeField(BaseTemporalField)格式：2015-09-01 11:12
 
DurationField(Field)            时间间隔：%d %H:%M:%S.%f
    ...
 
RegexField(CharField)
    regex,                      自定制正则表达式
    max_length=None,            最大长度
    min_length=None,            最小长度
    error_message=None,         忽略，错误信息使用 error_messages={'invalid': '...'}
 
EmailField(CharField)      
    ...
 
FileField(Field)
    allow_empty_file=False     是否允许空文件
 
ImageField(FileField)      
    ...
    注：需要PIL模块，pip3 install Pillow
    以上两个字典使用时，需要注意两点：
        - form表单中 enctype="multipart/form-data"
        - view函数中 obj = MyForm(request.POST, request.FILES)
 
URLField(Field)
    ...
 
 
BooleanField(Field)  
    ...
 
NullBooleanField(BooleanField)
    ...
 
ChoiceField(Field)
    ...
    choices=(),                选项，如：choices = ((0,'上海'),(1,'北京'),)
    required=True,             是否必填
    widget=None,               插件，默认select插件
    label=None,                Label内容
    initial=None,              初始值
    help_text='',              帮助提示
 
 
ModelChoiceField(ChoiceField)
    ...                        django.forms.models.ModelChoiceField
    queryset,                  # 查询数据库中的数据
    empty_label="---------",   # 默认空显示内容
    to_field_name=None,        # HTML中value的值对应的字段
    limit_choices_to=None      # ModelForm中对queryset二次筛选
     
ModelMultipleChoiceField(ModelChoiceField)
    ...                        django.forms.models.ModelMultipleChoiceField
 
 
     
TypedChoiceField(ChoiceField)
    coerce = lambda val: val   对选中的值进行一次转换
    empty_value= ''            空值的默认值
 
MultipleChoiceField(ChoiceField)
    ...
 
TypedMultipleChoiceField(MultipleChoiceField)
    coerce = lambda val: val   对选中的每一个值进行一次转换
    empty_value= ''            空值的默认值
 
ComboField(Field)
    fields=()                  使用多个验证，如下：即验证最大长度20，又验证邮箱格式
                               fields.ComboField(fields=[fields.CharField(max_length=20), fields.EmailField(),])
 
MultiValueField(Field)
    PS: 抽象类，子类中可以实现聚合多个字典去匹配一个值，要配合MultiWidget使用
 
SplitDateTimeField(MultiValueField)
    input_date_formats=None,   格式列表：['%Y--%m--%d', '%m%d/%Y', '%m/%d/%y']
    input_time_formats=None    格式列表：['%H:%M:%S', '%H:%M:%S.%f', '%H:%M']
 
FilePathField(ChoiceField)     文件选项，目录下文件显示在页面中
    path,                      文件夹路径
    match=None,                正则匹配
    recursive=False,           递归下面的文件夹
    allow_files=True,          允许文件
    allow_folders=False,       允许文件夹
    required=True,
    widget=None,
    label=None,
    initial=None,
    help_text=''
 
GenericIPAddressField
    protocol='both',           both,ipv4,ipv6支持的IP格式
    unpack_ipv4=False          解析ipv4地址，如果是::ffff:192.0.2.1时候，可解析为192.0.2.1， PS：protocol必须为both才能启用
 
SlugField(CharField)           数字，字母，下划线，减号（连字符）
    ...
 
UUIDField(CharField)           uuid类型
```

#### 实例：

**单radio值为字符串**

```python
class LoginForm(forms.Form):
    username = forms.CharField(  #其他选择框或者输入框，基本都是在这个CharField的基础上通过插件来搞的
        min_length=8,
        label="用户名",
        initial="张三",
        error_messages={
            "required": "不能为空",
            "invalid": "格式错误",
            "min_length": "用户名最短8位"
        }
    )
    pwd = forms.CharField(min_length=6, label="密码")
    gender = forms.fields.ChoiceField(
        choices=((1, "男"), (2, "女"), (3, "保密")),
        label="性别",
        initial=3,
        widget=forms.widgets.RadioSelect()
    )
```

**单选Select**

```python
class LoginForm(forms.Form):
    ...
    hobby = forms.fields.ChoiceField(  #注意，单选框用的是ChoiceField，并且里面的插件是Select，不然验证的时候会报错， Select a valid choice的错误。
        choices=((1, "篮球"), (2, "足球"), (3, "双色球"), ),
        label="爱好",
        initial=3,
        widget=forms.widgets.Select()
    )
```

**多选Select**

```python
class LoginForm(forms.Form):
    ...
    hobby = forms.fields.MultipleChoiceField( #多选框的时候用MultipleChoiceField，并且里面的插件用的是SelectMultiple，不然验证的时候会报错。
        choices=((1, "篮球"), (2, "足球"), (3, "双色球"), ),
        label="爱好",
        initial=[1, 3],
        widget=forms.widgets.SelectMultiple()
    )
```

**单选checkbox**

```python
#单选的checkbox
    class TestForm2(forms.Form):
        keep = forms.ChoiceField(
            choices=(
                ('True',1),
                ('False',0),
            ),

            label="是否7天内自动登录",
            initial="1",
            widget=forms.widgets.CheckboxInput(), 
        )
    选中:'True'   #form只是帮我们做校验,校验选择内容的时候,就是看在没在我们的choices里面,里面有这个值,表示合法,没有就不合法
    没选中:'False'
    ---保存到数据库里面  keep:'True'
    if keep == 'True':
        session 设置有效期7天
    else:
        pass
```

**多选ckeckbox**

```python
class LoginForm(forms.Form):
    ...
    hobby = forms.fields.MultipleChoiceField(
        choices=((1, "篮球"), (2, "足球"), (3, "双色球"),),
        label="爱好",
        initial=[1, 3],
        widget=forms.widgets.CheckboxSelectMultiple()
    )
```

**data类型**

```python
from django import forms
from django.forms import widgets
class BookForm(forms.Form):
    date = forms.DateField(widget=widgets.TextInput(attrs={'type':'date'}))  #必须指定type，不然不能渲染成选择时间的input框
```

### 字段校验

#### RegexValidator验证器

```python
from django.forms import Form
from django.forms import widgets
from django.forms import fields
from django.core.validators import RegexValidator
 
class MyForm(Form):
    """
    通过正则表达式验证手机号是否是以159开头的
    """
    user = fields.CharField(
        validators=[RegexValidator(r'^[0-9]+$', '请输入数字'), RegexValidator(r'^159[0-9]+$', '数字必须以159开头')],
    )
```

#### 自定义验证函数

```python
import re
from django.forms import Form
from django.forms import widgets
from django.forms import fields
from django.core.exceptions import ValidationError
 
 
# 自定义验证规则
def mobile_validate(value):
    mobile_re = re.compile(r'^(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$')
    if not mobile_re.match(value):
        raise ValidationError('手机号码格式错误')  #自定义验证规则的时候，如果不符合你的规则，需要自己发起错误
 
 
class PublishForm(Form):
    title = fields.CharField(max_length=20,
                            min_length=5,
                            error_messages={'required': '标题不能为空','min_length': '标题最少为5个字符','max_length': '标题最多为20个字符'},
							widget=widgets.TextInput(attrs={'class': "form-control",'placeholder': '标题5-20个字符'}))
 
 
    # 使用自定义验证规则
    phone = fields.CharField(validators=[mobile_validate, ],
                            error_messages={'required': '手机不能为空'},
                            widget=widgets.TextInput(attrs={'class': "form-control",
'placeholder': u'手机号码'}))
 
    email = fields.EmailField(required=False,
                            error_messages={'required': u'邮箱不能为空','invalid': u'邮箱格式错误'},
                            widget=widgets.TextInput(attrs={'class': "form-control", 'placeholder': u'邮箱'}))
```

### Hook钩子方法

除了上面两种方式，我们还可以在Form类中定义钩子函数，来实现自定义的验证功能。

#### 局部钩子

我们在Fom类中定义 clean_字段名() 方法，就能够实现对特定字段进行校验。

```python
class LoginForm(forms.Form):
    username = forms.CharField(
        min_length=8,
        label="用户名",
        initial="张三",
        error_messages={
            "required": "不能为空",
            "invalid": "格式错误",
            "min_length": "用户名最短8位"
        },
        widget=forms.widgets.TextInput(attrs={"class": "form-control"})
    )
    ...
    # 定义局部钩子，用来校验username字段,之前的校验股则还在，给你提供了一个添加一些校验功能的钩子
    def clean_username(self):
        value = self.cleaned_data.get("username")
        if "666" in value:
            raise ValidationError("光喊666是不行的")
        else:
            return value
```

#### 全局钩子

我们在Fom类中定义 clean() 方法，就能够实现对字段进行全局校验，字段全部验证完，局部钩子也全部执行完之后，执行这个全局钩子校验。

```python
class LoginForm(forms.Form):
    ...
    password = forms.CharField(
        min_length=6,
        label="密码",
        widget=forms.widgets.PasswordInput(attrs={'class': 'form-control'}, render_value=True)
    )
    re_password = forms.CharField(
        min_length=6,
        label="确认密码",
        widget=forms.widgets.PasswordInput(attrs={'class': 'form-control'}, render_value=True)
    )
    ...
    # 定义全局的钩子，用来校验密码和确认密码字段是否相同，执行全局钩子的时候，cleaned_data里面肯定是有了通过前面验证的所有数据
    def clean(self):
        password_value = self.cleaned_data.get('password')
        re_password_value = self.cleaned_data.get('re_password')
        if password_value == re_password_value:
            return self.cleaned_data #全局钩子要返回所有的数据
        else:
            self.add_error('re_password', '两次密码不一致') #在re_password这个字段的错误列表中加上一个错误，并且clean_data里面会自动清除这个re_password的值，所以打印clean_data的时候会看不到它
            raise ValidationError('两次密码不一致')
```

### 表单的Widgets

不要将Widgets和表单的fields字段混淆。表单字段负责验证输入并直接在模板中使用。而Widgets负责渲染网页上HML表单的输入元素和提取提交的原始数据widget时字段的一个内在属性，用于定义字段在浏览器的页面里以任何HTML元素展现。

#### 指定使用的widget

每个字段都有一个默认的widget类型。如果我们想要使用一个不同的Widget，可以在定义字段时使用widget参数。

```python
from django import forms

class CommentForm(forms.Form):
    name = forms.CharField()
    url = forms.URLField()
    comment = forms.CharField(widget=forms.Textarea)
```

这将使用一个Textarea Widget来战术表单的评论字段，而不是默认的Textinput Widget类型。

#### 设置widget的参数

许多widget具有可选的额外参数：

```python
from django import forms

BIRTH_YEAR_CHOICES = ('1980', '1981', '1982')
FAVORITE_COLORS_CHOICES = (
    ('blue', 'Blue'),
    ('green', 'Green'),
    ('black', 'Black'),
)

class SimpleForm(forms.Form):
    birth_year = forms.DateField(widget=forms.SelectDateWidget(years=BIRTH_YEAR_CHOICES))
    favorite_colors = forms.MultipleChoiceField(
        required=False,
        widget=forms.CheckboxSelectMultiple,
        choices=FAVORITE_COLORS_CHOICES,
    )
```

#### 为widget添加CSS样式

默认情况下，当Django渲染Widget为实际的HTML代码时，不会帮你添加任何的CSS样式，也就是说网页上所有的TextInput元素的外观是一样的。

```python
class CommentForm(forms.Form):
    name = forms.CharField(widget=forms.TextInput(attrs={'class': 'special'}))
    url = forms.URLField()
    comment = forms.CharField(widget=forms.TextInput(attrs={'size': '40'}))
```

#### 批量添加样式

可通过重写form类的init方法来实现。

```python
class LoginForm(forms.Form):
    username = forms.CharField(
        min_length=8,
        label="用户名",
        initial="张三",
        error_messages={
            "required": "不能为空",
            "invalid": "格式错误",
            "min_length": "用户名最短8位"
        }
    ...

    def __init__(self, *args, **kwargs):
        super(LoginForm, self).__init__(*args, **kwargs)
        for field in iter(self.fields):
            self.fields[field].widget.attrs.update({
                'class': 'form-control'
            })
```

### form表单总结

流程：

```
1，新建forms.py文件，我们在里面写我们的校验的字段（注意，为了后续的数据的持久化储存，这里验证的字段最好和models里面定义的字段的名称保持一致）我们创建好需要检验的字段之后，我们可以先给他们添加简单的限定条件，比如required=True，限定字段的最大长度等等
```

```
2，我们为了方便，创建的子字段也可以直接生成html的标签直接插入到html页面中，这个时候，我们就需要给标签加各种的样式了，我们可以使用widget给每个字段添加输入的数据类型，以及它的参数attr可以给标签加各种的样式，也可以是Bootstrap的样式；有一些样式每个标签是公有的，我们就可以使用__init__方法给他们统一添加样式
```

```
3，我们创建好这些标签后，毕竟不是真正的标签，我们还需要把他们放到页面去，才能真正的让这些标签生效，那么我们，在views里面对应的路由中，直接初始化form=forms.Users()对象(注意：在我们开始把标签放到页面的时候我们实例化时候是没有参数的，千万不要入坑)，把form作为参数，传递给我们的前端页面。现在我们就可以在前端{{ form}}就可以拿到我们的form为我们生成的网页标签
```

```
4，这个时候，我们就该去做字段的错误信息了error_message，只要我们能够想到限定以及可能出现的错误，我们都可以在这里定义。方法很多，简单的就是直接在字段里面直接error_message = {定义可能出错的信息},也可以定义验证器，函数等方法。但是使用最多的还是‘钩子函数’;我们通过定义全局钩子（作用于所有的标签），局部钩子(针对单个字段)进行字段的校验（注意：局部钩子有一个坑，就是我们在为一个字典定义了验证的格式之后，并不光需要添加错误的信息，可要返回正确的cleaned_data给我们的form对象，不然我们后台是接收不到前台发来的数据的）。
```

```
5，既然我们定义了错误信息，那么前端页面也要看到。但是他们输入了错误的格式，怎么展示错误信息呢？我们在后台很巧妙的使用is_valid()方法来进行字段的校验，通过校验，我们就换回正确的页面。否者我们通过form=forms.User(request.POST)的方式把前台返回的时候重新生成一个form对象，然后再次返回给前台页面，这个时候的form对象就携带了错误信息，然后我们可以在前台使用{{form.errors.0}}展示错误信息
```

### 模型表单ModelForm

　通常在Django项目中，我们编写的大部分都是与Django 的模型紧密映射的表单。 举个例子，你也许会有个Book 模型，并且你还想创建一个form表单用来添加和编辑书籍信息到这个模型中。 在这种情况下，在form表单中定义字段将是冗余的，因为我们已经在模型中定义了那些字段。我们的ModelForm可以卸载forms.py文件里面，然后在 views.py文件里面导入

#### 核心用法

基于这个原因，Django提供一个辅助我们利用Django的ORM模型model创建Form。

```python
from django.forms import ModelForm
from myapp.models import Article  #数据库创建的表

# 创建表单类
class ArticleForm(ModelForm):
     class Meta:
         model = Article
         fields = ['pub_date', 'headline', 'content', 'reporter']

# 创建一个表单，用于添加文章
form = ArticleForm()

# 创建表单修改已有的文章
article = Article.objects.get(pk=1)
form = ArticleForm(instance=article)
```

用法核心：

​	1，首先从django.forms导入ModelForm；

​	2，编写一个自己的类，继承ModelForm；

​	3，在新类里，设置元类Meta；

​	4，在Meta中，设置model属性为你要关联的ORM模型，这里是Article；

​	5，在Meta中，设置fields属性为你要在表单中使用的字段列表；

​	6，列表里的值，应该是ORM模型model中的字段名。

上面的例子中，因为model和form比较简单，字段数量少，看不出这么做的威力和效率。但如果是那种大型项目，每个模型的字段数量几十上百，这么做的收益将非常巨大，而且后面还有一招提高效率的大杀器，也就是一步保存数据的操作。

#### 字段类型

和model模型字段存在大量重复的，详细参见：http://www.liujiangblog.com/course/django/156

#### ModelForm的验证

两步走：

- 表单验证
- 验证模型实例

与普通的表单验证类似，模型表单的验证也是调用`is_valid()`方法或访问errors属性。**我们可以像使用Form类一样自定义局部钩子或者全局钩子的方法来实现自定义的校验规则**，如果我们不重复字段并设置validators属性的话，ModelForm是按照模型中字段的validators来校验的。

**save()方法**

每个ModelForm模型都具有一个save()方法，这个方法根据表单绑定的数据创建并保存数据库对象。ModelForm的子类可以接收现有的模型实例作为关键字参数instance；如果提供此功能，则save()将更新该实例。

之前我们通过form组件来保存书籍数据的时候的写法是这样的：

```python
def edit_book(request,n):

    book_obj = models.Book.objects.filter(pk=n).first()
    if request.method == 'GET':
        all_authors = models.Author.objects.all() #
        all_publish = models.Publish.objects.all()

        return render(request,'edit_book.html',{'book_obj':book_obj,'all_authors':all_authors,'all_publish':all_publish})
```

现在views.py是这样写的：

```python
def edit_book(request,n):

    book_obj = models.Book.objects.filter(pk=n).first()
    if request.method == 'GET':
        # all_authors = models.Author.objects.all() #
        # all_publish = models.Publish.objects.all()

        form = BookForm(instance=book_obj)

        return render(request,'edit_book.html',{'form':form,'n':n}) #传递的这个n参数是给form表单提交数据的是的action的url用的，因为它需要一个参数来识别是更新的哪条记录

    else:
        form = BookForm(request.POST,instance=book_obj) #必须指定instance，不然我们调用save方法的是又变成了添加操作
        if form.is_valid():
            form.save()
            return redirect('show')
        else:
            return render(request,'edit_book.html',{'form':form,'n':n})
```

之前的html文件时这样写的：

```python
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="{% static 'bootstrap-3.3.0-dist/dist/css/bootstrap.min.css' %}">
</head>
<body>

<h1>编辑页面</h1>
<div class="container-fluid">
    <div class="row">
        <div class="col-md-6 col-md-offset-3">
            <form action="">
                <div class="form-group">
                    <label for="title">书名</label>
                    <input type="text" class="form-control" id="title" placeholder="title" value="{{ book_obj.title }}">

                </div>
                <div class="form-group">
                    <label for="publishDate">出版日期</label>
                    <input type="text" class="form-control" id="publishDate" placeholder="publishDate" value="{{ book_obj.publishDate|date:'Y-m-d' }}">

                </div>
                <div class="form-group">
                    <label for="price">价格</label>
                    <input type="number" class="form-control" id="price" placeholder="price" value="{{ book_obj.price }}">

                </div>
                <div class="form-group">
                    <label for="publish">书名</label>
                    <select name="publish" id="publish" class="form-control">
                        {% for publish in all_publish %}
                                {% if publish == book_obj.publish %}
                                    <option value="{{ publish.id }}" selected>{{ publish.name }}</option>
                                {% else %}
                                    <option value="{{ publish.id }}">{{ publish.name }}</option>
                                {% endif %}
                        {% endfor %}

                    </select>

                </div>
                <div class="form-group">
                    <label for="authors">书名</label>
                    <select name="authors" id="authors" multiple class="form-control">
                        {% for author in all_authors %}
                            {% if author in book_obj.authors.all %}
                                <option value="{{ author.id }}" selected>{{ author.name }}</option>
                            {% else %}
                                 <option value="{{ author.id }}" >{{ author.name }}</option>
                            {% endif %}
                        {% endfor %}

                    </select>

                </div>
            </form>

        </div>
    </div>
</div>


</body>
<script src="{% static 'bootstrap-3.3.0-dist/dist/jQuery/jquery-3.1.1.js' %}"></script>
<script src="{% static 'bootstrap-3.3.0-dist/dist/js/bootstrap.min.js' %}"></script>
</html>
```

现在我们的html文件是这样写的：

```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="{% static 'bootstrap-3.3.0-dist/dist/css/bootstrap.min.css' %}">
</head>
<body>

<h1>编辑页面</h1>
<div class="container-fluid">
    <div class="row">
        <div class="col-md-6 col-md-offset-3">
            <form action="{% url 'edit_book' n %}" novalidate method="post">
                {% csrf_token %}
                {% for field in form %}
                    <div class="form-group">
                        <label for="{{ field.id_for_label }}">{{ field.label }}</label>
                        {{ field }}
                        <span class="text-danger">{{ field.errors.0 }}</span>
                    </div>
                {% endfor %}

                <div class="form-group">
                    <input type="submit" class="btn btn-primary pull-right">
                </div>

            </form>

        </div>
    </div>
</div>


</body>
<script src="{% static 'bootstrap-3.3.0-dist/dist/jQuery/jquery-3.1.1.js' %}"></script>
<script src="{% static 'bootstrap-3.3.0-dist/dist/js/bootstrap.min.js' %}"></script>
</html>
```

#### ModelForm的字段选择

强烈建议使用ModelForm的fields属性，在赋值的列表内，一个一个将要使用的字段添加进去。这样做的好处是，安全可靠。

然而，有时候，字段太多，或者我们想偷懒，不愿意一个一个输入，也有简单的方法：

**__all__**:

将fields属性的值设为`__all__`，表示将映射的模型中的全部字段都添加到表单类中来。

```python
from django.forms import ModelForm

class AuthorForm(ModelForm):
    class Meta:
        model = Author
        fields = '__all__'
```

**exclude属性**:

表示将model中，除了exclude属性中列出的字段之外的所有字段，添加到表单类中作为表单字段。

```python
class PartialAuthorForm(ModelForm):
    class Meta:
        model = Author
        exclude = ['title']
```

因为Author模型有3个字段name、`birth_date`和title，上面的例子会让`birth_date`和name出现在表单中。

#### class META下常用的参数

```python
model = models.Book  # 对应的Model中的类
fields = "__all__"  # 字段，如果是__all__,就是表示列出所有的字段
fields = ["字段一","字段二"...]   #展示自定字段
exclude = None  # 排除的字段
labels = None  # 提示信息
help_texts = None  # 帮助提示信息
widgets = None  # 自定义插件
error_messages = None  # 自定义错误信息

error_messages = {
    'title':{'required':'不能为空',...} #每个字段的所有的错误都可以写，...是省略的意思，复制黏贴我代码的时候别忘了删了...
}
```

#### 完整的实例

前言：我们做表单的重点不是去创建标签，而是去做数据合法性的验证，并能正确的接收前端传过来的数据。

**创建ModelForm**

```python
#首先导入ModelForm

from django.forms import ModelForm
#在视图函数中，定义一个类，比如就叫StudentList，这个类要继承ModelForm，在这个类中再写一个原类Meta（规定写法，并注意首字母是大写的）
#在这个原类中，有以下属性（部分）：

class StudentList(ModelForm):
    class Meta:
        model =Student #对应的Model中的类
        fields = "__all__" #字段，如果是__all__,就是表示列出所有的字段
        exclude = None #排除的字段
        #error_messages用法：
        error_messages = {
        'name':{'required':"用户名不能为空",},
        'age':{'required':"年龄不能为空",},
        }
        #widgets用法,比如把输入用户名的input框给为Textarea
        #首先得导入模块
        from django.forms import widgets as wid #因为重名，所以起个别名
        widgets = {
        "name":wid.Textarea(attrs={"class":"c1"}) #还可以自定义属性
        }
        #labels，自定义在前端显示的名字
        labels= {
        "name":"用户名"
        }
```

**在url对应的视图里面实例化这个类**

```python
def student(request):

    if request.method == 'GET':
        student_list = StudentList()
        return render(request,'student.html',{'student_list':student_list})
    
然后在前端，我们之间{{ student_list.as_p }}一下，所有的字段就都出来了，可以循环，也可以使用还是那个面的方法
```

**在前端页面循环展示数据**

```html
<body>
<div class="container">
    <h1>student</h1>
    <form method="POST" novalidate>
        {% csrf_token %}
        {# {{ student_list.as_p }}#}
        {% for student in student_list %}
            <div class="form-group col-md-6">
                {# 拿到数据字段的verbose_name,没有就默认显示字段名 #}
                <label class="col-md-3 control-label">{{ student.label }}</label>
                <div class="col-md-9" style="position: relative;">{{ student }}</div>
            </div>
        {% endfor %}
        <div class="col-md-2 col-md-offset-10">
            <input type="submit" value="提交" class="btn-primary">
        </div>
    </form>
</div>
</body>
```

**添加纪录**

注意：保存数据的时候，不用一个个的取数据了，只需要save一下：

```python
def student(request):

    if request.method == 'GET':
         student_list = StudentList()
         return render(request,'student.html',{'student_list':student_list})
    else:
         student_list = StudentList(request.POST)
         if student_list.is_valid():
         student_list.save()
         return redirect(request,'student_list.html',{'student_list':student_list})
```

**编辑数据**

如果我们不适用ModellForm，编辑的时候显示之前的数据，是不是还需要一个个的取出来，但是使用这个模块快，我们只需要instance=obj(obj是修改的数据库的一条数据的对象)，就可以得到同样的效果。

注意：保存的时候需要注意一下，一定要有这个对象（instance=obj

），否则不知道个更新哪一个数据

```python
from django.shortcuts import render,HttpResponse,redirect
from django.forms import ModelForm
# Create your views here.
from app01 import models
def test(request):
    # model_form = models.Student
    model_form = models.Student.objects.all()
    return render(request,'test.html',{'model_form':model_form})

class StudentList(ModelForm):
    class Meta:
        model = models.Student #对应的Model中的类
        fields = "__all__" #字段，如果是__all__,就是表示列出所有的字段
        exclude = None #排除的字段
        labels = None #提示信息
        help_texts = None #帮助提示信息
        widgets = None #自定义插件
        error_messages = None #自定义错误信息
        #error_messages用法：
        error_messages = {
        'name':{'required':"用户名不能为空",},
        'age':{'required':"年龄不能为空",},
        }
        #widgets用法,比如把输入用户名的input框给为Textarea
        #首先得导入模块
        from django.forms import widgets as wid #因为重名，所以起个别名
        widgets = {
        "name":wid.Textarea
        }
        #labels，自定义在前端显示的名字
        labels= {
        "name":"用户名"
        }
def student(request):
    if request.method == 'GET':
        student_list = StudentList()
        return render(request,'student.html',{'student_list':student_list})
    else:
        student_list = StudentList(request.POST)
        if student_list.is_valid():
            student_list.save()
            return render(request,'student.html',{'student_list':student_list})

def student_edit(request,pk):
    obj = models.Student.objects.filter(pk=pk).first()
    if not obj:
        return redirect('test')
    if request.method == "GET":
        student_list = StudentList(instance=obj)
        return render(request,'student_edit.html',{'student_list':student_list})
    else:
        student_list = StudentList(request.POST,instance=obj)
        if student_list.is_valid():
            student_list.save()
            return render(request,'student_edit.html',{'student_list':student_list})
```

**总结：从上边可以看到ModelForm用起来是非常方便的，比如增加修改之类的操作。但是也带来额外不好的地方，model和form之间耦合了。如果不耦合的话，mf.save()方法也无法直接提交保存。 但是耦合的话使用场景通常局限用于小程序，写大程序就最好不用了。**

#### 自定义ModelForm字段

#### 表单的继承

#### 提供初始值

不想写，详细参见：http://www.liujiangblog.com/course/django/156