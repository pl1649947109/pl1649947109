---
title: 第九讲——python函数初始
id: 9
date: 2019-7-28 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 函数初识
- 函数的参数
- 练习

<!-----more----->

### 函数初识

一、首先看一个需求

- 我们目前为止,已经可以完成一些软件的基本功能了,现在我们自己来实现一个len(),但是不能使用len()。


   ```python
   a = "alexdsb"
   count = 0
   for i in a:
       count += 1
   print(count)
   ```

   - 我们现在实现了一个求长度,我还想让你们求一下列表和元组的长度 是不是就要将我们写的内容再次拿过来。


   - 我们在求一个字典的长度,也需要将我们写好的内容拿过来使用 好像咱们这个程序中好多都是一样的啊,我们能不能把这些代码封装起来,用的时候拿过来就用啊。
   - 上面的一步步的需求就把我们函数引出来了，它将我们同一类需求的实现封装了起来，这样就减少了代码的重复性，使代码的可读性变的更好。

二、函数的结构

- 函数的结构

```python
def 函数名():

    函数体
```

- 接下来，我们来看一下函数的定义在内存中的空间中发生了什么：


   ![image-20190625111259461](http://39.97.99.7/python/assets/image-20190625111259461.png)

   就是这样，内存开辟了一块空间，但是里面存放的是代码。这样我们就将咱们写的代码封装起来了。

三、函数的调用

```python
def len():
  a = "alexdsb"
  count = 0
  for i in a:
      count += 1
  print(count)

len()  # 函数的调用
```

我们在函数调用执行的时候，才会执行func这个空间里面的代码，执行的时候开辟空间，这次 是在func里面开辟的空间。![image-20190625111755578](http://39.97.99.7/python/assets/image-20190625111755578.png)

当我们执行完函数后，函数里开辟的空间就销毁了，我们如果想用函数里的值就需要从函数中传递出来，这就是我们接下来说到的返回值。

四、函数的返回值

- 一个函数就是封装一个功能，这个功能一般都会有一个最终的结果，得到这个结果我们就能做很多的判断，或者获取自己需要的信息。

```python
def yue():
    print("约你")
    print("约我")
    print("约他")
    return   #这就是返回值，但是返回的是空
yue()
```

- 返回的结果是None，我们有实时候操作列表的方法打印的结果就是None，我们所有用字符串，列表的方法都是函数。


   - 函数中遇到return，此函数就立马结束了，不再继续执行。


   - 函数的返回值可以是多个结果：


   ```python
   def yue():   
       print("约你")   
       print("约我")   
       print("约他")   
       return "美女一枚", "萝莉一枚"
   girl = yue()
   print(type(girl))   # tuple
   ```

   注：当函数返回多个结果的时候，返回的是一个元组。

五、总结

1.遇到return,此函数结束,return下面的代码将不会在执行。

2.return 的返回值

- 如果return什么都不写或者干脆就没写return,返回的结果就是None。

- 如果return后面写了一个值,返回给调用者这个值。

- 如果return后面写了多个结果,返回给调用者一个tuple(元祖),调用者可以直接使用解构获取多个变量。

### 函数的参数

**函数的参数**

一、简述

参数，就是函数括号里面的类容，函数在调用的时候指定一具体的变量的值就是参数。这个参数叫做形参，如果我们在定义函数的时候写了形参，在调用的时候没有传递值，调用的时候右边括号会发黄，所以我们必须要传递参数，参数要一一对应，不能多也不能少。

二、参数

- 形参

  写在函数声明的位置的变量叫做形参，形式上的一个完整，表示这个函数需要***

- 实参

  在函数调用的时候给函数传递的值，加实参，实际执行的识货给韩式传递的信息表示给函数***

- 传参

  从调用函数的时候将值传递到定义函数的过程叫做传参。

**实参角度分析**

*位置参数*

```python
def date(sex, age, hobby):
    print('设置筛选条件：性别: %s，年龄：%s,爱好：%s' %(sex, age, hobby))
    
date('女','25~30','唱歌')  #这就是实参，位置和形参是一一对应的。
```

*关键字参数*

```python
def date(sex, age, hobby):
    print('设置筛选条件：性别: %s，年龄：%s,爱好：%s' %(sex, age, hobby))
    
date(hobby='唱歌',sex='女',age='25~30',)  #实参的关键字参数，对应的也是形参里面的变量。
```

*混合参数*

```python
def date(sex, age, hobby):

    print('设置筛选条件：性别: %s，年龄：%s,爱好：%s' %(sex, age, hobby))

date('女',hobby='唱歌',age='25~30',) #混合参数，位置参数必须在前。
```

**形参角度分析**

*位置参数*

```python
def date(sex, age, hobby): #形参的位置参数，和实参是一一对应的。
    print('设置筛选条件：性别: %s，年龄：%s,爱好：%s' %(sex, age, hobby))
    
date('女','25~30','唱歌')  
```

*默认参数*

```python
def date(age, hobby，sex="女"): #sex就是默认参数
    print('设置筛选条件：性别: %s，年龄：%s,爱好：%s' %(sex, age, hobby))
    
date('25~30','唱歌'，'男')  
date('25~30','唱歌')  
我们下面传参的时候可以传，也可以不传，不传的话，在函数内部的sex调用就自动是女；我们传‘男’的时候，他就是把sex的默认值替换掉。
```

**总结**

综上: 在实参的⾓角度来看. 分为三种:

1. 位置参数。
2. 关键字参数。
3. 混合参数,  位置参数必须在关键字参数前面。

注意:必须先声明在位置参数,才能声明关键字参数

 综上:在形参的角度来看

1. 位置参数。
2. 默认值参数(大多数传进来的参数都是一样的, 一般用默认参数，不然设设置默认参数干啥）。

### 练习

```
# 2.写函返回给数，检查获取传入列表或元组对象的所有奇数位索引对应的元素，
# 并将其作为新列表调用者。
# def foo(reg):
# 	list_new = []
# 	for i in range(len(reg)):
# 		if i % 2 == 0:
# 			list_new.append(reg[i])
# 	return list_new
# print (foo([1,2,3,4,5,6,7,8,9]))

```

```python
# 3.写函数，判断用户传入的对象（字符串、列表、元组）长度是否大于5。
# def foo(reg):
# 	if reg.__len__() > 5:
# 		return  True
# 	else:
# 		return  False
# print (foo([1,2,35,2]))
```

```python
# 4.写函数，检查传入列表的长度，如果大于2，
# 那么仅保留前两个长度的内容，并将新内容返回给调用者。

#方法一(这种是把列表里面的内容删除)
# def foo(reg):
# 	while len(reg) > 2:
# 		reg.pop()
# 	return reg
# print (foo([1]))

#方法二（这种方法是直接取得内容）
def foo (reg):
	if len(reg) > 2:
		return reg[:2]
	else:
		return reg
print (foo([1]))

```

```python
# 5.写函数，计算传入函数的字符串中,
# [数字]、[字母] 以及 [其他]的个数，并返回结果。
# def foo(reg):
# 	strCount = 0
# 	numCount = 0
# 	otherCount = 0
# 	for i in reg:
# 		if i.isalpha():
# 			strCount += 1
# 		elif i.isdecimal():
# 			numCount += 1
# 		else:
# 			otherCount += 1
# 	return "字母："+str(strCount),"数字："+str(numCount),"其他："+str(otherCount)
# print(foo('123ds54fsdg1gf2h[][opo'))
```

```python
# 6.写函数，接收两个数字参数，返回比较大的那个数字。
# def foo(num1,num2):
# 	if num1 > num2:
# 		return num1
# 	else:
# 		return num2
# print (foo(2,2))
```

```python
# 7.写函数，检查传入字典的每一个value的长度,如果大于2，
# 那么仅保留前两个长度的内容，并将新内容返回给调用者。
dic = {"k1": "v1v1sdfg", "k2": [11,22,33,44]}
# PS:字典中的value只能是字符串或列表
# def foo(reg):
# 	for k,v in reg.items():
# 		if type(v) == str:
# 			while len(v) > 2:
# 				lst = list(v)
# 				lst.pop()
# 				v = "".join(lst)
# 			reg[k] = v
# 		else:
# 			while len(v) > 2:
# 				v.pop()
# 			reg[k] = v
# 	return  reg
#
# print (foo(dic))
```

```python
# 8.写函数，此函数只接收一个参数这个参数必须是列表数据类型，
# 此函数完成的功能是返回给调用者一个字典，此字典的键值对为
# 列表的索引及对应的元素。例如传入的列表为：[11,22,33] 返
# 回的字典为 {0:11,1:22,2:33}。
# def foo(reg):
# 	dic = {}
# 	for i in range(len(reg)):
# 		dic[i] = reg[i]
# 	return dic
#
# print (foo([1,2,3,4,5,6,"asd","s"]))

#方法二
def foo(reg):
	dic = {}
#enmuerate(iterable[,start]):iterable是可迭代对象，start是指定开始的枚举值，默认从0开始
	for k,v in enumerate(reg):  #枚举，默认从0开始
		dic[k] = v
	return dic

print (foo([1,2,3,4,5,6,"asd","s"]))
```

```python
# 9.写函数，函数接收四个参数分别是：姓名，性别，年龄，学历。
# 用户通过输入这四个内容，然后将这四个内容传入到函数中，此函
# 数接收到这四个内容，将内容追加到一个student_msg文件中


# def foo(name,sex,age,educational):
# 	f = open("./student_msg",'a',encoding="utf-8")
# 	f.write(str(name)+" "+str(sex)+" "+str(age)+" "+str(educational)+"\n")
# 	f.close()
# 	return True
# print (foo("pl","boy",20,"本科"))
```

```python
# 10.对第9题升级：支持用户持续输入，Q或者q退出，
# 性别默认为男，如果遇到女学生，则把性别输入女。

def foo(name,age,educational,sex="男"):
	f = open("./student_msg",'a',encoding="utf-8")
	f.write(str(name)+" "+str(sex)+" "+str(age)+" "+str(educational)+"\n")
	f.close()
	return True
while True:
	name = input("name（Q/q）:")
	if name.upper() == "Q" :
		break
	sex = input("sex（Q/q）:")
	if sex.upper() == "Q":
		break
	age = input("age（Q/q）:")
	if age.upper() == "Q":
		break
	educational = input("educattional（Q/q）:")
	if educational.upper() == "Q":
		break
	else:
		if sex == "":
			foo(name=name,age=age,educational=educational)
			print ("信息录入成功。")
		else:
			foo(name=name,age=age,educational=educational,sex=sex)
			print("信息录入成功。")
```