---
title: 第九讲——scrapy爬虫框架之请求传参
id: 9
date: 2019-9-23 20:30:00
tags: 爬虫
comment: true
---

### 前言

- 在某些情况下，我们爬取的数据不在同一个页面中，例如，我们爬取一个电影网站，电影的名称，评分在一级页面，而要爬取的其他电影详情在其二级子页面中。这时我们就需要用到请求传参。
- 请求传参的使用场景
  - 当我们使用爬虫爬取的数据没有存在于同一张页面的时候，则必须使用请求传参
- 需求：爬取boos的岗位名称，岗位描述

<!----more---->

爬取文件：boss.py

```python
import scrapy
from bossPro.items import BossproItem
class BossSpider(scrapy.Spider):
    name = 'boss'
    # allowed_domains = ['www.xxx.com']
    start_urls = ['https://www.zhipin.com/job_detail/?query=python&city=101010100&industry=&position=']
    url = 'https://www.zhipin.com/c101010100/?query=python&page=%d'
    page_num = 2
   #回调函数接受item
    def parse_detail(self,response):
        item = response.meta['item']
        job_desc = response.xpath('//*[@id="main"]/div[3]/div/div[2]/div[2]/div[1]/div//text()').extract()
        job_desc = ''.join(job_desc)
        # print(job_desc)
        item['job_desc'] = job_desc
        yield item
    #解析首页中的岗位名称
    def parse(self, response):
        li_list = response.xpath('//*[@id="main"]/div/div[3]/ul/li')
        for li in li_list:
            item = BossproItem()
            job_name = li.xpath('.//div[@class="info-primary"]/h3/a/div[1]/text()').extract_first()
            item['job_name'] = job_name
            # print(job_name)
            detail_url = 'https://www.zhipin.com'+li.xpath('.//div[@class="info-primary"]/h3/a/@href').extract_first()
            #对详情页发请求获取详情页的页面源码数据
            #手动请求的发送
            #请求传参：meta={}，可以将meta字典传递给请求对应的回调函数
            yield scrapy.Request(detail_url,callback=self.parse_detail,meta={'item':item})
        #分页操作
        if self.page_num <= 3:
            new_url = format(self.url%self.page_num)
            self.page_num += 1
            yield scrapy.Request(new_url,callback=self.parse)
```

**为了提升scrapy的爬取效率，我们可以再配置文件里面做一些操作**：

配置文件：settings.py

- 增加并发：
  - 默认scrapy开启的并发线程为32个，可以适当进行增加。在settings配置文件中修改CONCURRENT_REQUESTS = 100值为100,并发设置成了为100。
- 降低日志级别：
  - 在运行scrapy时，会有大量日志信息的输出，为了减少CPU的使用率。可以设置log输出信息为INFO或者ERROR即可。在配置文件中编写：LOG_LEVEL = ‘INFO’
- 禁止cookie：
  - 如果不是真的需要cookie，则在scrapy爬取数据时可以禁止cookie从而减少CPU的使用率，提升爬取效率。在配置文件中编写：COOKIES_ENABLED = False
- 禁止重试：
  - 对失败的HTTP进行重新请求（重试）会减慢爬取速度，因此可以禁止重试。在配置文件中编写：RETRY_ENABLED = False
- 减少下载超时：
  - 如果对一个非常慢的链接进行爬取，减少下载超时可以能让卡住的链接快速被放弃，从而提升效率。在配置文件中进行编写：DOWNLOAD_TIMEOUT = 10 超时时间为10s