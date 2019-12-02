---
title: 第三讲——JavaScript
id: 3
date: 2019-9-16 20:00:00
tags: 前端
comment: true
---

### 简介

#### JavaScript的历史

```
要了解JavaScript，我们首先要回顾一下JavaScript的诞生。 在上个世纪的1995年，当时的网景公司正凭借其Navigator浏览器成为Web时代开 启时最著名的第一代互联网公司。 由于网景公司希望能在静态HTML页面上添加一些动态效果，于是叫Brendan Eich 这哥们在两周之内设计出了JavaScript语言。你没看错，这哥们只用了10天时间。 为什么起名叫JavaScript？原因是当时Java语言非常红火，所以网景公司希望借 Java的名气来推广，但事实上JavaScript除了语法上有点像Java，其他部分基本上 没啥关系。
```

#### 什么是ECMAScript

```
因为网景开发了JavaScript，一年后微软又模仿JavaScript开发了JScript，为了让 JavaScript成为全球标准，几个公司联合ECMA（European Computer Manufacturers Association）组织定制了JavaScript语言的标准，被称为 ECMAScript标准。 所以简单说来就是，ECMAScript是一种语言标准，而JavaScript是网景公司对 ECMAScript标准的一种实现。 那为什么不直接把JavaScript定为标准呢？因为JavaScript是网景的注册商标。 不过大多数时候，我们还是用JavaScript这个词。如果你遇到ECMAScript这个词， 简单把它替换为JavaScript就行了。
```

<!----more---->

### 基础

#### JavaScript组成部分

- 核心：ECMAScript
- DOM(文档对象模型):整合js、css、html
- BOM(浏览器对象模型)：整合js和浏览器

#### 数据类型

**变量**

```js
在这门语言设计之初，并不强制要求使用var声明变量，如果不使用var声明变量，那么这个变量就成了全局变量。使用 var 申明的变量则不是全局变量，它的范围被限制在该变量被申明的函数体 内（函数的概念将稍后讲解），同名变量在不同的函数体内互不冲突。
为了修补JavaScript这一严重设计缺陷，ECMA在后续规范中推出了strict模式，在 strict模式下运行的JavaScript代码，强制通过 var 申明变量，未使用 var 申明变 量就使用的，将导致运行错误。 启用strict模式的方法是在JavaScript代码的第一行写上： 
'use strict';
```

**Number**

不区分整形和浮点型，就只有一种数字类型。

**字符串**

常用的方法：

| 方法                                                         | 说明               |
| ------------------------------------------------------------ | ------------------ |
| .length   #不加括号的是属性                                  | 返回长度           |
| .trim()    #得到一个新值                                     | 移除空白(两边的)   |
| .trimLeft()                                                  | 移除左边的空白     |
| .trimRight()                                                 | 移除右边的空白     |
| .charAt(n) #n类似索引，从0开始，超过最大值返回''空字符串     | 返回第n个字符      |
| .concat(value, ...) #s='hello';s.concat('xx');得到helloxx    | 字符串拼接         |
| .indexOf(substring, start) #这个start是从索引几开始找，没有就返回-1 | 子序列位置         |
| .substring(from, to) #不支持负数，所以一般都不用它，了解一下就行了 | 根据索引获取子序列 |
| .slice(start, end) #var s1='helloworld';s1.slice(0,-5)看结果，就用它 | 切片               |
| .toLowerCase() #全部变小写                                   | 小写               |
| .toUpperCase()  #全部变大写                                  | 大写               |
| .split(delimiter, limit)#分隔,s1.splite(' '),后面还可以加参数s1.split(' '，2),返回切割后的元素个数 | 分割               |

**布尔值**

- null：表示空，一般在需要指定或清空一个变量时才使用。
- undefined（未定义）：区分两者的意义不大，再大多数的情况下我们都是用null，undefined仅仅在判断函数参数是否传递的情况下有用。
- null变量的值是空，undefined表示只声明了变量，但是还没有赋值。

**对象(Object)**

js中的所有的事物都是对象：字符串、数值、函数等。**通过new实例化对象**:var n = new Number(12)，然后n的数据类型就是Object了。对象只是带属性和特殊方法的特殊数据类型。

**数组**

数组对象的作用：使用单独的变量名来储存一系列的值。就和python的列表相似。

```js
两种定义的方法：
方法一：
var a = [1,2,3]
方法二：
var a = new Array([11,22,33,44])
```

常用的方法：

| 方法                                                         | 说明                                       |
| ------------------------------------------------------------ | ------------------------------------------ |
| .length                                                      | 数组的大小                                 |
| .push(ele)                                                   | 尾部追加元素                               |
| .pop()                                                       | 删除末尾的元素（返回值）                   |
| .unshift(ele)                                                | 头部插入元素                               |
| .shift()                                                     | 头部移除元素                               |
| .slice(start, end)                                           | 切片                                       |
| .reverse() #在原数组上改的                                   | 反转                                       |
| .join(seq)#a1.join('+')，seq是连接符                         | 将数组元素连接成字符串                     |
| .concat(val, ...) #连个数组合并,得到一个新数组，原数组不变   | 连接数组                                   |
| .sort()                                                      | 排序                                       |
| .forEach() #讲了函数再说                                     | 将数组的每个元素传递给回调函数             |
| .splice() #参数：1.从哪删(索引), 2.删几个  3.删除位置替换的新元素(可多个元素) | 删除元素，并向数组添加新元素。             |
| .map()  #讲了函数再说                                        | 返回一个数组元素调用函数处理后的值的新数组 |

```js
注意：
sort排序（有坑）：
	对a = [8,466,32,13,1313]进行排序
首先需要定义一个函数：
	升序排序：function sortNumber(a,b){return a - b};
	降序排序：function sortNumber(a,b){return b - a};
排序:a.sort(sortNumber) 
为什么排序的时候需要首先定义一个函数呢？因为我们a中既有数字和字母的时候排序是不准的。


删除 .splice() 
	示例:
		var a = ['aa','bb',33,44];
		单纯删除:a.splice(1,1)
		a -- ["aa", 33, 44]
		
		删除在替换新元素:
		var a = ["aa", 33, 44];
		a.splice(0,2,'hello','world');
		a --  ["hello", "world", 44];
	三个参数介绍:
		参数：1.从哪删(索引), 2.删几个  3.删除位置替换的新元素(可多个元素)
```

#### 内置对象和方法(字典)

**自定义对象**

```js
var a = {
	"name":"pl",
	"age":18
}
取值：两种方法
方法一：
a.age
方法二：
a["age"]
```

解释：对面本质就是键值对的集合（hash结构）,但是只能使用字符作为键。

查找和python里面是一样的。

取得对象里面的数据：**使用for循环**

```js
for xxx of xxx;
实例：
for  (var x of 可迭代对象){操作;}

for xxx in xx h和for xxx of xx的区别：
后者是历史的遗留问题，它遍历的实际上是对象的属性名称，手动给数组添加元素之后，会出现意想不到的结果。
然而，更好的方式是直接使用 iterable 内置的 forEach 方法，它接收一个函 数，每次迭代就自动回调该函数。
var a = ['A', 'B', 'C']; 
a.forEach(function (element, index, array) { 
    // element: 指向当前元素的值 
    // index: 指向当前索引 
    // array: 指向Array对象本身 
    alert(element); }
```

**类型查询**

typeof：它就是一个元运算符，不是一个函数也不是一个语句。

#### 运算符

**算数运算符**

```js
+ - * / % ++ --  i++,是i自加1，i--是i自减1   
i++的这个加1操作优先级低，先执行逻辑，然后再自加1，
++i，这个加1操作优先级高，先自加1，然后再执行代码后面的逻辑
```

**比较运算符**

```js
> >= < <= != == === !==

==（自动转换数据类型）最好不要使用，因为会出现非常诡异的情况
最好使用===（不自动转换数据类型）来进行比较运算
NaN是一个特殊的Number的数据类型，它和所有的其他值都不相等，包括它自己；唯一判断NaN的方法就是通过isNaN()函数
```

**逻辑运算符**

```
&& || !  #and，or，非（取反）!null返回true
```

**赋值运算符**

```
= += -= *= /=  #n += 1其实就是n = n + 1
```

#### 注释

```
单行注释：//
多行注释：/**/
```

###  流程控制

#### if判断

```
简单if-else判断
	var a = 4;
	if (a > 5){
        console.log('a大于5');

    }
    else{
        console.log('小于5');
    };

多条件判断
var a = 10;
if (a > 5){
  console.log("a > 5");
}else if(a < 5) {
  console.log("a < 5");
}else {
  console.log("a = 5");
}
```

#### switch 切换

```
示例:
	var a = 1;
	switch (a++){ //这里day这个参数必须是一个值或者是一个能够得到一个值的算式才行，这个值和后面写的case后面的值逐个比较，满足其中一个就执行case对应的下面的语句，然后break，如果没有加break，还会继续往下判断
        case 1:
            console.log('等于1');
            break;
        case 3:
            console.log('等于3');
            break;
        default:  case都不成立,执行default
            console.log('啥也不是!')	

    }
	
```

#### for循环

```
for (var i=0;i<10;i++) {  //就这么个写法，声明一个变量，变量小于10，变量每次循环自增1，for(;;){console.log(i)}；这种写法就是个死循环，会一直循环，直到你的浏览器崩了，就不工作了，回头可以拿别人的电脑试试~~
  console.log(i);
}
循环数组：
var l2 = ['aa','bb','dd','cc']
方式1
for (var i in l2){
   console.log(i,l2[i]);
}
方式2
for (var i=0;i<l2.length;i++){
　　console.log(i,l2[i])
}

循环自定义对象：
var d = {aa:'xxx',bb:'ss',name:'小明'};
for (var i in d){
    console.log(i,d[i],d.i)  #注意循环自定义对象的时候，打印键对应的值，只能是对象[键]来取值，不能使用对象.键来取值。
}
```

#### while循环

```js
var i = 0;
var a = 10;
while (i < a){
	console.log(i);
	if (i>5){
	continue;
		break;
	}
	i++;
};
```

####  三元运算

```js
var c = a>b ? a:b;  
```

### 函数

**定义函数**

```js
// 普通函数定义
function f1() {
  console.log("Hello world!");
}

// 带参数的函数
function f2(a, b) {
  console.log(arguments);  // 内置的arguments对象
  console.log(arguments.length);
  console.log(a, b);
}

// 带返回值的函数
function sum(a, b){
  return a + b;  //在js中，如果你想返回多个值是不行的，比如return a ，b；只能给你返回最后一个值，如果就想返回多个值，你可以用数组包裹起来 return [a,b]；
}
sum(1, 2);  // 调用函数  sum(1,2,3,4,5)参数给多了，也不会报错，还是执行前两个参数的和，sum(1)，少参数或者没参数也不报错，不过返回值就会是NAN

// 匿名函数方式，多和其他函数配合使用，后面我们就会用到了
var sum = function(a, b){  //在es6中，使用var，可能会飘黄，是因为在es6中，建议你使用let来定义变量，不过不影响你使用
  return a + b;  
}
sum(1, 2);

// 立即执行函数，页面加载到这里，这个函数就直接执行了，不需要被调用执行
(function(a, b){
  return a + b;
})(1, 2);  //python中写可以这么写：ret=(lambda x,y:x+y)(10,20) 然后print(ret)
```

asguments参数：接收全部的参数，即使没有定义参数，也可以接收外面传来的参数

rest参数，它的位置在所有的参数后面，接收传入法人多余的参数

#### 函数的全局变量和局部变量

**局部变量**：在JavaScript函数内部声明的变量（使用 var）是局部变量，所以只能在函数内部访问它（该变量的作用域是函数内部）。只要函数运行完毕，本地变量就会被删除。

**全局变量：**在函数外声明的变量是*全局*变量，网页上的所有脚本和函数都能访问它。

**变量生存周期：**

JavaScript变量的生命期从它们被声明的时间开始。

局部变量会在函数运行以后被删除。

全局变量会在页面关闭后被删除。

变量提升：就是js有一个特点，他会扫描整个函数体的语句，把所有申明的变量提升到函数的顶部。

**全局作用域**:

js会默认一个全局对象window，全局作用域不变量实际上被绑定带window全局作用域的变量实际上被绑定到window的一个属性。我们每次调用的alert()函数就是一个全局变量

**方法**：绑定到对象上的函数称为方法，它和普通的函数没有什么区别，但是在它的内部使用了一个this关键字，在一个方法内部this是一个特殊变量，它始终指向当前对象。

```js
用 var that = this; ，你就可以放心地在方法内部定义其他函数，而不是把所 有语句都堆到一个方法中。
```

**装饰器**：原理都是一样的，为了为封装好的函数增加功能，在调用它的时候，先去执行自己定义的函数。

**闭包**：函数内部函数调用非全局变量的外层函数变量和参数的过程，并返回值得过程叫做闭包。

**生成器（generator）的好处**：把异步回调代码变成同步回调代码，对于ajax请求非常的好用。

```js
首先在函数内部查找变量，找不到则到外层函数查找，逐步找到最外层。

var city = "BeiJing";
function f() {
  var city = "ShangHai";
  function inner(){
    var city = "ShenZhen";
    console.log(city);
  }
  inner();
}
f();  


var city = "BeiJing";
function Bar() {
  console.log(city);
}
function f() {
  var city = "ShangHai";
  return Bar;
}
var ret = f();
ret();

闭包:
	var city = "BeiJing";
    function f(){
        var city = "ShangHai";
        function inner(){
            console.log(city);
        }
        return inner;
    }
    var ret = f();
    ret();
```

### 面向对象

```js
function Person(name){
	this.name = name;
};

var p = new Person('taibai');  

console.log(p.name);

Person.prototype.sum = function(a,b){  //封装方法
	return a+b;
};

p.sum(1,2);
3
```

### Data对象

**创建Data对象**

```js
//方法1：不指定参数
var d1 = new Date(); //获取当前时间
console.log(d1.toLocaleString());  //当前时间日期的字符串表示
//方法2：参数为日期字符串
var d2 = new Date("2004/3/20 11:12");
console.log(d2.toLocaleString());
var d3 = new Date("04/03/20 11:12");  #月/日/年(可以写成04/03/2020)
console.log(d3.toLocaleString());
//方法3：参数为毫秒数，了解一下就行
var d3 = new Date(5000);  
console.log(d3.toLocaleString());
console.log(d3.toUTCString());  
 
//方法4：参数为年月日小时分钟秒毫秒
var d4 = new Date(2004,2,20,11,12,0,300);
console.log(d4.toLocaleString());  //毫秒并不直接显示
```

**Data对象的方法**

```js
var d = new Date(); 
//getDate()                 获取日
//getDay ()                 获取星期 ，数字表示（0-6），周日数字是0
//getMonth ()               获取月（0-11,0表示1月,依次类推）
//getFullYear ()            获取完整年份
//getHours ()               获取小时
//getMinutes ()             获取分钟
//getSeconds ()             获取秒
//getMilliseconds ()        获取毫秒
//getTime ()                返回累计毫秒数(从1970/1/1午夜),时间戳
```

### JSON对象

json更像是javascript的自己，应为开发这个东西的同学非常钟情于js，json规定死了字符集必须使用UTF-8；为了统一解析，json的字符串规定必须使用双引号“”；object的键必须使用双引号"";

```js
序列化方法：stringify()
反序列化方法：parse()

实例：
var str1 = '{"name": "chao", "age": 18}';
var obj1 = {"name": "chao", "age": 18};
// JSON字符串转换成对象
var obj = JSON.parse(str1); 
// 对象转换成JSON字符串
var str = JSON.stringify(obj1);
```

### RegExp对象

```js
//RegExp对象

//创建正则对象方式1
// 参数1 正则表达式(不能有空格)
// 参数2 匹配模式：常用g(全局匹配;找到所有匹配，而不是在第一个匹配后停止)和i(忽略大小写)

// 用户名只能是英文字母、数字和_，并且首字母必须是英文字母。长度最短不能少于6位 最长不能超过12位。

// 创建RegExp对象方式（逗号后面不要加空格），假如匹配用户名是只能字母开头后面是字母加数字加下划线的5到11位的
var reg1 = new RegExp("^[a-zA-Z][a-zA-Z0-9_]{5,11}$"); //注意，写规则的时候，里面千万不能有空格，不然匹配不出来你想要的内容，除非你想要的内容本身就想要空格，比如最后这个{5,11},里面不能有空格

// 匹配响应的字符串
var s1 = "bc123";

//RegExp对象的test方法，测试一个字符串是否符合对应的正则规则，返回值是true或false。
reg1.test(s1);  // true

// 创建方式2，简写的方式
// /填写正则表达式/匹配模式（逗号后面不要加空格）
var reg2 = /^[a-zA-Z][a-zA-Z0-9_]{5,11}$/; 

reg2.test(s1);  // true

注意，此处有坑：如果你直接写一个reg2.test()，test里面啥也不传，直接执行，会返回一个true，用其他的正则规则，可能会返回false，是因为，test里面什么也不传，默认传的是一个undefined，并且给你变成字符串undefined，所以能够匹配undefined的规则，就能返回true，不然返回false


// String对象与正则结合的4个方法
var s2 = "hello world";

s2.match(/o/g);         // ["o", "o"]             查找字符串中 符合正则 的内容 ，/o/g后面这个g的意思是匹配所有的o,
s2.search(/h/g);        // 0                      查找字符串中符合正则表达式的内容位置，返回第一个配到的元素的索引位置，加不加g效果相同
s2.split(/o/g);         // ["hell", " w", "rld"]  按照正则表达式对字符串进行切割，得到一个新值，原数据不变
s2.replace(/o/g, "s");  // "hells wsrld"          对字符串按照正则进行替换

// 关于匹配模式：g和i的简单示例
var s1 = "name:Alex age:18";

s1.replace(/a/, "哈哈哈");      // "n哈哈哈me:Alex age:18"
s1.replace(/a/g, "哈哈哈");     // "n哈哈哈me:Alex 哈哈哈ge:18"      全局匹配
s1.replace(/a/gi, "哈哈哈");    // "n哈哈哈me:哈哈哈lex 哈哈哈ge:18"  不区分大小写


// 注意事项1：
// 如果regExpObject带有全局标志g，test()函数不是从字符串的开头开始查找，而是从属性regExpObject.lastIndex所指定的索引处开始查找。
// 该属性值默认为0，所以第一次仍然是从字符串的开头查找。
// 当找到一个匹配时，test()函数会将regExpObject.lastIndex的值改为字符串中本次匹配内容的最后一个字符的下一个索引位置。
// 当再次执行test()函数时，将会从该索引位置处开始查找，从而找到下一个匹配。
// 因此，当我们使用test()函数执行了一次匹配之后，如果想要重新使用test()函数从头开始查找，则需要手动将regExpObject.lastIndex的值重置为 0。
// 如果test()函数再也找不到可以匹配的文本时，该函数会自动把regExpObject.lastIndex属性重置为 0。

var reg3 = /foo/g;
// 此时 regex.lastIndex=0
reg3.test('foo'); // 返回true
// 此时 regex.lastIndex=3
reg3.test('xxxfoo'); // 还是返回true
// 所以我们在使用test()方法校验一个字符串是否完全匹配时，一定要加上^和$符号，把匹配规则写的确定一些，尽量不用上面这种的写法/xxx/。

// 注意事项2(说出来你可能不信系列)：
// 当我们不加参数调用RegExpObj.test()方法时, 相当于执行RegExpObj.test(undefined)，然后将这个undefined又转为字符串"undefined",去进行匹配了, 并且/undefined/.test()默认返回true。
var reg4 = /^undefined$/;
reg4.test(); // 返回true
reg4.test(undefined); // 返回true
reg4.test("undefined"); // 返回true


RegExp相关
```

### Math对象

解释：类似于python的内置候函数，不需要new来创建对象，直接使用Math

方法：

```js
Math.abs(x)      返回数的绝对值。
exp(x)      返回 e 的指数。
floor(x)    小数部分进行直接舍去。
log(x)      返回数的自然对数（底为e）。
max(x,y)    返回 x 和 y 中的最高值。
min(x,y)    返回 x 和 y 中的最低值。
pow(x,y)    返回 x 的 y 次幂。
random()    返回 0 ~ 1 之间的随机数。
round(x)    把数四舍五入为最接近的整数。
sin(x)      返回数的正弦。
sqrt(x)     返回数的平方根。
tan(x)      返回角的正切。
```

### BOM和DOM

​        到目前为止，我们只学习了简单的js的预发，但是这些简单的语法并没有和浏览器有任何的交互。现在我们学习的BOM和DOM就是和浏览器之间做交互的东西。

**BOM**：（Browser Object Model）是指浏览器对象模型，它使js有能力与浏览器进行对话。

**DOM**（Docoment Object Model）是指文档对象模型，通过它，我们可以访问HTML文档的所有元素。

#### BOM

**window对象**

window对象不但充当全局作用域（所有的js全局对象、函数和变量都是它的成员），而且表示浏览器窗口。全局变量是 window 对象的属性。全局函数是 window 对象的方法。接下来要讲的HTML DOM 的 document 也是 window 对象的属性之一。

window对象有innerWidth和innerHeight属性，可以获取浏览器串口内部的高度和宽度。对应的，还有一个 outerWidth 和 outerHeight 属性，可以获取浏览器窗口的整 个宽高。

**window的子对象**

navigator：浏览器的信息

```
navigator.appName：浏览器名称； 
navigator.appVersion：浏览器版本； 
navigator.language：浏览器设置的语言； 
navigator.platform：操作系统类型； 
navigator.userAgent：客户端绝大部分嘻嘻。
```

screen：屏幕的信息

```
screen.width：屏幕宽度，以像素为单位； 
screen.height：屏幕高度，以像素为单位； 
screen.colorDepth：返回颜色位数，如8、16、24。
```

**location：获取当前页面URL，并把浏览器重定向到新的页面（重点）**

```
要加载一个新页面，可以调用 location.assign() 。
如果要重新加载当前页面， 调用 location.reload() 方法非常方便。
要获得URL各个部分的值，location.href 获取，跳转到指定的页面：localtion.href = "URL"
location.protocol; // 'http' 
location.host; // 'www.example.com' 
location.port; // '8080' 
location.pathname; // '/path/index.html' 
location.search; // '?a=1&b=2' 
location.hash; // 'TOP'
```

document：表示当前页面，document对象就是整个DOM树的根节点

```
用 document 对象提供的 getElementById() 和 getElementsByTagName() 可以按ID获得一个DOM节点和按Tag名称获得一组DOM节点
document 对象还有一个 cookie 属性，可以获取当前页面的Cookie。
```

history：保存浏览器的历史记录

```
JavaScript可以调用 history 对象 的 back() 或 forward () ，相当于用户点击了浏览器的“后退”或“前进”按钮。
```

**弹出框**

JS中创建三种消息框：警告框、确认框、提示框

**警告框**

当警告框出现后，用户需要点击确认按钮才能继续进行操作。

```js
alert("我就是弹出的内容");
```

**确认框**

```js
confirm("我就是确认的内容");
```

**提示框**

```js
prompt("我就是提示的内容");
```

**计时相关（重要）**

解释：通过使用js，我们可以在一段时间间隔之后来执行代码，而不是在函数被调用之后立即执行。我们称这就是计时事件。

setTimeout():一段时间后做一些事情

```js
var t=setTimeout("js语句",毫秒);
第一个参数js语句多数是写一个函数，不然一般的js语句到这里就直接执行了，先用函数封装一下，返回值t其实就是一个id值（浏览器给你自动分配的）
```

clearTimeout():取消set....的设置

```js
clearTimeout(变量);
```

setIntervl()：没隔一段时间做一些事情

```js
setInterval("JS语句",时间间隔);
返回值：一个可以传递给Window.clearInterval()从而取消对code的 周期性执行的值。
```

clearInterval():取消上面的set.....的设置。

```js
clearInterval(变量);
```

**总结**

```js
location对象
    location.href  获取URL
    location.href="URL" // 跳转到指定页面
    location.reload() 重新加载页面,就是刷新一下页面


定时器
    1. setTimeOut()  一段时间之后执行某个内容,执行一次
        示例 
            var a = setTimeout(function f1(){confirm("are you ok?");},3000);
            var a = setTimeout("confirm('xxxx')",3000);  单位毫秒
        清除计时器
            clearTimeout(a);  
    2.setInterval()  每隔一段时间执行一次,重复执行 
        var b = setInterval('confirm("xxxx")',3000);单位毫秒
        清除计时器
            clearInterval(b);
```

#### DOM

DOM是一套都文档的内容进行抽象和概念化的方法。当网页被加载时，浏览器会创建页面的文档对象模型

**HTML DOM树**

![](http://9017499461.linshutu.top/%E5%89%8D%E7%AB%AF%E4%B9%8BDOM%E6%A0%91.png)

始终记住：DOM是一个树形结构。

**查找标签**（用的少，jq解决）

和css是一样的道理，想要操作某个标签需要先找到它。

直接查找

```js
document.getElementById 根据ID获取一个标签
document.getElementsByClassName 根据class属性获取（可以获取多个元素，所以返回的是一个数组）
document.getElementsByTagName 根据标签名获取标签合集
操作：
var divEle = document.getElementById('d1');
var divEle = document.getElementsByClassName('c1');
var divEle = document.getElementsByTagName('div');
```

间接查找

```js
parentElement 父节点标签元素
children 所有子标签
firstElementChild 第一个子标签元素
lastElementChild 最后一个子标签元素
nextElementSibling 下一个兄弟标签元素
previousElementSibling 上一个兄弟标签元素
实例：
var divEle = document.getElementById('d1');
找父级:divEle.parentElement;
找儿子们:divEle.children;
找第一个儿子:divEle.firstElementChild;
找最后一个儿子:divEle.lastElementChild;
找下一个兄弟:divEle.nextElementSibling;
```

**节点操作**

操作一个DOM节点实际上就是那么几个操作：

- 更新：更新该DOM节点的内容，相当于更新了该DOM节点表示的HTML的内容；

- 遍历：遍历该DOM节点下的子节点，以便进行进一步操作； 

- 添加：在该DOM节点下新增一个子节点，相当于动态增加了一个HTML节点； 

- 删除：将该节点从HTML中删除，相当于删掉了该DOM节点的内容以及它包含 的所有子节点。 

创建节点：createElement(标签名)

添加节点：

```
追加一个子节点（作为最后的子节点）somenode.appendChild(newnode)；

把增加的节点放到某个节点的前边。
somenode.insertBefore(newnode,某个节点);
```

删除节点：somenode.removeChild(要删除的节点)

替换节点：somenode.replaceChild(newnode, 某个节点)

**节点属性**

获取文本节点的值

```js
var divEle = document.getEmlmentById("d1");
#获取该标签和内部所有标签的文本内容
divEle.innerText
#获取该标签内的多有内容，包括文本和标签
divEle.innerHTML
```

**操作表单**

```
用JavaScript操作表单和操作DOM是类似的，因为表单本身也是DOM树。 不过表单的输入框、下拉框等可以接收用户输入，所以用JavaScript来操作表单， 可以获得用户输入的内容，或者对一个输入框设置新的内容。
```

```
HTML表单的输入控件主要有以下几种： 文本框，对应的 &lt;input type="text"&gt; ，用于输入文本； 口令框，对应的 &lt;input type="password"&gt; ，用于输入口令； 单选框，对应的 &lt;input type="radio"&gt; ，用于选择一项； 复选框，对应的 &lt;input type="checkbox"&gt; ，用于选择多项； 下拉框，对应的 &lt;select&gt; ，用于选择一项； 隐藏文本，对应的 &lt;input type="hidden"&gt; ，用户不可见，但表单 提交时会把隐藏文本发送到服务器。
```

获取值

```
如果我们获得了一个 &lt;input&gt; 节点的引用，就可以直接调用 value 获得 对应的用户输入值：

实例：
// <input type="text" id="email"> 
var input = document.getElementById('email'); 
input.value; // '用户输入的值
```

类操作

```js
className  获取所有样式类名(字符串)

首先获取标签对象
标签对象.classList; 标签对象所有的class类值

标签对象.classList.remove(cls)  删除指定类
classList.add(cls)  添加类
classList.contains(cls)  存在返回true，否则返回false
classList.toggle(cls)  存在就删除，否则添加，toggle的意思是切换，有了就给你删除，如果没有就给你加一个


示例:
	var divEle = document.getElementById('d1');
	divEle.classList.toggle('cc2');  
	var a = setInterval("divEle.classList.toggle('cc2');",30);

	判断有没有这个类值的方法
		var divEle = document.getElementById('d1');	
		divEle.classList.contains('cc1');
```

**指定CSS操作**

　JS操作CSS属性的规律：

1.对于没有中横线的CSS属性一般直接使用style.属性名即可。如：

```
obj.style.margin
obj.style.width
obj.style.left
obj.style.position
```

2.对含有中横线的CSS属性，将中**横线后面的第一个字母换成大写**即可。如：

```
obj.style.marginTop
obj.style.borderLeftWidth
obj.style.zIndex
obj.style.fontFamily
```

**事件**

常用的几个事件（onfocus、onblur、onclick、onchange）

```js
onclick 当用户点击某个对象时调用的事件句柄。
ondblclick 当用户双击某个对象时调用的事件句柄。
onfocus元素获得焦点。               // 练习：输入框
onblur元素失去焦点。应用场景：用于表单验证,用户离开某个输入框时,代表已经输入完了,我们可以对它进行验证.
onchange 域的内容被改变。应用场景：通常用于表单元素,当元素内容被改变时触发.（select联动）
onkeydown 某个键盘按键被按下。应用场景: 当用户在最后一个输入框按下回车按键时,表单提交.
onkeypress 某个键盘按键被按下并松开。
onkeyup 某个键盘按键被松开。
onload 一张页面或一幅图像完成加载。
onmousedown 鼠标按钮被按下。
onmousemove 鼠标被移动。
onmouseout 鼠标从某元素移开。
onmouseover 鼠标移到某元素之上。
onselect 在文本框中的文本被选中时发生。
onsubmit 确认按钮被点击，使用的对象是form。
```

绑定事件

```js
方式1:
    <script>
        var divEle = document.getElementById('d1');  1.找到标签
        divEle.onclick = function () {       2.给标签绑定事件
            divEle.style.backgroundColor = 'purple';
        }
    </script>
    
    	下面的this表示当前点击的标签
        var divEle = document.getElementById('d1');
        divEle.onclick = function () {
            this.style.backgroundColor = 'purple';
        }
    
方式2
	标签属性写事件名称=某个函数();
	<div class="cc1 xx xx2" id="d1" onclick="f1();"></div>
	
    <script>
    	js里面定义这个函数
        function f1() {
            var divEle = document.getElementById('d1');
            divEle.style.backgroundColor = 'purple';
        }
    </script>
    
    
    获取当前操作标签示例,this标签当前点击的标签
    	<div class="cc1 xx xx2" id="d1" onclick="f1(this);"></div>
        function f1(ths) {
            ths.style.backgroundColor = 'purple';
        }
```

两个实例：

input框动态显示时间

三级联动

AJAX

意思就是JavaScript执行异步网络请求。默认情况下，js在发送ajax请求时，url的域名必须和当前页面完全一致

