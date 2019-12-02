---
title: 第三讲——cmdb资产管理之资产入库和变更、未采集的资产
id: 3
date: 2019-11-15 20:00:00
tags: cmdb
comment: true
---

## 今日概要

- 资产入库
- 资产变更记录
- 今日未采集的资产

<!----more---->

## 今日详细

### 1. 资产入库 & 2.资产变更记录

- 写入数据库

  ```python
      def post(self,request,*args,**kwargs):
          # 1. 获取到用户提交资产信息
          # 2. 保存到数据库（表关系）
          hostname = request.data.get('hostname')
          server_object = models.Server.objects.filter(hostname=hostname).first()
          if not server_object:
              return Response('主机不存在')
  
          disk_info = request.data['info']['disk']
          """
          if not disk_info['status']:
              print(disk_info['error'])
          else:
              for slot,row_dict in disk_info['data'].items():
                  # models.Disk.objects.create(**row_dict,server=server_object)
                  models.Disk.objects.create(**row_dict,server_id=server_object.id)
          """
          return Response('发送成功')
  
  ```


### 总结

```
中控机汇报到api的资产需要做入库以及变更记录的处理。
 - 由于资产自己时是利用工厂模式实现可扩展插件，方便与扩展。在api端也是使用相同模式，对插件进行一一处理。 
 - 在处理资产信息时候，对操作进行交集和差集的处理从而得到删除/更新/新增资产。
 - 在内部通过反射进行资产变更记录的获取，最终将资产以及变更记录写入数据库。 
```

### 3.今日未采集服务器

```
基于Q实现复杂的SQL查询
```

# 本周内容总结

## 第一部分 drf

```
1. restful规范

2. jwt
   pip3 install pyjwt
   pip3 install djangorestframework-jwt
   

3. drf基本使用
- 路由
- 视图
- 序列化
- 解析器
- 筛选器
- 渲染器

4. 源码（要求：流程的分析）
- 版本
- 认证
- 权限
- 节流

5. 继承过的视图类

6. GenericAPIView的作用

7. 全部和局部应用

8. 相关配置

9. 呼啦圈
```
## 第二部分 CMDB

```
1. cmdb背景
2. cmdb的实现
3. 中控机
   1. paramiko 
   2. pymysql操作数据库
   3. 单例模式（其它的单例模式）
   4. 工厂模式
   5. 日志
   6. 堆栈信息
   7. 对象进行数据封装 BaseReponse 
   8. 线程池
   9. requests模块：data/json 
   10. 采集资产的命令：dmidecode / megacli / saltstack 
4. Q获取未采集资产
5. 集合交并差
6. 反射
7. orm批量增加数据：bulk_create([], 10)1. cmdb背景
```

















































































