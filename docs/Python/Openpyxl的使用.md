# Openpyxl 的使用

## 安装库

```ruby
pip install openpyxl
```



## 简单使用

库导入：

```python
import openpyxl
```



## 读取

打开 Excel 文件：

```python
# 得到工作簿
book = openpyxl.load_workbook(filename = 'your-excel-file.xlsx')
```

获取 sheet

```python
sheet = book.active
# 或者
sheet = book['your-sheet-name']
```

获取格子内容：

```python
# 通过行、列的索引获取格子
cell = sheet.cell(row_index, col_index)
# 直接通过序号获取格子（Excel公式中常用的)
cell = sheet['E1']

# 获取格子里的文本
value = sheet['D5'].value
# 获取超链接 （注意： 不是所有的cell都有超链接，需要判空）
url = sheet['D5'].hyperlink.target
```

关闭 Excel 文件：

```python
# 正常模式下，可以不调用close()；但是，当以只读模式打开的工作簿时候，工作簿是懒加载的，必须显式的关闭！
book.close()
```



## 写入

打开或者新建文件：

```python
# 如果是打开文件，参考上面的读取内容👆🏻

# 如果是新建 Excel, 用下面的：
book = openpyxl.Workbook()
sheet = book.create_sheet()
```

向格子写入内容：

```python
# 创建新格子对象
cell = WriteOnlyCell(ws, value="hello world")
# 格子显示的内容
cell.value = '我是内容'
# 字体
cell.font = Font(name='Courier', size=36)
# 批注
cell.comment = Comment(text="A comment", author="Author's Name")
# 时间（未测试)
cell.value = datetime.datetime(2022, 3, 1)
# 公式
sheet['A12'] = '=SUM(A1:A11)' # 不知道直接用 cell.value = '=SUM(A1:A11)'可不可以
# 为 sheet 增加内容： 按行加！
sheet.append([cell, 3.14, None])
```

> 增加内容的时候，append 是按行加格子！

写入图片、视频等资源：

```ruby
# 需要通过 pillow 库，安装它
pip install pillow
```

```python
# todo: 待填充 使用方式
```



最后**必须**要保存：

```python
# file_name: 文件名。如果是打开的 Excel，修改后保存的，这里可以直接填原文件名，会覆盖源文件。要注意源文件的备用！
book.save(file_name)
```



## 附录：

1. [官方 API 文档](https://openpyxl.readthedocs.io/en/stable/formatting.html)
2. 

