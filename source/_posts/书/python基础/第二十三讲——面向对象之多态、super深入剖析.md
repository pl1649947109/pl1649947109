---
title: 第二十三讲——面向对象之多态、super深入剖析
id: 23
date: 2019-8-11 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 面向对象的三大特征
- 鸭子类型（就是一个多态）
- 类的约束（也是一种多态）
- super深入剖析
- 练习

<!-----more----->

### 面向对象的三大特征

- 封装：把很多的数据封装到一个对像中。把固定的代码封装到一个代码块，函数，对象，打包成模块，这就属于封装的思想。
- 继承：子类可以自动拥有父类中除了私有属性外(就是父类的__init__函数内部的属性，使用的时候需要重构父类的__init__方法)的其他所有的内容。
- 多态：Python默认支持多态。对于弱类型的语言来说，变量并没有声明类型，因此同一个变量完全可以在不同的时间引用不同的对象。当同一个变量在调用同一个方法时，完全可能呈现出多种行为（具体呈现出哪种行为由该变量所引用的对象来决定），这就是所谓的多态（Polymorphism）。

### 鸭子类型

- 什么鸭子？你看起来像鸭子，那么你就是鸭子。

- 优点：
  - 好记。
  - 虽然A,B两个类没有关系，但是我统一两个类中相似方法的方法名，在某种意义上统一标准。
  - 型。

- ```
  class A:
  	def login(self):
  		pass
  	def register(self):
  		pass
  	def func(self):
  		pass
  class B:
  	def login(self):
  		pass
  	def register(self):
  		pass
  	def func1(self):
  		pass
  # A,B 就互为鸭子（A,B有相同的功能和相同的方法，但是他俩没有关系）
  x = A()
  x.login()
  x = B()
  x.login()
  #上面的同一个变量在不同的时间，调用同一个方法展现出来了不同的状态。
  ```

### 类的约束

```python
类的约束：其实就是父类对子类进行约束。子类必须要写xxx方法。
```

**类的约束第一种**

实例带入：

版本一

```python
# 版本一：完成一个支付功能
class QQpay:
    def pay(self,money):
        print('使用qq支付%s元' % money)

class Alipay:
    def pay(self,money):
        print('使用阿里支付%s元' % money)

a = Alipay()
a.pay(100)

b = QQpay()
b.pay(200)
```

版本二

```python
# 版本二：由于版本一使用的太不方便了，现在我们需求统一付款的方式
class QQpay:
    def pay(self,money):
        print('使用qq支付%s元' % money)

class Alipay:
    def pay(self,money):
        print('使用阿里支付%s元' % money)

def pay(obj,money):  # 这个函数就是统一支付规则，这个叫做： 归一化设计。
    obj.pay(money)

a = Alipay()
b = QQpay()

pay(a,100)
pay(b,200)
```

版本三

```python
版本三：由于这个开发者的离职，我们新来的开发人员来了，老板安排了
#一个任务就是在原来的基础上面扩展一个微信支付
class QQpay:
    def pay(self,money):
        print('使用qq支付%s元' % money)

class Alipay:
    def pay(self,money):
        print('使用阿里支付%s元' % money)

class Wechatpay:  # 野生程序员一般不会看别人怎么写，自己才是最好，结果......
    def fuqian(self,money):
        print('使用微信支付%s元' % money)

def pay(obj,money):
    obj.pay(money)

a = Alipay()
b = QQpay()

pay(a,100)
pay(b,200)

c = Wechatpay()
c.fuqian(300)
```

版本四

```
# 版本四：由于新来的人员写的代码和原来的不统一，需要重新修改
class Payment:
	"""
 此类什么都不做，就是制定一个标准，谁继承我，必须定义我里面的方法。
   """
	def pay(self,money):pass

class QQpay(Payment):
    def pay(self,money):
        print('使用qq支付%s元' % money)

class Alipay(Payment):
    def pay(self,money):
        print('使用阿里支付%s元' % money)

class Wechatpay(Payment):
    def fuqian(self,money):
        print('使用微信支付%s元' % money)


def pay(obj,money):
    obj.pay(money)

a = Alipay()
b = QQpay()

pay(a,100)
pay(b,200)

c = Wechatpay()
c.fuqian(300)
```

版本五（最终版本）

```python
# 版本五：（终结版本）我们在开发时为了防止后续的开发不统一，我们就
#在开发的时候做好控制
class Payment:
	"""
	此类什么都不做，就是制定一个标准，谁继承我，必须定义我里面的方法
	"""
	def pay(self,money):
		raise Exception("子类没有该方法.")

class QQpay(Payment):
	def pay(self,money):
		print (f"使用QQ支付{money}钱")

class Alipay(Payment):
	def pay(self,money):
		print (f"使用支付宝支付{money}钱")

class Wechat(Payment):
	def pay(self,money):
		print(f"使用微信支付{money}钱")
def pay(obj,money):
	obj.pay(None,money)

pay(QQpay,200)
"""
总结：提取父类，然后在父类中定义好方法。在这个方法中什么都不用干
就抛出一个异常就可以了，这样所有的子类都必须重写这个方法，否则，访问的
时候就会报错
"""
```

**类的约束的第二种**

```python
#第二种
from abc import  ABCMeta,abstractclassmethod
class Payment(metaclass=ABCMeta): #抽象类 接口类规范和约束
	#metaclass指的是一个元类
	@abstractclassmethod
	def pay(self,money): #抽象方法
		pass

class QQpay(Payment):
	def pay(self,money):
		print (f"使用QQ支付{money}钱")

class Alipay(Payment):
	def pay(self,money):
		print (f"使用支付宝支付{money}钱")

class Wechat(Payment):
	def pay(self,money):
		print(f"使用微信支付{money}钱")
def pay(obj,money):
	obj.pay(None,money)
"""
总结：使用元类来描述父类，在元类中给出一个抽象方法，这样子类就
不得不给出抽象方法的具体实现，也可以起到约束的效果。
抽像类和接口类做的事情：建立规范，指定一个类的metaclass
是ABCMate，那么这个类就变成了一个抽象类。
"""
```

### super深入剖析

```python
首先我们先区分什么事方法的重写？什么是重构？
什么是父类的重写？
子类扩展父类，子类是一种特殊的父类。大部分的时候，子类总是以父类为基础，额外增加新的方法，但在一些场景中，子类需要重写父类的方法。（针对的是实例方法）

什么是父类的重构？
python子类也是继承父类的构造方法，但如果子类有多个直接父类，那么就会优先选择排在最前面的父类的构造方法。（针对父类的构造方法，构造方法本质上也是一个实例方法）

python要求，如果子类重写了父类中的构造方法，那么子类的构造方法必须调用父类的构造方法。
子类的构造调用父类的构造方法有两种方式：
方式一：使用未绑定方法，就是通过父类的类名来调用。因为构造方法也是实例方法，当然可以通过这种方式来调用。
方式二：使用super()函数调用父类的构造方法。
注意：通过super()对象的方法既可以调用父类的实例方法，也可以调用父类的类方法


super的本质:
    self会首先调用自己的方法或者属性，当自身没有目标属性或方法时，再去父类中寻找；super会直接去父类中寻找目标属性或方法

```

单继承

```python
#实例1
class A:
	def f1(self):
		print('in A f1')

	def f2(self):
		print('in A f2')


class Foo(A):
	def f1(self):  #这种子类包含与父类同名的现象叫做方法重写，也称作方法覆盖。
		super().f2() #调用父类的f2方法
		print('in A Foo')


obj = Foo()
obj.f1()


```

多继承1

```python
class A:
    def f1(self):
        print('in A')

class Foo(A):
    def f1(self):
        super().f1()
        print('in Foo')

class Bar(A):
    def f1(self):
        print('in Bar')

class Info(Foo,Bar):
    def f1(self):
        super().f1() #这个是多继承，info先继承foo，所以先
        #执行foo，foo几册会给你bar，再执行bar，就是多重调用。
        print('in Info f1')

obj = Info()
obj.f1()

'''
in Bar
in Foo
in Info f1
'''
print(Info.mro())  # [<class '__main__.Info'>, <class '__main__.Foo'>, <class '__main__.Bar'>, <class '__main__.A'>, <class 'object'>]

```

多继承2

```python
class A:
    def f1(self):
        print('in A')

class Foo(A):
    def f1(self):
        super().f1()
        print('in Foo')

class Bar(A):
    def f1(self):
        print('in Bar')

class Info(Foo,Bar):
    def f1(self):
        super(Bar,self).f1()  #因为super跳过了foo，
        #所以一开始就执行Bar的f1，按照mro的顺序调用父类的同名方法。
        print('in Info f1')

obj = Info()
obj.f1()

```

**小结：**

1. 在我们单继承的时候我们学习了super就是重构父类的方法，但是学了多继承之后我们的重构父类的方法就要按照mro的继承顺序来执行。

### 练习

~~~python
#看代码写结果【如果有错误，则标注错误即可，并且假设程序报错可以继续执行】

class Foo(object):
    a1 = 1

    def __init__(self,num):
        self.num = num
    def show_data(self):
        print(self.num+self.a1)

obj1 = Foo(666)
obj2 = Foo(999)
print(obj1.num)
#666
print(obj1.a1)
#1

obj1.num = 18
obj1.a1 = 99

print(obj1.num)
#18
print(obj1.a1)
#99

print(obj2.a1)
#1
print(obj2.num)
#999
print(obj2.num)
#999
print(Foo.a1)
#1
print(obj1.a1)
#999



# 看代码写结果，注意返回值。

class Foo(object):

	def f1(self):
		return 999

	def f2(self):
		v = self.f1()
		print('f2')
		return v

	def f3(self):
		print('f3')
		return self.f2()

	def run(self):
		result = self.f3()
		print(result)


obj = Foo()
v1 = obj.run()
print(v1)
#f3
#f2
#99
#None 为什么会返回这个None，因为最后返回的999是run打印出来的，v1的没有返回值所以返回None



# 看代码写结果


class Foo(object):
	def __init__(self, num):
		self.num = num


v1 = [Foo for i in range(10)]
v2 = [Foo(5) for i in range(10)]
v3 = [Foo(i) for i in range(10)]

print(v1)
#[Foo的地址 * 10]
print(v2)
#[v2的地址 * 10]
print(v3)
#[v3的地址 * 10]



#看代码写结果

class StarkConfig(object):

    def __init__(self, num):
        self.num = num

    def changelist(self, request):
        print(self.num, request)


config_obj_list = [StarkConfig(1), StarkConfig(2), StarkConfig(3)]
for item in config_obj_list:
    print(item.num)

#1
#2
#3


# 看代码写结果：

class StarkConfig(object):

    def __init__(self, num):
        self.num = num

    def changelist(self, request):
        print(self.num, request)


config_obj_list = [StarkConfig(1), StarkConfig(2), StarkConfig(3)]
for item in config_obj_list:
    item.changelist(666)

#1 666
#2 666
#3 666



# 看代码写结果：

class Department(object):
	def __init__(self, title):
		self.title = title


class Person(object):
	def __init__(self, name, age, depart):
		self.name = name
		self.age = age
		self.depart = depart


d1 = Department('人事部')
d2 = Department('销售部')

p1 = Person('武沛齐', 18, d1)
p2 = Person('alex', 18, d1)
p3 = Person('安安', 19, d2)

print(p1.name)
#武沛齐
print(p2.age)
#18
print(p3.depart)
#d2对象
print(p3.depart.title)
#销售部



# 看代码写结果：

class Department(object):
	def __init__(self, title):
		self.title = title


class Person(object):
	def __init__(self, name, age, depart):
		self.name = name
		self.age = age
		self.depart = depart

	def message(self):
		msg = "我是%s,年龄%s,属于%s" % (self.name, self.age, self.depart.title)
		print(msg)


d1 = Department('人事部')
d2 = Department('销售部')

p1 = Person('武沛齐', 18, d1)
p2 = Person('alex', 18, d1)
p1.message()
#我是武沛齐,年龄18,属于人事部
p2.message()
#我是alex,年龄18,属于人事部



# 看代码写结果：

class A:
    def f1(self):
        print('in A f1')

class B(A):
    def f1(self):
        print('in B f1')

class C(A):
    def f1(self):
        print('in C f1')

class D(B, C):
    def f1(self):
        super(B, self).f1()
        print('in D f1')

obj = D()
obj.f1()
#in C f1
#in D f1



# 看代码写结果：

class A:
    def f1(self):
        print('in A f1')

class B(A):
    def f1(self):
        super().f1()
        print('in B f1')

class C(A):
    def f1(self):
        print('in C f1')

class D(B, C):
    def f1(self):
        super().f1()
        print('in D f1')

obj = D()
obj.f1()
#in C f1
#in B f1
#in D f1



'''
运用类完成一个扑克牌类(无大小王)的小游戏：
用户需要输入用户名，以下为用户可选选项:
    1. 洗牌
    2. 随机抽取一张
    3. 指定抽取一张
    4. 从小到大排序
    5. 退出

1. 洗牌：每次执行的结果顺序随机。
2. 随机抽取一张：显示结果为：太白金星您随机抽取的牌为：黑桃K
3. 指定抽取一张：
    用户输入序号（1~52）
    比如输入5，显示结果为：太白金星您抽取的第5张牌为：黑桃A
4. 将此牌从小到大显示出来。A -> 2 -> 3 .......-> K

提供思路：
    52张牌可以放置一个容器中。
    用户名，以及盛放牌的容器可以封装到对象属性中。
'''
import random
brand = [j+i for j in ["黑桃","红桃","梅花","方块"] for i in (['A']+list('23456789')+['10','J','Q','K'])]
class Puke:
	puke_brand = [j + i for j in ["黑桃", "红桃", "梅花", "方块"] for i in (['A'] + list('23456789') + ['10', 'J', 'Q', 'K'])]
	def __init__(self,brand,name,):
		self.name = name
		self.brand = brand

	def shuffle_the_cards(self):
		self.brand = random.shuffle(self.brand)

	def draw_cards(self):
		brand_one = random.choice(self.brand)
		print (f"{self.name}您随机抽取的牌为：{brand_one}")

	def appoint_draw_cards(self):
		self.shuffle_the_cards()
		while True:
			s = input("抽第几张牌：")
			if s.isdecimal():
				brand_two = brand[int(s)-1]
				print (f"{self.name}您抽取的第{s}张牌为:{brand_two}")
				break
	def sort(self):
		print (Puke.puke_brand)

	def out(self):
		return False


while True:
	print ("1.洗牌"
	       "2.随机抽取一张"
	       "3.指定抽取一张"
	       "4.从小到大排序"
	       "5.退出")
	jugde = input("功能选择：")
	if jugde.isdecimal():
		p1 = Puke(brand, "pl")
		select = list(enumerate([p1.shuffle_the_cards,p1.draw_cards,p1.appoint_draw_cards,p1.sort,p1.out],1))
		if int(jugde) in [i[0] for i in select]:
			re = select[int(jugde)-1][-1]()
			if re == False:
				break
~~~