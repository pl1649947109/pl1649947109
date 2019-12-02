---
title: 第八讲——python基础之文件操作
id: 8
date: 2019-7-27 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 文件的读操作
- 文件的写操作
- 文件的追加操作
- 其他操作
- 练习

<!-----more----->

### 文件的读操作

**r模式**

一、上代码

```python
f = open("美丽的中国",'r',encoding="utf-8")
for i in f:
    print (i)#这种读取一行一行的把文件全部读取出来
print (f.read())#一下子把文件的内容读取出来
print(f.readline())#的读取文件一行的内容（默认在末尾追加\n,也就是自动换行）
print (f.readlines())#一行行的读取文件的内容（用的少）
注：读完的内容不会再次读取，就是光标自动定位到之前读取的位置
```

- 上面的代码中，f被称作文件句柄(一个迭代器)，文件操作符或者文件操作对象等。open就是调用操作系统的功能。

- open里面的参数：
     - 第一个参数是操作文件的路径。
  - 第二个参数是操作文件的模式，或读（r,rb,r+）或写(w,wb,w+)或追加(a,ab,a+)。
     - 第三个参数是指定文件的编码集。

二、大文件的读取

本质：一行一行的进行读取（因为如果文件过大，使用read()就会把缓内存撑爆,一般会控制读出的数量read(1024)）

就类似于：

Print(readline())

Print(readline())

Print(readline())

因此我们使用：

```python
f = ("我们美丽的中国",'r',encoding='utf-8')
for i in f:
	print (f)
```

扩展：repr():数据的原生态，就是什么数据类型就显示什么数据类型

```python
a = ["as",12,[12,"as"]]
print (a.__repr__())
>>>['as', 12, [12, 'as']]
```

**rb模式**

一、rb只读字节模式

```python
f = open('我们美丽的中国',mode='rb')
f = f.read()
print(f)
f.close()

结果:
b'\xe6\xa0\x87\xe9\xa2\x98\xe5\xbe\x88\xe5\xa5\xbd'
```

注：rb读出来的数据是bytes类型，在rb模式下，不能使用encoding字符集。

二、rb的作用：在读非文本文件的时候，比如要读取mp3，图像，视频等信息的时候就需要用到rb，因为这种数据是没办法直接显示出来的。这个模式是用于传输和储存的。

**相对路径与绝对路径**

- 相对路径解释：相对路径就是所操作的文件相对于本操作文件的位置。（同一个文件夹下面的文件，直接写文件名就可以。）

- 绝对路径的解释：就是操作文件在磁盘上面的位置（从根目录下开始一直到文件名。）

- ../就指向上一级的目录;./本级目录

- ```python
   open('C:\Users\Meet')  #这样程序是不识别的
   
   解决方法一:
   open('C:\\Users\\Meet') #这样就成功的将\进行转义   两个\\代表一个\    或者使用反斜杠
   
   解决方法二:
   open(r'C:\Users\Meet') #这样相比上边的还要省事,在字符串的前面加个小r也是转义的意思  推荐使用这种
   ```

   注：文件操作最好使用相对路径，因为我们打包安装之后，我们不可能把路径写到其他的目录。

### 文件的写操作

**w模式**

- f = open(“文件名”,’w’,编码集)


注：写进文件的时候只是在open这一步操作的时候才会清空文件的内容在后面的操作就不会影响，写的文件不存在就是创建一个文件。

**wb模式**

```python
如：
f = open('../我们美丽的中国',mode='wb')
f.write(“123456”)
f.write(“123456”)
f.write(“123456”)
上面的情况就是往文件里面写了3次“123456”（在）
```

wb写的时候一定要清楚文件的编码，不然会出现乱码。（在爬虫的时候会使用）

**追加模式**

- 只要是a或者ab,a+都是在文件的末尾写入,不论光标在任何位置.在追加模式下,我们写入的内容后追加在文件的末尾a模式如果文件不存在就会创建一个新文件

**r+模式**

对于读写模式，必须是先读后写，因为光标默认在开头位置，当读完了以后在进行写入，我们使用最多的就是r+模式。

```python
f1 = open('../path1/小娃娃.txt',mode='r+',encoding='utf-8')
msg = f1.read()
f1.write('这支烟灭了以后')
f1.close()
print(msg)
结果:
正常的读取之后,写在结尾
```

注：在该模式下如果读取了内容，不论读取的内容是多少光标显示的是多少。再写入或者操作文件的时候都是在结尾进行操作。

**w+和a+**

在这两个模式下都是先写再读的，但是写的时候就把所有的内容清空了，写入之后光标就定位到了文件的最末尾，因此我们是读不到内容的。

### 其他操作

**read**

- 文件打开方式为文本模式时，代表读取n个字符

- 文件打开方式为b模式时，代表读取n个字节

**seek()**

- seek()里面是一个参数：seek(n)光标移动到n位置,注意: 移动单位是byte,所有如果是utf-8的中文部分要是3的倍数。
  - seek(6)这个指针就移动到中文的第二个字符的后面。

- seek()里面有两个参数：
  - seek(0,0)：光标移动到文件的开头
  - seek(0,1)：光标移动到文件的当前位置
  - seek(0,2)：光标移动到文件的末尾

```python
f = open("小娃娃", mode="r+", encoding="utf-8")
f.seek(0) # 光标移动到开头
content = f.read() # 读取内容, 此时光标移动到结尾
print(content)
f.seek(0) # 再次将光标移动到开头
f.seek(0, 2) # 将光标移动到结尾
content2 = f.read() # 读取内容. 什么都没有
print(content2)
f.seek(0) # 移动到开头
f.write("张国荣") # 写入信息. 此时光标在9 中文3 * 3个 = 9
f.flush()
f.close()　
tell()
```

**tell()**

- 作用：获取当前光标在什么位置

- ```python
   f = open("小娃娃", mode="r+", encoding="utf-8")
   f.seek(0) # 光标移动到开头
   content = f.read() # 读取内容, 此时光标移动到结尾
   print(content)
   f.seek(0) # 再次将光标移动到开头
   f.seek(0, 2) # 将光标移动到结尾
   content2 = f.read() # 读取内容. 什么都没有
   print(content2)
   f.seek(0) # 移动到开头
   f.write("张国荣") # 写入信息. 此时光标在9 中⽂文3 * 3个 = 9
   print(f.tell()) # 光标位置9
   f.flush()
   f.close()
   
   其实文件操作里面还有很多的方法，只是使用的频率不是太高，我们想去了解可以看看源码
   ```

**修改文件**

with   open() as f,open() as f1:

```python
需求：
文件修改: 只能将文件中的内容读取到内存中, 将信息修改完毕, 然后将源文件删除, 将新文件的名字改成老文件的名字.
import os
with open("../path1/小娃娃", mode="r", encoding="utf-8") as f1,\
open("../path1/小娃娃_new", mode="w", encoding="UTF-8") as f2:
    content = f1.read()
    new_content = content.replace("冰糖葫芦", "⼤白梨")
    #把旧文件中的"冰糖葫芦"修改成"大白梨".
    f2.write(new_content)
os.remove("../path1/小娃娃") # 删除源文件
os.rename("../path1/小娃娃_new", "小娃娃") # 重命名新文件
```

上面的代码是有弊端的，因为一次把所有的内容都加载到内存中，很容易就把内存空间撑爆了。因此我们可以一行行的去读.

```python
import os
with open("小娃娃", mode="r", encoding="utf-8") as f1,\
open("小娃娃_new", mode="w", encoding="UTF-8") as f2:
    for line in f1:
        new_line = line.replace("大白梨", "冰糖葫芦")
        f2.write(new_line)
os.remove("小娃娃") # 删除源⽂文件
os.rename("小娃娃_new", "小娃娃") # 重命名新文件
```

使用with open的好处就是：

自动关闭文件。

可以同时打开多个文件。

### 练习

```python

1.有如下文件，a1.txt，里面的内容为：
老男孩是最好的学校，
全心全意为学生服务，
只为学生未来，不为牟利。
我说的都是真的。哈哈
分别完成以下的功能：

#a,将原文件全部读出来并打印。
f = open("./a1.txt",'r',encoding="utf-8")
print (f.read())
f.close()
#b,在原文件后面追加一行内容：信不信由你，反正我信了。
f = open("./a1.txt",'a',encoding="utf-8")
f.write("\n信不信由你，反正我信了。")
f.close()
#c,将原文件全部读出来，并在后面添加一行内容：信不信由你，反正我信了。
f = open("./a1.txt",'r+',encoding="utf-8")
for i in f:
	print (i)
f.write("信不信由你，反正我信了。")
f.close()

#d,将原文件全部清空，换成下面的内容：

每天坚持一点，
每天努力一点，
每天多思考一点，
慢慢你会发现，
你的进步越来越大。

f = open("./a1.txt",'w',encoding="utf-8")
f.write("每天坚持一点，\n每天努力一点，\n每天多思考一点，\n慢慢你会发现，\n你的进步越来越大。")
f.close()


2.有如下文件，t1.txt,里面的内容为：
葫芦娃，葫芦娃，
一根藤上七个瓜
风吹雨打，都不怕，
啦啦啦啦。
我可以算命，而且算的特别准:
上面的内容你肯定是心里默唱出来的，对不对？哈哈
分别完成下面的功能：

#a,以r的模式打开原文件，利用for循环遍历文件句柄。
f = open("./t1.txt",'r',encoding="utf-8")
for i in f:
	print(i)
f.close()
#b,以r的模式打开原文件，以readlines()方法读取出来，并循环遍历 readlines(),并分析a,与b 有什么区别？深入理解文件句柄与 readlines()结果的区别。
f = open("./t1.txt",'r',encoding="utf-8")
lines = f.readlines()   #获得的是字典
for i in lines:
	print (i)
f.close()
#c,以r模式读取‘葫芦娃，’前四个字符。
f = open("./t1.txt",'r',encoding="utf-8")
print (f.read(4))
f.close()
#d,以r模式读取第一行内容，并去除此行前后的空格，制表符，换行符。
f = open("./t1.txt",'r',encoding="utf-8")
print  (f.readline().strip())
f.close()
#e,以a+模式打开文件，先追加一行：‘老男孩教育’然后在从最开始将 原内容全部读取出来。
f = open("./t1.txt",'a+',encoding="utf-8")
f.write('\n老男孩教育')
f.seek(0,0)
print (f.read())
f.close()

3.文件a.txt内容：
每一行内容分别为商品名字，价钱，个数。
apple 10 3
tesla 100000 1
mac 3000 2
lenovo 30000 3
chicken 10 3
通过代码，将其构建成这种数据类型：
[{'name':'apple','price':10,'amount':3},{'name':'tesla','price':1000000,'amount':1}......]并计算出总价钱。

res = []
f = open("./a.txt",'r',encoding="utf-8")
lines = f.readlines()
f.close()
for line in lines:
	dict = {}
	line_list = line.strip().split()  # 默认以空格为分隔符对字符串进行切片
	dict['name'] = line_list[0]
	dict['price'] = int(line_list[1])  # 读取出来的是字符
	dict['count'] = int(line_list[2])
	res.append(dict)

print (res)


4.有如下文件：
alex是老男孩python发起人，创建人。
alex其实是人妖。
谁说alex是sb？
你们真逗，alex再牛逼，也掩饰不住资深屌丝的气质。

#将文件中所有的alex都替换成大写的SB（文件的改的操作）。
with open('./a1.txt','r',encoding='utf-8') as f,open('./t1.txt','w',encoding='utf-8') as f1:
	for i in f:
		page_new = i.replace("alex","SB")
		f1.write(page_new)
os.remove("./a1.txt")


5.文件a1.txt内容(选做题)   有点东西

name:apple price:10 amount:3 year:2012
name:tesla price:100000 amount:1 year:2013
.......

通过代码，将其构建成这种数据类型：
[{'name':'apple','price':10,'amount':3,year:2012},
{'name':'tesla','price':1000000,'amount':1}......]
并计算出总价钱。

res = []
f = open('./7-8/a1.txt','r',encoding='utf-8')
for i in f:  #循环两次
	dic = {}  #这个字典的定义就是看有多少行的数据，就在哪一个循环里面定义
	for en in i.split():  #循环四次
		k,v = en.split(":")  #解构里面的每一个元素
		dic[k] = v      #把每一行的小元素添加到大字典中
	res.append(dic)
print (res)


6.文件a1.txt内容(选做题)    有点东西
序号 部门 人数 平均年龄 备注
1 python 30 26 单身狗
2 Linux 26 30 没对象
3 运营部 20 24 女生多
.......

通过代码，将其构建成这种数据类型：
[{'序号':'1','部门':Python,'人数':30,'平均年龄':26,'备注':'单身狗'},
......]
'''
res = []
f = open('./7-8/a1.txt','r',encoding='utf-8')
lst = f.readline().split()  #先把第一行读出来
for i in f:     #循环3次，有3行数据
	dic = {}   #在外循环下面创建空字典。
	for num in range(len(lst)):  #不要使用迭代源数据的方式，这种使用下标的方式简便
		dic[lst[num]] = i.split()[num]  #把下面的几行数据和第一行的数据一一对应。添加到字典中
	res.append(dic)
print (res)
```