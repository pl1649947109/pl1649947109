---
title: 第七讲——scrapy爬虫框架之数据持久化储存
id: 7
date: 2019-9-2120:30:00
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

#### 数据的持久化储存

需求：爬取糗事百科，将数据爬取并储存

##### 方式一：基于终端命令进对数据进行储存

```python
-基于终端指令
	--要求：只可以将parse方法的返回值储存到本地的文本文件中
scrapy crawl 脚本文件名称 -o 文件路径
	--注意：持久化储存对应的文本文件的类型只可以是('json', 'jsonlines', 'jl', 'csv', 'xml', 'marshal', 'pickle')数据格式
	--好处：简介高效便捷
	--缺点：局限性比较强（数据只可以储存到指定后缀的文本文件中）
```

爬取文件：first.py

```python
import scrapy
class FirstSpider(scrapy.Spider):
    #爬文件的名称：就是爬虫源文件的唯一标识
    name = 'first'
    #允许的域名：用来限制start urls中的哪些uel可以进行请求发送（一般不用，直接干掉）
    #allowed_domains = ['www.xxx.com']
    #起始url的列表：该列表中存放的url会被scrapy自动进行请求的发送
    start_urls = ["https://www.qiushibaike.com/text/"]

    #用作数据解析：response参数表示就是请求成功后对应的响应请求对象
    def parse(self, response):
       #糗图解析：作者名称+段子内容
        div_list = response.xpath('//*[@id="content-left"]/div')
        all_data = []
        for i in div_list:
            #我们发现xpath返回的是列表，但是列表元素一定是Selector类型的对象
            #extract可以将Selector对象中的data参数储存的字符串提取出来，需要调用extract()函数将解析的内容从Selecor中取出。
            auth_name = i.xpath('./div[1]/a[2]/h2/text()')[0].extract()
            #和上面的代码是等价的（针对列表中只有一个元素）
            # auth_name = i.xpath('./div[1]/a[2]/h2/text()').extract_first()
            #列表调用了ex之后，则表示将列表中每一个sel对象中data对应的字符串取出来
            content = i.xpath('./a[1]/div/span//text()').extract()
            content = ''.join(content)
            # 持久化存储
            dic = {
                'author':auth_name,
                'content':content,
            }
            all_data.append(dic)
        return all_data
```

上面我们拿到了数据，下面我们在终端执行指令

```python
执行输出指定格式进行存储：将爬取到的数据写入不同格式的文件中进行存储
scrapy crawl 爬虫名称 -o xxx.json
scrapy crawl 爬虫名称 -o xxx.xml
scrapy crawl 爬虫名称 -o xxx.csv
```

##### 方式二：基于管道对数据进行储存

```python
-基于管道（一般就使用它持久化储存）：
	--编码流程：
		--数据解析
		--在item类中定义相关的属性（items.py文件）
		--将解析的数据封装储存到item类型的对象
		--将item类型的对象提交给管道进行持久化存储的操作(pipeline.py文件)
		--在管道类的process_item中将其接受到的item对象中储存的数据进行持久化存储操作
		--在配置文件中开启管道
	--好处：通用性强
```

了解管道

- scrapy框架中已经为我们专门集成好了高效、便捷的持久化操作功能，我们直接使用即可。要想使用scrapy的持久化操作功能，我们首先来认识如下两个文件：
  - items.py：数据结构模板文件。定义数据属性。
  - pipelines.py：管道文件。接收数据（items），进行持久化操作。

爬取文件：first.py

```python
import scrapy
from  firstBlood.items import QiubaiProItem

class FirstSpider(scrapy.Spider):
	#基于管道
    name = 'first'
    start_urls = ["https://www.qiushibaike.com/text/"]

    #用作数据解析：response参数表示就是请求成功后对应的响应请求对象
    def parse(self, response):
       #糗图解析：作者名称+段子内容
        div_list = response.xpath('//*[@id="content-left"]/div')
        all_data = []
        for i in div_list:
            auth_name = i.xpath('./div[1]/a[2]/h2/text()')[0].extract()
            content = i.xpath('./a[1]/div/span//text()').extract()
            content = ''.join(content)
            
            #将解析到的数据封装到items对象中
            item = QiubaiProItem()  #实例化item对象
            item["author"] = auth_name
            item["content"] = content
            
            yield item #将item提交到管道
```

属性封装文件:items.py

```python
import scrapy
class SecondbloodItem(scrapy.Item):
  # define the fields for your item here like:
  # name = scrapy.Field()
  #封装属性
  author = scrapy.Field() #存储作者
  content = scrapy.Field() #存储段子内容
```

管道文件：pipelines.py

```python
import pymysql

class FirstbloodPipeline(object):
    def __init__(self):
        self.fp = None #定义一个文件描述符属性
        
#下面重写父类的方法：
	#该方法只在开始爬虫的时候只被调用一次
    def open_spider(self,spider):
        print("bein......")
        self.fp = open('./qiubai.txt','w',encoding='utf-8')

    #专门用来处理item类型对象
    #该方法可以接受爬虫文件提交过来的item对象
    #该方每接受到一个item就被调用一次
    def process_item(self, item, spider):
        author = item["author"]
        content = item["content"]
		#将爬虫程序提交的item进行持久化储存
        self.fp.write(author+":"+content+"\n")
        #就是传递给下一个即将执行的管道类
        return item 

    #结束爬虫时，执行一次
    def close_spider(self,spider):
        print ("结束爬虫.")
        self.fp.close()
#我们的问题来了，我们需要将爬取的数据在磁盘上面储存一份，在数据库中储存一份，我们该怎么操作？
#下面这个类就是解决上面的问题，连接mysql，并储存数据
class MysqlPipeLine(object):
    conn = None
    cursor = None
    def open_spider(self, spider):
        print ("begin。。。。")
        self.conn = pymysql.connect(host='127.0.0.1', user='root', password='123456', database='mysql')
    def process_item(self, item, spider):
        self.cursor = self.conn.cursor()

        try:
            self.cursor.execute("insert into pl values('%s','%s')"%(item['author'],item['content']))
            self.conn.commit()
        except Exception as e:
            print (e)
            self.conn.rollback()

        return item
    def close_spider(self,spider):
        print ("结束爬虫.")
        self.cursor.close()
        self.conn.close()
--解析：
	--管道文件中一个管道类对应的是将数据储存到一个平台
	--爬虫文件提交item只会给管道文件中第一个被执行的管道类接收
	--process_item中的return item便是键item传递给下一个即将被执行的管道类，这里就是先储存到磁盘再储存执行数据库的类，储存到数据库
```

配置文件：settings.py

```python
#下列结构为字典，字典中的键值表示的是即将被启用执行的管道文件和其执行的优先级。
ITEM_PIPELINES = {
 'doublekill.pipelines.FirstbloodPipeline': 300, #本地磁盘储存
 'doublekill.pipelines.MysqlPipeLine': 200,  #数据库储存
}
#上述代码中，字典中的两组键值分别表示会执行管道文件中对应的两个管道类中的process_item方法，实现两种不同形式的持久化操作
```

