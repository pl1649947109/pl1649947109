---
title: 第十一讲——scrapy爬虫框架之中间件和selenium中的应用
id: 11
date: 2019-9-25 20:30:00
tags: 爬虫
comment: true
---

### 中间件

- 下载中间件（Downloader Middlewares） 位于scrapy引擎和下载器之间的一层组件。
- 作用：我们主要使用下载中间件处理请求，一般会对请求设置随机的User-Agent ，设置随机的代理。目的在于防止爬取网站的反爬虫策略。
  - （1）引擎将请求传递给下载器过程中， 下载中间件可以对请求进行一系列处理。比如设置请求的 User-Agent，设置代理等
  - （2）在下载器完成将Response传递给引擎中，下载中间件可以对响应进行一系列处理。比如进行gzip解压等。

#### UA池：User-Agent池

- 作用：尽可能多的将scrapy工程中的请求伪装成不同类型的浏览器身份。

- 操作流程：

  - 1.在下载中间件中拦截请求
  - 2.将拦截到的请求的请求头信息中的UA进行篡改伪装
  - 3.在配置文件中开启下载中间件
<!----more---->
- UA池的封装：

  ```
    `    user_agent_list = [              "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 "              "(KHTML, like Gecko) Chrome/22.0.1207.1 Safari/537.1",              "Mozilla/5.0 (X11; CrOS i686 2268.111.0) AppleWebKit/536.11 "              "(KHTML, like Gecko) Chrome/20.0.1132.57 Safari/536.11",              "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.6 "              "(KHTML, like Gecko) Chrome/20.0.1092.0 Safari/536.6",              "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.6 "              "(KHTML, like Gecko) Chrome/20.0.1090.0 Safari/536.6",              "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.1 "              "(KHTML, like Gecko) Chrome/19.77.34.5 Safari/537.1",              "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/536.5 "              "(KHTML, like Gecko) Chrome/19.0.1084.9 Safari/536.5",              "Mozilla/5.0 (Windows NT 6.0) AppleWebKit/536.5 "              "(KHTML, like Gecko) Chrome/19.0.1084.36 Safari/536.5",              "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "              "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",              "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/536.3 "              "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",              "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_0) AppleWebKit/536.3 "              "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",              "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "              "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",              "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "              "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",              "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "              "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",              "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "              "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",              "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/536.3 "              "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",              "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "              "(KHTML, like Gecko) Chrome/19.0.1061.0 Safari/536.3",              "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/535.24 "              "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24",              "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/535.24 "              "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24"      ]`
  ```

#### 代理池

- 作用：尽可能多的将scrapy工程中的请求的IP设置成不同的。
- 操作流程：
  - 1.在下载中间件中拦截请求
  - 2.将拦截到的请求的IP修改成某一代理IP
  - 3.在配置文件中开启下载中间件

- 在通过scrapy框架进行某些网站数据爬取的时候，往往会碰到页面动态数据加载的情况发生，如果直接使用scrapy对其url发请求，是绝对获取不到那部分动态加载出来的数据值。但是通过观察我们会发现，通过浏览器进行url请求发送则会加载出对应的动态加载出的数据。那么如果我们想要在scrapy也获取动态加载出的数据，则必须使用selenium创建浏览器对象，然后通过该浏览器对象进行请求发送，获取动态加载的数据值。

#### selenium在scrapy中使用的原理分析

- 当引擎将国内板块url对应的请求提交给下载器后，下载器进行网页数据的下载，然后将下载到的页面数据，封装到response中，提交给引擎，引擎将response在转交给Spiders。Spiders接受到的response对象中存储的页面数据里是没有动态加载的新闻数据的。要想获取动态加载的新闻数据，则需要在下载中间件中对下载器提交给引擎的response响应对象进行拦截，切对其内部存储的页面数据进行篡改，修改成携带了动态加载出的新闻数据，然后将被篡改的response对象最终交给Spiders进行解析操作。

#### selenium在scrapy中的使用流程

- 重写爬虫文件的构造方法，在该方法中使用selenium实例化一个浏览器对象（因为浏览器对象只需要被实例化一次）
- 重写爬虫文件的closed(self,spider)方法，在其内部关闭浏览器对象。该方法是在爬虫结束时被调用
- 重写下载中间件的process_response方法，让该方法对响应对象进行拦截，并篡改response中存储的页面数据
- 在配置文件中开启下载中间件

#### 案例分析

- 需求：爬取网易新闻的国内板块下的新闻数据
- 需求分析：当点击国内超链进入国内对应的页面时，会发现当前页面展示的新闻数据是被动态加载出来的，如果直接通过程序对url进行请求，是获取不到动态加载出的新闻数据的。则就需要我们使用selenium实例化一个浏览器对象，在该对象中进行url的请求，获取动态加载的新闻数据。

#### 实例

```python
中间件（爬虫中间件、下载中间件）
    --下载中间件
        --位置：引擎和下载器之间
        --作用：批量拦截到整个工程中所有的请求和响应
        --拦截请求：
            --UA伪装:process_request
            --代理IP:process_exception:return  request
        --拦截响应：
            --篡改响应数据，响应对象
            --需求：爬取网易新闻中的新闻数据（标题和数据）
                -1，通过网页新闻的首页解析出五大板块对应的详情页的url（没有动态加载）
                -2，一个板块对应的新闻标题都是动态加载出来的
                -3，通过解析出来的每一条新闻详情页的url获取详情页的页面源码，解析出新闻内容

```

爬取文件：first.py

```python
# -*- coding: utf-8 -*-
import scrapy
from scrapy5.items import Scrapy5Item
from selenium import webdriver
import time

# class FirstSpider(scrapy.Spider):
#     name = 'first'
#     # allowed_domains = ['www.xxx.com']
#     start_urls = ['http://www.baidu.com/s?wd=ip']
#
#     def parse(self, response):
#         page_text = response.text
#         with open('ip.html','w',encoding='utf-8') as fp:
#             fp.write(page_text)

#网易新闻
class FirstSpider(scrapy.Spider):
    name = 'first'
    # allowed_domains = ['www.xxx.com']
    start_urls = ['https://news.163.com']
    models_urls = []  #存储五大板块对应详情页的url


    def __init__(self):
        self.bro = webdriver.Chrome(executable_path=r'C:\Users\pl\PycharmProjects\北京培训笔记\爬虫\selenium模块应用\chromedriver.exe')
        time.sleep(3)
    def parse(self, response):
        li_list = response.xpath('//*[@id="index2016_wrap"]/div[1]/div[2]/div[2]/div[2]/div[2]/div/ul/li')
        alist = [4,5,7,8,9]
        for i in alist:
            model_url = li_list[i].xpath('./a/@href').extract_first()
            self.models_urls.append(model_url)

        #依次对每一个板块对应的页面进行请求
        for url in self.models_urls:#对每一个板块的url进行请求发送
            yield scrapy.Request(url,callback=self.parse_model)

    #每一个板块对应的新闻的标题相关的内容都是动态加载出来的
    def parse_model(self,response):#解析每一板块页面中对应新闻的标题和新闻详情页的url
        div_list = response.xpath('/html/body/div/div[3]/div[4]/div[1]/div/div/ul/li/div')
        for div in div_list:
            title = div.xpath('./div/div[1]/h3/a/text()').extract_first()
            new_detail_url = div.xpath('./div/div[1]/h3/a/@href').extract_first()

            item = Scrapy5Item()
            item['title'] = title

            print (new_detail_url)
            #解析新闻的内容
        #这个请求的url出问题了，可能是解析出来的url有问题
            yield scrapy.Request(url = new_detail_url,callback=self.parse_detail,meta={'item':item})


    def parse_detail(self,response):
            content = response.xpath('//*[@id="endText"]//text()').extract()
            content = ''.join(content)

            item = response.meta['item']
            item['content'] = content

            yield item

    def closed(self,spider):
        self.bro.quit()
```

属性封装：items.py

```python
import scrapy

class Scrapy5Item(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    title = scrapy.Field()
    content = scrapy.Field()
```

中间件：middlewares.py

```python
from scrapy import signals
from scrapy.http import HtmlResponse
from selenium import webdriver
import random
import time

#下载中间件
class Scrapy5DownloaderMiddleware(object):
    # Not all methods need to be defined. If a method is not defined,
    # scrapy acts as if the downloader middleware does not modify the
    # passed objects.

    user_agent_list = ["Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 "
                       "(KHTML, like Gecko) Chrome/22.0.1207.1 Safari/537.1",
        "Mozilla/5.0 (X11; CrOS i686 2268.111.0) AppleWebKit/536.11 "
        "(KHTML, like Gecko) Chrome/20.0.1132.57 Safari/536.11",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.6 "
        "(KHTML, like Gecko) Chrome/20.0.1092.0 Safari/536.6", "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.6 "
                                                               "(KHTML, like Gecko) Chrome/20.0.1090.0 Safari/536.6",
        "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.1 "
        "(KHTML, like Gecko) Chrome/19.77.34.5 Safari/537.1", "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/536.5 "
                                                              "(KHTML, like Gecko) Chrome/19.0.1084.9 Safari/536.5",
        "Mozilla/5.0 (Windows NT 6.0) AppleWebKit/536.5 "
        "(KHTML, like Gecko) Chrome/19.0.1084.36 Safari/536.5", "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "
                                                                "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_0) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3", "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "
                                                               "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3", "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "
                                                               "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3", "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/536.3 "
                                                               "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.0 Safari/536.3", "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/535.24 "
                                                               "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24",
        "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/535.24 "
        "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24"]

    PROXY_http = ['153.180.102.104:80', '195.208.131.189:56055', ]
    PROXY_https = ['120.83.49.90:9000', '95.189.112.214:35508', ]

    #实例化一个浏览器对象

    @classmethod
    def from_crawler(cls, crawler):
        # This method is used by Scrapy to create your spiders.
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened)
        return s

    # 拦截请求
    #该方法拦截五大板块对应的响应对象，进行篡改
    def process_request(self, request, spider):
       #  # Called for each request that goes through the downloader
       #  # middleware.
       #  # UA伪装
       #  request.headers['User-Agent'] = random.choice(self.user_agent_list)
       #
       #  #为了验证代理的操作是否正确
       #  request.meta['proxy'] = 'http://116.62.198.43:8080'
       # # Must either:
       #  # - return None: continue processing this request
       #  # - or return a Response object
       #  # - or return a Request object
       #  # - or raise IgnoreRequest: process_exception() methods of
       #  #   installed downloader middleware will be called
        return None



    #拦截所有的响应
    def process_response(self, request, response, spider):
    #     # Called with the response returned from the downloader.
    #
    #     # Must either;
    #     # - return a Response object
    #     # - return a Request object
    #     # - or raise IgnoreRequest
    #     return response
    #
    # #拦截发生异常的请求
    # def process_exception(self, request, exception, spider):
    #     # Called when a download handler or a process_request()
    #     # (from other downloader middleware) raises an exception.
    #
    #     if request.url.split(":")[0] == 'http':
    #         #代理IP
    #         request.meta['proxy'] = 'http://'+random.choice(self.PROXY_http)
    #     else:
    #         request.meta['proxy'] = 'http://' + random.choice(self.PROXY_https)
    #
    #     return request  #将修正之后的请求对象进程重新的修正
    #
    #     # Must either:
    #     # - return None: continue processing this exception
    #     # - return a Response object: stops process_exception() chain
    #     # - return a Request object: stops process_exception() chain

        bro = spider.bro #获取了在爬虫类中定义的浏览器对象
        # 挑选出指定的响应对象进行篡改
        # 通过url指定request
        # 通过request指定response
        if request.url in spider.models_urls:
            bro.get(request.url) #五大板块对应详情页的url
            time.sleep(2)
            page_text = bro.page_source #包含了动态加载的新闻数据
            # response  #五大板块对应详情页的url
            # 针对定位到的这些response进行篡改
            # 实例化一个新的响应对象（符合需求：包涵动态加载出的新闻数据），代替旧的对象
            # 基于selenium便捷的获取动态加载数据
            new_response = HtmlResponse(url=request.url, body=page_text, encoding='utf-8', request=request)
            return new_response
        else:
            # response ##其他板块对应详情页的url
            return response

    def spider_opened(self, spider):
        spider.logger.info('Spider opened: %s' % spider.name)
```

配置文件：settings.py

```python
ROBOTSTXT_OBEY = False

#设置日志级别
LOG_LEVEL = "ERROR"

#开启中间件
SPIDER_MIDDLEWARES = {
   'scrapy5.middlewares.Scrapy5DownloaderMiddleware': 543,
}

#开启管道
ITEM_PIPELINES = {
   'scrapy5.pipelines.Scrapy5Pipeline': 300,
}
```

