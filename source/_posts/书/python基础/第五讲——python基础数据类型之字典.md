---
title: 第五讲——python基础数据类型之字典
id: 5
date: 2019-7-24 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 字典
- 练习

<!-----more----->

### 字典

##### 基本了解

- 我们学了列表知道了它可以储存大量的数据，但是数量巨大的查询 的时候就会很慢；而且列表只能按照顺序储存，数据间的关联性不强。于是我们就今天学的dict这种数据类型就很好的解决了上述的缺点。

- 字典是无序的，可变的。

- 在python3.5版本之前，字典是无序的。但是在python3.6之后，字典会按照初建字典时的顺序排序。

- 不可变（可哈希）的数据类型：int，str，bool，tuple。

   可变（可哈希）的数据类型：list，dict，set。

- 字典用于存储大量的数据，将数据与数据之间进行关联。

- 字典的的速度比列表快，并且在python的基本类型中用的是最多的。

##### 字典的储存

- python中的字典有很多 的名称，比如（“映射“，”哈希“，”散列“，”关系数组“）等。

- 知道哈希（Hash）：哈希就是将消息或者数据压缩成摘要，使得数据量变小，将数据的格式固定下来。该函数将数据打乱混合，重新创建一个叫做散列值得指纹，散列值通常用来代表一个短的随机字母和数字组成的字符串。有很多朋友不清楚什么是哈希，这里有个链接大家可以去深入的了解一下。https://baike.baidu.com/item/Hash/390310?fr=aladdin

- 图例：![img](https://xxx.ilovefishc.com/forum/201403/21/1646027llm8gff7ghwn6f7.png)

- 字典为什么消耗电脑的空间：

   ```python
   由于字典使用了散列表，而散列表又必须是稀疏的，这导致它在空间上的效率低下。举例而言，如果你需要存放数量巨大的记录，那么放在由元组或是具名元组构成的列表中会是比较好的选择；最好不要根据 JSON 的风格，用由字典组成的列表来存放这些记录。用元组取代字典就能节省空间的原因有两个：其一是避免了散列表所耗费的空间，其二是无需把记录中字段的名字在每个元素里都存一遍。记住我们现在讨论的是空间优化。如果你手头有几百万个对象，而你的机器有几个GB 的内存，那么空间的优化工作可以等到真正需要的时候再开始计划，因为优化往往是可维护性的对立面。
   ```

- 通过以上的知识我们知道了：

   - 字典的键必须是可哈希的（就是不可变的数据类型，比如字符串，元组，数值）。
   - 而且键是唯一的，如果定义的一样，后面的键会覆盖掉前面的键。因为哈希是一样的，就是执行的内存地址是一样的。

- 字典的创建

   ```python
   # 方式1:
   dic = dict((('one', 1),('two', 2),('three', 3)))
   # dic = dict([('one', 1),('two', 2),('three', 3)])
   print(dic) 
   #>>>{'one': 1, 'two': 2, 'three': 3}
   
   # 方式2:
   dic = dict(one=1,two=2,three=3)
   print(dic)  
   #>>>{'one': 1, 'two': 2, 'three': 3}
   
   # 方式3:
   dic = dict({'one': 1, 'two': 2, 'three': 3})
   print(dic)  
   #>>>{'one': 1, 'two': 2, 'three': 3}
   
   # 方式4:拉链（后面会讲到）
   dic = dict(zip(['one', 'two', 'three'],[1, 2, 3]))
   print(dic)
   #>>>{'one': 1, 'two': 2, 'three': 3}
   
   # 方式5: 字典推导式 后面会讲到
   dic = { k: v for k,v in [('one', 1),('two', 2),('three', 3)]}
   print(dic)
   #>>>{'one': 1, 'two': 2, 'three': 3}
   
   # 方式6:利用fromkey后面会讲到。(共享值)
   dic = dict.fromkeys('abcd','pl')
   print(dic)  
   #>>>{'a': 'pl', 'b': 'pl', 'c': 'pl', 'd': 'pl'}
   ```

##### 字典的增删改查

```python
#增加
dic = {"首都":"北京","a":123,"s":"下"}
dic["name"] = "pl" #暴力添加
dic.setdefault("s","s")  #函数（无键添加，有键不变）
print (dic)
#>>>{'首都': '北京', 'a': 123, 's': '下', 'name': 'pl'}

#删除
dic = {"首都":"北京","a":123,"s":"下"}
del dic["s"]  #del 索引修改
print (dic)
#>>>{'首都': '北京', 'a': 123}
dic.pop("a")  #按照键去删除，有返回值
print (dic)
#>>>{'首都': '北京'}
dic.clear()   #清空字典
print (dic)
#>>>{}
dic = {"首都":"北京","a":123,"s":"下"}
dic.popitem() #在python3.5之前随机删除，在3.6之后删除最后一个，有返回值。
print (dic)
#>>>{'首都': '北京', 'a': 123}
#修改
dic = {"首都":"北京","a":123,"s":"下"}
dic["s"] = "pl"  #和暴力添加一样，但是各种滋味，细细品味
print (dic)
#>>>{'首都': '北京', 'a': 123, 's': 'pl'}
dic.update({"a":456,"name":"pl"})  #函数
print (dic)
#>>>{'首都': '北京', 'a': 456, 's': 'pl', 'name': 'pl'}
#查
dic = {"首都":"北京","a":123,"s":"下"}
print (dic.get("首都"))  #get获得的就是即使字典中没有所查找的键，也不会报错，只是返回None
```

##### 解构

```python
#解构（也叫拆包）
a,b = 10,20
print (a,b)
a,b = [100,200]
print (a,b)
a,b = (1000,2000)
print (a,b)
a,_,b = (3,4,5)   #必须一一对应，但是有的时候我们只想要其中的某一些数据的时候，可以用其他的字符接收但是我们不去用它就可以了
print (a,b)
a = 10,20  #本质就是一个元组
print (a)
a,b ="pl"  #必须一一对应
print (a,b)
a,b = {"a":1,"b":2}
print (a,b)
```

##### 返回迭代器

- dict.keys()返回字典的所有键的高仿列表。

- dict.values()返回字典的所有值的高仿列表。

- dict.items()返回字典的所有键值对的高仿列表。

- 高仿列表转真正的列表：list（dict.keys()/dict.values()/dict.items()）。

- 那么高仿列表和真正列表之间有什么不同呢？

   - 顾名思义就是高仿列表具有一些和真正列表相同的属性，但是还有一些他们不具备的属性。

- 遍历查找字典的数据：

   ```python
   for i in dic:   #遍历字典的键
   	print (i,dic[i])
   for i in dic.items():#方式一：输出字典的键和值
       print (i[0],i[1])
   for k,v in dic.items():#方式二：输出字典的键和值
       print (k,v)
   ```

注：一般遍历字典的时候，推荐使用第一种方式，第二种方式需要进行转换，效率比较低。所谓的高仿就是拥有一些真品的属性。

##### 字典的嵌套

```python
dic = {
    'name':'汪峰',
    'age':48,
    'wife':[{'name':'国际章','age':38}],
    'children':{'girl_first':'小苹果','girl_second':'小怡','girl_three':'顶顶'}
}
1. 获取汪峰的名字。
name = dic['name']
print(name)
2.获取这个字典：{'name':'国际章','age':38}。
di = dic['wife'][0]
print(di)
3. 获取汪峰妻子的名字。
wife_name = dic['wife'][0]['name]
print(wife_name)
4. 获取汪峰的第三个孩子名字。
name = dic['children']['girl_three']
print(name)
```

##### 字典与列表的对比

1. list：顺序查找，速度随元素的增加而变慢，占用的内存少。
2. dict：查找速度快，不会随key增加而变慢，但是内存占用大。牺牲空间换取时间

##### 面试题

1. 用一行代码将下列的a,b进行数据交换。

   a = 10

   b = 20

   代码实现：

   print（a,b = b,a）

### 练习

```python
#1.请将列表中的每个元素通过 "_" 链接起来。
users = ['大黑哥','龚明阳',666,'渣渣辉']
users[2] = str(users[2])
print ("_".join(users))
```

```python
#2.请将元组 v1 = (11,22,33) 中的所有元素追加到列表 v2 = [44,55,66] 中。
v1 = (11,22,33)
v2 = [44,55,66]
for i in v1:
	v2.append(i)
print (v2)
```

```python
#3.请将元组 v1 = (11,22,33,44,55,66,77,88,99)中的所有偶数索引位置的元素追加到新列表中。
v1 = (11,22,33,44,55,66,77,88,99)
lst = []
for i in v1:
	if i%2 == 0:
		lst.append(v1.index(i))
print (lst)
```

```python
#4.将字典的键和值分别追加到key_list 和 value_list 两个列表中，如：
key_list = []
value_list = []
info = {'k1':'v1','k2':'v2','k3':'v3'}
key_list = list(info.keys())
value_list = list(info.values())
print (key_list,value_list)
```

```python
#5.字典
#a. 请循环输出所有的key
dic = {'k1': "v1", "k2": "v2", "k3": [11,22,33]}
for i in dic.keys():
	print(i)
#b. 请循环输出所有的value
for i in dic.values():
	print (i)
#c. 请循环输出所有的key和value
for i in dic.keys():
	print (i,dic[i])
#d. 请在字典中添加一个键值对，"k4": "v4"，输出添加后的字典
dic.setdefault("k4","v4")
print (dic)
#e. 请在修改字典中 "k1" 对应的值为 "alex"，输出修改后的字典
dic = {'k1': "v1", "k2": "v2", "k3": [11,22,33]}
dic.update({'k1': "alex", "k2": "v2", "k3": [11,22,33]})
print(dic)
#f. 请在k3对应的值中追加一个元素 44，输出修改后的字典
dic = {'k1': "v1", "k2": "v2", "k3": [11,22,33]}
lst = dic["k3"]
lst.append(44)
print (dic)
#g. 请在k3对应的值的第 1 个位置插入个元素 18，输出修改后的字典
dic = {'k1': "v1", "k2": "v2", "k3": [11,22,33]}
lst = dic["k3"]
lst.insert(0,18)
print (dic)

```

```python
#6.有如下字典,实现以下需求的内容
av_catalog = {
"欧美":{
"www.太白.com": ["很多免费的,世界最大的","质量一般"],
"www.alex.com": ["很多免费的,也很大","质量挺好"],
"oldboy.com": ["多是自拍,高质量图片很多","资源不多,更新慢"],
"hao222.com":["质量很高,真的很高","全部收费,屌丝请绕过"]
},
"日韩":{
"tokyo-hot":["质量怎样不清楚,个人已经不喜欢日韩范了","verygood"]
},
"大陆":{
"1024":["全部免费,真好,好人一生平安","服务器在国外,慢"]
}
}
#1)给此 ["很多免费的,世界最大的","质量一般"]列表第二个位置插入一个 元素：'量很大'。
av_catalog["欧美"]["www.太白.com"].insert(2,"量很大")
print (av_catalog)
#2)将此 ["质量很高,真的很高","全部收费,屌丝请绕过"]列表的 "全部收费,屌丝请绕过" 删除。
av_catalog["欧美"]["hao222.com"].remove("全部收费,屌丝请绕过")
print(av_catalog)
#3)将此["质量怎样不清楚,个人已经不喜欢日韩范了","verygood"]列表的 "verygood"全部变成大写。
av_catalog["日韩"]["tokyo-hot"][1].upper()
print (av_catalog)
#4)给 '大陆' 对应的字典添加一个键值对 '1048' :['一天就封了']
av_catalog["大陆"].setdefault('1048',['一天就封了'])
print (av_catalog)
#5)删除这个键值对："oldboy.com": ["多是自拍,高质量图片很多","资源不多,更新慢"]
av_catalog["欧美"].pop("oldboy.com")
print (av_catalog)
#6)给此["全部免费,真好,好人一生平安","服务器在国外,慢"]列表的第一个元素，加上一句话：'可以爬下来'
lst = av_catalog["大陆"]["1024"]
lst[0]  += '可以爬下来'
print(av_catalog)
```

```python
#7.请循环打印k2对应的值中的每个元素。
info = {
'k1':'v1',
'k2':[('alex'),('wupeiqi'),('oldboy')],
}
for i in info['k2']:
	print (i)
```

```python
#8.有字符串"k: 1|k1:2|k2:3 |k3 :4" 处理成字典 {'k':1,'k1':2....}
s = "k: 1|k1:2|k2:3 |k3 :4"
news  = "".join(s.split())
lst1 = news.strip('|').split('|')
lst2 = []
for i in lst1:
	lst2.append(i.split(':'))
	for i in lst2:
		i[-1] = int(i[-1])
print(dict(lst2))

#方法二：
s = "k: 1|k1:2|k2:3 |k3 :4|k4:8754 |k5:68623"
dic = {}
for i in s.replace(" ","").split("|"):
	k,v = i.split(":")
	dic[k] = int(v)
print (dic)

#拓展
s ="a = 1,b =2 ,c = 3"
newDic = dict(((lambda i:(i[0],int(i[1])))(i.split("="))  for i in (s.replace(" ","").split(","))))
#{'a': 1, 'b': 2, 'c': 3}
```

```python
#9.有如下值 li= [11,22,33,44,55,77,88,99,90] ,
# 将所有大于 66 的值保存至字典的第一个key对应的列表中，
# 将小于 66 的值保存至第二个key对应的列表中。
li= [11,22,33,44,55,77,88,99,90]
dic = {"key1":[],"key2":[]}
for i in li:
	if i > 66:
		dic["key1"].append(i)
	else:
		dic["key2"].append(i)
print (dic)
```

```python
#10.输出商品列表，用户输入序号，显示用户选中的商品
'''
商品列表：
goods = [
{"name": "电脑", "price": 1999},
{"name": "鼠标", "price": 10},
{"name": "游艇", "price": 20},
{"name": "美女", "price": 998}
]
要求:
1：页面显示 序号 + 商品名称 + 商品价格，如：
1 电脑 1999
2 鼠标 10
3 游艇 20
4 美女 998
2：用户输入选择的商品序号，然后打印商品名称及商品价格
3：如果用户输入的商品序号有误，则提示输入有误，并重新输入。
4：用户输入Q或者q，退出程序。
'''
goods = [
{"name": "电脑", "price": 1999},
{"name": "鼠标", "price": 10},
{"name": "游艇", "price": 20},
{"name": "美女", "price":998}
		]
while True:
	Count = 1
	goodsId = []
	print ("--------------------------------------------")
	for i in goods:
		goodsId.append(Count)
		print (Count,i["name"],i["price"])
		Count += 1
	print("--------------------------------------------")
	luru = input("请输入选择的商品号：")
	if luru in ["Q", "q"]:
		break
	if int(luru)  in goodsId:
		print(">>>",int(luru), goods[int(luru) - 1]["name"], goods[int(luru) - 1]["price"])
	else:
		print ("你输入的商品序号有误,请重新输入。")

 #10-1升级
while True:
	goodsId = []
	print ("--------------------------------------------")
	for i in range(len(goods)):
		print (i+1,goods[i]["name"],goods[i]["price"])
		goodsId.append(i+1)
	print("--------------------------------------------")
	luru = input("请输入选择的商品号：")
	if luru in ["Q", "q"]:
		break
	if luru.isdecimal():
		if int(luru)  in goodsId:
			print(">>>",goods[int(luru) - 1]["name"], goods[int(luru) - 1]["price"])
		else:
			print ("你输入的商品序号有误,请重新输入。")
	else:
		print("你输入的商品序号有误,请重新输入。")
```

```python
#11.看代码写结果(一定要先看代码在运行)
v = {}
for index in range(10):
	v['users'] = index
print(v)
#就是每次就把index索引到的值给字典v的键'users',直到循环结束，把最后字典输出打印出来。
```

