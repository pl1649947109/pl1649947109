---
title: 第一讲——Django之知识带入
id: 1
date: 2019-9-24 20:00:00
tags: Django
comment: true
---

很简洁的一个学习dj的网站：https://code.ziqiangxuetang.com/django/django-admin.html

### HTTP协议相关

#### HTTP协议简介

HTTP，超文本传输协议（HyperText Transfer Protocol)是互联网上应用最为广泛的 一种网络协议。所有的WWW文件都必须遵守这个标准。设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法，HTTP是万维网的数据通信的基础。**现在应用最广泛的是HTTP1.1版本**

#### HTTP工作流程

HTTP是一个客户端终端（用户）和服务器端（网站）请求和应答的标准（TCP）。通过使用网页浏览器、网络爬虫或者其它的工具，客户端发起一个HTTP请求到服务器上指定端口（默认端口为80）。我们称这个客户端为用户代理程序（user agent）。应答的服务器上存储着一些资源，比如HTML文件和图像。我们称这个应答服务器为源服务器（origin server）。在用户代理和源服务器中间可能存在多个“中间层”，比如代理服务器、网关或者隧道（tunnel）。

<!----more---->

注意：

- 这个协议在TCP/IP协议栈的应用层，因此我们无需操心HTTP是如何传输的，只需要关心我们传输的内容是否正确的被接收端识别。
- HTTP是基于TCP实现的，简单的说就是TCP协议负责可靠的内容传输，HTTP协议负责识别内容，两者不在一个层面上。
- HTTP无状态的含义是每一次的内容解析是没有关联的。TCP有状态连接是指两端在连接过程的。
- HTTP包含两种报文信息：请求报文和响应报文。

#### HTTP的工作原理

HTTP协议定义Web客户端如何从Web服务器请求Web页面，以及服务器如何把Web页面传送给客户端。HTTP协议采用了请求/响应模型。客户端向服务器发送一个请求报文，请求报文包含**请求的方法、URL、协议版本、请求头部和请求数据**。服务器以一个状态行作为响应，响应的内容包括**协议的版本、成功或者错误代码、服务器信息、响应头部和响应数据。**

以下是 HTTP 请求/响应的步骤：

1. 客户端连接到Web服务器
   一个HTTP客户端，通常是浏览器，与Web服务器的HTTP端口（默认为80）建立一个TCP套接字连接。例如，http://www.luffycity.com。
2. 发送HTTP请求
   通过TCP套接字，客户端向Web服务器发送一个文本的请求报文，一个请求报文由请求行、请求头部、空行和请求数据4部分组成。
3. 服务器接受请求并返回HTTP响应
   Web服务器解析请求，定位请求资源。服务器将资源复本写到TCP套接字，由客户端读取。一个响应由状态行、响应头部、空行和响应数据4部分组成。
4. 释放连接TCP连接
   若connection 模式为close，则服务器主动关闭TCP连接，客户端被动关闭连接，释放TCP连接;若connection 模式为keepalive，则该连接会保持一段时间，在该时间内可以继续接收请求;
5. 客户端浏览器解析HTML内容
   客户端浏览器首先解析状态行，查看表明请求是否成功的状态代码。然后解析每一个响应头，响应头告知以下为若干字节的HTML文档和文档的字符集。客户端浏览器读取响应数据HTML，根据HTML的语法对其进行格式化，并在浏览器窗口中显示。

例如：在浏览器地址栏键入URL，按下回车之后会经历以下流程：

1. 浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址;
2. 解析出 IP 地址后，根据该 IP 地址和默认端口 80，和服务器建立TCP连接;
3. 浏览器发出读取文件(URL 中域名后面部分对应的文件)的HTTP 请求，该请求报文作为 TCP 三次握手的第三个报文的数据发送给服务器;
4. 服务器对浏览器请求作出响应，并把对应的 html 文本发送给浏览器;
5. 释放 TCP连接;
6. 浏览器将该 html 文本并显示内容; 

#### HTTP的特点　

- 基于 请求-响应 的模式：HTTP协议规定,请求从客户端发出,最后服务器端响应该请求并 返回。
- 无状态保存：HTTP协议 自身不对请求和响应之间的通信状态进行保存。也就是说在HTTP这个 级别,协议对于发送过的请求或响应都不做持久化处理，每次到来的请求都是一个新请求。
- 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。

#### HTTP请求方法

根据HTTP标准，HTTP请求可以使用多种请求方法。

HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法。

**注： 有请求体(请求体有数据)肯定是post提交(表单method是post)** 

HTTP1.1新增了五种请求方法：OPTIONS, PUT, DELETE, TRACE 和 CONNECT 方法。

| 序号 | 方法    | 描述                                                         |
| ---- | ------- | ------------------------------------------------------------ |
| 1    | GET     | 请求指定的页面信息，并返回实体主体。使用GET方法应该只用在读取数据。 |
| 2    | HEAD    | 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头 |
| 3    | POST    | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改，或二者皆有。 |
| 4    | PUT     | 从客户端向服务器传送的数据取代指定的文档的内容。             |
| 5    | DELETE  | 请求服务器删除Request-URI所标识的资源。                      |
| 6    | CONNECT | HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。     |
| 7    | OPTIONS | 允许客户端查看服务器的性能。                                 |
| 8    | TRACE   | 回显服务器收到的请求，主要用于测试或诊断。                   |

**对比GET和POST请求：**

- GET提交的数据会放在URL之后，也就是请求行里面，以?分割URL和传输数据，参数之间以&相连，如EditBook?name=test1&id=123456.（请求头里面那个content-type做的这种参数形式，后面讲）；POST方法是把提交的数据放在HTTP包的请求体中。
- GET提交的数据大小有限制（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制。

#### HTTP状态码

当浏览者访问一个网页时，浏览者的浏览器会向网页所在服务器发出请求。当浏览器接收并显示网页前，此网页所在的服务器会返回一个包含HTTP状态码的信息头（server header）用以响应浏览器的请求。

下面是常见的HTTP状态码：

- 200 - 请求成功
- 301 - 资源（网页等）被永久转移到其它URL
- 404 - 请求的资源（网页等）不存在
- 500 - 内部服务器错误

#### HTTP状态码分类

HTTP状态码由三个十进制数字组成，第一个十进制数字定义了状态码的类型，后两个数字没有分类的作用。HTTP状态码共分为5种类型：

| 分类 | 分类描述                                       |
| ---- | ---------------------------------------------- |
| 1**  | 信息，服务器收到请求，需要请求者继续执行操作   |
| 2**  | 成功，操作被成功接收并处理                     |
| 3**  | 重定向，需要进一步的操作以完成请求             |
| 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
| 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

HTTP状态码列表:

| 状态码 | 状态码英文名称                  | 中文描述                                                     |
| ------ | ------------------------------- | ------------------------------------------------------------ |
| 100    | Continue                        | 继续。[客户端](http://www.dreamdu.com/webbuild/client_vs_server/)应继续其请求 |
| 101    | Switching Protocols             | 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议 |
|        |                                 |                                                              |
| 200    | OK                              | 请求成功。一般用于GET与POST请求                              |
| 201    | Created                         | 已创建。成功请求并创建了新的资源                             |
| 202    | Accepted                        | 已接受。已经接受请求，但未处理完成                           |
| 203    | Non-Authoritative Information   | 非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本 |
| 204    | No Content                      | 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档 |
| 205    | Reset Content                   | 重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域 |
| 206    | Partial Content                 | 部分内容。服务器成功处理了部分GET请求                        |
|        |                                 |                                                              |
| 300    | Multiple Choices                | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择 |
| 301    | Moved Permanently               | 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替 |
| 302    | Found                           | 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI |
| 303    | See Other                       | 查看其它地址。与301类似。使用GET和POST请求查看               |
| 304    | Not Modified                    | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
| 305    | Use Proxy                       | 使用代理。所请求的资源必须通过代理访问                       |
| 306    | Unused                          | 已经被废弃的HTTP状态码                                       |
| 307    | Temporary Redirect              | 临时重定向。与302类似。使用GET请求重定向                     |
|        |                                 |                                                              |
| 400    | Bad Request                     | 客户端请求的语法错误，服务器无法理解                         |
| 401    | Unauthorized                    | 请求要求用户的身份认证                                       |
| 402    | Payment Required                | 保留，将来使用                                               |
| 403    | Forbidden                       | 服务器理解请求客户端的请求，但是拒绝执行此请求               |
| 404    | Not Found                       | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面 |
| 405    | Method Not Allowed              | 客户端请求中的方法被禁止                                     |
| 406    | Not Acceptable                  | 服务器无法根据客户端请求的内容特性完成请求                   |
| 407    | Proxy Authentication Required   | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权 |
| 408    | Request Time-out                | 服务器等待客户端发送的请求时间过长，超时                     |
| 409    | Conflict                        | 服务器完成客户端的PUT请求是可能返回此代码，服务器处理请求时发生了冲突 |
| 410    | Gone                            | 客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置 |
| 411    | Length Required                 | 服务器无法处理客户端发送的不带Content-Length的请求信息       |
| 412    | Precondition Failed             | 客户端请求信息的先决条件错误                                 |
| 413    | Request Entity Too Large        | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息 |
| 414    | Request-URI Too Large           | 请求的URI过长（URI通常为网址），服务器无法处理               |
| 415    | Unsupported Media Type          | 服务器无法处理请求附带的媒体格式                             |
| 416    | Requested range not satisfiable | 客户端请求的范围无效                                         |
| 417    | Expectation Failed              | 服务器无法满足Expect的请求头信息                             |
|        |                                 |                                                              |
| 500    | Internal Server Error           | 服务器内部错误，无法完成请求                                 |
| 501    | Not Implemented                 | 服务器不支持请求的功能，无法完成请求                         |
| 502    | Bad Gateway                     | 充当网关或代理的服务器，从远端服务器接收到了一个无效的请求   |
| 503    | Service Unavailable             | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中 |
| 504    | Gateway Time-out                | 充当网关或代理的服务器，未及时从远端服务器获取请求           |
| 505    | HTTP Version not supported      | 服务器不支持请求的HTTP协议的版本，无法完成处理               |

#### HTTP请求格式

客户端发送一个HTTP请求到服务器的请求消息包括以下格式：请求行（request line）、请求头部（header）、空行和请求数据四个部分组成，下图给出了请求报文的一般格式。

![img](https://images2018.cnblogs.com/blog/867021/201803/867021-20180322001733298-201433635.jpg)

| 常见请求头        | 描述 （红色掌握，其他了解）                                  |
| ----------------- | ------------------------------------------------------------ |
| Referer           | 浏览器通知服务器，当前请求来自何处。如果是直接访问，则不会有这个头。常用于：防盗链 |
| If-Modified-Since | 浏览器通知服务器，本地缓存的最后变更时间。与另一个响应头组合控制浏览器页面的缓存。 |
| **Cookie**        | 与会话有关技术，用于存放浏览器缓存的cookie信息。             |
| **User-Agent**    | 浏览器通知服务器，客户端浏览器与操作系统相关信息             |
| Connection        | 保持连接状态。Keep-Alive 连接中，close 已关闭                |
| Host              | 请求的服务器主机名                                           |
| Content-Length    | 请求体的长度                                                 |
| Content-Type      | 如果是POST请求，会有这个头，默认值为application/x-www-form-urlencoded，表示请求体内容使用url编码 |
| Accept：          | 浏览器可支持的MIME类型。文件类型的一种描述方式。MIME格式：大类型/小类型[;参数]例如：   text/html ，html文件   text/css，css文件   text/javascript，js文件   image/*，所有图片文件 |
| Accept-Encoding   | 浏览器通知服务器，浏览器支持的数据压缩格式。如：GZIP压缩     |
| Accept-Language   | 浏览器通知服务器，浏览器支持的语言。各国语言（国际化i18n）   |

#### HTTP请求头

请求头中主要包含本次请求的附加消息，其中长用的字段有：

Accept：指定客户端能够接收的内容类型

Accept-Encoding:指定浏览器可以支持的web服务器返回内容压缩编码类型

Accept-Language:浏览器可接受的语言

Content-Length:请求的内容长度

Content-Type:请求的与实体对应的MIME信息

Date：请求发送的日期和时间

详细的请求头的所有完整内容请浏览：

**http://tools.jb51.net/table/http_header**

#### 响应报文

1. 响应行：包涵HTTP的版本和本次请求的状态，就是响应码。
2. 响应头：用于描述服务器的基本信息和数据。
   - Allow 服务器支持哪些请求方法（如GET、POST等）
   - Content-Encoding 文档的编码（Encode）方法。
   - Content-Length 表示内容长度。
   - Content-Type 表示后面的文档属于什么MIME类型。
3. 响应实体：包涵的就是客户端从服务器中获取的数据了，数据的格式和长度都会在响应体头中描述。

详细的请求头的所有完整内容请浏览：**http://tools.jb51.net/table/http_header**

### 对比几种python框架

Django，发布于2003年，最初被用来制作在线新闻的Web站点。Django的各模块之间结合的比较紧密，多一在功能强大的同时又是一个相对封闭的系统。

Thrnado：一个强大的、支持协程、高效并发且可扩展的Web服务器，发布于2009年，它的强项在于可以利用异步协程机制实现高并发的服务。

Flask：Python Web家族里面比较年轻的一个，发布于2010年，它吸收了其他框架的优点并且把自己的主要领域定义在微小的项目上，以短小精炼，简洁明了著称。

Twisted：一个有着10多年历史的开源事件驱动框架，它不像前面的三种着眼于Web应用开发，而是适用从传输层到自定义应用协议的所有类型的网络程序开发，并能在不同的操作系统上提供很高的运行效率，但是对python3的支持有限。

### 基于python的web开发技术栈

![](http://9017499461.linshutu.top/%E5%9F%BA%E4%BA%8Epython%E7%9A%84web%E5%BC%80%E5%8F%91%E6%8A%80%E6%9C%AF%E6%A0%88.png)

### 一般web框架的架构图

![](http://9017499461.linshutu.top/%E4%B8%80%E8%88%AC%E7%9A%84Web%E6%A1%86%E6%9E%B6%E7%9A%84%E6%9E%B6%E6%9E%84.png)



### django的特点

功能齐全，完善的文档，强大的数据库访问组件（model的ORM组件），灵活的URL映射，丰富的template模板语言（类似jinja2模板语言），自带免费的后台管理系统，完整的错误信息提示。

### MVC和MTV开发模式

**MVC（model-view-contoller）：也被称作软件架构（flask使用该模式）**

M：数据存取部分，由django数据层处理

V：选择显示哪些数据要以及怎么显示，由视图和模板处理(js/html/css)

C：根据用户输入委派视图的部分，由django框架通过URLconf设置，对给定URL调用合适的python函数来处理进行(相当于mtv的view)

**MTV开发模式（model-template-view）：Django使用该模式**

M 代表模型（Model），即数据存取层。该层处理与数据相关的所有事务：如何存取、如何确认有效性、包含哪些行为以及数据之间的关系等。

T 代表模板(Template)，即表现层。该层处理与表现相关的决定：如何在页面或其他类型文档中进行显示。

V 代表视图（View），即业务逻辑层。该层包含存取模型及调取恰当模板的相关逻辑。你可以把它看作模型与模板之间的桥梁。

### Django的MTV模型组织

![](http://9017499461.linshutu.top/Django%E7%9A%84MTV%E6%A8%A1%E5%9E%8B%E7%BB%84%E7%BB%87.png)

### 常用的命令

```python
键入：python manage.py 
django-admin startproject projectname 创建django程序
startapp appname  创建app
runserver  启动django服务
runserver 8001 当提示端口被占用时，我们可以启用其他的端口
runserver 0.0.0.0:8000 监听机器的所有ip

**（下面两个是配套使用的命令 ，先执行前面一个再执行后面一个）** makemigrations   #检测model.py下的类是否有改动
migrate    #将model.py下的代码翻译成sql语句

flush  清空数据库
createsuperuser 创建超级管理员（用户名和密码必填）
changepassword username 修改用户密码
shell 启动项目环境终端（就是ipython或者bpython）
dbshell 自动进入settings配置的数据库，在这个终端可以执行数据库的sql语句

在终端输入python manage.py 可以查看更多的命令
```

### Django处理请求的本质

简单一点就是写视图函数views并用URLconfs把他们和URLs链接起来。详细：当server收到一个Http请求以后，server特定的handler会创建HTTPRequest并传递给下一个组件并处理。这个handler然后会调用所有可用的Request或者View中间件。只要中间件返回一个HTTPRequest，系统就跳过对视图的处理。如果视图函数抛出异常，控制器就会传递给异常处理中间件处理，如果这个中间件没有返回HTTPRequest，那么就不能处理这个异常，这个时候Django包含缺省的视图生成友好的404和500的（request）.

### Django功能文件解析

Manage.py:一种命令执行工具，可使我们以多种方式与Django项目进行交互

Settings.py:django项目的设置和配置，数据库配置，app配置，静态文件配置等等都在这里面完成

urls.py:django项目的URL的声明，也就是django所支撑站点的内容列表，就是网站的入口。

Templates：存放网页模板html、css和js等

__init__.py：启动应用所需的处理都在这里，比如应用的入口，model的导入，数据库连接的初始化，设置文件的读取等，此外，文件的调用必须在package里面有这个文件。

wsgi.py:一个基于WSGI的web服务器进入点，提供底层的网络通信功能通常不用关心，因为人家已经帮我们做好了

### URL 配置和松耦合

松耦合就是两段代码是这种关系，当我们改变其中一个代码的时候，不会影响另外的一段代码，或者只是少许的影响。在Django中视图函数和URL就是这种关系。其他的使用紧耦合的对后期的维护与改动非常的不利。其实在这个架构中使用这种耦合的方式很多地方都有体现。

### Django的模板系统

Python代码编写和HTML设计是两项不同的工作。这样讲这两项工作分离开进行就会是我们的工作开展的更快。因此就需要我们的系统模板：

基础知识：概念：Django模板用于分割文档的表示（presentation）和数据（data）的字符串文本。

模板包括：占位符（placeholders）和各种定义文档应该如何显示的基本逻辑（即模板标签，template tag）

在python代码中使用系统模板，只需要遵循以下两个步骤：

1，使用原始的模板代码字符串创建一个Template对象；

2，调用Template对象的render（）方法并提供给他变量（i.e.,内容）。它将返回一个完整的模板字符串，包含了所有标签块和变量解析后的内容

### 自动化测试

为什么我会把自动化测试放在这里，因为，我们在这里需要有一个概念：当我们的一个项目搭建好了之后并不意味着任务的完结，反而需要各种方法去检测我们的代码和逻辑。

测试是一种例行的、不可缺失的工作，用于检查你的程序是否符合预期。

测试可以分为手动测试和自动测试。手动测试很常见，有时候print一个变量内容，都可以看做是测试的一部分。手动测试往往很零碎、不成体系、不够完整、耗时费力、效率低下，测试结果也不一定准确。

自动化测试则是系统地较为完整地对程序进行测试，效率高，准确性高，并且大部分共同的测试工作会由系统来帮你完成。一旦你创建了一组自动化测试程序，当你修改了你的应用，你就可以用这组测试程序来检查你的代码是否仍然同预期的那样运行，而无需执行耗时的手动测试。

创建一个测试实例：

```python
import datetime
from django.utils import timezone
from django.test import TestCase
from .models import Question

class QuestionMethodTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        在将来发布的问卷应该返回False
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```

运行测试程序：

在终端运行：

```python
python manage.py test app01
```

运行这行代码都发生了什么？

- 首先执行该命令之后，回去查找app01应用中所有的测试程序
- 发现一个django.test.TestCase的子类
- 为测试创建一个专用的数据库
- 查找test开头的测试方法
- 在`test_was_published_recently_with_future_question`方法中，创建一个Question实例，该实例的pub_data字段的值是30天后的未来日期。
- 然后利用assertIs()方法，它发现`was_published_recently()`返回了True，而不是我们希望的False。
- 最后，测试程序会通知我们哪个测试失败了，错误出现在哪一行

### 自定义Admin后台管理站点

jango的admin站点是自动生成的、高度可定制的，它是Django相较其它Web框架独有的内容，广受欢迎。如果你觉得它不够美观，还有第三方美化版**xadmin**。请一定不要忽略它，相信我，**它值得拥有**！

简单的配置：http://www.liujiangblog.com/course/django/93