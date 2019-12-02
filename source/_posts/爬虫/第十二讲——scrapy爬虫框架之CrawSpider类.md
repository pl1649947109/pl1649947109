---
title: 第十二讲——scrapy爬虫框架之CrawSpider类
id: 12
date: 2019-9-26 20:30:00
tags: 爬虫
comment: true
---

```python
CrawSpider类：Spider的一个子类
    --全栈数据爬取的方式
        -基于Spider：手动请求
        -基于CrawlSpider
    --CrawlSpider的使用：
        -创建一个工程
        -cd xx
        -创建爬虫文件（CrawlSpider）
            --scrapy genspider -t crawl name www.xxx.com
            --链接提取器：
                -作用：将链接提取器到的链接进行指定规则（callback）的解析。
    #需求：爬取sun网站中的编号，新闻标题，新闻内容，标号
        --分析：爬取的数据没有在同一张页面中。
            -1，可以使用链接提取器提取所有网页的链接
            -2，让链接提取器提取所有的新闻详情页的连接
```

<!----more---->

爬虫文件：firtst.py

```python
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from scrapy6.items import Scrapy6Item,DetailItem

#需求：爬取sun网站中的编号，新闻标题，新闻内容，标号
class PlSpider(CrawlSpider):
    name = 'pl'
    # allowed_domains = ['www.xxx.com']
    start_urls = ['http://wz.sun0769.com/index.php/question/questionType?type=4&page=']

    #连接提取器：根据指定规则（allow='正则'）进行指定链接的提取
    link = LinkExtractor(allow=r'type=4&page=\d+')
    link_detail = LinkExtractor(allow=r'question/\d+/\d+\.shtml')
    rules = (
        #规则解析器:将链接提取到的连接链接进行制定规则（callback）的解析操作
        Rule(link,callback='parse_item', follow=True),
        #follow=True:可以将链接提取器 继续作用到 连接提取提取器提取到的链接 所对应的页面中（就是可以将全栈所有的页码提取到）

        Rule(link_detail,callback='parse_detail'),
    )

    #解析新闻的编号和新闻的标题
    #如下两个解析方法是不可以实现请求传参的
    #如何将两个解析方法解析的数据储到同一个Item中，可以以此储存到两个item
    def parse_item(self, response):
        # item = {}
        # #item['domain_id'] = response.xpath('//input[@id="sid"]/@value').get()
        # #item['name'] = response.xpath('//div[@id="name"]').get()
        # #item['description'] = response.xpath('//div[@id="description"]').get()
        # return item

        # xpath表达式中不可以出现tbody标签,有就去掉用双杠代替
        tr_list = response.xpath('//*[@id="morelist"]/div/table[2]//tr/td/table/tr')
        for tr in tr_list:
            new_num = tr.xpath('./td[1]/text()').extract_first()
            new_title = tr.xpath('./td[2]/a[2]/@title').extract_first()
            item = Scrapy6Item()
            item['title'] = new_title
            item['new_num'] = new_num
            yield item

    #在详情页解析新闻内容和新闻编号
    def parse_detail(self,response):
        new_id = response.xpath('/html/body/div[9]/table[1]//tr/td[2]/span[2]/text()').extract_first()
        new_content = response.xpath('/html/body/div[9]/table[2]//tr[1]//text()').extract()
        new_content = ''.join(new_content)
        print (new_id,"\n",new_content)
        item = DetailItem()
        item['content'] = new_content
        item['new_id'] = new_id
        yield item
```

属性封装：items.py

```python
import scrapy


class Scrapy6Item(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    title = scrapy.Field()
    new_num = scrapy.Field()

class DetailItem(scrapy.Item):
    new_id = scrapy.Field()
    content = scrapy.Field()
```

管道储存：

```python
class Scrapy6Pipeline(object):
    def process_item(self, item, spider):
        #如何判断item的类型
        #将数据写入数据库时怎么保证数据的一致性（插入数据库以id为作为唯一id）
        if item.__class__.__name__ == 'DetailItem':
            print (item['new_id'],item['content'])
        else:
            print(item['new_num'], item['title'])
        return item
```

