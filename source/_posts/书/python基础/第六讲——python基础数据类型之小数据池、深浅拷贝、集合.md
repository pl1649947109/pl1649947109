---
title: 第六讲——python基础数据类型之小数据池、深浅拷贝、集合
id: 6
date: 2019-7-25 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 小数据池
- 深浅拷贝
- 集合
- 补充几个重要的知识点
- 练习

<!-----more----->

### 小数据池

- id：id是内存地址，我门只要创建一个数据对象那么都会在内存中开辟一个空间，将这个数据临时加载到内存中，那么这个空间是有一个唯一的标识的，那么这个标识就是我们所说的内存地址，也就是这个数据对象的id。

- == 和 is

   - ==：判断==两端的值是否相等。
   - is: 判断两边的内存地址是否相同。

- 缓存机制

   - 内容：python在执行同一个代码块的初始化对象命令时，会检查是否其值是否已经存在，如果存在，会将其重用。说的简单点，我们的代码快中的变量初始化的时候，我们都把他放到一个容器中，当我们在初始化下面的代码的时候，遇到的新的变量会去这个容器中查询，如果有一样的值，就把这两个变量都指向同一片的地址空间。

- 使用的对象

   - int、str、bool

- 代码块（一个文件，一个函数，一个类，一个模块，终端的每一行）

   - int(float):任何数字在同一代码块下都会复用。

   - bool:True和False在字典中会以1，0方式存在，并且复用。

   - str：几乎所有的字符串都会符合缓存机制

   - ```python
     def foo():
     	a = "asdfghjkuesrdfgiopgjhXTYGHijokjiohigyuty" * 20
     	b = "asdfghjkuesrdfgiopgjhXTYGHijokjiohigyuty" * 20
     	print (a is b)
     foo()
     # True
     
     def foo():
     	a = 255555555
     	b = 255555555
     	print (a is b)
     foo()
     # True
     优点：能够提高一些字符串，整数处理人物在时间和空间上的性能；需要值相同的字符串，整数的时候，直接从‘字典’中取出复用，避免频繁的创建和销毁，提升效率，节约内存。
     ```

- 小数据池：不同的代码快的缓存机制，也称作小整数缓存机制或者驻留机制等。

- 什么是小数据池？

   - 前提：在不同的代码块内
   - 机制内容：

   一、int：自动将-5 ~ 256的整数进行了缓存。

   ```python
   #小数据池
   a = 255
   b = 255
   print (id(a))
   print (id(b))
   #140735711474656
   #140735711474656
   a = 257
   b = 257
   print (id(a))
   print (id(b))
   #2835973053264
   #2835973053168
   a = -5
   b = -5
   print (id(a))
   print (id(b))
   #1453943648
   #1453943648
   a = -6
   b = -6
   print (id(a))
   print (id(b))
   #2835973053232
   #2835973053168
   ```

   二、字符串：python会将一定规则的字符串在字符串驻留池中创建一份。在同一代码块下，只要内容相同就采用相同的内容地址；乘法的时候总长度不能超过20。

   三、布尔值：在同一代码块下，只要内容相同就采用相同的内存地址。

- 解释

   - 小数据池的验证方法，必须脱离代码块才能进行验证
   - 代码块和 小数据池中，代码块的优先级高

- 总结:

   - 如果在同一块代码块下，就采用同一代码块下的缓存机制。
   - 在不同的代码块下，采用小数据池的驻留机制。


### 深浅拷贝

一、赋值（=）

赋值就是把多个变量标签同时指向一块内存地址。

二、浅拷贝

- `浅拷贝的时候，只是开辟了一个新的容器空间，其他的元素使用的都是源列表中的元素。`

- 自我理解：2变（第二层）2不变（第一层）

- 第一层元素变化其实是指向了新的内存空间地址；

- 第二层元素变化其实都是操作同一片的内存空间，所以说只要一变具变。

- 总结：2变2不变（以[1,2,3,[4,5,6]]为例）

  列表1：第1层，第2层

  1,1变-->2,1不变；1,2变-->2,2变

  列表2：第1层，第2层

  2,1变-->1,1不变；2,2变-->1,2变



- 在字典中：字典里面的内容都属于第二层的内容。所以说一变都变。

三、深拷贝

- 深拷贝并非是将所有的数据拿来放到开辟的新内存中，而是说将第二层的数据拿出来放到新的内存空间中，而第一层还放在原来的地方不动（因为第一层的数据类型本来就是不能改变的。），第二层里面的第一层数据地址不变，第二层变，以此类推。

- ```
  # 深拷贝开辟一个容器空间(列表),不可变数据公用,可变数据数据类型(再次开辟一个新的空间)
  # ,空间里的值是不可变的数据进行共用的,可变的数据类型再次开辟空间
  ```

其他的总结：

```
# 浅拷贝的时候只拷贝第一层元素
# 浅拷贝在修改第一层元素(不可变数据类型)的时候,拷贝出来的新列表不进行改变
# 浅拷贝在替换第一层元素(可变数据类型)的时候,拷贝出来的新列表不进行改变
# 浅拷贝在修改第一层元素中的元素(第二层)的时候,拷贝出来的新列表进行改变

# 深拷贝开辟一个容器空间(列表),不可变数据公用,可变数据数据类型(再次开辟一个新的空间)
# ,空间里的值是不可变的数据进行共用的,可变的数据类型再次开辟空间
```

### 集合

**集合的特点**

- 无序的，{}，它就是无值得字典的键的集合（但是内容不能是可变的数据类型，这个和字典的键是一模一样的）。

**集合的作用**

- 做大的作用就是去重（因为集合的内容是唯一的）。
- 其次，他还可以做关系测试，测试两组数据之间的交并补集等关系。

**集合的创建**

```python
set1 = set({1,2,'barry'})
set2 = {1,2,'barry'}
print(set1,set2)  
# {1, 2, 'barry'} {1, 2, 'barry'}
```

```
   
4. 函数：

   ```python
   增
   s. add(值) 增加一个值
   s.update(可迭代对象)  迭代添加
   
   删
   s.pop(集合中某一个元素)随机删除
   s.remove(字典中一个元素) 指定元素删除
   s.clear()清空
   del  s 删除集合
   
   改
   先删再加
   
   查
   For i in s:
   	Print (i)
```

**功能测试**

```python
# 其他操作:
# s1 = {1,2,3,4,5,6,7}
# s2 = {5,6,7,1}
# print(s1 & s2)  # 交集
# print(s1 | s2)  # 并集
# print(s1 - s2)  # 差集
# print(s1 ^ s2)  # 反交集
# print(s1 > s2)  # 父集(超集)
# print(s1 < s2)  # 子集

# print(frozenset(s1))  # 冻结集合 （让集合变成不可变的数据类型）
# dic = {frozenset(s1):1}
# print(dic)
```

### 补充几个重要的知识点

**for循环**

```python
#迭代字符串
msg = 'god is a gril'
for item in msg:
    print(item)
g
o
d
 
i
s
 
a
 
g
r
i
l

#迭代列表
li = ['pl1','pl2','pl3','pl4','pl5']
for i in li:
    print(i)
pl1
pl2
pl3
pl4

#迭代字典
dic = {'name':'pl','age':18,'sex':'man'}
for k,v in dic.items():
    print(k,v)
name pl
age 18
sex man
```

**enumerate**

枚举：对于一个可迭代的（iterable）/可遍历的对象（如列表、字符串），enumerate将其组成一个索引序列，利用它可以同时获得索引和值。

```python
li = ['pl1','pl2','pl3','pl4','pl5']
for i in enumerate(li):
    print(i)
(0, 'pl1')
(1, 'pl2')
(2, 'pl3')
(3, 'pl4')
(4, 'pl5')

for index,name in enumerate(li,1):
    print(index,name)
1 pl1
2 pl2
3 pl3
4 pl4
5 pl5

for index, name in enumerate(li, 100):  # 起始位置默认是0，可更改
    print(index, name)
100 pl1
101 pl2
102 pl3
103 pl4
104 pl5
```

### 练习

```python
#1.请用代码验证 "alex" 是否在字典的值中？
info = {'name':'王刚蛋','hobby':'铁锤','age':'18'}
if "alex" in list(info.values()):
	print ("alex在字典得值中")
else:
	print ("alex不在字典的值中")
```

```python
#2.有如下
v1 = {'郭宝元','李杰','太白','梦鸽'}
v2 = {'李杰','景女神'}
#请得到 v1 和 v2 的交集并输出
print (v1 & v2)
#请得到 v1 和 v2 的并集并输出
print (v1 | v2)
#请得到 v1 和 v2 的 差集并输出
print (v1 - v2)
#请得到 v2 和 v1 的 差集并输出
print (v2 - v1)
```

```python
#3.循环提示用户输入，并将输入内容追加到列表中（如果输入N或n则停止循环）
while True:
	lst = []
	luru = "".join(input("请输入内容：").split())
	if luru in ("N","n"):
		break
	else:
		lst.append(luru)
```

```python
#4.写代码实现循环提示用户输入，如果输入值在v1中存在，
# 则追加到v2中，如果v1中不存在，则添加到v1中。（如果输入N或n则停止循环）
v1 = {'alex','武sir','黑哥'}
v2 = []
while True:
	luru = input("请输入内容：")
	if luru in ("N","n"):
		break
	else:
		if luru in v1:
			v2.append(luru)
		else:
			v1.add(luru)
print (v1,v2)
```

```python
#5.判断以下值那个能做字典的key ？那个能做集合的元素？
'''
1
-1
""
None
[1,2]
(1,)
{11,22,33,4}
{'name':'wupeiq','age':18}
'''
#只有1,-1,(1,)可以做字典的key，又可以做集合的元素。
```

```python
#6.is 和 == 的区别？
#is :判断的它左右两边的内存地址
#== ：判断它左右两边的值是否相等
```

```python
#7.type使用方式及作用？
#用法：type(变量)
#作用：用来判断变量的类型
```

```python
#8.id的使用方式及作用？
#用法：id(变量)
#作用：用量查看变量指向数据的内存地址
```

```python
#9.看代码写结果并解释原因
v1 = {'k1':'v1','k2':[1,2,3]}
v2 = {'k1':'v1','k2':[1,2,3]}
result1 = v1 == v2
result2 = v1 is v2
print(result1)
print(result2)
#解释：因为v1和v2的内容是一样的，所有在result1 = v1 == v2比较的是v1和v2的内容，故返回True
#但是他们两个定义的时候使用了两个变量，所以在内存中开辟了两块控空间，故内存地址不一样，所以在
#result2 = v1 is v2比较中返回的是False
```

```python
#10.看代码写结果并解释原因
v1 = {'k1':'v1','k2':[1,2,3]}
v2 = v1
result1 = v1 == v2
result2 = v1 is v2
print(result1)
print(result2)
#解释：因为v2是v1赋值过来的，所以这两个变量指向的内存地址是一样的，故他们不论是
#比较内存地址还是比较内容都应该返回True

```

```python
#11.看代码写结果并解释原因
v1 = {'k1':'v1','k2':[1,2,3]}
v2 = v1
v1['k1'] = 'wupeiqi'
print(v2)
#解释：因为v2是v1赋值来的，故他们指向同一内存地址，所以改变v1的数据返回v2时，数据一样跟着改变了
```

```python
#12.看代码写结果并解释原因
v1 = '人生苦短，我用Python'
print (id(v1))
v2 = [1,2,3,4,v1]
print (id(v2[-1]))
v1 = "人生苦短，用毛线Python"
print (id(v1))
print(v2)
#解释：列表里面存的本质上及时内存地址，在第二句代码中，把v1变量指向的内存地址写了进去，
#第三句的v1变量重新赋值，其实就是内外开辟了一片内存空间。
```

```python
#13.看代码写结果并解释原因
info = [1,2,3]
userinfo = {'account':info, 'num':info, 'money':info}
info.append(9)
print(userinfo)
info = "题怎么这么多"
print(userinfo)
#解释：本质上和上一题的解释是一样的。
```

```python
#14.看代码写结果并解释原因
info = [1,2,3]
print (id(info))
userinfo = [info,info,info,info,info]
print (id(userinfo[0]))
info[0] = '不仅多，还特么难呢'
print(info,userinfo)
print (id(info))
print (id(userinfo[0]))
#解释：因为userinfo里面的每一个元素都指向info还行的内存地址，所以改变info的内容其实也是修改
#userinfo的内容
```

```python
#15.看代码写结果并解释原因
info = [1,2,3]
print(id(info),id(info[-1]))
userinfo = [info,info,info,info,info]
print (id(userinfo[0][-1]))
userinfo[2][0] = '闭嘴'
print (id(userinfo))
print(info,userinfo)
print (id(userinfo[0][-1]))
print(id(info),id(info[-1]))
#解释：username里面的每一个元素都共享同一个内存，就是info指向的内存地址，所以改变一个
#元素变相的把所有的内容都改变了.
```

```python
#16.看代码写结果并解释原因
info = [1,2,3]
user_list = []
for item in range(10):
	user_list.append(info)
	info[1] = "是谁说Python好学的？"
	print(user_list)
```

```python
#17.看代码写结果并解释原因
data = {}
for i in range(10):
	data['user'] = i
print(data)
#解释：应为data是一个字典，它的一个特性就是：键不能重复，尽管循环了10次，
#也只添加了1个键值对(最后循环的一对)
```

```python
#18.看代码写结果并解释原因（有疑问）
data_list = []
data = {}
for i in range(10):
	data['user'] = i  
	data_list.append(data)
print(data,data_list)
#解释：#解释：因为内存里面存的是地址，所以循环10次就是将‘user’的内存地址存了10次
#最后一次循环的时候，'user'的值就是9，所以这10次指向的内存地址的值都是9.

```

```python
#19.看代码写结果并解释原因
data_list = []
for i in range(10):
	data = {}   #这个就控制了上一题的每次循环完的情况
	data['user'] = i
	data_list.append(data)
print(data_list)
#解释：把每次循环清空字典，每次data字典只接收一个键值对，然后把键值对添加到列表中。
```

```python
#20.使用循环打印出一下效果：
'''
* 
** 
*** 
**** 
***** 
'''
for i in range(5):
	print ("*" * (i+1))
'''
**** 
***
** 
*
'''
for i in range(4,0,-1):
	print (("*" * i))
'''
* 
*** 
***** 
******* 
*********
'''
for i in range(1,10,2):
	print ("*" * i)

'''
```

```python
21.敲七游戏. 从1开始数数. 遇到7或者7的倍数（不包含17,27,这种数）
要在桌上敲⼀下. 编程来完成敲七. 给出⼀个任意的数字n. 从1开始数. 
数到n结束. 把每个数字都放在列表中, 在数的过程中出现7或 者7的倍数
（不包含17,27,这种数）.则向列表中添加⼀个'咣'
例如, 输⼊10. lst = [1, 2, 3, 4, 5, 6, '咣', 8, 9, 10]
'''
lst = []
while True:
	n = input("请输入结束数字：")
	if n.isdecimal() !=  True:
		print ("请输入数字。")
	else:
		for i in range(1,int(n)+1,1):
			if i == 7 or i%7 == 0:
				lst.append("咣")
			else:
				lst.append(i)
		break
print(lst)
```