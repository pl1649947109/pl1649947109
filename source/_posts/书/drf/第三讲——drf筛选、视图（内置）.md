---
title: 第三讲——drf筛选、视图（内置）
id: 4
date: 2019-11-7 20:00:00
tags: drf
comment: true
---

## 内容回顾

- restful规范

```
- URL中一般用名词： 
	http://www.luffycity.com/article/ (面向资源编程，网络上东西都视为资源)
- 根据请求不同做不同操作：GET/POST/PUT/DELTE/PATCH
- 筛选条件，在URL参数中进行传递：
	http://www.luffycity.com/article/?page=1&category=1
	
一般传输的数据格式都是JSON
```

- 分页的 两种方式
  - Page
  - Offset

## 今日概要

- 作业：呼啦圈
- 筛选
- 试图（源码）

## 今日详细

### 1.作业：呼啦圈

#### 1.1 表结构设计

- 不会经常变化的值放在内存：choices形式，避免跨表性能低（这里的类型的保存就是把数据储存在内存）
- 分表：如果表中列太多/大量内容可以选择水平分表(这里的内容拆分就是水平分表)
- 表自关联
  ![](http://9017499461.linshutu.top/%E8%A1%A8%E8%87%AA%E5%85%B3%E8%81%94.JPG)

<!----more---->

```python
from django.db import models

class UserInfo(models.Model):
	"""
	用户表
	"""
	username = models.CharField(verbose_name="用户名",max_length=32)
	password = models.CharField(verbose_name="密码",max_length=128)

class Article(models.Model):
	"""
	文章详情表
	"""
	status = (
		(1,"发布"),
		(2,"删除"),
	)
	#因为hulaquan这种上面的标签分类一般是不变的，我们可以以这种方式放在我们的内存里面
	#如果还件一张表的话，可以，但是连表查询就很慢的
	category_choices = (
		(1,'咨询'),
		(2, '公司状态'),
		(3, '分享'),
		(4, '答疑'),
		(5, '其他'),
	)
	title = models.CharField(verbose_name="标题",max_length=128)
	create_at = models.DateTimeField(verbose_name="创建时间",auto_now_add=True,)
	read_count = models.IntegerField(verbose_name="浏览数量",default=0)
	comment_count = models.IntegerField(verbose_name="评论数",default=0)
	summary = models.CharField(verbose_name="内容简介",max_length=64)
	photo = models.ImageField(verbose_name="文章图片",upload_to=None,null=True,blank=True)
	article_status = models.IntegerField(verbose_name="文章状态",choices=status,default=1)
	category = models.IntegerField(verbose_name="标签分类",choices=category_choices)
	author = models.ForeignKey(verbose_name="作者",to="UserInfo")

class ArticleDetail(models.Model):
	"""
	文章详情表：这里为什么把文章内容表抽离出来新建一张表？因为内容的信息可能过大，我们在加载全部分
	#文章的时候，其实不需要文章的内容，这样就会降低用户访问的速度，所以在访问详情页的时候返回内容比较好
	"""
	article = models.OneToOneField(verbose_name="文章表",to="Article")
	content = models.TextField(verbose_name="文章内容")

class Comment(models.Model):
	"""
	文章评论
	"""
	content = models.TextField(verbose_name="评论内容")
	content_time = models.DateTimeField(verbose_name="评论时间",auto_now_add=True)
	zan = models.IntegerField(verbose_name="点赞数",default=0)
	article = models.ForeignKey(verbose_name="评论文章",to="Article")
	user = models.ForeignKey(verbose_name="评论用户", to='UserInfo')
	'''这里本来还可以建立一个自关联的字段，这样回复的内容就可以和进行多重回复'''
	#parent = models.ForeignKey(verbose_name='回复', to='self', null=True, blank=True)
```

#### 1.32功能实现

1.2.1内容相关

serializer.py（为了数据更好的输出可以定义更多的serializer）

```python
from rest_framework import serializers

from api import models


class ArticleSerializer(serializers.ModelSerializer):
	comment_count = serializers.SerializerMethodField()

	class Meta:
		model = models.Article
		# fields = "__all__"
		exclude = ['author']

	def get_comment_count(self, obj):
		# xx_set反向查表
		return obj.comment_set.count()


class ArticleDetailSerializer(serializers.ModelSerializer):
	class Meta:
		model = models.ArticleDetail
		# fields = "__all__"
		exclude = ['article']

class CommentSerializer(serializers.ModelSerializer):
	class Meta:
		model = models.Comment
		fields = "__all__"


class PostCommentSerializer(serializers.ModelSerializer):
	class Meta:
		model = models.Comment
		exclude = ['user']


class ArticleListSerializer(serializers.ModelSerializer):
	class Meta:
		model = models.Article
		fields = "__all__"


class PageArticleSerializer(serializers.ModelSerializer):
    #反向查表
	content = serializers.CharField(source="articledetail.content")
	author = serializers.CharField(source="author.username")
    #choice选择
	category = serializers.CharField(source="get_category_display")
    #定义时间钩子，显示固定的格式
	date = serializers.SerializerMethodField()

	class Meta:
		model = models.Article
		fields = "__all__"

	def get_date(self, obj):
		return obj.create_at.strftime('%Y-%m-%d %H:%M')
```

views.py(有几个知识点)

```python
from django.shortcuts import render
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.pagination import PageNumberPagination
from api import models
from api import seriailzer


class ArticleView(APIView):
	"""文章表"""
	def get(self,request,*args,**kwargs):
		"""
		获取文章列表
		:param request:
		:param args:
		:param kwargs:
		:return:
		"""
		pk = kwargs.get('pk')
		if not pk:
			condition = {}
			#query_params == request.GET
			category = request.query_params.get('category')
			if category:
				condition['category'] = category
			#filter(None)获取所有的数据
			queryset= models.Article.objects.filter(**condition).order_by('-create_at')
			pager = PageNumberPagination()
			result = pager.paginate_queryset(queryset,request,self)
			ser = seriailzer.ArticleListSerializer(instance=result,many=True)
			return Response(ser.data)
		article_object = models.Article.objects.filter(pk=pk).first()
		ser = seriailzer.PageArticleSerializer(instance=article_object, many=False)
		return Response(ser.data)

	def post(self, request, *args, **kwargs):
		"""
		新增文章:这里面有两个知识点，一个是文章和内容可以同时实例化多个seializers，并且
				同时保存，第二个是保存的时候可以加一些字段，实现自定制。
		:param request:
		:param args:
		:param kwargs:
		:return:
		"""
		ser = seriailzer.ArticleSerializer(data=request.data)
		ser_detail = seriailzer.ArticleDetailSerializer(data=request.data)
		# 注意一点，我们post数据的时候一定要把两张表的该填的数据填上
		if ser.is_valid() and ser_detail.is_valid():
			# 文章表保存之后返回的是一个对象，它里面包含了ser的所有信息
			article_object = ser.save(author_id=1)
			# ser_detail.save(article.id=article_object.id)
			# 可以使用上面的方式给我们的文章内容表的article字段加内容，也可是使用下面的方式给我们的
			# 文章外键加内容，都是可以的
			ser_detail.save(article=article_object)
			return Response("保存成功")
		return Response("失败")


class CommentView(APIView):
	""" 评论接口 """
	def get(self, request, *args, **kwargs):
		""" 评论列表 """
		article_id = request.query_params.get('article')
		queryset = models.Comment.objects.filter(article_id=article_id)
		ser = seriailzer.CommentSerializer(instance=queryset, many=True)
		return Response(ser.data)

	def post(self, request, *args, **kwargs):
		""" 添加评论 """
		ser = seriailzer.PostCommentSerializer(data=request.data)
		if ser.is_valid():
			ser.save(user_id=2)
			return Response('成功')
		return Response('失败')

```

1.2.2评论列表

- 查看评论列表
  访问时：`http://127.0.0.1:8000/hg/comment/?article=2`

- 添加评论（两种方式：推荐使用第一种）

  ```
  http://127.0.0.1:8000/hg/comment/
  
  {
  	article:1,
  	content:'xxx'
  }
  ```

  ```
  http://127.0.0.1:8000/hg/comment/?article=1
  
  {
  	content:'xxx'
  }
  ```

### 2. 筛选

![](http://9017499461.linshutu.top/%E7%BB%A7%E6%89%BF%E5%9B%BE1.JPG)

![](http://9017499461.linshutu.top/%E7%BB%A7%E6%89%BF%E5%9B%BE2.JPG)

案例：在文章列表时候，添加筛选功能。

```
全部：http://127.0.0.1:8000/hg/article/
筛选：http://127.0.0.1:8000/hg/article/?category=2
```

```python
class ArticleView(APIView):
    """ 文章视图类 """

    def get(self,request,*args,**kwargs):
        """ 获取文章列表 """
        pk = kwargs.get('pk')
        if not pk:
            condition = {}
            category = request.query_params.get('category')
            if category:
                condition['category'] = category
            queryset = models.Article.objects.filter(**condition).order_by('-date')
            
            pager = PageNumberPagination()
            result = pager.paginate_queryset(queryset,request,self)
            ser = ArticleListSerializer(instance=result,many=True)
            return Response(ser.data)
        article_object = models.Article.objects.filter(id=pk).first()
        ser = PageArticleSerializer(instance=article_object,many=False)
        return Response(ser.data)
```

##### drf的组件：内置了筛选的功能

```python
from django.shortcuts import render
from rest_framework.views import APIView
from rest_framework.response import Response
from . import models

from rest_framework.filters import BaseFilterBackend

class MyFilterBackend(BaseFilterBackend):

    def filter_queryset(self, request, queryset, view):
        val = request.query_params.get('cagetory')
        return queryset.filter(category_id=val)
    

class IndexView(APIView):

    def get(self,request,*args,**kwargs):
        # http://www.xx.com/cx/index/
        # models.News.objects.all()

        # http://www.xx.com/cx/index/?category=1
        # models.News.objects.filter(category=1)

        # http://www.xx.com/cx/index/?category=1
        # queryset = models.News.objects.all()
        # obj = MyFilterBackend()
        # result = obj.filter_queryset(request,queryset,self)
        # print(result)
        
        return Response('...')
```

### 3.视图（***）

- APIView，感觉没提供功能。
- GenericAPIView，桥梁，内部定义：get_queryset/get_serilizer/get_page...

```python
ListAPIView（展示所有）,CreateAPIView（创建）,RetrieveAPIView（单条数据展示）,UpdateAPIView（更新）,DestroyAPIView（删除）
```

```python
class TagSer(serializers.ModelSerializer):
    class Meta:
        model = models.Tag
        fields = "__all__"

class TagView(ListAPIView,CreateAPIView):
	"""
	展示所有的数据和添加数据
	"""
    queryset = models.Tag.objects.all()
    serializer_class = TagSer

    def get_serializer_class(self):
        """
        重写父类的方法，定制展示和提交数据时的serializer
        """
        if self.request.method == 'GET':
            return TagSer
        elif self.request.method == 'POST':
            return OtherTagSer
    def perform_create(self,serializer):
        """
        重写父类里面的方式，自定制保存的数据
        """
        serializer.save(author=1)

class TagDetailView(RetrieveAPIView,UpdateAPIView,DestroyAPIView):
    """
    展示单条信息，更新个删除数据：为什么和上面的分开？因为单条的展示和多条数据的展示是有冲突的
    因为，他们里面都定义了get方法，所以我们在使用的时候把他们分开并且可定制不同的serializer
    """
    queryset = models.Tag.objects.all()
    serializer_class = TagSer
```

### 扩展

#### GenericAPIView类视图都为我们提供了内容

##### 支持定义的属性

- 列表视图与详情视图通用：(查询和序列化)
  - **queryset** 列表视图的查询集
  - **serializer_class** 视图使用的序列化器
- 列表视图使用：(分页和过滤)
  - **pagination_class** 分页控制类
  - **filter_backends** 过滤控制后端
- 详情页视图使用：（对单条数据的处理）
  - **lookup_field** 查询单一数据库对象时使用的条件字段，默认为'`pk`'
  - **lookup_url_kwarg** 查询单一数据时URL中的参数关键字名称，默认与**look_field**相同

##### 提供的方法

**get_queryset(self)**

返回视图使用的查询集，是列表视图与详情视图获取数据的基础，默认返回`queryset`属性，**可以重写**

**get_serializer_class(self)**

返回序列化器类，默认返回`serializer_class`，**可以重写**，

**get_serializer(self, args, *kwargs)**

返回序列化器对象，被其他视图或扩展类使用，如果我们在视图中想要获取序列化器对象，可以直接调用此方法。

**get_object(self)** 

返回详情视图所需的模型类数据对象，默认使用`lookup_field`参数来过滤queryset。 在试图中可以调用该方法获取详情信息的模型类对象。

##### 源码

```python
class GenericAPIView(views.APIView):
    #序列化，这两项需要重写
	queryset = None
    serializer_class = None
    
    #对于单条内容的处理默认使用pk，否则重写get_object()
    lookup_field = 'pk'
    lookup_url_kwarg = None
    
    #使用过滤器的后端类
    filter_backends = api_settings.DEFAULT_FILTER_BACKENDS
    
    #使用分页的后端类
    pagination_class = api_settings.DEFAULT_PAGINATION_CLASS
    def get_queryset(self):
        """
        缓存queryset
        """
        queryset = self.queryset
        if isinstance(queryset, QuerySet):
            # Ensure queryset is re-evaluated on each request.
            queryset = queryset.all()
        return queryset

    def get_object(self):
        """
        返回视图显示的对象
        """
        queryset = self.filter_queryset(self.get_queryset())
        lookup_url_kwarg = self.lookup_url_kwarg or self.lookup_field
        filter_kwargs = {self.lookup_field: self.kwargs[lookup_url_kwarg]}
        obj = get_object_or_404(queryset, **filter_kwargs)
        return obj

    def get_serializer(self, *args, **kwargs):
        """
        用于验证和序列化输入输出
        """
        serializer_class = self.get_serializer_class()
        kwargs['context'] = self.get_serializer_context()
        return serializer_class(*args, **kwargs)

    def get_serializer_class(self):
        """
        返回序列化类
        """
        return self.serializer_class

    def get_serializer_context(self):
        """
        提供序列化类额外上下文：不知道是干嘛的
        """
        return {
            'request': self.request,
            'format': self.format_kwarg,
            'view': self
        }

    def filter_queryset(self, queryset):
        """
        对给定的queryset进行过滤筛选
        """
        for backend in list(self.filter_backends):
            queryset = backend().filter_queryset(self.request, queryset, self)
        return queryset

    #下面的都是和分析相关的
    @property
    def paginator(self):

    def paginate_queryset(self, queryset):
        
    def get_paginated_response(self, data):
'PAGE_SIZE':2,
"DEFAULT_PAGINATION_CLASS":"rest_framework.pagination.PageNumberPagination",
    
#过滤类
class MyFilterBackend(BaseFilterBackend):
	"""
	重写过滤类
	"""
    def filter_queryset(self, request, queryset, view):
        val = request.query_params.get('cagetory')
        return queryset.filter(category_id=val)
    
class BaseFilterBackend:
	#必须重写
    def filter_queryset(self, request, queryset, view):
        """
        Return a filtered queryset.
        """
        raise NotImplementedError(".filter_queryset() must be overridden.")

    def get_schema_fields(self, view):
        #定义错误
        assert coreapi is not None, 'coreapi must be installed to use `get_schema_fields()`'
        assert coreschema is not None, 'coreschema must be installed to use `get_schema_fields()`'
        return []
```

#### 五个扩展类

```
ListModelMixin
CreateModelMixin
RetrieveModelMixin
UpdateModelMixin
DestroyModelMixin
```

#### 几个子类视图

```python
### ListAPIView（展示所有数据）
提供 get 方法
继承自：GenericAPIView、ListModelMixin

### CreateAPIView
提供 post 方法
继承自： GenericAPIView、CreateModelMixin

### RetireveAPIView（展示单条数据）
提供 get 方法
继承自: GenericAPIView、RetrieveModelMixin

### DestoryAPIView
提供 delete 方法
继承自：GenericAPIView、DestoryModelMixin

### UpdateAPIView
提供 put 和 patch 方法（put跟新表的全部数据，patch跟新表的部分数据）
继承自：GenericAPIView、UpdateModelMixin

### RetrieveUpdateAPIView
提供 get、put、patch方法
继承自： GenericAPIView、RetrieveModelMixin、UpdateModelMixin

### RetrieveUpdateDestoryAPIView
提供 get、put、patch、delete方法
继承自：GenericAPIView、RetrieveModelMixin、UpdateModelMixin、DestoryModelMixin
```

