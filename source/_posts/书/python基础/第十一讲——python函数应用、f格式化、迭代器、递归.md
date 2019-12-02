---
title: 第十一讲——python函数应用、f格式化、迭代器、递归
id: 11
date: 2019-7-30 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 函数名的第一类对象及使用
- f格式化详讲
- 迭代器 VS 可迭代对象
- 递归
- 练习

<!-----more----->

### 函数名的第一类对象及使用

**简述**

这一块的内容其实不难，但是在实际的应用中是非常的常见。在这一块，就是把函数名看做一个变量，指向的内存空间是一个内存地址的字符串。同时，函数的内存地址+()就可以执行一个函数，这一点就是我们这一个小知识点的精髓所在。

**函数名可以当作值赋给变量**

```python
#函数可以赋值
def func():
	print (123)
	def foo():
		pass
	return foo
a = func
print (a)  #打印func()函数的内存地址
>>><function func at 0x00000213072B2840>
```

**函数名当作元素存放在容器中**

```python
#函数可以存放在容器中
def func():
	pass
lst = [func,func,func]
print (lst)  #把func()函数的内存存在列表打印出来
>>>[<function func at 0x00000213072B2730>, <function func at 0x00000213072B2730>, <function func at 0x00000213072B2730>]
```

**函数名当作函数的参数**

```python
#函数名可以当作函数的参数
def func(f):
	f()
def foo():
	print (123)
func(foo)  #把foo()这个函数的地址当作func()的参数，在func()函数中，在次调用f()函数，从而打印123。
>>>123
```

**函数名可以当作函数的返回值**

```python
#函数名可以当作函数的返回值
def func():
	def foo():
		print (123)
	return foo  #返回函数foo的内存地址
foo1 = func()  #把把foo的内存地址复制给foo1
foo1()#执行foo函数，执行print(123)
>>>123
```

### f格式化详讲

字符串的格式化有三种：%(s,d......)，.format()，f三种类型。

在python3.6版本以上才能使用f。

```python
#填充字符串
s = f"你好{'world'}"      
s1 = F"你好{'world'}"

#填充变量
s = "123"
s1 = f"你还是挺好的{s}" 
print(s1)

#填充计算公式
s1 = f"{35+15}"         
print(s1)

#填充表达式
a = 10
b = 20
s1 = f"{a if a>b else b}"   #三元运算
print(s1)

#填充大括号
s1 = f"{{{{{{'美丽的中国..'}}}}}}"   #两个{}打印一个{}
print(s1)

#打印{}{}{}{}{}
s1 = f"{'{}{}{}{}{}'}"
print(s1)

s1 = f"{print(123)}" #首先执行函数print(),函数没有返回值就返回了None
print(s1)  
>>>123  None

#打印列表
lst = [1,3,4,5,6]
s1 = f"{lst[0:5]}"
print(s1)  # [1,3,4,5,6]
#打印字典的值
dic = {"key1":123,"key2":345}
s1 = f"{dic['key2']}"
print(s1)
```

### 迭代器 VS 可迭代对象

这一部分的内容因为比较重要，我单独写了一篇blog：https://blog.csdn.net/qq_40890660/article/details/97618316

**可迭代对象**

- 可迭代对象有哪些：range(20)，list，dict，tuple，str，set。

- 怎么判断一个对象的是不是可迭代对象：

   - 具有很多私有方法，查看该对象是否有.__iter__()的方法，有就是。
   - print（dir(对象)）看看里面是不是有上面的iter方法。
   - 官方说明有__iter__（）方法的对象都是可迭代对象。
   - 查看源码。

- 可迭代对象的优点：

   - 使用灵活。
   - 直接查看值。

- 可迭代对象的缺点：

   - 消耗内存。

- for的本质是什么：

   - 本质就是一个迭代器。

   - ```python
     a = "这是一个可迭代对象"
     new_a = a.__iter__() #将可迭代对象转换成一个迭代器
     while True: 
        try:  #监视程序 
           print (new_a.__next__())  #将迭代器的内容一个个的输出出来。
        except StopIteration:
           break
     ```

**迭代器**

一、怎么判断迭代器：

- 官方说明：只要具有__iter__()方法和__next__()方法的对象就是迭代器。

- ```python
  f = open(打开一个文件)
  print(f.__iter__())
  print(f.__next__())
  ```

二、迭代器的优点:

- 节省内存

- 惰性机制（要去取迭代器的值要取一下返回一个值）

三、迭代器的缺点：

- 使用的不灵活
- 操作比较繁琐
- 不能查看元素

四、迭代器的特点：

- 一次性的（用完就没有了）
- 不能逆行（只能从前往后取）
- 用一次 去取一次

  - 迭代器什么时候使用：

  当容器中数据数量较大的时候使用。像操作文件的时候，文件较大的时候，我们就去操作文件的句柄，否则还是使用可迭代对象。

  注：迭代器就是一个可迭代对象，但是可迭代对象不一定是一个迭代器。

  - 怎么取迭代器的值：

  list.__next__()  #每次只能取迭代器的值，基于上一个取值的位置去取下一个值。

### 递归

一、什么是有效递归：
1. 自己调用自己（不断调用自己本身），这个叫做死递归
2. 递归有明确的终止条件

- ```python
  #死递归
  def func():
     print (123)
     func()
  func()
  #有终止条件的递归
  def age(n): # 1,2,3
      if n == 3:
          return "猜对了"
      else:
          age(n+1)
  print(age(1))
  
  例子一：
  #该递归的执行流程：
  def age2(n):#1
  	if n == 3:#9
  		return "猜对了"#10  #这个返回给了第8步
  def age1(n):#2
  	if n == 3:#7
  		return  "猜对了"
  	else:
  		age2(n+1)#8  #这个接受到了age2的返回值，但是自己没有return，所以返回None
  def age(n): #3
  	if n == 3:#5
  		return "猜对了"
  	else:
  		age1(n+1)#6 #接收到的返回值就是None
  age(1)#4   #输出None
  例子二：
  def age(n):  #递归了3次
      if n == 4:
          return 18
      else:
          return age(n+1)-2
  print(age(1))
  >>>12
  ```

二、官方说明：最大的递归次数：1000次，但是实际中就是998,997次。

三、递归是什么?

- 说白了就是递和归
- 递：一直执行知道碰到结束条件
- 归：从结束条件开始回退

**for...else：**

在for条件满足的时候，执行for下面的代码，否则就执行else下的代码。

**while...else**

在while满足的时候，执行while下面的代码，否则就执行else下的代码。

### 练习

```python
# 1.请写出下列代码的执行结果：
# 例一：
def func1():
	print('in func1')
def func2():
	print('in func2')
ret = func1
ret()
ret1 = func2
ret1()
ret2 = ret
ret3 = ret2
ret2()
ret3()
#执行结果：in func1 in func2  in func1 in func1
```

```python
# 例二：

def func1():
	print('in func1')

def func2():
	print('in func2')

def func3(x, y):
	x()
	print('in func3')
	y()

print(111)
func3(func2, func1)
print(222)
# 执行结果：111 in func2  in func3  in func1 222
```

```python
# 例三（选做题）：

def func1():
	print('in func1')

def func2(x):
	print('in func2')
	return x

def func3(y):
	print('in func3')
	return y

ret = func2(func1)
ret()
ret2 = func3(func2)
ret3 = ret2(func1)
ret3()
# 执行结果：in func2  in func1  in func3  in func2  in func1
```

```python
# 例四:

def func(arg):
	return arg.replace('苍老师', '***')

def run():
	msg = "Alex的女朋友苍老师和大家都是好朋友"
	result = func(msg)
	print(result)

run()
data = run()
print(data)
# 看代码写结果：Alex的女朋友***和大家都是好朋友  Alex的女朋友***和大家都是好朋友  None
```

```python
# 例五:
data_list = []

def func(arg):
	print (data_list.insert(0, arg))  #为什么打印的是None，因为这个打印的不是列表，而是把值插入列表的过程
	return data_list.insert(0, arg)  #同理同上

data = func('绕不死你')
print(data)
print(data_list)
# 看代码写结果：[绕不死你]  [绕不死你]
```

```python
# 例六:

def func():
    print('你好呀')
    return '好你妹呀'

func_list = [func, func, func]

for item in func_list:
    val = item()
    print(val)
# 看代码写结果：你好呀  好你妹呀  你好呀  好你妹呀  你好呀  好你妹呀

```

```
例七:

def func():
    print('你好呀')
    return '好你妹呀'

func_list = [func, func, func]

for i in range(len(func_list)):
    val = func_list[i]()
    print(val)
# 看代码写结果：你好呀  好你妹呀  你好呀  好你妹呀  你好呀  好你妹呀
```

```python
# 例八:

def func():
    return '大烧饼'

def bar():
    return '吃煎饼'

def base(a1, a2):
    return a1() + a2()

result = base(func, bar)
print(result)
# 看代码写结果：大烧饼吃煎饼
```

```python
# 例九:

for item in range(10):
	print(item)

print(item)
# 看代码写结果：9

```

```python
# 例十:

def func():
    for item in range(10):
        pass
    print(item)
func()
# 看代码写结果：9

```

```python
# 例十一:

item = '老男孩'
def func():
    item = 'alex'
    def inner():
        print(item)
    for item in range(10):
        pass
    inner()
func()
# 看代码写结果：9
```

```python
# 例十二:

l1 = []
def func(args):
    l1.append(args)
    return l1
print(func(1))
print(func(2))
print(func(3))
# 看代码写结果：[1] [1,2] [1,2,3]
```

```python
#例十三:

name = '宝元'
def func():
    global name
    name = '男神'
print(name)
func()
print(name)
# 看代码写结果： 宝元  男神
```

```python
# 例十四:

name = '宝元'
def func():
    print(name)
func()
# 看代码写结果：宝元
```

```python
# 例十五:

name = '宝元'
def func():
    print(name)#变量放的位置错误
    name = 'alex'
func()
# 看代码写结果：UnboundLocalError: local variable 'name' referenced before assignment

```

```python
# 例十六:

def func():
    count = 1
    def inner():
        nonlocal count
        count += 1
        print(count)
    print(count)
    inner()
    print(count)
func()
# 看代码写结果：1  2  2
```

```python
# 例十七:(有点东西)

def extendList(val,list=[]):
    list.append(val)
    return list

list1 = extendList(10)
list2 = extendList(123,[])
list3 = extendList('a')

print('list1=%s'%list1)
print('list2=%s'%list2)
print('list3=%s'%list3)
# 看代码写结果：list1=[10,'a']  list2=[123] list3=[10,'a']
```

```python
# 例十八:

def extendList(val,list=[]):
    list.append(val)
    return list
print('list1=%s'% extendList(10))
print('list2=%s'% extendList(123,[]))
print('list3=%s'% extendList('a'))
#>>>[10] [123] [10,'a']

```

```python
2.用你的理解解释一下什么是可迭代对象，什么是迭代器。
可迭代对象就是有__iter__()方法的，可遍历元素的对象。
迭代器就是有__iter__()和__next__()方法，不可遍历对象的一个容器。
```

```python
3.使用while循环实现for循环的本质(面试题)
s = "可迭代对象"
s.__iter__()
while True:
	try:
		print (s.__next__())
	except StopIteration:
		break
```