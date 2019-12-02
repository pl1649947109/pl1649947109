---
title: 第一讲——爬虫初始、requests模块、数据解析
id: 1
date: 2019-9-15 20:30:00
tags: 爬虫
comment: true
---

## 爬虫初始

**什么是爬虫**

爬虫就是通过编写程序**模拟**浏览器上网，让其去互联网上**抓取**数据的过程。

### 爬虫的安全性问题

链接：https://mp.weixin.qq.com/s/-iv6QjCXLAdrwymNHek_3g

**风险来源**

- 爬虫干扰了被访问网站的正常运营；
- 爬虫抓取了受到法律保护的特定类型的数据或信息。

### 爬虫的分类

**通用爬虫：**通用爬虫是搜索引擎（Baidu、Google、Yahoo等）“抓取系统”的重要组成部分。主要目的是将互联网上的网页下载到本地，形成一个互联网内容的镜像备份。 简单来讲就是尽可能的；把互联网上的所有的网页下载下来，放到本地服务器里形成备分，在对这些网页做相关处理(提取关键字、去掉广告)，最后提供一个用户检索接口。

**聚焦爬虫：**聚焦爬虫是根据指定的需求抓取网络上指定的数据。例如：获取豆瓣上电影的名称和影评，而不是获取整张页面中所有的数据值。

**增量式爬虫：**增量式是用来检测网站数据更新的情况，且可以将网站更新的数据进行爬取（后期会有章节单独对其展开详细的讲解）。

<!----more---->

### robots协议

几乎是和爬虫技术诞生的同一时刻，反爬虫技术也诞生了。在90年代开始有搜索引擎网站利用爬虫技术抓取网站时，一些搜索引擎从业者和网站站长通过邮件讨论定下了一项“君子协议”—— robots.txt。即网站有权规定网站中哪些内容可以被爬虫抓取，哪些内容不可以被爬虫抓取。这样既可以保护隐私和敏感信息，又可以被搜索引擎收录、增加流量。

历史上第一桩关于爬虫的官司诞生在2000年，eBay将一家聚合价格信息的比价网站BE告上了法庭，eBay声称自己已经将哪些信息不能抓取写进了robots协议中，但BE违反了这一协议。但BE认为eBay上的内容属于用户集体贡献而不归用户所有，爬虫协议不能用作法律参考。最后经过业内反复讨论和法庭上的几轮唇枪舌战，最终以eBay胜诉告终，也开了用爬虫robots协议作为主要参考的先河。

最后，可以通过网站域名 + /robots.txt的形式访问该网站的协议详情，例如：www.taobao.com/robots.txt

## Requests模块

### 反爬机制：

- User-Agent：请求载体的身份标识，使用浏览器发起的请求，请求载体的身份标识为浏览器，使用爬虫程序发起的请求，请求载体为爬虫程序。
- UA检测：相关的门户网站通过检测请求该网站的载体身份来辨别该请求是否为爬虫程序，如果是，则网站数据请求失败。因为正常用户对网站发起的请求的载体一定是基于某一款浏览器，如果网站检测到某一请求载体身份标识不是基于浏览器的，则让其请求失败。因此，UA检测是我们整个课程中遇到的第二种反爬机制，第一种是robots协议。
- UA伪装：通过修改/伪装爬虫请求的User-Agent来破解UA检测这种反爬机制。
- 获取浏览器的User_Agent:输入：http://www.useragentstring.com
- 使用什么浏览器的UA就需要使用什么浏览器去解析抓取来的网页，不然有的内容解析不出来（由于浏览器的内核不同造成的）。

### 基于requests模块的get请求

模块祥讲：https://chpl.top/2019/08/22/%E7%AC%AC%E4%B8%89%E6%96%B9%E6%A8%A1%E5%9D%97/requests%E2%80%94%E2%80%94%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E6%A8%A1%E5%9D%97/

```python
需求：爬取搜狗指定词条对应的搜索结果页面
import requests
#指定搜索关键字
word = input('enter a word you want to search:')
#自定义请求头信息:UA伪装,将包含了User-Agent的字典作用到请求方法的headers参数中即可
headers={
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
    }
#指定url，原始url可能是https://www.sogou.com/web?query=撩妹，发现该url携带了参数
url = 'https://www.sogou.com/web'
#封装get请求参数：如果请求携带了参数，则可以将参数封装到字典中结合这requests请求方法中的data/params参数进行url参数的处理
param = {
    'query':word,
}
#发起请求
response = requests.get(url=url,params=param,headers=headers)
#获取响应数据
page_text = response.text
#持久化存储
fileName = word+'.html'
with open(fileName,'w',encoding='utf-8') as fp:
    fp.write(page_text)
```

### 基于requests模块的post请求

```python
需求：破解百度翻译
import requests
import json
word = input('enter a English word:')
#自定义请求头信息:UA伪装,将包含了User-Agent的字典作用到请求方法的headers参数中即可
headers={
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
    }
#指定url，原始url可能是https://www.sogou.com/web?query=撩妹，发现该url携带了参数
url = 'https://fanyi.baidu.com/sug'
#封装post请求参数：如果请求携带了参数，则可以将参数封装到字典中结合这requests请求方法中的data/params参数进行url参数的处理
data = {
    'kw':word,
}
#发起请求
response = requests.post(url=url,data=data,headers=headers)
#获取响应数据:如果响应回来的数据为json，则可以直接调用响应对象的json方法获取json对象数据
json_data = response.json()
#持久化存储
fileName = word+'.json'
fp = open(fileName,'w',encoding='utf-8')
json.dump(json_data,fp,ensure_ascii=False)
```

### 基于requests模块ajax的get请求

```python
需求：爬取豆瓣电影分类排行榜 https://movie.douban.com/中的电影详情数据
import requests
if __name__ == "__main__":
    #指定ajax-get请求的url（通过抓包进行获取）
    url = 'https://movie.douban.com/j/chart/top_list?'
    #定制请求头信息，相关的头信息必须封装在字典结构中
    headers = {
        #定制请求头中的User-Agent参数，当然也可以定制请求头中其他的参数
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36',
    }
    #定制get请求携带的参数(从抓包工具中获取)
    param = {
        'type':'5',
        'interval_id':'100:90',
        'action':'',
        'start':'0',
        'limit':'20'
    }
    #发起get请求，获取响应对象
    response = requests.get(url=url,headers=headers,params=param)
    #获取响应内容
    print(response.json())
```

### 基于requests模块ajax的post请求

```python
需求：爬取肯德基餐厅查询http://www.kfc.com.cn/kfccda/index.aspx中指定地点的餐厅数据
import requests
if __name__ == "__main__":
    #指定ajax-post请求的url（通过抓包进行获取）
    url = 'http://www.kfc.com.cn/kfccda/ashx/GetStoreList.ashx?op=keyword'
    #定制请求头信息，相关的头信息必须封装在字典结构中
    headers = {
        #定制请求头中的User-Agent参数，当然也可以定制请求头中其他的参数
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36',
    }
    #定制post请求携带的参数(从抓包工具中获取)
    data = {
        'cname':'',
        'pid':'',
        'keyword':'北京',
        'pageIndex': '1',
        'pageSize': '10'
    }
    #发起post请求，获取响应对象
    response = requests.get(url=url,headers=headers,data=data)
    #获取响应内容
    print(response.json())
```

### 综合

```python
需求：爬取国家药品监督管理总局中基于中华人民共和国化妆品生产许可证相关数据http://125.35.6.84:81/xk/
#分析
'''
页面的数据是动态加载的
首页中对应的企业信息数据是通过ajax动态请求得到的  
分析：
通过对比详情页url可以看见：
	url的域名都是一样的
	id值可以从首页对应的ajax请求到json串中获取
	域名和id值拼接处一个完整的企业对应的详情页的url
详情页的企业详情数据也是动态加载出来的
http://125.35.6.84:81/xk/itownet/portal/dzpz.jsp?id=018a3b77cb50485cac23004b76b31903
http://125.35.6.84:81/xk/itownet/portal/dzpz.jsp?id=f0a11683d8b64fb69a29576c495da2d4  
观察后发现：
	所有的post请求的url都是一样的，只有参数id值是不同的
	如果我们可以批量获取多级企业的id后，就可以将id和url形成一个完整的详细页面对应的ajax的请求
'''
import requests
from fake_useragent import UserAgent
#UA池，防止UA过期，或者访问次数过多被限制
ua = UserAgent(use_cache_server=False,verify_ssl=False).random
headers = {
    'User-Agent':ua
}
url = 'http://125.35.6.84:81/xk/itownet/portalAction.do?method=getXkzsList'
pageNum = 3  #分页
for page in range(3,5): 
    data = {
        'on': 'true',
        'page': str(page),
        'pageSize': '15',
        'productName':'',
        'conditionType': '1',
        'applyname':'',
        'applysn':''
    }
    json_text = requests.post(url=url,data=data,headers=headers).json()
    all_id_list = []
    for dict in json_text['list']:
        id = dict['ID']#用于二级页面数据获取
        #下列详情信息可以在二级页面中获取
        # name = dict['EPS_NAME']
        # product = dict['PRODUCT_SN']
        # man_name = dict['QF_MANAGER_NAME']
        # d1 = dict['XC_DATE']
        # d2 = dict['XK_DATE']
        all_id_list.append(id)
    #该url是一个ajax的post请求
    post_url = 'http://125.35.6.84:81/xk/itownet/portalAction.do?method=getXkzsById'
    for id in  all_id_list:
        post_data = {
            'id':id
        }
        response = requests.post(url=post_url,data=post_data,headers=headers)
        if response.headers['Content-Type'] == 'application/json;charset=UTF-8':
            #print(response.json())
            #进行json解析
            json_text = response.json()
            print(json_text['businessPerson'])
```

## 数据解析

### 聚焦爬虫

想获得某一个页面的一部分内容，可以先把整张页面爬下来，再去解析

```python
#流程：
#指定url
#发起请求
#获取响应数据
#数据解析
#持久化储存
```
### 数据解析原理

- 解析的局部的文本内容都会在标签之间或者便签对应的属性中进行储存

- 进行指定标签的定位

- 标签或者标签对应属性中存储的数据值进行提取（解析）

### 正则表达式解析

链接：https://chpl.top/2019/08/05/%E9%83%A8%E5%88%86%E8%AF%A6%E8%AE%B2/python%E4%B9%8B%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/

```python
项目需求：爬取糗事百科指定页面的糗图，并将其保存到指定文件夹中
import requests
import re
import os
if __name__ == "__main__":
     url = 'https://www.qiushibaike.com/pic/%s/'
     headers={
         'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
     }
     #指定起始也结束页码
     page_start = int(input('enter start page:'))
     page_end = int(input('enter end page:'))
     #创建文件夹
     if not os.path.exists('images'):
         os.mkdir('images')
     #循环解析且下载指定页码中的图片数据
     for page in range(page_start,page_end+1):
         print('正在下载第%d页图片'%page)
         new_url = format(url % page)
         response = requests.get(url=new_url,headers=headers)
         #解析response中的图片链接
         e = '<div class="thumb">.*?<img src="(.*?)" alt.*?</div>'
         pa = re.compile(e,re.S)
         image_urls = pa.findall(response.text)
          #循环下载该页码下所有的图片数据
         for image_url in image_urls:
             image_url = 'https:' + image_url
             image_name = image_url.split('/')[-1]
             image_path = 'images/'+image_name
             image_data = requests.get(url=image_url,headers=headers).content
             with open(image_path,'wb') as fp:
                 fp.write(image_data)
 
```

### bs4解析

**数据解析的原理**
1.标签定位
2.提取标签、标签属性中储存的数据值

**bs4数据解析的原理**
1.实例化一个beautifulSoup对象，并且将页面源码数据加载到该对象中
2.通过调用bs4中的属性或者方法进行标签定位和数据提取

**怎么操作bs4**

```python
使用流程：       
    - 导包：from bs4 import BeautifulSoup
    - 使用方式：可以将一个html文档，转化为BeautifulSoup对象，然后通过对象的方法或者属性去查找指定的节点内容
        （1）转化本地文件：
             - soup = BeautifulSoup(open('本地文件'), 'lxml')
        （2）转化网络文件：
             - soup = BeautifulSoup('字符串类型或者字节类型', 'lxml')
        （3）打印soup对象显示内容为html文件中的内容
基础巩固：
    （1）根据标签名查找
        - soup.a   只能找到第一个符合要求的标签
    （2）获取属性
        - soup.a.attrs  获取a所有的属性和属性值，返回一个字典
        - soup.a.attrs['href']   获取href属性
        - soup.a['href']   也可简写为这种形式
    （3）获取内容
        - soup.a.string
        - soup.a.text
        - soup.a.get_text()
       【注意】如果标签还有标签，那么string获取到的结果为None，而其它两个，可以获取文本内容
    （4）find：找到第一个符合要求的标签
        - soup.find('a')  找到第一个符合要求的
        - soup.find('a', title="xxx")
        - soup.find('a', alt="xxx")
        - soup.find('a', class_="xxx")
        - soup.find('a', id="xxx")
    （5）find_all：找到所有符合要求的标签
        - soup.find_all('a')
        - soup.find_all(['a','b']) 找到所有的a和b标签
        - soup.find_all('a', limit=2)  限制前两个
    （6）根据选择器选择指定的内容
               select:soup.select('#feng')
        - 常见的选择器：标签选择器(a)、类选择器(.)、id选择器(#)、层级选择器
            - 层级选择器：
                div .dudu #lala .meme .xixi  下面好多级
                div > p > a > .lala          只能是下面一级
        【注意】select选择器返回永远是列表，需要通过下标提取指定的对象
```

**实例**

```python
需求：使用bs4实现将诗词名句网站中三国演义小说的每一章的内容爬去到本地磁盘进行存储http://www.shicimingju.com/book/sanguoyanyi.html
import requests
from bs4 import BeautifulSoup
headers={
         'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
     }
def parse_content(url):
    #获取标题正文页数据
    page_text = requests.get(url,headers=headers).text
    soup = BeautifulSoup(page_text,'lxml')
    #解析获得标签
    ele = soup.find('div',class_='chapter_content')
    content = ele.text #获取标签中的数据值
    return content
if __name__ == "__main__":
     url = 'http://www.shicimingju.com/book/sanguoyanyi.html'
     reponse = requests.get(url=url,headers=headers)
     page_text = reponse.text
     #创建soup对象
     soup = BeautifulSoup(page_text,'lxml')
     #解析数据
     a_eles = soup.select('.book-mulu > ul > li > a')
     print(a_eles)
     cap = 1
     for ele in a_eles:
         print('开始下载第%d章节'%cap)
         cap+=1
         title = ele.string
         content_url = 'http://www.shicimingju.com'+ele['href']
         content = parse_content(content_url)
         with open('./sanguo.txt','w') as fp:
             fp.write(title+":"+content+'\n\n\n\n\n')
             print('结束下载第%d章节'%cap)
```

### xpath解析

xpath解析是我们在爬虫中最常用也是最通用的一种数据解析方式，由于其高效且简介的解析方式受到了广大程序员的喜爱。在后期学习scrapy框架期间，也会再次使用到xpath解析。

**解析原理**

- 使用通用爬虫爬取网页数据
- 实例化etree对象，且将页面数据加载到该对象中
- 使用xpath函数结合xpath表达式进行标签定位和指定数据提取

**常用的xpath表达式**

```python
属性定位：
    #找到class属性值为song的div标签
    //div[@class="song"] 
层级&索引定位：
    #找到class属性值为tang的div的直系子标签ul下的第二个子标签li下的直系子标签a
    //div[@class="tang"]/ul/li[2]/a
逻辑运算：
    #找到href属性值为空且class属性值为du的a标签
    //a[@href="" and @class="du"]
模糊匹配：
    //div[contains(@class, "ng")]
    //div[starts-with(@class, "ta")]
取文本：
    # /表示获取某个标签下的文本内容
    # //表示获取某个标签下的文本内容和所有子标签下的文本内容
    //div[@class="song"]/p[1]/text()
    //div[@class="tang"]//text()
取属性：
    //div[@class="tang"]//li[2]/a/@href
```

**etree实例化对象**

```
- 本地文件：tree = etree.parse(文件名)
                tree.xpath("xpath表达式")
- 网络数据：tree = etree.HTML(网页内容字符串)
                tree.xpath("xpath表达式")
```

**项目一**

```python
 项目需求：解析58二手房的相关数据
 #解析出一级页面的标题和二级页面的价格和描述
import requests
from lxml import etree
headers = {
    'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'
}
url = 'https://bj.58.com/changping/ershoufang/?utm_source=sem-baidu-pc&spm=105916147073.26420796294&PGTID=0d30000c-0000-17fc-4658-9bdfb947934d&ClickID=3'
page_text = requests.get(url=url,headers=headers).text
tree = etree.HTML(page_text)
li_list = tree.xpath('//ul[@class="house-list-wrap"]/li')
data = []
for li in li_list:
    #解析标题
    title = li.xpath('.//div[@class="list-info"]/h2/a/text()')[0]
    detail_page_url = li.xpath('.//div[@class="list-info"]/h2/a/@href')[0]
    if detail_page_url.split('//')[0] != 'https:':
        detail_page_url = 'https:'+detail_page_url
    detail_text = requests.get(url=detail_page_url,headers=headers).text
    tree = etree.HTML(detail_text)
    #解析二级页面的描述和价格
    desc = ''.join(tree.xpath('//div[@id="generalDesc"]//div[@class="general-item-wrap"]//text()')).strip(' \n \t')
    price = ''.join(tree.xpath('//div[@id="generalExpense"]//div[@class="general-item-wrap"]//text()')).strip(' \n \t')
    dic = {
        'title':title,
        'desc':desc,
        'price':price
    }
    data.append(dic)
#进行持久化存储
print(data)
```

**实例二**

```python
项目需求：解析图片数据：http://pic.netbian.com/4kmeinv/
import requests
from lxml import etree
from fake_useragent import UserAgent
import base64
import urllib.request
url = 'http://jandan.net/ooxx'
ua = UserAgent(verify_ssl=False,use_cache_server=False).random
headers = {
    'User-Agent':ua
}
page_text = requests.get(url=url,headers=headers).text
#查看页面源码：发现所有图片的src值都是一样的。
#简单观察会发现每张图片加载都是通过jandan_load_img(this)这个js函数实现的。
#在该函数后面还有一个class值为img-hash的标签，里面存储的是一组hash值，该值就是加密后的img地址
#加密就是通过js函数实现的，所以分析js函数，获知加密方式，然后进行解密。
#通过抓包工具抓取起始url的数据包，在数据包中全局搜索js函数名（jandan_load_img），然后分析该函数实现加密的方式。
#在该js函数中发现有一个方法调用，该方法就是加密方式，对该方法进行搜索
#搜索到的方法中会发现base64和md5等字样，md5是不可逆的所以优先考虑使用base64解密
#print(page_text)
tree = etree.HTML(page_text)
#在抓包工具的数据包响应对象对应的页面中进行xpath的编写，而不是在浏览器页面中。
#获取了加密的图片url数据
imgCode_list = tree.xpath('//span[@class="img-hash"]/text()')
imgUrl_list = []
for url in imgCode_list:
    #base64.b64decode(url)为byte类型，需要转成str
    img_url = 'http:'+base64.b64decode(url).decode()
    imgUrl_list.append(img_url)
for url in imgUrl_list:
    filePath = url.split('/')[-1]
    urllib.request.urlretrieve(url=url,filename=filePath)
    print(filePath+'下载成功')
```

**实例三**

```python
- 项目需求：解析出所有城市名称https://www.aqistudy.cn/historydata/
import requests
from lxml import etree 
url = 'https://www.aqistudy.cn/historydata/'
headers = {
    'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'
}
response = requests.get(url=url,headers=headers)
#获取页面原始编码格式
print(response.encoding)
page_text = response.text
tree = etree.HTML(page_text)
li_list = tree.xpath('//div[@class="bottom"]/ul/li | //div[@class="bottom"]/ul//li')
for li in li_list:
    city_name = li.xpath('./a/text()')[0]
    city_url = 'https://www.aqistudy.cn/historydata/'+li.xpath('./a/@href')[0]
    print(city_name,city_url)
```

现在觉得xpath特别好写啊，那么我现在告诉你，其实**xpath可以直接在网页中copy**的喔！！！

资源：

http://www.scrapyd.cn/example/
http://www.scrapyd.cn/doc/186.html
https://scrapy-chs.readthedocs.io/zh_CN/0.24/intro/tutorial.html#











