---
title: 前端拓展一：css之::before和::after的实际用例
id: 6
date: 2019-9-21 20:00:00
tags: 前端
comment: true
---

以下例子多数是在特定平台上使用过的，未做兼容处理，建议在chrome下浏览

### 1.间隔符用法

如文章最开始的例子，使用::after伪元素做间隔符，并使用伪类:not排除掉最后一个元素。

效果：http://htmlpreview.github.io/?https://github.com/laden666666/css-before-and-after-test/blob/master/example1.html

### 2.做border三角图标

很多开发者都用过border做的三角图标，本身三角符号就不属于文档，使用伪元素做三角符最合适了。

效果：http://htmlpreview.github.io/?https://github.com/laden666666/css-before-and-after-test/blob/master/example2.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        p{
            width: 200px;
            height: 50px;
            line-height: 50px;
            background: red;
        }
        p::before{
            display: inline-block;
            border: 5px solid transparent;
            border-right-color: red;
            content: "";
            position: relative;
            left: -10px;
        }
    </style>
</head>
<body>
    <p></p>
</body>
</html>
```

<!---more---->

### 3.字符图标

最近笔者在开发微信小程序，因为微信小程序不支持svg和背景图，于是笔者大量使用字符图标，感觉字符图标非常方便，就是受设备系统字体库限制。

效果：http://htmlpreview.github.io/?https://github.com/laden666666/css-before-and-after-test/blob/master/example3.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .search{
            position: relative;
        }
        .search::before{
            content: "☌";
            transform: rotate(180deg);
            float: left;
            font-size: 25px;
            position: absolute;
            line-height: 30px;
            left: 5px;
        }
        .search input{
            display: block;
            padding-left: 25px;
            height: 30px;
            box-sizing: border-box;
        }
    </style>
</head>
<body>
    <div class="search">
        <input placeholder="请输入搜索词" />
    </div>
</body>
</html>
```

### 4.webfont的图标

现在webfont图标的最佳实践就是使用i标签和::after或者::before，实现这种图标最佳实践的工具非常多，比如http://fontello.com/，从这个网站我们可以下载svg的图标库。这种例子太多了，这里就不再列举。

### 5.做单位、标签、表单必填标准

笔者一直认为表单输入框的必填标记（往往是红色的“*”字符），不应该放到文档当中，使用::before可以很优雅地解决这个问题（其实就是字符图标的进一步应用）。

对于单位和前（后）置标签，也可以这样做。但是多数情况下不推荐这种做法，因为单位和标签应该是文档的一部分。

效果：http://htmlpreview.github.io/?https://github.com/laden666666/css-before-and-after-test/blob/master/example5.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        label{
            display: block;
        }
        .required::before{
            content: "*";
            color: red;
            display: inline-block;
            width: 0;
            position: relative;
            left:-5px;
        }
        .unit::after{
            content: "万元";
        }
    </style>
</head>
<body>
    <label class="required">姓名 <input /> </label>
    <label class="unit">金额 <input /> </label>
</body>
</html>
```

### 6.做一些效果

可以参考[《理解伪元素 :before 和 :after》](http://blog.jobbole.com/49173/)这篇文章的效果，笔者曾经在实际项目中使用过“迷人的阴影”效果，也曾在微场景开发中实现过一些类似的动画。

效果：http://htmlpreview.github.io/?https://github.com/laden666666/css-before-and-after-test/blob/master/example6.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        .flashlight{
            position: relative;
        }
        .flashlight::after,.flashlight::before{
            position: absolute;
            width: 10px;
            height: 10px;
            box-sizing: border-box;
            border-radius: 10px;
            background: red;
            content: "";
            margin: auto;
            bottom: 0;
            top: 0;
            left: 0;
            right: 0;
            z-index: -1;
        }
        .flashlight::after{
            animation:flashlight-after 2s linear 0s infinite;
            -webkit-animation:flashlight-after 2s linear 0s infinite;
        }
        .flashlight::before{
            animation:flashlight-before 2s linear 0s infinite;
            -webkit-animation:flashlight-before 2s linear 0s infinite;
        }
        @keyframes flashlight-before
        {
            0% {transform: scale(1,1);opacity: 0.5}
            50% {transform: scale(10,10);opacity: 0}
            100% {transform: scale(1,1);opacity: 0}
        }
        @-webkit-keyframes flashlight-after
        {
            0% {transform: scale(1,1);opacity: 0}
            15% {transform: scale(1,1);opacity: 0.5}
            50% {transform: scale(5,5);opacity: 0}
            100% {transform: scale(1,1);opacity: 0}
        }
    </style>
</head>
<body>
    <span class="flashlight">我是会闪光的文字</span>
</body>
</html>
```

### 7.实现一些标签对placeholder的支持

只有几个标签支持placeholder，而且如<input type='date' />虽然是input但是也不支持。使用::before可以让一部分标签也实现对placeholder属性的支持。

效果：http://htmlpreview.github.io/?https://github.com/laden666666/css-before-and-after-test/blob/master/example7.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .text{
            border: 1px solid #000;
            padding: 5px;
        }
        .text:empty::before{
            content: attr(placeholder);
            color: darkgrey;
        }
    </style>
</head>
<body>
    <div class="text" placeholder="请输入文本" contenteditable="true"></div>

</body>
</html>
```

### 8.实现文字图片居中对齐

优雅地实现[张鑫旭老师的inline-box居中方法](http://www.zhangxinxu.com/wordpress/2009/08/大小不固定的图片、多行文字的水平垂直居中/)，使用一个高度为100%的::before将自身的对齐线移动到自己的中线，这样里面的所有内联元素都居中对齐了。

效果：http://htmlpreview.github.io/?https://github.com/laden666666/css-before-and-after-test/blob/master/example8.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .div{
            height: 100px;
            background: blue;
        }
        .middle::after{
            vertical-align: middle;
            display: inline-block;
            content: "";
            height: 100%;
        }
        img{
            background: red;
            width: 30px;
            height: 30px;
            vertical-align: middle;
            line-height: 100px;
        }
    </style>
</head>
<body>
    <div class="div">
        <img />
    </div>
    <hr/>
    <div class="middle div">
        <img />
    </div>
</body>
</html>
```

### 9.清除浮动

这个很常用，bootstrap的`clearfix类就是使用这个方法。`

效果：http://htmlpreview.github.io/?https://github.com/laden666666/css-before-and-after-test/blob/master/example9.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .div{
            width: 100%;
            height: 10px;
            background: blue;
        }
        .clearfix::before,.clearfix::after {
             content: "";
             display: table;
             clear: both;
        }
        .float{
            float: left;
            width: 40px;
            height: 40px;
            background: red;
        }
    </style>
</head>
<body>
    <table width="200">
        <tr>
            <td>
                <div>
                    <div class="float"></div>
                </div>
                <div class="div"></div>
            </td>
        </tr>
        <tr>
            <td>
                <div class="clearfix">
                    <div class="float"></div>
                </div>
                <div class="div"></div>
            </td>
        </tr>
    </table>
</body>
</html>
```

### 10.使用pointer-events消除伪元素事件

之前提到过，伪元素::after和::before会替所在元素捕获用户事件，有时候这并非我们想要的，因为这样会影响被::after和::before覆盖的子节点或者兄弟节点捕获用户事件，使用pointer-events可以消除这种问题。

效果：http://htmlpreview.github.io/?https://github.com/laden666666/css-before-and-after-test/blob/master/example10.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .parent{
            width: 100px;
            height: 100px;
            background: red;
            position: relative;
        }
        .parent::after{
            position: absolute;
            width: 40px;
            height: 40px;
            content: '请点击我';
            background: blue;
            top: 0;
            left: 0;
            color: #fff;
        }
        .pointer-events::after{
            pointer-events: none;
        }
    </style>
</head>
<body>
    <div class="parent">
        <p>文字文字文字文字</p>
    </div>
    <div class="parent pointer-events">
        <p>文字文字文字文字</p>
    </div>

    <script>
        document.querySelectorAll(".parent").forEach(function (parent) {
            parent.addEventListener("click",function () {
                alert("点击的父容器");
            })
        })
        document.querySelectorAll("p").forEach(function (parent) {
            parent.addEventListener("click",function (event) {
                event.stopPropagation();
                alert("点击的是里面的文字");
            })
        })
    </script>
</body>
</html>
```

简单就分享这么多，总之使用伪元素的核心是更利于语义化，这是我们活用::after和::before的前提，否则就是胡乱使用了。总体可以分为四种用法：

1.用css创建装饰性元素

2.用css创建用于布局的元素，实现特殊布局的特殊需要

3.做显示图标的实现手段

4.配合attr显示属性值

