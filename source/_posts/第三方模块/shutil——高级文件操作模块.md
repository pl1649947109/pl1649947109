---
title: shutil——高级文件操作模块
id: 10
date: 2019-8-126 20:30:00
tags: 第三方模块
comment: true
---

介绍：该模块对文件和文件集合提供了许多高级操作。特别是，提供了支持文件复制和删除的功能。单个文件的操作，请参考os模块。

注意：高级别的文件的复制的功能，也无法复制所有文件元数据。

### 文件夹和文件操作

```python
shutil.copyfileobj(src,dst,length=16*1024):
功能：将src文件内容复制到dst文件，length为src每次读取的长度，用作缓冲区大小。
参数：
fsrc： 源文件
fdst： 复制至fdst文件
length： 缓冲区大小，即fsrc每次读取的长度
实例:
import shutil
f1 = open('file.txt','r')
f2 = open('filel_copy.txt','a+')
shutil.copyfileobj(f1,f2,length=1024)
```

<!----more---->

```python
shutil.copyfile(src,dst):
功能：将src文件内容复制到dst文件。
参数：
src： 源文件路径
dst： 复制到dst文件，若dst文件不存在，将会生成一个dst文件；若存在将会被覆盖
实例：
import shutil
shutil.copyfile('file.txt','file_copy.txt')
```

```python
shuilt.copymode(src,dst):
功能：将src文件权限复制到dst文件，文件内容，所有组和组不受影响
参数：
src： 源文件路径
dst： 将权限复制至dst文件，dst路径必须是真实的路径，并且文件必须存在，否则将会报文件找不到错误
实例：
import shutil
shutil.copymode('file.txt','file_copy.txt')
```

```python
shutil.copytat(src,dst):
功能：将权限，上次访问时间，上次修改时间以及src的标志复制到dst。文件内容，所有者和组不受影响
参数：
src： 源文件路径
dst： 将权限复制至dst文件，dst路径必须是真实的路径，并且文件必须存在，否则将会报文件找不到错误
实例：
import shutil 
shutil.copystat('file.txt','file_copy.txt')
```

```python
shuilt.copy(src,dst):
功能：将文件src复制到dst。dst可以是个目录，会在该目录下创建与src同名的文件，若该目录下存在同名文件，将会报错提示已经存在同名文件。权限会被一并复制本质是先后调用了copyfile和copymode。
参数：
src：源文件路径
dst：复制至dst文件夹或文件
实例：
import shutil
import os
shutil.copy('file.txt','file_copy.txt')

#切换到指定的路径，然后复制到切换的地址
shutil.copy('file.txt',os.path.join(os.getcwd(),'copy'))
```

```python
shuilt.copy2(src,dst):
功能：copy2(src, dst)： 将文件src复制至dst。dst可以是个目录，会在该目录下创建与src同名的文件，若该目录下存在同名文件，将会报错提示已经存在同名文件。权限、上次访问时间、上次修改时间和src的标志会一并复制至dst。本质是先后调用了copyfile与copystat方法而已
参数：
src：源文件路径
dst：复制至dst文件夹或文件
实例：
import shutil 
import os
shutil.copy('file.txt','file_copy.txt')

#切换到指定的路径，然后复制到切换的地址
shutil.copy('file.txt',os.path.join(os.getcwd(),'copy'))
```

```python
shuilt.copytree(src,dst,symlinks=False,ignore=None):
功能：拷贝文档树，将src文件夹里面的所有内容拷贝至dst文件夹。
参数：
src：源文件夹
dst：复制至dst文件夹，该文件夹会自动创建，需保证此文件夹不存在，否则将报错
symlinks：是否复制软连接，True复制软连接，False不复制，软连接会被当成文件复制过来，默认False
ignore：忽略模式，可传入ignore_patterns()
实例：
import shutil
import os
folder1 = os.path.join(os.getcwd(),'aaa')
#bbb和ccc文件夹都可以不存在，会自动创建
folder2 = os.path.join(os.getcwd(),'bbb','ccc')
#将'abc.txt'，'bcd.txt'忽略，不复制，是
shutil.copytree(folder1,folder2,ignore=shutil.ignore_patterns('abc.txt','bcd.txt'))
```

```python
shuilt.rmtree(path,ignore_errors=False,onerror=None):
功能：移除文件树，将文件夹目录删除
参数：
ignore_errors：是否忽略错误，默认False
onerror：定义错误处理函数，需传递一个可执行的处理函数，该处理函数接收三个参数：函数、路径和excinfo
实例：
import shutil
folder1 = os.path.join(os.getcwd(),'aaa')
shutil.rmtree(folder1)
```

```python
shutil.move(src,dst):
功能：将src移动到dst目录下，若dst目录不存在，则效果等同于src改名为dst。若dst目录存在，将会把src文件夹的所有内容移动到目录下面。
参数：
src：源文件夹或文件
dst：移动至dst文件夹，或将文件改名为dst文件。如果src为文件夹，而dst为文件将会报错
实例：
import shutil 
import os
# 示例一，将src文件夹移动至dst文件夹下面，如果bbb文件夹不存在，则变成了重命名操作
folder1 = os.path.join(os.getcwd(),"aaa")
folder2 = os.path.join(os.getcwd(),"bbb")
shutil.move(folder1, folder2)
# 示例二，将src文件移动至dst文件夹下面，如果bbb文件夹不存在，则变成了重命名操作
file1 = os.path.join(os.getcwd(),"aaa.txt")
folder2 = os.path.join(os.getcwd(),"bbb")
shutil.move(file1, folder2)
# 示例三，将src文件重命名为dst文件(dst文件存在，将会覆盖)
file1 = os.path.join(os.getcwd(),"aaa.txt")
file2 = os.path.join(os.getcwd(),"bbb.txt")
shutil.move(file1, file2)
```

```python
shutil.disk_usage(path):
功能：获取当前目录所在硬盘的使用情况。
参数：
path：文件夹或者文件路径。windows必须是文件夹路径。
import shutil
import os
path = os.path.join(os.getcwd(),'aaa')
info = shutil.disk_usage(path)
print (info) 
# usage(total=95089164288, used=7953104896, free=87136059392)
```

```python
shutil.chown(path,user=None,group=None):
功能：修改路径指向的文件或文件夹的所有者或者分组
参数：
path：路径
user：所有者，传递user的值必须是真实的，否则将报错no such user
group：分组，传递group的值必须是真实的，否则将报错no such group
实例：
import shutil 
import os
path = os.path.join(os.getcwd(),'fiel.txt')
shutil.chown(path,user='root',group='root')
```

```python
shutil.which(cmd,mode=os.F_OF|os.X_OK,path=None):
功能：获取给定的cmd命令命令的可执行文件的路径。
实例：
import shutil
info = shutil.which('python')
print (info)
#>>>/usr/bin/python
```

### 归档操作

```python
make_archive(base_name,format,root_dir,....)
功能:生成压缩文件
参数：
base_name：压缩文件的文件名，不允许有扩展名，因为根据压缩格式生成相应的扩展名。
format：压缩格式。
root_dir：将指定文件夹进行压缩。
实例：
import shutil 
import os
base_name = os.path.join(os.getcwd(),'aaa')
format = 'zip'
root_dir = oa.path.join(os.getcwd(),'aaa')
shutil.make_archive(base_name,format,root_dir)
```

```python
shutil.get_archive_formats()
功能：获取支持的压缩文件格式。目前的支持有：tar、zip、gztar、bztar、xztar。
```

```python
shutil.unpack_archive(filename, extract_dir=None, format=None)： 
功能：解压操作。
参数：
filename：文件路径
extract_dir：解压至的文件夹路径。文件夹可以不存在，会自动生成
format：解压格式，默认为None，会根据扩展名自动选择解压格式
实例：
import shutil 
import os
zip_path = os.path.join(os.getcwd(),"aaa.zip")
extract_dir = os.path.join(os.getcwd(),"aaa")
shutil.unpack_archive(zip_path, extract_dir)
```

```
shutil.get_unpack_formats()： 
功能：获取支持的解压文件格式。目前支持的有：tar、zip、gztar、bztar和xztar。
```

### 查询输出终端的大小

```
shutil.get_terminal_size(fallback=())
功能：获取终端窗口的大小
返回命名元组：os.terminal_size(columns=120, lines=30)
就是终端窗口的长和宽（手动拉伸缩放，值就会变）
```

