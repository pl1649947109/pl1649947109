---
title: 第一部分——Python基础篇
id: 1
date: 2019-10-20 20:00:00
tags: 小绿本
toc: true
comment: true
---

## Python基础篇

### 1：为什么学习Python

```
家里有在这个IT圈子里面，也想让我接触这个圈子，然后给我建议学的Python，然后自己通过百度和向有学过Python的同学了解了Python，Python这门语言，入门比较简单，它简单易学，生态圈比较强大，涉及的地方比较多，特别是在人工智能，和数据分析这方面。在未来我觉得是往自动化，人工智能这方面发展的，所以学习了Python
```

<!----more---->

### 2：通过什么途径学习Python

```
刚开始接触Python的时候，到网上里面跟着视频学基础，再后来网上到看技术贴，然后看到有人推荐廖雪峰的Python教程，
练项目到GitHub上面找一些小项目学习。
```

### 3：谈谈对Python和其他语言的区别

```
Python属于解释型语言，当程序运行时，是一行一行的解释，并运行，所以调式代码很方便，开发效率高，还有龟叔给Python定位是任其自由发展、优雅、明确、简单，所以在每个领域都有建树，所有它有着非常强大的第三方库，特点：语法简洁优美，功能强大，标准库与第三方库都非常强大，而且应用领域也非常广可移植性，可扩展性，可嵌入性缺点：　　运行速度慢，- 解释型
    - python/php
- 编译型
    - c/java/c#
        
- Python弱类型
```

 （1）与java相比：在很多方面，Python比Java要简单，比如java中所有变量必须声明才能使用，而Python不需要声明,用少量的代码构建出很多功能;（高效的高级数据结构）

（2）与php相比：python标准包直接提供了工具，并且相对于PHP代码更易于维护;

（3）Python与c相比：

Python 和 C Python这门语言是由C开发而来

　　对于使用：Python的类库齐全并且使用简洁，如果要实现同样的功能，Python 10行代码可以解决，C可能就需要100行甚至更多.
　　对于速度：Python的运行速度相较与C，绝逼是慢

### Python的优势：

1、Python 易于学习;

2、用少量的代码构建出很多功能;（高效的高级数据结构）

3、Python 拥有最成熟的程序包资源库之一;

4、Python完全支持面向对象;

5、Python 是跨平台且开源的。

6、动态类型:

### 4：简述解释型和编译型编程语言

```
解释型：就是边解释边执行（Python，php）
编译型：编译后再执行（c、java、c#）
```

### 5：Python的解释器种类以及相关特点？

CPython
是官方版本的解释器：CPython。是使用C语言开发的，所以叫CPython。在命令行下运行python就是启动CPython解释器。CPython是使用最广的Python解释器。教程的所有代码也都在CPython下执行。

PyPyJythonJython是运行在Java平台上的Python解释器，可以直接把Python代码编译成Java字节码执行。


小结：Python的解释器很多，但使用最广泛的还是CPython。如果要和Java或.Net平台交互，最好的办法不是用Jython或IronPython，而是通过网络调用来交互，确保各程序之间的独立性。

### 6：位和字节的关系

```
1字节 = 8 位位（bit），数据存储是以“字节”（Byte）为单位，数据传输是以大多是以“位”（bit，又名“比特”）为单位，一个位就代表一个0或1（即一个二进制），二进制是构成存储器的最小单位，每8个位（bit，简写为b）组成一个字节（Byte，简写为B），字节是最小一级的信息单位
```

### 7：b、B、KB、MB、GB的关系

b --->位(bit)    

B --->字节      一个字节等于8位

1B = 8 bit

1kb = 1024 B

1 MB = 1024 KB

1 GB = 1024 MB

### 8:PE8规范

```
1、使用4个空格而不是tab键进行缩进。
2、每行长度不能超过79
3、使用空行来间隔函数和类，以及函数内部的大块代码
4、必要时候，在每一行下写注释
5、使用文档注释，写出函数注释
6、在操作符和逗号之后使用空格，但是不要在括号内部使用
7、命名类和函数的时候使用一致的方式，比如使用CamelCase来命名类，
           使用lower_case_with_underscores来命名函数和方法
8、在类中总是使用self来作为默认
 9、尽量不要使用魔法方法
10、默认使用UTF-8，甚至ASCII作为编码方式11、换行可以使用反斜杠，最好使用圆括号。12、不要在一句import中多个库，空格的使用
```

1. 各种右括号前不要加空格。
2. 逗号、冒号、分号前不要加空格。
3. 函数的左括号前不要加空格。如Func(1)
4. 序列的左括号前不要加空格。如list[2]
5. 操作符左右各加一个空格，不要为了对齐增加空格
6. 函数默认参数使用的赋值符左右省略空格
7. 不要将多句语句写在同一行，尽管使用‘；’允许
8. if/for/while语句中，即使执行语句只有一句，也必须另起一行

```
 函数命名使用全部小写的方式，常量命名使用大写，类属性（方法和变量）使用小写类的命名首字母大写
```

### 9：通过代码实现如下转换(进制之间转换）

```
# 二进制转换成十进制-->int
v = "0b1111011"
b = int(v,2)
print(b)  # 123


# 十进制转换成二进制--->bin
v2 = 18
print(bin(int(v2)))
# 0b10010

# 八进制转换成十进制
v3 = "011"
print(int(v3))
# 11

# 十进制转换成八进制：---> oct
v4 = 30
print(oct(int(v4)))
# 0o36

# 十六进制转换成十进制：
v5 = "0x12"
print(int(v5,16))
# 18

# 十进制转换成十六进制：---> hex
v6 = 87
print(hex(int(v6)))
# 0x57
```

### 10:请编写一个函数实现将IP地址转换成一个整数

```
请编写一个函数实现将IP地址转换成一个整数。
如 10.3.9.12 转换规则为：
        10            00001010
          3            00000011 
         9            00001001
         12            00001100 
再将以上二进制拼接起来计算十进制结果：00001010 00000011 00001001 00001100 = ？


def v1(addr):
    # 取每个数
    id = [int(x) for x in addr.split(".")]
    print(id)
    return sum(id[i] << [24, 16, 8, 0][i] for i in range(4))

print(v1("127.0.0.1"))

# [127, 0, 0, 1]
# 2130706433
```

\------------------------------------------------

### 11、python递归的最大层数？

理论上是1000,但是在实际的运行中多是998、997

### 12：求结果(and or or)

```
1. 求结果：1 or 3
print(1 or 3)  # 1

2. 求结果：1 and 3
print(1 and 3)  # 3

3. 求结果：0 and 2 and 1
print(0 and 2 and 1)  # 0

4. 求结果：0 and 2 or 1
print(0 and 2 or 1)  # 1

5. 求结果：0 and 2 or 1 or 4
print(0 and 2 or 1 or 4)  # 1

6. 求结果：0 or Flase and 1
print(0 or False and 1)  # Flase

总结：
　　# x or y 如果 x为真，则值为x，   否则为y
　　# x and y 如果 x 为真，则值为 y，否则为 x
```

### 运算符

\1. 求结果：2 & 5

```
print(2 & 5)  # 10 & 101 => 000 => 0
```

\2. 求结果：2 ^ 5

```
print(2 ^ 5)  # 10 ^ 101 => 111 => 1*2**0+1*2**1+1*2**2=1+2+4=7
```

### 13 ：ascii、unicode、utf-8、gbk 区别

```
python2内容进行编码（默认ascii）,而python3对内容进行编码的默认为utf-8。
ascii   最多只能用8位来表示（一个字节），即：2**8 = 256，所以，ASCII码最多只能表示 256 个符号。
unicode  万国码，任何一个字符==两个字节
utf-8     万国码的升级版  一个中文字符==三个字节   英文是一个字节  欧洲的是 2个字节
gbk       国内版本  一个中文字符==2个字节   英文是一个字节
gbk 转 utf-8  需通过媒介 unicode
```

### 14:字节码和机器码的区别

机器码，学名机器语言指令，有时也被称为原生码，是电脑的CPU可直接解读的数据。

字节码是一种中间状态（中间码）的二进制代码（文件）。需要直译器转译后才能成为机器码。

```
什么是机器码

机器码(machine code)，学名机器语言指令，有时也被称为原生码（Native Code），是电脑的CPU可直接解读的数据。
通常意义上来理解的话，机器码就是计算机可以直接执行，并且执行速度最快的代码。
总结：机器码是电脑CPU直接读取运行的机器指令，运行速度最快，但是非常晦涩难懂，也比较难编写

什么是字节码
字节码（Bytecode）是一种包含执行程序、由一序列 op 代码/数据对 组成的二进制文件。字节码是一种中间码，它比机器码更抽象，需要直译器转译后才能成为机器码的中间代码。

总结：字节码是一种中间状态（中间码）的二进制代码（文件）。需要直译器转译后才能成为机器码。
```

```
#is  比较的是内存地址
#== 比较的是值
# int     具有范围：-5---256
#对于int 小数据池
 范围：-5----256 创建的相间的数字，都指向同一个内存地址

#对于字符串 （面试）
1、小数据池 如果有空格，那指向两个内存地址，
2、长度不能超过 20
3、不能用特殊字符

i = 'a'*20
j = 'a'*20
print(i is j)   # True

i = "a"*21
j = "a"*21
print(i is j)   # False

关于编码所占字节
unicode： 所有字符（无论英文、中文等）   1个字符：2个字节
gbk：一个字符，英文1个字节，中文两个字节
utf-8：英文1个字节、 欧洲：2个字节， 亚洲：3个字节


在utf-8中，一个中文字符占用3个字节
在gbk中一个汉字占用2个字节
黎诗 = utf-8(6字节)=48
黎诗 = gbk(4字节)=32

字节和位的关系。
　　#一个字节(byte) = 8 位(bit)
　　# 位为最小的单位

简述变量命名规范
　　#1、以字母，数字，下划线任由结合
　　#2、不能以命名太长，不使用拼音，中文
　　#3、不能以数字开头
　　#4、不能用关键词
```

### 15:三元运算写法和应用场景？

```
应用场景：简化if语句# 关于三元运算
# 结果+　if  + 条件  + else + 结果
result='gt' if 1>3 else 'lt'
print(result)       # lt
# 理解：如果条件为真，把if前面的值赋值给变量，否则把else后面的值赋值给变量。


lambda 表达式
temp = lambda x,y:x+y
print(temp(4,10))   # 14

可替代：
def foo(x,y):
    return x+y
print(foo(4,10))    # 14
```

### 16:Python3和Python2的区别？

```
1：打印时，py2需要可以不需要加括号，py3 需要
python 2 ：print ('lili')   ,   print 'lili'
python 3 : print ('lili')   
python3 必须加括号

exec语句被python3废弃，统一使用exec函数

2：内涵
Python2：1，臃肿，源码的重复量很多。
          　  2，语法不清晰，掺杂着C，php，Java，的一些陋习。
Python3：几乎是重构后的源码，规范，清晰，优美。

3、输出中文的区别
python2：要输出中文 需加 # -*- encoding:utf-8 -*-
Python3 ： 直接搞

4：input不同
python2 ：raw_input
python3 ：input 统一使用input函数

5：指定字节
python2在编译安装时，可以通过参数-----enable-unicode=ucs2 或-----enable-unicode=ucs4分别用于指定使用2个字节、4个字节表示一个unicode；
python3无法进行选择，默认使用 ucs4
查看当前python中表示unicode字符串时占用的空间：

impor sys
print（sys.maxunicode）
#如果值是65535，则表示使用usc2标准，即：2个字节表示
#如果值是1114111，则表示使用usc4标准，即：4个字节表示

6：
py2：xrange
　　　　range
py3：range  统一使用range，Python3中range的机制也进行修改并提高了大数据集生成效率

7：在包的知识点里
包：一群模块文件的集合 + __init__
区别：py2 ： 必须有__init__
　　　py3：不是必须的了

8：不相等操作符"<>"被Python3废弃，统一使用"!="

9：long整数类型被Python3废弃，统一使用int

10：迭代器iterator的next()函数被Python3废弃，统一使用next(iterator)

11：异常StandardError 被Python3废弃，统一使用Exception

12：字典变量的has_key函数被Python废弃，统一使用in关键词

13：file函数被Python3废弃，统一使用open来处理文件，可以通过io.IOBase检查文件类型
```

### 17：用一行代码实现数值交换

a = 1 
b = 2

```
a, b = b, a
```

### 18：Python3和Python2中int和long区别

在python3里，只有一种整数类型int,大多数情况下，和python２中的长整型类似。

### 19：xrange和range的区别

都在循环时使用，xrange内存性能更好，xrange用法与range完全相同，range一个生成list对象，xrange是生成器

要生成很大的数字序列的时候，用xrange会比range性能优很多，因为不需要一上来就开辟一块很大的内存空间。

在python2中：

range([start,] stop[, step])，根据start与stop指定的范围以及step设定的步长，生成一个序列

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)例子

xrange用法与range完全相同，所不同的是生成的不是一个数组，而是一个生成器。

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)例子

由上面的示例可以知道：要生成很大的数字序列的时候，用xrange会比range性能优很多，因为不需要一上来就开辟一块很大的内存空间，这两个基本上都是在循环的时候用。

在 Python 3 中，range() 是像 xrange() 那样实现，xrange()被抛弃。

### 20：文件操作时：xreadlines和readlines的区别？

readlines     返回一个列表

xreadlines   返回一个生成器

### 21： 列列举布尔值为False的常见值？

```
0，“”，{}，[],（），set（）
0 Flask 负数 不成立的表达式  None 等
```

### 22. 字符串、列表、元组、字典每个常用的5个方法？

```
字符串：字符串用单引号(')或双引号(")括起来，不可变
1，find通过元素找索引，可切片，找不到返回-1
2，index，找不到报错。
3，split 由字符串分割成列表，默认按空格。
4，captalize 首字母大写，其他字母小写。
5，upper 全大写。
6，lower 全小写。
7，title，每个单词的首字母大写。
8，startswith 判断以什么为开头，可以切片，整体概念。
9，endswith 判断以什么为结尾，可以切片，整体概念。
10，format格式化输出#format的三种玩法 格式化输出res='{} {} {}'.format('egon',18,'male')  ==>  egon 18 maleres='{1} {0} {1}'.format('egon',18,'male')  ==> 18 egon 18res='{name} {age} {sex}'.format(sex='male',name='egon',age=18)
11,strip 默认去掉两侧空格，有条件， 12，lstrip,rstrip 14,center 居中，默认空格。 15，count查找元素的个数，可以切片，若没有返回0 16，expandtabs 将一个tab键变成8个空格，如果tab前面的字符长度不足8个，则补全8个， 17，replace（old，new,次数） 18，isdigit 字符串由字母或数字组成 isalpha, 字符串只由字母组成 isalnum 字符串只由数字组成 19,swapcase 大小写翻转 20，for i in 可迭代对象。 字典：1无序（不能索引）2：数据关联性强3:键值对，键值对。唯一一个映射数据类型。
#字典的键必须是可哈希的   不可变类型。
在同一个字典中，键(key)必须是唯一的。
列表是有序的对象集合，字典是无序的对象集合。两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取
key： 输出所有的键
clear：清空                       
dic：删除的键如果没有则报错
pop：键值对删，有返回，没有原来的键会报错（自行设置返回键就不会报错）
popitem：随机删键值对
del：删除的键如果没有则报错
改 update
查  用get时。不会报错# 没有可以返回设定的返回值
    注意：
1、字典是一种映射类型，它的元素是键值对。
2、字典的关键字必须为不可变类型，且不能重复。
3、创建空字典使用 { }。

列表：索引，切片，加，乘，检查成员。
增加：有三种，
append：在后面添加。
Insert按照索引添加，
extend：迭代着添加。
list.extend(seq) - 在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表）
pop 删除   (pop 有返回值)
remove 可以按照元素去删
clear  清空列表
del 1、可以按照索引去删除 2、切片 3、步长（隔着删）
改  1、索引  2、切片：先删除，再迭代着添加
list.count(obj) - 统计某个元素在列表中出现的次数
list.index(obj) - 从列表中找出某个值第一个匹配项的索引位置
list.reverse() - 反向列表中元素
list.sort([func]) - 对原列表进行排序
注意：
1、List写在方括号之间，元素用逗号隔开。
2、和字符串一样，list可以被索引和切片。
3、List可以使用+操作符进行拼接。
4、List中的元素是可以改变的。

元组：（）元组的元素不能修改
1、cmp(tuple1, tuple2)：比较两个元组元素。----->python2中的方法，3中删除了
2、len(tuple)：计算元组元素个数。
3、max(tuple)：返回元组中元素最大值。
4、min(tuple)：返回元组中元素最小值。
5、tuple(seq)：将列表转换为元组。
注意
1、与字符串一样，元组的元素不能修改。
2、元组也可以被索引和切片，方法一样。
3、注意构造包含0或1个元素的元组的特殊语法规则。
4、元组也可以使用+操作符进行拼接。


Set（集合）：集合（set）是一个无序不重复元素的序列。
可以使用大括号 { } 或者 set() 函数创建集合，注意：创建一个空集合必须用 set() 而不是 { }，因为 { } 是用来创建一个空字典。
```

### 23、 lambda表达式格式以及应用场景？

```
匿名函数：为了解决那些功能很简单的需求而设计的一句话函数
函数名 = lambda 参数 ：返回值

#参数可以有多个，用逗号隔开
#匿名函数不管逻辑多复杂，只能写一行，且逻辑执行结束后的内容就是返回值
#返回值和正常的函数一样可以是任意数据类型

lambda 表达式
temp = lambda x,y:x+y
print(temp(4,10))   # 14

可替代：
def foo(x,y):
    return x+y
print(foo(4,10))    # 14
```

### 24. pass的作用

pass是空语句，是为了保持程序结构的完整性。pass 不做任何事情，一般用做占位语句。

### 25. *arg和**kwarg作用

```
 *args代表位置参数，它会接收任意多个参数并把这些参数作为元祖传递给函数。**kwargs代表的关键字参数，返回的是字典，位置参数一定要放在关键字前面
```

### 26. is和==的区别

```
a = 'lishi'
str1 = "li"
str2 = "shi"
str3 = str1 + str2
print("a == str3",a == str3)
print("a is str3",a is str3)
print("id(a)",id(a))
print("id(str3)",id(str3))
# a == str3 True    ==  ---> 只需要内容相等
# a is str3 False   is  ---> 只需要内存地址相等
# id(a) 38565848
# id(str3) 39110280
```

```
is 比较的是两个实例对象是不是完全相同，它们是不是同一个对象，占用的内存地址是否相同。

== 比较的是两个对象的内容是否相等，即内存地址可以不一样，内容一样就可以了。默认会调用对象的 __eq__()方法。
```

### 27：谈谈Python的深浅拷贝？以及实现方法和应用场景。

浅拷贝只是增加了一个指针指向一个存在的地址，

而深拷贝是增加一个指针并且开辟了新的内存，这个增加的指针指向这个新的内存，
采用浅拷贝的情况，释放内存，会释放同一内存，深拷贝就不会出现释放同一内存的错误

一层的情况：

```
import copy
 
# 浅拷贝
li1 = [1, 2, 3]
li2 = li1.copy()
li1.append(4)
print(li1, li2)  # [1, 2, 3, 4] [1, 2, 3]
 
# 深拷贝
li1 = [1, 2, 3]
li2 = copy.deepcopy(li1)
li1.append(4)
print(li1, li2)  # [1, 2, 3, 4] [1, 2, 3]
```

多层的情况：

```
import copy
 
# 浅拷贝 指向共有的地址
li1 = [1, 2, 3,[4,5],6]
li2 = li1.copy()
li1[3].append(7)
print(li1, li2)  # [1, 2, 3, [4, 5, 7], 6] [1, 2, 3, [4, 5, 7], 6]
 
# 深拷贝 重指向
li1 = [1, 2, 3,[4,5],6]
li2 = copy.deepcopy(li1)
li1[3].append(7)
print(li1, li2)  # [1, 2, 3, [4, 5, 7], 6] [1, 2, 3, [4, 5], 6]
```

deepcopy如果你来设计，如何实现？

```python
自己实现一个deepcopy，可以考虑用递归，实现对list的deepcopy,这里只考虑的list，但是实际还有tuple，dict，set

def deepcopy(obj):
    mylist = []

    for item in obj:
        #判断item是不是list，对于是list这种可变的数据类型就新开辟一片空间
        if isinstance(item, list):
            i = deepcopy(item)
            mylist.append(i)
        else:
            mylist.append(item)
    return mylist
```

### 28. Python垃圾回收机制？

引用计数

标记清除

分代回收

### 29. Python的可变类型和不可变类型？

可变数据类型：列表、字典、可变集合

不可变数据类型：数字、字符串、元组、不可变集合

### 30、求结果

```
def a():
    return [lambda x:i*x for i in range(4)]
b=a()   #返回个列表函数
# b[2](1)

print(b[1](1))
# print(type(b),b)
print([m(1) for m in a()])
print([i*i for i in [1,2,3]])
[3, 3, 3, 3]
[1, 4, 9]


'''
def multipliers():
    return [lambda x:i*x for i in range(4)]
print([m(2) for m in multipliers()])
#解释：
　　函数返回值为一个列表表达式，经过4次循环结果为包含四个lambda函数的列表，
由于函数未被调用，循环中的i值未被写入函数，经过多次替代，循环结束后i值为3，
故结果为：6,6,6,6



func=lambda x:x+1
print(func(1))
#2
print(func(2))
#3

#以上lambda等同于以下函数
def func(x):
    return(x+1)
'''
```

```
请修改multipliers的定义来产生期望的结果（0,2,4,6）。
def multipliers():
    return （lambda x:i*x for i in range(4)）         #返回一个生成器表达式
print([m(2) for m in multipliers()])
```

```
-面试题2：现有两个元组(('a'),('b')),(('c'),('d'))，请使用python中匿名函数生成列表[{'a':'c'},{'b':'d'}]

#匿名函数形式：
l1=(('a'),('b'))
l2=(('c'),('d'))
ret=map(lambda n:{n[0]:n[1]},zip(l1,l2))
print(list(ret))
#列表表达式形式：
l1=(('a'),('b'))
l2=(('c'),('d'))
print([{n[0]:n[1]} for n in zip(l1,l2)])
```

### 31、求结果

```
v = dict.fromkeys(['k1', 'k2'], [])
v['k1'].append(666)
print(v)
v['k1'] = 777
print(v)

结果：
{'k1': [666], 'k2': [666]}
{'k1': 777, 'k2': [666]}

解释：
Python 字典(Dictionary) fromkeys() 函数用于创建一个新字典，以序列seq中元素做字典的键，value为字典所有键对应的初始值，默认为None。

v1 = dict.fromkeys(['k1', 'k2'])
print(v1)  # {'k1': None, 'k2': None}
 
v2 = dict.fromkeys(['k1', 'k2'], [])
print(v2)  # {'k1': [], 'k2': []}
```

### 32、列举常见的内置函数

### abs（）

```
返回数字的绝对值
```

### map

```
根据函数对指定序列做映射map()函数接收两个参数，一个是函数，一个是可迭代对象，map将传入的函数依次作用到序列的每个元素，并把结果作为新的list返回。返回值：　　Python2  返回列表　　Python3  返回迭代器例子1：
def mul(x):
    return x*x
n=[1,2,3,4,5]
res=list(map(mul,n))
print(res)  #[1, 4, 9, 16, 25]例子2：abs()  返回数字的绝对值ret = map(abs,[-1,-5,6,-7])print(list(ret))# [1, 5, 6, 7]
```

### filter

```
filter()函数接收一个函数 f(函数)和一个list（可迭代对象），这个函数 f的作用是对每个元素进行判断，返回 True或 False，
filter()根据判断结果自动过滤掉不符合条件的元素，返回由符合条件元素组成的新list。
def is_odd(x):
    return x % 2 == 1

v=list(filter(is_odd, [1, 4, 6, 7, 9, 12, 17]))
print(v)  #[1, 7, 9, 17]
```

### **map与filter总结**

```
# filter 与 map 总结
# 参数: 都是一个函数名 + 可迭代对象
# 返回值: 都是返回可迭代对象
# 区别:
# filter 是做筛选的，结果还是原来就在可迭代对象中的项
# map 是对可迭代对象中每一项做操作的，结果不一定是原来就在可迭代对象中的项
```

### **isinstance\type**

```
isinstance() 函数来判断一个对象是否是一个已知的类型，类似 type()。
isinstance() 与 type() 区别：
type() 不会认为子类是一种父类类型，不考虑继承关系。
isinstance() 会认为子类是一种父类类型，考虑继承关系。
如果要判断两个类型是否相同推荐使用 isinstance()。
# 例一a = 2print(isinstance(a,int))   # Trueprint(isinstance(a,str))   # False

# type() 与 isinstance() 区别
class A:
    pass

class B(A):
    pass

print("isinstance",isinstance(A(),A))   # isinstance True
print("type",type(A())  == A)    # type True

print('isinstance',isinstance(B(),A) )   # isinstance True
print('type',type(B()) == A)     #  type False
```

### zip 拉链函数

```
# zip 拉链函数，
# 将对象中对应的元素打包成一个个元组，
# 然后返回由这些元组组成的列表迭代器。
# 如果各个迭代器的元素个数不一致，则返回列表长度与最短的对象相同。
print(list(zip([0,1,3],[5,6,7],['a','b'])))
# [(0, 5, 'a'), (1, 6, 'b')]
```

```
zip() 函数用于将可迭代的对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表。
>>>a = [1,2,3]
>>> b = [4,5,6]
>>> c = [4,5,6,7,8]
>>> zipped = zip(a,b)     # 打包为元组的列表
[(1, 4), (2, 5), (3, 6)]
>>> zip(a,c)              # 元素个数与最短的列表一致
[(1, 4), (2, 5), (3, 6)]
>>> zip(*zipped)          # 与 zip 相反，可理解为解压，返回二维矩阵式
[(1, 2, 3), (4, 5, 6)]
```

**reduce**

```
'''
reduce()  函数
reduce() 函数会对参数序列中元素进行累积
函数将一个数据集合(链表、元组等)中的所有数据进行下列操作
'''

注意：
Python3已经将reduce() 函数从全局名字空间里移除了，它现在被放置在 fucntools 模块里，如果想要使用它，则需要通过引入 functools 模块来调用 reduce() 函数：

from functools import reduce
def add(x,y):
    return x + y

print(reduce(add,[1,2,3,4,5]))
#  15

print(reduce(lambda x, y: x+y, [1,2,3,4,5]))  # 15

print(reduce(add,range(1,101)))
#  5050
```

### 33. [filter、map、reduce的作用？](https://www.cnblogs.com/Xrinehart/p/3506467.html)

### 内置函数：map、reduce、filter的用法和区别

**map**:根据函数对指定序列做映射

```
map
参数
接收两个参数：一个是函数，一个是序列（可迭代对象）
返回值
Python2 返回列表
Python3 返回迭代器

# 例子：
# abs() 函数返回数字的绝对值
# 新的内容的个数等于原内容的个数
# ret = map(abs,[-1,-5,6,-7])
# print(list(ret))
# [1, 5, 6, 7]
```

**filter:**过滤函数 新的内容少于等于原内容的时候。才能使用filter

```
filter() 函数用于过滤序列，过滤不符合条件的元素，返回由符合条件元素组成的心列表

参数：
function  函数
iterable  可迭代对象
返回值:
返回列表

# 筛选大于10的数
def is_odd(x):
    if x>10:
        return True

ret = filter(is_odd,[1,4,5,7,8,9,76])  # 为迭代器
print(list(ret))
# [76]
```

**reduce**:对于序列内所有元素进行累计操作

```
'''
reduce()  函数
reduce() 函数会对参数序列中元素进行累积
函数将一个数据集合(链表、元组等)中的所有数据进行下列操作
'''

from functools import reduce
def add(x,y):
    return x + y

print(reduce(add,[1,2,3,4,5]))
#  15

print(reduce(lambda x, y: x+y, [1,2,3,4,5]))  # 15

print(reduce(add,range(1,101)))
#  5050
```

### 34、 一行代码实现9*9乘法表

```
print('\n'.join([' '.join(['%s*%s=%-2s' % (j, i, i * j) for j in range(1, i + 1)]) for i in range(1, 10)]))
```

### 35. 如何安装第三方模块？以及用过哪些第三方模块？

```
1：pip包管理器
2：源码下载
    -下载
    -解压
-python setup.py build
-python setup.py install
```

用过的第三方模块：requests,pymysql,DbUtils,SQLAlchemy等

### 36、 常用模块都有那些？

re模块，os模块，json模块，time模块，

爬虫里面的requests/beautifulsoup4（bs4）

### 37. re的match和search区别？

re.match 尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配成功的话，match()就返回none。

re.search 扫描整个字符串并返回第一个成功的匹配。

### 38. 什么是正则的贪婪匹配？

匹配一个字符串没有节制，能匹配多少就去匹配多少，知道没有匹配的为止

### 39. 求结果：

a. [ i % 2 for i in range(10) ]

```
print([ i % 2 for i in range(10) ])  # [0, 1, 0, 1, 0, 1, 0, 1, 0, 1]
print([ i  for i in range(10) ])     # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print([ 10 % 2])   # [0]
# %是个运算符。
```

b. ( i % 2 for i in range(10) )

```
print(( i % 2 for i in range(10) ))
#  <generator object <genexpr> at 0x00000000020CEEB8> 生成器
# 在Python中，有一种自定义迭代器的方式，称为生成器（Generator）。
# 定义生成器的两种方式：
# 1.创建一个generator，只要把一个列表生成式的[]改成()，就创建了一个generator：
# generator保存的是算法，每次调用next()，就计算出下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出StopIteration的错误。
# 2.定义generator的另一种方法。如果一个函数定义中包含yield关键字，那么这个函数就不再是一个普通函数，而是一个generator
```

### 40. 求结果：

a. 1 or 2
b. 1 and 2
c. 1 < (2==2)
d. 1 < 2 == 2

```
>>> 1 or 2
1
>>> 1 and 2
2
>>> 1 < (2==2)
False
>>> 1 < 2 == 2
True
```

###  41、def func(a,b=[]) 这种写法有什什么坑？

```
def func(a,b = []):
    b.append(1)
    print(a,b)

func(a=2)
func(2)
func(2)


'''
    2 [1]
    2 [1, 1]
    2 [1, 1, 1]
    函数的默认参数是一个list 当第一次执行的时候实例化了一个list 
    第二次执行还是用第一次执行的时候实例化的地址存储 
    所以三次执行的结果就是 [1, 1, 1] 想每次执行只输出[1] ，默认参数应该设置为None
'''
```

### 42、如何实现 “1,2,3” 变成 [‘1’,’2’,’3’]

```
list("1,2,3".split(','))
```

### 43. 如何实现[‘1’,’2’,’3’]变成[1,2,3]

```
[int(x) for x in ['1','2','3']]


python 里如何把['1','2','3'] 变成[1,2,3]

a = ['1','2','3']
b = [int(i) for i in a]
print(b)
# [1, 2, 3]
```

### 44. a = [1,2,3] 和 b = [(1),(2),(3) ] 以及 b = [(1,),(2,),(3,) ] 的区别？

 补充：

```

```

```
a=[1,2,3,4,5]，b=a和b=a[:]，有区别么？


a = [1,2,3,4,5]
b = a
b1 = a[:]
print(b)    #  [1, 2, 3, 4, 5]
# print(b)   #  [1, 2, 3, 4, 5]

b.append(6)
print("a",a)  # a [1, 2, 3, 4, 5, 6]
print("b",b)  # b [1, 2, 3, 4, 5, 6]  传递引用
print("b1",b1) # b1 [1, 2, 3, 4, 5]   拷贝
```

```
# 一个列表A=[2，3，4]，Python如何将其转换成B=[(2,3),(3,4),(4,2)]？
# B = zip(A, A[1:]+A[:1])
```

### 45. 如何用一行代码生成[1,4,9,16,25,36,49,64,81,100]

```
[i*i for i in range(1,11)]
```

### 46. 一行代码实现删除列表中重复的值

```
list(set([1, 2, 3, 4, 45, 1, 2, 343, 2, 2]))
```

### 47. 如何在函数中设置一个全局变量

python中的global语句是被用来声明全局变量的。

```
x = 2
def func():
    global x
    x = 1
    return x
func()
print(x)  # 1
```

### 48. logging模块的作用？以及应用场景？

```
logging 
模块定义的函数和类为应用程序和库的开发实现了一个灵活的事件日志系统

作用：可以了解程序运行情况，是否正常　　　　在程序的出现故障快速定位出错地方及故障分析
```

### 49. 请用代码简答实现stack

- Stack() 创建一个新的空栈
- push(item) 添加一个新的元素item到栈顶
- pop() 弹出栈顶元素
- peek() 返回栈顶元素
- is_empty() 判断栈是否为空
- size() 返回栈的元素个数

```
# 实现一个栈stack,后进先出

'''
class Stack:
    def __init__(self):
        self.items = []

    def is_empty(self):
        # 判断是否为空
        return self.items == []

    def push(self,item):
        # 加入元素
        self.items.append(item)

    def pop(self):
        # 弹出元素
        return self.items.pop()

    def peek(self):
        # 返回栈顶元素
        return self.items[len(self.items)-1]

    def size(self):
        # 返回栈的大小
        return len(self.items)

if __name__ == "__main__":
    stack = Stack()
    stack.push("H")
    stack.push("E")
    stack.push("L")
    print(stack.size())  # 3
    print(stack.peek())  # L 
    print(stack.pop())   # L
    print(stack.pop())   # E
    print(stack.pop())   # H
'''
```

### 50. 常用字符串格式化哪几种？

1.占位符%

%d 表示那个位置是整数；%f 表示浮点数；%s 表示字符串。

```
print('Hello,%s' % 'Python')
print('Hello,%d%s%.2f' % (666, 'Python', 9.99)) # 打印：Hello,666Python10.00
```

2.format

```
print('{k} is {v}'.format(k='python', v='easy'))  # 通过关键字
print('{0} is {1}'.format('python', 'easy'))      # 通过关键字
```

### 51. 简述 生成器、迭代器、可迭代对象 以及应用场景？

### 迭代器

含有__iter__和__next__方法 (包含__next__方法的可迭代对象就是迭代器)

### 生成器

包括含有yield这个关键字，生成器也是迭代器，调动next把函数变成迭代器。

```
应用场景：
range/xrange
    - py2： range(1000000)  ,会立即创建，xrange(1000000)生成器
    - py3：range（10000000）生成器 
```

\- redis获取值
conn = Redis(...)

　　　　def hscan_iter(self, name, match=None, count=None):
　　　　　　"""
　　　　　　Make an iterator using the HSCAN command so that the client doesn't
　　　　　　need to remember the cursor position.

　　　　　　``match`` allows for filtering the keys by pattern

　　　　　　``count`` allows for hint the minimum number of returns
　　　　　　"""
　　　　　　cursor = '0'
　　　　　　while cursor != 0:

去redis中获取数据：12

cursor，下一次取的位置

data：本地获取的12条数数据

　　　　　　　　cursor, data = self.hscan(name, cursor=cursor,match=match, count=count)
　　　　　　　　for item in data.items():
　　　　　　　　　　yield item

```
stark组件def index(request):
```

　　　　data = [
　　　　　　{'k1':1,'name':'alex'},
　　　　　　{'k1':2,'name':'老男孩'},
　　　　　　{'k1':3,'name':'小男孩'},
　　　　]
　　　　new_data = []
　　　　for item in data:
　　　　　　item['email'] = "xxx@qq.com"
　　　　　　new_data.append(item)

　　　　return render(request,'xx.html',{'data':new_data})

### 可迭代对象

 一个类内部实现__iter__方法且返回一个迭代器。

```
应用场景： 
    - wtforms中对form对象进行循环时候，显示form中包含的所有字段。
        class LoginForm(Form):
            name = simple.StringField(
                label='用户名',
                validators=[
                    validators.DataRequired(message='用户名不能为空.'),
                    validators.Length(min=6, max=18, message='用户名长度必须大于%(min)d且小于%(max)d')
                ],
                widget=widgets.TextInput(),
                render_kw={'class': 'form-control'}
            )
            pwd = simple.PasswordField(
                label='密码',
                validators=[
                    validators.DataRequired(message='密码不能为空.'),
                    validators.Length(min=8, message='用户名长度必须大于%(min)d'),
                    validators.Regexp(regex="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[$@$!%*?&])[A-Za-z\d$@$!%*?&]{8,}",
                                      message='密码至少8个字符，至少1个大写字母，1个小写字母，1个数字和1个特殊字符')

                ],
                widget=widgets.PasswordInput(),
                render_kw={'class': 'form-control'}
            )

        
        form = LoginForm()
        for item in form:
            print(item)
            
    - 列表、字典、元组
```

### 装饰器

```
 装饰器：
能够在不修改原函数代码的基础上，在执行前后进行定制操作，闭包函数的一种应用
场景：
   - flask路由系统
   - flask before_request
   - csrf
   - django内置认证
   - django缓存
# 手写装饰器；
import functools
def wrapper(func):
   @functools.wraps(func)  #不改变原函数属性
   def inner(*args, **kwargs):
      执行函数前
      return func(*args, **kwargs)
      执行函数后
   return inner
1. 执行wapper函数，并将被装饰的函数当做参数。 wapper(index)
2. 将第一步的返回值，重新赋值给  新index =  wapper(老index)
@wrapper    #index=wrapper(index)
def index(x):
   return x+100
```

调用装饰器其实是一个闭包函数，为其他函数添加附加功能，不修改被修改的源代码和不修改被修饰的方式，装饰器的返回值也是一个函数对象。
比如：插入日志、性能测试、事物处理、缓存、权限验证等，有了装饰器，就可以抽离出大量与函数功能本身无关的雷同代码并继续重用。

###  52. 用Python实现一个二分查找的函数。

二分查找算法：简单的说，就是将一个列表先排序好，比如按照从小到大的顺序排列好，当给定一个数据，比如3，查找3在列表中的位置时，可以先找到列表中间的数li[middle]和3进行比较，当它比3小时，那么3一定是在列表的右边，反之，则3在列表的左边，比如它比3小，则下次就可以只比较[middle+1, end]的数，继续使用二分法，将它一分为二，直到找到3这个数返回或者列表全部遍历完成（3不在列表中） 

优点：效率高，时间复杂度为O(logN)； 
缺点：数据要是有序的，顺序存储。

```
li = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
 
 
def search(someone, li):
    l = -1
    h = len(li)
 
    while l + 1 != h:
        m = int((l + h) / 2)
        if li[m] < someone:
            l = m
        else:
            h = m
    p = h
    if p >= len(li) or li[p] != someone:
        print("元素不存在")
    else:
        str = "元素索引为%d" % p
        print(str)
 
 
search(3, li)  # 元素索引为2
```

### 53. 谈谈你对闭包的理解？

```
ef foo():
    m=3
    n=5
    def bar():
        a=4
        return m+n+a
    return bar
  
>>>bar =  foo()
>>>bar()
12
```

说明：
bar在foo函数的代码块中定义。我们称bar是foo的内部函数。
在bar的局部作用域中可以直接访问foo局部作用域中定义的m、n变量。
简单的说，这种内部函数可以使用外部函数变量的行为，就叫闭包。

闭包的意义与应用

### 54. os和sys模块的作用？

os模块负责程序与操作系统的交互，提供了访问操作系统底层的接口;
sys模块负责程序与python解释器的交互，提供了一系列的函数和变量，用于操控python的运行时环境。

```
os与sys模块的官方解释如下：
os: This module provides a portable way of using operating system dependent functionality.
这个模块提供了一种方便的使用操作系统函数的方法。
sys: This module provides access to some variables used or maintained by the interpreter and to functions that interact strongly with the interpreter.
这个模块可供访问由解释器使用或维护的变量和与解释器进行交互的函数。
os 常用方法
os.remove() 删除文件
os.rename() 重命名文件
os.walk() 生成目录树下的所有文件名
os.chdir() 改变目录
os.mkdir/makedirs 创建目录/多层目录
os.rmdir/removedirs 删除目录/多层目录
os.listdir() 列出指定目录的文件
os.getcwd() 取得当前工作目录
os.chmod() 改变目录权限
os.path.basename() 去掉目录路径，返回文件名
os.path.dirname() 去掉文件名，返回目录路径
os.path.join() 将分离的各部分组合成一个路径名
os.path.split() 返回( dirname(), basename())元组
os.path.splitext() 返回 (filename, extension) 元组
os.path.getatime\ctime\mtime 分别返回最近访问、创建、修改时间
os.path.getsize() 返回文件大小
os.path.exists() 是否存在
os.path.isabs() 是否为绝对路径
os.path.isdir() 是否为目录
os.path.isfile() 是否为文件
sys 常用方法
sys.argv 命令行参数List，第一个元素是程序本身路径
sys.modules.keys() 返回所有已经导入的模块列表
sys.exc_info() 获取当前正在处理的异常类,exc_type、exc_value、exc_traceback当前处理的异常详细信息
sys.exit(n) 退出程序，正常退出时exit(0)
sys.hexversion 获取Python解释程序的版本值，16进制格式如：0x020403F0
sys.version 获取Python解释程序的版本信息
sys.maxint 最大的Int值
sys.maxunicode 最大的Unicode值
sys.modules 返回系统导入的模块字段，key是模块名，value是模块
sys.path 返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值
sys.platform 返回操作系统平台名称
sys.stdout 标准输出
sys.stdin 标准输入
sys.stderr 错误输出
sys.exc_clear() 用来清除当前线程所出现的当前的或最近的错误信息
sys.exec_prefix 返回平台独立的python文件安装的位置
sys.byteorder 本地字节规则的指示器，big-endian平台的值是'big',little-endian平台的值是'little'
sys.copyright 记录python版权相关的东西
sys.api_version 解释器的C的API版本
总结：
os模块负责程序与操作系统的交互，提供了访问操作系统底层的接口;sys模块负责程序与python解释器的交互，提供了一系列的函数和变量，用于操控python的运行时环境。
```

### 55. 如何生成一个随机数？

```
import random
 
print(random.random())          # 用于生成一个0到1的随机符点数: 0 <= n < 1.0
print(random.randint(1, 1000))  # 用于生成一个指定范围内的整数
```

### 56. 如何使用python删除一个文件？

```
import os
file = r'D:\test.txt'
if os.path.exists(file):
    os.remove(file)
    print('delete success')
else:
    print('no such file:%s' % file)
```

### 57. 谈谈你对面向对象的理解

#### 三大特性以及解释？

面对对象是一种编程思想，以类的眼光来来看待事物的一种方式。将有共同的属性和方法的事物封装到同一个类下面。

继承：将多个类的共同属性和方法封装到一个父类下面，然后在用这些类来继承这个类的属性和方法

封装：将有共同的属性和方法封装到同一个类下面

- 第一层面：创建类和对象会分别创建二者的名称空间，我们只能用类名.或者obj.的方式去访问里面的名字，这本身就是一种封装
- 第二层面：类中把某些属性和方法隐藏起来(或者说定义成私有的)，只在类的内部使用、外部无法访问，或者留下少量接口（函数）供外部访问。

多态：Python天生是支持多态的。指的是基类的同一个方法在不同的派生类中有着不同的功能

### 58. Python面向对象中的继承有什么特点

```
继承概念的实现方式主要有2类：实现继承、接口继承。

         实现继承是指使用基类的属性和方法而无需额外编码的能力；
         接口继承是指仅使用属性和方法的名称、但是子类必须提供实现的能力(子类重构爹类方法)；
python 两种类：经典类 新式类
python3 新式类 —— 都默认继承object class Animal(object): == class Animal:
python2 经典类和新式类 并存
        class Animal:  经典类 —— 继承顺序 个别使用方法
        class Animal(object):  新式类

继承分为单继承和多继承
Python是支持多继承的
如果没有指定基类，python的类会默认继承object类，object是所有python类的基类，它提供了一些常见方法（如__str__）的实现。
```

补充继承的应用（面试题）

```
1、对象可以调用自己本类和父类的所有方法和属性， 先调用自己的 自己没有才调父类的。谁（对象）调用方法，方法中的self就指向谁

class Foo:
    def __init__(self):
        self.func()

    def func(self):
        print('Foo.func')

class Son(Foo):
    def func(self):
        print('Son.func')

s = Son()
 # Son.func

========================================================
class A:
    def get(self):
        self.say()

    def say(self):
        print('AAAAA')

class B(A):
    def say(self):
        print('BBBBB')

b = B()
b.get()   #输出结果为：BBBBB
```

### 59. 面向对象深度优先和广度优先是什么？

```
Python的类可以继承多个类，Python的类如果继承了多个类，那么其寻找方法的方式有两种
当类是经典类时，多继承情况下，会按照深度优先方式查找  py3
当类是新式类时，多继承情况下，会按照广度优先方式查找  py2
简单点说就是：经典类是纵向查找，新式类是横向查找
经典类和新式类的区别就是，在声明类的时候，新式类需要加上object关键字。在python3中默认全是新式类
```

### 60. 面向对象中super的作用？

```
用于子类继承基类的方法
```

```
class FooParent(object):
    def __init__(self):
        self.parent = 'I\'m the parent.'
        print('Parent')
        print('1111')

    def bar(self, message):
        print("%s from Parent" % message)


class FooChild(FooParent):
    def __init__(self):
        # super(FooChild,self) 首先找到 FooChild 的父类（就是类 FooParent），然后把类B的对象 FooChild 转换为类 FooParent 的对象
        super(FooChild, self).__init__()
        print('Child')

    # def bar(self, message):
    #     # super(FooChild, self).bar(message)
    #     print('Child bar fuction')
    #     print(self.parent)


if __name__ == '__main__':
    fooChild = FooChild()
    fooChild.bar('HelloWorld')
```

### 61. 是否使用过functools中的函数？其作用是什么？

用于修复装饰器

```
import functools
 
def deco(func):
    @functools.wraps(func)  # 加在最内层函数正上方
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
 
    return wrapper
 
 
@deco
def index():
    '''哈哈哈哈'''
    x = 10
    print('from index')
 
 
print(index.__name__)
print(index.__doc__)
 
# 加@functools.wraps
# index
# 哈哈哈哈
 
# 不加@functools.wraps
# wrapper
# None
```

### 62. 列举面向对象中带双下划线的特殊方法，如：__new__、__init__

- __new__：生成实例

- __init__：生成实例的属性

- __call__：实例对象加( )会执行def __call__:... 方法里边的内容。

  __del__：析构方法，当对象在内存中被释放时，自动触发执行。如当 del obj 或者应用程序运行完毕时，执行该方法里边的内容。

  __enter__和__exit__：出现with语句,对象的__enter__被触发,有返回值则赋值给as声明的变量；with中代码块执行完毕时执行__exit__里边的内容。

  __module__：表示当前操作的对象在那个模块   obj.__module__
  __class__ ：表示当前操作的对象的类是什么     obj.__class__

  __doc__：类的描述信息，该描述信息无法被继承

  __str__：改变对象的字符串显示 print函数 --->obj.__str__()
  __repr__：改变对象的字符串显示 交互式解释器 --->obj.__repr__()
  __format__：自定制格式化字符串

  __slots__:一个类变量 用来限制实例可以添加的属性的数量和类型  

  __setitem__,__getitem,__delitem__:

```
class Foo:
    def __init__(self,name):
        self.name=name

    def __getitem__(self, item):
        print(self.__dict__[item])

    def __setitem__(self, key, value):
        self.__dict__[key]=value
    def __delitem__(self, key):
        print('del obj[key]时,我执行')
        self.__dict__.pop(key)
    def __delattr__(self, item):
        print('del obj.key时,我执行')
        self.__dict__.pop(item)

f1=Foo('sb')
f1['age']=18
f1['age1']=19
del f1.age1
del f1['age']
f1['name']='alex'
print(f1.__dict__)
```

__get__():调用一个属性时,触发
__set__():为一个属性赋值时,触发
__delete__():采用del删除属性时,触发

__setattr__,__delattr__,__getattr__ :

### 63. 如何判断是函数还是方法？

看他的调用者是谁，如果是类，就需要传入一个参数self的值，这时他就是一个函数，

如果调用者是对象，就不需要给self传入参数值，这时他就是一个方法

print(isinstance(obj.func, FunctionType))   # False

print(isinstance(obj.func, MethodType))    # True

```
class Foo(object):
    def __init__(self):
        self.name = 'lcg'
 
    def func(self):
        print(self.name)
 
 
obj = Foo()
print(obj.func)  # <bound method Foo.func of <__main__.Foo object at 0x000001ABC0F15F98>>
 
print(Foo.func)  # <function Foo.func at 0x000001ABC1F45BF8>
 
# ------------------------FunctionType, MethodType------------#
 
 
from types import FunctionType, MethodType
 
obj = Foo()
print(isinstance(obj.func, FunctionType))  # False
print(isinstance(obj.func, MethodType))  # True
 
print(isinstance(Foo.func, FunctionType))  # True
print(isinstance(Foo.func, MethodType))  # False
 
# ------------------------------------------------------------#
obj = Foo()
Foo.func(obj)  # lcg
 
obj = Foo()
obj.func()  # lcg
 
"""
注意：
    方法，无需传入self参数
    函数，必须手动传入self参数
"""
```

### 64. 静态方法和类方法区别？

尽管 classmethod 和 staticmethod 非常相似，但在用法上依然有一些明显的区别。classmethod 必须有一个指向类对象的引用作为第一个参数，而 staticmethod 可以没有任何参数。

举个栗子:

```
class Num:
    # 普通方法：能用Num调用而不能用实例化对象调用   
    def one():  
        print ('1')
 
    # 实例方法：能用实例化对象调用而不能用Num调用
    def two(self):
        print ('2')
 
    # 静态方法：能用Num和实例化对象调用
    @staticmethod 
    def three():  
        print ('3')
 
    # 类方法：第一个参数cls长什么样不重要，都是指Num类本身，调用时将Num类作为对象隐式地传入方法   
    @classmethod 
    def go(cls): 
        cls.three() 
 
Num.one()          #1
#Num.two()         #TypeError: two() missing 1 required positional argument: 'self'
Num.three()        #3
Num.go()           #3
 
i=Num()                
#i.one()           #TypeError: one() takes 0 positional arguments but 1 was given         
i.two()            #2      
i.three()          #3
i.go()             #3 
```

### 65. 列举面向对象中的特殊成员以及应用场景

```
__call__

__new__

__init__

__doc__

__class__

__del__

__dict__

__str__

在falsk源码用到......
```

### 66. 1、2、3、4、5 能组成多少个互不相同且无重复的三位数

60个

题意理解：组成后的数值不相同，且组合的三个位数之间数字不重复。

使用python内置的排列组合函数（不放回抽样排列）

product 笛卡尔积　　（有放回抽样排列）

permutations 排列　　（不放回抽样排列）

combinations 组合,没有重复　　（不放回抽样组合）

combinations_with_replacement 组合,有重复　　（有放回抽样组合）

```
import itertools
 
print(len(list(itertools.permutations('12345', 3))))  # 60
```

### 67. 什么是反射？以及应用场景？

```
反射的核心本质就是以字符串的形式去导入个模块，利用字符串的形式去执行函数。

Django中的 CBV就是基于反射实现的。
```

### 68. metaclass作用？以及应用场景？

metaclass用来指定类是由谁创建的。

类的metaclass 默认是type。我们也可以指定类的metaclass值。在python3中：

```
class MyType(type):
    def __call__(self, *args, **kwargs):
        return 'MyType'
  
  
class Foo(object, metaclass=MyType):
    def __init__(self):
        return 'init'
  
    def __new__(cls, *args, **kwargs):
        return cls.__init__(cls)
  
    def __call__(self, *args, **kwargs):
        return 'call'
  
  
obj = Foo()
print(obj)  # MyType
```



 ![img](https://images2018.cnblogs.com/blog/1258691/201805/1258691-20180524152039412-696009410.png)



### 69. 用尽量多的方法实现单例模式。

```
1：使用模块
Python的模块就是天然的单例模式。
因为模块在第一次导入时，会生成 .pyc 文件，当第二次导入时，就会直接加载 .pyc 文件，而不会再次执行模块代码。
因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。
例如：
class V1(object):
    def foo(self)
        pass
V1 = V1()
将上面代码保存在文件test.py,要使用时，直接在其他文件中导入此文件中的对象，这个对象既是单例模式的对象

如：from a import V1

2：使用装饰器
def Singleton(cls):
    _instance = {}
    def _singleton(*args, **kargs):
        if cls not in _instance:
            _instance[cls] = cls(*args, **kargs)
        return _instance[cls]
    return _singleton
@Singleton
class A(object):
    a = 1
    def __init__(self, x=0):
        self.x = x
a1 = A(2)
a2 = A(3)


3：使用类

4：基于__new__方法实现
当我们实例化一个对象时，是先执行了类的__new__方法
当：（我们没写时，默认调用object.__new__），实例化对象；然后再执行类的__init__方法，对这个对象进行初始化，所有我们可以基于这个，实现单例模式
```

### 70. 装饰器器的写法以及应用场景。

含义：装饰器本质就是函数，为其他函数添加附加功能

原则：

不修改被修饰函数的代码

不修改被修饰函数的调用方式

应用场景：

无参装饰器在用户登录 认证中常见

有参装饰器在flask的路由系统中见到过

```python
import functools
def wrapper(func):
    @functools.wraps(func)
    def inner(*args, **kwargs):
        print('我是装饰器')
        return func
return inner

@wrapper
def index():
    print('我是被装饰函数')
    return None
index()

# 应用场景
    - 高阶函数
    - 闭包
    - 装饰器 
    - functools.wraps(func)
```

### 71. 异常处理写法以及如何主动跑出异常（应用场景）

```python
# 触发异常
def temp_convert(var):
    try:
        return int(var)
    except ValueError as Argument:
        print ("参数没有包含数字%s"%Argument)

# 调用函数
temp_convert("xyz")
# 以10为基数的int()的无效文字:“xyz”
----------------------------------------------------------------------------
# raise语法
#raise [Exception [, args [, traceback]]]
# 语句中 Exception 是异常的类型，args 是自已提供的异常参数。

class Networkerror(RuntimeError):
    def __init__(self, arg):
        self.args = arg
try:
    raise Networkerror("Bad hostname")
except Networkerror as e:
    print(e.args)
```

### 72、什么是面向对象的mro

mro是面向对象中多继承中的一个继承顺序。

### 73. isinstance作用以及应用场景？

isinstance(对象，类)  判断这个对象是不是这个类或者这个类的子类的实例化

```python
# # 判断a 属不属于A这个类（可以判断到祖宗类）
class A:
    pass

class B(A):
    pass
a = A()
b = B()
print(isinstance(b,A)) # ===> True  判断到祖宗类

# 任何与object都是True,内部都继承object
class A:pass
a = A()  # 实例化
print(isinstance(a,object))  #  True
```

应用场景：

rest framework 认证的流程

scrapy-redis

### 74. 写代码并实现

给定一个整数数组，返回两个数字的索引，使它们加起来成为一个特定的目标不使用同一元素两次。
Example:
Given nums = [2, 7, 11, 15], target = 9,
Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1]

### 75. json序列化时，可以处理的数据类型有哪些？如何定制支持datetime类型？

json序列化支持：字符串、数字、布尔值、Null、对象、数组

```python
自定义时间序列化转换器
import json
from json import JSONEncoder
from datetime import datetime
class ComplexEncoder(JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.strftime('%Y-%m-%d %H:%M:%S')
        else:
            return super(ComplexEncoder,self).default(obj)
d = { 'name':'alex','data':datetime.now()}
print(json.dumps(d,cls=ComplexEncoder))
# {"name": "alex", "data": "2018-05-18 19:52:05"}
```

###  76. json序列化时，默认遇到中文会转换成unicode，如果想要保留中文怎么办？

```
 在序列化时，中文汉字总是被转换为unicode码，在dumps函数中添加参数ensure_ascii=False即可解决。
```

### 77. 什么是断言？应用场景？

```
assert 是的作用？断言
条件成立（布尔值为True）则继续往下，否则跑出异常，一般用于：满足某个条件之后，才能执行，否则应该跑出异常。

写API的时候，继承GenericAPIView

class GenericAPIView(views.APIView):
                    """
                    Base class for all other generic views.
                    """
                    # You'll need to either set these attributes,
                    # or override `get_queryset()`/`get_serializer_class()`.
                    # If you are overriding a view method, it is important that you call
                    # `get_queryset()` instead of accessing the `queryset` property directly,
                    # as `queryset` will get evaluated only once, and those results are cached
                    # for all subsequent requests.
                    queryset = None
                    serializer_class = None

                    # If you want to use object lookups other than pk, set 'lookup_field'.
                    # For more complex lookup requirements override `get_object()`.
                    lookup_field = 'pk'
                    lookup_url_kwarg = None

                    # The filter backend classes to use for queryset filtering
                    filter_backends = api_settings.DEFAULT_FILTER_BACKENDS

                    # The style to use for queryset pagination.
                    pagination_class = api_settings.DEFAULT_PAGINATION_CLASS

                    def get_queryset(self):

                        assert self.queryset is not None, (
                            "'%s' should either include a `queryset` attribute, "
                            "or override the `get_queryset()` method."
                            % self.__class__.__name__
                        )

                        queryset = self.queryset
                        if isinstance(queryset, QuerySet):
                            # Ensure queryset is re-evaluated on each request.
                            queryset = queryset.all()
                        return queryset
```

### 78. 有用过with statement吗？它的好处是什么？

with语句的作用就是通过某种方式简化异常处理，它是所谓的上下文管理器的一种。

**上下文管理协议（Context Management Protocol）**：包含方法 __enter__() 和 __exit__()，支持该协议的对象要实现这两个方法。

**上下文管理器（Context Manager）**：支持上下文管理协议的对象，这种对象实现了__enter__() 和 __exit__() 方法。上下文管理器定义执行 with 语句时要建立的运行时上下文，负责执行 with 语句块上下文中的进入与退出操作。通常使用 with 语句调用上下文管理器，也可以通过直接调用其方法来使用。

**运行时上下文（runtime context）**：由上下文管理器创建，通过上下文管理器的 __enter__() 和__exit__() 方法实现，__enter__() 方法在语句体执行之前进入运行时上下文，__exit__() 在语句体执行完后从运行时上下文退出。with 语句支持运行时上下文这一概念。

**上下文表达式（Context Expression）**：with 语句中跟在关键字 with 之后的表达式，该表达式要返回一个上下文管理器对象。

**语句体（with-body）**：with 语句包裹起来的代码块，在执行语句体之前会调用上下文管理器的 __enter__() 方法，执行完语句体之后会执行 __exit__() 方法。

**基本语法**

with 语句的语法格式如下：

```python
with context_expression [as target(s)]:
    with-body
```

https://blog.csdn.net/ghalcyon/article/details/82178251

###  79. 使用代码实现查看列举目录下的所有文件。

os.listdir(),使用的是os模块：https://chpl.top/2019/08/05/%E4%B9%A6/python%E5%9F%BA%E7%A1%80/%E7%AC%AC%E5%8D%81%E4%B8%83%E8%AE%B2%E2%80%94%E2%80%94%E6%A8%A1%E5%9D%97%E4%BA%8C/

###  80. 简述 yield和yield from关键字

```
yield返回的是一个可迭代对象
yield from是将可迭代对象中的元素一个个的yield出来
```

### 81.65什么是元类

```
也就是object和type的关系
记住两句话：
所有的类都继承object，包括type
type实例化一切，包括自己

扩展：可以这么说，元类是类的类，元定义了类的实例的行为，而元类（type）定义了类（class）的行为方式。类是元类的实例。
回答这个问题的时候不要去引的太深，不然就很容易掉坑，我们就说上面的内容就好了。
```

### 82.链表如何逆转？

```python
class Lnode:#d定义节点
    def __init__(self,x):
        self.val=x
        self.next=None

def Reverse(phead):
        if phead or phead.next is None: #判断是否为空表
            return phead
        newhead = None #建立新链表
        while phead:
            temp = phead.next #先将原链表的第二个节点保存在temp中
            phead.next = newhead #将原链表的第一个节点变为新链表的最后一个节点
            newhead = phead
            phead = temp
        return newhead

if __name__ == '__main__':
    phead = Lnode(1)#建立链表1->2->3->None
    p1 = Lnode(2)
    p2 = Lnode(3)
    phead.next = p1
    p1.next = p2
    p = Reverse(phead)
    while p:
        print(p.val)
        p = p.next
```

### 83.两个队列生成一个栈？

```python
思路：栈（后进先出）
进栈：元素如队列1
出栈：判断队列1中是否只有一个元素，直接出队。否则1中元素出队并入队2，直到1中只有一个元素，直接出队
class TwoQueueOneStack(object):
	def __init__(self):
        self.queue1 = []
        self.queue2 = []
    def push(self,item):
        #进队
        self.queue1.append(item)
        
    def pop(self):
        #弹出到最后一个为止
        #队列1和2交换为止
        #弹出队列2中的元素
        if len(self.queue1) == 0:
            return None
        while len(self.queue1) != 1:
            #这里1把最后一个进来的元素留下，下面交换的时候，把这最后进来的数据给了2，这样pop的时候就把后进来的数据pop出去
        	self.queue2.append(self.queue1.pop(0))
        self.queue1,self.queue2 = self.queue2,self.queue1
        return self.queue2.pop(0)

stack = TwoQueueOneStack()
stack.push(1)
stack.pop()
```

### 84.什么是偏函数？

```python
	定义：偏函数是将所要承载的函数作为partial()函数的第一个参数，原函数的各个参数依次作为partial()函数后续的参数，除非使用关键字参数。

import functools
# 无法体会偏函数参数的位置问题，容易给人造成partial的第二个参数也是原函数的第二个参数的假象
def remainder(m, n):
    return m % n

print(remainder(100, 7)) # 2

# 使用偏函数的
new_rmd = functools.partial(remainder, 100)
print(new_rmd(7)) # 2
使用偏函数可以通过有效地“冻结”那些预先确定的参数，来缓存函数参数，然后进行运行时，当获取需要的剩余参数后，可以将他们解冻，传递到最终的参数中，从而使用最终确定的所有参数去调用函数。

在哪里使用比较合适：对于有很多可调用对象，并且许多调用都反复使用相同参数的情况，使用偏函数比较合适。
```

### 85.介绍一下range和xrange的区别？

```
在python3中，range返回的是一个可迭代的对象，我们在range里面写入什么就给我们打印什么
在python2中，range返回的是一个list
xrange返回的是一个生成器，在python3之前，xrange生成大量数据的方式比range的方式好，因为生成器不需要开辟大片的空间。所以，后来新的版本，就修改了range。
```



