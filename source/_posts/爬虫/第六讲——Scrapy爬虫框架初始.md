---
title: 第六讲——Scrapy爬虫框架初始
id: 6
date: 2019-9-20 20:30:00
tags: 爬虫
comment: true
---

##### 简介

- 什么是框架？
  - 所谓的框架简单通用解释就是就是一个具有很强通用性并且集成了很多功能的项目模板，该模板可被应用在不同的项目需求中。也可被视为是一个项目的半成品。
- 如何学习框架？
  - 对于刚接触编程或者初级程序员来讲，对于一个新的框架，只需要掌握该框架的作用及其各个功能的使用和应用即可，对于框架的底层实现和原理，在逐步进阶的过程中在慢慢深入即可。
- 什么是scrapy？
  - Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架，非常出名，非常强悍。其内部已经被集成了各种功能（高性能异步下载，队列，分布式，解析，持久化等）。对于框架的学习，重点是要学习其框架的特性、各个功能的用法即可。

<!----more---->

#### scrapy基本使用

- 环境安装：
  - linux和mac操作系统：
    - pip install scrapy
  - windows系统：
    - pip install wheel
    - 下载twisted，下载地址为http://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted
    - 安装twisted：pip install Twisted‑17.1.0‑cp36‑cp36m‑win_amd64.whl
    - pip install pywin32
    - pip install scrapy
      测试：在终端里录入scrapy指令，没有报错即表示安装成功！
- scrapy使用流程：
  - 创建工程：
    - scrapy startproject ProName
  - 进入工程目录：
    - cd ProName
  - 创建爬虫文件：
    - scrapy genspider spiderName www.xxx.com
  - 编写相关操作代码
  - 执行工程：
    - scrapy crawl spiderName
- 爬虫文件剖析

```python
# -*- coding: utf-8 -*-
  import scrapy
  class QiubaiSpider(scrapy.Spider):
      name = 'qiubai' #应用名称
      #允许爬取的域名（如果遇到非该域名的url则爬取不到数据）
      allowed_domains = ['https://www.qiushibaike.com/']
      #起始爬取的url
      start_urls = ['https://www.qiushibaike.com/']
      #访问起始URL并获取结果后的回调函数，该函数的response参数就是向起始的url发送请求后，获取的响应对象.该函数返回值必须为可迭代对象或者NUll 
      def parse(self, response):
          print(response.text) #获取字符串类型的响应内容
          print(response.body)#获取字节类型的相应内容
```

- 配置文件settings.py修改：

```python
#修改内容及其结果如下：
USER_AGENT = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36' #伪装请求载体身份

ROBOTSTXT_OBEY = False  #可以忽略或者不遵守robots协议
```

- scrapy基于xpath数据解析操作：
  - 爬取糗事百科的段子数据

```python
# -*- coding: utf-8 -*-
import scrapy
class QiubaiSpider(scrapy.Spider):
  name = 'qiubai'
  allowed_domains = ['https://www.qiushibaike.com/']
  start_urls = ['https://www.qiushibaike.com/']
  def parse(self, response):
      #xpath为response中的方法，可以将xpath表达式直接作用于该函数中
      odiv = response.xpath('//div[@id="content-left"]/div')
      content_list = [] #用于存储解析到的数据
      for div in odiv:
          #xpath函数返回的为列表，列表中存放的数据为Selector类型的数据。我们解析到的内容被封装在了Selector对象中，需要调用extract()函数将解析的内容从Selecor中取出。
          author = div.xpath('.//div[@class="author clearfix"]/a/h2/text()')[0].extract()
          content=div.xpath('.//div[@class="content"]/span/text()')[0].extract()
          #打印展示爬取到的数据
          print(author,content)
```

