---
title: python之日志详讲
id: 2
date: 2019-7-30 20:00:00
tags: python部分内容详讲
comment: true
---

导读：日志在我们日后的开发中是非常有用的一个工具，本文的内容将讲解3个日志版本，由浅入深的带领大家学习。当然在实际的开发中进阶版日志是最有用的也是最常用的。

### 学习大纲：

- 日志的分类
- 日志的级别

- 阉割版日志

- 简单版日志

- 进阶版日志

<!-----more----->

### 日志的分类

```
#--系统日志：记录操作系统、服务器的硬件性能（cpu，网卡，内存运行等）将获得的。
	#数据以文件的形式保存在文件里面(一般是运维人员来做的)，记录运维人员的命令。
#--网站日志：用户的访问次数，用户的停留时间，访问量。
	#各地区的访问量等等（开发用的多）
#--开发辅助日志：debug，info模式，代替print(在开发中print是非常消耗性能的)。
	#try  except下面的日志记录
#用户信息日志：记录用户的转账，流水等用户对系统的操作。
```

### 日志的级别

```python
# logging：日志（总共就5个级别）
# logging.debug("我是调试")
# logging.info("我是信息")
# logging.warning("我是警告")
# logging.error("我是错误")
# logging.critical("我是危险")
默认是从warning开始记录
```

### 阉割版日志

```python
logging.basicConfig(level=logging.DEBUG,
                     #输出的信息，也就是写入文件的信息
                    format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s', 
                     #获取时间
                    datefmt='%a, %d %b %Y %H:%M:%S', 
                    #把输出保存到文件
                    filename='./test.log', 
                    filemode='w') 

logging.debug('debug message')
logging.info('info message')
logging.warning('warning message')
logging.error('error message')
logging.critical('critical message')
解释：上面的logging.basicConfig是一个元组，里面封装的都是自选定的一些功能。

#basicConfig()函数中可以通过具体的参数来更改logging模块的默认行为：
如下参数：
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
```

### 简单版日志

```python
#首先logger对象配置
logger = logging.getLogger()

# 创建一个handler，用于写入日志文件
fh = logging.FileHandler('test.log',encoding='utf-8')

# 再创建一个handler，用于输出到控制台
ch = logging.StreamHandler()
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

#配置输级别（写入文件的）
fh.setLevel(logging.DEBUG)

fh.setFormatter(formatter)
ch.setFormatter(formatter)

#logger对象可以添加多个fh和ch对象
logger.addHandler(fh) 
logger.addHandler(ch)

logger.debug('logger debug message')
logger.info('logger info message')
logger.warning('logger warning message')
logger.error('logger error message')
logger.critical('logger critical message')
解释上述的组件：
"""
logging库提供了多个组件：Logger、Handler、Fifler、Formatter。
Logger：提供应用程序可直接使用的接口。
Handler：发送日志到适当的目的地。
Fifler：提供了过滤日志信息的方法。
Formatter：指定日志显示格式。
另外，可以通过logger.setLevel(logging.Dubug)设置级别，
当然也可以通过fh.setLevel(logging.Debug)单对文件流设置某个级别
"""
```

### 进阶版日志

```python
import os
import logging.config
# 定义三种日志输出格式（标准版、简单版、加级别的简单版）
#其中name为getlogger指定的名字
standard_format = '[%(asctime)s][%(threadName)s:%(thread)d][task_id:%(name)s][%(filename)s:%(lineno)d][%(levelname)s][%(message)s]' 

simple_format = '在 %(asctime)s %(message)s'

id_simple_format = '[%(levelname)s][%(asctime)s] %(message)s'


# log文件的全路径
logfile_path = 'all2.log'

# log配置字典
LOGGING_DIC = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        #封装上面定义的日志格式
        'standard': {
            'format': standard_format
        },
        'simple': {
            'format': simple_format
        },
    },
    'filters': {},
    'handlers': {
        #打印到终端的日志
        'stream': {
            'level': 'DEBUG',
            #打印到屏幕
            'class': 'logging.StreamHandler', 
            #输出的模式，是上面定义的
            'formatter': 'simple'
        },
        #打印到文件的日志,收集info及以上的日志
        'file': {
            #设置信息保存的等级
            'level': 'DEBUG',
            # 保存到文件
            'class': 'logging.handlers.RotatingFileHandler',  
            #输出的模式，是上面定义的
            'formatter': 'standard',
            #日志文件
            'filename': None,  
            #日志大小（最好不要设置太大）
            'maxBytes': 1024*1024*1024,
            #默认保存几个文件，超过了就覆盖了
            'backupCount': 5,
            #日志文件的编码，再也不用担心中文log乱码了
            'encoding': 'utf-8',  
        },
    },
    'loggers': {
        #logging.getLogger(__name__)拿到的logger配置
        '': {
            'handlers': ['stream', 'file'],  # 这里把上面定义的两个handler都加上，即log数据既写入文件又打印到屏幕
            'level': 'DEBUG',
            'propagate': True,  # 向上（更高level的logger）传递
        },
    },
}


def get_logger():
    path = 保存到自己指定的路径
    LOGGING_DIC['handlers']['file']['filename'] = path
    # 导入上面定义的logging配置
    logging.config.dictConfig(LOGGING_DIC)  
    # 生成一个log实例
    logger = logging.getLogger(__name__)  
    return logger

def save():
    logger = get_logger()
    # 记录该文件的运行状态，就是把信息输出到文件或者屏幕
    logger.info(f'{} 存入300元')  

save()
```