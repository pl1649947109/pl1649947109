---
title: 第一讲——python数据分析初始
id: 1
date: 2019-10-5 20:30:00
tags: 数据分析
comment: true
---

## 初始

在数据分析和处理领域，毫无疑问，Python是主流语言，其原因在于：

- Python语法简单，代码量少
- Numpy、Scipy、Pandas和Matplotlib的科学计算生态圈过于强大
- Ipython和Jupyter notebook的交互式环境
- 容易整合C/C++/FORTRAN代码，使用过往的存量代码
- 从代码走向工程很快捷

<!----more---->

下面是Python数据分析和处理任务中重要的库与工具：

### Numpy

官网：http://www.numpy.org/

Numpy库是Python数值计算的基石。它提供了多种数据结构、算法以及大部分涉及Python数值计算所需的接口。主要包括以下内容：

- 快速、高效的多维数组对象ndarray
- 基于元素的数组计算或者数组间的数学操作函数
- 用于读写硬盘中基于数组的数据集的工具
- 线性代数操作、傅里叶变换以及随机数生成
- 成熟的C语言API，拓展代码

### Scipy

官网：https://www.scipy.org/

这个库是Python科学计算领域内针对不同标准问题域的包集合，主要包括以下内容：

- integrate:数值积分例程和微分方程求解器
- linalg：线性代数例程和基于numpy.linalg的矩阵分解
- optimize：函数优化器和求根算法
- signal:信号处理工具
- sparse:稀疏矩阵与稀疏线性系统求解器
- special：SPECFUN的包装其
- stats：标准的连续和离散概率分布

Scipy与Numpy一起为很多传统科学计算应用提供了一个合理、完整、成熟的科学计算基础。

###  Pandas

官网： http://pandas.pydata.org/

Pandas提供了高级数据结构和函数，使得利用结构化、表格化数据的工作快速、简单、有表现力。Pandas将表格和关系型数据库的灵活数据操作能力与Numpy的高性能数组计算的理解相结合。提供复杂的索引函数，使得数据的重组、切块、切片、聚合、子集选择更为简单。Pandas是数据分析和处理工作中，实际使用占比最多的工具，使用频率最高，也是本教程的主要介绍内容。

###  matplotlib

官网：https://matplotlib.org/

matplotlib是最流行的用于制图以及其它数据可视化的Python库。在基于Python的数据可视化工作中，这个库是行业默认选择，虽然也有其它可视化库，但matplotlib依然是使用最为广泛，并且与生态系统的其它库良好整合。

此工具是本教材主要介绍内容之一，实际上，学会了这个工具，其它可视化库，甚至Matlab绘图，基本套路都是类似的，可以一通百通。

## Jupyter notebook

官网：https://jupyter.org/

基于Python的交互式编程环境有IPython、IPython notebook以及Jupyter notebook。但如果对于数据分析、处理、机器学习等相关工作，我强烈推荐基于web的Jupyter notebook。

这个代码测试、开发、编辑、文字工具，真的是谁用谁知道，并且也是本教程的主要内容之一，吐血推荐！

**本质：一个基于web服务的浏览器应用！**

所以它有web服务器，有可以访问的url。

- 在浏览器中编辑代码，自动高亮语法、缩进和制表符完成/反省。
- 在浏览器中执行代码，计算结果附加到代码后。
- 支持如HTML、LaTex、PNG、SVG等大量富媒体格式显示计算结果。例如，Matplotlib库呈现的出版物质量图可以内联显示。
- 支持Markdown标记语言编辑富文本（可以为代码提供注释）不限于纯文本。
- 能够使用LaTex在标记单元中轻松编辑数学符号，并由Mathjax渲染。
- 支持包括Python, R, Julia, and Scala在内的超过40种编程语言。但是其本身是依赖Python的，需要安装Python环境才可以使用。
- 可以通过邮件, Dropbox, GitHub和Jupyter Notebook Viewer与他人分享。
- 丰富的输出格式，包括HTML、图片、视频、LaTeX印刷甚至自定义的MIME类型。
- 大数据一体化，与Spark，pandas，scikit-learn，ggplot2，TensorFlow等工具和框架密切配合

#### 安装

```
python3 -m pip install --upgrade pip
python3 -m pip install jupyter
```

#### 运行

```
在dos环境下：
#直接启动
jupyter notebook
#指定运行的端口
jupyter notebook --port 9999
#启动服务器但是不打开浏览器
jupyter notebook --no-browser
#查看服务器帮助内容
jupyter notebook --help
```

#### 运行机制

Jupyter notebook本质上是一个基于BS结构的Web服务器，使用Tornado框架开发。

![](http://9017499461.linshutu.top/jupyter%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6.png)

用户只和浏览器前端页面进行交互。浏览器通过HTTP和Websocket协议与后台Notebook的web服务器交互。web服务器通过ZMQ与编程语言核心进行计算和通信。web服务器对磁盘存储器上的Notebook文件进行读写。一切都是围绕Notebook的web服务器展开的。

#### 页面讲解

http://www.liujiangblog.com/course/data/191

#### 多人协作

每一次启动Jupyter notebook，程序都会随机生成一个token，复制这个字符串，输入就行了。Jupyter的多人协作模式，其实就是通过用户登录来实现的，多个用户可以同时使用一个服务器。

#### 模式

编辑模式：左边的框冒绿光

命令模式：左边的框冒蓝光，命令模式下，不可以对单元的内容进行修改。ESC按键或者鼠标点击单元格外面，可以进入命令模式。

#### 快捷键

http://www.liujiangblog.com/course/data/194

#### 视频和图片

http://www.liujiangblog.com/course/data/197

#### 魔法命令

魔法命令是用于控制notebook的特殊命令。它们运行在代码单元中，以`%`或者`%%`开头，前者控制单行，后者控制单元。

http://www.liujiangblog.com/course/data/200