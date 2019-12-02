---
title: 第五讲——nginx安装、配置文件、项目部署
id: 5
date: 2019-11-10 20:00:00
tags: linux
comment: true
---

### 安装编译nginx

方式一：yum源安装

```
yum install nginx（自动解决依赖）
```

方式二：wget手动安装

```
1.下载tengine（淘宝源）
wget http://tengine.taobao.org/download/tengine-2.3.2.tar.gz

2.编译三部曲
	第一曲：指定安装路径 
	./configure --prefix=/opt/tbnginx/
	第二曲：开始编译，生成makefile
	make 
	#第三曲：开始安装 
	make install  
3.启动nginx，配置nginx的环境变量
	vim /etc/profile 
	配置PATH，把/usr/local/nginx/sbin加入进入
	让PATH生效： 执行source /etc/profile
4.操作nginx
	第一次启动：直接nginx
	nginx -s stop 停止
	nginx -s reload  平滑重启，不停止进程，重新读取配置文件
	nginx -t  检查nginx.conf语法是否正确，更加安全
5.nginx的配置文件
	conf存放nginx的配置文件
	html存放nginx静态文件
	logs   nginx的运行日志，错误日志，访问日志
	sbin存放可执行的命令
```

<!----more---->

### nginx的nginx.conf配置文件

#### nginx的静态文件服务器功能

```shell
#server{}是定义虚拟主机功能 
    server {
        #定义网站的端口
        listen       80;
        #访问的网址,该网站必须是注册的并且已解析的
        server_name  linshutu.top;

        #定义虚拟主机的访问日志功能，记录用户的ip，以及请求信息，和爬虫代理后面的真实ip等功能
        #access_log  logs/host.access.log  main;
        #access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;
        # location作用是匹配url，如同 django的 url.py功能一样
        location / {
            #这个root关键词 是定义静态文件存放目录的，这里指定额的是/opt/s24html文件夹下
            root   /opt/s24html;
        	#index参数，定义网站首页文件名的 
            index  index.html index.htm;
        }
        #  这个location可以定义多个，比如 你想让 192.168.16.85:81/static/js/luffy.js
        location  /static  {
        #给路径添加别名，/opt/s24crm/static/js/luffy.js
        alias   /opt/s24crm/static/;
}
```

- nginx的多虚拟主机功能 

```shell
#一台服务器，基于域名的不同，访问不同的网站资料,也就是准备2个server{}的定义

第一个server{}虚拟主机 ，linshutu.top
    server {
				#定义网站的端口
				listen       80;
				#定义网站匹配的域名 
				server_name  linshutu.top;
				#server_name  _;

				location / {
					root   /opt/s24pian; 
					index  index.html index.htm;
				}
				#当你的请求时 linshutu.top/static/55kai.jpg  
				location  /static  {
				#给路径添加别名 
				alias   /opt/s24crm/static/;
		}
    }

第二个虚拟主机  www.server2.com  

    server {
        listen  80;
        server_name  www.server2.com;
        location  / {
        root  /opt/server2;
        index   index.html;
        }
   }
```

#### nginx的访问日志功能，404页面功能

```shell
 #把下面的这些注掉的取消就行了
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;
    access_log  "pipe:rollback logs/access_log interval=1d baknum=7 maxsize=2G"  main;

404错误页面优化(找到一个server{}标签的配置，添加一行代码即可 )

        location / {
           #这个root关键词 是定义静态文件存放目录的
            root   /opt/s24pian;
        #index参数，定义网站首页文件名的 
            index  index.html index.htm;
        }
		#nginx错误页面的定义
		error_page  404     /404.html;
		
#注意：自定制404界面的时候，404.html查找的位置是root后面定义的文件夹的位置
```

#### 配置反向代理 

```shell
server {
    listen       80;
    server_name  _;
    location / {
    #反向代理的参数 
    proxy_pass   http://127.0.0.1:81;
    }
    error_page  404              /404.html;
}
```
#### nginx负载均衡

```shell
#通过upstream关键词，定义服务器地址池：这里是模拟的
upstream  lishutu  {
    server  127.0.0.1:81;
    server  127.0.0.1:82;
    server  127.0.0.1:83;
}
   #定义第一个虚拟主机 server{} ，功能是：进行反向代理，负载均衡 
   server {
		listen 80;
		#下面配置的网址需要是可访问到本服务器的，否则就设置为server_name _;
		server_name linshutu.top;
		location  /  {
		#nginx代理，指向上面的地址池
		proxy_pass http://linshutu;
	}
}

#为了区分，实验我们下面的文件就不使用一样的项目，而是使用不同的页面便于观察。

#第二个server，模拟第一台django
server  {
listen  81;
server_name _;
location  /  {
	#/opt/pl1/index.html  404.html
	root  /opt/pl1;
	index  index.html;
}
error_page  404     /404.html;
}
#定义第三个server，模拟第二台django
server {
  listen 82;
server_name  _;
location  / {
    ##/opt/pl2/index.html  404.html
    root  /opt/pl2;
    index  index.html;
}
error_page  404     /404.html;
}

#定义第四个server，模拟第三台django，讲道理，这三台django应该提供一样的数据 
server {
  listen 83;
server_name  _;
location  / {
	##/opt/pl3/index.html  404.html
    root  /opt/pl3;
    index  index.html;
}
error_page  404     /404.html;
}


#这样，我们在访问我们的页面linshutu.top/index.html页面的时候刷新页面就会循环展示我们上述的三个html文件。当我们在linshutu.top页面下输入错误的网址的时候就会展示我么文件夹下的404页面
```
#### nginx的负载均衡规则

```shell
1.轮训机制 ，每个机器，解析一次
2.权重机制 ，哪台机器的权重高，请求优先发给谁，有权重比例设置
		upstream  linshutu  {
	server  127.0.0.1:81 weight=1;
	server  127.0.0.1:82 weight=3;
	server  127.0.0.1:83 weight=4;
	}

3.ip_hash  ，真对用户的ip地址得到哈希值，永久发给一台机器 ，ip哈希方式，不得和权重一起用 
	#通过upstream关键词，定义服务器地址池
	upstream  linshutu  {
	server  127.0.0.1:81 ;
	server  127.0.0.1:82 ;
	server  127.0.0.1:83 ;
	ip_hash;
	}
4.配置好access.log日志 ，进行实时刷新
	进入nginx/logs下运行：
		tail -f access.log
```