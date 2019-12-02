---
title: git——分布式版本控制系统
id: 1
date: 2019-10-5 20:00:00
tags: git版本控制系统
comment: true
---

### 介绍

git：分布式版本控制系统  VS  cvs：集中式版本控制系统

​		集中式的工作方式就是一个主server，下面的人工作就去主server拿，改完再放回去。它最大的缺点就是必须联网才能工作，局域网内还好，但是在公网上网速一旦慢下来就非常的难受。集中式的工作方式就是一个主server，下面的人工作就去主server拿，改完再放回去。它最大的缺点就是必须联网才能工作，局域网内还好，但是在公网上网速一旦慢下来就非常的难受。

​		GIT是分布式的，不存在集中的server，每个人的电脑都是一个完整的版本库，工作的时候就不需要联网了，我们把修改的内容推送给团队的其他人员就可以进行内容的共享了。一般来说，分布式系统也有一台电脑充当中央服务器的功能，但这个服务器仅仅是用来方便交换大家的修改，没有它，大家一样的干活。为了就是方便。

​		Git的开发者就是linux的开发者linus，由于最早在linux上面开发的，所以开始只能在linux和unix上面跑。但是后来慢慢的可以移植到windows上面了。现在基本上可以在所有的平台上面跑起来了。

<!----more---->

#### github和gitlab的区别

- github是国外公共仓库，不安全，国内的码云代码仓库等。可能会暴露自己公司代码机密。
- gitlab是自建是有的代码仓库，更加安全。

#### gitlab的配置流程

https://www.cnblogs.com/pyyu/p/10164834.html

### 操作流程

#### 安装git

**linux下的安装**

```shell
sudo apt-get install git
```

**windows下的安装**

进入官网：https://git-scm.com/下载安装即可

#### 创建版本库

**什么是版本库？**

简单的理解就是位于自己主机上面的一个文件夹（目录），这个目录里面的所有文件都可以使用git管理起来，每个人文件的修改、删除等操作，git都能够跟踪（记录），以便任何时候都可以追踪到历史或者将某个时刻还原。

**创建目录执行命令，初始化一个版本库**

```
git init 
```

**查看一下当前仓库中的文件或者文件夹的状态，新增的或者修改的都可以查看到（时刻掌握仓库当前的状态）**

```
git status    
```

**将文件提交到版本库**

```
git add xxx.txt     管理某一个文件
git add .			管理所有的文件
```

**为我们建立用户名并连接邮箱**

```
git config --global user.name "用户名"
git config --global user.email  "邮箱"
```

**git进行版本管理**

```
git commit -m "描述信息"   #描述信息就是版本的信息
```

#### 本地版本库命令

**显示最近到最远的提交日志**

```
git log    #查看commit id很有用
```

**返回之前的版本**

```
git reset --hard HEAD(这个是commit id)
#下面的这个实例就是从v3版本返回到v1版本
```

![](http://9017499461.linshutu.top/git1.JPG)

**记录输入的每一条命令，通过它可以返回之前返回的版本**

```
git reflog
```

![](http://9017499461.linshutu.top/git2.png)

**把指定文件的工作区的修改全部撤销**

```
git checkout "文件"

#场景一：
当我们乱改了工作区某个文件的内容，想直接丢弃工作区的修改时，就可以使用该命令git checkout "文件"

#场景二：当我们不但乱改了工作区某个文件的内容，，还添加到了缓存区，想要丢弃修改，我们可以分两步操作：
第一步命令：git reset HEAD file
第二步命令：git checkout "文件"
```

![](http://9017499461.linshutu.top/git.JPG)

#### 远程版本库命令

免费的仓库有github（国外的）和码云（国内的）

**关联一个远程版本库**

```
git　remote add origin "远程版本库的git地址"
#注意：远程版本库的地址可以直接拷贝过来，origin是起的别名

关联后，使用下述命令第一次推送master分支的所有内容：
git push -u origin master 
#注意：此后，每次提交后，只要有必要，就可以使用命令git push origin master推送最新的修改。当然，只要连接不出问题，我们的git commit的时候自动就会推送到远端
```

分布式系统的好处之一就是在本地工作完全不需要考虑远程版本库的存在，也就是你没有联网都可以正常的工作。

**从远端克隆版本库**

```
git clone "远端的版本库的git地址"
```

**下载远程仓库代码，合并到本地master分支**

```
git pull origin master  #一般push失败做这个操作就能成功push
```

#### 分支管理

**创建分支**

```
git　checkout -b dev    #创建并切换到dev分支
```

**查看当前所有的分支**

```
git branch
```

**分支之间的切换**

```
git checkout "指定的分支"
```

**将指定分支的工作成果合并到当前的分支上面**

```
git merge "指定的分支"
```

**删除分支**

```
git branch -d "指定分支"
```

**查看分支合并图**

```
git log --graph
```

#### 修复bug时的操作

保护现场

```
git stash
```

恢复现场

```
git stash pop
```

#### 多人协作

**查看远程版本库的信息**

```
git remote -v
```

**推送指定的分支**

```
git push origin "指定的分支"
```

当冲突的时候，冲突的内容和提示就会在我们的文件中展示出来，我们手动把那些特殊的标记去掉就可以了。还有一种方式就是：

集成外部软件（他就是一个文件对比软件）

第一步： 安装beyond compare软件，下载地址：http://www.scootersoftware.com/download.php，就直接点击下载安装，然后和安装其他软件一样，点点点就可以了。

第二步：在git中进行以下配置

```shell
git config --local merge.tool bc3 #--local的意思是只对当前项目有效，其他的本地仓库是不 生效的 
git config --local mergetool.path'/usr/local/bin/bcomp' #beyond compare的执行程 序的安装路径 
git config --local mergetool.keepBackup false
```

第三步：应用这个软件来解决冲当我们执行merge合并的时候，比如说，我们执行了一下git merge dev分支的指令，会报错，报一个代码冲突的错误，然后我们知道产生冲突了，此时我们就可以使用我们的beyond compare来进行冲突排查和修改，使用下面的指令来调用工具：

```
git mergetool
```

#### 标签管理

打标签

```
git tag v1.0
```

查看标签

```
git tag
```

查看说明

```
git show v1.0
```

### 新入职，新电脑，怎么下载代码？

```
clone项目
但是默认获取的只有master分支(git breach查看)
创建新的分支，并且和远程的dev分支同步，注意确保远程的github有dev分支，没有就提前创建好git breach dev origin/dev
切换到分支git checkout dev
开始在dev下面写新的代码
	git add .
	git commit -m "xxx"
提交代码到远程githun托管的dev分支git push origin dev
合并分支到master主干上，注意此时还是呆在dev分之下
	git checkout master
	git status
	git merge dev
	git push origin master
```

### Git忽略文件

我们让git管理文件的时候，可以设置一些让git忽略的文件或者文件夹，通过一个叫做 .gitignore 的文件，首先在我们的git本地仓库中创建一个这样的文件，文件中写我们忽略文件的名称。

还可以添加*.txt把所有以.txt结尾的文件全部忽略掉。

注意：在公司进行开发的时候，一定要加上.gitignore，不然容易将一些敏感的信息提交到远程，不安全。

### gitlab结合pychrm

https://www.cnblogs.com/pyyu/p/10165109.html

### github使用大全

https://www.cnblogs.com/pyyu/p/10160969.html



