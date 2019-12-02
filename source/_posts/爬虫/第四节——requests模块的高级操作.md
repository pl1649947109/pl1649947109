---
title: 第四讲——requests模块的高级操作
id: 4
date: 2019-9-18 20:30:00
tags: 爬虫
comment: true
---

- 模拟登录
- cookie处理
- IP代理

#### 模拟登录

前言：相关的需求会让我们去爬取基于某些用户的相关用户信息，例如爬取张三人人网账户中的个人身份信息、好友账号信息等。那么这个时候，我们就需要对当前用户进行登录操作，登录成功后爬取其用户的相关用户信息。在浏览器中我们可以很便捷的进行用户登录操作，但是使用requests模块如何进行模拟浏览器行为的登录操作呢？

<!----more---->

需求：模拟登录人人网

分析：

```python
# #模拟登陆
# 	#--爬取基于某些用户的个人信息
# #需求：对人人网进行模拟登陆。
# 	#--点击登陆按钮之后回发起一个post请求
# 	#--post请求中会携带登陆之前录入的相关的登陆信息（用户名，密码，验证码等）
# 	#--验证码：每次请求都会发生变化
# #编码实现上面的案例
# 	#--验证码的识别，获取验证码图片上面的文字数据
# 	#--对响应数据进行持久化的存储
```

实现：

```python
#验证码识别（云打码）
def getCodeText(img_path,code_type):
    # 用户名
    username    = 'xxx'
    # 密码
    password    = 'xxx'
    # 软件ＩＤ，开发者分成必要参数。登录开发者后台【我的软件】获得！
    appid       = 8396
    # 软件密钥，开发者分成必要参数。登录开发者后台【我的软件】获得！
    appkey      = 'b4ba006c7c8c0d7ce005575b15859cb7'
    # 图片文件
    filename    = img_path
    # 验证码类型，# 例：1004表示4位字母数字，不同类型收费不同。请准确填写，否则影响识别率。在此查询所有类型 http://www.yundama.com/price.html
    codetype    = code_type
    # 超时时间，秒
    timeout     = 60
    # 检查
    result = None
    if (username == 'username'):
        print('请设置好相关参数再测试')
    else:
        # 初始化
        yundama = YDMHttp(username, password, appid, appkey)
        # 登陆云打码
        uid = yundama.login()
        print('uid: %s' % uid)
        # 查询余额
        balance = yundama.balance()
        print('balance: %s' % balance)
        # 开始识别，图片路径，验证码类型ID，超时时间（秒），识别结果
        cid, result = yundama.decode(filename, codetype, timeout)
        print('cid: %s, result: %s' % (cid, result))
    return result
if __name__ == "__main__":
    	#创建一个session对象
	session = requests.session()

	headers = {
		'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36",
		}
	#捕获验证码并保存到本地
	url = 'http://www.renren.com/SysHome.do'
	page_text = requests.get(url=url,headers=headers).text
	tree = etree.HTML(page_text)
	code_img_src = tree.xpath('//*[@id="verifyPic_login"]/@src')[0]
	code_img_data = requests.get(url=code_img_src,headers=headers).content
	with open('./code.jpg','wb') as fp:
		fp.write(code_img_data)
	#调用云打码进行图片内容的识别
	result = getCodeText('code.jpg',1004)
	print (result)
	#post请求的发送（模拟登陆）
	url_login = 'http://www.renren.com/ajaxLogin/login?1=1&uniqueTimestamp=2019622055351'
	data = {
		'email':'xxx',
		'icode':'',
		'origURL':'http: // www.renren.com / home',
		'domain':'renren.com',
		'key_id':1,
		'captcha_type':'web_login',
		'password':'40950ac8a7f8881f8045715b51e9857ecd0752ffe66894ec0205459b7e8dc20e',
		'rkey':'5b625fd2e7bb436f20f4c7905430b4e1',
		'f':'',
	}
	#使用session进行post请求的发送
	response = session.post(url=url_login,headers=headers,data=data)
	#怎么验证是否登陆成功，使用返回的状态码是否为200如果是就成功了
	print (response.status_code )
```

#### Cookie处理

**会话和Cookies**

在浏览网站的过程中，我们经常会遇到需要登录的情况，有些页面只有登录之后才可以访问，而且登录之后可以连续访问很多次网站，但是有时候过一段时间就需要重新登录。还有一些网站，在打开浏览器时就自动登录了，而且很长时间都不会失效，这种情况又是为什么？其实这里面就涉及到了会话和cookie的相关知识，本节就来揭开他们的神秘面纱。

**无状态HTTP**

HTTP的无状态指的是http协议对事物处理是没有记忆能力的，也就是说服务器不知道客户端是什么状态。当我们向服务器发送请求后，服务器解析此请求，然后返回对应的响应，服务器负责完成这个过程，而且这个过程是完全独立的，服务器不会记录前后状态的变化，也就是缺少状态记录。这就意味着如果后续需要处理前面的信息，则必须重传，这导致需要额外传递一些前面的重复请求，才能获取后续响应，然而这种效果显然不是我们想要的。为了保持前后状态，我们肯定不能将前面的请求全部重传一次，这太浪费资源了，对于这种需要用户登录页面来说，更是棘手。

这时两个保持http连接状态的技术出现了，分别是会话和cookie。会话在服务端，用来保存用户的会话信息。cookie在客户端，有了cookie，浏览器在下次访问网页时会自动附带上cookie发送给服务器，服务器识别cookie并鉴定出是哪个用户，然后在判断出用户的相关状态，然后返回对应的响应。

- 会话：会话（对象）是用来存储特定用户进行会话所需的属性及配置信息的。
- cookie：指的是某些网站为了辨别用户身份、进行会话跟踪而存储在用户本地终端上的数据。
- 会话维持：当客户端第一次请求服务器时，服务器会返回一个响应对象，响应头中带有Set-Cookie字段，cookie会被客户端进行存储，该字段表明服务器已经为该客户端用户创建了一个会话对象，用来存储该用户的相关属性机器配置信息。当浏览器下一次再请求该网站时，浏览器会把cookie放到请求头中一起提交给服务器，cookie中携带了对应会话的ID信息，服务器会检查该cookie即可找到对应的会话是什么，然后再判断会话来以此辨别用户状态。
- 形象案例：当iphone用户第一次向iphone的售后客服打电话咨询相关问题时，售后客服会针对当前用户创建一个唯一的“问题描述”，用来记录当前用户的iphone产品出现的相关问题，然后当用户阐述清楚问题后，售后客服就会将问题记录在所谓的“问题描述”中，并将“问题描述”的唯一编号通过短信告诉该用户。这样的好处就是，下次该用户的产品再次出现问题向售后电话咨询时，提供了“问题描述”的唯一编码后，就不需要在将该产品之前的问题再次进行描述了。此案例中，“问题描述”就相当于是会话，“问题描述”的唯一编码就是会话ID，短信就是cookie。

**需求：使用requests模块实现github的模拟登录**

```python
分析：
#需求：登录成功之后爬取当前用户个人主页的相关信息
	#http/https协议特性：无状态
	#没有请求到页面数据的原因：发起的第二次基于个人主页页面请求的时候，服务器端
	#并不知道该请求是基于登录状态下的请求。
	#cookie：用来记录服务器端记录客户端的相关状态。
	#处理cookie的两种方式：
		#--手动处理：作用不强
		#--自动处理：
			#--cookie的来源是哪里？
				#模拟登录post请求后，由服务器端创建。
			#session会话对象：
				#--作用：
                    #可以进行请求的发送
                    #如果请求过程中产生了cookie，则该cookie会被自动存储/携带该seesion对象中
                    
import requests
from lxml import etree
headers = {
 'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'
}
#创建会话对象,该会话对象可以调用get和post发起请求
session = requests.Session()
#使用会话对面对登录页面发起请求
page_text = session.get(url='https://github.com/login',headers=headers).text
#解析出动态的taken值
tree = etree.HTML(page_text)
t = tree.xpath('//*[@id="login"]/form/input[2]/@value')[0]
#指定模拟登录请求的url
url = 'https://github.com/session'
#参数封装（处理动态taken值）
data = {
 'commit': 'Sign in',
 'utf8': '✓',
 'authenticity_token': t,
 'login': 'xxx@sina.com',
 'password': 'xxx',
 'webauthn-support': 'supported',
}
#使用会话对象进行模拟登录请求发送（携带cookie）
page_text = session.post(url=url,headers=headers,data=data).text
#持久化存储
with open('./git.html','w',encoding='utf-8') as fp:
 fp.write(page_text)
```

#### IP代理

**前言：**

我们在做爬虫的过程中经常会遇到这样的情况，最初爬虫正常运行，正常抓取数据，一切看起来都是那么美好，然而一杯茶的功夫可能就会出现错误，比如403，这时打开网页一看，可能会看到“您的IP访问频率太高”这样的提示。出现这种现象的原因是网站采取了一些反爬措施。比如，服务器会检测某个IP在单位时间内请求的次数，如果超过了某个阈值，就会直接拒绝服务，返回一些错误信息，这种情况可以称为封IP。

既然服务器检测的是某个IP单位时间的请求次数，那么借助某种方式来伪装我们的IP，让服务器识别不出是由我们本机发起的请求，不就可以成功防止封IP了吗？一种有效的方式就是使用代理。

**什么是代理**

代理实际上指的就是代理服务器，它的功能就是代理网络用户去取得网络信息。形象的说，它是网络信息的中转站。在我们正常请求一个网站时，是发送了请求给Web服务器，Web服务器把响应传回我们。如果设置了代理服务器，实际上就是在本机和服务器之间搭建了一个桥梁，此时本机不是直接向Web服务器发起请求，而是向代理服务器发出请求，请求会发送给代理服务器，然后代理服务器再发送给Web服务器，接着由代理服务器再把Web服务器返回的响应转发给本机。这样我们同样可以正常访问网页，但这个过程中Web服务器识别出的真实IP就不再是我们本机的IP了，就成功实现了IP伪装，这就是代理的基本原理。

**代理的作用**

- 突破自身IP访问的限制，访问一些平时不能访问的站点。
- 隐藏真实IP，免受攻击，防止自身IP被封锁

实例：

```python
import requests
import random
if __name__ == "__main__":
    #不同浏览器的UA
    header_list = [
        # 遨游
        {"user-agent": "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Maxthon 2.0)"},
        # 火狐
        {"user-agent": "Mozilla/5.0 (Windows NT 6.1; rv:2.0.1) Gecko/20100101 Firefox/4.0.1"},
        # 谷歌
        {
            "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_0) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11"}
    ]
    #不同的代理IP
    proxy_list = [
        {"http": "112.115.57.20:3128"},
        {'http': '121.41.171.223:3128'}
    ]
    #随机获取UA和代理IP
    header = random.choice(header_list)
    proxy = random.choice(proxy_list)
    url = 'http://www.baidu.com/s?ie=UTF-8&wd=ip'
    #参数3：设置代理
    response = requests.get(url=url,headers=header,proxies=proxy)
    response.encoding = 'utf-8'
    with open('daili.html', 'wb') as fp:
        fp.write(response.content)
```

