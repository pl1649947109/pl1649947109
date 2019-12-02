---
title: docx——word文档模块
id: 3
date: 2019-8-22 20:30:00
tags: 第三方模块
comment: true
---

### 导入模块

```python
from docx import Document
from docx.shared import Pt
from docx.oxml.ns import qn
from docx.shared import Inches
```

### 操作流程

- 打开文档

  doc = Cocument

- 操作文档

  具体见详述

- 保存文档

  doc.save("保存路径")

<!-----more----->

### 具体内容操作

- 添加不同等级的标签(就是他们现在的这种格式)

  ## **Doc.add_heading(‘标签的内容’,0)**

  ### **Doc.add_heading(**‘标签的内容’,1)

  #### **Doc.add_heading(**‘标签的内容’2)

- 添加文本

  doc.add_paragraph(‘添加的文本内容’)

- 设置字号

- 设置字体

- 设置斜体

- 设置粗体

  **https://www.cnblogs.com/gaosai/p/9825233.html**

- 增加引用

  doc.add_paragraph(‘Intense quote’,style=’Intense Quote’)

- 增加有序列表

  doc.add_paragraph(‘有序列表的内容’,style=’List Number’)

- 增加无序列表

  doc.add_paragraph(‘无序列表的内容’,style=’List Bullet’)

- 增加图片

  doc.add_picture(‘添加的图片’,参数)

- 添加表格

  Table = doc.add_table(3,3)

  ```python
  Table = doc.add_table(3,3)
  Hdr_cells = table.rows[0].cells  #第一行
  Hdr_cells[0].text=’第一行第一列’
  Hdr_cells[1].text=’第一行第二列’
  Hdr_cells[2].text=’第一行第三列’
  
  Hdr_cells = table.rows[1].cells   #第二行
  Hdr_cells[0].text=’第二行第一列’
  Hdr_cells[1].text=’第二行第二列’
  Hdr_cells[2].text=’第二行第三列’
  
  Hdr_cells = table.rows[1].cells   #第三行
  Hdr_cells[0].text=’第三行第一列’
  Hdr_cells[1].text=’第三行第二列’
  Hdr_cells[2].text=’第三行第三列’
  ```

- 增加分页

  doc.add_page_break()

