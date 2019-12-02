---
title: 第二讲——drf分页
id: 3
date: 2019-11-6 20:00:00
tags: drf
comment: true
---

### 内容回顾与扩展

1.什么是restful规范？

```
是一套规则，用于程序之间进行数据交换的约定。 
他规定了一些协议，对我们感受最直接的的是，以前写增删改查需要写4个接口，restful规范的就是1 个接口，根据method的不同做不同的操作，比如：get/post/delete/put/patch/delete. 除此之外，resetful规范还规定了： 
	- 数据传输通过json 

扩展：前后端分离、app开发、程序之间（与编程语言无关）
```

```json
JSON:{ name:'alex', age:18, gender:'男' }
```

```xml
以前用webservice协议，数据传输格式xml。<name>alex</name> 
<age>alex</age> 
<gender>男</gender>
```
<!----more---->
2.drf是什么？

```
drf是一个基于django开发的组件，本质是一个django的app。 
drf可以办我们快速开发出一个遵循restful规范的程序。
```

3.drf是如何帮助我们快速开发的？drf提供了哪些功能？

```python
- 视图，APIView用处还不知道。 
- 解析器，根据用户请求体格式不同进行数据解析，解析之后放在request.data中。 
	在进行解析时候，drf会读取http请求头 content-type. 
	如果content-type:x-www-urlencoded，那么drf会根据 & 符号分割的形式去处理请求体。 
	user=wang&age=19 
	如果content-type:application/json,那么drf会根据 json 形式去处理请求体。 
	{"user":"wang","age":19} 
- 序列化，可以对QuerySet进行序列化，也可以对用户提交的数据进行校验。 
- 渲染器，可以帮我们把json数据渲染到页面上进行友好的展示。（内部会根据请求设备不同做不同的展示）
```

4.序列化：many=True or False，序列化展示 多条数据，默认是False

5.序列化：战术特殊的数据(choices,forkey,manytomay)可用

```
depth 
source，无需加括号，在源码内部会去判断是否可执行，如果可执行自动加括号。【fk/choice】 SerializerMethodField，定义钩子方法。【m2m】
```

6.写程序的潜规则

```python
# 约束子类中必须实现f1 
class Base(object): 
	def f1(self): 
		raise NotImplementedError('asdfasdfasdfasdf') 
class Foo(Base): 
	def f1(self): 
		print(123) 
obj = Foo() 
obj.f1()
```

7.面向对象的继承

```python
class URLPathVersion(object): 
	def determin_version(self): 
		return 'v1'
    
class APIView(object): 
    version_class = None 
    def dispatch(self,method): 
        version = self.initial() 					print(version) 
        getattr(self,method)() 
    def initial(self): 
        self.process_version() 
        def process_version(): 
            obj = self.version_class() 
            return obj.determine_version() 
        
class UserView(APIView): 
    version_class = URLPathVersion
    def get(self): 
        print('userview.get') 
        
obj = UserView() 
obj.dispatch('get')
#运行的结果：v1,userview.get  总之记住一点：self是谁调用查找的时候就从该类开售查找
```

### 多对多序列化展示

```python
#这两段代码解决了两个问题
class NewArticleSerializer(serializers.ModelSerializer): 
    tag_info = serializers.SerializerMethodField() 
    class Meta: 
        model = models.Article fields = ['title','summary','tag_info'] 
        def get_tag_info(self,obj): 
            #方式一：对于多对多的跨表数据展示，使用钩子函数的方式，我们的obj.tag.all()取得是一个queryset()对象，在这我们就可以使用列表推导式的方式自己构建字典或者直接使用values方法，返回的就是一个字典。
            return [row for row in obj.tag.all().values('id','title')] 
        
class FormNewArticleSerializer(serializers.ModelSerializer): 
    """
    对于我们上展示一部分字段，但是我们在post写入数据的时候发现，有一字段，比如id等写入的时候会报错，因为我们上面序列化器展示里面没有该字段，因此我们的解决办法就是再构造一个序列化器，fields = '__all__'，这样我们在写入的时候就不会报错了（因此，我们也学到了，我们在实现某些功能的时候不一定只能使用一个构造器，我们可能为我们的单独的一个字段构造一个序列器，这个值得注意）
    """
    class Meta: 
        model = models.Article 
        fields = '__all__'
        
        
扩展：
#自定义序列化器获取manytomany外键名称
class TagsSerializer(serializers.Serializer):
	id = serializers.IntegerField()
	title = serializers.CharField()
class ArticleSerializer(serializers.ModelSerializer):
	tags = serializers.SerializerMethodField(read_only=True,required=False)
	class Meta:
		model = models.Article
		fields = "__all__"
	#方式二：这种方式也可展示我们的多对多的字段，这里用的就是构造一个序列器，用这个序列器来序列化我们的queryset对象
	def get_tags(self,obj):
		#obj.tag.all()返回的是一个queryset对象
		return TagsSerializer(obj.tag.all(),many=True).data
```

### 分页

#### PageNumberPagination:第一种分页方式

- 配置 settings.py

  ```python
  REST_FRAMEWORK = {
      'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
      "PAGE_SIZE":2, #每页展示多少条数据
  }
  ```

- 在视图的列表页面

```python
from rest_framework.pagination import PageNumberPagination
from rest_framework import serializers

class PageArticleSerializer(serializers.ModelSerializer):
	class Meta:
		model = models.Article
		fields = "__all__"


class PageArticleView(APIView):
	def get(self, request, *args, **kwargs):
		queryset = models.Article.objects.all()
		# 方式一：仅数据
		"""
		# 分页对象
		page_object = PageNumberPagination()
		# 调用 分页对象.paginate_queryset方法进行分页，得到的结果是分页之后的数据
		# result就是分完页的一部分数据
		result = page_object.paginate_queryset(queryset,request,self)
		# 序列化分页之后的数据
		ser = PageArticleSerializer(instance=result,many=True)
		return Response(ser.data)
		"""
		# 方式二：数据 + 分页信息
		"""
		page_object = PageNumberPagination()
		result = page_object.paginate_queryset(queryset, request, self)
		ser = PageArticleSerializer(instance=result, many=True)
		return page_object.get_paginated_response(ser.data)
		"""
		# 方式三：数据 + 部分分页信息
		page_object = PageNumberPagination()
		result = page_object.paginate_queryset(queryset, request, self)
		ser = PageArticleSerializer(instance=result, many=True)
		return Response({'count': page_object.page.paginator.count, 'result': ser.data})

```

#### LimitOffsetPagination:第二种分页方式

```python
from rest_framework.pagination import PageNumberPagination
from rest_framework.pagination import LimitOffsetPagination
from rest_framework import serializers
class PageArticleSerializer(serializers.ModelSerializer):
	class Meta:
		model = models.Article
		fields = "__all__"
        
class HulaLimitOffsetPagination(LimitOffsetPagination):
	max_limit = 2   #重写LimitOffsetPagination里面的该字段，限定一次取得数据过撑爆我们的内存
    
class PageArticleView(APIView):
	def get(self,request,*args,**kwargs):
		queryset = models.Article.objects.all()
		page_object = HulaLimitOffsetPagination()
		result = page_object.paginate_queryset(queryset, request, self)
		ser = PageArticleSerializer(instance=result, many=True)
		return Response(ser.data)
```

### 扩展

这种方式也可以实现分页

```python
from rest_framework.generics import ListAPIView
class PageViewArticleSerializer(serializers.ModelSerializer):
	class Meta:
		model = models.Article
		fields = "__all__"
class PageViewArticleView(ListAPIView):
	queryset = models.Article.objects.all()
	serializer_class = PageViewArticleSerializer

#settings.py
REST_FRAMEWORK = {
"PAGE_SIZE":2,
"DEFAULT_PAGINATION_CLASS":"rest_framework.pagination.PageNumberPagination"
}
```

### 回忆

```
被关联表反向查询使用xx_set。
```



