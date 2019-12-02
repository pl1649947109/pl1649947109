---
title: 第五讲——前端框架Bootstrap
id: 5
date: 2019-9-20 20:00:00
tags: 前端
comment: true
---

### Bootstrap

所有的关于Bootstrap：https://v3.bootcss.com/

**介绍**：

- Bootstrap是Twitter开源的基于HTML、CSS、JavaScript的前端框架。
- 它是为实现快速开发Web应用程序而设计的一套前端工具包。
- 它支持响应式布局，并且在V3版本之后坚持移动设备优先。
- 就是复制黏贴一把梭，html\css\js代码的封装组合

注意：记住bootstrap的导入是基于jQuery的，所以引入之前必须先引入jq。

<!----more---->

**为什么要使用Bootstrap？**

```
在Bootstrap出现之前：
　　命名：重复、复杂、无意义（想个名字费劲）
　　样式：重复、冗余、不规范、不和谐
　　页面：错乱、不规范、不和谐
在使用Bootstrap之后： 
各种命名都统一并且规范化。 页面风格统一，画面和谐。
```

**引入**

```js
<link href="bootstrap/bootstrap.css">
```

排版、按钮、表格、表单、图片等我们常用的HTML元素，Bootstrap中都提供了全局样式。

我们只要在基本的HTML元素上通过设置class就能够应用上Bootstrap的样式，从而使我们的页面更美观和谐。

#### **响应式开发**

**为什么要进行响应式开发？**

随着移动设备的流行，网页设计必须要考虑到移动端的设计。同一个网站为了兼容PC端和移动端显示，就需要进行响应式开发。

**什么是响应式**

利用媒体的询问，让同一个网站兼容不同终端（PC端和移动端）呈现不同的页面布局。

**使用的技术**

CSS3@media查询：用于查询设备是否符合某一特定条件，这些特定条件包括屏幕尺寸、是否可触摸、屏幕精度、横屏竖屏等信息。

常见的属性：

```
1.device-width, device-height 屏幕宽、高

2.width,height 渲染窗口宽、高

3.orientation 设备方向

4.resolution 设备分辨率
```

语法：

```js
@media mediatype and|not|only (media feature) {
    CSS-Code;
}
```

viewport：手机浏览器是把页面放在一个虚拟的"窗口"（viewport）中，通常这个虚拟的"窗口"（viewport）比屏幕宽，这样就不用把每个网页挤到很小的窗口中（这样会破坏没有针对手机浏览器优化的网页的布局），用户可以通过平移和缩放来看网页的不同部分。

代码：

```
<meta name=”viewport” content=”width=device-width, initial-scale=1, maximum-scale=1″>
属性：
width：控制 viewport 的大小，可以指定的一个值，如果 600，或者特殊的值，如 device-width 为设备的宽度（单位为缩放为 100% 时的 CSS 的像素）。
height：和 width 相对应，指定高度。
initial-scale：初始缩放比例，也即是当页面第一次 load 的时候缩放比例。
maximum-scale：允许用户缩放到的最大比例。
minimum-scale：允许用户缩放到的最小比例。
user-scalable：用户是否可以手动缩放。
```

Bootstrap的栅格系统：

- container
- row
- column