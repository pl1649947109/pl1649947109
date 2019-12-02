---
title: python操作office系列及cvs文件
id: 12
date: 2019-10-25 20:30:00
tags: 第三方模块
comment: true
---

### python操作office系列文件

#### Word自动化

```python
import  win32com
import win32com.client
def makeword(name):
    print(name)
    #操作word,
    word=win32com.client.Dispatch("Word.Application")#启动word引擎
    doc=word.Documents.Add() #插入文档
    word.Visible=True #可见
    rng=doc.Range(0,0)#开始位置
    rng.InsertAfter(u"尊敬的%s先生\n"%name)#匹配字符串
    rng.InsertAfter(u"    我是xxx，定于2017.10.1与ccc大婚，诚邀来参加婚礼，先准备好分子钱")
    filename="C:\\Users\\Administrator\\Desktop\\"+name+".doc"
    doc.SaveAs(filename)#保存
doc.Close(True)#关闭
    word.Application.Quit()#退出
names=["李鑫","申羚锐","何丰城","孙雨"]
for name in names:
    makeword(name)
```

#### Excle办公自动化

```python
import  win32com
import win32com.client
def makeexcel(name):
    print(name)
    #操作word,
    ex=win32com.client.Dispatch("Excel.Application")
    wk=ex.Workbooks.Add()#加一张表格
    nowwk=wk.ActiveSheet #当前的焦点表格
    ex.Visible=True#显示
    for i in range(1,10):
        nowwk.Cells(i,i).value=name+str(i)#插入数据Cells方法参数为横坐标和纵坐标
    filename = "C:\\Users\\Administrator\\Desktop\\" + name + ".xls"
    wk.SaveAs(filename) #保存
    wk.Close(True)#关闭
    ex.Application.Quit()#退出
names=["xxx","ccc","vvv","bbb"]
for name in names:
    makeexcel(name)
```

扩展：

##### 用xlrd包读取Excle文件

```python
# 引用包
	import xlrd
# 打开文件
	xlrd.open_workbook(r'/root/excel/chat.xls')
# 获取所有sheet
	sheet_name = workbook.sheet_names()[0]
# 根据sheet索引或者名称获取sheet内容
	sheet = workbook.sheet_by_index(0) # sheet索引从0开始
# 获取指定单元格里面的值
	sheet.cell_value(第几行,第几列)
# 获取某行或者某列的值
    rows = sheet.row_values(1) # 获取第2行内容
    cols = sheet.col_values(2) # 获取第3列内容
# 获取sheet的名称、行数、列数
	print (sheet.name,sheet.nrows,sheet.ncols)
#源码：
import xlrd
from datetime import date,datetime

arrayNum = 6
#array = {'L1':'','L2':'','L3':'','L4':'','Question':'','Answer':''}
tables = []
newTables = []

def read_excel():
    # 打开文件
    workbook = xlrd.open_workbook(r'/root/chat.xls')
    # 获取所有sheet
    sheet_name = workbook.sheet_names()[0]

    # 根据sheet索引或者名称获取sheet内容
    sheet = workbook.sheet_by_index(0) # sheet索引从0开始
    # sheet = workbook.sheet_by_name('Sheet1')

    #print (workboot.sheets()[0])
    # sheet的名称，行数，列数
    print (sheet.name,sheet.nrows,sheet.ncols)

    # 获取整行和整列的值（数组）
    rows = sheet.row_values(1) # 获取第2行内容
    # cols = sheet.col_values(2) # 获取第3列内容
    print (rows)
    # print (cols)

    for rown in range(sheet.nrows):
       array = {'L1':'','L2':'','L3':'','L4':'','Question':'','Answer':''}
       array['L1'] = sheet.cell_value(rown,0)
       array['L2'] = sheet.cell_value(rown,1)
       array['L3'] = sheet.cell_value(rown,2)
       array['L4'] = sheet.cell_value(rown,3)
       array['Question'] = sheet.cell_value(rown,4)
       array['Answer'] = sheet.cell_value(rown,5)
       tables.append(array)

    print (len(tables))
    #print (tables)
    #print (tables[5])
if __name__ == '__main__':
    # 读取Excel
    read_excel();
    print ('读取成功')
```

##### 用openpyxl包写入Excle文件

```python
# 引用包
	import openpyxl
# 创建工作簿
	f = openpyxl.Workbook() #创建工作簿
# 创建sheet
	sheet1 = f.create_sheet()
# 设置每个单元格里面的值
	 for jkey in range(len(newTables)):
    jk = 1
    for cT in range(arrayNum):
      jk = jkey + 1
      if cT == 0:
        sheet1.cell(row=jk,column=cT+1).value='1'
      else:
        sheet1.cell(row=jk,column=cT+1).value='2'
# 保存文件
	f.save("chatPy.xlsx") #保存文件

#源码实例
import openpyxl

#写excel
def write_excel():
    f = openpyxl.Workbook() #创建工作簿

    sheet1 = f.create_sheet()
    #sheet1 = f.add_sheet(u'sheet1',cell_overwrite_ok=True) #创建sheet
    row0 = [u'L1',u'L2',u'L3',u'L4',u'问题',u'答案']

    #生成第一行
    #for i in range(len(row0)):
    #    sheet1.cell(column=i,row=0).value='L1')

    #生成后续

    for jkey in range(len(newTables)): 
       jk = 1
       for cT in range(arrayNum):
         jk = jkey + 1
         if cT == 0:
           sheet1.cell(row=jk,column=cT+1).value='1'
         else:
           sheet1.cell(row=jk,column=cT+1).value='2'
           
    f.save("chatPy.xlsx") #保存文件

if __name__ == '__main__':
    # 写入Excel
    write_excel();
    print ('写入成功')
```

##### 用xlsxwriter包写入Excle文件

```python
#引用包
import xlsxwriter
#创建工作簿
workbook = xlsxwriter.Workbook('demo1.xlsx')#创建一个excel文件
#创建sheet
worksheet = workbook.add_worksheet(u'sheet1')#在文件中创建一个名为TEST的sheet,不加名字默认为sheet1
#设置单元格里面的值
worksheet.write(3,0,35.5)#第4行的第1列设置值为35.5
#关闭工作簿
workbook.close()

#源码：
import xlsxwriter

#写excel
def write_excel(): 
  workbook = xlsxwriter.Workbook('chat.xlsx')#创建一个excel文件
  worksheet = workbook.add_worksheet(u'sheet1')#在文件中创建一个名为TEST的sheet,不加名字默认为sheet1
 
  worksheet.set_column('A:A',20)#设置第一列宽度为20像素
  bold= workbook.add_format({'bold':True})#设置一个加粗的格式对象
 
  worksheet.write('A1','HELLO')#在A1单元格写上HELLO
  worksheet.write('A2','WORLD',bold)#在A2上写上WORLD,并且设置为加粗
  worksheet.write('B2',U'中文测试',bold)#在B2上写上中文加粗
 
  worksheet.write(2,0,32)#使用行列的方式写上数字32,35,5
  worksheet.write(3,0,35.5)#使用行列的时候第一行起始为0,所以2,0代表着第三行的第一列,等价于A4
  worksheet.write(4,0,'=SUM(A3:A4)')#写上excel公式
  workbook.close()

if __name__ == '__main__':
    # 写入Excel
    write_excel();
    print ('写入成功')
```

#### PPT办公自动化

```python
import  win32com
import win32com.client
def makeppt(name):
    try:
        ppt=win32com.client.Dispatch("PowerPoint.Application")
        pres=ppt.Presentations.Add()#增加一个页面
        ppt.Visible=True
        s1=pres.Slides.Add(1,1) #增加一个页面
        s1_0=s1.Shapes[0].TextFrame.TextRange #找到第一个文本
        s1_0.Text="尊敬的%s"%name
        s1_1 = s1.Shapes[1].TextFrame.TextRange  # 找到第2个文本
        s1_1.Text = "hello world"
        filename = "C:\\Users\\ts\\Desktop\\" + name + ".ppt"
        pres.SaveAs(filename)  # 保存
        pres.Close()  # 关闭
        ppt.Application.Quit()  # 退出
    except AttributeError:
        pass

names=["xxx","ccc","vvv","bbb"]
for name in names:
    makeppt(name)
```

### python读写cvs文件

https://blog.csdn.net/guoziqing506/article/details/52014506