---
layout: post
title: Django商城项目开发(二) 
categories: Python Django
tags: Python Django
---

* content
{:toc}



## 基于Django框架的项目搭建


### 摆放后台首页面


```py
 使用事先准备好的后台模板：

    从github上下载一个后台简洁模板：https://github.com/alecfan/mstp_17_akira
    将后台模板目录中的资源目录：css、js、img复制到项目的后台静态资源目录static/myadmin/中
    在templates/myadmin/目录中创建一个基类父模板文件base.html
    在templates/myadmin/目录中创建一个首页模板文件index.html
    在templates/myadmin/目录中创建一个信息提示模板文件info.html
    修改myobject/myadmin/views/index.py视图文件中index函数中代码：

    def index(request):
      '''管理后台首页'''
      return render(request,"myadmin/index.html")

```

> 编辑父类模板：[/templates/myadmin/base.html](view-source:https://www.datadatadata.cn/files/001/myadmin/index.html)

编辑后台首页模板：/templates/myadmin/index.html

后台公共提示信息模板：/templates/myadmin/info.html





