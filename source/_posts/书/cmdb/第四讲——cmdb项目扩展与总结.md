---
title: 第四讲——cmdb项目扩展与总结
id: 4
date: 2019-11-19 20:00:00
tags: cmdb
comment: true
---

## 内容回顾

1. jwt的优势？
2. 节流是如何实现的?
3. 权限的实现流程？
4. 认证的实现流程？
5. cmdb是如何实现的？
6. cmdb开发的过程中你觉得什么最难搞？
7. 什么是跨域？如何解决跨域？

## 今日概要

- CPU和主板信息的采集入库
- 后台管理示例
- agent和第三方工具的模式

<!----more---->

### 1.CPU和主板信息

### 2. 简单的后台

- 首页，服务器列表
- 图表
  - echarts
  - highcharts
    注意： https://www.cnblogs.com/wupeiqi/articles/6216618.html

### 3. agent和第三方工具的模式

- 基于SSH实现

- 基于saltstack实现

  - 在进入公司之前，公司已经再用 saltstack（100服务器）

    ```
    https://www.cnblogs.com/wupeiqi/articles/6415436.html
    ```

    - master: 1台
    - minion: 99台

- 基于agent实现

## 作业

基于saltstack实现资产的采集

两个虚拟机

- master

  ```
  1. 安装salt-master
      yum install salt-master
  2. 修改配置文件：/etc/salt/master
      interface: 0.0.0.0    # 表示Master的IP 
  3. 启动
      service salt-master start
  ```

- minion

  ```
  1. 安装salt-minion
      yum install salt-minion
  
  2. 修改配置文件 /etc/salt/minion
      master: 10.211.55.4           # master的地址
      id: c2.salt.com                    # 客户端在salt-master中显示的唯一ID
  3. 启动
      service salt-minion start
  ```

- 在master上进行授权

  ```
  salt-key -L                # 查看已授权和未授权的slave
  salt-key -a  salve_id      # 接受指定id的salve
  
  salt-key -r  salve_id      # 拒绝指定id的salve
  salt-key -d  salve_id      # 删除指定id的salve
  ```

- 在master上远程执行命令

  ```
  salt 'c2.salt.com' cmd.run  'ifconfig'
  ```

## 内容梳理

#### 1. 环境

- 开发环境，在我们自己电脑上进行项目开发。

  ```
  例如：开发一个网站。 在自己电脑上开发+连接自己电脑的数据库。
  ```

- 测试环境

  ```
  公司会给你提供一台服务器（阿里云）。 
  	IP：47.93.2.18
  	用户名：root
  	密码：123123
  把你开发的网站要部署在这个服务器上。
  	- 安装必要的环境：python/django/requests/mysql
  	- 把代码拷贝到这个服务器上
  	- 让我们的程序运行起来。
  		python aa.py 
  		python manage.py runserver 0.0.0.0:8000
  ```

- 线上环境

  ```
  公司会给运维一个台服务器（阿里云）。
  	IP：47.93.2.22
  	用户名：root
  	密码：123123
  
  把你开发的网站要部署在这个服务器上。
  	- 安装必要的环境：python/django/requests/mysql
  	- 把代码拷贝到这个服务器上
  	- 让我们的程序运行起来。
  		python aa.py 
  		python manage.py runserver 0.0.0.0:8000
  ```

#### 2.远程连接服务器

开发者一般都是用xshell，通过ssh连接上远程服务器然后进行操作。 

```
ssh root@192.168.16.85
```

- 用户名和密码连接
- 公钥和私钥进行连接

#### 3.向服务器上传文件

- FTP/xftp6

- lrzsz

- scp命令（对于五图形界面使用的频率最高）

  ```
  scp 文件 root@目标服务器:目录
  ```

- 基于git ( 公司做代码上线会利用 ) 

  ```
  开发：git push origin master 
  运维：git clone xxx
  ```

一般情况下：

- windows

  ```
  FTP
  lrzsz
  git终端来继续操作 scp
  ```

- mac/linux

  ```
  scp
  ```

#### 4.运维

运维管理500台服务器，由于数量太多，所以他们会借助于一些管理工具，例如：saltstack / ansible(paramiko)

未借助工具

```
                                                               服务器A（公钥）
                                                               服务器B（公钥）
运维      电脑（私钥）                                            服务器C（公钥）
                                                               服务器D（公钥）
                                                               服务器E（公钥）
                                                               服务器...（公钥）
```



借助工具（ansible/saltstack）

```
                                                               服务器A（公钥）minion
                                                               服务器B（公钥）minion
运维      电脑（私钥）          服务器（salt-master）              服务器C（公钥）minion
                                                               服务器D（公钥）minion
                                                               服务器E（公钥）minion
                                                               服务器...（公钥）minion
```

- 单独操作

  ```
  在自己电脑上连接远程服务器A
  	ssh root@服务器A 
  ```

- 批量操作

  ```
  在自己电脑上连接salt-master
  	ssh root@salt-master 
  在master服务器上执行命令
  	salt "*" cmd.run "ifconfig"
  让所有的minion都去执行ifconfig命令，并返回。
  ```


#### 5.cmdb项目

- 基于ssh实现，适用场景：公司没有用  saltstack/用了ansible/没有任何工具 

  ```
  服务器(autoserver)       服务器（autoclient）
                                                                  服务器A（公钥）
  DB  API                  中控机(私钥)                             服务器B（公钥）
  后台管理                                                          服务器C（公钥）
    人                                                              服务器D（公钥）
                                                                   服务器E（公钥）
                                                                   ...
  ```

- 基于salt实现，使用场景：公司本身就有saltstack

  ```
  服务器(autoserver)       服务器（autoclient）
                                                                  服务器A(salt-minion)
  DB  API                 中控机（salt-master）                    服务器B(salt-minion)
  后台管理                                                          服务器C(salt-minion)
    人                                                              服务器D(salt-minion)
                                                                   服务器E(salt-minion)
                                                                   ...
  ```

#### 6.cmdb项目测试

##### 6.1 salt模式进行

我们需要三台服务器进行测试。

```
192.168.16.64     做API和后台管理项目的部署
192.168.16.85     做salt-master，在上面部署autoclient，用于资产采集。
192.168.16.22     做salt-minion，让master去连接他并采集资产。 
```

###### 第一步：安装并使用salt （运维）

- master（192.168.16.85）

  ```
  1. 安装salt-master
      yum install salt-master
  2. 修改配置文件：/etc/salt/master
      interface: 0.0.0.0    # 表示Master的IP 
  3. 启动
      service salt-master start
  ```

- minion（192.168.16.22）

  ```
  1. 安装salt-minion
      yum install salt-minion
  
  2. 修改配置文件 /etc/salt/minion
      master: 10.211.55.4           # master的地址
      id: c2.salt.com                    # 客户端在salt-master中显示的唯一ID
  3. 启动
      service salt-minion start
  ```

- 在master上进行授权

  ```
  salt-key -L                # 查看已授权和未授权的slave
  salt-key -a  salve_id      # 接受指定id的salve
  
  salt-key -r  salve_id      # 拒绝指定id的salve
  salt-key -d  salve_id      # 删除指定id的salve
  ```

- 在master上远程执行命令

  ```
  salt 'c2.salt.com' cmd.run  'ifconfig'
  ```

###### 第二步：项目开发

....

.....

......

###### 第三步：项目部署

- 运行autoserver

  ```
  在django项目的setting中要修改
  
  ALLOWED_HOSTS = ["*",]
  ```

  ```
  # 启动配置
  python manage.py runserver 0.0.0.0:8000
  ```

  ```
  # 其他人可以使用IP地址进行访问
  http://192.168.16.64:8000/server/index/
  ```

- 运行autoclient

  ```
  # 将autoclient上传到192.168.16.85
  scp 
  ```

##### 6.2 SSH模式

###### 第一步：生成一对公钥和私钥

```
ssh-keygen
```

###### 第二步：公钥拷贝到远程服务器

```
ssh-code-id -i id_rsa.pub root@192.168.16.22
```

###### 第三步：修改配置围巾啊

```
在settings中修改：
	SSH_PRIVATE_KEY_PATH = r'..id_rsa'
	MODE = "SSH" # SALT/SSH
```

###### 第四步：运行程序

```
python app.py 
```

#### 7.公司里的规范

公司里对于服务器很多都是用主机名，不用IP。

```
c1-bj-zw-shopping.com     192.168.16.64     做API和后台管理项目的部署
c2-bj-zw-shopping.com     192.168.16.85     做salt-master，在上面部署autoclient，用于资产采集。
c3-bj-zw-shopping.com     192.168.16.22     做salt-minion，让master去连接他并采集资产。 
```

#### 8.资产采集多久进行一次？如何进行？

定时任务在中控机上定期执行脚本。

```
30  1   *   *  *     /opt/python367/bin/python3  /data/app.py
```

#### 9.前端示例 cmdbmanage.py （2.7版本）

- 安装python2.7

- 安装django

  ```
  pip2 install django==1.7.6
  ```

- 安装MySQLdb / 安装pymysql

  ```
  pip2 install pymysql
  
  
  import pymysql
  pymysql.install_as_MySQLdb()
  ```

- 安装 xlrd （读excel）

  ```
  pip2 install xlrd
  ```

- 安装paramiko

  ```
  pip2 install paramiko
  ```

- 安装 pillow

  ```
  pip2 install pillow 
  ```

- 在数据库中导入数据

  ```
  create databse cmdb DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
  
  mysql -u root -p  cmdb < cmdb.sql
  ```

- 在项目中修改数据库连接

  ```
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'cmdb',
          'USER': 'root',
          'PASSWORD': '222',
          'HOST': '127.0.0.1',
          'PORT': '3306',
      }
  }
  
  ```

- 运行项目

  ```
  python2 manage.py runserver 
  ```

- 登录

  ```
  用户名：wupeiqi
  密码：123
  ```


## 项目总结

- 项目名称

  ```
  CMDB / 资产管理系统 / 服务器配置管理系统 / 运维自动化平台
  ```

- 项目描述

  ```
  CMDB是一套用于自动化采集服务器资产信息的项目，由于公司对于资产维护成本比较高并且数据准确性越来越低，因为原来都是搭建了samba服务,在内部共享了excel实现。通过cmdb项目可以改善资产采集的功能，减低人员成本提高工作效率，本项目实现主要有 采集中控机/restful api/资产管控平台 实现。
  
  对于采集中控机可以支持多种模式进行操作,如：saltstack/ansible/paramiko默认，并且开发过程中遵循开放封闭原则并且利用 工厂模式 实现可扩展性的插件。
  
  对于api，是严格是遵循restful规范并使用 django rest framework框架实现，并在内通过反射机制实现资产变更记录以及资产的持久化处理。
  
  资产管控平台主要为运维及主管提供数据支持和部分报表，支持excel批量导入导出，支持利用时间轴清晰的展示服务器生命周期，基于highcharts实现数据报表的展示。 
  ```

- 技术点（项目功能/我的职责）

  - 针对不同公司的业务开发，使用 paramiko/ansible/saltstack 实现远程采集资产的扩展。 

  - 参考 middleware 实现源码并结合工厂模式，开发出了可插拔式的采集资产插件。

  - 考虑到项目的严谨性，对于项目中的插件使用主动抛出异常进行约束。

    ```
    - 有没有其他的约束？
    	通过abc实现抽象类和抽象方法实现约束。
    - 你为什么不用abc？而用异常？
    	abc 操作起来比较麻烦，也可以实现。 
    	我觉得异常会更加简洁一些，并且我参考了一些源码，他们内部也是通过异常实现。 
    
    import abc
    
    # 抽象类
    class Base(metaclass=abc.ABCMeta):
        
        # 抽象方法
        @abc.abstractmethod
        def process(self):
            pass
    class Foo(Base):
        pass
    Foo()
    ```

  - 通过定制和扩展drf 内置authentication组件，实现用户认证。 

  - 在restful api中实现api/版本/认证的功能。 

  - 支持最服务器资产进行批量的导入导出，内部只用xlrd/xlwt模块进行操作。 

  - 对于公司的服务器资产进行根据业务线做 数据报表的处理。  

  - 基于rbac实现权限的信息的校验。

- 面试题相关

  - 你的cmdb是怎们实现？

  - 为什么要开发cmdb？

  - 你们公司有多少台服务器？（物理机）

    ```
    70台服务器
    ```

  - 什么是品牌的服务器？

    ```
    戴尔
    ```

  - 资产采集都用到了那些命令？

    ```
    demidecode 
    Megacli
    ```

  - cmdb都用到了那些表？（13张表）

    ```
    用户表
    部门
    机房IDC
    服务器
    硬盘
    网卡
    内存
    变更记录
    
    菜单表
    权限
    角色表
    角色和权限关系
    用户和角色的关系表
    ```

  - 多少人开发？

    ```
    1个人/2个人 + 运维人员
    ```

  - 开发了多久？

    ```
    3个月 ~ 6个月
    质疑时间短，开发资产采集很简单；资产管控平台也是有你来开发。 
    ```

    





























































































































































































