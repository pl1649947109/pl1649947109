---
title: 第十讲——python函数之动态参数、命名空间、嵌套、关键字
id: 10
date: 2019-7-29 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 函数参数之动态参数
- 函数的注释
- 函数的名称空间
- 函数的嵌套
- global和nonlocal
- 练习

<!-----more----->

### 函数参数之动态参数

**简述**

1. 简述：上一节中我们说过传参，如果我们在传参的时候不清楚有哪些的时候，或者说给一个函数传了很多参数，我们就需要在函数定义的时候定义很多的参数，这样是非常麻烦的，因此，我们想要一个东西代替这一坨的参数，那么，这个东西就是动态参数。说白了就是我不管你这个函数会不会用到我们的参数，我只要把他们全部塞给你，用到了你就去拿，不用你就把他们搁在那。
2. 回顾一下我们之前所学的形参：
   - 位置参数：优先级最高func(a,b,c)。
   - 默认参数：优先级比位置参数优先级低func(a,b,c,d=3)。

**动态接收位置参数*args**

- 在参数位置用*表示接受任何参数（接收到的参数把他们封装在一个元组里面）


```python
def eat(*args):
    print('我想吃',args)
eat('大米饭','中米饭','小米饭')  # 收到的结果是一个tuple元祖
#当然，实参可以是多个列表，数字，元组，字典，集合等任意的数据类型
```

- 优先级


形参的顺序：位置参数>动态参数>默认参数

**动态接收关键字参数**

- 我们使用**来接收动态的关键字参数（把接收到的参数按照字典的键值对的形式封装在一个字典里面）


```python
def func(**kwargs):
    print(kwargs)     
func(a=1, b=2, c=3)
结果:
{'a': 1, 'b': 2, 'c': 3}
```

- 优先级


形参的顺序：位置参数>动态位置参数>默认值参数>动态默认值参数（也是关键字参数）

- 万能参数（*args,**kwargs）：可以接收所有的参数


```python
def func(*args,**kwargs):
    print(args,kwargs)
func(1,23,5,[1,2,3],(1,2,3),a=1,b=6)
```

- 传参的两种特殊形式：


```python
def func(*args):
    print(args)
func(*[1,2,3,4,5,6]，*[4,5,6,7,8])  
#这种情况就是在实参里面先将可迭代对象打散，就可以实现把可迭代对象的每一个元素传进args里面进行封装成一个字典
print(*[1,2,3,4,5,6])
>>>1，2，3，4，5，6  这种情况不会报错，可以打散
```

```python
dic = {'a':1,'b':2}
dicc1 = {'c':2,"d":3}
def func(**kwargs):
    print(kwargs)
func(**dic,**dic)
#这种情况和上面一样，不过对象不再是可迭代对象，而是字典，将里面的对象打散全部传递到kwargs里面封装成一个字典
print (**dic)  错
#这种情况是错误的，字典不能打散输出，只能作为实参的时候才能进行打散传参，不能像*一样
注：字典的键不能是int类型的，只能是字符串类型的。

*还可以这样灵活应用
a,b = (1,2)
print(a, b) # 1 2
# 其实还可以这么用：
a,*b = (1, 2, 3, 4,)
print(a, b)
# 1 [2, 3, 4]
*rest,a,b = range(5)
print(rest, a, b) 
# [0, 1, 2] 3 4
print([1, 2, *[3, 4, 5]]) 
# [1, 2, 3, 4, 5]
记住：*返回的一定是一个列表。
```

- 实例代码：


```python
def func(a,b,*args): #*args是万能的参数，能接受任意多个的位置参数。*叫做聚合，就是把任意多个元素封装到一个元组里面
	print (a,b,args)
func(1,2,3,4,5,6)

def func1(a,b,*args):
	print (a,b,*args)  #打散，就是将聚合之后的元组一个个的取出来
func1(1,2,3,4,5,6)

def func2(*args,a=1,b=2):#动态位置参数的优先级要比默认参数的优先级高
	print (args,a,b)
func2(1,2,3,4,5,6)

def func3(a,b,**kwargs):  #**kwargs接收关键字的参数
	print (a,b,kwargs)
func3(1,2,c=1,d=2)

def func4(a,b,*args,c=2,d=3,**kwargs): #优先级：位置参数>动态位置参数>默认参数>动态关键字参数
	print (a,b,*args,c,d,*kwargs,kwargs) #*kwargs获取字典的键
func4(1,2,c=1,d=2,e="sdd",f=12)

def func6(*args,**kwargs):  #万能组合，可以接受任意的位置参数和关键字参数。
	print (args,*args)
	print (kwargs,*kwargs)
func6(1,2,3,4,5,6,a=12,b=23,c=33)
```

**补充一：形参的第五种参数：仅限关键字参数**

```python
仅限关键字参数是python3之后更新的特性，他的位置放在*args后面，**kwargs前面，也就是默认参数的位置，它和默认参数的位置谁先谁后无所谓。
# 这样传参是错误的，因为仅限关键字参数c只接受关键字参数
def func(a,b,*args,c):
print(a,b) 
# 1 2
print(args) 
# (4, 5)
# func(1, 2, 3, 4, 5)
# 这样就正确了：
def func(a,b,*args,c):
print(a,b) 
# 1 2
print(args) 
# (3, 4)
print(5)
func(1, 2, 3, 4, c=5)
#这个仅限关键字参数从名字我们就可以看出他只能通过关键字参数传参，其实可以把他当成不设置默认参数而且必须要的参数，不传就会报错
所以形参角度的所有形参的最终顺序为：**位置参数，*args，默认参数，仅限关键字参数，**kwargs。
```

**补充二：命名关键字参数**

```python
#命名关键字参数在关键字参数的基础上限制传入的的关键字的变量名,和普通关键字参数不同，命名关键字参数需要一个用来区分的分隔符*，它后面的参数被认为是命名关键字参数。

#这里星号分割符后面的city、job是命名关键字参数
person_info(name, age, *, city, job):
    print(name, age, city, job)

#正确传参
person_info("Alex", 17, city = "Beijing", job = "Engineer")
Alex 17 Beijing Engineer    #看来这里不再被自动组装为字典

不过也有例外，如果参数中已经有一个***可变参数***的话，前面讲的星号分割符就不要写了（其实星号是写给Python解释器看的，如果一个星号也没有的话就无法区分命名关键字参数和位置参数了，而如果有一个星号即使来自变长参数就可以区分开来）

#args是变长参数，而city和job是命名关键字参数
person_info(name, age, *args, city, job):
    print(name, age, args, city)

person_info("Liqiang", 43, "balabala", city = "Wuhan", job = "Coder")
Liqiang 43 balabala Wuhan Coder
 
```

### 函数的注释

```python
def func5(a,b,**kwargs):
    #在函数的下面打上3对引号自动生成了以下的内容
	"""   
	功能说明
	:param a:干啥的 str
	:param b:干啥的 int
	:param kwargs:干啥的 {}
	:return:干啥的 bool
	"""
print (func5.__doc__)  #查看这个函数的说明
```

### 函数的名称空间

**简述**

在python解释器开始执行之后，就会在内存中开辟一个空间，每当遇到一个变量的时候，就把变量名和值之间的关系记录下来；但是，当遇到函数定义的时候，解释器只是把函数名读入内存，表示这个函数存在了，至于函数内部的变量和逻辑，解释器是不关心的。简单的说一开始的时候函数只是加载进去，仅此而已，只有当函数被调用和访问的时候，解释器才会根据函数内部声明的变量来进行开辟变量的内部空间。随着函数的执行完毕，这些函数所占用的空间也会随着函数执行而被销毁。

我们给存放名字和值的关系的空间起一个名字就叫做：命名空间，我们的变量在储存的时候就储存哎这片空间中。

**名称空间**

一、名称空间的分类
- 全局命名空间：就是我们直接在py中，函数外声明的变量。
- 局部命名空间：在函数中声明的变量。
- 内置命名空间：就是解释器为我们提供的名字，如len(),print,input等都是的。

二、加载的顺序
1. 首先加载内置名称空间。
2. 再加载全局名称空间。
3. 最后加载局部名称空间（函数被执行才会加载）。

三、取值的顺序
1. 局部。
2. 全局。
3. 内置。

四、作用域
- 全局作用域：全局命名空间+内置命名空间

- 局部作用域：局部命名空间。

- 内置函数：globals()和locals()

   - globals(): 以字典的形式返回全局作用域所有的变量对应关系

   - locals(): 以字典的形式返回当前作用域的变量的对应关系。

     ```python
     # 在全局作用域下打印，则他们获取的都是全局作用域的所有的内容。
     a = 2
     b = 3
     print(globals())
     print(locals())
     '''
     {'__name__': '__main__', '__doc__': None, '__package__': None,
     '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x000001806E50C0B8>, 
     '__spec__': None, '__annotations__': {},
     '__builtins__': <module 'builtins' (built-in)>}
     {'__file__': 'D:/lnh.python/py project/teaching_show/day09~day15/function.py',
     '__cached__': None, 'a': 2, 'b': 3}
     '''
     ```

### 函数的嵌套(高阶函数)

说明：

- 不管再什么位置，只要是函数名 + ()就是调用此函数。
- 函数定义的时候，开辟一片空间，在函数执行完之后，函数体开辟的空间就自动销毁了。
- 同级函数之间不能相互共享变量（私有变量神圣不可侵犯）。

实例说明

```python
def func():  #1
	print (1)#3
	def f1():#4
		print (2)#6
	return f1()#5
func()#2
#>>>1，2
上面的代码的执行流：
1：在全局空间中开辟一片空间给func。
2：调用func函数。
3：打印1
4:在func的局部空间中再开辟一片空间给f1.
5:func返回值，调用f1，但是f1没有return，所以得到None，也就是func()就是None。
6:打印2
```

```python
def a():#1
	a = 1#7
	c()#8
	print (c)
def b():#2
	b = 2
	print (b)
def c():#3
	c = 3#9（结束）
	print (a)
def run():#4
	a()#6
run()#5
>>>最后打印的是a函数的地址。在本函数里面没有变量，当然不可能去其他的函数里面找啊，这个时候它就去想去全局空间找，诶，这个时候它在全局空间下发现了a变量，但是a连接的是一个函数（一个地址），它也不管三七二十一拿到就返回打印了。
```

```python
def func():#1
	a = 1#13
	def b():#14
		print (a)
def foo():#2
	b = 1#6
	def z():#7
		print (func)#9
		print (b)#10
	ret = z()#8 11
	func()#12
	return ret#15
def run():#3
	foo()#5 16
print(run())#4 17
>>>fun的地址,1,None
```

### global和nonlocal

解释：
- global：

  - 声明一个全局变量
  - 在局部作用域想要对全局作用域的全部变量进行修改时，需要使用global（仅限于字符串，数字）。

- nonlocal：

  - 不能更改全局变量

  - 局部修改上一层的变量，如果上一层函数中找不到的话，就再往上找，直到找到最外层，如果没有找到就报错。

代码实例：

```python
global(在局部空间修改全局变量的值) 和 nonlocal（在局部变量修改局部变量的值，往上一层找，直到找到最外层函数，没有就报错）
a = 1
def foo():
	global a
	a = a+1
	print (a)
print (foo(),a)

a = 10
def f1():
	a = 10
	def f2():
		a = 15
		def f3():
			global a
			a += 1
			print (a) #11
		print (a) #15
		f3()
	print (a)#10
	f2()
f1()

a = 10
def func():
	def f1():
		a = 10
		def foo():
			nonlocal a
			a += 1
			print (a)#11
		foo()
		print (a)#11
	f1()

func()
print (a)
```

### 练习

```python
# 2.写函数，接收n个数字，求这些参数数字的和。（动态传参）
def num_sum(*args):
	count = 0
	for i in args:
		count += int(i)
	return count
print (num_sum(1,2,3,4,5,6,7,8,9))
```

```python
#3.读代码，回答：代码中,打印出来的值a,b,c分别是什么？为什么？
a=10
b=20
def test5(a,b):
	print(a,b)
c = test5(b,a)
print(c)
#>>>a = 20 ;b = 10 ;c = None
#因为a，b是为位置传值，所以test5接收的是20和10，print（c）打印的是test5的返回值，但是
#它没有return，索引返回的是None
```

```python
#4.读代码，回答：代码中,打印出来的值a,b,c分别是什么？为什么？
a=10
b=20
def test5(a,b):
   a=3
   b=5
   print(a,b)
c = test5(b,a)
print(c)
#>>>a = 3;b = 5;None
#解释：因为在test5局部变量里重新定义了a和b，所以在函数里面寻找值得时候，先找局部空间的。故3和5
#c同理
```

```python
#5.传入函数中多个列表和字典,如何将每个列表的每个元素依次添加到函数的动
# 态参数args里面？如何将每个字典的所有键值对依次添加到kwargs里面？
def foo(*args,**kwargs):
	print (args,kwargs)
foo(*[1,2,3,4,5,6],*[7,8,9],**{'1':2,'2':3,'3':4},**{'4':5,'5':6})
#注意，字典的键不能是数字，只能是String类型
```

```python
# 6.下面代码成立么?如果不成立为什么报错?怎么解决?
# 6.1
a = 2
def wrapper():
	print(a)
wrapper()
#不会报错
#6.2
a = 2
def wrapper():
	#加global
	global a
	a += 1
print(a)
wrapper()
#会报错，因为作用域的不同，局部空间不能修改全局变量
#6.3
def wrapper():
	a = 1
def inner():
	print(a)
	inner()
wrapper()
#不错，但是没意义啊
#6.4
def wrapper():
	a = 1
	def inner():
		#加一个nonlocal
		nonlocal a
		a += 1
		print(a)
	inner()
wrapper()
#错了，局部变量修改其父函数的变量是需要声明的。
```

```python
#7.写函数,接收两个列表,将列表长度比较小的列表返回.
def judge_littliLise(list1,list2):
	if len(list1) > len(list2):
		return list2
	elif len(list1) < len(list2):
		return list1
	else:
		return list1,list2
print (judge_littliLise([1,2,3],[1,2,5]))

```

```python
#8.写函数,接收一个参数(此参数类型必须是可迭代对象),
# 将可迭代对象的每个元素以’_’相连接,形成新的字符串,并返回.
#例如 传入的可迭代对象为[1,'老男孩','宝元']返回的结果为’1_老男孩_宝元’
def foo(*args):
	zifu = ""
	for i in args[0]:
		zifu += str("_"+str(i))
	new_zifu = zifu[1:]
	print (new_zifu)
foo('asddsfhj6232fdbsva')
```

```python
#9.有如下函数:你可以任意添加代码,执行inner函数.
def wrapper():
	def inner():
		print(666)
	#添加
	inner()
wrapper()
```

```python
#10.补充代码,可以使以下的代码可以运行
a = 10
def func():
	global a
	a += 1
	print(a)
func()
```

