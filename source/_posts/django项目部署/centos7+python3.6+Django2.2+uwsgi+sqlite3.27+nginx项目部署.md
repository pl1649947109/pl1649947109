---
title: centos7+python3.6+Django2.2+uwsgi+sqlite3.27+nginx项目部署
id: 1
date: 2019-10-7 20:30:00
tags: django项目部署
comment: true
---

### 完整的项目部署流程

不懂点我：https://chpl.top/2019/11/03/%E4%B9%A6/linux/%E7%AC%AC%E5%9B%9B%E8%AE%B2%E2%80%94%E2%80%94%E5%AE%89%E8%A3%85python%E3%80%81%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83/

#### 完整的通信流程

![](http://9017499461.linshutu.top/django%E9%83%A8%E7%BD%B2%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

#### 一、更新系统软件包

```shell
yum update -y
```

#### **二、安装软件管理包和可能使用的依赖**

```shell
yum -y groupinstall "Development tools"
yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel
```

<!----more---->

#### 三、下载Pyhton3到/usr/local 目录**

```shell
#下载
wget https://www.python.org/ftp/python/3.6.6/Python-3.6.6.tgz

#当前文件夹解压
tar -zxvf Python-3.6.6.tgz

#解压成功之后进入python-3.6.6文件夹（编译安装到指定的文件路径）
./configure --prefix=/usr/local/python3

#现在开始暗转python3
make
make install

#现在python3安装成功，开始建立软连接，添加变量，就相当于在windows下添加环境变量
ln -s /usr/local/python3/bin/python3.6 /usr/bin/python3

#同样的，pip3也和python3一起安装了，现在也一起建立软链接
ln -s /usr/local/python3/bin/pip3.6 /usr/bin/pip3
```

#### **四、查看Python3和pip3安装情况**

```shell
#在根目录环境中
键入python3
键入pip3
```

#### **五、安装virtualenv ，建议大家都安装一个virtualenv，方便不同版本项目管理。**

```shell
#下载包
pip3 install virtualenv

#建立软连接
ln -s /usr/local/python3/bin/virtualenv /usr/bin/virtualenv  #这里可能在在python3文件夹下找不到bin这个文件夹，但是我们不同的担心，后面我们一样可以使用，跳过就可以了

#根据个人的习惯，创建项目的文件夹和虚拟环境的文件夹
mkdir -p /data/env  #用来防止我们的虚拟环境的文件夹
mkdir -p /data/wwwroot  #用来防止我们的项目
```

扩展：

```python
vrtaulenvwrapper是virtualenv的扩展包，用于更方便管理虚拟环境
#安装
pip3 install virtualenvwrapper

#现在还不能使用它，还需要配置：
1、查看virtualenvwrapper的安装路径
    sudo find / -name virtualenvwrapper.sh
2、创建目录用来存放虚拟环境
    mkdir ~/.myvirtualenvs
3、在~/.bashrc末尾添加行(vim ~/.bashrc)
    export WORKON_HOME=/home/Linux的用户名/.myvirtualenvs
    source /usr/share/virtualenvwrapper/virtualenvwrapper.sh
4、运行： 
    source ~/.bashrc
    
#功能
workon:                 列出虚拟环境列表
lsvirtualenv:           列出虚拟环境列表
mkvirtualenv:           新建虚拟环境
workon [虚拟环境名称]:    切换/进入虚拟环境
rmvirtualenv :          删除虚拟环境
deactivate:             离开虚拟环境
```

#### **六、切换到/data/env/下，创建指定版本的虚拟环境。**

```python
#创建虚拟环境
virtualenv --python=/usr/bin/python3 pyweb

#现在我们可以看见env下面多了几个文件夹，现在我们进入/data/env/pyweb/bin ，启动虚拟环境
source activate
#我们现在可以看见我们的命令行前面出现(pyweb)，说明是成功进入虚拟环境。
```

#### **七、虚拟环境里用python3安django和uwsgi**

```python
#下载这两个重要的模块
pip3 install django #（如果用于生产的话，则需要指定安装和你项目相同的版本）  
pip3 install uwsgi
#注意，版本是非常重要的，因此我们在决定搭建一个项目的时候，我们就需要事前决定我们使用什么版本的组件。

#给uwsgi建立软链接，方便使用
ln -s /usr/local/python3/bin/uwsgi /usr/bin/uwsgi
```

#### **八、切换到网站目录/data/wwwroot,创建Django项目**

```python
#创建项目
django-admin.py startproject mysite

#进入我们创建的项目里面创建app
python3 manage.py startapp blog

#进入项目文件夹/data/wwwroot/mysite,添加static和templates，分别用于存放静态文件和模板文件。
mkdir templates
mkdir static

#编辑项目里mysite/settings.py文件
vim /data/wwwroot/mysite/mysite/settings.py
1,在INSTALLED_APPS 列表里添加'blog'APP
2,修改ALLOWED_HOSTS，['*']，可以让任何IP访问
3,TEMPLATES里添加模板路径os.path.join(BASE_DIR, 'templates')
4,文件的尾部添加：
STATICFILES_DIRS = (
    os.path.join(BASE_DIR,'static'),
    )
5，保存：wq（这是linux里面的vim的命令，不懂的可以自行百度）
```

#### **九、在templates下添加index.html文件，输入下面内容。**

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>我的网站</title>
</head>
<body>
<h1>欢迎光临我的网站！</h1>
</body>
</html>
```

#### **十、配置URL**

```python
from django.contrib import admin
from django.urls import path
from blog import views  #导入视图

urlpatterns = [
    path('admin/', admin.site.urls),
    path('index/',views.index),  #配置路由
]
```

#### **十一、编辑blog APP下的views.py**

```python
from django.shortcuts import render,HttpResponse

# Create your views here
def index(request):
    return render(request,'index.html')
```

#### **十二、启动项目**

```
python3 manage.py runserver
```

#### **十三、Django正常运行之后我们就开始配置一下uwsgi。**

注意：安装uwsgi之前python和django需要部署完成。

uwsgi配置文件格式 ini、xml、json 都可，最常用的是ini的配置格式。

```shell
方式一：xml
#我们网站项目路径是 /data/wwwroot/mysite/,在项目根目录下创建mysite.xml文件，输入如下内容：
<uwsgi>    
   <socket>127.0.0.1:8000</socket><!-- 内部端口，自定义 --> 
   <chdir>/data/wwwroot/mysite/</chdir><!-- 项目路径 -->            
   <module>mysite.wsgi</module> 
   <processes>4</processes> <!-- 进程数 -->     
   <daemonize>uwsgi.log</daemonize><!-- 日志文件 -->
</uwsgi>

方式二：ini
[uwsgi]
vhost = false
plugins = python
socket = 127.0.0.1:8000
master = true
enable-threads = true
workers = 1
wsgi-file = /data/wwwroot/mysite/wsgi.py
chdir =/data/wwwroot/mysite/
```

ini文件详细配置说明项：

```ini
[uwsgi]
#自定义变量
projectname = MyDjango
base = /www/DjangoProject/MyDjango/
# 启动uwsgi的用户名和用户组
uid = www
gid = www
# 我的项目目录
chdir = %(base)
# 指定项目的application
module = %(projectname).wsgi:application
# 进程个数
workers = 5
# 启用主进程
master = true
# 自动移除unix Socket和pid文件当服务停止的时候
vacuum = true
# 序列化接受的内容，如果可能的话
thunder-lock = true
# 启用线程
enable-threads = true
# 设置自中断时间
harakiri = 30 
# 设置缓冲  
post-buffering = 4096
#pid文件保存路径
pidfile = /tmp/uwsgi.pid
# 设置日志目录
daemonize = /tmp/uwsgi.log
# 指定sock的文件路径,可以用端口或sock文件
#socket = 192.168.88.20:8099
socket = /tmp/uwsgi.sock
```

uwsgi相关运行：

```ini
uwsgi --ini uwsgi.ini
```

重载uwsgi配置文件：

```pid
uwsgi --reload /tmp/uwsgi.pid
```

停止uwsgi服务：

```pid
uwsgi --stop /tmp/uwsgi.pid
```

#### 十四、安装nginx和配置nginx.conf文件

```shell
#进入home目录，执行下面命令
wget http://nginx.org/download/nginx-1.13.7.tar.gz

#本地解压
tar -zxvf nginx-1.13.7.tar.gz

#暗转nginx
./configure
make
make install  #nginx一般默认安装好的路径为/usr/local/nginx

#现在我们修改nginx的配置文件，在/usr/local/nginx/conf/中先备份一下nginx.conf文件，以防意外。
cp nginx.conf nginx.conf.bak

#然后打开nginx.conf，把原来的内容删除，直接加入以下内容：
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    server {
        listen       80;
        server_name  www.django.cn;  
        charset utf-8;
        location / {
           include uwsgi_params;
           uwsgi_pass 127.0.0.1:8000;  #注意
           uwsgi_param UWSGI_SCRIPT mysite.wsgi;
           uwsgi_param UWSGI_CHDIR /data/wwwroot/mysite;  #注意
           
        }
        location /static/ {
        root data/wwwroot/mysite/static/;   #注意
        }
    }
}

#进入/usr/local/nginx/sbin/目录,执行
./nginx -t  #该命令先检查配置文件是否有错，没有错就执行以下命令：
./nginx   #终端没有任何提示就证明nginx启动成功，可以使用你的服务器地址查看，成功之后就会看到一个nginx欢迎页面。
```

#### **十五、访问项目的页面。**

```shell
#进入网站项目目录cd /data/wwwroot/mysite/执行命令：
uwsgi -x mysite.xml

#以上步骤都没有出错的话,进入/usr/local/nginx/sbin/目录,执行：
./nginx -s reload

#然后在浏览器里访问你的项目地址！我们发现一个问题就是我们的admin后台没有生效，后面的你内容我们会讲到
```

**至此，我们的项目都部署成功了！里面最值得留意的就是项目的路径不要弄错，还有，项目的所有操作都要在虚拟环境下进行。**

### 本地项目搬迁到服务器

如果原来项目是在本地的，想要部署上线，可以参考下面的步骤：

1、备份本地数据库。(使用sqlite数据库的话，直接打包数据库文件上传到服务器即可)

2、在项目目录下用下面的命令把当前的环境依赖包导出到requirements.txt文件

```
pip freeze > requirements.txt
```

3、把项目源码压缩打包。

4、把项目上传到对应的目录里。

5、创建新的虚拟环境

6、安装requirements.txt里的依赖。

```
pip install -r requirements.txt
```

7、导入数据库到服务器。

然后重复上面的13、14、15的步骤，即可。

### **关于线上部署admin后台样式没有生效的问题：**

1、在settings.py尾部：

```python
STATIC_ROOT = '/www/mysite/mysite/static'  #设置一个目录，把后台CSS样式放到这个目录里
```

2、收集CSS样式，在终端输入：

```python
python manage.py collectstatic  #运行这个命令之后，就会自动把后台CSS样式收集到/static/目录下。
```

3、把STATIC_ROOT = '/www/mysite/mysite/static'  注释掉，不然启动服务会出错。

然后刷新页面，后台样式恢复！**本人亲测无效，后续会更新**

### **Django启用SSL证书**

群里好多朋友都需要使用SSL证书，在使用我这个部署教程的基础上部署SSL证书，总是遇到不少坑。在这，我在这补充一下安装SSL证书的方法，供大家参考。

1、进入之前我们下载nginx的源码目录

```
cd /home/nginx-1.13.7/
```

2、安装PCRE库

```
yum -y install pcre
```

3、安装SSL

```
yum -y install openssl openssl-devel
```

4、依次执行下面两行代码重新编译一下

```
./configure
./configure --with-http_ssl_module
```

5、执行make

```
make
```

注意：是make而不是make install 

6、备份原来的nginx

```
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
```

7、将新的 nginx 覆盖旧安装目录

```
cp objs/nginx /usr/local/nginx/sbin/nginx
```

如果报错，刚用执行下面的命令覆盖

```
cp -rfp objs/nginx /usr/local/nginx/sbin/nginx
```

8、把域名证书复制到网站根目录里去，后缀为.crt和.key的文件。

9、配置nginx.conf文件

```
 1 worker_processes  1;
 2 events {
 3     worker_connections  1024;
 4 }
 5 http {
 6     include       mime.types;
 7     default_type  application/octet-stream;
 8     sendfile        on;
 9     server {
10         listen 443 ssl http2;
11         server_name www.django.cn django.cn;
12         root /data/wwwroot/mysite;
13         charset utf-8;
14         ssl_certificate    /data/wwwroot/mysite/1_www.django.cn.crt;
15         ssl_certificate_key    /data/wwwroot/mysite/2_www.django.cn.key;
16         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
17         ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
18         ssl_prefer_server_ciphers on;
19         ssl_session_cache shared:SSL:10m;
20         ssl_session_timeout 10m;
21         error_page 497  https://$host$request_uri;
22         location / {
23            include uwsgi_params;
24            uwsgi_pass 127.0.0.1:8997;
25            uwsgi_param UWSGI_SCRIPT wechatProject.wsgi;
26            uwsgi_param UWSGI_CHDIR /data/wwwroot/mysite;
27            
28         }
29         location /static/ {
30         alias /data/wwwroot/mysite/static/; 
31         }
32         access_log  /data/wwwroot/mysite/www.django.cn.log;
33         error_log  /data/wwwroot/mysite/www.django.cn.error.log;
34     }
35 }
```

10、测试配置文件是否正确

```
/usr/local/nginx/sbin/nginx -t
```

如果没有报错则重启nginx即可。

```
/usr/local/nginx/sbin/nginx -s reload
```

本文转自：https://www.django.cn/article/show-4.html