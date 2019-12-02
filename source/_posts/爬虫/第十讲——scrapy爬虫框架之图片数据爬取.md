---
title: 第十讲——scrapy爬虫框架之图片数据爬取
id: 10
date: 2019-9-24 20:30:00
tags: 爬虫
comment: true
---

```python
爬取图片ImagesPipeline
    --基于scrapy爬取字符串数据类型的数据和爬取图片类型的数据的区别
        --字符串：只需要基于xpath进行数据解析且提交管道进行持久化储存
        --图片：xpath解析出图片src的属性值，单独的对图片地址发起请求获取图片二进制类型的数据
    --ImagesPipline：
        --只需要将img的src的属性值进行解析，提交到管道，管道会对图片进行发送请求获取图片的二进制数据
        --使用流程：
           -需求：爬取站长素材中的高清图片
           -数据解析：获取图片的地址
           -将储存图片地址的item提交到指定的管道类
           -在管道文件中自定制一个基于ImagesPileLine的一个管道类
           -重写3个管道类的方法
           -在配置文件中：
                -指定图片储存的目录：IMAGES_STORE
                -指定开启的管道：就是自定制的管道
```

爬取文件：first.py

```python
import scrapy
from scrapy4.items import Scrapy4Item


class FirstSpider(scrapy.Spider):
    name = 'first'
    # allowed_domains = ['www.xxx.com']
    start_urls = ['http://sc.chinaz.com/tupian/']

    def parse(self, response):
        div_list = response.xpath('//*[@id="container"]/div')
        for div in div_list:
            #注意：使用伪属性（图片的懒加载）
            src = div.xpath('./div/a/img/@src2').extract_first()

            item = Scrapy4Item()
            item['src'] = src
            yield item

```

属性封装：items.py

```python
import scrapy

class Scrapy4Item(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    src = scrapy.Field()
```

管道储存：pipelines.py

```python
#自定义
from scrapy.pipelines.images import ImagesPipeline
import scrapy

class ImgproPipeline(object):
    item = None

    def process_item(self, item, spider):
        # print(item)
        return item

#ImagesPipeline专门用于文件下载的管道类，下载过程支持异步和多线程
class ImagesPipeline(ImagesPipeline):
    #就是可以根据图片的地址进行图片数据的请求
    def get_media_requests(self, item, info):
        yield scrapy.Request(item['src'])

    #定制图片的名称
    def file_downloaded(self, response, request, info=None):
        imageName = request.url.split('/')[-1]
        return imageName

    def item_completed(self, results, item, info):
        return item #返回给下一个即将被执行的管道类
```

配置文件：settings.py

```python
#UA伪装
USER_AGENT = '"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36"'

#不遵循该协议
ROBOTSTXT_OBEY = False

#开启管道
ITEM_PIPELINES = {
   'scrapy4.pipelines.ImagesPipeline': 300,
}

#指定图片指定的路径
IMAGES_STORE = ‘./imgs’：表示最终图片存储的目录
```

