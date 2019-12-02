---
title: flask入门初始
id: 1
date: 2019-11-15 20:00:00
tags: flask
toc: true
comment: true
---

**简介：**

Flask是一个基于Python开发并且依赖jinja2模板和Werkzeug WSGI服务的一个微型框架。

Micro（微）：**微的旨在保持核心简单而易于拓展**；众多的扩展提供了数据库集成、表单验证、上传处理、各种各样的开放认证技术等功能。

虚拟环境：virtualenv，为什么要采用虚拟环境呢？当我们的项目越来越多的时候，同时拥有不同版本的python工作的可能性就越大，常常会有库（内，外部库）破坏向后兼容性，

当出现两个或多个依赖性冲突时，我们该怎么做？肯定是产生虚拟环境，给每一个项目提供一份虚拟的环境（不是副本），我们在使用指定的项目的时候就可以激活相应的环境。

依赖的两个外部库：Jinja2（http://jinja.pocoo.org/docs/2.10/）和Werkzeug WSGI（https://werkzeug.palletsprojects.com/en/0.15.x/）工具集

 <!--more-->

**简单入门：**

- 调试模式：这个架构和django不同的是，如果不进行设置，服务器不能自动更新写入的代码，因此我们需要这样做：app.run(debug=True)

- 路由：[利用route()装饰器把一个函数绑定到对应的URL上》》@app.route(‘/hello’)](mailto:利用route()装饰器把一个函数绑定到对应的URL上》》@app.route(‘/hello’))

  变量规则：@app.route(‘/user/<username>’)  @app.route(‘/post/<int:post_id>’) ：后面接收的可以是字符串和整型两种方式

  重定向：@app.route(‘/user/’)    @app.route(‘/user’)  这是两种。访问一个结尾不带斜线的URL时会被flask重定向到带斜线的URL上面去；如果没有页面就会返回一个404错误。所以说我们一般还是带斜线的好。

  构造URL：也就是生成URL，通过url_for来给指定的函数构造URL，

  With app.test_request_context(): url_for(func,username)

  HTTP 方法：[route（）装饰器可以传递methods参数@app.route(‘/login’,methods=[‘GET’,’POST’\])](mailto:route（）装饰器可以传递methods参数@app.route(‘/login’,methods=[‘GET’,’POST’]))

  GET：浏览器告诉服务器，只获取页面上的信息并发给我。最常用

  POST：浏览器告诉服务器：想在URL上发布新消息，并且服务器必须确保数据已经储存且仅储存一次。这个是HTML表单通常发送数据到服务器的方法

  PUT：类似POST，但是服务器可能触发了多次储存过程，多次覆盖掉旧值。

  DELETE：删除给定位置的信息

  静态文件：通常是css/js文件。给静态文件生成URL：url_for(‘static’,filrname=’style.css’)

- 模板渲染：flask配备了Jinja2作为渲染引擎，可以使用render_template()方法来渲染模板。只需要return render_template(‘hello.html’,name=name):后面的目录的名称

- 访问请求数据：客服端和服务器的数据交互非常的重要，当然是通过request实现的。

  那么request就是全局变量，是否安全呢？

  环境局部变量：

  请求对象：

  文件上传：

  Cookies：详细的自己看

- 重定向和错误：通过redirect()函数把用户重定向到其他的地方。当放弃请求并返回错误代码，用abort()函数  return redirect(url_for(‘login’))    abort(401)

- 关于响应：视图函数的返回值会被自动转换成一个响应对象。

- 会话：两个对象，请求对象和session对象。它允许你在不同请求间存储特定用户的信息。它是在 Cookies 的基础上实现的，并且对 Cookies 进行密钥签名。这意味着用户可以查看你 Cookie 的内容，但却不能修改它，除非用户知道签名的密钥。生成密钥：import os os.urandom(24)

- 消息闪现：反馈，是良好的应用和用户界面的重要构成。如果用户得不到足够的反馈，他们很可能开始厌恶这个应用。 Flask 提供了消息闪现系统，可以简单地给用户反馈。 消息闪现系统通常会在请求结束时记录信息，并在下一个（且仅在下一个）请求中访问记录的信息。展现这些消息通常结合要模板布局。使用 [flash()](#flask.flash) 方法可以闪现一条消息。要操作消息本身，请使用[get_flashed_messages()](#flask.get_flashed_messages) 函数，并且在模板中也可以使用
- 日志记录：logger是一个标准的日志类，用于查看日志信息。App.logger.warning() 。。。

- 整合WSGI中间件：

- 部署到WEB服务器：在Heroku/dotCloud上面免费托管

 

**教程（完成一个博客的简单开发）**

介绍flaskr：就是一个登入/登出》发布消息前端显示的页面

创建文件夹：利用pycharm自动生成

数据库模式：使用自带的sqlist数据库

应用设置代码：flaskr.py功能代码块

数据库连接：数据库连接和关闭，本身不是很有用的而且是低效的，所以我们需要让连接保持更长的时间，因为数据库连接封装了事务，我们也需要确保在同一时刻只有一个请求使用这个连接。我们怎么去实现？在flask中提供了两种环境（Request Context（请求环境）和Application Context（应用环境））

创建数据库：init_db（）

视图函数：在flaskr中利用函数实现

模板：添加登陆界面的html，登陆之后的后台的html，发布界面的html

添加样式：装饰html的css

应用测试：下次接着看

 