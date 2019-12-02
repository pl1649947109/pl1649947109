---
title: 第四讲——安装python、创建虚拟环境
id: 4
date: 2019-11-3 20:00:00
tags: linux
comment: true
---

### python3的编译安装

1.解决系统的基础开发工具，防止python3编译过程出错

```shell
yum install gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y
	-确保机器可以上网，在线下载软件包
	-配置好阿里云的yum仓库，yum源，加速下载，提供大量的软件包 
```

2.获取python3的源代码，去官网下载即可

```shell
wget https://www.python.org/ftp/python/3.6.7/Python-3.6.7.tar.xz

#wget 在线下载一个资源 ,wget就是在www网站下载数据的意思
```

3.解压缩源代码包，进入源码包

```shell
xz -d  Python-3.6.7.tar.xz  #去掉.xz压缩后缀
tar -xvf Python-3.6.7.tar  #解压缩 
cd Python-3.6.7
```

4.编译三部曲，几乎所有的linux软件，编译安装都是这个步骤，nginx，redis都是这样

```shell
第一曲：指定安装路径
	执行configure脚本文件 ，指定软件的安装路径
		./configure --prefix=/opt/python367/
第二曲：编译源代码
	指定make指令 ，针对当前文件夹下的makefile开始读取
	输入 make 即可 
第三曲：开始安装
	这一步才是生成解释器的步骤
	make install 
```
5.配置PATH环境变量，让命令可以快捷执行

```shell
	取出当前PATH的值
	注意！！！！！PATH的值是自上而下，从前往后的读取顺序，这里要和虚拟环境配置有关了
	echo $PATH 
把python3的路径，放到最前面，因为虚拟环境的创建的时候，可能会有坑，
PATH的加载顺序是自上而下的
[root@s24_linux bin]# echo  $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

修改PATH的值，永久修改，写入到/etc/profile ，每次用户登录都加载这个文件，因此变量永久生效

vim  /etc/profile  在最底行，写入如下信息
PATH="/opt/python367/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin"

读取/etc/profile  #让他永久生效 
source  /etc/profile  #用source命令，去读这个文件 内容，让变量生效 

#上面的过程是在我们环境变量加载文件的时候，开始直接把我们的位置直接加如该文件中并应用上，这样就可以在我们环境每个地方使用了。
还有一种方式做上述的事情：（建立软连接）
ln -s /usr/local/python3/bin/python3.6 /usr/bin/python3  这个如软连接就是把我们的python解释器的路径指向/usr/bin下面，但是这个路径是一直就在PATH中，因此它的原理也是和上面的差不多，上面的方式更加直接，这个方式是曲线的方式，通过/usr/bin的方式加载进去。
但是，这个就涉及到了环境变量加载的问题，前面的方式是第一个加载查找，而后面的软连接是在中间的位置，所以查找的位置靠后。
```

通过上述的过程，我们就把python3解释器安装好了，并正确配置了环境变量。

### 创建各种虚拟环境

```shell
1.虚拟环境工具的学习

python的虚拟环境，其实就是在机器上，方便的创建出多个解释器，每个解释器运行一个项目，互相之间不受影响

2.virtualenv工具，可以方便的创建，使用，删除也很方便

3.安装virtualenv 

pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple  virtualenv 

4.创建虚拟环境 venv ，用于运行django1
virtualenv  --no-site-packages --python=python3    venv1  
	--no-site-packages  #这个参数 ，创建虚拟环境是干净隔离的
	--python=python3   #这个--python参数，是指定解释器的版本
	s24django1 是虚拟环境的名字，文件夹的名

5.激活虚拟环境，需要执行如下命令
source /opt/s24django1/bin/activate  #这是激活虚拟环境的命令

6.deactivate 	#退出虚拟环境

7.在s24django1这个虚拟环境下，运行一个django1版本
	得先安装django模块
	pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple  django==1.11.9
```

7.学习更优秀的虚拟环境工具，virtualenvwrapper，直接选择它就行 ，不用再装上面那个virtualenv 

安装

```
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple  virtualenvwrapper
```

8.配置系统的全局变量，加载virtualenvwrapper这个工具
vim  /etc/profile  #写入如下内容 

```shell
WORKON_HOME=~/Envs   #设置virtualenv的统一管理目录
VIRTUALENVWRAPPER_VIRTUALENV_ARGS='--no-site-packages'   #添加virtualenvwrapper的参数，生成干净隔绝的环境
VIRTUALENVWRAPPER_PYTHON=/opt/python367/bin/python3      #指定python解释器
source /opt/python367/bin/virtualenvwrapper.sh          #执行virtualenvwrapper安装脚本
```

9.退出回话，重新登录，加载/etc/profile 文件，然后可以使用如下命令创建虚拟环境了 

```shell
mkvirtualenv     venv1   #创建虚拟环境venv1 
mkvirtualenv     venv2	#创建虚拟环境venv2 
mkvirtualenv  	 ven3	#创建虚拟环境venv3
workon 					 #激活虚拟环境，支持tab键补全

cdvirtualenv  				#进入虚拟环境家目录
workon:                 列出虚拟环境列表
lsvirtualenv:           列出虚拟环境列表
lssitepackages 			列出当前解释器，所有的模块文件夹 
cdsitepackages			进入当前解释器的模块文件夹 
mkvirtualenv:           新建虚拟环境
workon [虚拟环境名称]:    切换/进入虚拟环境
rmvirtualenv :          删除虚拟环境
deactivate:             离开虚拟环境
```

### win下的mysql数据导入linux

1.下载数据库

```shell
yum install mariadb-server  # yum装的是什么 
```

2.启动服务

```shell
systemctl start mariadb  
```

3.进入数据库（没有密码）

```
mysql -u root -p
```

4.设置密码

```
set password=password("我们设置的密码")
```

5.创建数据库

```
create database 数据库名称;（和win下的数据库名称一样）
```

6.window下导出数据库

```
#注意：这个操作是早cmd下面操作的，不是在mysql数据库里面操作的
mysqldump -u 用户名 -p --database 数据库名 > 指定地址:\导出的文件名
```

7.导入到linux

```
use database_name;
source /win下数据库在linux的位置/database_name;
```

### 保证开发环境和生产环境的模块一致性的方法	

保证windows的模块和linux的模块的一致性

pip3 freeze >  requirements.txt #这是导出解释器所有模块信息的命令，且

通过命令安装这个文件中，所有的模块
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple   -r requirements.txt  