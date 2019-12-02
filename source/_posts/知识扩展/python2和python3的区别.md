---
title: python2和python3的区别
id: 3
date: 2019-11-15 20:30:00
tags: 知识扩展
comment: true
---

区别一：

```python
默认编码 
py2默认为 ascII 编码 不支持汉字输入和输出 如果想要支持 在首行加上 # encoding:utf-8

py3默认就是utf-8
```

区别二：

```python
输入input
py2用的是：rawinput 

py3用的是：input
```

<!----more---->

区别三：

```python
打印print
python2的print可以不用括号-->print "hello"

python3必须有-->print("hello")

在python2中是使用的print是语句，
在python3中使用的是print()函数。
```

区别四：

```python
python2和python3的unicode编码

python2的时候没有str和bytes的区别

python3就有了，于是就有了encode和decode（他们就是上述的类型的转换）
```

区别五：

```
Python2.2起，如果整数发生溢出，Python会自动将整数数据转换为长整数(long)，所以如今在长整数数据后面不加字母L也不会导致严重后果了。

在Python3里不再有long类型了，全都是int
```

区别六：

```
range和xrange
python2中有range和xrange

但是python3中只有range
```

区别七：

```
package包
package 在py2 中必须有__init__方法，

但是在py3中可以没有
```

区别八：

```
对于betys和str类型，
在3.0版本之前不区分betys和str类型，内部隐式自动切换

在3.0版本之后不再隐形的自动切换了，而是需要我们使用decode(“ascall”)和encode(“utf-8”)
```

区别九：

```python
try except 语句的变化

2版本: 
try:
   ......
except Exception, e :
   ......

3版本：
try:
   ......
except Exception as e :
   ......
```

区别十:

```python
打开文件

2版本： 
    file( ..... )
    或 open(.....)

3版本：
	只能用 open(.....)
```

区别十一： 

```
chr(K) 与 ord(c)
python 2.4.2以前
   chr(K)   将编码K 转为字符，K的范围是 0 ~ 255
   ord(c)   取单个字符的编码, 返回值的范围: 0 ~ 255
python 3.0
   chr(K)   将编码K 转为字符，K的范围是 0 ~ 65535
   ord(c)   取单个字符的编码, 返回值的范围: 0 ~ 65535
```

区别十二：

```python
除法运算符

数据类型的不同，在2.x下数字的数据类型有int和long，在3.x之后就中有float

python 2.4.2以前
   10/3      结果为 3     

python 3.0
   10 / 3 结果为 3.3333333333333335
   10 // 3 结果为 3
```

区别十三：

```
不等号的区别：

在python2中，不等号有两种写法：!=  和<>  

在python3中，不等号只剩下了 !=这一种
```

