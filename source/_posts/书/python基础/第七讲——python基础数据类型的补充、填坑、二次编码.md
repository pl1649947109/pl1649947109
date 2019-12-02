---
title: 第七讲——python之基础数据类型的补充、填坑、二次编码
id: 7
date: 2019-7-26 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 数据类型方法及其他知识的补充
- 填坑
- 二次编码
- 练习

<!-----more----->

### 数据类型方法及其他知识的补充

**内容**

- 5种基本数据类型的方法的补充
- 对我们所学的基本的数据类型按照一定的规则进行分类
- bool返回False的所有情况
- 数据类型之间的转换

```python
# str
#首字母大写
s = "god is a girl."
print (s.capitalize())
#每个单词的首字母大写
s = "god is a girl."
print (s.title())
#大小写反转
s = "god is A girl."
print (s.swapcase())
#查找
s = "god is a girl."
print(s.find("s"))  #都是按照元素值查找元素的位置，没有改元素时返回None
print (s.index("d"))   #和find一样的效果，但是找不到元素时会报错
#居中
s = "god is a girl."
print (s.center(20)) #该字符串占据了20个位置，同时处在这20个字符串的中间
#填充
#.format()   %   f
s = "god {} {} {}"
print (s.format("is","a","girl"))  #按照位置顺序填充
s = "god {1} {2} {0}"
print (s.format("is","a","gril"))#按照下标填充
s = "god {a} {b} {c}"
print (s.format(a="is",b="a",c="gril")) #按照关键字填充
#字符串+
s = "god is"
s1 = " a  gril."
print (id(s),id(s1),id(s+s1))   #把s1的字符串追加到s后面
#字符串*
s = "good"
print (s*4)
#注：字符+和*都是开辟新的空间


# list
#反转
lst = [1,2,3,4]
#print (lst.reverse())  #这个返回的是None
lst.reverse()
print (lst)
#排序
lst = [1,2,3,45,2,36,5]
lst.sort()    #默认升序
print(lst)
#排序之降序
lst.sort(reverse=True)
print (lst)
#统计
print (lst.count(2))  #统计2出现的次数
#注：列表的*的时候元素都是共享的


# tuple
'''
Tu的数据类型（数据类型在括号里面的数据类型都是其本身的数据类型）
Tu = (1)      int
Tu = (“1”   str
Tu = (1,)     tuple
'''
#注：元组也是可+可*的，但那是元组内存都是共享的

# dict

#字典的定义方式：dict(key=1,key=2,key=3)
#随机删除
dic = {1:1,2:2,3:3}
print (dic)
dic.popitem()
print (dic)
#批量创建字典
dic = {}
#dic1 =dic.fromkeys(可迭代数据，所有键公用的值)
#注：共享值最好使用不可变的数据类型，这样可以避免所有的键都使用一片可变的值得空间地址
#这样，在改变其中的某一个值的时候，去其他所有的键的值都会发生改变的，；但是当我们使用的
#不可变的数据类型当作值得时候，就算我们去改变其中的某一个键对应的值得时候，其他的键对应的值
#也不会改变。
其实update也是一种删除字典元素的方式


# set
#创建一个空的集合
s = set()
print (type(s))
#集合元素的迭代添加
s = set("asdfghj")
print (s)

"""
#对我们所学的基本的数据类型按照一定的规则进行分类
#有序
数字
字符串
列表
元组
#无序
字典
集合
#可变的数据类型
列表
字典
集合
#不可变的数据类型
数字
字符串
布尔值
元组
#取值的顺序
	直接取值：字符串，数字，布尔值
	按照顺组取值：列表，元组，字符串
	通过键取值：字典
"""

"""
布尔值：返回False
数字：0
字符串:""
列表：[]
元组：()
字典：{}
集合：set()
其他：None
"""
print (bool(0),bool(""),bool([]),bool({}),bool(()),bool(set()),bool(None))

#数据类型之间的转换
#元组转列表：list(tuple)
#列表转元组：tuple(list)
#列表转字符串：str.join(list)
#字符串转列表：str.split()
```

### 填坑

##### 循环添加

```python
lst  = [1,2,3,4,5,6]
for i in lst:
	lst.append(7)  #这样会循环一直的往列表里面添加7，进入死循环
	print (lst)
print (lst)
```

##### 列表循环删除的错误的实例

```python
列表: 循环删除列表中的每⼀个元素
li = [11, 22, 33, 44]
for e in li:
 li.remove(e)
print(li)
结果:
[22, 44]

分析原因: for的运⾏过程. 会有⼀个指针来记录当前循环的元素是哪⼀个, ⼀开始这个指针指向第0 个.

然后获取到第0个元素. 紧接着删除第0个. 这个时候. 原来是第⼀个的元素会⾃动的变成 第0个.

然后指针向后移动⼀次, 指向1元素. 这时原来的1已经变成了0, 也就不会被删除了.
li = [11, 22, 33, 44]
for i in range(0, len(li)):
    del li[i]
print(li)
结果: 报错
# 删除的时候li[0] 被删除之后. 后⾯⼀个就变成了第0个.
# 以此类推. 当i = 2的时候. list中只有⼀个元素. 但是这个时候删除的是第2个 肯定报错啊


经过分析发现. 循环删除都不⾏. 不论是⽤del还是⽤remove. 都不能实现. 那么pop呢?

用pop删除试一试
for el in li:
 li.pop() # pop也不⾏
print(li)
结果:
[11, 22]
```

##### 列表循环删除成功

```python
只有这样才是可以的:
for i in range(0, len(li)): # 循环len(li)次, 然后从后往前删除
 li.pop()
print(li)
或者. ⽤另⼀个列表来记录你要删除的内容. 然后循环删除

li = [11, 22, 33, 44]
del_li = []
for e in li:
 del_li.append(e)
for e in del_li:
 li.remove(e)
print(li)

li = [1,2,3,4]
lst = li[:]
for i in lst:
    li.remove(i)
print(li)

li = [1,2,3,4]
lst = li[:]
for i in lst:
    li.remove(i)
print(li)
注意: 由于删除元素会导致元素的索引改变, 所以容易出现问题. 尽量不要再循环中直接去删除元素. 可以把要删除的元素添加到另⼀个容器中然后再批量删除.
```

##### 字典的坑

```python
dict中的fromkey(),再次重提 可以帮我们通过list来创建⼀个dict
dic = dict.fromkeys(["jay", "JJ"], ["周杰伦", "麻花藤"])
print(dic)
结果:
{'jay': ['周杰伦', '麻花藤'], 'JJ': ['周杰伦', '麻花藤']}


代码中只是更改了jay那个列表. 但是由于jay和JJ⽤的是同⼀个列表. 所以. 前⾯那个改了. 后面那个也会跟着改　

```

##### 字典在循环时不能修改数据的大小

```python
dict中的元素在迭代过程中是不允许进⾏ 添加 和 删除 的
dic = {'k1': 'alex', 'k2': 'wusir', 'k3': '大宝哥'}
# 删除key中带有'k'的元素
for k in dic:
  if 'k' in k:
 del dic[k] # dictionary changed size during iteration, 在循环迭
代的时候不允许进⾏删除操作
print(dic)


那怎么办呢? 把要删除的元素暂时先保存在⼀个list中, 然后循环list, 再删除
dic = {'k1': 'alex', 'k2': 'wusir', 'k3': '大宝哥'}
dic_del_list = []
# 删除key中带有'k'的元素
for k in dic:
     if 'k' in k:
     dic_del_list.append(k)

for el in dic_del_list:
     del dic[el]
print(dic)

# 使用两个字典进行删除
dic = {'k1': 'alex', 'k2': 'wusir', 'k3': '大宝哥'}
dic1 = dic.copy()
for i in dic1:
    dic.pop(i)
print(dic)


集合在循环时不能修改数据的大小
set1 = {1,2,3,4,5,6}
for i in set1:
    set1.pop()
print(set1)

Traceback (most recent call last):
  File "/python项目/m2.py", line 224, in <module>
    for i in set1:
RuntimeError: Set changed size during iteration


成功进行删除
set1 = {1,2,3,4,5,6}
set2 = set1.copy()
for i in set2:
    set1.remove(i)
print(set1)
```

### 二次编码

##### 基本的4中编码

- ascll:没有中文，每个字符1个Byte。

- gbk：中文国标码，每个字符2个Byte.

- unicode:万国码，里面包含了全世界的文字编码，每个字符4个Byte。

- utf-8：可变长度的万国码。英文1个Byte，欧洲2个Byte，亚洲3个Byte。

- 注：在python3中，使用的就是unicode编码，为了满足所有的人来使用，但是这种储存的方式非常的占用内用，这个时候就出现转码，那么转码是怎么实现的呢?及时数据经过编码之后的数据类型是bytes的形式。

- bytes的表现形式：

  - 英⽂ b'alex' 英⽂的表现形式和字符串没什么两样

  - 中⽂ b'\xe4\xb8\xad' 这是⼀个汉字的UTF-8的bytes表现形式

##### 编码

```python
s = "alex"
print(s.encode("utf-8")) # 将字符串编码成UTF-8
print(s.encode("GBK")) # 将字符串编码成GBK
结果:
b'alex'
b'alex'
s = "中"
print(s.encode("UTF-8")) # 中⽂编码成UTF-8
print(s.encode("GBK")) # 中⽂编码成GBK
结果:
b'\xe4\xb8\xad'
b'\xd6\xd0'


记住: 英⽂编码之后的结果和源字符串⼀致. 中⽂编码之后的结果根据编码的不同. 编码结果也不同. 　　 我们能看到. ⼀个中⽂的UTF-8编码是3个字节. ⼀个GBK的中⽂编码是2个字节. 　　 编码之后的类型就是bytes类型. 在⽹络传输和存储的时候我们python是发送和存储的bytes 类型. 　　 那么在对⽅接收的时候. 也是接收的bytes类型的数据. 我们可以使⽤decode()来进⾏解码操作. 　　 把bytes类型的数据还原回我们熟悉的字符串:
s = "我叫李嘉诚"
print(s.encode("utf-8")) #
b'\xe6\x88\x91\xe5\x8f\xab\xe6\x9d\x8e\xe5\x98\x89\xe8\xaf\x9a'
print(b'\xe6\x88\x91\xe5\x8f\xab\xe6\x9d\x8e\xe5\x98\x89\xe8\xaf\x9a'.decod
e("utf-8")) # 解码

```

##### 解码

```python
s = "我是⽂字" bs = s.encode("GBK") 
# 我们这样可以获取到GBK的⽂字 
# 把GBK转换成UTF-8 
# ⾸先要把GBK转换成unicode. 也就是需要解码 
s = bs.decode("GBK") # 解码 
# 然后需要进⾏重新编码成UTF-8 
bss = s.encode("UTF-8") # 重新编码 
print(bss)
```

图解![image-20190712001047937](http://39.97.99.7/python/assets/image-20190712001047937.png)

**对比str和bytes**

| 类名     | str类型                                      | bytes类型                                                    | 标注                                                         |
| -------- | -------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 名称     | str,字符串,文本文字                          | bytes,字节文字                                               | 不同，可以通过文本文字或者字节文字加以区分                   |
| 组成单位 | 字符                                         | 字节                                                         | 不同                                                         |
| 组成形式 | '' 或者 "" 或者 ''' ''' 或者 """ """         | b'' 或者 b""  或者 b''' ''' 或者 b""" """                    | 不同，bytes类型就是在引号前面+b(B)大小写都可以               |
| 表现形式 | 英文： 'alex' 中文： '中国'                  | 英文：b'alex'中文：b'\xe4\xb8\xad\xe5\x9b\xbd'               | 字节文字对于ascii中的元素是可以直接显示的，但是非ascii码中的元素是以十六进制的形式表示的，不易看出。 |
| 编码方式 | Unicode                                      | 可指定编码（除Unicode之外）比如UTF-8，GBK 等                 | 不同                                                         |
| 相应功能 | upper lower spllit 等等                      | upper lower spllit 等等                                      | 几乎相同                                                     |
| 转译     | 可在最前面加r进行转译                        | 可在最前面加r进行转译                                        | 相同                                                         |
| 重要用途 | python基础数据类型，用于存储少量的常用的数据 | 负责以二进制字节序列的形式记录所需记录的对象，至于该对象到底表示什么（比如到底是什么字符）则由相应的编码格式解码所决定。Python3中，bytes通常用于网络数据传输、二进制图片和文件的保存等等 | bytes就是用于数据存储和网络传输数据                          |
| 更多     | ......                                       | ......                                                       |                                                              |

### 练习

```python
#1.看代码写结果
v1 = [1,2,3,4,5]
v2 = [v1,v1,v1]
v1.append(6)
print(v1) #>>>[1,2,3,4,5,6]
print(v2) #>>>[[1,2,3,4,5,6],[1,2,3,4,5,6],[1,2,3,4,5,6]]
```

```python
#2.看代码写结果
v1 = [1,2,3,4,5]
v2 = [v1,v1,v1]
v2[1][0] = 111
v2[2][0] = 222
print(v1) #>>>[222,2,3,4,5]
print(v2) #>>>[[1,2,3,4,5],[111,2,3,4,5],[222,2,3,4,5]]
```

```python
#3.看代码写结果，并解释每一步的流程。
v1 = [1,2,3,4,5,6,7,8,9]
v2 = {}
for item in v1:     #遍历v1
    if item < 6:    #for循环1~5进入这个循环
        continue    #退出本次循环
    if 'k1' in v2:  #if‘k1’在字典v2中，执行条件内的代码
        v2['k1'].append(item)  #满足条件，把item追加到字典v2的key'k1'的值中
    else:   #以上条件都不满足，进入该条件下的代码
        v2['k1'] = [item ]  #把item添加到列表，并把它赋值给字典的k1键的值
print(v2)  #>>>{'k1': [6, 7, 8, 9]}  #打印字典v2
```

```python
#4.简述赋值和深浅拷贝？
'''
解释：
赋值就是变量指向一片内存空间
深拷贝就是从新开辟一片内存空间，可变的数据类型把它放到新开辟的内存空间中，
不可变的数据类型，共享内存
浅拷贝：复制第一层的数据，共享第二层的数据。
'''
```

```python
#5.看代码写结果
import copy
v1 = "alex"    #不可变的数据类型
v2 = copy.copy(v1)  #浅拷贝
v3 = copy.deepcopy(v1) #小数据池
print(v1 is v2)  #>>>True
print(v1 is v3) #>>>True
```

```python
py
#6.看代码写结果
import copy
v1 = [1,2,3,4,5]    #可变的数据类型
v2 = copy.copy(v1)  #浅拷贝开辟一片新的内存地址
v3 = copy.deepcopy(v1)  #深拷贝也开辟一片新的内存空间
print(v1 is v2) #>>>True
print(v1 is v3) #>>>False
```

```python
#7.看代码写结果
import copy
v1 = [1,2,3,4,5]    #可变的数据类型，但是里面该共享的还是共享
v2 = copy.copy(v1)
v3 = copy.deepcopy(v1)
print(v1[0] is v2[0])   #True
print(v1[0] is v3[0])   #True
print(v2[0] is v3[0])   #True
```

```python
#8.看代码写结果
import copy
v1 = [1,2,3,4,[11,22]]  #可变的数据类型，分层
v2 = copy.copy(v1)
v3 = copy.deepcopy(v1)
print(v1[-1] is v2[-1]) #>>>True   浅拷贝共享所有的内存
print(v1[-1] is v3[-1]) #>>>False  因为可变的数据类型里面嵌套了第二层的数据，必须放在新的空间
print(v2[-1] is v3[-1]) #>>>False  v1和v2共享内存
```

```python
#9.看代码写结果
import copy
v1 = [1,2,3,{"name":'太白',"numbers":[7,77,88]},4,5]  #可变的数据类型
v2 = copy.copy(v1)  #浅拷贝
print(v1 is v2) #>>>True
print(v1[0] is v2[0])   #>>>True
print(v1[3] is v2[3])   #>>>True
print(v1[3]['name'] is v2[3]['name'])   #True
print(v1[3]['numbers'] is v2[3]['numbers']) #True
print(v1[3]['numbers'][1] is v2[3]['numbers'][1])   #True
```

```python
#10.看代码写结果
import copy
v1 = [1,2,3,{"name":'太白',"numbers":[7,77,88]},4,5]  #可变的数据类型
v2 = copy.deepcopy(v1)  #深拷贝
print ("\n")
print(v1 is v2) #>>>False
print(v1[0] is v2[0])   #>>>True
print(v1[3] is v2[3])   #>>>False  #可变的数据类型，新开辟空间
print(v1[3]['name'] is v2[3]['name'])   #>>>True  #不可变的数据类型，共享
print(v1[3]['numbers'] is v2[3]['numbers']) #>>>False  #可变的数据类型，开辟新的空间
print(v1[3]['numbers'][1] is v2[3]['numbers'][1]) #>>>False #不可变的数据类型，共享空间
```

```python
#11.请说出下面a,b,c三个变量的数据类型。
a = ('太白金星')    #字符串
b = (1,)    #元组
c = ({'name': 'barry'}) #字典
print(type(a),type(b),type(c))
#12.按照需求为列表排序：
l1 = [1, 3, 6, 7, 9, 8, 5, 4, 2]
# 从大到小排序
l1.sort()
print (l1)
# 从小到大排序
l1.sort(reverse=True)
print (l1)
# 反转l1列表
l1 = [1, 3, 6, 7, 9, 8, 5, 4, 2]
l1.reverse()
print (l1)

```

```python
#13.利用python代码构建一个这样的列表(升级题)：
#[['_','_','_'],['_','_','_'],['_','_','_']]
lst = []
lst1 = []
for i in range(3):
    lst.append('_')
for j in range(3):
    lst1.append(lst)
print (lst1)
```

```python
#14.看代码写结果：
l1 = [1,2,]
l1 += [3,4]
print(l1)   #>>>[1,2,3,4]
```

```python
#15.看代码写结果：
dic = dict.fromkeys('abc',[])   #迭代的值是可变的数据 类型所以只要有一个键改变了，所有的都变了
dic['a'].append(666)
dic['b'].append(111)
print(dic)  #>>>{'a':[666，111],'b':[666,111],'c':[666,111]}
```

```python
# #16.l1 = [11, 22, 33, 44, 55]，请把索引为奇数对应的元素删除（不能一个一个删除）
l1 = [11, 22, 33, 44, 55]
Count = l1.__len__()
lst = []
for i in range(1,Count,2):
    lst.append(l1[i])
for j in lst:
    l1.remove(j)
print (l1)

#方法二
l1 = [11, 22, 33, 44, 55]
del l1[1::2]
print (l1)
#这种方法是利用了列表的特性，删除列表中的元素之后，里面其他的元素就会自动补位。因此就会出现跳删的
#情况，就是这一种。但是不推荐使用。
```

```python
# 17.dic = {'k1':'太白','k2':'barry','k3': '白白', 'age': 18}
# 请将字典中所有键带k元素的键值对删除.
dic = {'k1':'太白','k2':'barry','k3': '白白', 'age': 18}
for i in list(dic.keys()):
    for j in i:
        if j == 'k':
            del dic[i]
print (dic)
```

```python
# #18.完成下列需求：
# s1 = '太白金星'
# 将s1转换成utf-8的bytes类型。
# 将s1转化成gbk的bytes类型。
# b = b'\xe5\xae\x9d\xe5\x85\x83\xe6\x9c\x80\xe5\xb8\x85'
# b为utf-8的bytes类型，请转换成gbk的bytes类型。
s1 = '太白金星'
print (s1.encode('utf-8'))
print (s1.encode('gbk'))
b = b'\xe5\xae\x9d\xe5\x85\x83\xe6\x9c\x80\xe5\xb8\x85'
print (b.decode('utf-8').encode('gbk'))
```

```python
# 19.用户输入一个数字，判断一个数是否是水仙花数。
# 水仙花数是一个三位数, 三位数的每一位的三次方的和还等于这个数. 那这个数就是一个水仙花数,
# 例如: 153 = 1**3 + 5**3 + 3**3
while True:
    num = input("请输入数字:")
    if num.isdecimal():
        if (int(num)//100)**3+((int(num)//10)%10)**3+(int(num)%10)**3 == int(num):
            print ("这是一个水仙花数。")
            break
        else:
            print ("这不是一个水仙花数。")
            break
    else:
        print ("你输入的数字有误，请重新输入.")
```

```python
# 20.把列表中所有姓周的⼈的信息删掉(此题有坑, 请慎重):
# lst = ['周⽼⼆', '周星星', '麻花藤', '周扒⽪']
# 结果: lst = ['麻花藤']
lst = ['周⽼⼆', '周星星', '麻花藤', '周扒⽪']
lst1 = []
for i in range(len(lst)):
    if lst[i][0] == '周':
        lst1.append(i)
lst1.sort(reverse=True)
for j in lst1:
    del lst[j]
print (lst)#方法一（把需要删除的元素的下边挑出来放到列表中，最后一起删除，也是从后往前删，del lst(下标)）
lst = ['周⽼⼆', '周星星', '麻花藤', '周扒⽪']
lst1 = []
for i in range(len(lst)):
    if lst[i][0] == '周':
        lst1.append(i)
lst1.sort(reverse=True)
for j in lst1:
    del lst[j]
print (lst)


#方法二(复制一个列表，一边查一遍删，remove（元素）)
lst = ['周⽼⼆', '周星星', '麻花藤', '周扒⽪']
lst1 = []
for i in lst:
    lst1.append(i)
for i in lst1:
    if i[0] == "周":
        lst.remove(i)
print (lst)
#方法三（这个range里面迭代的内容是很有趣的，从大往小迭代，并且还是从后往前删除，就算不删除某一位上的元素，在下次循环的时候也跳过了该元素。这也是为什么-1，-5这种形式不行的原因）
lst = ['周⽼⼆', '周星星', '麻花藤', '周扒⽪']
for i in range(len(lst)-1,-1,-1):
    if lst[i][0] == "周":
        del lst[i]
print (lst)

#错误示例(这种情况和前面往后删除的原理是一样的，删了之后，再增加就跳过了本该判断的元素，所以删不干净。)
lst = ['周⽼⼆', '周星星', '麻花藤', '周扒⽪']
for i in range(-1,-len(lst)-1,-1):
    if lst[i][0] == "周":
        del lst[i]
print (lst)
#注：这一题是个大坑，只有在全部删除列表中的元素时，我们才能使用从后往前删除，但是我们删除列表中的一些元素的时候，我们必须去利用第三方的列表。而不能去迭代删除。
```

```python
#21.车牌区域划分, 现给出以下车牌. 根据车牌的信息, 分析出各省的车牌持有量. (选做题)
cars = ['鲁A32444','鲁B12333','京B8989M','⿊C49678','⿊C46555','沪 B25041']
locals = {'沪':'上海', '⿊':'⿊⻰江', '鲁':'⼭东', '鄂':'湖北', '湘':'湖南'}
#结果: {'⿊⻰江':2, '⼭东': 2, '上海': 1}
dic = {}
num = 0
for i in cars:
    if i[0] in locals:
        if i[0]  not in dic:
            dic[i[0]] = 1
        else:
            dic[i[0]] += 1
print (dic)

#方法二
cars = ['鲁A32444','鲁B12333','京B8989M','⿊C49678','⿊C46555','沪 B25041']
locals = {'沪':'上海', '⿊':'⿊⻰江', '鲁':'⼭东', '鄂':'湖北', '湘':'湖南'}
dic = {}
num = 0
for i in cars:
    if i[0] in locals:
        #方法二
        dic[i[0]] =  dic.get(i[0],0) + 1
        #解释这一行代码：如果i[0]不在字典中，dic[i[0]] = 0，如果在字典中，dic[i[0]] + 1
print (dic)
```