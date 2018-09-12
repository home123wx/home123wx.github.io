---
title: 使用Python实现Excel文件的读写
tags: [python,excel]
date: 2017-12-23
categories: Python
---

#### 使用xlwt和xlrd来操作excel
* `xlwt`库用于写excel文件
* `xlrd`库用于读excel文件

#### Demo代码

``` python
# coding=utf8

import xlwt
import xlrd


class ExcelWrite:
    def __init__(self):
        # 创建workbook和sheet对象
        self.workbook = xlwt.Workbook()

        # 构建一个sheet页  (cell_overwrite_ok: true->单元格内容被覆盖)
        self.sheet = self.workbook.add_sheet(u"Sheet名字", cell_overwrite_ok=True)

        # 写入自定义的标题内容
        self._write_title()

    # 保存文件
    def save(self, file_name):
        self.workbook.save(file_name)

    # 写入内容
    def write_text(self, row_data_list):
        """
        写入数据
        :param row_data_list: 准备写入excel文件的数据(定义了一个二维数组结构)
        :return:
        """
        # 从第2行开始, 第一行为标题行
        for i in range(0, len(row_data_list)):
            self._write_row_text(i+1, row_data_list[i])

    # 写入行内容
    def _write_row_text(self, row_index, col_data_list):
        """
        写数据到指定行中
        :param list_data: 对应行中每一列的数据
        :return:
        """
        for i in range(0, len(col_data_list)):
            self.sheet.write(row_index, i, col_data_list[i])

    # 写入标题
    def _write_title(self):
        title_list = [u"列1", u"列2", u"列3"]
        for i in range(0, len(title_list)):
            self.sheet.write(0, i, title_list[i])


class ExcelRead:
    def __init__(self):
        pass

    def read_excel(self, file_name):
        # 打开excel文件
        data = xlrd.open_workbook(file_name)

        # 获取sheet标签
        table = data.sheet_by_name("Sheet名字")

        # sheet标签中的行数
        rows = table.nrows
        # sheet标签中的列数
        cols = table.ncols

        # 获取每行的数据
        for i in range(0, rows):
            print(table.row_values(i))

        # 获取每列的数据
        for x in range(0, cols):
            print(table.col_values(x))
        #
        # # 逐个获取单元格数据
        for i in range(0, rows):
            for j in range(cols):
                print(table.cell(i, j).value)


if __name__ == '__main__':
    eo = ExcelWrite()

    col_list = ["aaa", "bbb", "ccc"]
    col_list1 = ["aaa111", "bbb111", "ccc111"]
    col_list2 = ["aaa222", "bbb222", "ccc222"]

    data_list = list()
    data_list.append(col_list)
    data_list.append(col_list1)
    data_list.append(col_list2)

    # 写入数据
    eo.write_text(data_list)
    # 保存文件
    eo.save("111.xls")

    # 读取文件
    er = ExcelRead()
    er.read_excel("111.xls")


```
