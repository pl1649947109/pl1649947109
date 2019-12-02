---
title: python常用的内置函数之方法大全
id: 1
date: 2019-7-20 20:00:00
tags: python部分内容详讲
comment: true
﻿declare: true
---

导读：写这篇文章的目的一个是为了以后使用模块的时候方便去查询它的方法;也是为了记录一些不常用的方法，但是在特定的工作中的时候有些不常用的方法能发挥很大的作用。

### 内容大纲

- time 和datetime
- random
- json 和pickle
- os
- sys
- hashlib
- collections

<!-----more----->

### time 和datetime

```python
经常使用的：
import time
from datetime import datetime,timedelta

 #获取当前时间的时间戳  一个浮点数
print (time.time()) 
#1564041048.1111746

#睡眠 阻塞
time.sleep(1)  

#获取当前时间，并格式化输出
print(time.strftime("%Y-%m-%d %H:%M:%S"))
#2019-07-25 15:52:32

#获取当前时间的结构化时间（格林尼治时间） 数据类型是命名元组
print (time.gmtime())  
#time.struct_time(tm_year=2019, tm_mon=7, tm_mday=25, tm_hour=7, tm_min=53, tm_sec=15, tm_wday=3, tm_yday=206, tm_isdst=0)

#将时间戳转化为字符串时间（时间戳>结构化时间>字符串时间）
print (time.strftime("%Y-%m-%d %H:%M:%S",time.gmtime(1564028611.631374)))
#2019-07-25 04:23:31

#将字符串时间转化成时间戳（字符串时间>结构化时间>时间戳）
print (time.mktime(time.strptime("2024-3-16 12:30:30","%Y-%m-%d %H:%M:%S")))
#1710563430.0

#获取当前时间  类型的是一个<class 'datetime.datetime'>对象
print (datetime.now())  
#2019-07-25 15:57:38.599890

#转化格式
print(datetime(2019, 5, 20, 15, 14, 00)) 
#2019-05-20 15:14:00

#将当前时间转化成时间戳
print (datetime.now().timestamp())
#1564041586.28922

#将时间戳转化成当前时间
print (datetime.fromtimestamp(13543443435))
#2399-03-06 03:37:15

#将字符串转化成象
print (type(datetime.strptime("2019-10-10 22:23:24","%Y-%m-%d %H:%M:%S")))
#<class 'datetime.datetime'>

#将对象转化成字符串
print (type(datetime.now().strftime("%Y-%m-%d %H:%M:%S")))
#<class 'str'>

#datetime加
print(datetime.now() + timedelta(hours=30*24*12))
#2020-07-19 16:05:25.403830
#datetime减
print(datetime.now() - timedelta(hours=30 * 24 * 12))
#2018-07-30 16:06:14.273110

补充：不常使用的方法：
#这将返回本地DST时区偏差
>>> time.altzone
-32400

#返回这种格式的字符串时间
>>> time.asctime()
'Thu Aug  1 21:39:49 2019'

#返回的也是当前的时间，但是不同的是，它比time.time更加精确
>>> time.clock()
69.2107448191019
#注：time.clock()是统计cpu时间 的工具，这在统计某一程序或函数的执行速度最为合适。两次调用time.clock()函数的插值即为程序运行的cpu时间。

#返回这种格式的字符串时间
>>> time.ctime()
'Thu Aug  1 21:44:56 2019'

>>> time.localtime()
time.struct_time(tm_year=2019, tm_mon=8, tm_mday=1, tm_hour=21, tm_min=47, tm_sec=11, tm_wday=3, tm_yday=213, tm_isdst=0)
```

### random

```python
import random
#0~1  浮点
print (random.random())  
# 0.8059617097306809

#1-10  浮点
print (random.uniform(1,10)) 
# 1.8943544182817131

#1-50
print (random.randint(1,50)) 
#22

#(起始，终止，步长)
print (random.randrange(1,30,2))  
#5

#随机选择一个元素
print(random.choice([1,2,3,563,13,654,6413,1,3,131]))  
#131

#随机选择多个元素，会有重复，返回列表
print (random.choices([1,51,1313,15,331,321,1313],k=2)) 
#[1313, 331]

#随机选择多个元素，不会有重复，返回列表
print(random.sample((12,22,31,12,212,1,231,212,12,12,1,2,12,1,21,),k=3))
#[231, 1, 12]

# 顺序打乱
lst = [1,2,3,4,5,6,7,8,9,0]
random.shuffle(lst)  
print (lst)

不常用
#seed()方法改变随机数的种子，可以在调用其他随机模块函数之前调用次函数
>>> random.seed(20)
>>> random.random()
0.9056396761745207
>>> random.seed(20)
>>> random.random()
0.9056396761745207

#从列表里面随机选择5个元素并以列表的形式返回
>>> random.sample([1,2,3,4,5,6,7,8,9,10],5)
[3, 5, 2, 7, 10]

#返回一个当前生成器的内部状态对象
random.getstate()

#传入一个先前利用getstate方法获得的状态对象，使得生成器恢复到这个状态
random.setstate(state)

#返回一个不大于K位的Python整数（十进制），比如k=10，则结果在0~2^10之间的整数
random.getrandbits(k)

#返回一个low <= N <=high的三角形分布的随机数。参数mode指明众数出现位置。
random.triangular(low, high, mode)

#β分布。返回的结果在0~1之间
random.betavariate(alpha, beta)

#指数分布
random.expovariate(lambd)

#伽马分布
random.gammavariate(alpha, beta)

#高斯分布
random.gauss(mu, sigma)

#对数正态分布
random.lognormvariate(mu, sigma)

#正态分布
random.normalvariate(mu, sigma)

#卡帕分布
random.vonmisesvariate(mu, kappa)

#帕累托分布
random.paretovariate(alpha)
```

### json 和pickle

```python
import json
import pickle
json：
序列化与反序列化(2组4个方法）
dumps和loads操作字符串
dumps：将对象序列化（转换成）字符串
loads：将字符串反序列化成对象本身
	lst = [1,2,55,6,3,32345,45,45]
	lst1 = json.dumps(lst)
	print (lst1,type(lst1))
	lst2 = json.loads(lst1)
	print (lst2,type(lst2))
	#[1, 2, 55, 6, 3, 32345, 45, 45] <class 'str'>
	#[1, 2, 55, 6, 3, 32345, 45, 45] <class 'list'>
	dic = {"1":"中国","2":22,"3":3}
	dic1 = json.dumps(dic)
	print (dic1,type(dic1))
	#关闭ascll码,不然中文转不了
	dic1_1 = json.dumps(dic,ensure_ascii=False)
	print(dic1_1, type(dic1_1))
	dic2 = json.loads(dic1)
	print (dic2,type(dic2))
	#{"1": "\u4e2d\u56fd", "2": 22, "3": 3} <class 'str'>
	#{"1": "中国", "2": 22, "3": 3} <class 'str'>
	#{'1': '中国', '2': 22, '3': 3} <class 'dict'>

dump和load（操作文件）
dump:将对象序列化（转换成）字符串类型
	load：将文件中的
	lst = [1,2,5,23,6,555,56,4,64,15,4,'asd','中文']
	dic = {"1":"中国","2":22,"3":3}
	f = open('info','w',encoding='utf-8')
	# json.dump(lst,f,ensure_ascii=False)
	json.dump(dic, f, ensure_ascii=False)
	f.close()
load:将字符串类型反序列化成对象
	f1 = open('info','r',encoding='utf-8')
	# lst1 = json.load(f1)
	#只能是一行数据
	# dic1 = json.load(f1)
	#当数据是多行的时候，这么处理
	for  i in f1:
		dic1 = json.loads(i)
		print(dic1)
	f1.close()
	# print (lst)

pickle：序列化（pytohn所有对象进行转换）--python自带的，只有python自己可以用
也是两组4个方法（dumps/loads/dump/load）用法和json一样，不过支持python的所有的数据类型
dumps和loads
	#列表，字符串
	lst = [1,2,46,13,"ss",'中国']
	lst1 = pickle.dumps(lst)
	print (lst1,type(lst1))
#b'\x80\x03]q\x00(K\x01K\x02K.K\rX\x02\x00\x00\x00ssq\x01X\x06\x00\x00\x00\xe4\xb8\xad\xe5\x9b\xbdq\x02e.' <class 'bytes'>
	print (pickle.loads(lst1))
	#字典
	dic = {"user":"郭宝元"}
    t_list = pickle.dumps(dic) # 转换成类似字节
    print(t_list)
    print(pickle.loads(t_list))
	#函数
    def func():
        print(111)

    import json
    fun = json.dumps(func)
    print(fun)

	fun = pickle.dumps(func)
    print(fun)
    pickle.loads(fun)()
         
dump和load
	dic = {"usern":"baoyuian"}
	dic = {"usern":"宝元"}
	pickle.dump(dic,open("info","wb"))
	print(pickle.load(open("info","rb")))

	import pickle
	dic = {"user":"123"}
	pickle.dump(dic,open("info","ab"))

	import pickle
	dic = {"1":2}
	f = open("info","wb")
	s = "\n".encode("utf-8")
	f.write(pickle.dumps(dic)+ s)
	f.write(pickle.dumps(dic)+ s)
	f.write(pickle.dumps(dic)+ s)
	f.close()

	f1 = open("info","rb")
	for i in f1:
	    print(pickle.loads(i))
总结：推荐使用json（因为json在各种语言中是通用的）
```

### os

```python
	#工作目录
    #查看当前路径  ***
	print (os.getcwd()) 
	C:\Users\pl\PycharmProjects\笔记\作业
        
    #切换路径  ***
	os.chdir("path") 
    
    #.
	print(os.curdir) 
    
    #..
	print(os.pardir)   
    
	#文件夹
    #创建文件夹    ***
	os.mkdir("文件路径") 
    
    #删除文件夹    ***
	os.rmdir("文件路径") 
    
     #递归创建文件夹，就是文件夹的嵌套    ***
	os.makedirs("tt/ss/nn") 
    
    #递归删除文件夹（空的就删掉，不空就停掉）    ***
	os.removedirs("tt/ss/nn")  
    
    #获取当前文件夹下的所有文件名    ***
	print (os.listdir("F://blog"))  
    
	#文件
    #修改文件名称    ***
	os.rename("旧名称","新名称") 
    
    #删除文件        ***
	os.remove("文件")  
    
	#路径
    #通过相对路径返回绝对路径    ***
	print (os.path.abspath("三国.txt")) 
    
    #返回元组，切最后一个反斜杠
	print(os.path.split(os.path.abspath("三国.txt")))  
	C:\Users\pl\PycharmProjects\笔记\作业\三国.txt
	('C:\\Users\\pl\\PycharmProjects\\笔记\\笔作业', '国.txt')
    
    #获取文件路径    ***
	print(os.path.dirname(r"C:\Users\pl\PycharmProjects\笔记\作业\三国.txt"))
    
    #判断文件路径是否存在    ***
	print(os.path.exists("F:"))  
    
    #判断是不是路径    ***
	print(os.path.isdir("E:")) 
    
    #判断是不是文件    ***
	print (os.path.isfile("三国.txt")) 
    
    #路径拼接    ****
	print(os.path.join("F:\\","pl","baba"))  
    
    #返回时间戳，最后的修改时间
	print (os.path.getatime("F:blog"))  
    
    #获取文件名
    print (os.path.basename("三国.txt"))
    
     #判断是不是绝对路径
    print (os.path.isabs("")) 
    
    #返回文件的大小，但是不准    ***
	print(os.path.getsize("文件路径"))  
    
 补充：
 程序3种死亡方式：
     自然死完：就是函数中的return 0.
     自杀：一般都是自己os请求将自己毙掉。
     被杀：它杀往往是由自己的至亲完成的，通常及时它的父母。
     #这几种方法都是自杀调用的方法
     os.absot() #默认的程序结束函数
     os.exit() #附加了关闭文件与返回状态码执行给环境
     os.assert()#为oc中的宏，只在debug模式下有用，当条件不成立，程序就终止
```

| 1    | [os.access(path, mode)](https://www.runoob.com/python/os-access.html) 检验权限模式 |
| ---- | ------------------------------------------------------------ |
| 2    | [os.chdir(path)](https://www.runoob.com/python/os-chdir.html) 改变当前工作目录 |
| 3    | [os.chflags(path, flags)](https://www.runoob.com/python/os-chflags.html) 设置路径的标记为数字标记。 |
| 4    | [os.chmod(path, mode)](https://www.runoob.com/python/os-chmod.html) 更改权限 |
| 5    | [os.chown(path, uid, gid)](https://www.runoob.com/python/os-chown.html) 更改文件所有者 |
| 6    | [os.chroot(path)](https://www.runoob.com/python/os-chroot.html) 改变当前进程的根目录 |
| 7    | [os.close(fd)](https://www.runoob.com/python/os-close.html) 关闭文件描述符 fd |
| 8    | [os.closerange(fd_low, fd_high)](https://www.runoob.com/python/os-closerange.html) 关闭所有文件描述符，从 fd_low (包含) 到 fd_high (不包含), 错误会忽略 |
| 9    | [os.dup(fd)](https://www.runoob.com/python/os-dup.html) 复制文件描述符 fd |
| 10   | [os.dup2(fd, fd2)](https://www.runoob.com/python/os-dup2.html) 将一个文件描述符 fd 复制到另一个 fd2 |
| 11   | [os.fchdir(fd)](https://www.runoob.com/python/os-fchdir.html) 通过文件描述符改变当前工作目录 |
| 12   | [os.fchmod(fd, mode)](https://www.runoob.com/python/os-fchmod.html) 改变一个文件的访问权限，该文件由参数fd指定，参数mode是Unix下的文件访问权限。 |
| 13   | [os.fchown(fd, uid, gid)](https://www.runoob.com/python/os-fchown.html) 修改一个文件的所有权，这个函数修改一个文件的用户ID和用户组ID，该文件由文件描述符fd指定。 |
| 14   | [os.fdatasync(fd)](https://www.runoob.com/python/os-fdatasync.html) 强制将文件写入磁盘，该文件由文件描述符fd指定，但是不强制更新文件的状态信息。 |
| 15   | [os.fdopen(fd[, mode[, bufsize\]])](https://www.runoob.com/python/os-fdopen.html) 通过文件描述符 fd 创建一个文件对象，并返回这个文件对象 |
| 16   | [os.fpathconf(fd, name)](https://www.runoob.com/python/os-fpathconf.html) 返回一个打开的文件的系统配置信息。name为检索的系统配置的值，它也许是一个定义系统值的字符串，这些名字在很多标准中指定（POSIX.1, Unix 95, Unix 98, 和其它）。 |
| 17   | [os.fstat(fd)](https://www.runoob.com/python/os-fstat.html) 返回文件描述符fd的状态，像stat()。 |
| 18   | [os.fstatvfs(fd)](https://www.runoob.com/python/os-fstatvfs.html) 返回包含文件描述符fd的文件的文件系统的信息，像 statvfs() |
| 19   | [os.fsync(fd)](https://www.runoob.com/python/os-fsync.html) 强制将文件描述符为fd的文件写入硬盘。 |
| 20   | [os.ftruncate(fd, length)](https://www.runoob.com/python/os-ftruncate.html) 裁剪文件描述符fd对应的文件, 所以它最大不能超过文件大小。 |
| 21   | [os.getcwd()](https://www.runoob.com/python/os-getcwd.html) 返回当前工作目录 |
| 22   | [os.getcwdu()](https://www.runoob.com/python/os-getcwdu.html) 返回一个当前工作目录的Unicode对象 |
| 23   | [os.isatty(fd)](https://www.runoob.com/python/os-isatty.html) 如果文件描述符fd是打开的，同时与tty(-like)设备相连，则返回true, 否则False。 |
| 24   | [os.lchflags(path, flags)](https://www.runoob.com/python/os-lchflags.html) 设置路径的标记为数字标记，类似 chflags()，但是没有软链接 |
| 25   | [os.lchmod(path, mode)](https://www.runoob.com/python/os-lchmod.html) 修改连接文件权限 |
| 26   | [os.lchown(path, uid, gid)](https://www.runoob.com/python/os-lchown.html) 更改文件所有者，类似 chown，但是不追踪链接。 |
| 27   | [os.link(src, dst)](https://www.runoob.com/python/os-link.html) 创建硬链接，名为参数 dst，指向参数 src |
| 28   | [os.listdir(path)](https://www.runoob.com/python/os-listdir.html) 返回path指定的文件夹包含的文件或文件夹的名字的列表。 |
| 29   | [os.lseek(fd, pos, how)](https://www.runoob.com/python/os-lseek.html) 设置文件描述符 fd当前位置为pos, how方式修改: SEEK_SET 或者 0 设置从文件开始的计算的pos; SEEK_CUR或者 1 则从当前位置计算; os.SEEK_END或者2则从文件尾部开始. 在unix，Windows中有效 |
| 30   | [os.lstat(path)](https://www.runoob.com/python/os-lstat.html) 像stat(),但是没有软链接 |
| 31   | [os.major(device)](https://www.runoob.com/python/os-major.html) 从原始的设备号中提取设备major号码 (使用stat中的st_dev或者st_rdev field)。 |
| 32   | [os.makedev(major, minor)](https://www.runoob.com/python/os-makedev.html) 以major和minor设备号组成一个原始设备号 |
| 33   | [os.makedirs(path[, mode\])](https://www.runoob.com/python/os-makedirs.html) 递归文件夹创建函数。像mkdir(), 但创建的所有intermediate-level文件夹需要包含子文件夹。 |
| 34   | [os.minor(device)](https://www.runoob.com/python/os-minor.html) 从原始的设备号中提取设备minor号码 (使用stat中的st_dev或者st_rdev field )。 |
| 35   | [os.mkdir(path[, mode\])](https://www.runoob.com/python/os-mkdir.html) 以数字mode的mode创建一个名为path的文件夹.默认的 mode 是 0777 (八进制)。 |
| 36   | [os.mkfifo(path[, mode\])](https://www.runoob.com/python/os-mkfifo.html) 创建命名管道，mode 为数字，默认为 0666 (八进制) |
| 37   | [os.mknod(filename[, mode=0600, device\])](https://www.runoob.com/python/os-mknod.html) 创建一个名为filename文件系统节点（文件，设备特别文件或者命名pipe）。 |
| 38   | [os.open(file, flags[, mode\])](https://www.runoob.com/python/os-open.html) 打开一个文件，并且设置需要的打开选项，mode参数是可选的 |
| 39   | [os.openpty()](https://www.runoob.com/python/os-openpty.html) 打开一个新的伪终端对。返回 pty 和 tty的文件描述符。 |
| 40   | [os.pathconf(path, name)](https://www.runoob.com/python/os-pathconf.html) 返回相关文件的系统配置信息。 |
| 41   | [os.pipe()](https://www.runoob.com/python/os-pipe.html) 创建一个管道. 返回一对文件描述符(r, w) 分别为读和写 |
| 42   | [os.popen(command[, mode[, bufsize\]])](https://www.runoob.com/python/os-popen.html) 从一个 command 打开一个管道 |
| 43   | [os.read(fd, n)](https://www.runoob.com/python/os-read.html) 从文件描述符 fd 中读取最多 n 个字节，返回包含读取字节的字符串，文件描述符 fd对应文件已达到结尾, 返回一个空字符串。 |
| 44   | [os.readlink(path)](https://www.runoob.com/python/os-readlink.html) 返回软链接所指向的文件 |
| 45   | [os.remove(path)](https://www.runoob.com/python/os-remove.html) 删除路径为path的文件。如果path 是一个文件夹，将抛出OSError; 查看下面的rmdir()删除一个 directory。 |
| 46   | [os.removedirs(path)](https://www.runoob.com/python/os-removedirs.html) 递归删除目录。 |
| 47   | [os.rename(src, dst)](https://www.runoob.com/python/os-rename.html) 重命名文件或目录，从 src 到 dst |
| 48   | [os.renames(old, new)](https://www.runoob.com/python/os-renames.html) 递归地对目录进行更名，也可以对文件进行更名。 |
| 49   | [os.rmdir(path)](https://www.runoob.com/python/os-rmdir.html) 删除path指定的空目录，如果目录非空，则抛出一个OSError异常。 |
| 50   | [os.stat(path)](https://www.runoob.com/python/os-stat.html) 获取path指定的路径的信息，功能等同于C API中的stat()系统调用。 |
| 51   | [os.stat_float_times([newvalue\])](https://www.runoob.com/python/os-stat_float_times.html) 决定stat_result是否以float对象显示时间戳 |
| 52   | [os.statvfs(path)](https://www.runoob.com/python/os-statvfs.html) 获取指定路径的文件系统统计信息 |
| 53   | [os.symlink(src, dst)](https://www.runoob.com/python/os-symlink.html) 创建一个软链接 |
| 54   | [os.tcgetpgrp(fd)](https://www.runoob.com/python/os-tcgetpgrp.html) 返回与终端fd（一个由os.open()返回的打开的文件描述符）关联的进程组 |
| 55   | [os.tcsetpgrp(fd, pg)](https://www.runoob.com/python/os-tcsetpgrp.html) 设置与终端fd（一个由os.open()返回的打开的文件描述符）关联的进程组为pg。 |
| 56   | [os.tempnam([dir[, prefix\]])](https://www.runoob.com/python/os-tempnam.html) 返回唯一的路径名用于创建临时文件。 |
| 57   | [os.tmpfile()](https://www.runoob.com/python/os-tmpfile.html) 返回一个打开的模式为(w+b)的文件对象 .这文件对象没有文件夹入口，没有文件描述符，将会自动删除。 |
| 58   | [os.tmpnam()](https://www.runoob.com/python/os-tmpnam.html) 为创建一个临时文件返回一个唯一的路径 |
| 59   | [os.ttyname(fd)](https://www.runoob.com/python/os-ttyname.html) 返回一个字符串，它表示与文件描述符fd 关联的终端设备。如果fd 没有与终端设备关联，则引发一个异常。 |
| 60   | [os.unlink(path)](https://www.runoob.com/python/os-unlink.html) 删除文件路径 |
| 61   | [os.utime(path, times)](https://www.runoob.com/python/os-utime.html) 返回指定的path文件的访问和修改的时间。 |
| 62   | [os.walk(top[, topdown=True[, onerror=None[, followlinks=False\]]])](https://www.runoob.com/python/os-walk.html) 输出在文件夹中的文件名通过在树中游走，向上或者向下。 |
| 63   | [os.write(fd, str)](https://www.runoob.com/python/os-write.html) 写入字符串到文件描述符 fd中. 返回实际写入的字符串长度 |
| 64   | [os.path 模块](https://www.runoob.com/python/python-os-path.html) 获取文件的属性信息。 |

### sys

```python
和python解释器交互的接口

#返回当前文件的路径  很有用    ***
print (sys.argv)  

#状态码，程序中间退出，arg=0为正常退出
print (sys.exit([arg])) 

#返回版本号
print (sys.version)

#添加自定义模块查找路径，解释器的   ***
print (sys.path)  
>>> sys.path
['', 'F:\\大三下期学习的软件\\vs\\sdk和工具\\Python36_64\\python36.zip', 'F:\\大三下期学习的软件\\vs\\sdk和工具\\Python36_64\\DLLs', 'F:\\大三下期学习的软件\\vs\\sdk和工具\\Python36_64\\lib', 'F:\\大三下期学习的软件\\vs\\sdk和工具\\Python36_64', 'F:\\大三下期学习的软件\\vs\\sdk和工具\\Python36_64\\lib\\site-packages']

#返回解释器的系统信息    *** 区分操作系统然后进行相关的逻辑操作
print(sys.platform) 

#获取系统当前编码，一般默认为ascii。
sys.getdefaultencoding()

#设置系统默认编码，执行dir（sys）时不会看到这个方法，在解释器中执行不通过，可以先执行reload(sys)，在执行 setdefaultencoding('utf8')，此时将系统默认编码设置为utf8。
sys.setdefaultencoding()

#获取文件系统使用编码方式，Windows下返回'mbcs'，mac下返回'utf-8'.
sys.getfilesystemencoding()

#stdin , stdout , 以及stderr 变量包含与标准I/O 流对应的流对象. 如果需要更好地控制
#输出,而print 不能满足你的要求, 它们就是你所需要的. 你也可以替换它们, 这时候你就可以重定向#输出和输入到其它设备( device ), 或者以非标准的方式处理它们
sys.stdin,sys.stdout,sys.stderr
```

### hashlib

```python
import hashlib
加密:将加密的内容转换成字节
	md5:32位长
	sha1:40位长（其他的同理）
	md5 = hashlib.md5()  #步骤一：初始化
	#sha1 = hashlib.sha1()
	#sha1.update("pl123".encode('utf-8'))
	md5.update("pl123".encode('gbk'))  #步骤二:放入内容
	print (md5.hexdigest())  #步骤三：做转换
	(可见加密编码的方式没关系都是一样的，只要内容是一样的)
	#>>>14e44c50482e8c85caf7a481ae581f81
注：最常用的是md5，平时加密的时候使用sha1

多重加密：叫做加盐
加盐：
	固定加盐，可以把这个固定的内容改成动态的
	md5 = hashlib.md5("pl".encode('utf-8'))  #给一个key
	md5.update("123456".encode('utf-8'))  #加密内容
	print (md5.hexdigest())
    
	动态加盐
	user = input("username:")
	pwd = input("password")
	md5 = hashlib.md5(user.encode("utf-8"))  
	md5.update(pwd.encode("utf-8"))
	print(md5.hexdigest())
    
文件的一致性校验
需求：我们在安装python解释器的时候,在安装python解释器的时候计算本地的md5值是否一致,一致安装,不一致的删除.
	def file_check(file_path):
		with open(file_path, mode='rb') as f1:
			sha256 = hashlib.md5()
			while 1:
				content = f1.read(1024)
				if content:
					sha256.update(content)
				else:
					return sha256.hexdigest()
	print(file_check('python-3.6.6-amd64.exe'))

补充：
什么事摘要算法？
摘要算法又称作哈希算法，散列算法。它是通过一个函数，把任意长度固定的数据串（通常是16进制的字符串表示）用于加密相关的操作。
```

### collections

```python
在python内置数据类型（num,bool,str,dict、list、set、tuple）的基础上，collections模块还提供了几个额外的数据类型：Counter、deque、defaultdict、namedtuple和OrderedDict等。
1.namedtuple: 生成可以使用名字来访问元素内容的tuple
2.deque: 双端队列，可以快速的从另外一侧追加和推出对象
3.Counter: 计数器，主要用来计数
4.OrderedDict: 有序字典
5.defaultdict: 带有默认值的字典

#namedtuple
我们知道tuple可以表示不变数据，例如，一个点的二维坐标就可以表示成：p = (1, 2)
但是，看到(1, 2)，很难看出这个tuple是用来表示一个坐标的。这时，namedtuple就派上了用场：
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)
print(p)
结果：Point(x=1, y=2)
类似的，如果要用坐标和半径表示一个圆，也可以用namedtuple定义：
namedtuple('名称', [属性list]):
Circle = namedtuple('Circle', ['x', 'y', 'r'])

#deque
使用list存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为list是线性存储，数据量大的时候，插入和删除效率很低。
deque是为了高效实现插入和删除操作的双向列表，适合用于队列和栈：
from collections import deque
q = deque(['a', 'b', 'c'])
q.append('x')
q.appendleft('y')
q = deque(['y', 'a', 'b', 'c', 'x'])
deque除了实现list的append()和pop()外，还支持appendleft()和popleft()，这样就可以非常高效地往头部添加或删除元素。
补充：
方法：
Append():从队列的右侧添加元素
Appendleft():从队列的左侧添加元素
Clear():清空队列
Count():统计队列中元素的个数
Extend():从队列的右侧扩展，就是从队列的右侧添加多个元素
Extendleft()：从队列左侧扩展
Index():索引指定元素的位置
Insert():在队列的任意位置插入值
Pop():从队列的右侧移除值
Poplleft():从队列的左侧移除值
Remove():移除指定的值
Revere():将队列中的元素反转
Rotate():移动队列中的元素，若n<0,则将队列最左侧的元素依次移动到右侧。反之
Empty():判断队列是否为空，返回布尔值
Full():判断队列是否以满，返回布尔值
Put():往队列中放一个元素
Get():从队列中取一个元素，依据先进先出的原则
put_nowait():无阻塞的向队列中添加元素，若队列已满，不等待直接报错
get_nowait()：无阻塞从队列中获取元素，若队列为空，不等待直接报错（empty）
Qsize()：表示队列长度，即元素个数
Join()：阻塞调用线程，直到队列中的所有任务都被处理完成，与task_done配合使用

#OrderedDict
使用dict时，Key是无序的。在对dict做迭代时，我们无法确定Key的顺序。
如果要保持Key的顺序，可以用OrderedDict：
from collections import OrderedDict
d = dict([('a', 1), ('b', 2), ('c', 3)]) # 另一种定义字典的方式
print(d)
# 结果:
{'a': 1, 'c': 3, 'b': 2}
od = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
print(od)
# 结果:
OrderedDict([('a', 1), ('b', 2), ('c', 3)])
注意，OrderedDict的Key会按照插入的顺序排列，不是Key本身排序：

>>> od = OrderedDict()
>>> od['z'] = 1
>>> od['y'] = 2
>>> od['x'] = 3
>>> od.keys() # 按照插入的Key的顺序返回
['z', 'y', 'x']
补充：
Clear()：清空字典
Popitem():有序删除，类似于栈，按照先进先出的顺序依次删除
Pop():删除指定的键值对
Move_to_end():将指定的键值对移到字典的最后面
Setdefult()：设置指定值，默认为None
Update():更新字典，有则更新，无则添加

#defaultdict
有如下值集合 [11,22,33,44,55,66,77,88,99,90...]，将所有大于 66 的值保存至字典的第一个key中，将小于 66 的值保存至第二个key的值中。
即： {'k1': 大于66 , 'k2': 小于66}
li = [11,22,33,44,55,77,88,99,90]
result = {}
for row in li:
    if row > 66:
        if 'key1' not in result:
            result['key1'] = []
        result['key1'].append(row)
    else:
        if 'key2' not in result:
            result['key2'] = []
        result['key2'].append(row)
print(result)
from collections import defaultdict
values = [11, 22, 33,44,55,66,77,88,99,90]
my_dict = defaultdict(list)
for value in  values:
    if value>66:
        my_dict['k1'].append(value)
    else:
        my_dict['k2'].append(value)
使用dict时，如果引用的Key不存在，就会抛出KeyError。如果希望key不存在时，返回一个默认值，就可以用defaultdict：
from collections import defaultdict
dd = defaultdict(lambda: 'N/A')
dd['key1'] = 'abc'
 # key1存在
print(dd['key1'])
dd['key2'] # key2不存在，返回默认值
print(dd['key2'])

#Counter
Counter类的目的是用来跟踪值出现的次数。它是一个无序的容器类型，以字典的键值对形式存储，其中元素作为key，其计数作为value。计数值可以是任意的Interger（包括0和负数）。Counter类和其他语言的bags或multisets很相似。
c = Counter('abcdeabcdabcaba')
print c
输出：Counter({'a': 5, 'b': 4, 'c': 3, 'd': 2, 'e': 1})
```

