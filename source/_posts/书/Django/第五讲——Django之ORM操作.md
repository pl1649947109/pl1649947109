---
title: 第五讲——Django之ORM操作
id: 5
date: 2019-9-28 20:00:00
tags: Django
toc: true
comment: true
---

 API的使用快速查找：https://www.django.cn/article/show-15.html

### ORM的由来

如果你有很多的数据库操作，并且你的Python程序员不是专业的DBA，写的SQL语句很烂，甚至经常写错，怎么办？

聪明的人想出了一个办法：用Python语法来写，然后使用一个中间工具将Python代码翻译成原生的SQL语句，这样你总不会写错了吧？这个中间工具就是所谓的ORM（对象关系映射）！

ORM将一个Python的对象映射为数据库中的一张关系表。它将SQL封装起来，程序员不再需要关心数据库的具体操作，只需要专注于自己本身代码和业务逻辑的实现。

![](http://9017499461.linshutu.top/orm%E7%94%B1%E6%9D%A5.png)

### ORM介绍

- MVC或者MVC框架中包括一个重要的部分，就是ORM，它实现了数据模型与数据库的解耦，即数据模型的设计不需要依赖于特定的数据库，通过简单的配置就可以轻松更换数据库，这极大的减轻了开发人员的工作量，不需要面对因数据库变更而导致的无效劳动
- ORM是“对象-关系-映射”的简称。（Object Relational Mapping，简称ORM）(将来会学一个sqlalchemy，是和他很像的，但是django的orm没有独立出来让别人去使用，虽然功能比sqlalchemy更强大，但是别人用不了)
- 类对象--->sql--->pymysql--->mysql服务端--->磁盘，orm其实就是将类对象的语法翻译成sql语句的一个引擎，明白orm是什么了，剩下的就是怎么使用orm，怎么来写类对象关系语句。
- **一个Python的类，就是一个模型，代表数据库中的一张数据表！Django奉行Python优先的原则，一切基于Python代码的交流，完全封装SQL内部细节。**

先上图：

![](http://9017499461.linshutu.top/dj_ORM.png)

<!----more---->

**orm与关系型数据库对应关系**

| 关系型数据库 |     ORM      |
| :----------: | :----------: |
|      表      |      类      |
|     字段     |    类属性    |
|    表记录    | 类实例化对象 |

**原生的sql和orm代码对比**

```python
#sql中的表                                                      
 #创建表:
     CREATE TABLE employee(                             
                id INT PRIMARY KEY auto_increment ,     
                name VARCHAR (20),                     
                gender BIT default 1,                   
                birthday DATA ,                         
                department VARCHAR (20),               
                salary DECIMAL (8,2) unsigned,         
              );
                               
#添加一条表纪录:                                       
      INSERT employee 				       
    (name,gender,birthday,salary,department) 
    VALUES("alex",1,"1985-12-12",8000,"保洁部");         
#查询一条表纪录:                                       
      SELECT * FROM employee WHERE age=24;             
#更新一条表纪录:                                         
      UPDATE employee SET birthday="1989-10-24" WHERE id=1;              
#删除一条表纪录:                                        
      DELETE FROM employee WHERE name="alex" 

#python的类
class Employee(models.Model):
     id=models.AutoField(primary_key=True)
     name=models.CharField(max_length=32)
     gender=models.BooleanField()
     birthday=models.DateField()
     department=models.CharField(max_length=32)
     salary=models.DecimalField(max_digits=8,decimal_places=2)

#添加一条表纪录:
emp=Employee(name="alex",gender=True,birthday="1985-12-12",epartment="保洁部")
emp.save()
#查询一条表纪录:
Employee.objects.filter(age=24)
#更新一条表纪录:
Employee.objects.filter(id=1).update(birthday="1989-10-24")
#删除一条表纪录:
Employee.objects.filter(name="alex").delete()
```

### 连接数据库

配置settings.py

```python
DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',  #修改成mysql
            'NAME': 'orm02',    #修改到自己的数据库
            'USER':'root',    #用户名
            'PASSWORD':'666',    #密码
            'HOST':'127.0.0.1',
            'PORT':3306,
        }
    }
#在修改settings文件时，请顺便将TIME_ZONE设置为国内所在的时区Asia/Shanghai
```

配置项目目录下的__init__文件

```python
import pymysql
pymysql.install_as_MySQLdb()    #用pymysql替换默认的mysqldb
```

创建类之后写到数据库

```python
注意：执行数据库同步指令,添加字段的时候别忘了,该字段不能为空,所有要么给默认值,要么设置它允许为空 null=True
python manage.py makemigrations  #生成表结构变化文件
python manage.py migrate   #根据变化文件，创建表

也许大家会问：为什么把数据迁移到数据数需要借助这个makemigrations命令呢？答：之所一要将创建和实施迁移的动作分为两个命令两步走是因为我们可以要通过版本控制系统(git,svn)提交我们的代码，日过没有这个中间过程保存我见，那么github如何直达以及记录、同步、实施我们所进行的模型修改动作呢？毕竟，github不和数据库直接打交道，也没法和我们本地的数据库进行通信。因此分开之后，我们只需要将我们的文件上传到github，它就会知道一切。

### settings.py文件

'''
默认情况，INSTALLED_APPS中会自动包含下列条目，它们都是Django自动生成的：

django.contrib.admin：admin管理后台站点
django.contrib.auth：身份认证系统
django.contrib.contenttypes：内容类型框架
django.contrib.sessions：会话框架
django.contrib.messages：消息框架
django.contrib.staticfiles：静态文件管理框架
上面的一些应用也需要建立一些数据库表，对于极简主义者，你完全可以在INSTALLED_APPS内注释掉任何或者全部的Django提供的通用应用。这样，migrate也不会再创建对应的数据表。
''' 
```

### 模型和字段

基本原则：

- 每个模型在Django中存在形式为一个Python类
- 每个模型都是django.db.models.Model的子类
- 模型的每个字段(属性)代表数据表的某一列
- Django将自动为我们生成数据库访问API

**来个对比**

```python
#python类对象创建
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
#上面的代码原生语句
CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(30) NOT NULL
);

#注意：
1.myapp_person是Django自动为我们生成的，默认格式就是项目名称+小写类名，我们可以重写这个规则
2.Django默认自动创建自动自增主键id
3.上面的SQL语句基于PostgreSQL语法，因此从侧面看orm支持很多数据库的语法
```

#### 常用非关系型字段类型

字段类型的作用：

- 决定数据库中对应的数据类型
- HTML中对应的表单标签的类型
- 在admin后台和自动生成的表单最小的数据验证需求

常用的内置字段类型（采用驼峰命名法）

| 类型                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| **AutoField**              | 一个自动增加的整数类型字段。通常你不需要自己编写它，Django会自动帮你添加字段：`id = models.AutoField(primary_key=True)`，这是一个自增字段，从1开始计数。如果你非要自己设置主键，那么请务必将字段设置为`primary_key=True`。**Django在一个模型中只允许有一个自增字段，并且该字段必须为主键！** |
| BigAutoField               | (1.10新增)64位整数类型自增字段，数字范围更大，从1到9223372036854775807 |
| BigIntegerField            | 64位整数字段（看清楚，非自增），类似IntegerField ，-9223372036854775808 到9223372036854775807。在Django的模板表单里体现为一个textinput标签。 |
| BinaryField                | 二进制数据类型。使用受限，少用。                             |
| **BooleanField**           | 布尔值类型。默认值是None。**在HTML表单中体现为CheckboxInput标签**。如果要接收null值，请使用NullBooleanField。 |
| **CharField**              | 字符串类型。**必须接收一个max_length参数，表示字符串长度不能超过该值。**默认的表单标签是input text。最常用的filed，没有之一！ |
| CommaSeparatedIntegerField | 逗号分隔的整数类型。必须接收一个max_length参数。常用于表示较大的金额数目，例如1,000,000元。 |
| **DateField**              | `class DateField(auto_now=False, auto_now_add=False, **options)`日期类型。**一个Python中的datetime.date的实例。在HTML中表现为TextInput标签。在admin后台中，Django会帮你自动添加一个JS的日历表和一个“Today”快捷方式，以及附加的日期合法性验证。**两个重要参数：（参数互斥，不能共存） `auto_now`:每当对象被保存时将字段设为当前日期，常用于保存最后修改时间。`auto_now_add`：每当对象被创建时，设为当前日期，常用于保存创建日期(注意，它是不可修改的)。设置上面两个参数就相当于给field添加了`editable=False`和`blank=True`属性。如果想具有修改属性，请用default参数。例子：`pub_time = models.DateField(auto_now_add=True)`，自动添加发布时间。 |
| DateTimeField              | 日期时间类型。Python的datetime.datetime的实例。与DateField相比就是多了小时、分和秒的显示，其它功能、参数、用法、默认值等等都一样。 |
| DecimalField               | 固定精度的十进制小数。**相当于Python的Decimal实例，必须提供两个指定的参数！**参数`max_digits`：最大的位数，必须大于或等于小数点位数 。`decimal_places`：小数点位数，精度。 当`localize=False`时，它在HTML表现为NumberInput标签，否则是text类型。例子：储存最大不超过999，带有2位小数位精度的数，定义如下：`models.DecimalField(..., max_digits=5, decimal_places=2)`。 |
| DurationField              | 持续时间类型。存储一定期间的时间长度。类似Python中的timedelta。在不同的数据库实现中有不同的表示方法。常用于进行时间之间的加减运算。但是小心了，这里有坑，PostgreSQL等数据库之间有兼容性问题！ |
| **EmailField**             | 邮箱类型，默认max_length最大长度254位。使用这个字段的好处是，**可以使用DJango内置的EmailValidator进行邮箱地址合法性验证。** |
| **FileField**              | `class FileField(upload_to=None, max_length=100, **options)`上传文件类型，后面单独介绍1。 |
| FilePathField              | 文件路径类型，后面单独介绍2                                  |
| FloatField                 | 浮点数类型，参考整数类型                                     |
| **ImageField**             | 图像类型，后面单独介绍3。                                    |
| **IntegerField**           | **整数类型，最常用的字段之一。取值范围-2147483648到2147483647。在HTML中表现为NumberInput标签。** |
| **GenericIPAddressField**  | `class GenericIPAddressField(protocol='both', unpack_ipv4=False, **options)[source]`,**IPV4或者IPV6地址，字符串形式，例如`192.0.2.30`或者`2a02:42fe::4`在HTML中表现为TextInput标签。参数`protocol`默认值为‘both’，可选‘IPv4’或者‘IPv6’，表示你的IP地址类型。** |
| NullBooleanField           | 类似布尔字段，只不过额外允许`NULL`作为选项之一。             |
| PositiveIntegerField       | 正整数字段，包含0,最大2147483647。                           |
| PositiveSmallIntegerField  | 较小的正整数字段，从0到32767。                               |
| SmallIntegerField          | 小整数，包含-32768到32767。                                  |
| **TextField**              | **大量文本内容，在HTML中表现为Textarea标签，最常用的字段类型之一！如果你为它设置一个max_length参数，那么在前端页面中会受到输入字符数量限制，然而在模型和数据库层面却不受影响。只有CharField才能同时作用于两者。** |
| TimeField                  | **时间字段，Python中datetime.time的实**例。接收同DateField一样的参数，只作用于小时、分和秒。 |
| **URLField**               | 一个用于保存URL地址的字符串类型，默认最大长度200。           |
| **UUIDField**              | **用于保存通用唯一识别码（Universally Unique Identifier）的字段。**使用Python的UUID类。在PostgreSQL数据库中保存为uuid类型，其它数据库中为char(32)。这个字段是自增主键的最佳替代品，后面有例子展示4。 |

详细解析部分扩展

1.**FileField**：文件上传

格式：

```python
class FileField(upload_to=None, max_length=100, **options)[source]
#说明：
上传文件字段（不能设置为主键）。默认情况下，该字段在HTML中表现为一个ClearableFileInput标签。在数据库内，我们实际保存的是一个字符串类型，默认最大长度100，可以通过max_length参数自定义。真实的文件是保存在服务器的文件系统内的。
#参数解析
upload_to：用于设置上传地址的目录和文件名；也可以接收一个回调函数，该函数返回具体的路径字符串

#实例
1.upload_to用于设置上传地址的目录和文件名
class MyModel(models.Model):
    # 文件被传至MEDIA_ROOT/uploads目录，MEDIA_ROOT由你在settings文件中设置
    upload = models.FileField(upload_to='uploads/')
    
2.upload_to参数也可以接收一个回调函数，该函数返回具体的路径字符串
def user_directory_path(instance, filename):
    #文件上传到MEDIA_ROOT/user_<id>/<filename>目录中
    return 'user_{0}/{1}'.format(instance.user.id, filename)

class MyModel(models.Model):
    upload = models.FileField(upload_to=user_directory_path)
'''
注意：user_directory_path这种回调函数，必须接收两个参数，然后返回一个Unix风格的路径字符串。参数instace代表一个定义了FileField的模型的实例，说白了就是当前数据记录。filename是原本的文件名。
'''
```

2.**ImageField**：图片保存

格式：

```python
class ImageField(upload_to=None, height_field=None, width_field=None, max_length=100, **options)[source]
#说明：
用于保存图像文件的字段。其基本用法和特性与FileField一样，只不过多了两个属性height和width。默认情况下，该字段在HTML中表现为一个ClearableFileInput标签。在数据库内，我们实际保存的是一个字符串类型，默认最大长度100，可以通过max_length参数自定义。真实的图片是保存在服务器的文件系统内的。
#参数说明
height_field：保存有图片高度信息的模型字段名。 
width_field：保存有图片宽度信息的模型字段名。
#注意
使用Django的ImageField需要提前安装pillow模块，pip install pillow即可。
```

**总结上面两个字段**

```python
#使用FileField或者ImageField字段的步骤：

1.在settings文件中，配置MEDIA_ROOT，作为你上传文件在服务器中的基本路径（为了性能考虑，这些文件不会被储存在数据库中）。再配置个MEDIA_URL，作为公用URL，指向上传文件的基本路径。请确保Web服务器的用户账号对该目录具有写的权限。

2.添加FileField或者ImageField字段到你的模型中，定义好upload_to参数，文件最终会放在MEDIA_ROOT目录的“upload_to”子目录中。

3.所有真正被保存在数据库中的，只是指向你上传文件路径的字符串而已。可以通过url属性，在Django的模板中方便的访问这些文件。例如，假设你有一个ImageField字段，名叫mug_shot，那么在Django模板的HTML文件中，可以使用{{ object.mug_shot.url }}来获取该文件。其中的object用你具体的对象名称代替。

4.可以通过name和size属性，获取文件的名称和大小信息。
```

**强烈的安全建议**

无论你如何保存上传的文件，一定要注意他们的内容和格式，避免安全漏洞！务必对所有的上传文件进行安全检查，确保它们不出问题！如果你不加任何检查就盲目的让任何人上传文件到你的服务器文档根目录内，比如上传了一个CGI或者PHP脚本，很可能就会被访问的用户执行，这具有致命的危害。

3.**FilePathField**：保存文件路径信息的字段

格式：

```python
class FilePathField(path=None, match=None, recursive=False, max_length=100, **options)[source]
#说明
一种用来保存文件路径信息的字段。在数据表内以字符串的形式存在，默认最大长度100，可以通过max_length参数设置。
#参数说明
path：必须指定的参数。表示一个系统绝对路径。

match:可选参数，一个正则表达式，用于过滤文件名。只匹配基本文件名，不匹配路径。例如foo.*\.txt$，只匹配文件名foo23.txt，不匹配bar.txt与foo23.png。

recursive:可选参数，只能是True或者False。默认为False。决定是否包含子目录，也就是是否递归的意思。

allow_files:可选参数，只能是True或者False。默认为True。决定是否应该将文件名包括在内。它和allow_folders其中，必须有一个为True。

allow_folders： 可选参数，只能是True或者False。默认为False。决定是否应该将目录名包括在内。
#实例
FilePathField(path="/home/images", match="foo.*", recursive=True)
```

4.**UUIDField**:自增主键的最佳替代品

```python
#数据库无法自己生成uuid，因此需要如下使用default参数：
import uuid     # Python的内置模块
from django.db import models

class MyUUIDModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    # 其它字段
```

#### 常用关系型字段类型

(这个在后面的表操作再去总结)

#### 字段的参数

|    字段的参数    |                             解释                             |
| :--------------: | :----------------------------------------------------------: |
|       null       | 该值为True时，Django在数据库用NULL保存空值。默认值为False。  |
|      blank       | True时，字段可以为空。默认False。和null参数不同的是，null是纯数据库层面的，而**blank是验证相关的，它与表单验证是否允许输入框内为空有关，与数据库无关**。所以要小心一个null为False，blank为True的字段接收到一个空值可能会出bug或异常。 |
|     choices      | 用于页面上的选择框标签，需要先提供一个二维的二元元组，第一个元素表示存在数据库内真实的值，第二个表示页面上显示的具体内容。在浏览器页面上将显示第二个元素的值。 |
|    db_column     | 该参数用于定义当前字段在数据表内的列名，如果没有指定，Django将使用字段名作为列名 |
|     db_index     |   该参数接收布尔值。如果为True，数据库将为该字段创建索引。   |
|  db_tablespace   |  用于字段索引的数据表空间的名字，前提是当前字段设置了索引。  |
|     default      | 字段的默认值，可以是值或者一个调用对象。如果是可调用对象，那么每次创建新对象的时候都会调用。**设置的默认值不能是一个可变对象，比如列表等** |
|     editable     | 如果设置为False，那么当前的字段将不会在admin后台或者其他的ModelForm表单中显示，同时还会被模型验证功能跳过，默认为True |
|  error_messages  | 用于自定义错误信息。参数接收字典类型的值，字典的键可以是`null`、 `blank`、 `invalid`、 `invalid_choice`、 `unique`和`unique_for_date`其中的一个。 |
|    help_text     | 额外显示在表单部件上的帮助文本。使用时注意转义为纯文本，防止脚本攻击。 |
|   primary_key    | 如果你没有给模型的任何字段设置这个参数为True，Django将自动创建一个AutoField自增字段，名为‘id’，并设置为主键。也就是`id = models.AutoField(primary_key=True)`。 |
|      unique      | 设为True时，在整个数据表内该字段的数据不可重复。**注意：对于ManyToManyField和OneToOneField关系类型，该参数无效。** |
| unique_for_date  | 日期唯一。有点类似联合所引，比如title的字段是“标题”,日期是“2019-8-9”,那么就不能出现标题和日期一样的数据。 |
| unique_for_month |                       同上，月份唯一。                       |
| unique_for_year  |                       同上，年份唯一。                       |
|   verbose_name   | 为字段设置一个人类可读，更加直观的别名。在多表关联的时候，它都是第一个参数，Django默认自动创建它，并将下划线转换成空格。 |
|    validators    |               **运行在该字段上的验证器的列表**               |

#### 模型的元数据Meta

模型的元数据，指的是“除了字段外的所有内容”，例如排序方式，数据库的表名，人类刻度的单数或者复数名等等。所有的这些都是非必须的，甚至元数据本身对模型也是非必须的。但是，我要说但是，有些元数据选项能给予你极大的帮助，在实际使用中具有重要的作用，是实际应用的‘必须’。

想在模型中增加元数据，方法很简单，在模型的类总添加一个类。名字是古帝国的Meta。然后在这个Meta类下面增加各种元数据选项或者说设置项。**每个模型都可以有自己的元数据类，每个元数据类也只对自己所在模型起作用。**

```python
#实例
from django.db import models

class Ox(models.Model):
    horn_length = models.IntegerField()
    
    class Meta:
        ordering = ["horn_length"]
        verbose_name_plural = "oxen"
#上面的例子中，我们为模型Ox增加了两个元数据‘ordering’和‘verbose_name_plural’，分别表示排序和复数名
```

|         参数          |                             解释                             |
| :-------------------: | :----------------------------------------------------------: |
|       abstract        | 如果`abstract=True`，那么模型会被认为是一个抽象模型。抽象模型本身不实际生成数据库表，而是作为其它模型的父类，被继承使用。 |
|       app_label       | 如果定义了模型的app没有在`INSTALLED_APPS`中注册，则必须通过此元选项声明它属于哪个app。 |
|   base_manager_name   | 自定义模型的`_base_manager`管理器的名字。模型管理器是Django为模型提供的API所在。 |
|     **db_table**      | 指定在数据库中，当前模型生成的数据表的表名。例如：db_table="my_friends" |
|     db_tablespace     | 自定义数据表空间的名字。默认值是工程的`DEFAULT_TABLESPACE`设置。 |
| default_manager_name  |         自定义模型的`_default_manager`管理器的名字。         |
| default_related_name  |          从一个模型反向关联设置有关系字段的源模型。          |
|     get_latest_by     | Django管理器给我们提供有latest()和earliest()方法，分别表示获取最近一个和最前一个数据对象。这就是他来实现的。实例：get_latest_by = "order_date"，按照日期的加入先后排序的 |
|        managed        | 该元数据默认值为True，表示Django将按照既定的规则，管理数据库表的生命周期。 |
| order_with_respect_to |      其用途是根据指定的字段进行排序，通常用于关系字段。      |
|     **ordering**      | 用于指定该模型生成的所有对象的排序方式，接收一个字段名组成的元祖或列表。**默认按升序排列，如果在字段名前加上字符“-”则表示按降序排列，如果使用字符问号“？”表示随机排列。**实例：ordering = ['-pub_date', 'author'] |
|      permission       |           该元数据用于当创建对象时增加额外的权限。           |
|  default_permission   | Django默认给所有的模型设置('add', 'change', 'delete')的权限，也就是增删改。你可以自定义这个选项，比如设置为一个空列表，表示你不需要默认的权限，但是这一操作必须在执行migrate命令之前。 |
|         proxy         |  如果设置了`proxy = True`，表示使用代理模式的模型继承方式。  |
| required_db_features  |                  声明模型依赖的数据库功能。                  |
|  required_db_vendor   | 声明模型支持的数据库。Django默认支持`sqlite, postgresql, mysql, oracle`。 |
|        indexes        |              接收一个应用在当前模型上的索引列表              |
|  **unique_together**  |                           下面详细                           |
|   **verbose_name**    | 最常用的元数据之一！用于设置模型对象的直观、人类可读的名称。可以用中文。实例：verbose_name = "披萨" |
|  verbose_name_plural  |                    英语有单数和复数形式。                    |
|         label         | 前面介绍的元数据都是可修改和设置的，但还有**两个只读的元数据，label就是其中之一。**label等同于`app_label.object_name`。例如`polls.Question`，polls是应用名，Question是模型名。 |
|      label_lower      |                  同上，不过是小写的模型名。                  |

**unique_together**解析：

这个元数据是非常重要的一个，它等同于数据库的联合唯一。

```python
#实例
unique_together = (('name','birth_day','address'),)
#这样，哪怕有两个在同一天出生的张伟，但他们的籍贯不同，也就是两个不同的用户。一旦三者都相同，则会被Django拒绝创建。这一元数据经常被用在admin后台，并且强制应用于数据库层面。
```

注意：联合唯一无法作用于普通的多对多字段。

### 单表操作

#### QuerySet对象

格式：

```python
class QuerySet(model=None, query=None, using=None)[source]
```

定义：

QuerySet类具有两个公有属性用于内省：

- ordered:如果QuerySet是排好序的则返回True，否则返回False
- db:如果现在执行，则返回使用的数据库

**QuerySet何时被提交？**

在内部，创建、过滤、切片和传递一个QuerySet不会真实操作数据库，在你对查询集提交之前，不会发生任何实际的数据库操作。

我们可以使用下面的方法对QuerySet提交查询操作：

- 迭代：QuerySet是可迭代的，在首次迭代查询集时执行实际的数据库查询。
- 切片：如果使用切片的”step“参数，Django 将执行数据库查询并返回一个列表。
- Pickling/缓存
- repr()
- len():当你对QuerySet调用len()时， 将提交数据库操作。
- list():对QuerySet调用list()将强制提交操作`entry_list = list(Entry.objects.all())`
- bool()

#### 创建表

models.py文件

```python
from django.db import models

class UserInfo(models.Model):
	id = models.AutoField(primary_key=True)
	name = models.CharField(max_length=20,unique=True)
	age = models.IntegerField()
```

#### 创建记录（增）

##### 单条数据插入

views.py文件

方式一：

```python
from django.db import models

def query(request):
    obj1 = models.UserInfo(
		id = 1,
        name = "pl",
        age = 18,
    )    #先创建对象
    obj1.save()     #插入数据库
```

方式二：（用这个）

```python
from django.db import models
def query(request):
    obj1 = models.UserInfo.objects.create(
		id = 1,
        name = "pl",
        age = 18,
    )
#这个UserInfo.objects就像是一个UserInfo表的管理器一样，提供了增删改查所有的方法
```

##### 批量数据插入

```python
#方法：bulk_create()
student_list = []
for i in range(10):
	info_obj = models.UserInfo(
		name = "pl"+str(i),
		age = 12+i,
	)
	student_list.append(info_obj)
models.UserInfo.objects.bulk_create(student_list)
```

#### 删除记录（删）

delete()方法的调用者可以是一个model对象，也可以是一个queryset集合。

```python
#首先了解：filter() --它读出来的是queryset类型，它的里面是一个个的model对象，也就是说这个列表里面包含的是一条条的记录。
models.UserInfo.objects.filter(id=3).delete()  #删除的是queryset对象，也就是查询出来的所有记录
models.UserInfo.objects.filter(id=3)[0].delete()    #删除的是queryset对象里面的元素，也就是单条记录
```

#### 修改记录（改）

方式一：

```python
from django.db import models
def query(request):
	models.UserInfo.objects.filter(id=1).update(
        name = "pl2",
        age = 20,
    )
#这个修改的定位必须准确到每一条数据,update只能是querset类型才能调用，model对象不能直接调用更新方法
```

方式二：

```python
ret = models.UserInfo.objects.filter(id=1)[0]
ret.name = "pl2"
ret.age = 20
ret.save()
```

**补充**：

```python
#update_or_create:有就更新，没有就创建 ，
#get_or_create:有就查询出来，没有就创建
obj,created = models.UserInfo.objects.update_or_create(
    user = user #查询筛选的条件
    defaults = {  #添加或者更新的数据
        "token":randeom_str,
    }
)
```

#### 查询记录(查)

##### 查询集API（也就是查询方法）

###### 返回QuerySet对象

**以下的方法都将返回一个新的QuerySets。**重点是加粗的几个API，其它的使用场景很少。

| 方法名                | 解释                                         |
| --------------------- | -------------------------------------------- |
| **all()**             | 获取所有的对象                               |
| **filter(****kwargs)  | 过滤查询对象。                               |
| **exclude(****kwargs) | 排除满足条件的对象                           |
| **annotate()**        | 使用聚合函数                                 |
| **order_by()**        | 对查询集进行排序                             |
| **reverse()**         | 反向排序                                     |
| **distinct()**        | 对查询集去重                                 |
| **values()**          | 返回包含对象具体值的字典的QuerySet           |
| **values_list()**     | 与values()类似，只是返回的是元组而不是字典。 |
| **none()**            | 创建空的查询集                               |
| **select_related()**  | 附带查询关联对象                             |
| dates()               | 根据日期获取查询集                           |
| datetimes()           | 根据时间获取查询集                           |
| union()               | 并集                                         |
| intersection()        | 交集                                         |
| difference()          | 差集                                         |
| prefetch_related()    | 预先查询                                     |
| extra()               | 附加SQL查询                                  |
| defer()               | 不加载指定字段                               |
| only()                | 只加载指定的字段                             |
| using()               | 选择数据库                                   |
| `select_for_update()` | 锁住选择的对象，直到事务结束。               |
| raw()                 | 执行原始的SQL查询                            |

###### 返回Model对象

**以下的方法不会返回QuerySets，但是作用非常强大，尤其是粗体显示的方法，需要背下来。**

| 方法名                 | 解释                             |
| ---------------------- | -------------------------------- |
| **get(****kwargs)      | 获取单个对象                     |
| **count()**            | 统计对象的个数                   |
| **create()**           | 创建对象，无需save()             |
| **get_or_create()**    | 查询对象，如果没有找到就新建对象 |
| **update_or_create()** | 更新对象，如果没有找到就创建对象 |
| **bulk_create()**      | 批量创建对象                     |
| **in_bulk()**          | 根据主键值的列表，批量返回对象   |
| **iterator()**         | 获取包含对象的迭代器             |
| **latest()**           | 获取最近的对象                   |
| **earliest()**         | 获取最早的对象                   |
| **first()**            | 获取第一个对象                   |
| **last()**             | 获取最后一个对象                 |
| **aggregate()**        | 聚合操作                         |
| **exists()**           | 判断queryset中是否有对象         |
| **update()**           | 批量更新对象                     |
| **delete()**           | 批量删除对象                     |
| **as_manager()**       | 获取管理器                       |

###### 上述API详讲(13个)

```python
# 1. all
# 查询所有数据对象
models.Student.objects.all()
```

```python
# 2. filter 
# 查询所有符合筛选条件的对象:条件可以是：参数，字典，Q
models.Student.objects.filter(name='张三')
```

```python
# 3. get 
# 查询符合筛选条件的对象，但返回值只能有一个，如果匹配到的对象个数不只一个的话，触发MultipleObjectsReturned异常；如果根据给出的参数匹配不到对象的话，触发DoesNotExist异常。
models.Student.objects.get(age=12)
```

**对比filter()和get()**：

get()返回的结果只有一个，filter()则是返回所有的结果；

get()返回的是Model对象而filter()返回的是QuerySet对象；

get()如果符合筛选条件的对象超过一个或者没有对象都会抛出错误，但是filter()发生这种错误。

```python
# 4. exclude 
# 返回一个新的QuerySet，它包含不满足给定的查找参数的对象。
# 如果有多个参数，参数之间为且的关系,条件可以是：参数，字典，Q
models.Student.objects.exclude(classes_id=2)
```

```python
# 5.  order_by（升序）
默认情况下，根据模型的Meta类中的ordering属性对QuerySet中的对象进行排序
# 对结果进行排序
models.Student.objects.order_by("classes_id", "age")

#降序排序：（在age的前面加一个-号）
models.Student.objects.order_by("classes_id", "-age")
```

```python
# 6. reverse order_by 为sql 中为ASC reverse sql为DESC
# 对结果进行排序
models.Student.objects.order_by("pk").reverse()

#多条件排序：
models.Student.objects.order_by("-pk"，"price")
在pk相等的情况下，按照price的升序进行排序

#如要获取QuerySet中最后五个元素，可以这样做：
models.Student.objects.reverse()[:5]
解释：这与Python直接使用负索引有点不一样。 Django不支持负索引，只能曲线救国。
```

```python
# 7. count
# 计数
models.Student.objects.filter(classes_id=2).count()
```

```python
# 8. first
# 返回第一条数据
models.Student.objects.first()
```

```python
# 9. last
# 返回最后一条数据
models.Student.objects.last()
```

```python
# 10. exists 
# 判断查询数据是否为空，有数据返回True，空则返回False
 models.Student.objects.filter(age=12).exists()
```

```python
# 11. values，QuerySet中的元素为字典
# 返回一个QuerySet，列表中每条数据为一个**字典**
models.Student.objects.values()
```

```python
# 12. values_list， QuerySet中的元素为元组
# 返回一个QuerySet，列表中每条数据为一个**元组**
models.Student.objects.values_list()
```

```python
# 13. distinct
# 对结果去重
models.Student.objects.filter(classes_id=1).values("classes_id").distinct()

all_books = models.Book.objects.all().distinct() #这样写是表示记录中所有的字段重复才叫重复，但是我们知道有主键的存在，所以不可能所有字段数据都重复

all_books=models.Book.objects.all().distinct('price')
#报错，不能在distinct里面加字段名称

all_books=models.Book.objects.all().values('price').distinct()
#<QuerySet [(Decimal('11.00'),),(Decimal('111.00'),), (Decimal('120.00'),), (Decimal('11111.00'),)]>

all_books=models.Book.objects.all().values_list('price').distinct()
#<QuerySet [{'price': Decimal('11.00')}, {'price': Decimal('111.00')}, {'price': Decimal('120.00')}, {'price': Decimal('11111.00')}]> 只能用于valuse和values_list进行去重

all_books=models.Book.objects.all().values_list('title','price').distinct() 
#title和price两个同时重复才算一条重复的记录
```

具体内容：

返回QuerySet：http://www.liujiangblog.com/course/django/130

不返回QuerySet：http://www.liujiangblog.com/course/django/131

##### 查询参数及聚合函数

###### 双下划线的模糊查询

|       方法        |                      说明                      |
| :---------------: | :--------------------------------------------: |
|     field__lt     |                      小于                      |
|     field__gt     |                      大于                      |
|    field__lte     |                    小于等于                    |
|    field__gte     |                    大于等于                    |
|     field__in     |              在列表中,代替多个or               |
|   field__range    |                 在范围内(1,3)                  |
|  field__contains  |         获取字段中包含指定字符串的数据         |
| field__icontains  |  获取字段中包含指定字符串的数据，大小写不敏感  |
| field__startwith  |        获取字段中以指定字符串开头的数据        |
| field__istartwith | 获取字段中以指定字符串开头的数据，大小写不敏感 |
|  field__endswith  |        获取字段中以指定字符串结尾的数据        |
| field__iendswith  | 获取字段中以指定字符串结尾的数据，大小写不敏感 |
|    field__date    |                    日期匹配                    |
|    filed__year    |                       年                       |
|   filed__month    |                       月                       |
|    filed__day     |                       日                       |
|   field_isnull    |                字段值为空的数据                |
|   field__exact    |                    精确匹配                    |
|   field__iexact   |             不区分大小写的精确匹配             |
|    field__time    |                      时间                      |
|    field__hour    |                       时                       |
|   field__minute   |                       分                       |
|   field__second   |                       秒                       |
|   field__regex    |              区分大小写的正则匹配              |
|   field__iregex   |             不区分大小写的正则匹配             |

###### 上述参数详讲(9个)

```python
#1. exact：精确匹配。 默认的查找类型！
Entry.objects.get(id__exact=14)
Entry.objects.get(id__exact=None)
```

```python
# 包含字段
models.Teacher.objects.filter(name__contains='c')
# SELECT "s_teacher"."id", "s_teacher"."name", "s_teacher"."course_id" FROM "s_teacher" WHERE "s_teacher"."name" LIKE '%c%' ESCAPE '\'  LIMIT 21;
```

```python
# 包含字段，大小写不敏感
models.Teacher.objects.filter(name__icontains='c')
```

```python
# 以某字符串开头
models.Teacher.objects.filter(name__startswith='con')
```

```python
# 以某字符串开头，大小写不敏感
models.Teacher.objects.filter(name__istartswith='con')
```

```python
# 以某字段结束
models.Teacher.objects.filter(name__endswith='nor')
```

```python
# 以某字段结束，大小写不敏感
models.Teacher.objects.filter(name__iendswith='nor')
```

```python
#时间日期相关

all_books=models.Book.objects.filter(pub_date__year=2012)
#找2012年的所有书籍

all_books=models.Book.objects.filter(pub_date__year__gt=2012)
#找大于2012年的所有书籍

all_books=models.Book.objects.filter(pub_date__year=2019,pub_date__month=2)#找2019年月份的所有书籍，如果明明有结果，你却查不出结果，是因为mysql数据库的时区和咱们django的时区不同导致的，了解一下就行了，你需要做的就是将django中的settings配置文件里面的USE_TZ = True改为False，就可以查到结果了，以后这个值就改为False，而且就是因为咱们用的mysql数据库才会有这个问题，其他数据库没有这个问题。
	models.Book.objects.filter(publish_date__isnull=True) #这个字段值为空的那些数据
```

```python
#regex：区分大小写的正则表达式匹配。
models.Book.objects.get(title__regex=r'^(An?|The) +')
建议使用原始字符串（例如，r'foo'而不是'foo'）来传递正则表达式语法。
```

详细讲解：http://www.liujiangblog.com/course/django/132

###### 聚合函数

| 聚合函数 |                             解释                             |
| :------: | :----------------------------------------------------------: |
|   Avg    | 返回给定表达式的平均值，它必须是数值，除非指定不同的`output_field`。 |
|  Count   |              返回与expression相关的对象的个数。              |
|   Max    |                   返回expression的最大值。                   |
|   Min    |                   返回expression的最小值。                   |
|  StdDev  |                   返回expression的标准差。                   |
|   Sum    |                 计算expression的所有值的和。                 |
| Variance |                    返回expression的方差。                    |

models.py下的__str__的写法：

```python
from django.db import models

# Create your models here.

class Book(models.Model):
    id = models.AutoField(primary_key=True)
    title = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=8,decimal_places=2,)
    pub_date = models.DateTimeField() #必须存这种格式"2012-12-12"
    publish = models.CharField(max_length=32)
    def __str__(self): #后添加这个str方法，也不需要重新执行同步数据库的指令
        return self.title #当我们打印这个类的对象的时候，显示title值
```

#### 时间问题

```python
models.UserInfo.objects.create(
        name='pl',
        bday=current_date,
        # now=current_date,  直接插入时间没有时区问题
        checked=0
    )
	但是如果让这个字段自动来插入时间,就会有时区的问题,auto_now_add创建记录时自动添加当前创建记录时的时间,存在时区问题
now = models.DateTimeField(auto_now_add=True,null=True)
解决方法:
    settings配置文件中将USE_TZ的值改为False
    # USE_TZ = True
    USE_TZ = False  # 告诉mysql存储时间时按照当地时间来储存,不要用utc时间
#使用pycharm的数据库客户端的时候,时区问题要注意
```

### 多表操作

#### 创建模型

##### **一对一**

```python
OneToOneField(to="表名",to_field="字段名",on_delete=CASCASE)
```

##### **多对一(ForeignKey)**

```python
#外键定义在多的一方
Foreign(to="表名",to_field="字段名",on_delete=CASCASE)

参数详讲
#to_field
默认情况下，外键都是关联到被关联对象的主键上（一般为id）。如果指定这个参数，可以关联到指定的字段上，但是该字段必须具有unique=True属性，也就是具有唯一属性。

#on_delete,这个参数在dj2.0之后，不可以省略，还需要显示的指定。
CASCASE：级联模式，有外键的的模型对象同时删除
PROTECT：严格模式，出错就报错
SET_NULL：将外键字段设置为null
SET_DEFAULT：将外键的字段设置为默认值
DO_DEFAULT：什么也不做
SET()：设置一耳光传递给SET()的值或者一个回调的函数的返回值

#limit_choices_to
该参数用于限定外键所能官关联的对象，只能用于Django的ModelForm(dj的表单模块)和admin后台，对其他的场合无限制功能。其值可以是一个字典、Q对象、一个返回字典或Q对象的函数调用
staff_member = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    limit_choices_to={'is_staff': True},
)
讲解：这样定义，则ModelForm的staff_member字段列表中，只会出现那些is_staff=True的Users对象，这一功能对于admin后台非常有用。
```

##### **多对多**

```python
ManyToManyField(to="表名")
#注意：在数据库后台，Django实际上会额外创建一张用于体现多对多关系的中间表。默认情况下，该表的名称是“多对多字段名+关联对象模型名+一个独一无二的哈希码”，例如‘author_books_9cdf4’，当然你也可以通过db_table选项，自定义表名。

#through字段：自定义中间表，下面详讲

#through_fields字段
接着上面的例子。Membership模型中包含两个关联Person的外键，Django无法确定到底使用哪个作为和Group关联的对象。所以，在这个例子中，必须显式的指定through_fields参数，用于定义关系。

through_fields参数接收一个二元元组('field1', 'field2')，field1是指向定义有多对多关系的模型的外键字段的名称，这里是Membership中的‘group’字段（注意大小写），另外一个则是指向目标模型的外键字段的名称，这里是Membership中的‘person’，而不是‘inviter’。

再通俗的说，就是through_fields参数指定从中间表模型Membership中选择哪两个字段，作为关系连接字段。
```

**多对多中间表详解（through字段）**

一般情况下，普通的多对多已经够用了，无需自己家创建第三张表关系，但是在某些复杂的情况下，比如我们想保存某个人加入某个分组的时间，保存进组的原因等等。

```python
#实例
from django.db import models
class Person(models.Model):
    name = models.CharField(max_length=128)
    def __str__(self):
        return self.name
class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person,through='Membership')
    def __str__(self):
        return self.name
class Membership(models.Model):
    person = models.ForeignKey(Person,on_delete=models.CASCADE)
    group = models.ForgineKey(Group,on_delete=models.CAECADE)
    date_joined = models.DateField()	#进组时间
    invite_reason = models.CahrField(max_length=64)	#邀请原因
    
#上面的代码中，通过class Membership(models.Model)定义了一个新的模型，用来保存Person和Group模型的多对多关系，并且同时增加了‘邀请时间’和‘邀请原因’的字段。
在中间表中，我们至少要编写两个外键字段，分别指向关联的两个模型。在本例中就是‘Person’和‘group’。 这里，我们额外增加了‘date_joined’字段，用于保存人员进组的时间，‘invite_reason’字段用于保存邀请进组的原因。
```

#### 表结构

```python
from django.db import models

# Create your models here.

class Author(models.Model):
    """
    作者表
    """
    name=models.CharField( max_length=32)
    age=models.IntegerField()
    # authorDetail=models.OneToOneField(to="AuthorDetail",to_field="nid",on_delete=models.CASCADE)  完整的写法#
    au=models.OneToOneField("AuthorDetail",on_delete=models.CASCADE)

class AuthorDetail(models.Model):
    """
    作者详细信息表
    """
    birthday=models.DateField()
    telephone=models.CharField(max_length=11)
    addr=models.CharField(max_length=64)
    # class Meta:
        # db_table='authordetail' #指定表名
        # ordering = ['-id',]
class Publish(models.Model):
    """
    出版社表
    """
    name=models.CharField( max_length=32)
    city=models.CharField( max_length=32)

class Book(models.Model):
    """
    书籍表
    """
    title = models.CharField( max_length=32)
    publishDate=models.DateField()
	price=models.DecimalField(max_digits=5,decimal_places=2)
    publishs=models.ForeignKey(to="Publish",on_delete=models.CASCADE,)
    authors=models.ManyToManyField('Author',)
```

#### 创建记录(增)

```python
#一对一
方式一(属性.的方式)：
au_obj = models.AuthorDetail.objects.get(id=4)
au_obj.name = "pl"
au_obj.age = 18
au_obj.au = 4    #一对一的表关系
方式二(直接对象的方式)：
models.Author.objects.create(
	name = "pl",
    age = 18,
	au = 4,  #一对一的表关系
)

#一对多
方式一：
pub_obj = models.Publish.objects.get(id=3)
......
方式二：
models.Book.objects.create(
	title = "从你的全世界路过",
    price = 33,
    publishs_id = 3,   #如果关键字为数据库字段名称，那么值为关联数据的值，操作的是有关联字段的表
)

#多对多
pl1 = models.Author.objects.get(id=3)
pl2 = models.Author.objects.get(id=5)

book_obj = models.Book.objects.create(
	title = "皮囊",
    price = 20,
    publishs_id = 2,
)
book_obj.authors.add(3,5)   #*args  **kwargs
book_obj.authors.add(*[3,5])  # 用的最多,打散
book_obj.authors.add(pl1, pl2)
```

#### 删除记录(删)

```python
#一对一
models.AuthorDetail.objects.filter(id=3).delete()
models.Author.objects.filter(id=3).delete()
#一对多
models.Publish.objects.filter(id=3).delete()
models.Book.objects.filter(id=4).delete()
#多对多
book_obj = models.Book.objects.get(id=2)
book_obj.authors.remove(1)  #删除
book_obj.authors.clear()  # 清除
book_obj.authors.set(['1','5'])  # 先清除再添加,相当于修改
```

#### 修改记录(改)

```python
ret = models.Publish.objects.get(id=2)
models.Book.objects.filter(id=5).update(
	title = "左耳",
	publish = ret,
)
```

#### 查询记录(查)

##### **基于对象的跨表查询**

```python
正向查询和反向查询：关系属性写在表1,关联到表2,那么通过表1的数据去找表2的数据,叫做正向查询,返过来就是反向查询
#一对一
正向查询：查名字是pl的手机号码（对象.属性）
obj = models.Author.objects.filter(name="pl").first()
ph = obj.au.thlephone
反向查询：查手机号码是120的作者的名字（对象.小写的表名）
obj = models.AuthorDetail.objects.filter(telephone=120).first()
ret = obj.author.name 

#一对多
正向查询：查询《左耳》这本书是哪个出版社出版的
obj = models.Book.objects.filter(title="左耳").first()
ret = obj.publish.name
反向查询：查询一下xx出版社出版了哪些书籍
obj = models.Publish.objects.filter(name="xx出版社").first()
ret = obj.book_set.all()  #qset对象，需要进一步处理

#多对多
正向查询：查询左耳的作者有哪些
obj = models.Book.objects.filter(title="左耳").first()
ret = obj.authors.all()
for i in ret:
    print (i.name)
反向查询：查询pl一共写过那些书籍
obj = models.Author.objects.filter(name="pl").first()
ret = obj.book_set.all()
for i in ret:
    print (i.title)
```

**明确：想要从数据库内检索对象，需要基于模型类，通过管理器（Manager）构造一个查询结果集（QuerySet）。每个QuerySet代表一些数据库对象的集合。**

注意：下面总结的可能和上面单表查询的有重复的，，但是这里的总结更加的有代表性和针对性。

##### **基于双下划线的跨表查询（join）**

```python
#一对一
#查名字是pl的手机号码（对象.属性）
#正：
models.Author.objects.filter(name="pl").values(telephone="au_telephone")
#反：
models.AuthorDetail.objects.filter(author_name="pl").values("telephone")

#多对一
#左耳这本书是那个出版社出版的(查一)
models.Book.objects.filter(title="左耳").values('publish__name')
models.Publish.objects.filter(book__title="左耳").values('name')

#查询“书香出版社”出版了那些书(查多)
models.Publish.objects.filter(name="书香出版社").values("book_title")
models.Book.objects.filter(publish_name="书香出版社").values('title')
'''注意：上面查询的方法一样，但是可以查出所有的内容'''

#多对多
#查"左耳"是哪些作者写的
models.Book.objects.filter(title="左耳").values("author__name")
models.Author.objects.filter(book_title="左耳").values("name")

#查询“书香出版社”出版了那些书
models.Publish.objects.filter(name="书香出版社").values("book__title")
```

##### 聚合查询

`aggregate`(*args, **kwargs)

```python
from django.db.models import Avg, Sum, Max, Min, Count
ret = models.Book.objects.all().aggregate(a=Avg('price'),m=Max('price'))
print(ret) 
#{'price__avg': 45.1, 'price__max': Decimal('200.00')} python字典格式,也就是说,聚合查询是orm语句的结束
```

##### 分组查询

`annotate`(*args,**kwargs)

```python
每个出版社出版的书的平均价格
# 用的是publish表的id字段进行分组
ret = models.Book.objects.values('publishs__id').annotate(a=Avg('price'))
# 用的book表的publishs_id字段进行分组
ret = models.Book.objects.values('publishs_id').annotate(a=Avg('price'))
print(ret)
#查询的是publish整个表的
ret = models.Publish.objects.annotate(a=Avg('book__price')).values('a')
print(ret) 
#<QuerySet [{'a': None}, {'a': 71.166667}, {'a': 6.0}]>
```

##### 过滤对象

有两个方法可以过滤QuerySet对象的结果，分别是：

- fifter(**kwargs):返回一个根据指定参数查询出来的QuerySet
- exclude(**kwargs):根据条件查询出来的反集

**QuerySets都是惰性的**：一个创建QuerySets的动作不会立刻导致任何的数据库行为。你可以不断地进行filter动作一整天，Django不会运行任何实际的数据库查询动作，直到QuerySets被提交(evaluated)。简而言之就是，只有碰到某些特定的操作，Django才会将所有的操作体现到数据库内，否则它们只是保存在内存和Django的层面中。这是一种提高数据库查询效率，减少操作次数的优化设计。

```python
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
#注意：过滤对象支持链式操作
```

**上面的例子，看起来执行了3次数据库访问，实际上只是在print语句时才执行1次访问。**通常情况，QuerySets的检索不会立刻执行实际的数据库查询操作，**直到出现类似print的请求，也就是所谓的evaluated**。

##### 检索单一对象

get(),在get方法中你可以使用任何filter方法中的查询参数，用法也是一模一样。和filter()对比的内容在上面，这个方法慎用。

##### QuerySet使用限制

- 使用类似Python对列表进行切片的方法可以对QuerySet进行范围取值。它相当于SQL语句中的LIMIT和OFFSET子句。

```python
>>> Entry.objects.all()[:5]      # 返回前5个对象
>>> Entry.objects.all()[5:10]    # 返回第6个到第10个对象
#注意：不支持负索引
```

- 通常情况，切片操作会返回一个新的QuerySet，并且不会被立刻执行。但是有一个例外，那就是指定步长的时候，查询操作会立刻在数据库内执行。
- 若要获取单一的对象而不是一个列表（例如，SELECT foo FROM bar LIMIT 1），可以简单地使用索引而不是切片。

注意：如果没有匹配到对象，那么第一种方法会抛出IndexError异常，而第二种方式会抛出DoesNotExist异常。也就是说在使用get和切片的时候，要注意查询结果的元素个数。

##### F查询：运算

F() 的实例可以在查询中引用字段，来比较同一个 model 实例中两个不同字段的值。

**导入模块**

```python
from django.db.models import F
```

实例

```python
#Django支持对F()对象进行加、减、乘、除、取模以及幂运算等算术操作。两个操作数可以是常数和其它F()对象。

# 查询评论数大于收藏数的书籍
from django.db.models import F
Book.objects.filter(commentNum__lt=F('keepNum'))
# 查询评论数大于收藏数2倍的书籍
Book.objects.filter(commentNum__lt=F('keepNum')*2)
#将每一本书的价格提高30元：
Book.objects.all().update(price=F("price")+30)　
```

##### Q查询：与或非

filter()方法默认是and，并没有提供or和not方法，无法使用复杂的查询方法,可以使用Q方法封装字段进行复杂的查询，可以组合使用 &（and）, |（or），~（not）操作符对Q对象进行操作

**导入模块**

```python
from django.db.models import Q
```

**and**

```python
# 查询年龄15岁，并且在1506班上课的同学
models.Student.objects.filter(Q(age=15) & Q(classes__name='1508'))
```

**or**

```python
# 查询年龄12岁或在1508班上课的同学
models.Student.objects.filter(Q(age=12) | Q(classes__name='1508'))
```

**not**

```python
# 查询年龄12岁或在1508班上课的同学
models.Student.objects.filter(Q(age=12) | Q(classes__name='1508'))
```

**组合使用**

```python
# 查找年龄大于25岁或者年龄等于12岁并且不在1506班的学生
models.Student.objects.filter(Q(age__gt=25) | Q(age=12) & ~Q(classes__name='1506'))
```

##### 主键的快捷查询方式：pk

pk就是`primary key`的缩写。通常情况下，一个模型的主键为“id”，所以下面三个语句的效果一样：

```python
>>> Blog.objects.get(id__exact=14) # Explicit form
>>> Blog.objects.get(id=14) # __exact is implied
>>> Blog.objects.get(pk=14) # pk implies id__exact
```

##### 缓存与查询集

每个QuerySet都包含一个缓存，用于减少对数据库的实际操作。理解这个概念，有助于你提高查询效率。

对于新创建的QuerySet，它的缓存是空的。当QuerySet第一次被提交后，数据库执行实际的查询操作，Django会把查询的结果保存在QuerySet的缓存内，随后的对于该QuerySet的提交将重用这个缓存的数据。

```python
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # 提交查询
>>> print([p.pub_date for p in queryset]) # 重用查询缓存

#这种方式就是很好的利用了缓存的机制，我们把一次操作IO，包数据加载到内存中，我们在对象的基础上进行操作，显然提高了效率。
```

注意：有一些操作不会缓存QuerySet，例如切片和索引。

### 模型的继承

很多时候，我们都不是从‘一穷二白’开始编写模型的，有时候可以从第三方库中继承，有时候可以从以前的代码中继承，甚至现写一个模型用于被其它模型继承。这样做的好处，我就不赘述了，每个学习Django的人都非常清楚。

Django中所有的模型都必须继承`django.db.models.Model`模型，不管是直接继承也好，还是间接继承也罢。

**Django的三种继承的方式：**

- 抽象基类

  被用来继承的模型被称为`Abstract base classes`，将子类共同的数据抽离出来，供子类继承重用，它不会创建实际的数据表。

- 多表继承

  `Multi-table inheritance`，每一个模型都有自己的数据库表。

- 代理模型

  如果我们只想改变模型的Python层面的行为，并不想改动模型的字段，可以使用代理模型。

注意：同Python的继承一样，Django也是可以同时继承两个以上父类的。

#### 抽象基类

只需要在模型的Meta类里添加`abstract=True`元数据项，就可以将一个模型转换为抽象基类。Django不会为这种类创建实际的数据库表，它们也没有管理器，不能被实例化也无法直接保存，它们就是用来被继承的。抽象基类完全就是用来保存子模型们共有的内容部分，达到重用的目的。当它们被继承时，它们的字段会全部复制到子模型中。看下面的例子：

```python
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()
	
    class Meta:
        abstract = True
class Student(commonInfo):
    home_group = models.CharField(max_length=5)
#解释：Student模型将拥有name,age,home_group这三个字段，并且CommonInfo模型不能当作一个正常的模型使用。
```

对于抽象基类的Meta数据，如果子类没有声明自己的Meta类，那么它将继承抽象基类的Meta类：

```python
from django.db import models

class CommonInfo(models.Model):
    # ...
    class Meta:
        abstract = True
        ordering = ['name']

class Student(CommonInfo):
    # ...
    class Meta(CommonInfo.Meta):
        db_table = 'student_info'
```

- 抽象基类中有的元数据，子模型没有的话，直接继承
- 抽象基类中有的元数据，子模型也有的话，直接覆盖
- 子模型可以额外添加元数据
- 抽象基类中的abstract=True这个元数据不会被继承，也就是说如果想让一个抽象基类的紫魔性，同样成为一个抽象基类，那我们必须显示的在该紫魔性的Meta中同样声明它是一个抽象基类
- 有一些元数据对抽象基类无效，比如db_table，首先是抽象基类本身不会创建数据表，其次它的所有子类也不会按照这个元数据来设置表名

#### 多表继承

这种继承方式下，父类和子类都是各自自主、功能完整、可正常使用的模型，都有自己的数据表，内部隐含了一个一对一的关系。

```python
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)
```

Restaurant将包含Place的所有字段，并且各有各的数据库表和字段.

**Meta和多表继承**

在多表继承的情况下，由于父类和子类都在数据库内有物理存在的表，父类的Meta类会对子类造成不确定的影响，因此，Django在这种情况下关闭了子类继承父类的Meta功能，这一点和抽象基类继承方式有所不同。但是，还是有两个Meta元数据特殊一点，那就是`ordering`和`get_latest_by`，这两个参数是会被继承的。因此，如果在多表继承中，你不想让你的子类继承父类的上面两种参数，就必须在子类中显示的指出或重写。如下：

```python
class ChildModel(ParentModel):
    # ...
    class Meta:
        # 移除父类对子类的排序影响
        ordering = []
```

**多表继承和反向关联**

因为多表继承使用了一个隐含的OneToOneField来链接子类与父类，所以象上例那样，你可以从父类访问子类。但是这个OnetoOneField字段默认的`related_name`值与ForeignKey和 ManyToManyField默认的反向名称相同。如果你与父类或另一个子类做多对一或是多对多关系，你就必须在每个多对一和多对多字段上强制指定`related_name`。如果你没这么做，Django就会在你运行或验证(validation)时抛出异常。

解决办法：向customers字段中添加`related_name`参数.

```python
customers = models.ManyToManyField(Place, related_name='provider')
```

#### 代理模型

**声明一个代理模型只需要将Meta中proxy的值设为True。**

加入我们给Person模型添加一个方法：

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)

class MyPerson(Person):
    class Meta:
        proxy = True

    def do_something(self):
        # ...
        pass
#MyPerson类操作和Person类同一张数据库表，并且任何新的Person实例都可以通过MyPerson类进行访问，反之亦然。
```

通过代理进行排序，但父类却不排序：

```python
class OrderedPerson(Person):
    class Meta:
        # 现在，普通的Person查询是无序的，而OrderedPerson查询会按照`last_name`排序。
        ordering = ["last_name"]
        proxy = True
```

- 代理模型必须继承自一个非抽象的基类，并且不能同时继承多个非抽象基类
- 代理模型可以同时继承任意多个抽象基类，前提是这些抽象基类没有定义任何模型字段
- 代理模型可以同时继承多个比欸的代理模型，前提是这些代理模型继承同一个非抽象基类
- 自类默认继承父类的管理器；如果我们自定义了管理器，那它就会成为默认管理器，但是父类的管理器依然有效。

#### 多重继承

Django的模型体系支持多重继承，就像Python一样。如果多个父类都含有Meta类，则只有第一个父类的会被使用，剩下的会忽略掉。

**一般情况，能不要多重继承就不要，尽量让继承关系简单和直接，避免不必要的混乱和复杂。**

### 用包类组织模型

在我们使用`python manage.py startapp xxx`命令创建新的应用时，Django会自动帮我们建立一个应用的基本文件组织结构，其中就包括一个`models.py`文件。通常，我们把当前应用的模型都编写在这个文件里，但是如果你的模型很多，那么将单独的`models.py`文件分割成一些独立的文件是个更好的做法。

首先，我们需要在应用中新建一个叫做`models`的包，再在包下创建一个`__init__.py`文件，这样才能确立包的身份。然后将`models.py`文件中的模型分割到一些`.py`文件中，比如`organic.py`和`synthetic.py`，然后删除`models.py`文件。最后在`__init__.py`文件中导入所有的模型。如下例所示：

```python
#  myapp/models/__init__.py

from .organic import Person
from .synthetic import Robot
```

要显式明确地导入每一个模型，而不要使用`from .models import *`的方式，这样不会混淆命名空间，让代码更可读，更容易被分析工具使用。

### 练习

```python
#1 查询每个作者的姓名以及出版的书的最高价格
    ret = models.Author.objects.values('name').annotate(max_price=Max('book__price'))
    print(ret) #注意：values写在annotate前面是作为分组依据用的，并且返回给你的值就是这个values里面的字段（name）和分组统计的结果字段数据(max_price)
	ret = models.Author.objects.annotate(max_price=Max('book__price')).values('name','max_price')#这种写法是按照Author表的id字段进行分组，返回给你的是这个表的所有model对象，这个对象里面包含着max_price这个属性，后面写values方法是获取的这些对象的属性的值，当然，可以加双下划线来连表获取其他关联表的数据，但是获取的其他关联表数据是你的这些model对象对应的数据，而关联获取的数据可能不是你想要的最大值对应的那些数据

# 2 查询作者id大于2作者的姓名以及出版的书的最高价格
ret = models.Author.objects.filter(id__gt=2).annotate(max_price=Max('book__price')).values('name','max_price')#记着，这个values取得是前面调用这个方法的表的所有字段值以及max_pirce的值，这也是为什么我们取关联数据的时候要加双划线的原因
    print(ret)

#3 查询作者id大于2或者作者年龄大于等于20岁的女作者的姓名以及出版的书的最高价格
ret = models.Author.objects.filter(Q(id__gt=2)|Q(age__gte=20),sex='female').annotate(max_price=Max('book__price')).values('name','max_price')
    
#4 查询每个作者出版的书的最高价格 的平均值
     ret = models.Author.objects.values('id').annotate(max_price=Max('book__price')).aggregate(Avg('max_price')) #{'max_price__avg': 555.0} 注意，aggregate是queryset的终止句，得到的是字典
     ret = models.Author.objects.annotate(max_price=Max('book__price')).aggregate(Avg('max_price')) #{'max_price__avg': 555.0} 注意，aggregate是queryset的终止句，得到的是字典

#5 每个作者出版的所有书的最高价格以及最高价格的那本书的名称（通过orm玩起来就是个死题，需要用原生sql）
select title,price from (select app01_author.id,app01_book.title,app01_book.price from app01_author INNER JOIN app01_book_authors on app01_author.id=
app01_book_authors.author_id INNER JOIN app01_book on app01_book.id=
app01_book_authors.book_id ORDER BY app01_book.price desc) as b  GROUP BY id

print(ret)
```

### 补充

```python
class UserInfo(models.Model):
	"""
	用户表
	"""
	username = models.CharField(max_length=32, verbose_name="用户名")
	password = models.CharField(max_length=128, verbose_name="密码")
	roles = models.ManyToManyField('Role', verbose_name="拥有的所有的角色")

	class Meta:
		verbose_name = "用户表"
		verbose_name_plural = "用户表"

	def __str__(self):
		#它的作用是什么呢，就是在站点中加载的时候不是以
		#对象的方式展示，这样就不友好了，返回一个可是别的内容
		#比如名称，就可以替代掉显示的对象
		return self.username
    
#比如，我们的多对多的表，我们在admin添加数据的时候，我们取得的关联数据显示如果不使用__str__返回这个人性化的显示的话，就会显示一个对象，这样很不友好的，而使用时候，我们在选择关联的时候就会展示我们的用户名称，就很好知道我们关联的对象是哪一个了
```







