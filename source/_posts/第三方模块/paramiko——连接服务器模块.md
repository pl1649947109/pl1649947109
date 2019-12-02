---
title: paramiko——连接服务器模块
id: 13
date: 2019-11-12 20:30:00
tags: 第三方模块
comment: true
---

### paramiko

解释：通过SSH服务可以远程连接到Linux服务器，查看上面的日志状态，批量配置远程服务器，文件上传，文件下载等，Python的paramiko模块同样实现了这一功能。

#### 理解名词

- SSHClient：包装了Channel、Transport、SFTPClient
- Channel：是一种类Socket，一种安全的SSH传输通道
- Transport：是一种加密的会话（但是这样一个对象的Session并未建立），并且创建了一个加密的tunnels，这个tunnels叫做Channel
- Session：是client与Server保持连接的对象，用connect()/start_client()/start_server()开始会话

<!----more---->

### 功能

#### 远程执行命令【用户名和密码】

```python
import paramiko

# 创建SSH对象
ssh = paramiko.SSHClient()

# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

# 连接服务器
ssh.connect(hostname='192.168.16.85', port=22, username='root', password='123456')

# 执行命令
stdin, stdout, stderr = ssh.exec_command('df')
# 获取命令结果
result = stdout.read()
# 关闭连接
ssh.close()

print(result.decode('utf-8'))
```

#### 远程执行命令【公钥和私钥】

```python
import paramiko

private_key = paramiko.RSAKey.from_private_key_file(r'C:/Users/Administrator/.ssh/id_rsa')

# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname='192.168.16.85', port=22, username='root', pkey=private_key)

# 执行命令
stdin, stdout, stderr = ssh.exec_command('df')
# 获取命令结果
result = stdout.read()

# 关闭连接
ssh.close()

print(result)
```

#### 远程上传和下载文件【用户名和密码】

```python
import paramiko

transport = paramiko.Transport(('192.168.16.85', 22))
transport.connect(username='root', password='123456')
sftp = paramiko.SFTPClient.from_transport(transport)


# 将location.py 上传至服务器 /tmp/test.py
# sftp.put('wy.txt', '/data/wy.txt')
sftp.get('/data/wy.txt', 'xx.txt')

transport.close()
```

#### 远程上传和下载文件【公钥和私钥】

```python
import paramiko

private_key = paramiko.RSAKey.from_private_key_file(r'C:/Users/Administrator/.ssh/id_rsa')

transport = paramiko.Transport(('192.168.16.85', 22))
transport.connect(username='root', pkey=private_key)

sftp = paramiko.SFTPClient.from_transport(transport)
# 将location.py 上传至服务器 /tmp/test.py
# sftp.put('/tmp/location.py', '/tmp/test.py')

# 将remove_path 下载到本地 local_path
# sftp.get('remove_path', 'local_path')

transport.close()
```

#### 通过私钥字符串也可以连接远程服务器

```python
key = """-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----"""


import paramiko
from io import StringIO

private_key = paramiko.RSAKey(file_obj=StringIO(key))

# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname='192.168.16.85', port=22, username='root', pkey=private_key)

# 执行命令
stdin, stdout, stderr = ssh.exec_command('df')
# 获取命令结果
result = stdout.read()

# 关闭连接
ssh.close()

print(result)
```

### 实现xshell功能

```python
import paramiko
import os
import select
import sys
import tty
import termios
 
'''
实现一个xshell登录系统的效果，登录到系统就不断输入命令同时返回结果
支持自动补全，直接调用服务器终端
'''
# 建立一个socket
trans = paramiko.Transport(('192.168.2.129', 22))
# 启动一个客户端
trans.start_client()
 
# 如果使用rsa密钥登录的话
'''
default_key_file = os.path.join(os.environ['HOME'], '.ssh', 'id_rsa')
prikey = paramiko.RSAKey.from_private_key_file(default_key_file)
trans.auth_publickey(username='super', key=prikey)
'''
# 如果使用用户名和密码登录
trans.auth_password(username='super', password='super')
# 打开一个通道
channel = trans.open_session()
# 获取终端
channel.get_pty()
# 激活终端，这样就可以登录到终端了，就和我们用类似于xshell登录系统一样
channel.invoke_shell()
 
# 获取原操作终端属性
oldtty = termios.tcgetattr(sys.stdin)
try:
    # 将现在的操作终端属性设置为服务器上的原生终端属性,可以支持tab了
    tty.setraw(sys.stdin)
    channel.settimeout(0)
 
    while True:
        readlist, writelist, errlist = select.select([channel, sys.stdin,], [], [])
        # 如果是用户输入命令了,sys.stdin发生变化
        if sys.stdin in readlist:
            # 获取输入的内容，输入一个字符发送1个字符
            input_cmd = sys.stdin.read(1)
            # 将命令发送给服务器
            channel.sendall(input_cmd)
 
        # 服务器返回了结果,channel通道接受到结果,发生变化 select感知到
        if channel in readlist:
            # 获取结果
            result = channel.recv(1024)
            # 断开连接后退出
            if len(result) == 0:
                print("\r\n**** EOF **** \r\n")
                break
            # 输出到屏幕
            sys.stdout.write(result.decode())
            sys.stdout.flush()
finally:
    # 执行完后将现在的终端属性恢复为原操作终端属性
    termios.tcsetattr(sys.stdin, termios.TCSADRAIN, oldtty)
 
# 关闭通道
channel.close()
# 关闭链接
trans.close()
```

