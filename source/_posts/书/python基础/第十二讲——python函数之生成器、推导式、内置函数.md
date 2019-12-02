---
title: 第十二讲——python函数之生成器、推导式、内置函数
id: 12
date: 2019-7-31 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 生成器
- 推导式
- 内置函数
- 练习

<!-----more----->

### 生成器

这一部分的内容因为比较重要，我单独写了一篇blog：https://blog.csdn.net/qq_40890660/article/details/97618316

- 首先我们回顾一下什么是迭代器：迭代器的本质就是python内置的一种节省空间的一种工具。

- 什么是生成器？

   - 其实生成器的本质就是一个迭代器，在python社区中，大多数的视乎都把迭代器和生成器是同一个概念。那么他们有什么不同呢？可能就是一个是python直接为我们提供的，一个是我们手写的区别了。

- 生成器函数

   - ```python
     def foo():
     	if 3 > 1:
     		yield "我是第一次返回"
     	if 3 > 2:
     		yield "我是第二次返回"
     
     s = foo()   #生成一个迭代器
     # print (s.__next__())
     # print (s.__next__())
     
     for i in s:  #把生成器的元素全部打印出来
     	print (i)
     ```

- **yield与return的区别：**

   return一般在函数中只设置一个，他的作用是终止函数，并且给函数的执行者返回值。

   yield在生成器函数中可设置多个，他并不会终止函数，next会获取对应yield生成的元素。

   **举例：**

   我们来看一下这个需求：老男孩向楼下卖包子的老板订购了10000个包子.包子铺老板非常实在，一下就全部都做出来了　

   ```python
   def eat():
       lst = []
       for i in range(1,10000):
           lst.append('包子'+str(i))
       return lst
   e = eat()
   print(e)
   ```

   这样做没有问题，但是我们由于学生没有那么多，只吃了2000个左右，剩下的8000个，就只能占着一定的空间，放在一边了。如果包子铺老板效率够高，我吃一个包子，你做一个包子，那么这就不会占用太多空间存储了，完美。

   ```python
   def eat():
       for i in range(1,10000):
           yield '包子'+str(i)
   e = eat()
   for i in range(200):
       next(e)
   ```

   **这两者的区别:**

    第一种是直接把包子全部做出来，占用内存。

    第二种是吃一个生产一个，非常的节省内存，而且还可以保留上次的位置。

   ```python
   def eat():
       for i in range(1,10000):
           yield '包子'+str(i)
   e = eat()
   for i in range(200):
       next(e)
   for i in range(300):
       next(e)
   # 多次next包子的号码是按照顺序记录的。
   ```

- 坑

   ```python
   例子1：
   def foo1():
   	for i in range(10):
   		pass
   	yield i
   	count = 0
   	while True:
   		yield  count
   		count += 1
   		yield count
   #这是一个坑，为什么会一直打印9，因为每次的foo1（）都是生一个迭代器。所以需要把迭代器赋值
   print (foo1().__next__())  #打印9
   print (foo1().__next__())  #打印9
   print (foo1().__next__())  #打印9
   #正确的取值
   s = foo1()
   for i in foo1():
    	print (i)
   例子2：
   def func():
       lst1 = ['卫龙', '老冰棍', '北冰洋', '牛羊配']
       lst2 = ['馒头', '花卷', '豆包', '大饼']
       yield from lst1
       yield from lst2
   #第一眼看的时候，我们以为会先返回第一个列表的第一个元素，再但会第二个列表的第一个元素。但是这是错误的思想。
   g = func()
   for i in g:
       print(i)
   #正确的答案应该是：先把第一个列表的元素全部打印出来，然后再把第二个列表的元素打印出来。
   ```

- 总结：

   1，写一个生成器：

   Def foo():

   Yield 123  #把return替换成yield就是一个生成器

   2，Foo()#这就是一个迭代器 Yield可以记录执行的位置

   3，生成器的本质就是一个迭代器

   4，生成器和迭代器的区别：一个是python自带的，一个是 自己写的

   5，Yield也是返回的意思，可以返回多次，return可以写多个但是只能执行一个。

   6，一个__next__()对应一个yield

   7，生成器可以使用for循环获取

   For i in s:

   ​	Yiled i

   #yield from s这行代码就相当于上面的两行代码

   8，在函数的内部yield能将for循环和while循环进行临时暂停 

### 推导式

```python
#推导式
# list推导式
#循环模式
lst = []
for i in range(20):
	lst.append(i)
print(lst)
#推到print ([i for i in range(20)])
#筛选模式
print([i for i in range(20)])
lst = []
for i in range(20):
	if i % 2 == 0:
		lst.append(i)
print (lst)
#推到print ([i for i in range(20) if i % 2 == 0])

#生成器表达式
#循环模式
g = (i for i in range(20))
print (next(g))
print (next(g))
print (next(g))
print (next(g))
#推到print (list((i for i in range(20))))
#筛选模式
g = (i for i in range(50) if i % 2 == 1)
for i in g:
	print(i)

#字典
print ({i:i+2 for i in range(20)})
#集合
print ({i for i in range(20)})

#练习
print ([i**2 for i in range(10)])
print ([f"python{i}期" for i in range(1,25)])
lst = [1,323,6,465,22,3465,43,13]
print ([i for i in lst if i>3])
```

**生成器表达式和列表推导式的区别**

1. 列表推导式比较消耗内存，所有的数据一次性加载到内存中。而生成器表达式遵循迭代器协议，逐个产生元素。
2. 得到的值不一样，列表推导式得到的是一个列表，生成器表达式得到的是一个生成器。
3. 列表推导式一目了然，生成器表达式只是一个内存地址。
4. 注意： 无论是生成器表达式，还是列表推导式，他只是Python给你提供了一个相对简单的构造方式，因为使用推导式非常简单，所以大多数都会为之着迷，这个一定要深重，推导式只能构建相对复杂的并且有规律的对象，对于没有什么规律，而且嵌套层数比较多（for循环超过三层）这样就不建议大家用推导式构建。

**总结：**

- List:
  - [变量（加工后的变量） for 循环]
  - [变量(加工后的变量) for循环 加工条件]

- 生成器表达式（元组）:
  - (变量(加工后的变量) for循环)
  - (变量(加工后的变量) for循环 加工条件)

- Dict:
  - {键:值 for循环 加工条件}

- Set:
  - {变量 for循环}

### 内置函数

**不是太常用的**：all()  any()  bytes() callable() chr() complex() divmod() eval() exec() format() frozenset() globals() hash() help() id() input() int()  iter() locals() next()  oct()  ord()  pow()    repr()  round()

**重点讲解**（非常重要）：abs() enumerate() filter()  map() max()  min() open()  range() print()  len()  list()  dict() str()  float() reversed()  set()  sorted()  sum()    tuple()  type()  zip()  dir() 

**在类中会讲**（重要）： classmethod()  setattr() delattr() getattr() hasattr()  issubclass()  isinstance()  object() property()    staticmethod()  super()

```python
# s = """
# for i in range(10):
#     print(i)
# """

# s1 = """
# def func():
#     print(123)
# func()
# """
# print(eval(s))
# print(exec(s1))  # 牛逼 不能用

# print(hash("asdfas"))

# print(help(list))
# help(dict)


# def func():
#     pass
# print(callable(func))  # 查看是否可调用

# print(float(2))     # 浮点数
# print(complex(56))  # 复数

# print(oct(15))        # 八进制
# print(hex(15))        # 十六进制

# print(divmod(5,2))     # (2, 1) 2商 1余

# print(round(5.3234,2))     # 四舍五入 -- 默认是整数,可以指定保留小数位

# print(pow(2,3))            # 幂
# print(pow(2,3,4))          # 幂,余

# s = "alex"
# print(bytes(s,encoding="utf-8"))

# print(ord("你"))    # 当前编码
# print(chr(20320))

# s = "C:\u3000"
# print(repr(s))

# print("\u3000你好")

# lst = [1,2,3,False,4,5,6,7]
# print(all(lst))   # 判断元素是否都为真  相似and
# print(any(lst))     # 判断元素是否有真    相似or

# name = 1
# def func():
#     a = 123
#     # print(locals())
#     # print(globals())
# func()

# print(globals())   # 全局空间中的变量
# print(locals())   # 查看当前空间的变量
```

### 练习

```python
# 2.用列表推导式做下列小题
# 过滤掉长度小于3的字符串列表，并将剩下的转换成大写字母
print ([i for i in ['ssdff',"sdf12","sd25d",'54','s'] if len(i) > 3])
# 求(x,y)其中x是0-5之间的偶数，y是0-5之间的奇数组成的元祖列表
print ([(i,j) for i in range(5) for j in range(5) if i%2==0 and j%2!=0])
# 求M中3,6,9组成的列表
M = [[1,2,3],[4,5,6],[7,8,9]]
print ([i[2] for i in M if (i[2] % 3 == 0)])
# 求出50以内能被3整除的数的平方，并放入到一个列表中。
print ([i**2 for i in range(1,51) if i % 3 ==0])
# 构建一个列表：['python1期', 'python2期', 'python3期', 'python4期', 'python6期', 'python7期', 'python8期', 'python9期', 'python10期']
print ([f"python{i}期" for i in range(1,11)])
# 构建一个列表：[(0, 1), (1, 2), (2, 3), (3, 4), (4, 5), (5, 6)]
print ([(i,i+1) for i in range(6)])
# 构建一个列表：[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
print ([i for i in range(20) if i % 2 == 0])
# 有一个列表l1 = ['alex', 'WuSir', '老男孩', '太白']将其构造成这种列表['alex0', 'WuSir1', '老男孩2', '太白3']
l1 = ['alex', 'WuSir', '老男孩', '太白']
print ([l1[i] + str(i) for i in range(4)])
```

```python
# 3.有以下数据类型：
x = {
'name':'alex',
'Values':[
{'timestamp':1517991992.94,
'values':100,},
{'timestamp': 1517992000.94,
'values': 200,},
{'timestamp': 1517992014.94,
'values': 300,},
{'timestamp': 1517992744.94,
'values': 350},
{'timestamp': 1517992800.94,
'values': 280}
],}
# 将上面的数据通过列表推导式转换成下面的类型：[[1517991992.94, 100], [1517992000.94, 200], [1517992014.94, 300], [1517992744.94, 350], [1517992800.94, 280]]
print ([[i['timestamp'],i['values']] for i in x["Values"]])
```

```python
# 4.构建一个列表，列表里面是三种不同尺寸的T恤衫，每个尺寸都有两个颜色（列表里面的元素为元组类型）。
colors = ['black', 'white']
sizes = ['S', 'M', 'L']
print ([(i,tuple(colors)) for i in sizes])
```

```python
# 5,构建一个列表,列表里面的元素是扑克牌除去大小王以后，所有的牌类（列表里面的元素为元组类型）。
print ([(i,j) for i in ['A','k','Q','J','10','9','8','7','6','5','4','3','2'] for j in ['spades','diamonds','clubs','hearts']])
```

```python
#6.简述一下yield 与yield from的区别。
#简述：yield：如果不循环，就是想对象返回。循环就是将可迭代对象逐个返回
def foo():
	for i in "可迭代对象":
		yield i
#yield from：将可迭代对象元素逐个返回。
def foo1():
	yield from "可迭代对象"  #(这个就是上面代码的简化)

for i in foo():
	print (i)
for i in foo1():
	print (i)
```

```python
# 7.看代码求结果（面试题）：
v = [i % 2 for i in range(10)]
print(v)
#>>>[0,1,0,1,0,1,0,1,0,1]
v = (i % 2 for i in range(10))
print(v)
#>>>(0,1,0,1,0,1,0,1,0,1)  #正确答案<generator object <genexpr> at 0x00000270BE36FB10>
#这道题为什么会错？就是因为，没有所谓的元组的推导式，()这种形式的其实构造的是一个生成器，所以这
#就是一个内存地址，但是可以通过for循环取里面的值
for i in range(5):
     print(i)
print(i)
#0 1 2 3 4 4
```

