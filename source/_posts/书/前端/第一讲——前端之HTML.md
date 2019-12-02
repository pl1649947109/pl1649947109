---
title: 第一讲——前端之HTML
id: 1
date: 2019-9-6 20:00:00
tags: 前端
comment: true
---

**标签快速查找：**http://jquery.cuishifeng.cn/html5.html

### 简介

- 浏览器发请求 --> HTTP协议 --> 服务端接收请求 --> 服务端返回响应 --> 服务端把HTML文件内容发给浏览器 --> 浏览器渲染页面


- 对于不同的浏览器，对同一个标签可能有不同的解释。（兼容问题）


- HTML（超文本标记语言）是一种标记语言，它不是一种编程语言


```html
1. <!DOCTYPE html>声明为HTML5文档。

2. <html>、</html>是文档的开始标记和结束的标记。是HTML页面的根元素，在它们之间是文档的头部（head）和主体（body）。

3. <head>、</head>定义了HTML文档的开头部分。它们之间的内容不会在浏览器的文档窗口显示。包含了文档的元（meta）数据，配置信息等，是给浏览器看的，你看到的是在body标签里面写的。

4. <title>、</title>定义了网页标题，在浏览器标题栏显示。（修改一下title中的内容，然后看一下浏览器，你就会发现title是什么了）

5. <body>、</body>之间的文本是可见的网页主体内容。
```

**注意：**对于中文网页需要使用 **<meta charset="utf-8">** 声明编码，否则会出现乱码。有些浏览器会设置 GBK 为默认编码，则你需要设置为 **<meta charset="gbk">。**

- 属性：


id：定义标签的唯一ID，HTML文档树中唯一。

class：为HTML元素定义一个或者多个类名。

style：规定元素的行内样式。

- HTML的注释：<!--注释内容-->，快捷键crtl+/

<!----more---->

### head内常用的标签

```html
<title></title>定义网页标题
<style></style>定义内部样式表
<script></script>定义JS代码或引入外部的JS代码
<link/>引入外部样式表文件
<meta/>定义网页元信息
```

### Meta标签详讲

```html
<meta>元素可提供有关页面的元信息（meta-information）,针对搜索引擎和更新频度的描述和关键词。
<meta>标签位于文档的头部，不包含任何内容。
<meta>提供的信息是用户不可见的。
　　meta标签的组成：meta标签共有两个属性，它们分别是http-equiv属性和name 属性，不同的属性又有不同的参数值，这些不同的参数值就实现了不同的网页功能。 
　　
http_equiv属性：相当于http的文件头的作用，它可以向浏览器传回一些有用的信息，以帮助正确地显示网页内容，与之对应的属性值为content，content中的内容其实就是各个参数的变量值。

<!--2秒后跳转到对应的网址，注意引号-->
<meta http-equiv="refresh" content="2;URL=https://www.oldboyedu.com"> #如果把URL和后面的内容去掉，就是2秒钟刷新一次，这些内容了解一下就行
<!--指定文档的编码类型--> 
<meta http-equiv="content-Type" charset=UTF8">
<!--告诉IE以最高级模式渲染文档-->
<meta http-equiv="x-ua-compatible" content="IE=edge"> #edge是微软的一个全新的浏览器,其实就是告诉IE浏览器，你按照最高标准来渲染我的页面，了解一下就可以啦

name属性：主要用于描述网页，与之对应的属性值为content，content中的内容主要是便于搜索引擎机器人查找信息和分类信息用的。
                                                    
<meta name="keywords" content="meta总结,html meta,meta属性,meta跳转"> #关键字，也就是别人是可以通过这些关键字搜索到我的这个文章的，搜索引擎就是能够这个content内容来帮别人搜索到你的这个文档的
#SEO就是做这个的，就是怎么让你们公司的网站在别人搜索的时候能够靠前显示，不算那个花钱的，百度是充值的，你冲个20w，别人可能一天就给你点击完了，特别的贵

<meta name="description" content="xxxxxpythonxxx学习">  #是对这个文档的描述，在百度一些内容的页面上，f12打开看看
```

- 浏览器的内核:就是浏览器采用的渲染引擎，渲染引擎决定了浏览器如何显示网页的内容及网页的格式信息，渲染引擎是兼容新出现的根本原因。


```html
IE----trident

chrome----blink

火狐----gecko

Safari----webkit
```

### body内常用的标签

```html
<b>粗体</b>
<i>斜体</i>
<u>下划线</u>
<s>删除</s>
<p>段落标签</p>（注意：p标签里面为空，这一行不会出现空白行，因为没有高度撑起这一行）
<h1>标题1</h1>
<h2>标题2</h2>
<h3>标题3</h3>
<h4>标题4</h4>
<h5>标题5</h5>
<h6>标题6</h6>
<br>换行
<hr>单独的水平线
```

### 最常标签及分类

**div**：用来定义一个块级元素，并没有实际的意义，主要就是通过css样式为其赋予不同的表现。

**span**：用来定义内联（行内）元素，并无实际的意义。主要就是通过css样式为其赋予不同的表现。

这两个标签是没有特别样式的，但是这也是这两个标签最大的特点，可以通过css来控制。他们就像白白的画纸。

**块级元素与行级元素的区别**：

**块级标签（行外标签）**:独占一行，可以包涵内联标签和某些块级标签。

**内联标签（行内标签）**：不独占一行，不能包含块级标签，只能包含内联标签。

**块级标签和行级标签的分类**：

```
块级标签：div、p、h1—h6、hr(单独的水平线)、form、br(换行)
行级标签：span、b、i、u、s、a、img、select、input、texterea
```

注意：关于标签的嵌套：通常块级元素可以包含内联元素或某些块级元素，但是内联元素不能包括块级元素，它只能包含其他的内联元素。**p标签比较特殊，不能包含块级标签，p标签也不能包含p标签**。

### img标签

```html
<img src="图片的路径" alt="图片未加载成功时的提示" title="鼠标悬浮时提示信息(title不单单是img的特性)" width="宽" height="高(宽高两个属性只用一个会自动等比缩放)">
```

### a标签

超链接标签：所谓的超链接标签是指从一个网页指向一个目标的连接关系，这个目标可以是另外一个网页，也可以是相同网页上不同的位置进行跳转。

```html
<a href="http://www.baidu.com" target="_blank" >点我</a>
#target="_blank"在新的窗口打开标签页
#target="_self"在当前窗口打开标签页
```

注意：

- a标签里面没有href属性，只显示文本内容。

- a标签里面有href属性但是里面没有值，文本会显示一些特殊的效果，会刷新当前的页面。

- a标签里面有href标签，同时有值，点击文本会跳转到指定的超链接地址，可以是本页的其他地方也可以是非本页的地址。

### 列表

**无序列表**

```html
<ul type='disc'>
  <li>第一项</li>
  <li>第二项</li>
</ul>
```

　　type属性：

- disc（实心圆点，默认值）
- circle（空心圆圈）
- square（实心方块）
- none（无样式）

**有序列表**

```html
<ol type="1" start="2">
  <li>第一项</li>
  <li>第二项</li>
</ol>
```

　　type属性： start是从数字几开始

- 1 数字列表，默认值
- A 大写字母
- a 小写字母
- Ⅰ大写罗马
- ⅰ小写罗马

**标题列表**（就像大纲一样，有一个层级关系）

```html
<dl>
  <dt>标题1</dt>
  <dd>内容1</dd>
  <dt>标题2</dt>
  <dd>内容1</dd>
  <dd>内容2</dd>
</dl>
```

### 表格

　　表格是一个二维数据空间，一个表格由若干行组成，一个行又有若干单元格组成，单元格里可以包含文字、列表、图案、表单、数字符号、预置文本和其它的表格等内容。表格最重要的目的是显示表格类数据。表格类数据是指最适合组织为表格格式（即按行和列组织）的数据。

```html
<table border='1'>
  <thead> #标题部分
  <tr> #一行
    <th>序号</th> #一个单元格
    <th>姓名</th>
    <th>爱好</th>
  </tr>
  </thead>
  <tbody> #内容部分
  <tr> #一行
    <td>1</td> #一个单元格
    <td>Egon</td>
    <td>杠娘</td>
  </tr>
  <tr>
    <td>2</td>
    <td>Yuan</td>
    <td>日天</td>
  </tr>
  </tbody>
</table>
```

**效果图：**

![](http://9017499461.linshutu.top/%E5%89%8D%E7%AB%AFtable.png)

　属性:

- border: 表格边框.
- cellpadding: 内边距 （内边框和内容的距离）
- cellspacing: 外边距.（内外边框的距离）
- width: 像素 百分比.（最好通过css来设置长宽）
- rowspan: 单元格竖跨多少行（合并单元格）
- colspan: 单元格横跨多少列（合并单元格）

### input标签

　input元素会根据不同的 type 属性，变化为多种形态。

| type属性值 | 表现形式                 | 对应代码                                                     |
| ---------- | ------------------------ | ------------------------------------------------------------ |
| text       | 单行输入文本             | input type=text"                                             |
| password   | 密码输入框（不显示明文） | input type="password"                                        |
| date       | 日期输入框               | input type="date"                                            |
| checkbox   | 复选框                   | input type="checkbox" checked="checked" name='x'             |
| radio      | 单选框                   | input type="radio" name='x'                                  |
| submit     | 提交按钮                 | input type="submit" value="提交" #发送浏览器上输入标签中的内容，配合form表单使用，页面会刷新 |
| reset      | 重置按钮                 | input type="reset" value="重置"  #页面不会刷新，将所有输入的内容清空 |
| button     | 普通按钮                 | input type="button" value="普通按钮"                         |
| hidden     | 隐藏输入框               | input type="hidden"                                          |
| file       | 文本选择框               | input type="file"   （等学了form表单之后再学这个）           |

　属性:

- name：表单提交时的“键”，注意和id的区别
- value：表单提交时对应项的值
  - type="button", "reset", "submit"时，为按钮上显示的文本年内容
  - type="text","password","hidden"时，为输入框的初始值
  - type="checkbox", "radio", "file"，为输入相关联的值
- checked：radio和checkbox默认被选中的项
- readonly：text和password设置只读
- disabled：input标签无用

**总结：**

- input文本输入框，input标签如果想将数据提交到后台，**必须写name属性**。
- input选择框，必须写name属性和value属性。
- input选择框，name值相同的算是一组选择框。

### select标签

```html
<form action="" method="post">
  <select name="city" id="city">
    <option value="1">北京</option>
    <option selected="selected" value="2">上海</option> #默认选中，当属性和值相同时，可以简写一个selected就行了
    <option value="3">广州</option>
    <option value="4">深圳</option>
  </select>
</form>
```

　属性说明：

- multiple：布尔属性，设置后为多选下拉框，否则默认单选
- disabled：禁用
- selected：默认选中该项
- value：定义提交时的选项值

### label标签

label标签为 input元素定义标注，如果不用这个label给input标签一个标记，input会变黄，不影响使用，只是提醒你，别忘了给用户一些提示，也就是label标签。

- label 元素不会向用户呈现任何特殊效果。但是点击label标签里面的文本，那么和他关联的input标签就获得了光标，让你输入内容
- label标签的 for 属性值应当与相关元素的 id 属性值相同。

```html
写法一 
<form action="">
        <label for="username">用户名</label>  
        <input type="text" id="username" name="username">
    </form>
写法二
<label>
	<input type="text">
    提示信息(点击提示信息，光标自动定位到文本框的开始位置)
</label>
```

### textarea多行文本

```html
<textarea name="memo" id="memo" cols="30" rows="10">
  默认内容
</textarea>
```

属性说明：

- name：名称
- rows：行数  #相当于文本框高度设置
- cols：列数   #相当于文本框长度设置
- maxlength：最大文本内容长度
- disabled：禁用

### form标签

**功能**：

表单用于向服务器传输数据，从而实现用户与web服务器的交互

表单能够包含input系列的标签，

表单还可以包含textarea,select,filedset,label标签

**表单属性**

| 属性           | 描述                                                       |
| -------------- | ---------------------------------------------------------- |
| accept-charset | 规定在被提交表单中使用的字符集（默认：页面字符集）。       |
| action         | 规定向何处提交表单的地址（URL）（提交页面）。              |
| autocomplete   | 规定浏览器应该自动完成表单（默认：开启）。                 |
| enctype        | 规定被提交数据的编码（默认：url-encoded）。                |
| method         | 规定在提交表单时所用的 HTTP 方法（默认：GET）。            |
| name           | 规定识别表单的名称（对于 DOM 使用：document.forms.name）。 |
| novalidate     | 规定浏览器不验证表单。                                     |
| target         | 规定 action 属性中地址的目标（默认：_self）。              |

**注意**

- 想通过form表单标签提交用户输入的数据，必须在form表单里面写你的input标签，并且必须有个提交按钮，按钮有两种 （type="submit"或者button）,他们两者的效果是一样的。

### 一个大栗子

```html
<!--html的结构的学习-->
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">  <!--编码一般写在前面-->
        <meta name="keyword" content="星际1，星际2，星际3，星际4">
        <!--meta http-equiv="Refresh" Content="5"-->
        <!--meta http-equiv="Refresh" Content=5;Url="http://www.baidu.com"/-->

        <title>my html</title>

        <link rel="stylesheet" type="text/css" href="css/common.css">
        <link rel="shortcut icon" href="image/1.jpg">

    </head>
    <body>

        <div> my love </div>
        <div>my loveless</div>
        <span>you</span>
        <span><p> you2 </p></span>
        <span><br> you2 </br></span>
        <a href="http://www.baidu.com" target="_blank">跳转标签</a>

        <div>
            <a href="#id1">第一章</a>
            <a href="#id2">第二章</a>
            <a href="#id3">第三章</a>
        </div>
        <div id="id1" style="height: 1000px;background-color: red">第一章内容</div>
        <div id="id2" style="height: 1000px;background-color: green">第二章内容</div>
        <div id="id3" style="height: 1000px;background-color: blue">第三章内容</div>

        <h1>H1</h1>
        <h2>H2</h2>
        <h3>H3</h3>
        <h4>H4</h4>
        <h5>H5</h5>
        <h6>H6</h6>


    <select size="3" multiple="multiple">
        <option value="1">上海</option>
        <option value="2">北京</option>
        <option value="3">深圳</option>
    </select>
    <select>
        <optgroup label="河南省">
            <option>信阳</option>
            <option>驻马店</option>
            <option>周口</option>
        </optgroup>>
            <optgroup label="河北省">
            <option>邯郸</option>
            <option>保定</option>
            <option>石家庄</option>
        </optgroup>>
    </select>

    <input type="text" />
    <br/>
    <input type="password" />
    <br/>
    <input type="checkbox" />
    <input type="checkbox" />
    <input type="checkbox" />
    <br/>
    大：<input type="radio" name="gender"/>
    中：<input type="radio" name="gender"/>
    小: <input type="radio" name="gender"/>
    <br/>
    <input type="button" value="提交">
    <input type="submit" value="提交">
    <br/>
    <input type="file" value="文本上传">


    <br/>
    <textarea></textarea>
    <br/>

    <form action="form_action.asp" method="get">
        <p>User:<input name="nameuser"  type="text"/></p>
        <p>Password<input  name="password" type="password" /></p>
        <input  type="submit" value="提交" />
    </form>

    <label for="name2">姓名：<input id="name2" type="text"></label>

    <table border="1">
        <tr>
            <th>姓名</th>
            <th>性别</th>
            <th>爱好</th>
        </tr>
        <tr>
            <td colspan="2">张三</td>
            <td>男</td>
            <td>打球</td>
        </tr>
        <tr>
            <td rowspan="2">李四</td>
            <td>男</td>
            <td>玩游戏</td>
        </tr>
        <tr>
            <td>李思思</td>
            <td>女</td>
            <td>跑步</td>
        </tr>
    </table>

    <fieldset>
        <legend>groupbox</legend>
        <p>xxxxx</p>
    </fieldset>

    </body>
</html>
```

### 踩坑

- html文件不识别多个空格或者换行，都识别成一个空格。

### 拓展1:详述标签

```markdown
<!--基础标签-->

| 标签                                                         | 描述             |
| :----------------------------------------------------------- | :--------------- |
| [<!DOCTYPE>](https://www.w3school.com.cn/tags/tag_doctype.asp) | 定义文档类型。   |
| [<html>](https://www.w3school.com.cn/tags/tag_html.asp)      | 定义 HTML 文档。 |
| [<title>](https://www.w3school.com.cn/tags/tag_title.asp)    | 定义文档的标题。 |
| [<body>](https://www.w3school.com.cn/tags/tag_body.asp)      | 定义文档的主体。 |
| [<h1>to<h6>](https://www.w3school.com.cn/tags/tag_hn.asp)    | 定义 HTML 标题。 |
| [<p>](https://www.w3school.com.cn/tags/tag_p.asp)            | 定义段落。       |
| [<br>](https://www.w3school.com.cn/tags/tag_br.asp)          | 定义简单的换行。 |
| [<hr>](https://www.w3school.com.cn/tags/tag_hr.asp)          | 定义水平线。     |
| [<!--..-->](https://www.w3school.com.cn/tags/tag_comment.asp) | 定义注释。       |

<!--格式-->

| 标签                                                         | 描述                             |
| :----------------------------------------------------------- | :-------------- |
| [<address>](https://www.w3school.com.cn/tags/tag_address.asp) | 定义文档作者或拥有者的联系信息。 |
| [<b>](https://www.w3school.com.cn/tags/tag_font_style.asp)   | 定义粗体文本。                   |
| [<i>](https://www.w3school.com.cn/tags/tag_font_style.asp)   | 定义斜体文本。                   |
| [<small>](https://www.w3school.com.cn/tags/tag_font_style.asp) | 定义小号文本。                   |
| [<time>](https://www.w3school.com.cn/tags/tag_time.asp)      | 定义日期/时间。                  |
| [<var>](https://www.w3school.com.cn/tags/tag_phrase_elements.asp) | 定义文本的变量部分。             |

<!--表单-->

| 标签                                                         | 描述                           |
| :----------------------------------------------------------- | :----------- |
| [<form>](https://www.w3school.com.cn/tags/tag_form.asp)      | 定义供用户输入的 HTML 表单。   |
| [<input>](https://www.w3school.com.cn/tags/tag_input.asp)    | 定义输入控件。                 |
| [<textarea>](https://www.w3school.com.cn/tags/tag_textarea.asp) | 定义多行的文本输入控件。       |
| [<button>](https://www.w3school.com.cn/tags/tag_button.asp)  | 定义按钮。                     |
| [<select>](https://www.w3school.com.cn/tags/tag_select.asp)  | 定义选择列表（下拉列表）。     |
| [<optgroup>](https://www.w3school.com.cn/tags/tag_optgroup.asp) | 定义选择列表中相关选项的组合。 |
| [<option>](https://www.w3school.com.cn/tags/tag_option.asp)  | 定义选择列表中的选项。         |
| [<label>](https://www.w3school.com.cn/tags/tag_label.asp)    | 定义 input 元素的标注。        |
| [<legend>](https://www.w3school.com.cn/tags/tag_fieldset.asp) | 定义围绕表单中元素的边框。     |
| [<legend>](https://www.w3school.com.cn/tags/tag_legend.asp)  | 定义 fieldset 元素的标题。     |
| [<datalist>](https://www.w3school.com.cn/tags/tag_datalist.asp) | 定义下拉列表。                 |
| [<keygen>](https://www.w3school.com.cn/tags/tag_keygen.asp)  | 定义生成密钥。                 |
| [<output>](https://www.w3school.com.cn/tags/tag_output.asp)  | 定义输出的一些类型。           |

<!--框架-->

| 标签                                                        | 描述                     |
| :---------------------------------------------------------- | :---------- |
| [<frame>](https://www.w3school.com.cn/tags/tag_frame.asp)   | 定义框架集的窗口或框架。 |
| [<iframe>](https://www.w3school.com.cn/tags/tag_iframe.asp) | 定义内联框架。           |

<!--图像-->

| 标签                                                        | 描述                     |
| :---------------------------------------------------------- | :------- |
| [<img>](https://www.w3school.com.cn/tags/tag_img.asp)       | 定义图像。               |
| [<map>](https://www.w3school.com.cn/tags/tag_map.asp)       | 定义图像映射。           |
| [<area>](https://www.w3school.com.cn/tags/tag_area.asp)     | 定义图像地图内部的区域。 |
| [<canvas>](https://www.w3school.com.cn/tags/tag_canvas.asp) | 定义图形。               |

<!--音频/视频-->

| 标签                                                        | 描述                             |
| :---------------------------------------------------------- | :---- |
| [<audio>](https://www.w3school.com.cn/tags/tag_audio.asp)   | 定义声音内容。                   |
| [<source>](https://www.w3school.com.cn/tags/tag_source.asp) | 定义媒介源。                     |
| [<track>](https://www.w3school.com.cn/tags/tag_track.asp)   | 定义用在媒体播放器中的文本轨道。 |
| [<video>](https://www.w3school.com.cn/tags/tag_video.asp)   | 定义视频。                       |

##### 链接

| 标签                                                    | 描述                       |
| :------------------------------------------------------ | :-------- |
| [<a>](https://www.w3school.com.cn/tags/tag_a.asp)       | 定义锚。                   |
| [<link>](https://www.w3school.com.cn/tags/tag_link.asp) | 定义文档与外部资源的关系。 |
| [<nav>](https://www.w3school.com.cn/tags/tag_nav.asp)   | 定义导航链接。             |

<!--列表-->

| 标签                                                         | 描述                   |
| :----------------------------------------------------------- | :------- |
| [<ul>](https://www.w3school.com.cn/tags/tag_ul.asp)          | 定义无序列表。         |
| [<ol>](https://www.w3school.com.cn/tags/tag_ol.asp)          | 定义有序列表。         |
| [<li>](https://www.w3school.com.cn/tags/tag_li.asp)          | 定义列表的项目。       |
| [<dl>](https://www.w3school.com.cn/tags/tag_dl.asp)          | 定义定义列表。         |
| [<dt>](https://www.w3school.com.cn/tags/tag_dt.asp)          | 定义列表中的项目。     |
| [<dd>](https://www.w3school.com.cn/tags/tag_dd.asp)          | 定义列表中项目的描述。 |
| [<menu>](https://www.w3school.com.cn/tags/tag_menu.asp)      | 定义命令的菜单/列表。  |
| [<command>](https://www.w3school.com.cn/tags/tag_command.asp) | 定义命令按钮。         |

<!--表格-->

| 标签                                                         | 描述                     |
| :----------------------------------------------------------- | :--------- |
| [<table>](https://www.w3school.com.cn/tags/tag_table.asp)    | 定义表格                 |
| [<caption>](https://www.w3school.com.cn/tags/tag_caption.asp) | 定义表格标题。           |
| [<th>](https://www.w3school.com.cn/tags/tag_th.asp)          | 定义表格中的表头单元格。 |
| [<tr>](https://www.w3school.com.cn/tags/tag_td.asp)          | 定义表格中的行。         |
| [<td>](https://www.w3school.com.cn/tags/tag_td.asp)          | 定义表格中的单元。       |
| [<rhead>](https://www.w3school.com.cn/tags/tag_thead.asp)    | 定义表格中的表头内容。   |
| [<tbody>](https://www.w3school.com.cn/tags/tag_tbody.asp)    | 定义表格中的主体内容。   |

<!--样式/节-->

| 标签                                                         | 描述                          |
| :----------------------------------------------------------- | :------------ |
| [<style>](https://www.w3school.com.cn/tags/tag_style.asp)    | 定义文档的样式信息。          |
| [<div>](https://www.w3school.com.cn/tags/tag_div.asp)        | 定义文档中的节。              |
| [<span>](https://www.w3school.com.cn/tags/tag_span.asp)      | 定义文档中的节。              |
| [<header>](https://www.w3school.com.cn/tags/tag_header.asp)  | 定义 section 或 page 的页眉。 |
| [<footer>](https://www.w3school.com.cn/tags/tag_footer.asp)  | 定义 section 或 page 的页脚。 |
| [<article>](https://www.w3school.com.cn/tags/tag_article.asp) | 定义文章。                    |
| [<dialog>](https://www.w3school.com.cn/tags/tag_dialog.asp)  | 定义对话框或窗口。            |

<!--元信息-->

| 标签                                                    | 描述                                     |
| :------------------------------------------------------ | :------------ |
| [<head>](https://www.w3school.com.cn/tags/tag_head.asp) | 定义关于文档的信息。                     |
| [<meta>](https://www.w3school.com.cn/tags/tag_meta.asp) | 定义关于 HTML 文档的元信息。             |
| [<base>](https://www.w3school.com.cn/tags/tag_base.asp) | 定义页面中所有链接的默认地址或默认目标。 |
| 标签                                                        | 描述             |
| :---------------------------------------------------------- | :--------------- |
| [<script>](https://www.w3school.com.cn/tags/tag_script.asp) | 定义客户端脚本。 |
| [<object>](https://www.w3school.com.cn/tags/tag_object.asp) | 定义嵌入的对象。 |
| [<param>](https://www.w3school.com.cn/tags/tag_param.asp)   | 定义对象的参数。 |
```

### 拓展2：http协议

```markdown
[http协议](https://blog.csdn.net/qq_40890660/article/details/100170523)
```

### 拓展3：相关html操作

```markdown
[url编码](https://www.w3school.com.cn/tags/html_ref_urlencode.html)

[html特殊符号](https://www.w3school.com.cn/tags/html_ref_symbols.html)

[html字符集](https://www.w3school.com.cn/tags/html_ref_charactersets.asp)

[html颜色名](https://www.w3school.com.cn/tags/html_ref_colornames.asp)

[html画布](https://www.w3school.com.cn/tags/html_ref_canvas.asp)

[html视频/音频](https://www.w3school.com.cn/tags/html_ref_canvas.asp)

[html事件](https://www.w3school.com.cn/tags/html_ref_canvas.asp)

[html属性](https://www.w3school.com.cn/tags/html_ref_standardattributes.asp)
```

### 面试相关的

**盒子模型**

盒子模型就是布局网页的一种手段包括边框（border）、外边距（margin）、内边距（padding）、网页元素（content）、宽（width）、高（height）等元素。

外链：https://blog.csdn.net/baidu_29343517/article/details/81988791

**页面导入样式时有几种方法，以及他们之间的区别**

- link标签的引入，在当下使用最多的方式，除了能加载css外，还能定义rel、type、mdia等属性
- @import引入，@import是css提供的，只能加载css
- stype嵌入方式引入，减少页面请求（优点），但只会对当前页面有效，无法复用、会导致代码冗余，不利于项目维护（缺点），此方式一般只会在项目主站的首页使用（腾讯，淘宝）等大型网站的主页。

**简述块级标元素、内联元素、空元素有哪些，他们之间的区别**

- 块级元素：div、form、h1-h5、ul(ol(li))、dl(dt(dd))、section、p、hr
- 内联元素：span、b、i、u、s、a、img、select、input、texterea、em、strong
- 空元素：input type="hidden"、br、hr、link、meta

**简述你对HTML语义化的理解**

- 语义化是指根据内容的类型，选择合适的标签（代码语义化），即用明确的标签做正确的事情；
- html语义化让页面的内容ong结构化，结构更加清晰，有助于浏览器、搜索引擎解析对内容的抓取；
- 语义化的html在没有css的情况下也能呈现较好的内容结构和代码结构；
- 搜索引擎的爬虫也依赖于html标记来确定上下文和各个关键字的权重，利于seo