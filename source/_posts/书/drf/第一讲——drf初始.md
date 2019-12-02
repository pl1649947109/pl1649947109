---
title: 第一讲——drf初始
id: 2
date: 2019-11-5 20:00:00
tags: drf
comment: true
---

drf：django rest framewok

## 第一部分 问题

- 前后端分离？
  - vue.js
  - 后端给前段返回json数据

- 移动端盛行。
  - app
  - 后端给app返回json数据

- PC端应用？


```
crm项目，前段后端一起写，运行在浏览器上。 一般情况下都是PC端使用。 
```


<!----more---->

## 第二部分 任务

以前针对增删改查使用4条路由，根据restful规范，我们现在使用一条。

以前的我们 ：

```python
http://127.0.0.1:8000/info/get/
http://127.0.0.1:8000/info/add/
http://127.0.0.1:8000/info/update/
http://127.0.0.1:8000/info/delete/
```

现在的我们：要遵循restful规范

```python
http://127.0.0.1:8000/info/
	get,获取数据
	post，添加
	put，更新
	delete，删除
```

基于django可以实现遵循restful规范的接口开发：

- FBV，可以实现比较麻烦。
- CBV，相比较简答根据method做的了不同的区分。

## 第三部分  初识drf

### 3.1 安装

```python
pip3 install djangorestframework
```

### 3.2 使用

- 注册app

  ```python
  INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      'rest_framework'
  ]
  ```

- 写路由

  ```python
  from django.conf.urls import url
  from django.contrib import admin
  from api import views
  
  urlpatterns = [
      url(r'^drf/info/', views.DrfInfoView.as_view()),
  ]
  ```
  
- 写视图

  ```python
  from rest_framework.views import APIView
  from rest_framework.response import Response
  
  class DrfInfoView(APIView):
  
      def get(self,request,*args,**kwargs):
          data = [
              {'id': 1, 'title': '震惊了...王阳居然...', 'content': '...'},
              {'id': 2, 'title': '震惊了...王阳居然...', 'content': '...'},
              {'id': 3, 'title': '震惊了...王阳居然...', 'content': '...'},
              {'id': 4, 'title': '震惊了...王阳居然...', 'content': '...'},
          ]
          return Response(data)
  ```

### DRF的应用场景

```python
以后在公司参与前后端分离项目、参与为app写接口时，用drf会比较方便。
```

## 总结

- restful规范

  ```
  1.给别人提供一个URL，根据URL请求方式的不同，做不同操作。
  	get,获取
  	post,增加
  	put，全部更新（所有的字段必须全部写）
  	patch,局部更新(比如只修改很多字段中的某一条)
  	delete,删除
  2.数据传输基于json格式。
  ```

- drf框架

  ```
  不基于drf也可以实现restful规范来开发接口程序。
  
  使用了drf之后，可以快速帮我们开发restful规范来开发接口。
  ```


## 第四部分 

### 4.1 创建程序并初始化数据库

### 4.2 接口：实现访问接口时，创建一个文章类型

```python
from django.conf.urls import url
from django.contrib import admin
from api import views
urlpatterns = [
    url(r'^drf/category/', views.DrfCategoryView.as_view()),
]
```

```python
from rest_framework.views import APIView
from rest_framework.response import Response
class DrfCategoryView(APIView):
	pass
```

假设：我是前端，你是后端。

开发完毕之后告诉前端：

```python
http://127.0.0.1:8000/drf/category/
```

用工具模拟前端发请求：postman

x-www-urlencoded

```python
request.body: name=alex&age=19&gender=12
request.POST: {'name': ['alex'], 'age': ['19'], 'gender': ['12']}
```

json

```json
request.body: b'{"ID":1,"name":"Alex","age":19}'
request.POST: 没有值
```

#### 参考答案

```python
from django.conf.urls import url
from django.contrib import admin
from api import views
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^info/', views.InfoView.as_view()),
    url(r'^drf/info/', views.DrfInfoView.as_view()),
    url(r'^drf/category/', views.DrfCategoryView.as_view()),
]

```

```python
from api import models
class DrfCategoryView(APIView):

    def post(self,request,*args,**kwargs):
        """增加一条分类信息"""
        models.Category.objects.create(**request.data)
        return Response('成功')

```

### 4.3 接口：获取所有文章类型

```python
from api import models
class DrfCategoryView(APIView):
    def get(self,request,*args,**kwargs):
        """获取所有文章分类"""
        queryset = models.Category.objects.all().values('id','name')
        data_list = list(queryset)
        return Response(data_list)

    def post(self,request,*args,**kwargs):
        """增加一条分类信息"""
        models.Category.objects.create(**request.data)
        return Response('成功')
```

### 4.4 接口：获取一条文章类型的详细信息

```python
from django.conf.urls import url
from django.contrib import admin
from api import views
urlpatterns = [
    url(r'^drf/category/$', views.DrfCategoryView.as_view()),
    url(r'^drf/category/(?P<pk>\d+)/$', views.DrfCategoryView.as_view()),
]

```

```python
from api import models
from django.forms.models import model_to_dict
class DrfCategoryView(APIView):
    def get(self,request,*args,**kwargs):
        """获取所有文章分类/单个文章分类"""
        pk = kwargs.get('pk')
        if not pk:
            queryset = models.Category.objects.all().values('id','name')
            data_list = list(queryset)
            return Response(data_list)
        else:
            category_object = models.Category.objects.filter(id=pk).first()
            data = model_to_dict(category_object)
            return Response(data)

    def post(self,request,*args,**kwargs):
        """增加一条分类信息"""
        models.Category.objects.create(**request.data)
        return Response('成功')

```

### 4.5 接口：文章分类的更新和删除

```python
from django.conf.urls import url
from django.contrib import admin
from api import views
urlpatterns = [
    url(r'^drf/category/$', views.DrfCategoryView.as_view()),
    url(r'^drf/category/(?P<pk>\d+)/$', views.DrfCategoryView.as_view()),
]

```

```python
from api import models
from django.forms.models import model_to_dict
class DrfCategoryView(APIView):
    def get(self,request,*args,**kwargs):
        """获取所有文章分类/单个文章分类"""
        pk = kwargs.get('pk')
        if not pk:
            queryset = models.Category.objects.all().values('id','name')
            data_list = list(queryset)
            return Response(data_list)
        else:
            category_object = models.Category.objects.filter(id=pk).first()
            data = model_to_dict(category_object)
            return Response(data)

    def post(self,request,*args,**kwargs):
        """增加一条分类信息"""
        models.Category.objects.create(**request.data)
        return Response('成功')

    def delete(self,request,*args,**kwargs):
        """删除"""
        pk = kwargs.get('pk')
        models.Category.objects.filter(id=pk).delete()
        return Response('删除成功')

    def put(self,request,*args,**kwargs):
        """更新"""
        pk = kwargs.get('pk')
        models.Category.objects.filter(id=pk).update(**request.data)
        return Response('更新成功')
```

## 第五部分 drf的序列化（***）

drf的 serializers帮助我们提供了

- 数据校验
- 序列化

```python
from django.conf.urls import url
from django.contrib import admin
from api import views
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^info/$', views.InfoView.as_view()),
    url(r'^drf/info/$', views.DrfInfoView.as_view()),
    url(r'^drf/category/$', views.DrfCategoryView.as_view()),
    url(r'^drf/category/(?P<pk>\d+)/$', views.DrfCategoryView.as_view()),


    url(r'^new/category/$', views.NewCategoryView.as_view()),
    url(r'^new/category/(?P<pk>\d+)/$', views.NewCategoryView.as_view()),
]

```

```python
from rest_framework import serializers

class NewCategorySerializer(serializers.ModelSerializer):
    #是不是和我们的ModelForm创建的方法类似
    class Meta:
        model = models.Category
        # fields = "__all__"
        fields = ['id','name']

class NewCategoryView(APIView):
    def get(self,request,*args,**kwargs):
        pk = kwargs.get('pk')
        if not pk:
            queryset = models.Category.objects.all()
            #实例化ser对象
            ser = NewCategorySerializer(instance=queryset,many=True)
            #序列化数据（直接帮我们做好了）
            return Response(ser.data)
        else:
            model_object = models.Category.objects.filter(id=pk).first()
            ser = NewCategorySerializer(instance=model_object, many=False)
            return Response(ser.data)

    def post(self,request,*args,**kwargs):
        ser = NewCategorySerializer(data=request.data)
        #数据校验，和我们的MOdelform的校验方法是一样的
        if ser.is_valid():
            ser.save()
            return Response(ser.data)
        #格式或者空数据，这个将错误带回显示
        return Response(ser.errors)

    def put(self,request,*args,**kwargs):
        pk = kwargs.get('pk')
        category_object = models.Category.objects.filter(id=pk).first()
        #这个是跟新数据时的操作，将我们前端发来的数据也加在里面就和request.POST一样
        ser = NewCategorySerializer(instance=category_object,data=request.data)
        if ser.is_valid():
            ser.save()
            return Response(ser.data)
        return Response(ser.errors)

    def delete(self,request,*args,**kwargs):
        pk = kwargs.get('pk')
        models.Category.objects.filter(id=pk).delete()
        return Response('删除成功')
```

## 总结

1. 什么是前后端分离？

2. drf组件

   ```
   帮助们在django框架基础上快速搭建遵循restful规范接口的程序。
   ```

3. drf组件的功能

   - 解析器，解析请求体中的数据，将其变成我们想要的格式。request.data
   - 序列化，对对象或对象列表（queryset）进行序列化操作以及表单验证的功能。
   - 视图，继承APIView（在内部apiview继承了django的View）

4. postman

   ```
   模拟浏览器进行发送请求
   ```

5. 查找模板的顺序

   ```
   优先根目录下：templates
   根据app的注册顺序去每个app的templates目录中找。
   ```

6. 在URL的最后添加终止符（不然就会被截胡）

## 扩展两个知识点

- 在表中使用外键关联表，我们怎么去显示关联表的字段而不是以id的形式展示？

三种方法（推荐使用后面两种）：

```python
这里面的操作都是在这个serializer里面做的
#方式一
class
ArticleSerializer(serializers.ModelSerializer):
	class Meta:
    	model = models.Article
        fields = "__all__"
        #这个意思就是展示深度，默认展示的深度是0,就是只展示我们的第一张表的信息，它的范围是0-10，最多关联10张表的展示，但是不推荐使用这种方式，因为这种方式展示的不需要的东西太多。
        depth = 1

#方式二
class
ArticleSerializer(serializers.ModelSerializer):
    #这种方式就是使用这个方法里面的source源，用它就可以跨表查询我们需要的字段，后面的required=True这里是我们在写入数据不需要传值，不然就会报错
	    category_txt = serializers.CharField(source='category.name',required=False)
    class Meta:
    	model = models.Article
        fields = ['title'...'category_txt']

#方式三
class
ArticleSerializer(serializers.ModelSerializer):
    x1 = serializers.SerializerMethodField()
    class Meta:
    	model = models.Article
        fields = ['title'...'category_txt']
        
    def get_x1(self,obj):
        #这种方式和上面的一样，这个obj就是我们针对表数据一条条的查，这里的obj就是Article，obj查到category，它就是一个跨表的category对象
        return obj.category.name
```

- 比如本表内的chioce字段，性别选择的时候怎么显示？


其实也是上面的方式，我们还记得之前的ModelForm的针对chioce的数据显示中文的样子吗?是不是使用了get_xx_display()的方式展示的，这里面也是一样的，但是这里用有一个坑，我们下面看：

```python
from rest_framework import serializers
from api import models
class ArticleSerializer(serializers.ModelSerializer):
    #上面的方式二：这里也是使用之前的方式，但是没有加括号：因为drf为我们做了一步，检测到它是一个方法就加括号执行，没有检测到就找对应的属性
    status_txt = serializers.CharField(source='get_status_display',required=False)

    x2 = serializers.SerializerMethodField()
    class Meta:
        model = models.Article
        fields = ['id','title','summary','content','category',,'status_txt',,'x2']

    #上面的方式三：这个就正常了，直接加括号
    def get_x2(self,obj):
        return obj.get_status_display()
```



我们对文章做增删改查：

```python
from django.conf.urls import url
from django.contrib import admin
from api import views
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^info/$', views.InfoView.as_view()),
    url(r'^drf/info/$', views.DrfInfoView.as_view()),
    url(r'^drf/category/$', views.DrfCategoryView.as_view()),
    url(r'^drf/category/(?P<pk>\d+)/$', views.DrfCategoryView.as_view()),


    url(r'^new/category/$', views.NewCategoryView.as_view()),
    url(r'^new/category/(?P<pk>\d+)/$', views.NewCategoryView.as_view()),

    # get获取列表
    # post增加数据
    url(r'^drf/article/$', views.ArticleView.as_view()),
    url(r'^drf/article/(?P<pk>\d+)/$', views.ArticleView.as_view()),
]

```



```python
class ArticleView(APIView):
    def get(self,request,*args,**kwargs):
        pk = kwargs.get('pk')
        if not pk:
            queryset = models.Article.objects.all()
            ser = serializer.ArticleSerializer(instance=queryset,many=True)
            return Response(ser.data)
        article_object = models.Article.objects.filter(id=pk).first()
        ser = serializer.ArticleSerializer(instance=article_object, many=False)
        return Response(ser.data)

    def post(self,request,*args,**kwargs):
        ser = serializer.ArticleSerializer(data=request.data)
        if ser.is_valid():
            ser.save()
            return Response(ser.data)
        return Response(ser.errors)

    def put(self,request,*args,**kwargs):
        """全部更新"""
        pk = kwargs.get('pk')
        article_object = models.Article.objects.filter(id=pk).first()
        ser = serializer.ArticleSerializer(instance=article_object,data=request.data)
        if ser.is_valid():
            ser.save()
            return Response(ser.data)
        return Response(ser.errors)

    def patch(self,request,*args,**kwargs):
        """局部"""
        pk = kwargs.get('pk')
        article_object = models.Article.objects.filter(id=pk).first()
        ser = serializer.ArticleSerializer(instance=article_object, data=request.data,partial=True)
        if ser.is_valid():
            ser.save()
            return Response(ser.data)
        return Response(ser.errors)

    def delete(self,request,*args,**kwargs):
        pk = kwargs.get('pk')
        models.Article.objects.filter(id=pk).delete()
        return Response('删除成功')
```



```python
from rest_framework import serializers
from api import models
class ArticleSerializer(serializers.ModelSerializer):
    category_txt = serializers.CharField(source='category.name',required=False)
    x1 = serializers.SerializerMethodField()

    status_txt = serializers.CharField(source='get_status_display',required=False)

    x2 = serializers.SerializerMethodField()
    class Meta:
        model = models.Article
        # fields = "__all__"
        fields = ['id','title','summary','content','category','category_txt','x1','status','status_txt','x2']
        # depth = 1

    def get_x1(self,obj):
        return obj.category.name

    def get_x2(self,obj):
        return obj.get_status_display()
```

















































