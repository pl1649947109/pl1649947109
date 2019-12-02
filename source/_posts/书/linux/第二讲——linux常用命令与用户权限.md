---
title: 第二讲——linux常用命令与用户权限
id: 2
date: 2019-10-27 20:00:00
tags: linux
comment: true
---

```
whoami  我是谁
```

```
hostname  查看主机名  
```

修改linux的命令提示符，通过变量PS1控制

```shell
echo  $变量名  #打印出变量的值

#输出PS1变量的值

echo  $PS1
#修改变量的值 
PS1='[\u@\h \t \w]\$'
	\u  当前登录用户
	\h  当前主机名  
	\t  当前系统时间
	\w   输出绝对路径
	\W  输出工作路径的最后一位
```

<!----more---->

cat命令 

```shell
cat -n  filename   显示行号

cat -E  filename   每一行结尾加上$符
```

head  tail ,从头看,从尾巴看

```shell
head  -n  数字  filename   #看文件的前n行 

tail  -f  filename  #持续刷新文件内容的变化
#输出文件的10-20行 
head  -18  english.txt |  tail  -7
```

linux的查找命令

```shell
find   从哪找  -type  文件类型 -name  你要找什么名字的文件

文件类型 
l  快捷方式类型
d 文件夹类型 
f   文本类型  

#全局搜索,所有以.txt结尾的文件 
find  /  -type  f       -name  "*.txt"

#在opt目录下搜索 钢管的一生.txt  

find  /opt   -name '钢管的一生.txt' 

#在/opt下搜索和python有关的文件夹

find  /opt  -type  d   -name "python*"

  651  find  /opt  -type  d   -name "python"
  652  find  /opt  -type  d   -name "*python"
  653  mkdir  -p  钢哥/{s23python,s24python,pythons24,pythons23}
  654  find   /opt  -type d  -name  "*python"  
  655  find   /opt  -type d  -name  "python*"  
  656  find   /opt  -type d  -name  "python$"  
  657  find   /opt  -type d  -name  "*python*"  
```

查看进程的命令 

```shell
ps aux 或者 ps -ef  #查看机器所有进程信息

ps aux  |  grep  "vim"  #过滤出和vim有关的进程

ps aux |  grep  "python"  #找到机器所有和python有关的进程   

python  manage.py  runserver  
```

过滤字符串的命令 

```shell
grep  你想要的字符串   filename  
grep -i  'root'   filename   #找出文件中所有大小写的root

#找出文件的有用信息行

grep -v "^#" filename  |  grep -v "^$"
```

远程传输scp

```shell
语法

scp  你想要的内容  传输到哪里

[你在机器1]

#把我本地机器1的/tmp/刚哥.txt发到远程机器的/opt目录下

scp   /tmp/刚哥.txt   root@机器的ip:/opt/    

#把自己的东西发给别人

#把本地的first.py 发送给root@192.168.16.105这个机器，然后
	1.如果data文件夹存在，则放入data文件夹中
	2.如果没有data这个文件夹，则把first.py改名为data
	
scp  ./first.py   root@192.168.16.105:/data 

#把别人的东西拿过来 
scp  root@192.168.16.105:/data/钢哥的小秘密.txt   /opt 

#发送所有的文件和文件夹给别人  
scp  -r ./* root@192.168.16.105:/data/ 
```

统计文件夹大小命令

```shell
1. ls -h  

2. du命令  以du命令为准 
	-h  显示mb  gb单位
	-s  显示统计 
	
#统计/var/log文件夹大小

du -sh  /var/log/
```
```shell
查看文件特殊权限的命令 
lsattr 

设置文件特殊权限的命令 
chattr  
```

更新系统时间

```shell
yum install  ntpdate -y  
ntpdate   ntp.aliyun.com    #和阿里提供的时间服务器，进行时间同步
查看系统时间 
date命令
```

```shell
wget  -r -p   www.luffycity.com   # -r -p  递归爬取网站资源 
```

看内存了

```
free 
```

看磁盘信息了

```
fdisk  
```

磁盘使用率

```
df 命令 
```

linux用户管理

```
root    qq群主  

sudo命令    qq管理员 

普通用户(changxin)   普通群员
```

1.创建普通用户

```shell
useradd  用户名  #创建用户的同时，会创建用户组

useradd  s24  #系统创建s24用户，且创建s24这个组 

passwd  用户名   #更改用户密码
```

2.用户间切换的命令

```shell
su -  用户名  # 中间这个横杠代表 用户环境变量完全切换

root切换普通不要密码
普通用户间切换需要密码
```

3.查看linux用户的id信息

```
id  用户名  #
```

4.查看存放用户信息的文件 

```shell
/etc/passwd  

经过root创建的普通用户 id从 1000开始 
系统自带的用户，如 mysql  ,bin,nginx等用户，默认是1~999
```


5.删除用户 

```shell
userdel  -rf 用户名  #
```

6.管理员权限 sudo命令


```shell
配置sudo命令的方式
1.用visudo命令，打开配置文件，添加如下配置
	## Allow root to run any commands anywhere
	root    ALL=(ALL)       ALL
	gg      ALL=(ALL)       ALL

2.此时可以使用sudo命令了
sudo  你想敲的命令
```

7.文件权限 

```shell
文件的权限分三类人 

user   属主
group  属组
other  其他人	
```


```shell
文件的读写执行是：

顺序必须是 读写执行 
r  read   读
w   write  写 
x    exec可执行 
-   没有权限 
```

```shell
[root@s24_linux tmp]# ls -l
total 0
-rw-r--r--. 1 root root 0 Oct 27 16:36 gg.txt   

 

- 这是一个普通文件，   d  代表文件夹  l 代表软连接快捷方式 

rw-(users   属主的权限 )  读  写  权限 

r--(group   属组的权限，在这个组里面的人，都有)    只读的权限 

r-- （  others   当前登录的用户，和这个文件没关系，就是其他人的身份权限）   只读的权限



root  这个文件是root创建的

root  这个文件是root组里面的人，可以有权限操作
```


8.修改文件权限

```shell
chmod  (chang  mode)  

chmod  u+x  filename  #给文件的user用户，添加x可执行权限

chmod o-r gg.txt   #给其他人去掉r的读取权限 
```

9.文件的读写执行 

```shell
r   cat   more less  读取文件内容 
w   vim   echo追加  编辑文件内容
x   可以执行的为文件
```

10.文件夹的读写执行 

```she&#39;l&#39;l
r   ls  
w   在文件夹中 mkdir  或者touch等创建文件，必须有x权限才行
x    可以cd进入文件夹 
```


11.用户组

```shell
/etc/passwd 
/etc/shadow
/etc/group 
```

12.修改文件属主,把这个文件的主人,换一个

```shell
chown  wy  filename.txt  #修改文件的主人是 wy 
```

13.修改文件属组,

```shell
chgrp   wy   filename.txt   #修改文件的属组为wy组 
```

14.将用户加入某个组 ,把这个用户,加入文件的属组里面,也就有相应的权限了 

```shell
usermod -G  
```

15.软连接,快捷方式  

```shell
语法

ln -s  源文件绝对路径   快捷方式绝对路径 

ln -s  /tmp/电视机.txt   /opt/dsj.txt   
```

16.打包,压缩命令 

```shell
打包命令  
tar   
参数 
-z  调用gzip压缩
-x  解包
-c  打包
-v 显示过程 
-f  必须写参数结尾,指定tar包的名字 
```


```shell
1.案例	
把/tmp下所有内容打包成  alltmp.tar 文件 
tar -cvf  alltmp.tar  ./*

2.解包的命令,把alltmp.tar的内容,解压缩到/tmp目录下

3.打包且压缩的命令,能够节省60%-70%磁盘空间
tar  -zcvf  alltmp.tar.gz   ./*  

4.解压缩命令 
tar  -zxvf  ../alltmp.tar.gz    ./
```

17.另一个打包命令 zip,解包命令unzip  

18.查看linux网络端口的命令

netstat  -tunlp  

```shell
参数解释:
	netstat [选项]
	-t或--tcp：显示TCP传输协议的连线状况；
	-u或--udp：显示UDP传输协议的连线状况；
	-n或--numeric：直接使用ip地址，而不通过域名服务器；
	-l或--listening：显示监控中的服务器的Socket；
	-p或--programs：显示正在使用Socket的程序识别码和程序名称；
	-a或--all：显示所有连线中的Socket；
```

#案例
检查服务器的80端口是否打开

netstat  -tunlp  |  grep 80  

#安装nginx,且运行nginx,打开80端口
	#安装   yum install  nginx  -y  
	#启动   通过yum安装的软件,都可以通过系统服务管理命令,去启动,无论是nginx 还是mariadb等
	systemctl  start/stop/restart/status  nginx  

```shell
 #注意关闭防火墙的命令
systemctl stop  firewalld  		#停止防火墙服务 

systemctl disable  firewalld    #停止防火墙开机自启

iptables -F   				    #清空防火墙规则 	
```

19.杀死进程的命令

#通过进程id  杀死nginx 

```shell
1.  检查pid
ps aux  | grep  nginx 

2.杀死进程id  
kill  id 号码
```

20.定时任务

```shell
1.检查定时任务列表 

crontab -l  

2.设置定时任务

crontab  -e   

#语法  ,每分钟向一个文件中,追加一个信息 
```

格式：

```
分  时  日 月 周      执行命令的绝对路径
```

实例：

```
#每小时的3,15分组执行命令

    - - - - - 命令的绝对路径 

    3,15    *   *  *  *   

#在下午8-11点的第3和第15分钟执行

    - - - - - 命令的绝对路径 
    3,15  20-23   *  *  *  

    3,5    5-9(早上五点,6点,7点,8点,9点)

    第三分钟和第五分钟

#每晚21:30执行命令

    - - - - - 命令的绝对路径

    30   21   *  *  *   

#每晚的12点执行命令 

    0  0   *  *  *  

#每周六、日的1：30执行命令

    - - - - - 命令的绝对路径

    30    1    *  *   6,0

#每周一到周五的凌晨1点，清空/tmp目录的所有文件

    - - - - - 命令的绝对路径

    0   1   *  *  1-5   /usr/bin/rm -rf /tmp/*

#每晚的21:30重启nginx

    - - - - - 命令的绝对路径

    30  21   *  *  *   /usr/bin/systemctl  restart nginx  

#每月的1,10,22日的下午4:45重启nginx

    - - - - - 命令的绝对路径
              45   16    1,10,22  *  *   /usr/bin/systemctl  restart nginx  

#每个星期一的上午8点到11点的第3和15分钟执行命令

    - - - - - 命令的绝对路径

    3,15  8-11  * *   1    
```

21.软件包管理 yum命令 

```shell
软件包格式都是 

mysql-xxx.rpm 软件格式包 

rpm -ivh  mysql-xxx.rpm  #安装rpm软件包 

python-xx.rpm  
```

配置yum源的过程
centos的默认yum仓库路径是 /etc/yum.repos.d  ,在这目录下,第一层文件夹中的repo文件会识别为仓库文件

1.获取阿里云的yum源
打开网址https://opsx.alibaba.com/mirror

2.找到第一个仓库
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

3.下载第二个仓库
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

4.生成yum缓存,加速以后下载
yum makecache 