---
title: pymysql——和数据库交互的模块
id: 6
date: 2019-8-22 20:30:00
tags: 第三方模块
comment: true
---

Python 数据库的Connection、Cursor两大对象

![](http://9017499461.linshutu.top/%E6%95%B0%E6%8D%AE%E5%BA%931.png)

两者的比喻：

![](http://9017499461.linshutu.top/%E6%95%B0%E6%8D%AE%E5%BA%932.png)

<!-----more----->

### **Connection()的参数列表**

- host，连接的数据库服务器主机名，默认为本地主机(localhost)。
- user，连接数据库的用户名，默认为当前用户。
- passwd，连接密码，没有默认值。
- db，连接的数据库名，没有默认值。
- conv，将文字映射到Python类型的字典。 
- MySQLdb.converters.conversions
- cursorclass，cursor()使用的种类，默认值为MySQLdb.cursors.Cursor。
- compress，启用协议压缩功能。
- named_pipe，在windows中，与一个命名管道相连接。
- init_command，一旦连接建立，就为数据库服务器指定一条语句来运行。
- read_default_file，使用指定的MySQL配置文件。
- read_default_group，读取的默认组。
- unix_socket，在unix中，连接使用的套接字，默认使用TCP。
- port，指定数据库服务器的连接端口，默认是3306。

**方法**

- 连接对象的db.close()方法可关闭数据库连接，并释放相关资源。
- 连接对象的db.cursor([cursorClass])方法返回一个指针对象，用于访问和操作数据库中的数据。
- 连接对象的db.begin()方法用于开始一个事务，如果数据库的AUTOCOMMIT已经开启就关闭它，直到事务调用commit()和rollback()结束。
- 连接对象的db.commit()和db.rollback()方法分别表示事务提交和回退。
- 指针对象的cursor.close()方法关闭指针并释放相关资源。
- 指针对象的cursor.execute(query[,parameters])方法执行数据库查询。
- 指针对象的cursor.fetchall()可取出指针结果集中的所有行，返回的结果集一个元组(tuples)。
- 指针对象的cursor.fetchmany([size=cursor.arraysize])从查询结果集中取出多行，我们可利用可选的参数指定取出的行数。
- 指针对象的cursor.fetchone()从查询结果集中返回下一行。
- 指针对象的cursor.arraysize属性指定由cursor.fetchmany()方法返回行的数目，影响fetchall()的性能，默认值为1。
- 指针对象的cursor.rowcount属性指出上次查询或更新所发生行数。-1表示还没开始查询或没有查询到数据。

### **Cursor**

- close():关闭此游标对象。
- fetchone():得到结果集的下一行。
- fetchmany([size = cursor.arraysize]):得到结果集的下几行。
- fetchall():得到结果集中剩下的所有行。
- excute(sql[, args]):执行一个数据库查询或命令。
- excutemany(sql, args):执行多个数据库查询或命令。
- Lastrowid:获取最新的自增ID。

![](http://9017499461.linshutu.top/%E6%95%B0%E6%8D%AE%E5%BA%933.png)

![](http://9017499461.linshutu.top/%E6%95%B0%E6%8D%AE%E5%BA%934.png)

![](http://9017499461.linshutu.top/%E6%95%B0%E6%8D%AE%E5%BA%935.png)

![](http://9017499461.linshutu.top/%E6%95%B0%E6%8D%AE%E5%BA%936.png)

**补充：**

游标移动：

Cursor.scroll(1,mode=”relative”)#相对当前的位置移动

Cursor.scroll(2,mode=”absolute”)#相对绝对的位置移动

**实例**

解决SQL注入的问题

```python
简化连接过程：
import pymysql
import contextlib
#定义上下文管理器，连接后自动关闭连接
@contextlib.contextmanager
def mysql(host='127.0.0.1', port=3306, user='root', passwd='', db='tkq1',charset='utf8'):
    conn = pymysql.connect(host=host, port=port, user=user, passwd=passwd, db=db, charset=charset)
  cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
  try:
    yield cursor
  finally:
    conn.commit()
    cursor.close()
    conn.close()
 
# 执行sql
with mysql() as cursor:
  print(cursor)
  row_count = cursor.execute("select * from tb7")
  row_1 = cursor.fetchone()
  print row_count, row_1
```

数据库连接池：https://chpl.top/2019/11/20/%E4%B9%A6/flask/%E7%AC%AC%E4%BA%8C%E8%AE%B2%E2%80%94%E2%80%94%E9%9D%A2%E8%AF%95%E9%A2%98%E3%80%81%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E6%B1%A0%E3%80%81%E9%85%8D%E7%BD%AE%E3%80%81%E8%B7%AF%E7%94%B1%E3%80%81%E4%B8%AD%E9%97%B4%E4%BB%B6/