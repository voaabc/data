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

```
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

编辑父类模板：/templates/myadmin/base.html

{% load static from staticfiles %}
<!DOCTYPE html>
<html lang="cn">
    <head>
        <meta charset="utf-8">
        <title>网站后台管理</title>
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link href="{% static 'myadmin/css/bootstrap.min.css' %}" rel="stylesheet">
        <link href="{% static 'myadmin/css/bootstrap-responsive.min.css' %}" rel="stylesheet">
        <link href="{% static 'myadmin/css/site.css' %}" rel="stylesheet">
        <!--[if lt IE 9]><script src="{% static 'myadmin/js/html5.js' %}"></script><![endif]-->
    </head>
    <body>
        <div class="container">
            <!-- 页头开始 -->
            <div class="navbar">
                <div class="navbar-inner">
                    <div class="container">
                        <a class="btn btn-navbar" data-toggle="collapse" data-target=".nav-collapse">
                         <span class="icon-bar"></span> <span class="icon-bar"></span> 
                         <span class="icon-bar"></span> </a> 
                         <a class="brand" href="#">网站后台管理</a>
                        <div class="nav-collapse">
                            <ul class="nav">
                                <li class="active">
                                    <a href="index.html">首页</a>
                                </li>
                                <li>
                                    <a href="settings.htm">在线设置</a>
                                </li>
                                <li>
                                    <a href="help.htm">帮助</a>
                                </li>
                                <li class="dropdown">
                                    <a href="help.htm" class="dropdown-toggle" data-toggle="dropdown">更多 
                                    <b class="caret"></b></a>
                                    <ul class="dropdown-menu">
                                        <li>
                                            <a href="help.htm">Introduction Tour</a>
                                        </li>
                                        <li>
                                            <a href="help.htm">Project Organisation</a>
                                        </li>
                                        <li>
                                            <a href="help.htm">Task Assignment</a>
                                        </li>
                                        <li>
                                            <a href="help.htm">Access Permissions</a>
                                        </li>
                                        <li class="divider">
                                        </li>
                                        <li class="nav-header">
                                            Files
                                        </li>
                                        <li>
                                            <a href="help.htm">How to upload multiple files</a>
                                        </li>
                                        <li>
                                            <a href="help.htm">Using file version</a>
                                        </li>
                                    </ul>
                                </li>
                            </ul>
                            <form class="navbar-search pull-left" action="">
                                <input type="text" class="search-query span2" placeholder="Search" />
                            </form>
                            <ul class="nav pull-right">
                                <li>
                                    <a href="profile.htm">@管理员</a>
                                </li>
                                <li>
                                    <a href="login.htm">退出</a>
                                </li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div><!-- 页头结束-->


            <div class="row">
                <div class="span3">
                    <!-- 侧边导航开始 -->
                    <div class="well" style="padding: 8px 0;">
                        <ul class="nav nav-list">
                            <li class="nav-header">
                                导航栏
                            </li>
                            <li class="active">
                                <a href="index.htm"><i class="icon-white icon-home"></i> 首页</a>
                            </li>
                            <li class="nav-header">
                                会员管理
                            </li>
                            <li>
                                <a href="#"><i class="icon-folder-open"></i> 浏览会员</a>
                            </li>
                            <li>
                                <a href="#"><i class="icon-check"></i> 添加会员</a>
                            </li>
                            <li class="nav-header">
                                商品类别管理
                            </li>
                            <li>
                                <a href="messages.htm"><i class="icon-envelope"></i> 浏览商品类别</a>
                            </li>
                            <li>
                                <a href="files.htm"><i class="icon-file"></i> 添加商品类别</a>
                            </li>
                            <li class="nav-header">
                                商品信息管理
                            </li>
                            <li>
                                <a href="activity.htm"><i class="icon-list-alt"></i> 浏览商品信息</a>
                            </li>
                            <li>
                                <a href="activity.htm"><i class="icon-list-alt"></i> 添加商品信息</a>
                            </li>
                            <li class="divider">
                            </li>
                            <li>
                                <a href="help.htm"><i class="icon-info-sign"></i> Help</a>
                            </li>
                            <li class="nav-header">
                                Bonus Templates
                            </li>
                            <li>
                                <a href="gallery.htm"><i class="icon-picture"></i> Gallery</a>
                            </li>
                            <li>
                                <a href="blank.htm"><i class="icon-stop"></i> Blank Slate</a>
                            </li>
                        </ul>
                    </div><!-- 侧边导航结束 -->
                </div>
                <div class="span9">
                    <!-- 主体开始 -->
                    {% block mainbody %}
                    {% endblock %}
                    <!-- 主体结束 -->
                </div>
            </div>
        </div>
        <script src="{% static 'myadmin/js/jquery.min.js' %}"></script>
        <script src="{% static 'myadmin/js/bootstrap.min.js' %}"></script>
        <script src="{% static 'myadmin/js/site.js' %}"></script>
    </body>
</html>

编辑后台首页模板：/templates/myadmin/index.html

{% extends "myadmin/base.html" %}

{% block mainbody %}                
    <h2>
        商城网站后台管理首页
    </h2>
    <div class="hero-unit">
        <h3>
            Welcome!
        </h3>
        <p>
            To get the most out of Akira start with our 3 minute tour.
        </p>
        <p>
            <a href="help.htm" class="btn btn-primary btn-large">Start Tour</a> 
            <a class="btn btn-large">No Thanks</a>
        </p>
    </div>
    <div class="well summary">
        <ul>
            <li>
                <a href="#"><span class="count">3</span> Projects</a>
            </li>
            <li>
                <a href="#"><span class="count">27</span> Tasks</a>
            </li>
            <li>
                <a href="#"><span class="count">7</span> Messages</a>
            </li>
            <li class="last">
                <a href="#"><span class="count">5</span> Files</a>
            </li>
        </ul>
    </div>
{% endblock %}

 后台公共提示信息模板：/templates/myadmin/info.html

{% extends "myadmin/base.html" %}

{% block mainbody %}                
    <h2>
        操作信息提示
    </h2>
    <h4>
        {{ info }}
    </h4>
{% endblock %}


```
```

