---
title: 第二讲——前端之CSS
id: 2
date: 2019-9-10 20:00:00
tags: 前端
comment: true
---

**css快速查找**：http://css.cuishifeng.cn/

**CSS手册**：https://www.runoob.com/cssref/css-reference.html

**包含所有的css的实例代码**：https://www.runoob.com/css/css-examples.html

### 简介

**CSS（层叠样式表）**:定义如何显示HTML元素，给HML设置样式，让它更加美观。当浏览器读到一个样式表，它会按照这个样式表来对文档进行格式化（就是我们常说的渲染）

**CSS的组成**：选择器+声明（属性+属性值）

**语法结构**：

![](http://9017499461.linshutu.top/%E5%89%8D%E7%AB%AFcss%E6%A0%BC%E5%BC%8F.png)

**注释**：

```
/*没错，我就是注释*/
```

<!----more---->

### css的几种引入方式

**行内样式**

```
<p style="color: red">Hello world.</p>
```

总结：行内式是标记的style属性中设定的css样式，不推荐大规模使用

**内部样式**

```
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        p{
            background-color: #2b99ff;
        
    </style>
</head>
```

总结：嵌入式是将css样式集中写在网页的**head标签的style标签对**中。

**外部样式**

```
<link href="mystyle.css" rel="stylesheet" type="text/css"/> 
#href是我们css文件的路径，一般放在同一级
#rel表示引入的类型
```

总结：外部样式就是将css写在一个单独的文件中，然后再页面进行引入即可，推荐使用这种写法

### CSS选择器(找到对应的标签）

#### 基本选择器

**元素选择器（标签名，大范围）**

```
p {cloro: "green";}
/*找到页面所有的p标签*/
```

**ID选择器（按照id属性来找到对应的标签，小范围）**

```
#i1 {background-color: red;}
/*#号表示id属性，后面的i1表示属性的值，这样就可以唯一的确定某一个标签*/
```

**类选择器（按照一类的属性来找到对应的标签，中范围）**

```
.c1 {font-size: 14px;}
/*.表示class属性，c1表示class的值*/
p.c1 {color: red;}
/*找到所有p标签里面含有class属性的值为c1的p标签，渲染成红色*/
```

注意：样式类名不要使用数字开头（有的浏览器不认）

标签中的class属性如果有多个，要使用空格分隔

**总结：#代表id(不可重复)；.代表class(可重复)**

#### 组合选择器

**后代选择器**：

```
格式：选择器 空格 选择器
.c1 a{color: green;}
/*找到class类c1内部的a标签*/
#普通的选择器组合起来的构成的
```

**儿子选择器**

```
.c1>a{color: green;}
/*选择所有父级是 <class="c1"> 元素的 <a> 元素*/
```

**毗邻选择器**

```
.c1+p{color: green;}
/*找到<class="c1">下面紧挨着的第一个p标签/
```

**弟弟选择器**

```
.c1~p{color: green;}
/*找到<class="c1"后面所有的兄弟p标签>*/
```

#### 属性选择器

通过标签属性来查找对应的标签，这个属性是我们自己定义的，不是id或者class这种html自带的属性。

```
[属性]
[属性=值]
p[属性]{xx:xx}  找到xxx属性的所有标签
p[属性=值]{xx:xx}   找到所有xxx属性的并且属性值为xx的所有标签
实例：
/*用于选取带有指定属性的元素。*/
p[title] {
  color: red;
}
/*用于选取带有指定属性和值的元素。*/
p[title="213"] {
  color: green;
}
```

#### 分组和嵌套

**分组选择器**

```
div,p{xx:xx}
解释：div选择器和p选择器找到的所有标签设置共同的样式
过滤：
div.c1{属性:属性值;}找到类标签是c1的所有的div标签
```

**嵌套选择器**

和组合选择器是一样的

#### 伪类选择器

可以根据标签的不同状态再进行进一步的区分，比如一个a标签，点击前，点击时，点击后有不同的三个状态。

```
示例代码:
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        a:link{  /* a标签访问前设置样式 */
            color:red;
        }
        a:active{  /* a标签鼠标点下去显示样式 */
            color: green;
        }
        a:visited{ /* a标签访问后显示样式 */
            color: pink;
        }
        a:hover{ /* 鼠标悬浮到a标签时显示样式 */
            color:purple;
        }
        div:hover{   /* 鼠标悬浮到div标签时显示样式 */
            background-color: green;
        }
        input:focus{ /* input标签捕获光标时的样式显示 */
            background-color: orange;
        }
    </style>
</head>
<body>
    <a href="http://www.92py.com/">校草网</a>
    <div>
    </div>
    <input type="text">
</body>
</html>
```

小结：主要是提供给a标签的功能

#### 伪元素选择器

通过css来造标签。

first-letter:常用来给字母设置特殊样式

```
将p标签中的文本的第一个字变颜色成红色的大字
p:first-letter{
	font-size:48px;
	color:red;
}
```

before：在标签的前面插入内容

```
在p标签前面添加一些内容
p:brfore{
	content:"在p标签前面插入内容";
	color:red;
}
在页面的所有标签前面加入一些内容
::bofore{
	content:"在页面所有内容的前面加入一些内容"
}
```

after：在标签的后面插入内容

```
在p标签后面添加一些内容
p:after{
	content:"在p标签后面插入内容";
	color:red;
}
在页面的所有标签后面加入一些内容
::after{
	content:"在页面所有内容的后面加入一些内容"
}
```

#### 选择器总结

|      例子       |                   例子描述                    | CSS  |
| :-------------: | :-------------------------------------------: | :--: |
|     .intro      |        选择 class="intro" 的所有元素。        |  1   |
|   #firstname    |       选择 id="firstname" 的所有元素。        |  1   |
|        *        |                选择所有元素。                 |  2   |
|        p        |              选择所有 <p> 元素。              |  1   |
|      div,p      |     选择所有 <div> 元素和所有 <p> 元素。      |  1   |
|      div p      |     选择 <div> 元素内部的所有 <p> 元素。      |  1   |
|      div>p      |   选择父元素为 <div> 元素的所有 <p> 元素。    |  2   |
|      div+p      |  选择紧接在 <div> 元素之后的所有 <p> 元素。   |  2   |
|    [target]     |        选择带有 target 属性所有元素。         |  2   |
| [target=_blank] |       选择 target="_blank" 的所有元素。       |  2   |
| [title~=flower] | 选择 title 属性包含单词 "flower" 的所有元素。 |  2   |
|   [lang\|=en]   |   选择 lang 属性值以 "en" 开头的所有元素。    |  2   |
|     a:link      |           选择所有未被访问的链接。            |  1   |
|    a:visited    |           选择所有已被访问的链接。            |  1   |
|    a:active     |                选择活动链接。                 |  1   |
|     a:hover     |         选择鼠标指针位于其上的链接。          |  1   |
|   input:focus   |          选择获得焦点的 input 元素。          |  2   |
| p:first-letter  |          选择每个 <p> 元素的首字母。          |  1   |
|  p:first-line   |           选择每个 <p> 元素的首行。           |  1   |
|  p:first-child  | 选择属于父元素的第一个子元素的每个 <p> 元素。 |  2   |
|    p:before     |      在每个 <p> 元素的内容之前插入内容。      |  2   |
|     p:after     |      在每个 <p> 元素的内容之后插入内容。      |  2   |

#### CSS选择器的优先级

权重的优先级计算方式：

![](http://9017499461.linshutu.top/%E5%89%8D%E7%AB%AFcss%E9%80%89%E6%8B%A9%E5%99%A8%E6%9D%83%E9%87%8D%E4%BC%98%E5%85%88%E7%BA%A7.png)

```
最高级别！important
div{color:red!important;}
```

小 结：

- 权重越高，对应选择器的样式会被优先显示。
- 组合选择器，各选择器的权重相加。
- 权重不进位，就是说权重加起来等于11的类选择器组合在一起，也没有一个id选择器的优先级大，小就是小。
- 默认css样式是可以继承的，继承的权重为0。
- 权重相同的选择器，谁后写的就使用谁。

### CSS属相相关

#### 高和宽

width属性可以为元素设置高度

height属性可以为元素设置高度

**块级标签才能设置宽度，内联标签的宽度由内容来决定**

#### 字体的属性

**文字字体(font-family)**

font-family可以把多个字体名称作为一个“回退”系统来保存。**如果浏览器不支持第一个字体，则会尝试下一个。浏览器会使用它可识别的第一个值。如果没有就使用系统默认的字体**

```
body {
  font-family: "Microsoft Yahei", "微软雅黑", "Arial", sans-serif
}
```

**文字大小(font-size)**

```
p {
  font-size: 32px;
}
```

默认的自I提大小是16px,如果设置成inherit表示继承父元素字体大小。

**字重（粗细font-weight）**

|     值      |                      描述                      |
| :---------: | :--------------------------------------------: |
|   normal    |                默认值，标准粗细                |
|    bold     |                      粗体                      |
|   bolder    |                      更粗                      |
|   lighter   |                      更细                      |
| **100~900** | 设置具体粗细，400等同于normal，而700等同于bold |
|   inherit   |             继承父元素字体的粗细值             |

**文本颜色(color)**

三种方式：

十六进制：#红-绿-蓝;（#FF0000）

RGB值：RGB(红，绿，蓝); RGB（255,0,0）

颜色名称：red、blue、green;

#### **文字属性**

**文字对齐(text-align)**

text-align属性规定元素中的文本的水平对齐方式。

line-height:垂直对齐方式（就是在标签的竖直的方向），属性值是像素。

|   值   |      描述       |
| :----: | :-------------: |
|  left  | 左边对齐 默认值 |
| right  |     右对齐      |
| center |    居中对齐     |

**文字装饰(text-decoration)**

text-decoration属性用来给文字添加特殊的效果

|      值      |                 描述                  |
| :----------: | :-----------------------------------: |
|     none     |        默认。定义标准的文本。         |
|  underline   |         定义文本下的一条线。          |
|   overline   |         定义文本上的一条线。          |
| line-through |       定义穿过文本下的一条线。        |
|   inherit    | 继承父元素的text-decoration属性的值。 |

最大作用：**常用的为a标签去掉默认的自化线**

```
a{
	text-decoration:none;
}
```

**首行缩进(text-indent)**

将段落的第一行缩进32像素（一个字默认是16px）

```
p {
  text-indent: 32px; #首行缩进两个字符，因为我记得一个字在页面上的默认大小为16px
}
```

#### 背景属性

```python
#设置背景颜色 
background-color: blue;  
#背景图片,url属性值为图片路径 
background-image: url("meinv.jpg");   
#图片是否平铺,默认是平铺的,占满整个标签 
background-repeat: no-repeat;
     repeat(默认):背景图片沿着x轴和y轴重复平铺，铺满整个包裹它的标签
     repeat-x：背景图片只在水平方向上平铺
     repeat-y：背景图片只在垂直方向上平铺
     no-repeat：背景图片不平铺
#图片位置 
background-position: right bottom;   (九宫格的形式)
#图片位置,100px是距离左边的距离,50px是距离上面的距离 
background-position: 100px 50px;  
```

简写方式：

```
background: yellow url("meinv.jpg") no-repeat 100px 50px;
背景颜色 背景图片路径 是否平铺 图片位置
```

#### 边框

**边框属性**

border-width 宽度

border-style 样式

border-color 颜色

border-style属性

|   值   |      描述      |
| :----: | :------------: |
|  none  |    无边框。    |
| dotted | 点状虚线边框。 |
| dashed | 矩形虚线边框。 |
| solid  |   实线边框。   |

```
这样可以设置每一条边框的样式：
border-left:10px solid yellow ;
border-right:10px dashed red ;
broder-top:10px solid red;
broder-buttom:10px dashed yollow;
简写方式（四条边都是一样的）
#i1 {
  border: 2px solid red;
}
```

圆角边框

broder-radius：5%   圆角边框

broder-radius：50%    圆（高度和宽度是一样的）

broder-radius：50%    椭圆（高度和宽度不一样）

#### display属性

用于控制HTML元素的显示效果（转换块级标签和内联标签）

| 值                     | 意义                                                         |
| ---------------------- | ------------------------------------------------------------ |
| display:"none"         | HTML文档中元素存在，但是在浏览器中不显示。一般用于配合JavaScript代码使用。 |
| display:"block"        | 默认占满整个页面宽度，如果设置了指定宽度，则会用margin填充剩下的部分。 |
| display:"inline"       | 按行内元素显示，此时再设置元素的width、height、margin-top、margin-bottom和float属性都不会有什么影响。 |
| display:"inline-block" | 使元素同时具有行内元素和块级元素的特点。                     |

隐藏标签

visibility:hidden:可以隐藏某个元素，但隐藏的元素仍然占用与为隐藏之前一样的空间，即使隐藏了但是也影响布局空间，就是说页面还是被占用了。

display:none:可以隐藏某个元素，且隐藏的元素不会占用任何空间（可以用作动态效果）。

### CSS盒子模型

content：内容，width和weight是内容的高度和宽度。

padding：内边距，内容和边框之间的距离。

```
内边距的设置：
padding-left: 10px;
padding-top: 8px;
padding-right: 5px;
padding-bottom: 5px;
简写方式：
padding:10px 8px 5px 5px;
```

border：边框(同上面的边框的设置)

margin：外边距，标签之间的距离，如果两个标签都设置了margin，选最大的值，作为双方之间的距离

```
外边距的设置：
margin-left:10px;
margin-top:2px;
margin-right:8px;
margin-buttom:20px;
简写方式：
margin:10px 2px 8px 20px;
```

占用页面的空间大小：content+padding+border

![](http://9017499461.linshutu.top/%E5%89%8D%E7%AB%AFcss%E7%9B%92%E5%AD%90%E6%A8%A1%E5%9E%8B.png)

实例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .c1{
            width: 100px;
            height: 100px;
            border: 10px solid red;
            /*padding: 20px 20px; !* 内边距,内容和边框之间的距离 *!*/
            padding: 8px 2px 3px 6px; /* 上右下左 */
            margin: 20px 10px;

        }
        .c2{
            width: 100px;
            height: 100px;
            border: 10px solid green;
            margin: 10px 2px 6px 8px; /* 外边距,与其他标签的距离,如果旁边没有标签,按照父级标签的位置进行移动 */
        }
        .c3{
            width: 100px;
            height: 100px;
            border: 1px solid blue;
        }
    </style>
</head>
<body>
<div class="c1">
    div1
</div>
<div class="c2">
    div2
</div>
<div class="c3">
    div3
</div>
</body>
</html>
```

### float（浮动）

在CSS中，任何元素都可以浮动，浮动就像word里面的文字环绕的效果。现在多数用来做网页布局的。

**浮动一般用来进行页面布局**

- 浮动会脱离正常文档流
- 浮动会造成父级标签塌陷问题

**清除浮动（解决塌陷问题）**

三种方式：

- 固定高度，在胡标签里面加一个其他的标签
- 伪元素清除css解决（常用）
- overflow:hidden给塌陷的父级标签设置这个属性就可以清除浮动。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>css</title>
    <style>
        .c1{
            width:100px;
            height: 100px;
            background-color: red;
            float: left;
        }
        .c2{
            width:100px;
            height: 100px;
            background-color: red;
            float: right;
        }
        .c3{
            height: 100px;
            background: pink;
            /*clear: both;*/    /*clear清除浮动*/
        }
        /*伪元素清除法*/
        .clearfix:after{
            content:'';
            display: block;
            clear:both;
        }
        /*固定高度法*/
       /* .cc{
            height:100px;
        }*/
    </style>
</head>
<body>
    <div class="clearfix">
        <div class="c1">div1</div>
        <div class="c2">div2</div>
    </div>
    <div class="c3">div3</div>
</body>
</html>
```

注意：一般业内约定俗称，都把这个清楚浮动的class属性命名为clearfix，只要一看到这个就知道它是用来清除浮动的。

**clear**

clear属性固定元素的哪一侧不允许其他浮动元素。

| 值      | 描述                                  |
| ------- | ------------------------------------- |
| left    | 在左侧不允许浮动元素。                |
| right   | 在右侧不允许浮动元素。                |
| both    | 在左右两侧均不允许浮动元素。          |
| none    | 默认值。允许浮动元素出现在两侧。      |
| inherit | 规定应该从父元素继承 clear 属性的值。 |

总结：为什么要有浮动啊，是想做页面布局，但是浮动有副作用，父级标签塌陷，所以要想办法去掉这个副作用，使用了clear来清除浮动带来的副作用，我们当然也可以通过设置标签为inline-block来实现这种布局效果，但是把一个块级标签变成一个类似内敛标签的感觉，不好操控，容易混乱，所以一般都用浮动来进行布局。

### overflow溢出属性

什么是溢出属性：就是我们定义了一个块级的标签，我们往里面写的内容超出了这个标签的空间，就会出现一些内容的溢出。

| 值      | 描述                                                     |
| ------- | -------------------------------------------------------- |
| visible | 默认值。内容不会被修剪，会呈现在元素框之外。             |
| hidden  | 内容会被修剪，并且其余内容是不可见的。                   |
| scroll  | 内容会被修剪，但是浏览器会显示滚动条以便查看其余的内容。 |
| auto    | 如果内容被修剪，则浏览器会显示滚动条以便查看其余的内容。 |
| inherit | 规定应该从父元素继承 overflow 属性的值。                 |

- overflow（水平和垂直均设置）
- overflow-x（设置水平方向，只出现x轴的滚动条）
- overflow-y（设置垂直方向，只出现y轴的滚动条）

**圆形图案**

overflow:hidden（剪裁原型图片里面的内容）

```
.c1{
	width:100px;
	heifht:100px;
	border-radius:50%; #圆形的边框
	border:1px solid red;
	overflow:hidden;  #溢出的内容隐藏
}
```

### 定位

```
static定位（无定位，默认的）
相对定位：position:relativ
解释：相对自己原来的位置进行移动，原来的空间还占着

绝对定位：position:absolute
解释：不占用自己原来的位置，移动的时候如果父级标签以及祖先辈标签如果设置了相对定位，父级标签或者祖先级标签进行移动，如果父级标签都没有设置相对定位,那么就按照整个文档进行移动。

固定位置：position:fixed
解释：不管页面怎么动，都在整个屏幕的某个位置

所有定位的元素移动，都是通过top,left,right,bottom两个方向的值来移动。
往上移动:top:-100px（注意是负值）或者bottom：-100px（负值），往左移动:left:-100px（也是负值）或者right：-100px，
往下移动:bottom：100px（正值）或者top：100px（正值），
往右移动:right:100px（正值）或者left：100px。
```

实例：回到顶部

```
<div class="top">
	<span>回到顶部</span>
</div>

.top{
    height:30px;
    width:80px;
    background: lightskyblue; 
    display: inline-block; /*调整框的大小*/
    position: fixed;  /*使用固定位置，让回到顶部的按钮一直停留在当前可是的页面范围里面*/
    right: 40px; /*设定回到顶部的框位于当前可以页面的某个位置，这里设置的是距离右边的距离是40像素和距离下面的长度是40像素*/
    bottom: 40px;

}
.top span{
    position: absolute; /*设定里面的字体是绝对路径，跟随外面的大框位置 移动，同时设置它在里面的位置*/
    left: 8px;
    top:4px;
}
```

### z-index设置层级

z-index:100;

```
1.z-index 值表示谁压着谁，数值大的压盖住数值小的，
2.只有定位了的元素，才能有z-index,也就是说，不管相对定位，绝对定位，固定定位，都可以使用z-index，而浮动元素float不能使用z-index
3.z-index值没有单位，就是一个正整数，默认的z-index值为0如果大家都没有z-index值，或者z-index值一样，那么谁写在HTML后面，谁在上面压着别人，定位了元素，永远压住没有定位的元素。
4.从父现象：父亲怂了，儿子再牛逼也没用
```

**opacity透明度和rgba透明度的区别**

```
opactiy是整个标签的透明度
opacity:0.5这个设置的就是整个标签的透明度为50%，范围为（0.0-1.0）

rgba是单独给某个属性设置透明度
设置方式：color:rgba(255,0,0,0.5)这个表示标签设置为红色，透明度为50%，这个最后的数值就是透明度的系数（0.0-1.0）
```

