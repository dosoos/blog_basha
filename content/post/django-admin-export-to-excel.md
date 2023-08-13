---
title: "Django Admin管理后台如何导出Excel表格"
description: "Django自带的管理后无法导出Excel表格，第三方的插件导出的话会很繁琐。给大家看一下我的解决办法，后台数据原样导出"
image: "/img/django2excel0.png"
keywords: ""
readingTime: true
categories: ['开发技巧']
tags: ['Django', '管理后台', '数据导出', '导出Excel']
date: 2023-08-12T23:56:52Z
draft: false
---

Django自带的管理后无法导出Excel表格，第三方的插件导出的话会很繁琐，而且导出格式会携带很多数据库冗余字段，使得表格很难看。给大家看一下我的解决办法，100%所见即所得，原样导出后台界面数据为Excel表格

![django管理后台](/img/django2excel1.png "django管理后台")

## 一共分三步
 - 安装Excel依赖
 - 编写admin.ModelAdmin扩展
 - 给指定Admin模型添加导出方法（actions）

---

### 1.安装依赖
```
pip install openpyxl beautifulsoup4
```

### 2.编写admin.ModelAdmin扩展
```
from datetime import datetime
from django.http import HttpResponse
from django.contrib.admin.templatetags import admin_list
from django.contrib.admin.utils import lookup_field
from django.utils.safestring import SafeString
from bs4 import BeautifulSoup
from openpyxl import Workbook

class ExportExcelMixin(object):
    def get_export_ignore(self, request):
        return []

    def export_as_excel(self, request, queryset):
        response = HttpResponse(content_type='application/msexcel')
        download_filename = '{}_{}.xlsx'.format(self.model._meta, datetime.now().strftime('%Y-%m-%dT%H:%M:%S'))
        response['Content-Disposition'] = 'attachment; filename={}'.format(download_filename)
        wb = Workbook()
        ws = wb.active
        cl = self.get_changelist_instance(request)
        headers = admin_list.result_headers(cl)
        ws.append([str(field['text']) for field in headers][1:])
        for obj in queryset:
            results = []
            for display in self.get_list_display(request):
                if display in self.get_export_ignore(request):
                    results.append(None)
                    continue
                result = lookup_field(display, obj, cl.model_admin)[2]
                if type(result) is SafeString:
                    result = BeautifulSoup(result, features="html.parser").text
                results.append(str(result))
            row = ws.append(results)
        wb.save(response)
        return response

    export_as_excel.short_description = '导出Excel'
```

### 3.给指定Admin模型添加导出方法
```
from django.contrib import admin
from order.admin import ExportExcelMixin
from order.models import Order

# 例举订单模型
@admin.register(Order)
class OrderAdmin(admin.ModelAdmin, ExportExcelMixin):
    # other code
    actions = ['export_as_excel']
    # other code
```

---

导出的格式是这样
![django后台数据导出](/img/django2excel2.png "django后台数据导出")

现在订单模型的管理后台即可有导出按钮，选择指定列以后，点击导出Excel即可将页面内容导出为Excel。