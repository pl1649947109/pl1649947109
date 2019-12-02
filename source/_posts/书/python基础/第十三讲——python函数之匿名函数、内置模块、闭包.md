---
title: 第十三讲——python函数之匿名函数、内置模块、闭包
id: 13
date: 2019-8-1 20:00:00
tags: Python
comment: true
---

### 学习大纲：

- 匿名函数

- 重要的内置模块
- 闭包
- 练习

<!-----more----->

### 匿名函数

**介绍**

- 匿名函数，顾名思义就是没有名字的函数。
- 语法：变量 = lambda 参数：返回值
  - 此函数不是没有名字，它是有名字的，它的名字就是lambda
  - lambda后面直接加形参，形参加多少都是可以的，只要用逗号隔开就行。
  - 匿名函数不管多复杂，只能写一行，且逻辑结束后直接返回数据。

```python
#非常重要的：
# 1,lambda  匿名函数
f = lambda x,y:(x,y)
print(lambda x,y:(x,y))  #返回的也是内存地址
print (f(1,2))
print(f.__name__)  #这个方法是查看f变量对应的方法名的，就是lambda本身
print((lambda x:x)(2))  #同一行定义同一行调用
# 解释匿名函数：
# 	lambda：关键字，同时也是匿名函数的名字
# 	x,y：形参
# 	x+y：返回值--只能返回一个数据类型
lst = [lambda i:i*i for i in range(10)]
print (lst[2](2))
#>>>4
#上面代码拆开
lst = []
for i in range(10):
	def func(i):
		return  i*i
	lst.append(func)
#从推倒式我们知道[]里面，for后面的一部分是一部分，for前面的是一部分
#因此，我们的[]里面添加了10个内存地址，地址不一样但是指向的值是一样的，所以
#我们去lst[2]里面是9以内的都是可以调到这个函数的地址的。
lst = [lambda :i*i for i in range(10)]  #匿名函数可以没有参数，
print(lst[2]())  #为什么会打印81，就是因为函数没有参数，我们调用到这个函数，函数在本函数里面找不到i就去外面的for循环里面找，
#for循环了10次，最后i=9，因此，我们打印的结果就是81
#>>>81
lst = list((lambda i:i*i for i in range(5)))
print(lst[1](4))
#16
lst = [x for x in (lambda :i**i for i in range(5))]
print(lst[2]())
```

### 重要的内置模块

```python
#__*__coding:utf-8__*__
#内置函数
# abs:求绝对值
print (abs(-20))
s = [1,-2,98,-566,65]
print (list(abs (i) for i in s))

#enumerate：枚举(可迭代对象，序号的起始值（默认是0）)  返回的是元组
lst = [(0,1),(1,2),(2,3)]
print (enumerate(lst,10))  #返回的是一个内存地址
print ([i for i in enumerate (lst,10)])
#上面一行代码的拆开
lst = [11,22,30,40,-10,-60]
new_lst = []
for i in enumerate(lst):
	new_lst.append(i)
print (new_lst)
#max:求最大值
print (max([1,2,3,4,56,7,8]))
#min:求最小值
print (min([1,2,3,4,56,7,8]))
#sum:求和
print (sum([1,2,3,4,56,7,8]))

#print (sep="",end="\n")
print (1,2,3,sep="  ") #sep表示多个元素的连接符
print(1,2,3,end="\t")
print(2,end=" ")
print(12345,"open(文件路径)")  #这个代码就是把12345写进文件
print(dict(key=1,a="alex"))
print(dict(((1, 2), (2, 3), (3, 4))))
print(dict([i for i in enumerate(range(20),1)]))

#zip  #拉链--按照最少的进行合并   返回的是元组
lst1 = [1,2,3,4,5]
lst2 = ['a','b',52,12,'as','s']
print (zip(lst1,lst2 ))  # #返回的是一个内存地址
print (list(zip(lst1,lst2 )))
print (dict(list(zip(lst1,lst2 )))) #面试题
print (dict(zip(lst1,lst2))) #面试题

#dir 查看当前函数的方法
print (dir(zip))



# 2，format()的补充
print(format(12,">20")) #右对齐 就是在指定的空间内部右对齐，这里指定的空间就是20
print(format(12,"<20")) #左对齐
print(format(12,"^20")) #居中

print(format(13,"08b"))    # 2进制  在这里是补位的作用，这里面补了8位
print(format(13,"08d"))    # 10进制
print(format(13,"08o"))    # 8进制
print(format(12,"08x"))    # 16进制
# 00001101
# 00000013
# 00000015
# 0000000c

# 3，filter  过滤
lst = [2,3,5,6,8,9,12,1,2,36] 
def func(s):
	return s>3
print (filter(func,lst))  #返回的也是一个内存地址
print (list(filter(func,lst)))
#解释：
	# func就是自己定义的一个过滤的条件，list是要过滤的迭代对象
lst = [1,2,3,4,5,6,7]
print(list(filter(lambda x:x % 2 == 1,lst)))
#在面试的时候，filter常常和lambda匿名函数一起使用来考查

#4，map() 对象映射
lst = [1,2,3,4,5,6,7]
print (map(lambda x:x*x,lst))  #返回迭代器
print (list(map(lambda x:x*x,lst)))
#对可迭代对象中每个元素进行加工

#5，reversed() #返回迭代器
# 对比reverse
lst = [1,2,3,4,5]
lst.reverse()
print(lst)
print (id(lst),id(lst1))
#>>>1426993756040 1427106860040 可以看出这个反转是在原来的内存下反转的
lst1 = list(reversed(lst))
print(lst)
print(lst1)
print (id(lst),id(lst1))
#>>>1426993756040 1426993756232  可以卡出+d的反转另外开辟了一片新的内存空间

#6，sorted() 排序
# 同理上面的reversed也是新开辟了一片内存空间。
lst = [1,23,34,4,5,213,123,41,12,32,1]
print(sorted(lst))   # 升序
print(lst)

lst = [1,23,34,4,5,213,123,41,12,32,1]
print(sorted(lst,reverse=True))  # 降序
#对字典指定规则排序
dic = {"key":1,"key1":2,"key3":56}
print(sorted(dic,key=lambda x:dic[x],reverse=True))  # key是指定排序规则，就是一个函数，里面就是排序的规则

# 7，# reduce 累计算
from functools import reduce
print(reduce(lambda x,y:x-y,[1,2,3,4,5]))

#这些方法里面能方法函数的叫做:高阶函数
map()
filter()
sorted()
reduce()
min()
max()
```

### 闭包

一、什么是闭包：在嵌套函数内，使用非全局变量（不是自己内部的变量）就是闭包。

   ```python
   #方法一
   def func():
   	a = 1
   	def f1():
   		def foo():
   			print (a)
   		return foo
   	return f1
   ret = func() #ret=f1函数的地址
   a = ret() #a=foo函数的地址
   a() #调用foo()函数，print（a)调用func下的1，并打印1
   #也可以这样调用
   func()()()
   #方法二
   def func(a):
   	def f1():
   		def foo():
   			print (a)
   		return foo
   	return f1
   ret = func(1)
   a = ret() 
   a() 
   #传参就是相当于把a定义在函数里面
   ```

   #就像在这个代码中，foo函数调用的就是func下的a变量。这就是一个闭包。

二、 为什么使用闭包？使用闭包的好处是什么呢？

   ```python
   在问我们为什么使用闭包的时候，我们来看看这个需求：
   我们需要求一辆车本周每一天的平均单价：
   #怎么去解决这个问题呢？
   def func():
   	avg_lst = []  # 自由变量
   	def foo(pirce):
   		avg_lst.append(pirce)
   		avg = sum(avg_lst) / len(avg_lst)
   		return avg
   	return foo
   ret = func()()
   print(ret(150000))
   print(ret(160000))
   print(ret(170000))
   print(ret(150000))
   print(ret(180000))
   #这个时候，我们又可以使用了，因为我们把这个全局变量放在了函数里面，在外面我们就不能去调用这个
   #函数里面的函数了
   
   #判断是不是闭包，如果打印的是下面格式的内容就是一个闭包
   print (ret.__closure__)  
   #(<cell at 0x000002AC02885DF8: list object at 0x000002AC028DB848>,)
   ```

   闭包的作用：保证局部信息不被销毁，保证数据的安全性。

   闭包的好处：
   
   1. 可以保存一些非全局变量单数不易被销毁、改变的数据。
   2. 装饰器的本质就是一个闭包。

### 练习

```python
#作业
#2
# 1)用map来处理字符串列表,把列表中所有人都变成sb,比方alex_sb
l=[{'name':'alex'},{'name':'y'}]
s = list(map(lambda x:{"name":x["name"]+"_sb"},l))
print (list(map(lambda x:{"name":x["name"]+"_sb"},l)))
#2) 用map来处理下述l，然后用list得到一个新的列表，列表中每个人的名字都是sb结尾
l=[{'name':'alex'},{'name':'y'}]
print (list(map(lambda x:{"name":x["name"]+"sb"},l)))
# 3)用filter来处理,得到股票价格大于20的股票名字
shares={
 'IBM':36.6,
 'Lenovo':23.2,
 'oldboy':21.2,
 'ocean':10.2,
         }
print (list(filter(lambda x:shares[x]>20,shares.keys())))
# 4)有下面字典，得到购买每只股票的总价格，并放在一个迭代器中。
# 结果：list一下[9110.0, 27161.0,......]
portfolio = [
  {'name': 'IBM', 'shares': 100, 'price': 91.1},
    {'name': 'AAPL', 'shares': 50, 'price': 543.22},
    {'name': 'FB', 'shares': 200, 'price': 21.09},
    {'name': 'HPQ', 'shares': 35, 'price': 31.75},
    {'name': 'YHOO', 'shares': 45, 'price': 16.35},
{'name': 'ACME', 'shares': 75, 'price': 115.65}]
print ([x["shares"]*x["price"] for x in portfolio])
# 5)还是上面的字典，用filter过滤出单价大于100的股票。
print (list(filter(lambda x:x["price"]>100,portfolio)))
# 6)有下列三种数据类型，
l1 = [1,2,3,4,5,6]
l2 = ['oldboy','alex','wusir','太白','日天']
tu = ('**','***','****','*******')
# 写代码，最终得到的是（每个元祖第一个元素>2,第三个*至少是4个。）
# [(3, 'wusir', '****'), (4, '太白', '*******')]
print (list(zip(l1[2:],l2[2:],tu[2:])))
# 7)有如下数据类型(实战题)：
l1 = [ {'sales_volumn': 0},
   {'sales_volumn': 108},
   {'sales_volumn': 337},
   {'sales_volumn': 475},
   {'sales_volumn': 396},
   {'sales_volumn': 172},
   {'sales_volumn': 9},
   {'sales_volumn': 58},
   {'sales_volumn': 272},
   {'sales_volumn': 456},
   {'sales_volumn': 440},
   {'sales_volumn': 239}]
# 将l1按照列表中的每个字典的values大小进行排序，形成一个新的列表。
print (sorted(l1,key=lambda x:x["sales_volumn"]))
```

```python
# 3.有如下数据结构,通过过滤掉年龄大于16岁的字典
lst = [{'id':1,'name':'alex','age':18},
        {'id':1,'name':'wusir','age':17},
        {'id':1,'name':'taibai','age':16},]
print (list(filter(lambda x:x['age']>16,lst)))
```

```python
# 4.有如下列表,按照元素的长度进行升序
lst = ['天龙八部','西游记','红楼梦','三国演义']
print (sorted(lst,key=lambda x:len(x)))
```

```python
# 5.有如下数据,按照元素的年龄进行升序
lst = [{'id':1,'name':'alex','age':18},
    {'id':2,'name':'wusir','age':17},
    {'id':3,'name':'taibai','age':16},]
print (sorted(lst,key=lambda x:x['age']))
```

```python
# 6.看代码叙说,两种方式的区别

lst = [1,2,3,5,9,12,4]
lst.reverse()
print(lst)

print(list(reversed(lst)))
#这两个函数实现的都是列表元素的逆序。但是reverse反转是在原来的内存空间下进行的
#而reversed实现是开辟了一片新的内存空间。
```

```python
# 7.求结果(面试题)

v = [lambda :x for x in range(10)]  #list推导式
print(v) #>>>打印的是10个内存地址
print(v[0])##>>>打印的是1个内存地址
print(v[0]())#9

```

```python
# 8.求结果(面试题)
v = (lambda :x for x in range(10))   #迭代器生成式
print(v)#>>>一个迭代器的内存地址
# print(v[0])#>>>报错，因为迭代器不能这样取值
# print(v[0]())#>>>报错，因为迭代器不能这样取值
print(next(v))#>>>一个内存地址
print(next(v)())#>>>1  0-9的值都添加进了这个迭代器中，所以我们去取得时候，就是从第一个开始取
```

```python
# 9.map(str,[1,2,3,4,5,6,7,8,9])输出是什么? (面试题)
#>>>一个内存地址，因为map返回的一个内存地址
```

```python
# 10.有一个数组[3,4,1,2,5,6,6,5,4,3,3]请写一个函数，找出该数组中没有重复的数
# 的总和（上面数据的么有重复的总和为1+2=3)(面试题)
lst = [3,4,1,2,5,6,6,5,4,3,3]
print (sum(list(filter(lambda x:lst.count(x) == 1,lst))))
```

```python
# 11.求结果：（面试题，比较难，先做其他题）
def num():
	return [lambda x:i*x for i in range(4)]
print([m(2)for m in num()])
#>>>[6, 6, 6, 6]
```