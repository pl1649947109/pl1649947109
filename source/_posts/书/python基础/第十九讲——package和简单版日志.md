---
title: 第十九讲——package和简单版日志
id: 19
date: 2019-8-7 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 包
- 日志logging

<!-----more----->

### 包

- 我们今天来讲解一下模块和包,模块我们已经知道是什么东西了,我们现在来看看这个包是个什么? 我说的包可不是女同胞一看见就走不动的包,而是程序中一种组织文件的形式。
- 怎么判断一个文件目录是不是一个包？
   - 只要文件夹下含有__init__.py文件就是一个包,包是干什么的呢?
   - 注意，在python3中，即时包下没有_init.py文件，import包仍然不会报错，在python2包中，包下一定要有该文件，否则import就会报错。
- 为什么要使用包？
   - 包的本质就是一个文件夹，那么文件夹唯一的功能就是将文件组织起来，随着功能越写越多，我们无法将所以功能都放到一个文件中，于是我们使用模块去组织功能，而随着模块越来越多，我们就需要用文件夹将模块文件组织起来，以此来提高程序的结构性和可维护性。
- 包的结构

```python
bake            

    ├── __init__.py       

    ├── api               

        ├── __init__.py

        ├── policy.py

        └── versions.py

  ├── cmd             

    ├── __init__.py

    └── manage.py

  └── db                

      ├── __init__.py

      └── models.py
```

我们在bake同级创建一个test.py进行导入policy.py 我们使用模块的import的时候只能将api添加到sys.path的路劲中,我们来看看包使用import导入

```python
import bake.api.policy
bake.api.policy.get()
```

导入的太长了下边使用的时候还需要在重复写一遍,我们可以使用as起别名

```python
import bake.api.policy as p
p.get()
```

这样的操作只支持包,普通的文件夹无效,有人一定在想我把bake拿过来然后一层一层的打开那拿工具就可以了

```python
import bake
bake.api.policy.get()
```

不好使,这样导入是只将policy导入了,有人想怎么将api包下的模块全部导入不要急,先说单独导入的方式

咱们能够使用import进行导入,在来看看from的导入方式

```python
from bake.api import policy
policy.get()
from bake import api
print(api.versions.name)
```

还是不好使,通过这两个我们能够感觉都导入的时候指定要导入的内容,不能再导入后在进行开箱子

我们现在说了单独导入一个模块,现在来说道说道怎么导入某个包下的所有模块,想要导入某个包下的所有的模块 我们就需要在包中的__init__.py做点手脚

```python
bake包下的__init__.py
from . import api
```

.是当前路径,因为from的时候不能空着

```python
api包下的__init__.py
from . import policy
```

我们将包下的__init__配置好,然后在test.py进行导入

```python
import bake
bake.api.policy.get()
```

又好使了,这是为什么呢?

我们import导入bake这个包,因为bake是一个文件夹不能进行任何操作,就让__init__.py代替它 去将api这包中的模块导入,api也是一个文件夹不能操作就需要让api下边的__init__.py去找api下边的两个模块。

这个和公司的上下级关系一样,打比方现在test.py就是一个ceo要和policy这个小员工谈话,ceo先把这个想法人事经理,人事经理就是 bake这个包,人事经理通知人事让人事查找一下policy在那个部门,人事查到后通知部门的负责人,部门的负责人在通知部门的主管,主管告诉policy这个员工, 说ceo要找你,部门的主管带着policy去找人事,人事带着policy,人事然后在带着policy去找ceo.最后成功的和ceo进行了一番交流

如果在传达的时候中间一个环节忘记传递了,policy就不知道ceo在找他,ceo等了好久不来ceo就生气报错了

使用的时候需要注意: 有的同学,想在policy文件中导入versions就是直接使用import,在policy文件使用没有问题,很美,很高兴.但是在test.py执行的时候就会报错 因为我们在test.py中执行的import versions 相当于在test.py文件进行查找,肯定不会找到,我们需要在policy文件中向sys.path添加了当前的路劲就可以了 具体操作如下:

```python
import os
import sys
sys.path.insert(os.path.dirname(__file__)
```

__file__获取的是当前文件的路径,这样我们就能在test中正常使用了,我们使用from也能够将某个包下所有的模块全都导入 比如我们现在想将cmd包下的所有的模块导入需要在bake包下的__init__.py进行设置

```python
from . import *
```

我们需要在api包下设置__init__.py

```pyhton
__all__ = ["policy","versions"]
或
from . import policy
from . import versions
```

我们需要在db包下设置__init__.py

```python
__all__ = ["models"]
或
from . import models
```

我们需要在cmd包下设置__init__.py

```python
__all__ = ["manage"]
或
from . import manage
```

以上两种推荐使用下from . import manage 灵活,可读性高

test.py调用如下:

```python
from bake.api import *
print(versions.name)
policy.get()

from bake.db import *
models.register_models(123)

from bake.cmd import *
print(manage.name)
```

在使用import有个注意点,python2中如果import包,没有__init__.py文件就会报错 python3 import没有__init__.py文件的包不会报错 from 包 import 包或者模块(在import后边不能在进行.操作)

路径: 绝对路径:从最外层(bake)包.查找的就是绝对路径 相对路径:.就是相对路径, ..是上一级目录 例如：我们在bake/api/version.py中想要导入bake/cmd/manage.py

```python
# 绝对路径:
from bake.cmd import manage
manage.main()

#相对路径:
from ..cmd import manage
manage.main()
```

注意在使用相对路径的时候一定要在于bake同级的文件中测试 我们需要在和bake同级的test.py中测试

```python
from bake.cmd import manage
```

### 日志logging

详讲请移步博客：https://blog.csdn.net/qq_40890660/article/details/98482170

```python
一版：
#函数式的简单配置
#默认是从warning开始记录
#logging：日志（总共就5个级别）
logging.debug("我是调试")
logging.info("我是信息")
logging.warning("我是警告")
logging.error("我是错误")
logging.critical("我是危险")
#日志级别等级：CRITICAL>ERROR>WARNING>INFO>DEBUG

二版：
#灵活配置日志级别，日志格式，输出位置
logging.basicConfig(level=logging.DEBUG,
                    format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',  #输出的信息
                    datefmt='%a, %d %b %Y %H:%M:%S',  #获取时间
                    filename='./test.log',  #把输出保存到文件
                    filemode='w') #写入文件

logging.debug('debug message')
logging.info('info message')
logging.warning('warning message')
logging.error('error message')
logging.critical('critical message')
"""
basicConfig()函数中可以通过具体的参数来更改logging模块的默认行为：如下参数
filename：用指定的文件名创建FiledHandler，这样日志会被存储在指定的文件中。
filemode：文件打开方式，在指定了filename时使用这个参数，默认值为“a”还可指定为“w”。
format：指定handler使用的日志显示格式。
datefmt：指定日期时间格式。
level：设置记录日志的级别
stream：用指定的stream创建StreamHandler。可以指定输出到
sys.stderr,sys.stdout或者文件(f=open(‘test.log’,’w’))，默认为sys.stderr。若同时列出了filename和stream两个参数，则stream参数会被忽略。
#其中，format参数中可能用到的格式化串
%(name)s Logger的名字
%(levelno)s 数字形式的日志级别
%(levelname)s 文本形式的日志级别
%(pathname)s 调用日志输出函数的模块的完整路径名，可能没有
%(filename)s 调用日志输出函数的模块的文件名
%(module)s 调用日志输出函数的模块名
%(funcName)s 调用日志输出函数的函数名
%(lineno)d 调用日志输出函数的语句所在的代码行
%(created)f 当前时间，用UNIX标准的表示时间的浮 点数表示
%(relativeCreated)d 输出日志信息时的，自Logger创建以 来的毫秒数
%(asctime)s 字符串形式的当前时间。默认格式是 “2003-07-08 16:49:45,896”。逗号后面的是毫秒
%(thread)d 线程ID。可能没有
%(threadName)s 线程名。可能没有
%(process)d 进程ID。可能没有
%(message)s用户输出的消息
"""

三版：
#logger对象配置
logger = logging.getLogger()
# 创建一个handler，用于写入日志文件
fh = logging.FileHandler('test.log',encoding='utf-8')

# 再创建一个handler，用于输出到控制台
ch = logging.StreamHandler()
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

fh.setLevel(logging.DEBUG)

fh.setFormatter(formatter)
ch.setFormatter(formatter)
logger.addHandler(fh) #logger对象可以添加多个fh和ch对象
logger.addHandler(ch)

logger.debug('logger debug message')
logger.info('logger info message')
logger.warning('logger warning message')
logger.error('logger error message')
logger.critical('logger critical message')
'''
logging库提供了多个组件：Logger、Handler、Fifler、Formatter。
Logger：提供应用程序可直接使用的接口。
Handler：发送日志到适当的目的地。
Fifler：提供了过滤日志信息的方法。
Formatter：指定日志显示格式。
另外，可以通过logger.setLevel(logging.Dubug)设置级别，
当然也可以通过fh.setLevel(logging.Debug)单对文件流设置某个级别
'''
```

