---
title: 第十三讲——scrapy爬虫框架之分布式爬虫
id: 12
date: 2019-9-27 20:30:00
tags: 爬虫
comment: true
---

```python
#分布式爬虫
    --概念：我们需要搭建一个分布式的机群，让其对一组资源分布联合爬取
    --作用：提升爬去数据的效率
    -如何实现分布式？
        --安装一个scrapy-redis的组件，pip下载
        --原生的scrapy是不可以实现分布式爬虫的，必须要让scrapy结合组件一起实现分布式爬虫
        -为什么原生的scrapy不可以实现分布式？
            --调度器不可以被分布式机群共享
            --管道不可以被分布式机群共享
        -scrapy-redis组件的作用：
            --可以给原生scrapy框架提供被共享的管道和调度器
        重要：
        实现流程：
            -创建工程
            -创建一个基于CrawlSpider的爬虫文件
            -修改当前的爬虫文件：
                --导包：from scrapy_redis.spiders import ResidCrawlSpider
                --将start_urls和allow_domains进行注释
                --添加一个新的属性：redis_key = 'sun',表示可以被共享的调度器队列的名称
                --编写数据解析相关的操作
                --将当前爬虫类的父类修改程RedisCrawlSpiders
            -修改配置文件settings
                --指定使用可以被共享的管道：
                    ITEM_PIPELINES ={                    'scrapy_redis.pipelines.RedisPipeline': 400,
                          }
                --指定调度器：
                --指定redis服务器
            -redis相关的操作配置
                --打开配置文件redis.conf
                    --将bind 127.0.0.1进行删除
                    --关闭保护模式：protected-mode yes改为no
                --结合着配置文件卡其redis服务
                    --redis.server.exe
                --启动客户端
                    --redis.cli.exe
            --执行工程：
                --scrapy runspider xxx.py
            --向调度器的队列中放入一个起始的url
                -调度器的队列在redis的客户端中
                    --执行lpush xxx(这里的key是sun) 起始url
            --最终爬取到的数据存储在了redis的proName：items这个数据结构中
#增量式爬虫（比较小众）
    --概念：检测网站数据的更新的情况，只会爬取网站最新更新出来的数据
```

<!----more---->

爬取文件：first.py

```python
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from scrapy_redis.spiders import RedisCrawlSpider

from scrapy7.items import Scrapy7Item

class FbsSpider(RedisCrawlSpider):
    name = 'fbs'

    # allowed_domains = ['www.xxx.com']
    # start_urls = ['http://www.xxx.com/']

    redis_key = 'sun'
    rules = (
        Rule(LinkExtractor(allow=r'type=4&page=\d+'), callback='parse_item', follow=True),
    )

    def parse_item(self, response):
        #item['domain_id'] = response.xpath('//input[@id="sid"]/@value').get()
        #item['name'] = response.xpath('//div[@id="name"]').get()
        #item['description'] = response.xpath('//div[@id="description"]').get()
        tr_list = response.xpath('//*[@id="morelist"]/div/table[2]//tr/td/table/tr')
        for tr in tr_list:
            new_num = tr.xpath('./td[1]/text()').extract_first()
            new_title = tr.xpath('./td[2]/a[2]/@title').extract_first()
            item = Scrapy7Item()
            item['title'] = new_title
            item['new_num'] = new_num
            yield item
```

属性封装：items.py

```python
import scrapy

class Scrapy7Item(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    title = scrapy.Field()
    new_num = scrapy.Field()
```

管道储存：pipelnes.py

```python
class Scrapy7Pipeline(object):
    def process_item(self, item, spider):
        return item
```

配置文件：settings.py

```python
USER_AGENT = '"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36"',

# LOG_LEVEL = "ERROR"
# Obey robots.txt rules
ROBOTSTXT_OBEY = False

#开启管道
ITEM_PIPELINES = {
   'scrapy_redis.pipelines.RedisPipeline': 400,
}
#指定调度器
#
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
#
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
#
SCHEDULER_PERSIST = True


REDIS_HOST =  '127.0.0.1' #'指定redis服务器的ip地址',不在本机就写远程主机ip
REDIS_PORT = 6379
```

