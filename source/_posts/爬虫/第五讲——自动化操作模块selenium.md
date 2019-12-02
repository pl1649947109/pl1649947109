---
title: 第五讲——自动化操作模块selenium
id: 5
date: 2019-9-19 20:30:00
tags: 爬虫
comment: true
---

- 图片的懒加载
- selenium的操作
- Pyppeteer模块的使用

#### 图片的懒加载

需求：抓取站长素材http://sc.chinaz.com/中的图片数据

一般的爬取思路：

```python
import requests
from lxml import etree
if __name__ == "__main__":
     url = 'http://sc.chinaz.com/tupian/gudianmeinvtupian.html'
     headers = {
         'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
     }
     #获取页面文本数据
     response = requests.get(url=url,headers=headers)
     response.encoding = 'utf-8'
     page_text = response.text
     #解析页面数据（获取页面中的图片链接）
     #创建etree对象
     tree = etree.HTML(page_text)
     div_list = tree.xpath('//div[@id="container"]/div')
     #解析获取图片地址和图片的名称
     for div in div_list:
         image_url = div.xpath('.//img/@src')
         image_name = div.xpath('.//img/@alt')
         print(image_url) #打印图片链接
         print(image_name)#打印图片名称
```

<!----more---->

解决各种疑问：

- 运行结果观察发现，我们可以获取图片的名称，但是链接获取的为空，检查后发现xpath表达式也没有问题，究其原因出在了哪里呢？
- 图片懒加载概念：
  - 图片懒加载是一种网页优化技术。图片作为一种网络资源，在被请求时也与普通静态资源一样，将占用网络资源，而一次性将整个页面的所有图片加载完，将大大增加页面的首屏加载时间。**为了解决这种问题，通过前后端配合，使图片仅在浏览器当前视窗内出现时才加载该图片，达到减少首屏图片请求数的技术就被称为“图片懒加载”。**
- 网站一般如何实现图片懒加载技术呢
  - 在网页源码中，在img标签中首先会使用一个“伪属性”（通常使用src2，original……）去存放真正的图片链接而并非是直接存放在src属性中。当图片出现到页面的可视化区域中，会动态将伪属性替换成src属性，完成图片的加载。
- 站长素材案例后续分析：通过细致观察页面的结构后发现，网页中图片的链接是存储在了src2这个伪属性中

解决之后的实例：

```python
import requests
from lxml import etree
if __name__ == "__main__":
     url = 'http://sc.chinaz.com/tupian/gudianmeinvtupian.html'
     headers = {
         'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
     }
     #获取页面文本数据
     response = requests.get(url=url,headers=headers)
     response.encoding = 'utf-8'
     page_text = response.text
     #解析页面数据（获取页面中的图片链接）
     #创建etree对象
     tree = etree.HTML(page_text)
     div_list = tree.xpath('//div[@id="container"]/div')
     #解析获取图片地址和图片的名称
     for div in div_list:
         image_url = div.xpath('.//img/@src2') #src2伪属性
         image_name = div.xpath('.//img/@alt')
         print(image_url) #打印图片链接
         print(image_name)#打印图片名称
```

#### selenium的操作

**简介**

selenium最初是一个自动化测试工具,而爬虫中使用它主要是为了解决requests无法直接执行JavaScript代码的问题 selenium本质是通过驱动浏览器，完全模拟浏览器的操作，比如跳转、输入、点击、下拉等，来拿到网页渲染之后的结果，可支持多种浏览器

**环境安装**

- 下载安装selenium：pip install selenium
- 下载浏览器驱动程序：
  - http://chromedriver.storage.googleapis.com/index.html
- 查看驱动和浏览器版本的映射关系：
  - http://blog.csdn.net/huilan_same/article/details/51896672

**简单实例**

```python
from selenium import webdriver
from time import sleep
# 后面是你的浏览器驱动位置，记得前面加r'','r'是防止字符转义的
driver = webdriver.Chrome(r'驱动程序路径')
# 用get打开百度页面
driver.get("http://www.baidu.com")
# 查找页面的“设置”选项，并进行点击
driver.find_elements_by_link_text('设置')[0].click()
sleep(2)
# # 打开设置后找到“搜索设置”选项，设置为每页显示50条
driver.find_elements_by_link_text('搜索设置')[0].click()
sleep(2)
# 选中每页显示50条
m = driver.find_element_by_id('nr')
sleep(2)
m.find_element_by_xpath('//*[@id="nr"]/option[3]').click()
m.find_element_by_xpath('.//option[3]').click()
sleep(2)
# 点击保存设置
driver.find_elements_by_class_name("prefpanelgo")[0].click()
sleep(2)
# 处理弹出的警告页面   确定accept() 和 取消dismiss()
driver.switch_to_alert().accept()
sleep(2)
# 找到百度的输入框，并输入 美女
driver.find_element_by_id('kw').send_keys('美女')
sleep(2)
# 点击搜索按钮
driver.find_element_by_id('su').click()
sleep(2)
# 在打开的页面中找到“Selenium - 开源中国社区”，并打开这个页面
driver.find_elements_by_link_text('美女_百度图片')[0].click()
sleep(3)
# 关闭浏览器
driver.quit()
```

**浏览器的创建**

Selenium支持非常多的浏览器，如Chrome、Firefox、Edge等，还有Android、BlackBerry等手机端的浏览器。另外，也支持无界面浏览器PhantomJS。

```python
from selenium import webdriverb
rowser = webdriver.Chrome()
browser = webdriver.Firefox()
browser = webdriver.Edge()
browser = webdriver.PhantomJS()
browser = webdriver.Safari()
```

**元素定位**

webdriver 提供了一系列的元素定位方法，常用的有以下几种：

```
find_element_by_id()find_element_by_name()find_element_by_class_name()find_element_by_tag_name()find_element_by_link_text()find_element_by_partial_link_text()find_element_by_xpath()find_element_by_css_selector()
```

注意

1、find_element_by_xxx找的是第一个符合条件的标签，find_elements_by_xxx找的是所有符合条件的标签。

2、根据ID、CSS选择器和XPath获取，它们返回的结果完全一致。

3、另外，Selenium还提供了通用方法`find_element()`，它需要传入两个参数：查找方式`By`和值。实际上，它就是`find_element_by_id()`这种方法的通用函数版本，比如`find_element_by_id(id)`就等价于`find_element(By.ID, id)`，二者得到的结果完全一致。

**节点交互**

Selenium可以驱动浏览器来执行一些操作，也就是说可以让浏览器模拟执行一些动作。比较常见的用法有：输入文字时用`send_keys()`方法，清空文字时用`clear()`方法，点击按钮时用`click()`方法。示例如下：

```python
from selenium import webdriver
import time
browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input = browser.find_element_by_id('q')
input.send_keys('MAC')
time.sleep(1)
input.clear()
input.send_keys('IPhone')
button = browser.find_element_by_class_name('btn-search')
button.click()
browser.quit()
```

**动作链**

在上面的实例中，一些交互动作都是针对某个节点执行的。比如，对于输入框，我们就调用它的输入文字和清空文字方法；对于按钮，就调用它的点击方法。其实，还有另外一些操作，它们没有特定的执行对象，比如鼠标拖曳、键盘按键等，这些动作用另一种方式来执行，那就是动作链。

比如，现在实现一个节点的拖曳操作，将某个节点从一处拖曳到另外一处，可以这样实现：

```python
from selenium import webdriver
from selenium.webdriver import ActionChains
import time
browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)
browser.switch_to.frame('iframeResult')
source = browser.find_element_by_css_selector('#draggable')
target = browser.find_element_by_css_selector('#droppable')
actions = ActionChains(browser)
# actions.drag_and_drop(source, target)
actions.click_and_hold(source)
time.sleep(3)
for i in range(5):
    actions.move_by_offset(xoffset=17,yoffset=0).perform()
    time.sleep(0.5)
actions.release()
```

**执行JavaScript**

```python
from selenium import webdriver
browser = webdriver.Chrome()
browser.get('https://www.jd.com/')
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
browser.execute_script('alert("123")')
```

**获取页面源码数据**

通过`page_source`属性可以获取网页的源代码，接着就可以使用解析库（如正则表达式、Beautiful Soup、pyquery等）来提取信息了。

**前进后退**

```python
#模拟浏览器的前进后退
import time
from selenium import webdriver
browser=webdriver.Chrome()
browser.get('https://www.baidu.com')
browser.get('https://www.taobao.com')
browser.get('http://www.sina.com.cn/')
browser.back()
time.sleep(10)
browser.forward()
browser.close()
```

**Cookie处理**

使用Selenium，还可以方便地对Cookies进行操作，例如获取、添加、删除Cookies等。示例如下：

```python
from selenium import webdriver
browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
print(browser.get_cookies())
browser.add_cookie({'name': 'name', 'domain': 'www.zhihu.com', 'value': 'germey'})
print(browser.get_cookies())
browser.delete_all_cookies()
print(browser.get_cookies())
```

**异常处理**

```python
from selenium import webdriver
from selenium.common.exceptions import TimeoutException,NoSuchElementException,NoSuchFrameException
try:
    browser=webdriver.Chrome()
    browser.get('http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable')
    browser.switch_to.frame('iframssseResult')
except TimeoutException as e:
    print(e)
except NoSuchFrameException as e:
    print(e)
finally:
    browser.close()
```

**phantomJS**

PhantomJS是一款无界面的浏览器，其自动化操作流程和上述操作谷歌浏览器是一致的。由于是无界面的，为了能够展示自动化操作流程，PhantomJS为用户提供了一个截屏的功能，使用save_screenshot函数实现。

```python
from selenium import webdriver
import time
# phantomjs路径
path = r'PhantomJS驱动路径'
browser = webdriver.PhantomJS(path)
# 打开百度
url = 'http://www.baidu.com/'
browser.get(url)
time.sleep(3)
browser.save_screenshot(r'phantomjs\baidu.png')
# 查找input输入框
my_input = browser.find_element_by_id('kw')
# 往框里面写文字
my_input.send_keys('美女')
time.sleep(3)
#截屏
browser.save_screenshot(r'phantomjs\meinv.png')
# 查找搜索按钮
button = browser.find_elements_by_class_name('s_btn')[0]
button.click()
time.sleep(3)
browser.save_screenshot(r'phantomjs\show.png')
time.sleep(3)
browser.quit()
```

**谷歌无头浏览器**

由于PhantomJs最近已经停止了更新和维护，所以推荐大家可以使用谷歌的无头浏览器，是一款无界面的谷歌浏览器。

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
# 创建一个参数对象，用来控制chrome以无界面模式打开
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')
# 驱动路径
path = r'C:\Users\ZBLi\Desktop\1801\day05\ziliao\chromedriver.exe'
# 创建浏览器对象
browser = webdriver.Chrome(executable_path=path, chrome_options=chrome_options)
# 上网
url = 'http://www.baidu.com/'
browser.get(url)
time.sleep(3)
browser.save_screenshot('baidu.png')
browser.quit()
```

**实例一**

需求：登录qq空间，爬取数据

```python
import requests
from selenium import webdriver
from lxml import etree
import time
driver = webdriver.Chrome(executable_path='/Users/bobo/Desktop/chromedriver')
driver.get('https://qzone.qq.com/')
#在web 应用中经常会遇到frame 嵌套页面的应用，使用WebDriver 每次只能在一个页面上识别元素，对于frame 嵌套内的页面上的元素，直接定位是定位是定位不到的。这个时候就需要通过switch_to_frame()方法将当前定位的主体切换了frame 里。
driver.switch_to.frame('login_frame')
driver.find_element_by_id('switcher_plogin').click()
#driver.find_element_by_id('u').clear()
driver.find_element_by_id('u').send_keys('328410948')  #这里填写你的QQ号
#driver.find_element_by_id('p').clear()
driver.find_element_by_id('p').send_keys('xxxxxx')  #这里填写你的QQ密码
driver.find_element_by_id('login_button').click()
time.sleep(2)
driver.execute_script('window.scrollTo(0,document.body.scrollHeight)')
time.sleep(2)
driver.execute_script('window.scrollTo(0,document.body.scrollHeight)')
time.sleep(2)
driver.execute_script('window.scrollTo(0,document.body.scrollHeight)')
time.sleep(2)
page_text = driver.page_source
tree = etree.HTML(page_text)
#执行解析操作
li_list = tree.xpath('//ul[@id="feed_friend_list"]/li')
for li in li_list:
    text_list = li.xpath('.//div[@class="f-info"]//text()|.//div[@class="f-info qz_info_cut"]//text()')
    text = ''.join(text_list)
    print(text+'\n\n\n')
driver.close()
```

**实例二**

需求：尽可能多的爬取豆瓣网中的电影信息

```python
from selenium import webdriver
from time import sleep
import time
if __name__ == '__main__':
    url = 'https://movie.douban.com/typerank?type_name=%E6%81%90%E6%80%96&type=20&interval_id=100:90&action='
    # 发起请求前，可以让url表示的页面动态加载出更多的数据
    path = r'C:\Users\Administrator\Desktop\爬虫授课\day05\ziliao\phantomjs-2.1.1-windows\bin\phantomjs.exe'
    # 创建无界面的浏览器对象
    bro = webdriver.PhantomJS(path)
    # 发起url请求
    bro.get(url)
    time.sleep(3)
    # 截图
    bro.save_screenshot('1.png')
    # 执行js代码（让滚动条向下偏移n个像素（作用：动态加载了更多的电影信息））
    js = 'window.scrollTo(0,document.body.scrollHeight)'
    bro.execute_script(js)  # 该函数可以执行一组字符串形式的js代码
    time.sleep(2)
    bro.execute_script(js)  # 该函数可以执行一组字符串形式的js代码
    time.sleep(2)
    bro.save_screenshot('2.png') 
    time.sleep(2) 
    # 使用爬虫程序爬去当前url中的内容 
    html_source = bro.page_source # 该属性可以获取当前浏览器的当前页的源码（html） 
    with open('./source.html', 'w', encoding='utf-8') as fp: 
        fp.write(html_source) 
    bro.quit()
```

**selenium规避被检测识别**

现在不少大网站有对selenium采取了监测机制。比如正常情况下我们用浏览器访问淘宝等网站的 window.navigator.webdriver的值为 

undefined。而使用selenium访问则该值为true。那么如何解决这个问题呢？

只需要设置Chromedriver的启动参数即可解决问题。在启动Chromedriver之前，为Chrome开启实验性功能参数`excludeSwitches`，它的值为`[‘enable-automation’]`，完整代码如下：

```python
from selenium.webdriver import Chrome
from selenium.webdriver import ChromeOptions
option = ChromeOptions()
option.add_experimental_option('excludeSwitches', ['enable-automation'])
driver = Chrome(options=option)
```

#### Pyppeteer模块的使用

**引言**

Selenium 在被使用的时候有个麻烦事，就是环境的相关配置，得安装好相关浏览器，比如 Chrome、Firefox 等等，然后还要到官方网站去下载对应的驱动，最重要的还需要安装对应的 Python Selenium 库，确实是不是很方便，另外如果要做大规模部署的话，环境配置的一些问题也是个头疼的事情。那么本节就介绍另一个类似的替代品，叫做 Pyppeteer。

**Pyppeteer简介**

注意，本节讲解的模块叫做 Pyppeteer，不是 Puppeteer。Puppeteer 是 Google 基于 Node.js 开发的一个工具，有了它我们可以通过 JavaScript 来控制 Chrome 浏览器的一些操作，当然也可以用作网络爬虫上，其 API 极其完善，功能非常强大。 而 Pyppeteer 又是什么呢？它实际上是 Puppeteer 的 Python 版本的实现，但他不是 Google 开发的，是一位来自于日本的工程师依据 Puppeteer 的一些功能开发出来的非官方版本。

在 Pyppetter 中，实际上它背后也是有一个类似 Chrome 浏览器的 Chromium 浏览器在执行一些动作进行网页渲染，首先说下 Chrome 浏览器和 Chromium 浏览器的渊源。

```
Chromium 是谷歌为了研发 Chrome 而启动的项目，是完全开源的。二者基于相同的源代码构建，Chrome 所有的新功能都会先在 Chromium 上实现，待验证稳定后才会移植，因此 Chromium 的版本更新频率更高，也会包含很多新的功能，但作为一款独立的浏览器，Chromium 的用户群体要小众得多。两款浏览器“同根同源”，它们有着同样的 Logo，但配色不同，Chrome 由蓝红绿黄四种颜色组成，而 Chromium 由不同深度的蓝色构成。
```

Pyppeteer 就是依赖于 Chromium 这个浏览器来运行的。那么有了 Pyppeteer 之后，我们就可以免去那些繁琐的环境配置等问题。如果第一次运行的时候，Chromium 浏览器没有安装，那么程序会帮我们自动安装和配置，就免去了繁琐的环境配置等工作。另外 Pyppeteer 是基于 Python 的新特性 async 实现的，所以它的一些执行也支持异步操作，效率相对于 Selenium 来说也提高了。

**环境安装**

- 由于 Pyppeteer 采用了 Python 的 async 机制，所以其运行要求的 Python 版本为 3.5 及以上
- pip install pyppeteer

**快速上手**

\- 爬取http://quotes.toscrape.com/js/ 全部页面数据

```
import asynciofrom pyppeteer import launchfrom lxml import etreeasync def main():    browser = await launch()    page = await browser.newPage()    await page.goto('http://quotes.toscrape.com/js/')    page_text = await page.content()    tree = etree.HTML(page_text)    div_list = tree.xpath('//div[@class="quote"]')    print(len(div_list))    await browser.close()asyncio.get_event_loop().run_until_complete(main())
```

运行结果：10
解释：launch 方法会新建一个 Browser 对象，然后赋值给 browser，然后调用 newPage 方法相当于浏览器中新建了一个选项卡，同时新建了一个 Page 对象。然后 Page 对象调用了 goto 方法就相当于在浏览器中输入了这个 URL，浏览器跳转到了对应的页面进行加载，加载完成之后再调用 content 方法，返回当前浏览器页面的源代码。然后进一步地，我们用 pyquery 进行同样地解析，就可以得到 JavaScript 渲染的结果了。在这个过程中，我们没有配置 Chrome 浏览器，没有配置浏览器驱动，免去了一些繁琐的步骤，同样达到了 Selenium 的效果，还实现了异步抓取，爽歪歪！

详细用法

- 开启浏览器

  - 调用 launch 方法即可，相关参数介绍：
    - ignoreHTTPSErrors (bool): 是否要忽略 HTTPS 的错误，默认是 False。
    - headless (bool): 是否启用 Headless 模式，即无界面模式，如果 devtools 这个参数是 True 的话，那么该参数就会被设置为 False，否则为 True，即默认是开启无界面模式的。
    - executablePath (str): 可执行文件的路径，如果指定之后就不需要使用默认的 Chromium 了，可以指定为已有的 Chrome 或 Chromium。
    - args (List[str]): 在执行过程中可以传入的额外参数。
    - devtools (bool): 是否为每一个页面自动开启调试工具，默认是 False。如果这个参数设置为 True，那么 headless 参数就会无效，会被强制设置为 False。

- 关闭提示条：”Chrome 正受到自动测试软件的控制”，这个提示条有点烦，那咋关闭呢？这时候就需要用到 args 参数了，禁用操作如下：

  ```
  browser = await launch(headless=False, args=['--disable-infobars'])
  ```

- 处理页面显示问题:访问淘宝首页

```python
 import asyncio
  from pyppeteer import launch
  async def main():
      browser = await launch(headless=False)
      page = await browser.newPage()
      await page.goto('https://www.taobao.com')
      await asyncio.sleep(10)
  asyncio.get_event_loop().run_until_complete(main())
```

发现页面显示出现了问题，需要手动调用setViewport方法设置显示页面的长宽像素。设置如下：

```python
import asyncio
  from pyppeteer import launch
  width, height = 1366, 768
  async def main():
      browser = await launch(headless=False)
      page = await browser.newPage()
      await page.setViewport({'width': width, 'height': height})
      await page.goto('https://www.taobao.com')
      await asyncio.sleep(3)
  asyncio.get_event_loop().run_until_complete(main())
```

执行js程序：拖动滚轮。调用evaluate方法。

```python
 import asyncio
  from pyppeteer import launch
  width, height = 1366, 768
  async def main():
      browser = await launch(headless=False)
      page = await browser.newPage()
      await page.setViewport({'width': width, 'height': height})
      await page.goto('https://movie.douban.com/typerank?type_name=%E5%8A%A8%E4%BD%9C&type=5&interval_id=100:90&action=')
      await asyncio.sleep(3)
      #evaluate可以返回js程序的返回值
      dimensions = await page.evaluate('window.scrollTo(0,document.body.scrollHeight)')
      await asyncio.sleep(3)
      print(dimensions)
      await browser.close()
  asyncio.get_event_loop().run_until_complete(main())
```

规避webdriver检测：

```python
  import asyncio
  from pyppeteer import launch
  async def main():
      browser = await launch(headless=False, args=['--disable-infobars'])
      page = await browser.newPage()
      await page.goto('https://login.taobao.com/member/login.jhtml?redirectURL=https://www.taobao.com/')
      await page.evaluate(
          '''() =>{ Object.defineProperties(navigator,{ webdriver:{ get: () => false } }) }''')
      await asyncio.sleep(10)
  asyncio.get_event_loop().run_until_complete(main())
```

UA伪装：

```
  await self.page.setUserAgent('xxx')
```

节点交互

```python
import asyncio
from pyppeteer import launch
async def main():
  # headless参数设为False，则变成有头模式
  browser = await launch(
      headless=False
  )
  page = await browser.newPage()
  # 设置页面视图大小
  await page.setViewport(viewport={'width': 1280, 'height': 800})
  await page.goto('https://www.baidu.com/')
  #节点交互
  await page.type('#kw','周杰伦',{'delay': 1000})
  await asyncio.sleep(3)
  await page.click('#su')
  await asyncio.sleep(3)
  #使用选择器选中标签进行点击
  alist = await page.querySelectorAll('.s_tab_inner > a')
  a = alist[3]
  await a.click()
  await asyncio.sleep(3)
  await browser.close()
asyncio.get_event_loop().run_until_complete(main())
```

**综合练习**

需求：爬取头条和网易的新闻标题

```python
  import asyncio
    from pyppeteer import launch
  from lxml import etree
  async def main():
      # headless参数设为False，则变成有头模式
      browser = await launch(
          headless=False
      )
      page1 = await browser.newPage()
      # 设置页面视图大小
      await page1.setViewport(viewport={'width': 1280, 'height': 800})
      await page1.goto('https://www.toutiao.com/')
      await asyncio.sleep(2)
      # 打印页面文本
      page_text = await page1.content()
      page2 = await browser.newPage()
      await page2.setViewport(viewport={'width': 1280, 'height': 800})
      await page2.goto('https://news.163.com/domestic/')
      await page2.evaluate('window.scrollTo(0,document.body.scrollHeight)')
      page_text1 = await page2.content()
      await browser.close()
      return {'wangyi':page_text1,'toutiao':page_text}
  def parse(task):
      content_dic = task.result()
      wangyi = content_dic['wangyi']
      toutiao = content_dic['toutiao']
      tree = etree.HTML(toutiao)
      a_list = tree.xpath('//div[@class="title-box"]/a')
      for a in a_list:
          title = a.xpath('./text()')[0]
          print('toutiao:',title)
      tree = etree.HTML(wangyi)
      div_list = tree.xpath('//div[@class="data_row news_article clearfix "]')
      print(len(div_list))
      for div in div_list:
          title = div.xpath('.//div[@class="news_title"]/h3/a/text()')[0]
          print('wangyi:',title)
  tasks = []
  task1 = asyncio.ensure_future(main())
  task1.add_done_callback(parse)
  tasks.append(task1)
  asyncio.get_event_loop().run_until_complete(asyncio.wait(tasks))
```

**实例**：模拟登录12306

```python
#对12306模拟登录
# 难点在那里：识别验证码图片并点击选择
# 工具：超级鹰
	#--注册普通用户
	#--提分充值
	#--创建一个软件（id）
	#--下载实例代码

# 实例
	#--使用selenium打开登录页面
	#--对当前打开的这张页面进行截图
	#--对挡墙图片局部区域（验证码）进行截屏
		#这么做的目的就是将验证码和模拟登录进行一一登录
	#--使用超级鹰进行图片识别打印图片的位置

bro = webdriver.Chrome(executable_path='./chromedriver')
bro.get('https://kyfw.12306.cn/otn/login/init')
time.sleep(1)
#将当前页面进行截图并保存
bro.save_screenshot('quanping.png')

#确定验证码图片对应的左上角和右下角的坐标（就是需要裁剪的区域）
code_img_ele = bro.find_element_by_xpath('//*[@id="loginForm"]/div/ul[2]/li[4]/div/div/div[3]')
location = code_img_ele.location #验证码图片左上角的坐标
size = code_img_ele.size #验证码标签对应的长和宽
print(location,size)
"""
location定位偏差
之所以会出现这个坐标偏差是因为windows系统下电脑设置的显示缩放比例造成的，location获取的坐标是按显示100%时得到的坐标，而截图所使用的坐标却是需要根据显示缩放比例缩放后对应的图片所确定的，因此就出现了偏差。
解决这个问题有三种方法：
1.修改电脑显示设置为100%。这是最简单的方法；
2.缩放截取到的页面图片，即将截图的size缩放为宽和高都除以缩放比例后的大小；
3.修改Image.crop的参数，将参数元组的四个值都乘以缩放比例。
"""

#封装左上角的坐标和右下角的坐标
rangle = (
	int(location['x']),int(location['y']),
	int(location['x']+size['width']),int(location['y']+size['height']))
#至此验证码的位置就确定下来了
i = Image.open('./quanping.png')
code_img_name = 'code.png'
#crop根据指定区域进行图片剪裁
frame = i.crop(rangle)
frame.save(code_img_name)


#将验证码图片提交到超级鹰
chaojiying = Chaojiying_Client('pl12138', '1649947109', '900795')	#用户中心>>软件ID 生成一个替换 96001
im = open('./code.png', 'rb').read()#本地图片文件路径 来替换 a.jpg 有时WIN系统须要//
s = chaojiying.PostPic(im, 9004)['pic_str'] #1902 验证码类型  官方网站>>价格体系 3.4+版 print 后要加()
print (s)

#存被点击的点的坐标
list_photo_coordinate = []
all_list = []
if  '|' in s:
    list_photo_coordinate = s.split("|")
    count_s = len(list_photo_coordinate)
    for i in range(count_s):
        xy_list = []
        x = int(list_photo_coordinate[i].split(",")[0])
        y = int(list_photo_coordinate[i].split(",")[1])
        xy_list.append(x)
        xy_list.append(y)
        all_list.append(xy_list)
else:
    xy_list = []
    x = int(s.split(",")[0])
    y = int(s.split(",")[1])
    xy_list.append(x)
    xy_list.append(y)
    all_list.append(xy_list)
print (all_list)

for i in all_list:
    x = i[0]
    y = i[1]
    ActionChains(bro).move_to_element_with_offset(code_img_ele,x,y).click().perform()
    time.sleep(0.5)

#输入用户名和密码
bro.find_element_by_id('username').send_keys('yonghuming')
bro.find_element_by_id('password').send_keys('mima')
bro.find_element_by_id('loginSub').click()
bro.quit()
```

```python
超级鹰的代码
import requests
from hashlib import md5

class Chaojiying_Client(object):

    def __init__(self, username, password, soft_id):
        self.username = username
        password =  password.encode('utf8')
        self.password = md5(password).hexdigest()
        self.soft_id = soft_id
        self.base_params = {
            'user': self.username,
            'pass2': self.password,
            'softid': self.soft_id,
        }
        self.headers = {
            'Connection': 'Keep-Alive',
            'User-Agent': 'Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0)',
        }

    def PostPic(self, im, codetype):
        """
        im: 图片字节
        codetype: 题目类型 参考 http://www.chaojiying.com/price.html
        """
        params = {
            'codetype': codetype,
        }
        params.update(self.base_params)
        files = {'userfile': ('ccc.jpg', im)}
        r = requests.post('http://upload.chaojiying.net/Upload/Processing.php', data=params, files=files, headers=self.headers)
        return r.json()

    def ReportError(self, im_id):
        """
        im_id:报错题目的图片ID
        """
        params = {
            'id': im_id,
        }
        params.update(self.base_params)
        r = requests.post('http://upload.chaojiying.net/Upload/ReportError.php', data=params, headers=self.headers)
        return r.json()


if __name__ == '__main__':
    chaojiying = Chaojiying_Client('xxx', 'xxx', 'xxx')	#用户中心>>软件ID 生成一个替换 96001
    im = open('a.jpg', 'rb').read()#本地图片文件路径 来替换 a.jpg 有时WIN系统须要//
    print (chaojiying.PostPic(im, 9004))#1902 验证码类型  官方网站>>价格体系 3.4+版 print 后要加()
```

