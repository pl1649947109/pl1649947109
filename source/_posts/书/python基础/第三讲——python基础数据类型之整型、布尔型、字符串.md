---
title: 第三讲——python基础数据类型之整型、布尔型、字符串
id: 3
date: 2019-7-21 22:00:00
tags: Python
comment: true
---

### 今日学习大纲

- 了解python的基本数据类型

- 整型和布尔值的转换
- 字符串的讲解
- 补充
- 练习

<!-----more----->

### 了解python的基本数据类型

**在python中有以下常用的7中数据类型**

```python
 int：整型值,主要用于运算。1 ，2,3...
 bool：布尔值，判断真假：True, False.
 str：字符串，简单少量的储存数据，并进行相应的操作。name = 'alex',
 tuple：元组，只读，不能更改。(1,'alex') 
 list：列表，储存大量有序数据，[1,'ses',True,[1,2,3],{'name':'jinxin'}]
 dict：字典，储存大量数据，且是关联性比较强的数据  {'name':'jinxin','age':18,'name_list':['张三'，'李四']}
```

### 整型和布尔值的转换

**整型的转换**

- 在python2.x中，整型分为int 和long（长整型）两种，获取的是整数。

- 在python3.x中，整型就只有int一种，获取的数浮点数。其实这里的int更像python2中的long。

- 进制之间的转换。

   原理：

   二进制转十进制：2的本位幂之和。

   十进制转二进制：短除2直到除不尽，余数逆序组合成二进制数。

   函数：

   | 将二进制转换成十进制 |      | print(bin(21))        |
   | -------------------- | ---- | --------------------- |
   | 将十进制转换成二进制 |      | print(int("10101",2)) |

   注释：

   十进制转二进制：只能转换整数，不能转换小数。

   二进制转十进制：int里面的2可以换成其他的各种进制的类  型，这样类推就可以把任意类型的数据转换成十进制数。

**布尔值的转换**

- print(bool(0))  #数字为0就是False,非0就是True。

- print(bool("")) #字符串里面没有任何元素是False,其它为True，即使里面是空格。

- print(bool(None)) #None是False。

- ```python
   注意：在 Python2 中是没有布尔型的，它用数字 0 表示 False，用 1 表示 True。
   到 Python3 中，把 True 和 False 定义成关键字了，但它们的值还是 1 和 0，它们可以和数字相加。
   ```

### 字符串的讲解

**字符串的定义**

python中凡是用引号引起来的数据都可以称为字符串类型，它里面的每一个字符称为一个元素。它是用来存储少量的数据的。

**操作符**

- 标准类型的操作符的比较都是按照ascll值来比较大小的。

- 序列操作符：“+”可以用来连接多个字符串、%（下序）,但是join()的性能更佳。字符串的*是重复字符串。

- 格式化操作符：%d、%f、%s、等等（推荐使用BIF的.format()方法）


**索引和切片**

理解：

- 三种方式 :[],[:],[::]；

- [起始位置:结束位置:步长]步长的意思就是间隔多少个步长取一个数值；

- [-x:-x]反向切;

- [::-1]逆序；

- 切片索引可以超过序列的长度，不会报错。
- 实例

```python
s = 'Python最NB'
#获取s字符串中前3个内容
s[0:3]
#获取s字符串中第3个内容
s[2]
#获取s字符串中后3个内容
s[-3:]
#获取s字符串中第3个到第8个
s[2:8]
#获取s字符串中第2个到最后一个
s[1:]
#获取s字符串中第1,3,5个内容
s[0:6:2]
#获取s字符串中第2,4,6个内容
s[1:7:2]
#获取s字符串中所有内容
s[:]
#获取s字符串中第4个到最后一个,每2个取一个
s[3::2]
#获取s字符串中倒数第5个到最开始,每3个取一个
s[-5::-3]
```

**字符串常用的方法**

```python
s = "pl"
s1 = s.upper() #全部大写
print(s1)
#>>>PL

s = "PL"
s1 = s.lower() # 全部小写
print(s1)
#>>>pl

# 应用场景
s = input("验证码(AbC5)")
if s.upper() == "AbC5".upper():
	print("验证码正确")
else:
	print("验证码错误!")
    
# 判断以什么开头:
s = "ALEX"
s1 = s.startswith("E",2,6)
print(s1)
#>>>True

# 判断以什么开头:
s = "ALEX"
s1 = s.endswith("E",2,6)
print(s1)
#>>>False

# 统计字符串中元素出现的次数
s= "alexdxjbx"
s1 = s.count("x")
print(s1)
#>>>3

# 脱: 字符串头尾两端的空格和换行符以及制表符
s = 'go  d   is  a gril  '
print (s.strip())
#>>>go  d   is  a gril 默认脱掉字符串的开头和末尾的空格
#指定脱元素脱，只能脱第一个元素
print (s.strip("g"))
print(s.lstrip('*')) #从字符串的左边开始脱
print(s.rstrip('*')) #从字符串的右边开始脱

#分割:以空格和换行符以及制表符进行分割，最终形成一个列表此列表不含有这个分割的元素。
s = "aelxlaaa"
s1 = s.split("l",maxsplit=1)  #可以通过指定方式进行切割
s2 = s.split("l",maxsplit=5)  #全切，可以超过所切元素的个数
print(s1)
#>>>['ae', 'xlaaa']
print(s2)
#>>>['ae', 'x', 'aaa']

# 替换:
s = "小白兔喜欢大灰狼，小白兔喜欢大灰狼"
s1 = s.replace("小白兔","小青蛙")
print(s1)
#>>>小青蛙喜欢大灰狼，小青蛙喜欢大灰狼

# is 系列:
s = "12"
print(s.isalnum()) # 判断是不是字母,数字,中文
print(s.isalpha())  # 判断是不是字母,中文
print(s.isdigit())  # 判断字符串是不是全都是阿拉伯数字
print(s.isdecimal())  # 判断是否是十进制
#>>>True
#>>>False
#>>>True
#>>>True

# 获取字符串的长度
name = "pl"
print(len(name))
#>>>2

#format的三种玩法 格式化输出
res='{} {} {}'.format('egon',18,'male')
res='{1} {0} {1}'.format('egon',18,'male')
res='{name} {age} {sex}'.format(sex='male',name='egon',age=18)
```

**for循环**

1，for  变量  in 可迭代对象:        #字符串迭代

​	print(变量),

2，for  变量 in  range(可迭代对象):    #数字迭代

​	print(变量)

**面试**

```python
for i in "pl":
	pass
print (i)    
#>>> l
pass:占位符,就是在for循环的内部什么都不干
```

### 补充：

**字符串脱空格的所有方法**

1：strip()方法，去除字符串开头或者结尾的空格

> \>>> a = " a b c "
>
> \>>> a.strip()
>
> 'a b c'

2：lstrip()方法，去除字符串开头的空格

> \>>> a = " a b c "
>
> \>>> a.lstrip()
>
> 'a b c '

3：rstrip()方法，去除字符串结尾的空格

> \>>> a = " a b c "
>
> \>>> a.rstrip()
>
> ' a b c'

4：replace()方法，可以去除全部空格

\# replace主要用于字符串的替换replace(old, new, count)

> \>>> a = " a b c "
>
> \>>> a.replace(" ", "")
>
> 'abc'

5: join()方法+split()方法，可以去除全部空格

join为字符字符串合成传入一个字符串列表，split用于字符串分割可以按规则进行分割

> \>>> a = " a b c "
>
> \>>> b = a.split()  # 字符串按空格分割成列表
>
> \>>> b ['a', 'b', 'c']
>
> \>>> c = "".join(b) # 使用一个空字符串合成列表内容生成新的字符串
>
> \>>> c 'abc'
>
>  
>
> \# 快捷用法
>
> \>>> a = " a b c "
>
> \>>> "".join(a.split())
>
> 'abc'

**常错的实例：（字符串的切片）**

```python
实例：（弄清楚） 
b = “asdfgh”
取得:fgh
从前面计数开始计：
b[3:]
从后后面计数开始计：
b[-3:]  或者  b[-3:-1] (取得fg)  
错误实例：
b[-1:-3]  #这种情况什么都取不到，因为方向往右，但是其实位置在结束位置的右边，所以什么都取不到。
```

**and  or取前取后：**

and：

- 全真取后
- 一真一假取假
- 全假取后

or:

- 全真取前

- 一真一假取真

- 全假取前

   ```python
   print (9 or 2)  #or全真取9
   print (True and 2 or True) #and全真取2，or全真取前
   print (8 and 0 or 5 or 4) #and一真一假取假，or一真一假取真，or全真取前
   ```

### 练习

```python
'''
2.有变量name = "aleX leNb" 完成如下操作：
移除 name 变量对应的值两边的空格,并输出处理结果
判断 name 变量是否以 "al" 开头,并输出结果
判断name变量是否以"Nb"结尾,并输出结果
将 name 变量对应的值中的 所有的"l" 替换为 "p",并输出结果
将name变量对应的值中的第一个"l"替换成"p",并输出结果
将 name 变量对应的值根据 所有的"l" 分割,并输出结果。
将name变量对应的值根据第一个"l"分割,并输出结果。
将 name 变量对应的值变大写,并输出结果
将 name 变量对应的值变小写,并输出结果
判断name变量对应的值字母"l"出现几次，并输出结果
如果判断name变量对应的值前四位"l"出现几次,并输出结果
请输出 name 变量对应的值的第 2 个字符?
请输出 name 变量对应的值的前 3 个字符?
请输出 name 变量对应的值的后 2 个字符?
'''
name = "aleX leNb"
print (name.strip())
if name.startswith("al") == True:
	print ("是的呢")
else:
	print ("不是呢")
if name.endswith("Nb") == True:
	print ("是的呢")
else:
	print ("不是呢")
print (name.replace("l","p"))
print (name.replace("l","p",1))
print (name.split("l"))
print (name.split("l",1))
print (name.upper())
print (name.lower())
print (name.count("l"))
print (name.count("l",0,4))
print (name[1])
print (name[0:3])
print (name[-1:-3:-1])
#2
"""
3.有字符串s = "123a4b5c"
通过对s切片形成新的字符串s1,s1 = "123"
通过对s切片形成新的字符串s2,s2 = "a4b"
通过对s切片形成新的字符串s3,s3 = "1345"
通过对s切片形成字符串s4,s4 = "2ab"
通过对s切片形成字符串s5,s5 = "c"
通过对s切片形成字符串s6,s6 = "ba2"
"""
s = "123a4b5c"
s1 = s[0:3]
s2 = s[3:6]
s3 = s[::2]
s4 = s[1:-2:2]
s5 = s[-1]
s6 = s[-3::-2]
print (s1 +"\n"+s2 +"\n"+s3 +"\n"+s4 +"\n"+s5 +"\n"+s6 )

#4.使用while和for循环分别打印字符串s="asdfer"中每个元素。
s="asdfer"
length = s.__len__()
for i in s:
	print(i)
while length:
	print (s[6-length])
	length -= 1

#5使用for循环对s="asdfer"进行循环，但是每次打印的内容都是"asdfer"。
s="asdfer"
for i in s:
	s1 += i
print(s1)
#6.使用for循环对s="abcdefg"进行循环，每次打印的内容是每个字符加上sb， 例如：asb, bsb，csb,...gsb。
s="abcdefg"
s1 = ""
for i in s:
	print (i+"sb")

#7使用for循环对s="321"进行循环，打印的内容依次是："倒计时3秒"，"倒计时2秒"，"倒计时1秒"，"出发！"。
s="321"
for i in s:
	print (f"倒计时{i}秒")
print ("出发！")

#8.实现一个整数加法计算器(两个数相加)：
#如：content = input("请输入内容:") 用户输入：5+9或5+ 9或5 + 9，然后进行分割再进行计算。
content = input("请输入内容:")
content = content.strip()
jiashu = content.split("+")
sum = int(jiashu[0]) + int(jiashu[1])
print (f"结果是:{sum}")

#选做题：实现一个整数加法计算器（多个数相加）：
#如：content = input("请输入内容:") 用户输入：5+9+6 +12+ 13，然后进行分割再进行计算。
content = input("请输入内容:")
content = content.strip()
jiashu = content.split("+")
sum = 0
for i in jiashu:
	sum += int(i)
print (sum)

#10.计算用户输入的内容中有几个整数（以个位数为单位）。
#如：content = input("请输入内容：") # 如fhdal234slfh98769fjdla
content = input("请输入内容：")
count = 0
for i in content:
	if i.isdecimal() == True:
		count += 1
print (count)

#写代码：计算 1 - 2 + 3 ... + 99 中除了88以外所有数的总和？
sum = 0
for i in range(1,100):
	if i%2 == 0:
		sum += i
	else:
		sum -= i
print (sum+88)

"""
12.选做题：写代码，完成下列需求：
用户可持续输入（用while循环），用户使用的情况：
输入A，则显示走大路回家，然后在让用户进一步选择：
是选择公交车，还是步行？
选择公交车，显示10分钟到家，并退出整个程序。
选择步行，显示20分钟到家，并退出整个程序。
输入B，则显示走小路回家，并退出整个程序。
输入C，则显示绕道回家，然后在让用户进一步选择：
是选择游戏厅玩会，还是网吧？
选择游戏厅，则显示 ‘一个半小时到家，爸爸在家，拿棍等你。’并让其重新输入A，B,C选项。
选择网吧，则显示‘两个小时到家，妈妈已做好了战斗准备。’并让其重新输入A，B,C选项。
"""
while True:
	goHomeWay = input("请您选择您回家的方式（方式A/B/C）:")
	if goHomeWay not in ["A","B","C"]:
		print ("您输入的回家方式不再列表中，请重新输入。")
	else:
		if goHomeWay == "A":
			print ("您选择从大路回家。")
			HowGoHome = input("请您进一步选择您回家的方式（A:步行，B:坐车）:")
			if HowGoHome not in  ["A","B"]:
				print ("请您正确输入回家的方式。")
				HowGoHome = input("请您进一步选择您回家的方式（A:步行，B:坐车）:")
			else:
				if HowGoHome == "A":
					print("您还有10分钟到家")
					break
				else:
					print("您还有20分钟到家")
					break
		elif  goHomeWay == "B":
			print ("您选择从小路回家。")
			break
		else:
			print ("您选择绕道回家。")
			HowGoHome = input("你绕道想去干嘛？（A：去游戏厅，B:去网吧）:")
			if HowGoHome not in ["A","B"]:
				print ("不能拥有更多的想法哟。请重新选择：")
				HowGoHome = input("你绕道想去干嘛？（A：去游戏厅，B:去网吧）:")
			else:
				if  HowGoHome == "A":
					print ("一个半小时到家，爸爸在家，拿棍等你。")
				else:
					print ("两个小时到家，妈妈已做好了战斗准备。")



#13.选做题：判断⼀句话是否是回⽂. 回⽂: 正着念和反着念是⼀样的. 例如, 上海⾃来⽔来⾃海上(使用步长)
方法一：
string = input("说话：")
count = string.__len__()
flag = False
if string == "":
	print ("输入的字符串不能为空！")
else:
	for i in range(count//2):
		if string[i] == string[-(i+1)]:
			continue
		else:
			print ("这句话不回文。")
			flag = True
			break
	if ( flag != True):
		print("这句话回文。")

方法二：
string = input("说话：")
if string == string[::-1]
    print ("这是回文数。")
else:
    print ("这不是回文数。")
```

