---
title: 第八部分——drf、flask、爬虫、Git、linux、其他、项目
id: 8
date: 2019-11-22 20:00:00
tags: 面试题
comment: true
---

### drf

#### 1.接口的幂等性是什么意思？

```
'一个接口通过1次相同的访问，再对该接口进行N次相同的访问时，对资源不造影响就认为接口具有幂等性。'
    GET，  #第一次获取结果、第二次也是获取结果对资源都不会造成影响，幂等。
    POST， #第一次新增数据，第二次也会再次新增，非幂等。
    PUT，  #第一次更新数据，第二次不会再次更新，幂等。
    PATCH，#第一次更新数据，第二次不会再次更新，幂等。
    DELTE，#第一次删除数据，第二次不在再删除，幂等。
```

### flask

#### 1.Flask框架依赖组件

```
# 依赖jinja2模板引擎
# 依赖werkzurg协议
```

#### 2.Flask蓝图的作用

```
# blueprint把实现不同功能的module分开.也就是把一个大的App分割成各自实现不同功能的module.
# 在一个blueprint中可以调用另一个blueprint的视图函数, 但要加相应的blueprint名.
```

<!----more---->

#### 3.列举使用的Flask第三方组件？

```
# Flask组件
    flask-session  session放在redis
    flask-SQLAlchemy 如django里的ORM操作
    flask-migrate  数据库迁移
    flask-script  自定义命令
    blinker  信号-触发信号
# 第三方组件
    Wtforms 快速创建前端标签、文本校验
    dbutile     创建数据库连接池
    gevnet-websocket 实现websocket
# 自定义Flask组件
    自定义auth认证 
    参考flask-login组件
```

#### 4.简述Flask上下文管理流程?

```
# a、简单来说，falsk上下文管理可以分为三个阶段：
　　1、'请求进来时'：将请求相关的数据放入上下问管理中
　　2、'在视图函数中'：要去上下文管理中取值
　　3、'请求响应'：要将上下文管理中的数据清除
# b、详细点来说：
　　1、'请求刚进来'：
        将request，session封装在RequestContext类中
        app，g封装在AppContext类中
        并通过LocalStack将requestcontext和appcontext放入Local类中
　　2、'视图函数中'：
        通过localproxy--->偏函数--->localstack--->local取值
　　3、'请求响应时'：
        先执行save.session()再各自执行pop(),将local中的数据清除
```

#### 5.Flask中的g的作用？

```
# g是贯穿于一次请求的全局变量，当请求进来将g和current_app封装为一个APPContext类，
# 再通过LocalStack将Appcontext放入Local中，取值时通过偏函数在LocalStack、local中取值；
# 响应时将local中的g数据删除：
```

#### 6.Flask中上下文管理主要涉及到了那些相关的类？并描述类主要作用？

```
RequestContext  #封装进来的请求（赋值给ctx）
AppContext      #封装app_ctx
LocalStack      #将local对象中的数据维护成一个栈（先进后出）
Local           #保存请求上下文对象和app上下文对象
```

#### 7.为什么要Flask把Local对象中的的值stack 维护成一个列表？

```
# 因为通过维护成列表，可以实现一个栈的数据结构，进栈出栈时只取一个数据，巧妙的简化了问题。
# 还有，在多app应用时，可以实现数据隔离；列表里不会加数据，而是会生成一个新的列表
# local是一个字典，字典里key（stack）是唯一标识，value是一个列表
```

#### 8.Flask中多app应用是怎么完成？

```
请求进来时，可以根据URL的不同，交给不同的APP处理。蓝图也可以实现。
    #app1 = Flask('app01')
    #app2 = Flask('app02')
    #@app1.route('/index')
    #@app2.route('/index2')
源码中在DispatcherMiddleware类里调用app2.__call__，
原理其实就是URL分割，然后将请求分发给指定的app。
之后app也按单app的流程走。就是从app.__call__走。
```

#### 9.解释Flask框架中的Local对象和threading.local对象的区别？

```python
# a.threading.local
作用：为每个线程开辟一块空间进行数据存储(数据隔离)。

问题：自己通过字典创建一个类似于threading.local的东西。
storage = {
   4740: {val: 0},
   4732: {val: 1},
   4731: {val: 3},
   }

# b.自定义Local对象
作用：为每个线程(协程)开辟一块空间进行数据存储(数据隔离)。
class Local(object):
   def __init__(self):
      object.__setattr__(self, 'storage', {})
   def __setattr__(self, k, v):
      ident = get_ident()
      if ident in self.storage:
         self.storage[ident][k] = v
      else:
         self.storage[ident] = {k: v}
   def __getattr__(self, k):
      ident = get_ident()
      return self.storage[ident][k]
obj = Local()
def task(arg):
   obj.val = arg
   obj.xxx = arg
   print(obj.val)
for i in range(10):
   t = Thread(target=task, args=(i,))
   t.start()
```

#### 10.Flask中 blinker 是什么？

```
# flask中的信号blinker
信号主要是让开发者可是在flask请求过程中定制一些行为。
或者说flask在列表里面预留了几个空列表，在里面存东西。
简言之，信号允许某个'发送者'通知'接收者'有事情发生了
@ before_request有返回值，blinker没有返回值

# 10个信号

request_started = _signals.signal('request-started') #请求到来前执行

request_finished = _signals.signal('request-finished') #请求结束后执行

before_render_template = _signals.signal('before-render-template')#模板渲染前执行

template_rendered = _signals.signal('template-rendered')#模板渲染后执行

got_request_exception = _signals.signal('got-request-exception') #请求执行出现异常时执行

request_tearing_down = _signals.signal('request-tearing-down')#请求执行完毕后自动执行（无论成功与否）

appcontext_tearing_down = _signals.signal('appcontext-tearing-down')# 请求上下文执行完毕后自动执行（无论成功与否）

appcontext_pushed = _signals.signal('appcontext-pushed') #请求app上下文push时执行

appcontext_popped = _signals.signal('appcontext-popped') #请求上下文pop时执行

message_flashed = _signals.signal('message-flashed')#调用flask在其中添加数据时，自动触发
```









### 爬虫与celery

#### 1.爬虫中已经执行过的任务，如何不再执行？

```

```

#### 2.防爬虫策略？

```

```

#### 3.爬虫并发量？

```

```

#### 4.爬虫去重 布隆过滤器？

```

```

#### 5.简述 requests模块的作用及基本使用？

```
# 作用：
使用requests可以模拟浏览器的请求
# 常用参数：
   url、headers、cookies、data
   json、params、proxy
# 常用返回值：
   content
   iter_content
   text 
   encoding="utf-8"
   cookie.get_dict()
```

#### 6.简述 beautifulsoup模块的作用及基本使用？

```
# BeautifulSoup
用于从HTML或XML文件中提取、过滤想要的数据形式
#常用方法
解析：html.parser 或者 lxml（需要下载安装） 
   find、find_all、text、attrs、get 
```

#### 7.简述 seleninu模块的作用及基本使用?

```
Selenium是一个用于Web应用程序测试的工具，
他的测试直接运行在浏览器上，模拟真实用户，按照代码做出点击、输入、打开等操作

爬虫中使用他是为了解决requests无法解决javascript动态问题
```

#### 8.scrapy框架中各组件的工作流程？

```
#1、生成初始的Requests来爬取第一个URLS，并且标识一个回调函数
第一个请求定义在start_requests()方法内默认从start_urls列表中获得url地址来生成Request请求，
默认的回调函数是parse方法。回调函数在下载完成返回response时自动触发
#2、在回调函数中，解析response并且返回值
返回值可以4种：
    a、包含解析数据的字典
    b、Item对象
    c、新的Request对象（新的Requests也需要指定一个回调函数）
    d、或者是可迭代对象（包含Items或Request）
#3、在回调函数中解析页面内容
通常使用Scrapy自带的Selectors，但很明显你也可以使用Beutifulsoup，lxml或其他你爱用啥用啥。
#4、最后，针对返回的Items对象将会被持久化到数据库
    通过Item Pipeline组件存到数据库
    或者导出到不同的文件（通过Feed exports）
http://www.cnblogs.com/wupeiqi/articles/6229292.html
```

#### 9.scrapy框架中如何实现大文件的下载？

```python
from twisted.web.client import Agent, getPage, ResponseDone, PotentialDataLoss
from twisted.internet import defer, reactor, protocol
from twisted.web._newclient import Response
from io import BytesIO

class _ResponseReader(protocol.Protocol):
    def __init__(self, finished, txresponse, file_name):
        self._finished = finished
        self._txresponse = txresponse
        self._bytes_received = 0
        self.f = open(file_name, mode='wb')
    def dataReceived(self, bodyBytes):
        self._bytes_received += len(bodyBytes)
        # 一点一点的下载
        self.f.write(bodyBytes)
        self.f.flush()
    def connectionLost(self, reason):
        if self._finished.called:
            return
        if reason.check(ResponseDone):
            # 下载完成
            self._finished.callback((self._txresponse, 'success'))
        elif reason.check(PotentialDataLoss):
            # 下载部分
            self._finished.callback((self._txresponse, 'partial'))
        else:
            # 下载异常
            self._finished.errback(reason)
        self.f.close()
```

#### 10.scrapy中的pipelines工作原理？

```
Scrapy 提供了 pipeline 模块来执行保存数据的操作。
在创建的 Scrapy 项目中自动创建了一个 pipeline.py 文件，同时创建了一个默认的 Pipeline 类。
我们可以根据需要自定义 Pipeline 类，然后在 settings.py 文件中进行配置即可
```

#### 11.celery是什么以及应用场景？

```
# Celery是由Python开发的一个简单、灵活、可靠的处理大量任务的分发系统，
# 它不仅支持实时处理也支持任务调度。
```

#### 12.celery如何实现定时任务？

```python
# celery实现定时任务
启用Celery的定时任务需要设置CELERYBEAT_SCHEDULE 。 
CELERYBEAT_SCHEDULE='djcelery.schedulers.DatabaseScheduler'#定时任务
'创建定时任务'
# 通过配置CELERYBEAT_SCHEDULE：
#每30秒调用task.add
from datetime import timedelta
CELERYBEAT_SCHEDULE = {
    'add-every-30-seconds': {
        'task': 'tasks.add',
        'schedule': timedelta(seconds=30),
        'args': (16, 16)
    },
}
```

#### 13.简述 celery多任务结构目录

```
pro_cel
    ├── celery_tasks     # celery相关文件夹
    │   ├── celery.py    # celery连接和配置相关文件
    │   └── tasks.py     #  所有任务函数
    ├── check_result.py  # 检查结果
    └── send_task.py     # 触发任务
```

### Git

#### 1.git如何解决冲突？

```

```

#### 2.git如何做review？

```
1、你们公司的代码review分支怎么做？谁来做？
答：组长创建review分支，我们小功能开发完之后，合并到review分支交给老大（小组长）来看，
1.1、你组长不开发代码吗？
        他开发代码，但是它只开发核心的东西，任务比较少。
        或者抽出时间，我们一起做这个事情
2、你们公司协同开发是怎么协同开发的？
每个人都有自己的分支，阶段性代码完成之后，合并到review，然后交给老大看
```

https://blog.csdn.net/june_y/article/details/50817993

#### 3.git reabase的作用？

```
merge：
会将不同分支的提交合并成一个新的节点，之前的提交分开显示，
注重历史信息、可以看出每个分支信息，基于时间点,遇到冲突,手动解决,再次提交
rebase：
将两个分支的提交结果融合成线性，不会产生新的节点;
注重开发过程，遇到冲突，手动解决，继续操作
```

#### 4.git常用的命令

```
# git init
    初始化，当前所在的文件夹可以被管理且以后版本相关的数据都会存储到.git文件中
# git status
    查看当前文件夹以及子目录中文件是否发生变化：
    内容修改/新增文件/删除，已经变化的文件会变成红色，已经add的文件会变成绿色
# git add .
    给发生变化的文件（贴上一个标签）或 将发生变化的文件放到某个地方，
    只写一个句点符就代表把git status中红色的文件全部打上标签
# git commit -m
    新增用户登录认证功能以及xxx功能将“绿色”文件添加到版本中
# git log
    查看所有版本提交记录，可以获取版本号
# git reset --hard 版本号   
    将最新的版本回退到更早的版本
# git reflog   
    回退到之前版本后悔了，再更新到最新或者最新之前的版本
# git reset --hard 版本 回退
```

#### 5.公司如何基于git做协同开发？

```
# 大致工作流程
公司：
    下载代码
        git clone https://gitee.com/wupeiqi/xianglong.git
        或创建目录 
        cd 目录 
        git init 
        git remote add origin https://gitee.com/wupeiqi/xianglong.git
        git pull origin maste 
    创建dev分支
        git checkout dev 
        git pull origin dev 
        继续写代码
        git add . 
        git commit -m '提交记录'
        git push origin dev 
回家： 
    拉代码：
        git pull origin dev 
    继续写：
        继续写代码
        git add . 
        git commit -m '提交记录'
        git push origin dev
```

#### 6.如何为github开源代码贡献自己的代码

```
1、fork需要协作项目
2、克隆/关联fork的项目到本地
3、新建分支（branch）并检出（checkout）新分支
4、在新分支上完成代码开发
5、开发完成后将你的代码合并到master分支
6、添加原作者的仓库地址作为一个新的仓库地址
7、合并原作者的master分支到你自己的master分支,用于和作者仓库代码同步
8、push你的本地仓库到GitHub
9、在Github上提交 pull requests
10、等待管理员（你需要贡献的开源项目管理员）处理
```

#### 7.什么是敏捷开发

```
'敏捷开发'：是一种以人为核心、迭代、循序渐进的开发方式。

它并不是一门技术，而是一种开发方式，也就是一种软件开发的流程。
它会指导我们用规定的环节去一步一步完成项目的开发。
因为它采用的是迭代式开发，所以这种开发方式的主要驱动核心是人
```

### linux部分

#### 1.讲讲储常用的linux/git的命令和作用？

```

```

#### 2.什么是cdn？

```
目的是使用户可以就近到服务器取得所需内容，解决 Internet网络拥挤的状况，提高用户访问网站的响应速度。

cdn 即内容分发网络
```

#### 3.LVS是什么及作用？

```
LVS ：Linux虚拟服务器
作用：LVS主要用于多服务器的负载均衡。
它工作在网络层，可以实现高性能，高可用的服务器集群技术。它廉价，可把许多低性能的服务器组合在一起形成一个超级服务器。它易用，配置非常简单，且有多种负载均衡的方法。它稳定可靠，即使在集群的服务器中某台服务器无法正常工作，也不影响整体效果。另外可扩展性也非常好。
```

#### 4.Nginx是什么及作用？

```

```

####  5.keepalived是什么及作用?

```

```

####  6.haproxy是什么以及作用？

```

```

#### 7.什么是负载均衡？

```

```

#### 8.什么是rpc及应用场景？

```

```

### 其他

#### 1.微服务的问题

怎么理解微服务，服务如何划分，可以从哪几个方面去划分，为什么这样划分，微服务带来了哪些好处，哪些坏处，如何看待这个问题？

```

```

你对devops的了解？

```

```

#### 2.如何理解网关，网关带来的好处和坏处，如何解决

```

```

#### 3.设计模式

**掌握哪些设计模式，常用哪些，项目中如何使用的，为什么用这个，不用那个？手写一个线程安全的单例模式**

```

```

#### 4.讲一讲对Redis 的了解，项目中如何使用的，哪个地方使用的，为什么要使用？

```

```

#### 5.哨兵机制、Redis 两种备份方式的区别，项目中用的哪种，为什么？

```

```

#### 6.讲一讲对分布式锁的了解

```

```

#### 7.项目中系统监控怎么做的？

```

```

#### 8.Kafka问题

Kafka 如何保证消息顺序消费、在consumer group 中新增一个consumer 会提高消费消息的速度吗、那如果我想提高消息消费的速度，我要怎么办？

```

```

#### 9.你看过flask的源码吗？你是如何理解开源的？

```

```

#### 10.B Tree和B+ Tree的区别？

```
1.B树中同一键值不会出现多次，并且有可能出现在叶结点，也有可能出现在非叶结点中。

而B+树的键一定会出现在叶结点中，并有可能在非叶结点中重复出现，以维持B+树的平衡。
2.因为B树键位置不定，且在整个树结构中只出现一次，
```

#### 11.解释 PV、UV 的含义？

```
PV访问量（Page View），即页面访问量，每打开一次页面PV计数+1，刷新页面也是。
UV访问数（Unique Visitor）指独立访客访问数，一台电脑终端为一个访客。
```

#### 12.解释 QPS的含义？

```
'QPS(Query Per Second)'
每秒查询率，是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准
```

#### 13.什么是反向代理？

```
正向代理代理客户端(客户端找哟个代理去访问服务器，服务器不知道你的真实IP)
反向代理代理服务器(服务器找一个代理给你响应，你不知道服务器的真实IP)
```

### 项目

#### 1.聊项目

画项目架构图，画一个用户从发起请求到接收到响应，中间经过哪些服务，每个服务做什么事情的流程图，讲数据库设计具体到部分表中有哪些字段？

```

```

#### 2.crm中权限组件的实现流程？

```

```

#### 3.权限组件的实现流程？

```

```

#### 4.如何实现粒度控制到按钮？

```

```

#### 5.权限信息为什么放在session中？有什么不好？

```

```