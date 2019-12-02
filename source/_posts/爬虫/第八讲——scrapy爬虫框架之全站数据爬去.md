---
title: 第八讲——scrapy爬虫框架之全站数据爬去
id: 8
date: 2019-9-22 20:30:00
tags: 爬虫
comment: true

---

#### 文件目录结构

```python
|-firstBlood # #我们创建的爬虫工程的名称 scrapy startproject firstBlood(工程名)
----|-firstBlood #和项目同名目录，自动创建的
--------|-__init__.py
--------|-spiders #工程创建好之后，cd到该目录下
------------|-first.py #创建爬虫文件 scapy genspider first(文件名) www.xxx.com(别问，就这样写)
--------|-items.py  #下面都是自动创建的，都是我们的功能文件
--------|-middlewares.py
--------|-pipelines.py
--------|-settings.py
----|-scrapy.cfg
```

<!----more---->

#### 全站数据的爬取

```python
实现全站数据爬取：
    --大部分的网站展示的数据都进行了分页操作，那么将所有页码对应的页面数据进行爬取就是爬虫中的全站数据爬取。
    --需求：将糗事百科所有页码的作者和段子内容数据进行爬取切持久化存储
    --实现方式：
        --使用列表将url手动添加进去（不推荐使用）
        --自行手动进行请求发送（推荐使用）
            --yield scrapy.Request(url,callback):callback专门用做于数据解析，回调的方式
```

爬虫文件:first.py

```python
import scrapy
from qiushibaike.items import QiushibaikeItem
# scrapy.http import Request
class QiushiSpider(scrapy.Spider):
  name = 'qiushi'
  allowed_domains = ['www.qiushibaike.com']
  start_urls = ['https://www.qiushibaike.com/text/']
  #爬取多页
  pageNum = 1 #起始页码
  url = 'https://www.qiushibaike.com/text/page/%s/' #每页的url
  def parse(self, response):
      div_list=response.xpath('//*[@id="content-left"]/div')
      for div in div_list:
          #//*[@id="qiushi_tag_120996995"]/div[1]/a[2]/h2
          author=div.xpath('.//div[@class="author clearfix"]//h2/text()').extract_first()
          author=author.strip('\n')
          content=div.xpath('.//div[@class="content"]/span/text()').extract_first()
          content=content.strip('\n')
          item=QiushibaikeItem()
          item['author']=author
          item['content']=content
          yield item #提交item到管道进行持久化
       #爬取所有页码数据
      if self.pageNum <= 13: #一共爬取13页（共13页）
          self.pageNum += 1
          url = format(self.url % self.pageNum)
          #递归爬取数据：callback参数的值为回调函数（将url请求后，得到的相应数据继续进行parse解析），递归调用parse函数
          yield scrapy.Request(url=url,callback=self.parse)
```

配置文件：settings.py

```python
USER_AGENT = '"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36"'

#打印错误日志，日志设置级别比较高
LOG_LEVEL = "ERROR"

# Obey robots.txt rules
ROBOTSTXT_OBEY = False
```

#### scrapy五大核心组件介绍

```
五大核心组件：引擎、Spider、调度器、下载器、管道
	---引擎(Scrapy)
		---用来处理整个系统的数据流处理, 触发事务(框架核心)
	---调度器(Scheduler)
		---用来接受引擎发过来的请求, 压入队列中, 并在引擎再次请求的时候返回. 可以想像成一个URL（抓取网页的网址或者说是链接）的优先队列, 由它来决定下一个要抓取的网址是什么, 同时去除重复的网址
	---下载器(Downloader)
		---用于下载网页内容, 并将网页内容返回给蜘蛛(Scrapy下载器是建立在twisted这个高效的异步模型上的)
	---爬虫(Spiders)
		---爬虫是主要干活的, 用于从特定的网页中提取自己需要的信息, 即所谓的实体(Item)。用户也可以从中提取出链接,让Scrapy继续抓取下一个页面
	---项目管道(Pipeline)
		---负责处理爬虫从网页中抽取的实体，主要的功能是持久化实体、验证实体的有效性、清除不需要的信息。当页面被爬虫解析后，将被发送到项目管道，并经过几个特定的次序处理数据。
```

![](http://9017499461.linshutu.top/scrapy%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6.png)

