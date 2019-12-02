---
title: 第二十四讲——面向对象之成员、方法、属性、元类type
id: 24
date: 2019-8-12 20:00:00
tags: Python
comment: true
---

### 学习大纲

- 私有成员和共有成员（对比学习）
- 类方法和实例方法（对比学习）
- 静态方法
- 属性
- issubclass、isinstance
- 元类type
- 练习

<!-----more----->

### 私有成员和共有成员（对比学习）

```python
#私有类的属性
# class A:
# 	name = "pl"
# 	__name = "pl1" #类的私有属性  加盐：_A__name
#
# 	def func(self):
# 		print (self.name)
# 		print (self.__name)
# obj = A()
# obj.func()

#类的对象的私有属性
# class A:
# 	def __init__(self,name,pwd):
# 		self.name = name
# 		self.__pwd = pwd  #类的私有对象属性
#
# 	def md5(self):
# 		self.__pwd = self.__pwd + '123'
# obj = A('pl','pl123')
# # print (obj.__pwd)  #报错

#类的私有方法
class A:
	def func(self):
		self.__func()
		print ("in A func")

	def __func(self):
		print ("in A __func")
obj = A()
obj.func()
# obj.__func() #类外方法类的私有方法报错
#总结：类的私有的属性以及类的私有的对象属性和方法，只能在类的内部使用，不能在类的外部以及
#派生类中使用（就是也不支持继承的类使用）

# 一般来说,私有成员一般用在什么地方？
# 当我们遇到重要的数据，功能的时候（只允许在本类中使用的一些方法，数据）就可以把他们设置成私有成员。

# 当然，万事无绝对
# python所有的私有成员都是纸老虎，形同虚设（为什么会这样说？）
# 因为私有成员一样是可以和共有成员一样被调用
class A:
	name = "pl"
	__name = "pl1"

	def __func(self):
		print ("in A __func")
print (A.__dict__)
# { 'name': 'pl', '_A__name': 'pl1', '_A__func': <function A.__func at 0x000001D792363840>, }
#我们可以看见A的属性中的私有属性和方法前面都加上了 (_类名),因此我们通过使用共有成员的那种方式
#去调用是肯定取不到的，所以我们就是用下面的方法可以取到。
print (A._A__name)
obj = A()
obj._A__func()
#总结：类从加载时，只要遇到类中的私有成员，都会在私有成员前面加上_类名。但是在工作中我们不会这样的，也是不允许的。
#调用，因为显得不专业。因为人家设置的原因就是不让你去调用，是吧！
```

### 类方法和实例方法（对比学习）

```python
类的其他成员就是类方法（对比分析）：
#实例方法：
	- 定义：第一个参数必须是实例对象，该参数名一般约定为“self”，通过它来传递实例的属性和方法（也可以传类的属性和方法）；
    -调用：只能由实例对象调用。
#类方法：
	-  定义：使用装饰器@classmethod。第一个参数必须是当前类对象，该参数名一般约定为“cls”，通过它来传递类的属性和方法（不能传实例的属性和方法）；
    -调用：实例对象和类对象都可以调用。
#静态方法：
	- 定义：使用装饰器@staticmethod。参数随意，没有“self”和“cls”参数，但是方法体中不能使用类或实例的任何属性和方法；
    -调用：实例对象和类对象都可以调用。
#双下划线方法：
	-定义：双下方法是特殊方法，他是解释器提供的 由爽下划线加方法名加爽下划线 __方法名__的具有特殊意义的方法,双下方法主要是python源码程序员使用的，
    -　　我们在开发中尽量不要使用双下方法，但是深入研究双下方法，更有益于我们阅读源码。
	-调用：不同的双下方法有不同的触发方式，就好比盗墓时触发的机关一样，不知不觉就触发了双下方法，例如：__init__


#类方法和实例方法：（使用装饰器@classmethod）
# 在我们前面学的都是类的实例方法，现在对比学习类的其他方法
# 首先，我们来看一个需求：创建一个学生类，只要是实例化一个对象，写一个类方法，统计一下集体实例化多少个学生
class Student:
	count = 0
	def __init__(self,name,id):
		self.name = name
		self.id = id
		Student.counter()

	@classmethod    #类方法
	def counter(cls):
		cls.count += 1

	@classmethod   #类方法
	def get_counter(cls):
		return cls.count
#我们实例化3次
p1 = Student("pl",12)
p2 = Student("pl",12)
p3 = Student("pl",12)
print (Student.get_counter())
#3
# 解释：类方法：一般就是通过类名去调用的方法，并且自动将类名地址传给cls，
# 但是如果通过对象调用也是可以的，但传的地址还是类名地址，而不是对象地址。

# 类方法有什么用？
# 	——得到类名可以实例化对象
# 	——可以操作类的属性
```

### 静态方法

```python
#静态方法（使用装饰器@staticmethod)）
class A:
	def func(self):
		print ("实例方法")

	@classmethod
	def cls_func(cls):
		print("类方法")

	@staticmethod
	def static_func():
		print ("静态方法")
# 总结：静态是不依赖于对象与类的，其实静态方法就是函数，就是类外面的函数。
# 但是，为了保证代码的规范性，合理的划分，后续的维护性，就把外面的函数写到了类的里面
#也就是我们所谓的静态方法，但是调用的时候需要借助我们的类名
```

### 属性

```python
属性存在的意义：
方法属性时可以制造出和访问字段完全相同的假象，属性由方法衍生而来，如果python中没有属性，方法完全可以代替其功能。
定义属性可以动态获取某个属性值，属性值由属性对应的方式实现，应用更灵活。
可以制定属于自己的属性规则，用于防止他人随意修改属性。

# 属性
# 在学习我们的属性之前，我们先提一个小的需求：写一个类实现测量我们的体脂率
# 公式：bmi = 体重（kg）/身高（m）**2


class BMI:
	def __init__(self,kg,height):
		self.kg = kg
		self.height = height

	def _BMI(self):
		return self.kg/self.height**2
pl = BMI(71,1.7)
print (f"{pl._BMI()}")
#我们肯定都是这样写的，虽然我们的结果实现了，但是感觉上是不合理的。bmi应该是类似于
# name，height，age这样的名词的。但是我们把它当作方法使用了，可以大家觉得这不就是扯淡么
# ，那么，我们往下面看


class Bmi:
	def __init__(self,name,height,weight):
		self.name =name
		self.height = height
		self.weight = weight

	@property
	def bmi(self):
		return self.weight/self.height**2

obj = Bmi("pl",1.80,73)
print (obj.bmi)
# 22.530864197530864
# 解释：@property将执行的一个函数需要函数名（）变成直接函数名
# 将动态方法伪装成一个属性，虽然在代码级别上没有什么提升，但是看起来更加合理
#说白了，就是将一个名称的方法装饰一下，在调用的时候使用和属性一样的调用方式。

# 属性进阶（property是一个组合）
由于新式类中具有三种访问方式，我们可以根据他们几个属性的访问特点，分别将三个方法定义为对同一个属性：获取、修改、删除
class Foo:
	@property
	def bmi(self):
		print ('get的时候运行')

	@bmi.setter
	def bmi(self,value):
		print ("set的时候使用")

	@bmi.deleter
	def bmi(self):
		print ("del的时候使用")
p1 = Foo()
p1.bmi
p1.bmi = 666  #操作命令，这个命令并不是改变bmi的值，而是执行被bmi.setter装饰器装饰的函数
del p1.bmi    #操作命令，执行被装饰的bmi。deleter函数
#应用场景：
	# --1，面试会考一些基本的调用，流程
	# --2，工作中如果遇到了一些类似于属性的方法名，可以让其伪装成属性

# 设置属性的两种方式
# 第一种：利用装饰器设置属性，就是上面的代码
# 第二种：利用实例化对象的方式设置属性
class Foo:
    def get_AAA(self):
        print('get的时候运行我啊')

    def set_AAA(self,value):
        print('set的时候运行我啊')

    def delete_AAA(self):
        print('delete的时候运行我啊')

    AAA = property(get_AAA,set_AAA,delete_AAA) #内置property三个参数与get,set,delete一一对应

f1=Foo()
f1.AAA
f1.AAA='aaa'
del f1.AAA

python的内置类属性：
__dict__:类的属性，获取类的所有信息。
__doc__:类的文档字符串。
__name__:类名。
__moudel__:类定义所在的模块。
__bases__:类的所有父类。
```

### issubclass、isinstance

```python
# isinstance 判断的是对象与类的关系
class A:
    pass

class B(A):
    pass

obj = B()

# isinstance(a,b) 判断的是 a是否是b类 或者 b类派生类 实例化的对象.
# print(isinstance(obj, B))  # True
# print(isinstance(obj, A))  # True



# issubclass 类与类之间的关系

class A:
    pass

class B(A):
    pass

class C(B):
    pass

# isinstance(a,b) 判断的是 a类是否是b类 或者 b类派生类 的派生类.
# issubclass(a,b) 判断的是 a类是否是b类 子孙类.
# print(issubclass(B,A))
# print(issubclass(C,A))
实例：
需求：list str tuple dict等这些类与 Iterble类 的关系是什么？
from collections import Iterable

print(isinstance([1,2,3], list))  # True
print(isinstance([1,2,3], Iterable))  # True
print(issubclass(list,Iterable))  # True

# 由上面的例子可得，这些可迭代的数据类型，list str tuple dict等 都是 Iterable的子类。
```

总结：

```python
class A:
#私有成员与共有成员
    company_name = '老男孩教育'  # 静态变量(静态字段)
    __iphone = '1353333xxxx'  # 私有静态变量(私有静态字段)

    def __init__(self,name,age): #特殊方法

        self.name = name  #对象属性(普通字段)
        self.__age = age  # 私有对象属性(私有普通字段)

    def func1(self):  # 普通方法
        pass

    def __func(self): #私有方法
        print(666)

#类方法
    @classmethod  # 类方法
    def class_func(cls):
        """ 定义类方法，至少有一个cls参数 """
        print('类方法')

#静态方法
    @staticmethod  #静态方法
    def static_func():
        """ 定义静态方法 ，无默认参数"""
        print('静态方法')

#属性
    @property  # 属性
    def prop(self):
        pass
```

### 元类type

```python
按照python的一切皆对象理论，类其实也是一个对象，那么类这个对象时从哪里实例化出来的呢？
print(type('abc'))
print(type(True))
print(type(100))
print(type([1, 2, 3]))
print(type({'name': '太白金星'}))
print(type((1,2,3)))
print(type(object))

class A:
    pass

print(isinstance(object,type))
print(isinstance(A, type))

<class 'str'>
<class 'bool'>
<class 'int'>
<class 'list'>
<class 'dict'>
<class 'tuple'>
<class 'type'>
True
True
type元类是获取该对像从属于的类，而type类比较特殊，python原则是：一切皆对象，其实类也可以理解为“对象”，而type元类又称作构建类，python中大多数内置的类（包括object）以及自己定义的类，都是由type元类创造的。

* 而type类与object类之间的关系比较独特：object是type类的实例，而type类是object类的子类，这种关系比较神奇无法使用python的代码表述，因为定义其中一个之前另一个必须存在。所以这个只作为了解。
```

### 练习

```python
# 面向对象的私有成员有什么？
"""
类的私有属性
类对象的私有属性
类的私有方法
""
```

```python
# 如何设置面向对象的类方法与静态方法，类方法静态方法有什么用？
"""
类方法：@classmethod
静态方法：@staticmethod
类方法作用：得到类名可以操作实例化对象、可以操作类的属性
静态方法：保证代码的规范性，合理的划分以及后续代码的维护
"""
```

```python
# 面向对象中属性是什么？有什么作用？
"""
属性就是将动态方法伪装成一个属性，在我们调用的时候我们可以像调用类的属性那样
其实就是让它看起来更加的合理，符合我们的认知习惯。
"""
```

```python
# isinstance与issubclass的作用是什么？
"""
isinstance(a,b):判断a是b类或者b的派生类的实例化的对象.。
insubclass(a,b):判断a类是b类的子孙类。
"""
```

```python
# 看代码写结果：

class A:
    a = 1
    b = 2
    def __init__(self):
        c = 3

obj1 = A()
obj2 = A()
obj1.a = 3
obj2.b = obj2.b + 3
print(A.a)
#1
print(obj1.b)
#2
# print(obj2.c)
#报错
```

```python
# 看代码写结果：

class Person:
    name = 'aaa'

p1 = Person()
p2 = Person()
p1.name = 'bbb'  #实例化对象不能修改类的属性，只能查看，但是会在自己的空间中创建一份新的内容
print(p1.name,id(p1.name))
# 'bbb'
print(p2.name,id(p2.name))
# 'aaa'
print(Person.name,id(Person.name))
# 'aaa'
bbb 2651017125760
aaa 2651015008304
aaa 2651015008304
```

```python
# 看代码写结果：    有坑

class Person:
    name = []
# 对象只能查看类中不可变的数据类型的属性，如果类的属性是可变的，可以通过实例修改类的属性
p1 = Person()
p2 = Person()
p1.name.append(1)   
print(p1.name,id(p1.name))
# [1]
print(p2.name,id(p2.name))
# [1]
print(Person.name,id(Person.name))
# [1]
[1] 2651015012680
[1] 2651015012680
[1] 2651015012680
分析：为什么会是同样的内容？p1调用类中的name的时候就是指向它的内存地址，当我们去给name添加值得时候，只是给类的name添加了一个值而不是修改了地址空间。
```

```python
# 看代码写结果：

class A:

    def __init__(self):
        self.__func()

    def __func(self):
        print('in A __func')


class B(A): #继承A，自动运行__init__.

    def __func(self):
        print('in B __func')


obj = B()
#in A __func

新的理解
# 看代码写结果：    重要

class A:

    def __init__(self,name):
	    self.name = name
	    self.__func()  #_A__func()


    def __func(self): #_A__func()
        print('in A __func')


class B(A): #继承A，自动运行__init__
	def __func(self): #_B__func()
		print('in B __func')


obj = B("pl")
#in A __func
"""
我们肯定会想为什么不去执行B中的__func而是去执行了A中的__func呢？
其实，每次执行到__这样的私有方法就会把__func转换成_类名__func，
所以在A中__init__中的__func()就先去B中找_A__func()，找不到就去A中找，最后找到了
"""
```

```python
# 看代码写结果：

class Init(object):

    def __init__(self,value): #self=iner
        self.val = value

class Add2(Init):

    def __init__(self,val): #self=iner
        super(Add2,self).__init__(val)
        self.val += 2

class Mul5(Init):

    def __init__(self,val): #self=iner
        super(Mul5, self).__init__(val)
        self.val *= 5

class Pro(Mul5,Add2):  #多继承，按照mro数序，pro继承Mul5，mul5继承Add2
    pass

class Iner(Pro):
    csup = super(Pro)
    def __init__(self,val): #self=p
        self.csup.__init__(val)
        self.val += 1
# 虽然没有见过这样的写法，其实本质是一样的，可以按照你的猜想来。
p = Iner(5)
print(p.val)
#36
```

```python
'''
请按下列要求，完成一个商品类。

封装商品名，商品原价，以及折扣价。
实现一个获取商品实际价格的方法price。
接下来完成三个方法，利用属性组合完成下列需求：
利用属性property将此price方法伪装成属性。
利用setter装饰器装饰并实现修改商品原价。
利用deltter装饰器装饰并真正删除原价属性。
'''
class Goods:
	def __init__(self,name,original_price,discount_price):
		self.name = name
		self.original_price = original_price
		self.discount_price = discount_price

	def price(self):
		print (self.original_price)

	@property
	def price(self):
		print (self.original_price)

	@price.setter
	def price(self,dis):
		print (dis)

	@price.deleter
	def price(self):
		del self.original_price

obj = Goods("牛仔裤",300,100)
obj.price
obj.price = 200
del obj.price
print (obj.__dict__)
```

