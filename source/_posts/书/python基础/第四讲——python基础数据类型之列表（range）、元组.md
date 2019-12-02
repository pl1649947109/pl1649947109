---
title: 第四讲——python基础数据类型之列表（range）、元组
id: 4
date: 2019-7-23 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 列表
- 元组
- range
- 练习

<!-----more----->

#### 列表

- 列表是python的基础数据类型之一，其他的编程语言也有类似的数据类型，比如c的数组。

- 列表的特点：有序的，可变的，支持索引，基本支持所有的数据类型，相比较字符串可以储存更多的数据(2**计算机位数)。

- 操作符：[],[:],[::]in,not in,+,*。

- 列表的创建方法

   ```python
   #方式一：手动生成（推荐使用）
   a = ["pl",12,True,{"pl":18},{1,2,3},(1,2,3)]
   
   #方式二：迭代生成
   a = list("god is a gril")
   print (a)
   #>>>['g', 'o', 'd', ' ', 'i', 's', ' ', 'a', ' ', 'g', 'r', 'i', 'l']
   
   #方式三：列表推导式，后面会讲到
   print ([i for i in range(10)])  
   #>>>[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
   ```

- 列表的切片

   ```python
   l1 = ['a', 'b', 'pl', 3, 666]
   print(l1[0])  # 'a'
   print(l1[-1])  # 666
   print(l1[1:3])  # ['b', 'pl']
   print(l1[:-1])  # ['a', 'b', 'pl', 3]
   print(l1[::2])  # ['a', 'pl', 666]
   print(l1[::-1])  # [666, 3, 'pl', 'b', 'a']
   ```

- 常用方法

   - 增加

     `append():追加元素带列表的末尾。`

     `insert()：在列表指定的位置插入元素。`

     `extend(可迭代对象)：迭代添加到列表的尾部。`

     `注：一般情况下最好不要用insert，因为当数据量比较大的时候，插入值后面的元素后移就会影响效率的。`

   - 删除

     `pop():通过索引删除列表中的元素，默认情况下删除最后以一个元素，返回值为删除的元素。`

     `remove():按照元素值索引并删除索引到的第一个元素。`

     `clear():清空列表。`

     `del():按照索引，切片，步长删除列表中的元素，同时还可以删除列表。`

     ```python
     # 切片删除该元素
     l = ['pl', 'pl1', 'pl2', 'pl3']
     del l[1:]
     print(l) # ['pl1']
     
     # 切片(步长)删除该元素
     l = ['pl', 'pl2', 'pl3', 'pl4']
     del l[::2]
     print(l) # ['pl2', 'pl4']
     #注:这么删除会有坑，后面会遇到
     ```

   - 修改

     `通过索引，切片，步长进行元素的修改。`

     `例：lst = [123,456,"china","hello"]`

     `通过索引：lst[0] = 321 >>>lst = [321,456,"china","hello"]`

     `通过切片：lst[0:2] = “123” >>>lst = [1,2,3,"china","hello"]`

     `通过步长：lst[0:4:2] = "ss" >>>['s', 456, 's', 'hello']`

     `注：通过切片修改元素，就是将元素迭代进列表指定的位置，数据量不会影响列表。但是通过步长来修改列表，如果修改的元素超出列表的范围就会报错。`

   - 查看

     `利用for循环迭代列表中所有的元素进行查看。`

   - 补充

     `count()：统计某个元素在列表中出现的次数。`

     `index()：从列表中找出某个值第一个匹配的索引位置。`
     
     `sort():在列表的原空间位置对列表中的元素进行排序。`
     
     `reverse():在列表的原空间位置逆序列表中所有的元素。`

- 列表嵌套：

   ```python
   lis = [2, 3, "k", ["qwe", 20, ["k1", ["tt", 3, "1"]], 89], "ab", "adv"]
   将列表lis中的"tt"变成大写（用两种方式）。
   print (lis[3][2][1][0])#从前面取
   print (lis[-3][-2][0]) #从后面取
   #练习
   将列表中的数字3变成字符串"100"（用两种方式）。
   将列表中的字符串"1"变成数字101（用两种方式）。
   ```

- 列表的+和*

   ```python
   l1 = [1, 2, 3]
   l2 = [4, 5, 6]
   # print(l1+l2)  # [1, 2, 3, 4, 5, 6]
   print(l1*3)  # [1, 2, 3, 1, 2, 3, 1, 2, 3]
   ```

#### 元组

1. 什么是元组：对于容器型数据类型list，无论谁都可以对其增删改查，那么有一些重要的数据放在list中是不安全的，所以需要一种容器类的数据类型存放重要的数据，创建之初只能查看而不能增删改，这种数据类型就是元组。
2. 特点：元组一旦 初始化就不能修改了，有序，支持索引。
3. 用途：元组一般储存一些比较重要的信息，在配置文件中会使用。
4. 方法：index()、count()、len()。
5. t = (1,)：单元素元组，使用“，”来消除小括号的歧义。
6. 元组支持索引切片和列表一样。

#### range（范围）

1. 结构：range(第一个元素是起始位置，第二个元素是终止位置，第三个元素是步长)
2. 对比python3和python2中的range

- 在python3中，range是一个迭代对象，写入的什么就打印什么。
- 在python2中，range返回一个列表，它的xrange和python是很像的。

  注：一个很重要的应用：控制循环的次数，但是不使用循环的数据

  lst = []

  for i in range(3):

  ​	lst.append(input("请输入用户名："))   #这个例子借用了就是循环控制，相当于while，但是比它代码少

  print（lst）


#### 练习

```python
#1.写代码，有如下列表，按照要求实现每一个功能
#计算列表的长度并输出
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
print (li.__len__())
#列表中追加元素"seven",并输出添加后的列表
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
li.append("seven")
print(li)
#请在列表的第2个位置前插入元素"Tony",并输出添加后的列表
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
li.insert(1,"Tony")
print (li)
#请修改列表第2个位置的元素为"Kelly",并输出修改后的列表
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
li[1] = "Kelly"
print (li)
#请将列表l2=[1,"a",3,4,"heart"]的每一个元素添加到列表li中，一行代码实现，不允许循环添加。
l2=[1,"a",3,4,"heart"]
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
print(li+l2)
#请将字符串s = "qwert"的每一个元素添加到列表li中，一行代码实现，不允许循环添加。
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
s = "qwert"
li[5:] = s
print (li)
#请删除列表中的元素"ritian",并输出添加后的列表
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
li.remove("ritian")
print (li)
#请删除列表中的第2个元素，并输出删除的元素和删除元素后的列表
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
print (li.pop(1),li)
#请删除列表中的第2至4个元素，并输出删除元素后的列表
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
del li[1:4]
print (li)
```

```python
#2.写代码，有如下列表，利用切片实现每一个功能
#通过对li列表的切片形成新的列表l1,l1 = [1,3,2]
li = [1, 3, 2, "a", 4, "b", 5,"c",["a","b","cc"]]
l1 = li[0:3]
print (l1)
#通过对li列表的切片形成新的列表l2,l2 = ["a",4,"b"]
li = [1, 3, 2, "a", 4, "b", 5,"c",["a","b","cc"]]
l1 = li[3:6]
print (l1)
#通过对li列表的切片形成新的列表l3,l3 = ["1,2,4,5]
li = [1, 3, 2, "a", 4, "b", 5,"c",["a","b","cc"]]
l1 = li[0:7:2]
print (l1)
#通过对li列表的切片形成新的列表l4,l4 = [3,"a","b"]
li = [1, 3, 2, "a", 4, "b", 5,"c",["a","b","cc"]]
l1 = li[1:6:2]
print (l1)
#通过对li列表的切片形成新的列表l5,l5 = ["c"]
li = [1, 3, 2, "a", 4, "b", 5,"c",["a","b","cc"]]
l1 = [li[7]]
print (l1)
#通过对li列表的切片形成新的列表l7,l7 = ['cc', 'b', 'a']
li = [1, 3, 2, "a", 4, "b", 5,"c",["a","b","cc"]]
l1 = li[-1][::-1]
print (l1)
#通过对li列表的切片形成新的列表l6,l6 = ["b","a",3]
li = [1, 3, 2, "a", 4, "b", 5,"c",["a","b","cc"]]
l1 = li[-1][-2:-4:-1]
l2 = l1.append(3)
print (l1)
```

```python
#3.写代码，有如下列表，按照要求实现每一个功能。
#将列表lis中的"tt"变成大写（用两种方式）。
lis = [2, 33, "k", ["qwe", 20, ["k1", ["tt", 3, "1"]], 89], "ab", "adv"]
#方式一：
lis[-3][-2][-1][0] = "TT"
print (lis)
#方式二
a = lis[-3][-2][-1][0].upper()
lis[-3][-2][-1][0] = a
print (lis)
#将列表中的数字3变成字符串"100"（用两种方式）。
lis = [2, 33, "k", ["qwe", 20, ["k1", ["tt", 3, "1"]], 89], "ab", "adv"]
#方式一：
lis[-3][-2][-1][1] = "100"
print (lis)
#方式二
l1 = lis[-3][-2][-1]
l1.remove(3)
l1.insert(1, '100')
print (lis)
#将列表中的字符串"1"变成数字101（用两种方式）。
#方式一
lis = [2, 33, "k", ["qwe", 20, ["k1", ["tt", 3, "1"]], 89], "ab", "adv"]
lis[-3][-2][-1][-1] = 101
print (lis)
#方式二
l1 = lis[-3][-2][-1]
l1.remove("1")
l1.insert(2, 101)
print(lis)
```

```python
#4.请用代码实现：
#利用下划线将列表的每一个元素拼接成字符串"alex_wusir_taibai"
li = ["alex", "wusir", "taibai"]
print ("_".join(li))    #拼接的符号.join(可迭代对象)
```

```python
#5.利用for循环和range打印出下面列中每个元素的索引。
li = ["alex", "WuSir", "ritian", "barry", "wenzhou"]
for i in range(li.__len__()):
	print (li[i])
```

```python
#6.利用for循环和range将100以内所有的偶数添加到一个新列表中。
li = []
for i in range(1,101,1):
	if i%2 == 0:
		li.append(i)
print (li)
```

```python
#7.利用for循环和range找出50以内能被3整除的数，并将这些数插入到一个新列表中。
li = []
for i in range(3,51,3):
	li.append(i)
print (li)
```

```python
#8.利用for循环和range从100 ~ -1，倒序打印。
for i in range(100,-2,-1):
	print (i)
```

```python
#9.利用for循环和range从100~10，倒序将所有的偶数添加到一个新列表中
#然后在对列表的元素进行筛选，将能被4整除的数留下来。
li = []
for i in   range(100,9,-2):
	li.append(i)
for i in li:
	if i%4 !=0:
		li.remove(i)
print (li)
```

```python

#10.利用for循环和range，将1-30的数字中能被3整除的数改成* 依次添加到的列表当中
li = []
for i in range(1,31,1):
	if i%3 == 0:
		li.append("*")
	else:
		li.append(i)
print (li)

```

```python

#11.查找列表li中的元素，移除每个元素的空格，并找出以"A"或者"a"开头，
# 并以"c"结尾的所有元素，并添加到一个新列表中,最后循环打印这个新列表。
li = ["TaiBai          ", "alexC", "AbC ", "egon", " riTiAn", "WuSir", " aqc"]
l1 = []
l = ""
for i in li:
	l = i.strip()
	if (l.startswith("A") or l.startswith("a")) and l.endswith("c"):
		l1.append(l)
print(l1)
```

```python
12.开发敏感词语过滤程序，提示用户输入评论内容，如果用户输入的内容中包含特殊的字符：
敏感词列表 li = ["苍老师", "东京热", "武藤兰", "波多野结衣"]
则将用户输入的内容中的敏感词汇替换成等长度的*（苍老师就替换***），
并添加到一个列表中；如果用户输入的内容没有敏感词汇，则直接添加到上述的列表中。
"""
li = ["苍老师", "东京热", "武藤兰", "波多野结衣"]
neirong = input("请输入评论内容：")
neirong1 = "".join(neirong.split())
neirong2 = list(neirong1)
zifucount = neirong2.__len__()
put = []
for i in range(0,zifucount,1):
	if neirong2[i] in ["苍","东","武","波"]:
		if neirong2[i+1]+neirong2[i+2] in ["老师", "京热", "藤兰"]:
			neirong2[i:i+3] = "***"
		if neirong2[i+1]+neirong2[i+2]+neirong2[i+3]+neirong2[i+4] == "多野结衣":
			neirong2[i:i + 5] = "*****"
print (neirong2)

#12-1修改版本
li = ["苍老师", "东京热", "武藤兰", "波多野结衣"]
lst =[]
contend = input()
contend1 = "".join(contend.split())
print (contend1)
for i in li:
	if i in contend1:
		contend1 = contend1.replace(i,len(i)*"*")
lst.append(contend1)
print (lst)
```

```python
#13.有如下列表（选做题）
#循环打印列表中的每个元素，遇到列表则再循环打印出它里面的元素。
li = [1, 3, 4, "alex", [3, 7, 8, "TaiBai"], 5, "RiTiAn"]
for i in li:
	if type(i) == list:
		for j in i:
			print (j)
	else:
		print (i)
```

```python
#14.用户输入一个数字,使用列表输出这个数字内的斐波那契数列,如下列表:(选做题)
#用户输入100 输出[1,1,2,3,5,8,13,21,34,55,89]这个列表
inputNum = int(input())
put = []
if inputNum == 1:
	put = [1]
	print (put)
if inputNum == 2:
	put = [1,1]
	print (put)
if inputNum > 2:
	put = [1,1]
	a = 1
	b = 1
	c = 2
	while c < inputNum:
		put.append(c)
		c = a + b
		a = b
		b = c
	print (put)
    
#14-1
lst = [1,1]
num = input("请输入循环次数：")
if num.isdecimal():
	while lst[-1]+lst[-2] <= int(num):
		lst.append(lst[-1]+lst[-2])
	print (lst)
else:
	print("请你输入数字。")
```