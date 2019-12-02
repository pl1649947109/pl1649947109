---
title: 第四讲——jQuery
id: 4
date: 2019-9-17 20:00:00
tags: 前端
comment: true
---

全部内容：找标签+操作标签+事件

找标签：3种方式

- 查找：6(属性)+4(层级)=10个
- 筛选：基本9个+利用[]筛选，3个=12个 +表单(input)：input的属性，大概8-9个+4个属性=13个  共计25个
- 筛选方法：next系列3个+prev系列3个+parent3个+儿子兄弟2个+find1个+filter1个+补充的first()系列5个共计18个

操作标签：

### jQuery

连接：https://www.jquery123.com/category/css/

连接：https://www.w3school.com.cn/jquery/jquery_reference.asp

介绍：

- jQuery是一个轻量级的、兼容多浏览器的JavaScript库。
- jQuery使用户能够更加方便的处理HTML Document、Events、实现动画效果、方便地进行Ajax交互，能够极大地简化js编程。

**jQuery的优势**

- 一款轻量级的JS框架。jQuery核心js文件才几十kb，不会影响页面加载速度。
- 丰富的DOM选择器,jQuery的选择器用起来很方便，比如要找到某个DOM对象的相邻元素，JS可能要写好几行代码，而jQuery一行代码就搞定了，再比如要将一个表格的隔行变色，jQuery也是一行代码搞定。
- 链式表达式。jQuery的链式操作可以把多个操作写在一行代码里，更加简洁。
- 事件、样式、动画支持。jQuery还简化了js操作css的代码，并且代码的可读性也比js要强。
- Ajax操作支持。jQuery简化了AJAX操作，后端只需返回一个JSON格式的字符串就能完成与前端的通信。
- 跨浏览器兼容。jQuery基本兼容了现在主流的浏览器，不用再为浏览器的兼容问题而伤透脑筋。
- 插件扩展开发。jQuery有着丰富的第三方的插件，例如：树形菜单、日期控件、图片切换插件、弹出窗口等等基本前端页面上的组件都有对应插件，并且用jQuery插件做出来的效果很炫，并且可以根据自己需要去改写和封装插件，简单实用。

#### **jQuery的两种引入方式**

- 方式一：直接下载到本地，从本地导入（推荐使用）
- 方式二：使用文件的网络地址导入

#### **jquery和dom对象互相转换**

jQue虽然 `jQuery对象`是包装 `DOM对象`后产生的，但是 `jQuery对象`无法使用 `DOM对象`的任何方法，同理 `DOM对象`也没不能使用 `jQuery`里的方法。

- jQuery对象转成DOM对象：通过一个jQuery对象+[0]索引零，就变成了DOM对象，就可以使用JS的代码方法了，
- DOM对象转换成jQuery对象：$(DOM对象)，通过$符号包裹一下就可以了

```js
$("#i1").html();//DOM对象转换成jQuery对象
$("#i1")[0].innerHTML;//jQuery对象转成DOM对象
```

<!----more---->

#### 基础语法

##### **查找标签**

**基本选择器（同css）**

id选择器：

```js
$("#id")  
#不管找什么标签，用什么选择器，都必须要写$("")，引号里面再写选择器，通过jQuery找到的标签对象就是一个jQuery对象，用原生JS找到的标签对象叫做DOM对象，看我们上面的jQuery对象部分的内容
```

标签选择器：

```js
$("tagName")
```

class选择器：

```js
$(".className")
```

配合使用：

```js
$("div.c1") //找到c1 class类的div标签（通过里面的标签定位外面的标签）
```

所有元素选择器：

```js
$("*")
```

组合选择器：

```js
$("#id,.className,tagName")
```

**层级选择器(同css)**

注意：**x,y可以是任意选择器**

```js
$("x y");// x的所有后代y（子子孙孙）
$("x > y");// x的所有儿子y（儿子）
$("x + y")// 找到所有紧挨在x后面的y
$("x ~ y")// x之后所有的兄弟y
```

**基本筛选器（选择之后进行 过滤）**

```js
:first // 第一个
:last // 最后一个
:eq(index)// 索引等于index的那个元素
:even // 匹配所有索引值为偶数的元素，从 0 开始计数
:odd // 匹配所有索引值为奇数的元素，从 0 开始计数
:gt(index)// 匹配所有大于给定索引值的元素
:lt(index)// 匹配所有小于给定索引值的元素
:not(元素选择器)// 移除所有满足not条件的标签
:has(元素选择器)// 选取所有包含一个或多个标签在其内的标签(指的是从后代元素找)，就是子子孙孙有的就回馈父标签
```

练习

```js
$("div:has(h1)")// 找到所有后代中有h1标签的div标签，意思是首先找到所有div标签，把这些div标签的后代中有h1的div标签筛选出来
$("div:has(.c1)")// 找到所有后代中有c1样式类（类属性class='c1'）的div标签
$("li:not(.c1)")// 找到所有不包含c1样式类的li标签
$("li:not(:has(a))")// 找到所有后代中不含a标签的li标签
```

简单的事件绑定

```js
// 绑定事件的方法
$('#btn').click(function () {
    $('.mode')[0].classList.remove('hide');
    $('.shadow')[0].classList.remove('hide');
	jquery写法:
	$('.mode,.shadow').removeClass('hide');
})
```

**属性选择器（同css）**

```js
[attribute]
[attribute=value]// 属性等于
[attribute!=value]// 属性不等于
```

练习

```js
<input type="text">
<input type="password">
<input type="checkbox">
$("input[type='checkbox']");// 取到checkbox类型的input标签
$("input[type!='text']");// 取到类型不是text的input标签
```

**表单选择器**

说明：多用于form表单里面出现的input标签，当然通过属性选择器肯定也是没有问题的，这样写就是简单一些。

```js
:text
:password
:file
:radio
:checkbox
:submit
:reset
:button
属性：
:enabled  可用的标签
:disabled  不可用的标签
:checked  input选中的标签
:selected  下拉框中被选中的标签
实例：
<select id="s1">
  <option value="beijing">北京市</option>
  <option value="shanghai">上海市</option>
  <option selected value="guangzhou">广州市</option>
  <option value="shenzhen">深圳市</option>
</select>

$(":selected")  // 找到下拉框中所有被选中的option

示例:
	<div>
        用户名: <input type="text">
    </div>

    <div>
        密码: <input type="password" disabled>
    </div>
	
    <div>
        sex:
        <input type="radio" name="sex">男
        <input type="radio" name="sex">女
        <input type="radio" name="sex">不详
    </div>
    <select name="" id="">

        <option value="1">xx1</option>
        <option value="2">xx2</option>
        <option value="3">xx3</option>
        <option value="4">xx4</option>

    </select>
    操作:
    找到可以用的标签 -- $(':enabled')
    找select标签被选中的option标签  $(':selected')
```

##### **筛选器方法**

解释：选择器或者筛选器选择出来的都是对象，而筛选器方法其实就是通过对象来调用一个进一步过滤作用的方法，卸载对象后面加括号，不再是卸载选择器里面的了。

下一个元素：

```js
$("#id").next()
$("#id").nextAll()
$("#id").nextUntil("#i2") #直到找到id为i2的标签就结束查找，不包含它
```

练习：

```python
链式表达：
<ul>
    <li>xx1</li>
    <li>xx2</li>
    <li class="c1">xx3</li>
    <li>xx4</li>
    <li>xx5</li>
    <li class="c2">xx6</li>
    <li>xx7</li>

</ul>

操作		$('li:first').next().css('color','green').next().css('color','red');
```

上一个元素：

```js
$("#id").prev()
$("#id").prevAll()
$("#id").prevUntil("#i2")
```

父亲元素：

```js
$("#id").parent()
$("#id").parents()  // 查找当前元素的所有的父辈元素（爷爷辈、祖先辈都找到）
$("#id").parentsUntil('body') // 查找当前元素的所有的父辈元素，直到遇到匹配的那个元素为止，这里直到body标签，不包含body标签，基本选择器都可以放到这里面使用。
```

儿子和兄弟元素：

```js
$("#id").children();// 儿子们
$("#id").siblings();// 兄弟们，不包含自己，.siblings('#id')，可以在添加选择器进行进一步筛选
```

查找：

解释：搜索所有与指定表达式匹配的元素，这个函数是找出正在处理的元素后代元素的好方法。

```js
$("div").find("p")   //等价于$("div p")
```

筛选：

解释：筛选出与指定表达式匹配的元素集合，这个方法用于缩小匹配的范围。用逗号分隔表达式。

```js
$("div").filter(".c1")  // 等价于 $("div.c1")  从结果集中过滤出有c1样式类的，从所有的div标签中过滤出有class='c1'属性的div，和find不同，find是找div标签的子子孙孙中找到一个符合条件的标签。　　
```

补充：

```js
.first() // 获取匹配的第一个元素
.last() // 获取匹配的最后一个元素
.not() // 从匹配元素的集合中删除与指定表达式匹配的元素
.has() // 保留包含特定后代的元素，去掉那些不含有指定后代的元素。
.eq() // 索引值等于指定值的元素
```

##### 操作标签

样式类操作

```js
'''1·样式类操作：'''
	addClass();// 添加指定的CSS类名。
    removeClass();// 移除指定的CSS类名。
    hasClass();// 判断样式存不存在
    toggleClass();// 切换CSS类名，如果有就移除，如果没有就添加。
```

css操作

```js
'''2·css操作：'''
	$("p").css("color", "red"); //修改单个css样式
	$("p").css({"color":"red","width":400px})//修改多个css样式
```

位置操作

```js
'''3·位置操作：'''
offset()// 获取匹配元素在当前窗口的相对偏移或设置元素位置
position()// 获取匹配元素相对父元素的偏移，不能设置位置
$(window).scrollTop()  //滚轮向下移动的距离
$(window).scrollLeft() //滚轮向右移动的距离

.offset()方法允许我们检索一个元素相对于文档（document）的当前位置。
和 .position()的差别在于： .position()获取相对于它最近的具有相对位置(position:relative或position:absolute)的父级元素的距离，如果找不到这样的元素，则返回相对于浏览器的距离。
```

尺寸

```js
'''4·尺寸：'''
height() //盒子模型content的大小，就是我们设置的标签的高度和宽度
width()
innerHeight() //内容content高度 + 两个padding的高度
innerWidth()
outerHeight() //内容高度 + 两个padding的高度 + 两个border的高度，不包括margin的高度，因为margin不是标签的，是标签和标签之间的距离
outerWidth()
```

文本操作

```js
'''5·文本操作：'''
#html
    html()// 取得第一个匹配元素的html内容，包含标签内容
    html(val)// 设置所有匹配元素的html内容，识别标签，能够表现出标签的效果
#文本值
    text()// 取得所有匹配元素的内容，只有文本内容，没有标签
    text(val)// 设置所有匹配元素的内容，不识别标签，将标签作为文本插入进去
实例：
	取值	
		$('.c1').html();
		$('.c1').text();
	设置值
        $('.c1').text('<a href="">百度</a>');
        $('.c1').html('<a href="">百度</a>');
```

值操作

```js
'''6·值操作：'''
    val()// 取得第一个匹配元素的当前值
    val(val)// 设置所有匹配元素的值
    val([val1, val2])// 设置多选的checkbox、多选select的值
#取值:
	文本输入框:$('#username').val();
	input,type=radio单选框: $('[type="radio"]:checked').val();,首先找到被选中的标签,再进行取值
	input,type=checkbox多选框: 通过val方法不能直接获取多选的值,只能拿到一个,想拿到多个项的值,需要循环取值
		var d = $('[type="checkbox"]:checked');
		for (var i=0;i<d.length;i++){
			console.log(d.eq(i).val());
		}
	单选下拉框select: -- $('#s1').val();
	多选下拉框select: -- $('#s2').val(); -- ['1','2']

#设置值
	文本输入框: -- $('#username').val('xxx');
	input,type=radio单选框: -- $(':radio').val(['1']) 找到所有的radio,然后通过val设置值,达到一个选中的效果.
	给单选或者多选框设置值的时候,只要val方法中的值和标签的value属性对应的值相同时,那么这个标签就会被选中.
	此处有坑:$(':radio').val('1');这样设置值,不带中括号的,意思是将所有的input,type=radio的标签的value属性的值设置为1.
	
	input,type=checkbox多选框: -- $(':checkbox').val(['1','2']);
	
	单选下拉框select: -- $('#s1').val(['3']);
	多选下拉框select: -- $('#s2').val(['1','2']);
	
	统一一个方法:
		选择框都用中括号设置值.
```

属性操作

```js
'''6·属性操作：'''
设置属性: -- $('#d1').attr({'age1':'18','age2':'19'});
		单个设置:$('#d1').attr('age1','18');
查看属性值: -- $('#d1').attr('age1');
删除属性: -- $('#d1').removeAttr('age1'); 括号里面写属性名称

prop和attr方法的区别:
总结一下：
	1.对于标签上有的能看到的属性和自定义属性都用attr
	2.对于返回布尔值的比如checkbox、radio和option的是否被选中或者设置其被选中与取消选中都用prop。
	具有 true 和 false 两个属性的属性，如 checked, selected 或者 disabled 使用prop()，其他的使用 attr()
	checked示例:
		attr():
            查看值,checked 选中--'checked'  没选中--undefined
                $('#nv').attr({'checked':'checked'}); 
            设置值,attr无法完成取消选中
                $('#nv').attr({'checked':'undefined'});
                $('#nv').attr({'checked':undefined});
                
         prop():
         	查看值,checked 选中--true  没选中--false
         		$(':checkbox').prop('checked');
         	设置值:
         		$(':checkbox').prop('checked',true);
         		$(':checkbox').prop('checked',false);
```

文档处理

```js
姿势1:添加到指定元素内部的后面
    $(A).append(B)// 把B追加到A
    $(A).appendTo(B)// 把A追加到B
    
	append示例:
        方式1: 
            创建标签
                var a = document.createElement('a');
                $(a).text('百度');
                $(a).attr('href','http://www.baidu.com');
                $('#d1').append(a);
        方式2:(重点)
            $('#d1').append('<a href="xx">京东</a>');
	appendto示例
		$(a).appendTo('#d1');
		
姿势2:添加到指定元素内部的前面
	$(A).prepend(B)// 把B前置到A
	$(A).prependTo(B)// 把A前置到B
姿势3:添加到指定元素外部的后面
	$(A).after(B)// 把B放到A的后面
	$(A).insertAfter(B)// 把A放到B的后面
姿势4:
	$(A).before(B)// 把B放到A的前面
	$(A).insertBefore(B)// 把A放到B的前面

移除和清空元素
	remove()// 从DOM中删除所有匹配的元素。
	empty()// 删除匹配的元素集合中所有的子节点，包括文本被全部删除，但是匹配的元素还在
	示例:
		$('#d1').remove();
		$('#d1').empty();
		
替换:
	replaceWith()
	replaceAll()
     示例:
     	$('#d1').replaceWith(a);  用a替换前面的标签
     	$(a).replaceAll('#d1');   
```

克隆（复制标签）

```js
示例:
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

	<button class="btn">屠龙宝刀,点击就送!</button>

</body>
<script src="jquery.js"></script>
<script>
    $('.btn').click(function () {
        // var btnEle = $(this).clone(); // 不带参数的克隆不能克隆绑定的事件
        var btnEle = $(this).clone(true); // 参数写个true,就能复制事件
        $(this).after(btnEle);

    })
</script>
</html>
```

#### 事件

##### 常用事件

```js
click(function(){...})
hover(function(){...})
blur(function(){...})
focus(function(){...})
change(function(){...}) //内容发生变化，input，select等
keyup(function(){...})  
mouseover 和 mouseenter的区别是：mouseover事件是如果该标签有子标签，那么移动到该标签或者移动到子标签时会连续触发，mmouseenter事件不管有没有子标签都只触发一次，表示鼠标进入这个对象

示例:
	<!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <style>
            #d1{
                background-color: red;
                height: 200px;
                width: 200px;
            }
            #d2{
                background-color: green;
                height: 200px;
                width: 200px;
            }

            .dd1{
                background-color: yellow;
                height: 40px;
                width: 40px;
            }
            .dd2{
                background-color: pink;
                height: 40px;
                width: 40px;
            }

        </style>
    </head>
    <body>


    <div id="d1">
        <div class="dd1"></div>
    </div>
    <div id="d2">
        <div class="dd2"></div>
    </div>

    用户名:<input type="text" id="username">


    <br>

    <!--<select name="" id="s1">-->
    <!--    <option value="1">上海</option>-->
    <!--    <option value="2">深圳</option>-->
    <!--    <option value="3">贵州</option>-->
    <!--</select>-->

    <input type="text" id="xxx">

    </body>
    <script src="jquery.js"></script>
    <script>
        // 绑定事件的方式1
        // $("#d1").click(function () {
        //     $(this).css('background-color','green');
        // })

        // 方式2
        // $('#d1').on('click',function () {
        //     $(this).css('background-color','green');
        // });
        //
        // // 获取光标触发的事件
        // $('#username').focus(function () {
        //     $(this).css('background-color','green');
        // });
        // // 失去光标触发的事件
        // $('#username').blur(function () {
        //     $(this).css('background-color','white');
        // });
        // // 域内容发生变化触发的事件,一般用于select标签
        // $('#s1').change(function () {
        //      // $('#d1').css('background-color','black');
        //     console.log('xxxxx')
        //
        // });

        // $('#xxx').change(function () {
        //     console.log($(this).val());
        // })

        // 输入内容实时触发的事件,input事件只能on绑定
        // $('#xxx').on('input',function () {
        //     console.log($(this).val());
        // });
        //
        // //绑定多个事件 事件名称空格间隔
        // $('#xxx').on('input blur',function () {
        //     console.log($(this).val());
        // })


        // 鼠标进入触发的事件
        // $('#d1').mouseenter(function () {
        //     $(this).css('background-color','green');
        // });
        // // 鼠标离开触发的事件
        // $('#d1').mouseout(function () {
        //     $(this).css('background-color','red');
        // });

        // hover事件 鼠标进进出出的事件
        // $('#d1').hover(
        //     // 鼠标进入
        //     function () {
        //         $(this).css('background-color','green');
        //     },
        //     // 鼠标离开
        //     function () {
        //         $(this).css('background-color','red');
        //     }
        // );


        $('#d1').mouseenter(function () {
            console.log('xxx');
        });
        $('#d2').mouseover(function () {
            console.log('ooo');
        });
		// 键盘按下
		   // $(window).keydown(function (e) {
            //     console.log(e.keyCode);
            //
            // });
         // 键盘抬起
            $(window).keyup(function (e) {
                console.log(e.keyCode);
            });

    </script>
    </html>
```

##### 事件绑定

```js
方式一:
$('#id').click(funcyion){
	#操作
	$(this).css('background-color','green');
}
方式二：
格式：$('#id').on( events [, selector ],function(){
    #操作
    $(this).css('background-color','green');
})
参数：
	1.events： 事件
	2.selector: 选择器（可选的）
	3.function: 事件处理函数
```

##### 移除事件

格式：`.off( events [, selector ][,function(){}])`

实例：$("li").off("click")；就可以了

##### **阻止事件冒泡**

```js
方式一：
return false; // 常见阻止表单提交等，如果input标签里面的值为空就组织它提交，就可以使用这两种方法
方式二：
e.stopPropagation();
```

注意：

像click、keydown等DOM中定义的事件，我们都可以使用`.on()`方法来绑定事件，但是`hover`这种jQuery中定义的事件就不能用`.on()`方法来绑定了。

想使用事件委托的方式绑定hover事件处理函数，可以参照如下代码分两步绑定事件：

```js
$('ul').on('mouseenter', 'li', function() {//绑定鼠标进入事件
    $(this).addClass('hover');
});
$('ul').on('mouseleave', 'li', function() {//绑定鼠标划出事件
    $(this).removeClass('hover');
});
```

实例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>阻止事件冒泡</title>
</head>
<body>
<div>
    <p>
        <span>点我</span>
    </p>
</div>
<script src="jquery-3.3.1.min.js"></script>
<script>
    //冒泡的意思就是因为html可以嵌套，如果你给儿子标签绑定了点事件或者没有绑定点击事件，父级标签绑定了点击事件，那么你一点击子标签，不管子标签
    有没有绑定事件，都会触发父级标签的点击事件，如果有，会先触发子标签的点击事件，然后触发父级标签的点击事件，不管子标签有没有点击事件，都会一级一级的还往上找点击事件
    //所以我们要阻止这种事件冒泡
    $("span").click(function (e) { //这个参数e(只是个形参，写evt或者event名字的也很多)表示当前事件本身，这个事件也是一个对象
        alert("span");
        //return false；这个也可以阻止
        e.stopPropagation(); 用事件对象的这个方法就能阻止冒泡 （Propagation：传递的意思）
    });

    $("p").click(function () {
        alert("p");
    });
    $("div").click(function () {
        alert("div");
    })
</script>
</body>
</html>
```

##### **事件委托**

事件委托是**通过事件冒泡的原理，利用父标签去捕获子标签的事件，将未来添加进来的某些子标签自动绑定上事件**。

```js
<div id="d1">
    	<button class="btn">屠龙宝刀,点击就送!</button>.
    	
	</div>

    // 事件委托
    $('#d1').on('click','.btn',function () {
          // $(this)是你点击的儿子标签
        var a= $(this).clone();
        $('#d1').append(a);
    });
//中间的参数是个选择器，前面这个$('table')是父级标签选择器，选择的是父级标签，意思就是将子标签（子子孙孙）的点击事件委托给了父级标签
//但是这里注意一点，你console.log(this)；你会发现this还是触发事件的那个子标签，这个记住昂
```

##### **页面载入**

window.onload

```js
原生js的window.onload事件:// onload 等待页面所有内容加载完成之后自动触发的事件
	window.onload = function(){
            $('.c1').click(function () {
                $(this).addClass('c2');
            });
        };
```

jquery页面载入：

```js
$(document).ready(function(){
// 在这里写你的JS代码...
})

#简写
$(function(){
// 你在这里写你的代码
})
```

对比：**jquery页面载入与window.onload的区别**

　　　　1.window.onload()函数有覆盖现象，必须等待着图片资源加载完成之后才能调用

　　　　2.jQuery的这个入口函数没有函数覆盖现象，文档加载完成之后就可以调用（建议使用此函数）

##### jquery的each

语法：

$.each(collection, callback(indexInArray, valueOfElement))：

描述：一个通用的迭代函数，**它可以用来无缝迭代对象和数组**。数组和类似数组的对象通过一个长度属性（如一个函数的参数对象）来迭代数字索引，从0到length - 1。**其他对象通过其属性名进行迭代**。

```js
循环数组:
li =[10,20,30,40]
$.each(li,function(i, v){  
  console.log(i, v);//function里面可以接受两个参数，i是索引，v是每次循环的具体元素。
})

循环标签对象：
var d1 = {'name':'chao','age':18}
 $('d1').each(function(k,v){
     console.log(k,v)
 })
return false;//终止循环
解释：在遍历过程中可以使用return false提前结束each循环。类似于break，而直接使用return；后面什么都不加，不写false，就是跳出本次循环的意思，类似于continue
```

#### **Ajax**

什么是AJAX?

AJAX 是与服务器交换数据的艺术，它在不重载全部页面的情况下，实现了对部分网页的更新。AJAX = 异步 JavaScript 和 XML（Asynchronous JavaScript and XML）。

用JavaScript写AJAX前面已经介绍过了，主要问题就是不同浏览器需要写不同代 码，并且状态和错误处理写起来很麻烦。 用jQuery的相关对象来处理AJAX，不但不需要考虑浏览器问题，代码也能大大简 化。

语法格式： ajax(url, settings)

参数（settings）：

- async：是否异步执行AJAX请求，默认为 true ，千万不要指定为 false 
- method：发送的Method，缺省为 'GET' ，可指定为 'POST' 、 'PUT' 等； 
- contentType：发送POST请求的格式，默认值为 'application/x-www- 
- form-urlencoded; charset=UTF-8' ，也可以指定 为 text/plain 、 application/json ； 
- data：发送的数据，可以是字符串、数组或object。如果是GET请求，data将 被转换成query附加到URL上，如果是POST请求，根据contentType把data序 列化成合适的格式； 
- headers：发送的额外的HTTP头，必须是一个object； 
- dataType：接收的数据格式，可以指定 为 'html' 、 'xml' 、 'json' 、 'text' 等，缺省情况下根据响应 的 Content-Type 猜测。 

**load():**从服务器加载数据，并把返回的数据放入被选中的元素中。

```js
#语法：
$(selector).load(URL,data,callback)
#参数说明
URL：服务端的url
data：规定与请求一同发送的查询字符串键值对集合
callback：load()方法完成后所执行的函数名称
#实例
$("button").click(function(){
  $("#div1").load("demo_test.txt",function(responseTxt,statusTxt,xhr){
    if(statusTxt=="success")
      alert("外部内容加载成功！");
    if(statusTxt=="error")
      alert("Error: "+xhr.status+": "+xhr.statusText);
  });
});
```

**get():**通过HTTP GET请求从指定的资源请求数据。（可能返回缓存数据）

```js
#语法
$.get(URL,callback);
#实例
$("button").click(function(){
  $.get("demo_test.asp",function(data,status){
    alert("Data: " + data + "\nStatus: " + status);
  });
});
```

**post():**通过HTTP POST请求从指定的资源提交要处理的数据。（不会缓存数据，并且常用于连同请求一起发送数据）

```js
#语法
$.post(URL,data,callback);
#实例
$("button").click(function(){
  $.post("demo_test_post.asp",
  {
    name:"Donald Duck",
    city:"Duckburg"
  },
  function(data,status){
    alert("Data: " + data + "\nStatus: " + status);
  });
});
```

getJSON():快速获取GET获取一个JSON对象

```js
var xx = $.getJSON('/path/xxx',{
	name:"pl",
	check:1
}).done(function (data){
    //data已经被解析为JSON对象了
})
```

#### 动画效果

http://www.jeasyui.net/

http://www.jq22.com/

