##  django使用xlwt把数据导出到Excel

将数据导出到excel表格中，也算是我们开发中经常用到的功能，如果想把数据导出为csv格式，直接用python的内置csv模块就能实现，这里我们要导出的格式为xls格式，这里就需要我们的xlwt模块了，自己在这里写一个例子做参考，加快以后的开发速度，也分享出来帮助大家解决数据导出的问题。

> **1.安装xlwt模块**

首先要在你的虚拟环境中安装xlwt模块，直接pip安装

```python
pip install xlwt
```

> **2.models.py 生成数据表**

用customer做示范

```python
from django.db import models


class Customer(models.Model):
    """客户模型"""
    name = models.CharField(verbose_name='客户名称', max_length=64, unique=True)
    rank = models.SmallIntegerField(verbose_name='客户级别', choices=constants.CUSTOMER_RANK, default=constants.RANK_NORMAL)
    website = models.CharField(verbose_name='客户网址', max_length=255, blank=True, null=True)
    scale = models.SmallIntegerField(verbose_name='客户规模', choices=constants.CUSTOMER_SCALE, default=constants.SCALE_TEN, null=True, blank=True)
    nature = models.SmallIntegerField(verbose_name='客户性质', choices=constants.CUSTOMER_NATURE, default=constants.NATURE_YX, null=True, blank=True)
    industry = models.SmallIntegerField(verbose_name='客户行业', choices=constants.CUSTOMER_INDUSTRY, default=constants.INDUSTRY_JTSB, null=True, blank=True)
    remarks = models.CharField(verbose_name='客户备注', max_length=255, blank=True, null=True)
    user = models.ForeignKey(User, verbose_name='创建人', related_name='customer', on_delete=models.CASCADE)
    is_valid = models.BooleanField(verbose_name='是否有效', default=True)
    created_at = models.DateTimeField(verbose_name='创建时间', auto_now_add=True)
    updated_at = models.DateTimeField(verbose_name='修改时间', auto_now=True)
```

> **3.views.py 视图方法**

在这里我拿自己写的一个例子来做参考，用xlwt模块导出customer的数据内容，详细怎么用都在代码里用注释标明了

```python
import xlwt
from django.http import HttpResponse
from customer.models import Customer


def export_customer(request):
    """导出所有客户信息"""
    response = HttpResponse(content_type='application/ms-excel')
    response['Content-Disposition'] = 'attachment; filename="customer.xls"'
    # 创建一个workbook，设置编码
    wb = xlwt.Workbook(encoding='utf-8')
    # 创建一个worksheet，对应excel中的sheet，表名称
    ws = wb.add_sheet('customer')
    row_num = 0
    # 初始化样式
    font_style = xlwt.XFStyle()
    # 黑体
    font_style.font.bold = True
    # 写表头，对应自己的model类，将想要导出的字段表头写在这里
    columns = ['客户名称', '级别', '网址', '规模', '性质', '行业', '备注信息', '业务负责人', '创建时间']
    for col_num in range(len(columns)):
        # 参数解读：（行，列，值，样式），row行 col列
        ws.write(row_num, col_num, columns[col_num], font_style)
    # 写表格内容
    font_style = xlwt.XFStyle()
    font_style.font.bold = False
    # 拿到customer的所有数据信息，后面可以进行信息的过滤，条件自己来定，后面values_list的字段内容要喝表头一一相对应
    rows = Customer.objects.filter(is_valid=True).values_list('name', 'rank', 'website', 'scale', 'nature', 'industry', 'remarks', 'user__name', 'created_at')
    # 循环写入
    for row in rows:
        # 下面开始写表格内容
        row_num += 1
        for col_num in range(len(row)):
            ws.write(row_num, col_num, row[col_num], font_style)
    # 保存
    wb.save(response)
    return response
```

> **4.urls.py 配置路由**

```
from django.urls import path, include
from customer import views


urlpatterns = [
    # 导出所有客户信息
    path('export/customer', views.export_customer, name='export_customer'),
]
```

> **5.customer.html 用户页面**

```html
<!--把这个代码放任意地方都可以实现导出功能-->
<a class="export-xls" href="{% url 'export_customer' %}">将所有客户信息导出到Excel</a>
```

到此django导出Excel完成，如有不当之处，欢迎指正，谢谢！！！

最后附一个xlwt模块的详细内容，需要的同学自取：[https://xlwt.readthedocs.io/en/latest/](https://xlwt.readthedocs.io/en/latest/)



***

> *如需转载，请加上本文的链接并标明出处*
>
> *[一条爱吃屎的狗：https://www.qnmlgb.top](https://www.qnmlgb.top/)*
