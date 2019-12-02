---
title: subprocess模块
id: 1
date: 2019-8-15 20:30:00
tags: 第三方模块
comment: true
---

### 涉及的模块

- subproces
- subprocess.Popen

- os.system
- os.popen

**subprocess及相关模块的介绍**

- 在python中我们可以通过os.system来控制控制台的形式来运行程序或执行一些命令，但涉及到需要进程之间通信的时候，就需要用到subprocess模块。原来mulltiprocessing是一体的，但是后来官方进行了拆分，但是内容仍然有所重叠。
- 我门几乎可以在任何操作系统上通过命令行指令与操作系统之间进行交互，那么我们在python中就可以使用上面的模块来操作我们的命令行。
- 通常我们执行命令行比较关注两个结果：
  - 命令执行的状态码——表示命令是否执行成功
  - 命令执行的输出结果——命令执行成功后的输出
- 我们在开始的时候通过使用os.system()、os.popen().read()来执行命令行的指令，以及我们非常少使用的commands模块。
- 在python2.4之后，官方的文档建议使用subprocess模块，所以这篇文章我们就重点的介绍该模块的使用。

<!-----more----->

**os模块的简单介绍**

| 函数名                   | 描述                                                         | 实例                   |
| ------------------------ | ------------------------------------------------------------ | ---------------------- |
| os.system(command)       | 返回命令执行状态码，而将命令执行结果输出到屏幕               | os.system('dir')       |
| os.popen(command).read() | 可以获取命令执行结果，但是无法获取命令执行状态码             | os.popen('dir').read() |
| os.popen(command)        | 函数得到的是一个文件对象，因此除了read()方法外还支持write()等方法，具体要根据command来定 |                        |

**subprocess模块的详细介绍**

介绍：subprocess – 创建附加进程 ，subprocess是Python 2.4中新增的一个模块，它允许你生成新的进程，连接到它们的 input/output/error 管道，并获取它们的返回（状态）码。

一、常用的函数

  - subprocess .run()
    - 功能：执行指定的命令，等待命令执行完成后返回一个包含执行结果的CompletedProcess类的实例。
  - subprocess .call()
    - 功能：执行指定的命令，返回命令执行状态，其功能类似于os.system(cmd)。
  - subprocess .check_call()
    - 功能：执行指定的命令，如果执行成功则返回状态码，否则抛出异常。
  - subprocess .check_output()
    - 功能：执行指定的命令，如果执行状态码为0则返回命令执行结果，否则抛出异常。
  - subprocess .getoutput(cmd)
    - 功能：接收字符串格式的命令，执行命令并返回执行结果，其功能类似于os.popen(cmd).read()。
  - subprocess .getstatusoutput(cmd)
    - 功能：执行cmd命令，返回一个元组(命令执行状态, 命令执行结果输出)。

二、函数参数说明

  -   subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, shell=False, timeout=None, check=False, universal_newlines=False)
  - subprocess.call(args, *, stdin=None, stdout=None, stderr=None, shell=False, timeout=None)
  - subprocess.check_call(args, *, stdin=None, stdout=None, stderr=None, shell=False, timeout=None)
  - subprocess.check_output(args, *, stdin=None, stderr=None, shell=False, universal_newlines=False, timeout=None)
  - subprocess.getstatusoutput(cmd)
  - subprocess.getoutput(cmd)  
  - 参数说明：
    - args： 要执行的shell命令，默认应该是一个字符串序列，如['df', '-Th']或('df', '-Th')，也可以是一个字符串，如'df -Th'，但是此时需要把shell参数的值置为True。
    - shell： 如果shell为True，那么指定的命令将通过shell执行。如果我们需要访问某些shell的特性，如管道、文件名通配符、环境变量扩展功能，这将是非常有用的。当然，python本身也提供了许多类似shell的特性的实现，如glob、fnmatch、os.walk()、os.path.expandvars()、os.expanduser()和shutil等。
    - check： 如果check参数的值是True，且执行命令的进程以非0状态码退出，则会抛出一个CalledProcessError的异常，且该异常对象会包含 参数、退出状态码、以及stdout和stderr(如果它们有被捕获的话)。
    - stdout, stderr：
      run()函数默认不会捕获命令执行结果的正常输出和错误输出，如果我们向获取这些内容需要传递subprocess.PIPE，然后可以通过返回的CompletedProcess类实例的stdout和stderr属性或捕获相应的内容；
    - call()和check_call()函数返回的是命令执行的状态码，而不是CompletedProcess类实例，所以对于它们而言，stdout和stderr不适合赋值为subprocess.PIPE；
    - check_output()函数默认就会返回命令执行结果，所以不用设置stdout的值，如果我们希望在结果中捕获错误信息，可以执行stderr=subprocess.STDOUT。
    - input： 该参数是传递给Popen.communicate()，通常该参数的值必须是一个字节序列，如果universal_newlines=True，则其值应该是一个字符串。
    - universal_newlines： 该参数影响的是输入与输出的数据格式，比如它的值默认为False，此时stdout和stderr的输出是字节序列；当该参数的值设置为True时，stdout和stderr的输出是字符串。

三、subprocess.CompletedProcess类介绍

  - 说明：subprocess.run()函数是python3.5新增的一个高级函数，其返回值是一个sbprocess.CompletedProcess类的实例，因此，我们知道它表示的是一个已结束进程的状态信息。
  - 属性：
    - args： 用于加载该进程的参数，这可能是一个列表或一个字符串
    - returncode： 子进程的退出状态码。通常情况下，退出状态码为0则表示进程成功运行了；一个负值-N表示这个子进程被信号N终止了
    - stdout： 从子进程捕获的stdout。这通常是一个字节序列，如果run()函数被调用时指定
    - universal_newlines=True，则该属性值是一个字符串。如果run()函数被调用时指定
    - stderr=subprocess.STDOUT，那么stdout和stderr将会被整合到这一个属性中，且stderr将会为None
      注：stderr： 从子进程捕获的stderr。它的值与stdout一样，是一个字节序列或一个字符串。如果stderr灭有被捕获的话，它的值就为None
    - check_returncode()： 如果returncode是一个非0值，则该方法会抛出一个CalledProcessError异常。

四、Popen

  - 在一些复杂的场景中，我们需要将一个进程的执行输出作为另一个进程的输入。在另一些场景中没我们，我们需要先进入到某个环境中，然后执行一系列的指令等。这个时候，我们就需要使用到subprocess的Popen()方法。

  - 结构：`Popen(args, bufsize=0, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=False, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0`

  - 参数说明：

    - args：shell命令，可以是字符串，或者序列类型，如list,tuple。
    - bufsize`
      等同于`open`函数的`buffering`参数。设为0表示无缓存。设为1表示行缓存。大于1表示使用这个size的缓冲区。负数表示系统默认，可不用关心。
    - `executable`
      有一些晦涩而不常使用的作用，比较有用的是`shell = True`时会用它来指定用的shell。
    - stdin,stdout,stderr：分别表示程序的标准输入，标准输出及标准错误
    - preexec_fn` (POSIX)
      这个在下面的*强制终止进程*的讨论中会用到，其作用是在子程序执行前在**子程序上下文**中需要执行的语句，实际上是一个Hook。
    - `close_fds` (POSIX)
      对于Linux，子进程在执行前会先关闭除标准输入输出外的所有fd。我们使用以下的代码来测试这个特性
    - `cwd`
      这个设置子线程的当前目录
    - `env`
      这个设置子线程的环境变量，如果env=None，则默认从父进程继承环境变量
    - `universal_newlines`
      不同系统的的换行符不同，当该参数设定为true时，则表示使用\n作为换行符 `
    - `startupinfo` (Windows)
      Windows的`CreateProcess`的对应参数

  - 实例

    - 实例一

      ```python
      示例1，在/root下创建一个suprocesstest的目录：
      a = subprocess.Popen('mkdir subprocesstest',shell=True,cwd='/root')
      ```

    - 实例二

      ```python
      示例2，使用python执行几个命令：
      import subprocess
      
      obj = subprocess.Popen(["python"], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
      
      obj.stdin.write('print 1 \n')
      obj.stdin.write('print 2 \n')
      obj.stdin.write('print 3 \n')
      obj.stdin.write('print 4 \n')
      
      
      out_error_list = obj.communicate()
      print (out_error_list)
      ```

    - 实例三

      ```python
      示例3，将一个子进程的输出，作为另一个子进程的输入：
      import subprocess
      
      child1 = subprocess.Popen(["cat","/etc/passwd"], stdout=subprocess.PIPE)
      
      child2 = subprocess.Popen(["grep","0:0"],stdin=child1.stdout, stdout=subprocess.PIPE)
      
      out = child2.communicate()
      ```

    - 其他操作

      ```python
      import subprocess
      child = subprocess.Popen('sleep 60',shell=True,stdout=subprocess.PIPE)
      child.poll()    #检查子进程状态
      child.kill()     #终止子进程
      child.send_signal()    #向子进程发送信号
      child.terminate()   #终止子进程
      '''
      与进程的单向通信 
      通过Popen()方法调用命令后执行的结果,可以设置stdout值为PIPE，再调用communicate()获取结果 返回结果为tuple. 在python3中结果为byte类型，要得到str类型需要decode转换一下
      '''
      # 直接执行命令输出到屏幕
      subprocess.Popen("ls -l",shell=True)
      # 不输出到屏幕,输出到变量
      proc = subprocess.Popen(['echo','"Stdout"'],stdout=subprocess.PIPE)
      #将结果输出到文件
      file_handle = open("/home/ws/t.log",'w+')
      
      subprocess.Popen("ls -l",shell=True,stdout=file_handle)
      # 写入到输入管道
      proc.stdin.write(msg)
      # 以下实现打开python3的终端，执行一个print命令
      proc = subprocess.Popen(['python3'],stdin=subprocess.PIPE,stdout=subprocess.PIPE, stderr=subprocess.PIPE,)
      
      proc.stdin.write('print("helloworld")'.encode('utf-8'))
      out_value,err_value=proc.communicate()
      
      print(out_value)
      #捕获错误输出
      proc = subprocess.Popen(['python3'],stdin=subprocess.PIPE,stdout=subprocess.PIPE, stderr=subprocess.PIPE,)
      
      proc.stdin.write('print "helloworld"'.encode('utf-8'))
      ```

    - Popen其他方法

      ```python
      Popen.pid 查看子进程ID
      Popen.returncode 获取子进程状态码,0表示子进程结束,None未结束
      
      #实例应用
      在使用Popen调用系统命令式，建议使用communicate与stdin进行交互并获取输出(stdout），这样能保证子进程正常退出而避免出现僵尸进程。看下面例子
      
      
      proc = subprocess.Popen('ls -l', shell=True, stdout=subprocess.PIPE)
      # 当前子进程ID
      proc.pid
      28906
      # 返回状态为None，进程未结束
      print(proc.returncode)
      None
      # 通过communicate提交后
      out_value = proc.communicate()
      proc.pid
      28906
      # 返回状态为0，子进程自动结束
      print(proc.returncode)
      0 
      ```

  **总结**

  ```
  那么我们到底该用哪个模块、哪个函数来执行命令与系统及系统进行交互呢？下面我们来做个总结：
  
  首先应该知道的是，Python2.4版本引入了subprocess模块用来替换os.system()、os.popen()、os.spawn*()等函数以及commands模块；也就是说如果你使用的是Python 2.4及以上的版本就应该使用subprocess模块了。
  
  如果你的应用使用的Python 2.4以上，但是是Python 3.5以下的版本，Python官方给出的建议是使用subprocess.call()函数。Python 2.5中新增了一个subprocess.check_call()函数，Python 2.7中新增了一个subprocess.check_output()函数，这两个函数也可以按照需求进行使用。
  
  如果你的应用使用的是Python 3.5及以上的版本（目前应该还很少），Python官方给出的建议是尽量使用subprocess.run()函数。
  当subprocess.call()、subprocess.check_call()、subprocess.check_output()和subprocess.run()这些高级函数无法满足需求时，我们可以使用subprocess.Popen类来实现我们需要的复杂功能。  
  ```

  

