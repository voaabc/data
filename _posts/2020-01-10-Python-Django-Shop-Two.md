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
'''
 使用事先准备好的后台模板：

    从github上下载一个后台简洁模板：https://github.com/alecfan/mstp_17_akira
    将后台模板目录中的资源目录：css、js、img复制到项目的后台静态资源目录static/myadmin/中
    在templates/myadmin/目录中创建一个基类父模板文件base.html
    在templates/myadmin/目录中创建一个首页模板文件index.html
    在templates/myadmin/目录中创建一个信息提示模板文件info.html
    修改myobject/myadmin/views/index.py视图文件中index函数中代码：
'''
    def index(request):
      '''管理后台首页'''
      return render(request,"myadmin/index.html")

```

> 编辑父类模板：[/templates/myadmin/base.html](view-source:https://www.datadatadata.cn/files/001/myadmin/base.html)


> 编辑后台首页模板：[/templates/myadmin/index.html](view-source:https://www.datadatadata.cn/files/001/myadmin/index.html)

> 后台公共提示信息模板：[/templates/myadmin/info.html](view-source:https://www.datadatadata.cn/files/001/myadmin/info.html)


## 项目实战后台之会员管理

### 用户信息数据表：users

> 在数据库 shopdb 中创建users表，若此表已存在请跳过


```sql
CREATE TABLE `users`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(32) NOT NULL,
  `name` varchar(16) DEFAULT NULL,
  `password` char(32) NOT NULL,
  `sex` tinyint(1) unsigned NOT NULL DEFAULT '1',
  `address` varchar(255) DEFAULT NULL,
  `code` char(6) DEFAULT NULL,
  `phone` varchar(16) DEFAULT NULL,
  `email` varchar(50) DEFAULT NULL,
  `state` tinyint(1) unsigned NOT NULL DEFAULT '1',
  `addtime` datetime DEFAULT NULL, 
  PRIMARY KEY (`id`),      
  UNIQUE KEY `username` (`username`)
)ENGINE=MyISAM AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

```

### 定义模型Model类

> 进入common应用目录中编辑：myobject/common/models.py 模型文件


```py
from django.db import models
from datetime import datetime

#用户信息模型
class Users(models.Model):
    username = models.CharField(max_length=32)
    name = models.CharField(max_length=16)
    password = models.CharField(max_length=32)
    sex = models.IntegerField(default=1)
    address = models.CharField(max_length=255)
    code = models.CharField(max_length=6)
    phone = models.CharField(max_length=16)
    email = models.CharField(max_length=50)
    state = models.IntegerField(default=1)
    addtime = models.DateTimeField(default=datetime.now)

    def toDict(self):
        return {'id':self.id,'username':self.username,'name':self.name,'password':self.password,'address':self.address,'phone':self.phone,'email':self.email,'state':self.state,'addtime':self.addtime}


    class Meta:
        db_table = "users"  # 更改表名

```


### 项目urls路由信息配置

> 打开根路由文件：myobject/myadmin/urls.py路由文件，编辑路由配置信息

```py
from django.conf.urls import url

from myadmin.views import index,users

urlpatterns = [
    # 后台首页
    url(r'^$', index.index, name="myadmin_index"),

    # 后台用户管理
    url(r'^users$', users.index, name="myadmin_users_index"),
    url(r'^users/add$', users.add, name="myadmin_users_add"),
    url(r'^users/insert$', users.insert, name="myadmin_users_insert"),
    url(r'^users/del/(?P<uid>[0-9]+)$', users.delete, name="myadmin_users_del"),
    url(r'^users/edit/(?P<uid>[0-9]+)$', users.edit, name="myadmin_users_edit"),
    url(r'^users/update/(?P<uid>[0-9]+)$', users.update, name="myadmin_users_update"),
]
```


### 编辑视图文件

> 创建：myobject/myadmin/views/users.py 视图文件，并进行编辑

```py
from django.shortcuts import render
from django.http import HttpResponse

from common.models import Users
from datetime import datetime

# 浏览会员
def index(request):
    # 执行数据查询，并放置到模板中
    list = Users.objects.all()
    context = {"userslist":list}
    #return HttpResponse(list)
    return render(request,'myadmin/users/index.html',context) 

# 会员信息添加表单
def add(request):
    return render(request,'myadmin/users/add.html')

#执行会员信息添加    
def insert(request):
    try:
        ob = Users()
        ob.username = request.POST['username']
        ob.name = request.POST['name']
        #获取密码并md5
        import hashlib
        m = hashlib.md5() 
        m.update(bytes(request.POST['password'],encoding="utf8"))
        ob.password = m.hexdigest()
        ob.sex = request.POST['sex']
        ob.address = request.POST['address']
        ob.code = request.POST['code']
        ob.phone = request.POST['phone']
        ob.email = request.POST['email']
        ob.state = 1
        ob.addtime = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        ob.save()
        context = {'info':'添加成功！'}
    except Exception as err:
        print(err)
        context = {'info':'添加失败！'}

    return render(request,"myadmin/info.html",context)

# 执行会员信息删除
def delete(request,uid):
    try:
        ob = Users.objects.get(id=uid)
        ob.delete()
        context = {'info':'删除成功！'}
    except:
        context = {'info':'删除失败！'}
    return render(request,"myadmin/info.html",context)

# 打开会员信息编辑表单
def edit(request,uid):
    try:
        ob = Users.objects.get(id=uid)
        context = {'user':ob}
        return render(request,"myadmin/users/edit.html",context)
    except Exception as err:
        print(err)
        context = {'info':'没有找到要修改的信息！'}
    return render(request,"myadmin/info.html",context)

# 执行会员信息编辑
def update(request,uid):
    try:
        ob = Users.objects.get(id=uid)
        ob.name = request.POST['name']
        ob.sex = request.POST['sex']
        ob.address = request.POST['address']
        ob.code = request.POST['code']
        ob.phone = request.POST['phone']
        ob.email = request.POST['email']
        ob.state = request.POST['state']
        ob.save()
        context = {'info':'修改成功！'}
    except Exception as err:
        print(err)
        context = {'info':'修改失败！'}
    return render(request,"myadmin/info.html",context)
```


### 编写模板文件

```c
后台用户信息浏览模板：/templates/myadmin/users/index.html

{% extends "myadmin/base.html" %}

{% block mainbody %}                
    <h4>
        会员信息管理
    </h4>
    <table class="table table-bordered table-striped">
        <thead>
            <tr>
                <th>账号</th>
                <th>真实姓名</th>
                <th>性别</th>
                <th>邮箱</th>
                <th>注册时间</th>
                <th>状态</th>
                <th>操作</th>
            </tr>
        </thead>
        <tbody>
            {% for vo in userslist %}
            <tr>
                <td>{{ vo.username }}</td>
                <td>{{ vo.name }}</td>
                <td>{% if vo.sex == 1 %}男{% else %}女{% endif %}</td>
                <td>{{ vo.email }}</td>
                <td>{{ vo.addtime|date:'Y-m-d H:i:s' }}</td>
                <td>{{ vo.state }}</td>
                <td>
                    <a href="{% url 'myadmin_users_del' vo.id %}" class="view-link">删除</a>
                    <a href="{% url 'myadmin_users_edit' vo.id %}" class="view-link">编辑</a>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>                
    <div class="pagination">
        <ul>
            <li class="disabled">
                <a href="#">&laquo;</a>
            </li>
            <li class="active">
                <a href="#">1</a>
            </li>
            <li>
                <a href="#">2</a>
            </li>
            <li>
                <a href="#">3</a>
            </li>
            <li>
                <a href="#">4</a>
            </li>
            <li>
                <a href="#">&raquo;</a>
            </li>
        </ul>
    </div>
{% endblock %}



后台用户信息添加模板：/templates/myadmin/users/add.html

{% extends "myadmin/base.html" %}

{% block mainbody %}                
    <h3>
        会员信息管理
    </h3>
    <form id="edit-profile" action="{% url 'myadmin_users_insert' %}" class="form-horizontal" method="post">
        {% csrf_token %}
        <fieldset>
            <legend>添加会员信息</legend>
            <div class="control-group">
                <label class="control-label" for="input01">账号：</label>
                <div class="controls">
                    <input type="text" name="username" class="input-xlarge" id="input01" value="" />
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">密码：</label>
                <div class="controls">
                    <input type="password" name="password" class="input-xlarge" id="input01"/>
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">重复密码：</label>
                <div class="controls">
                    <input type="password" name="repassword" class="input-xlarge" id="input01"/>
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">真实姓名：</label>
                <div class="controls">
                    <input type="text" name="name" class="input-xlarge" id="input01"/>
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">性别：</label>
                <div class="controls">
                    <input type="radio" name="sex" class="input-xlarge" id="input01" value="1" /> 男
                    <input type="radio" name="sex" class="input-xlarge" id="input01" value="0" /> 女
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">地址：</label>
                <div class="controls">
                    <input type="text" name="address" class="input-xlarge" id="input01"/>
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">邮编：</label>
                <div class="controls">
                    <input type="text" name="code" class="input-xlarge" id="input01"/>
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">电话：</label>
                <div class="controls">
                    <input type="text" name="phone" class="input-xlarge" id="input01"/>
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">email邮箱：</label>
                <div class="controls">
                    <input type="text" name="email" class="input-xlarge" id="input01"/>
                </div>
            </div>                        
            <div class="form-actions">
                <button type="submit" class="btn btn-primary">添加</button> <button type="reset" class="btn">重置</button>
            </div>
        </fieldset>
    </form>
{% endblock %}


后台用户信息编辑模板：/templates/myadmin/users/edit.html

{% extends "myadmin/base.html" %}

{% block mainbody %}                
    <h3>
        会员信息管理
    </h3>
    <form id="edit-profile" action="{% url 'myadmin_users_update' user.id %}" class="form-horizontal" method="post">
        {% csrf_token %}
        <fieldset>
            <legend>编辑会员信息</legend>
            <div class="control-group">
                <label class="control-label" for="input01">账号：</label>
                <div class="controls">
                    {{ user.username }}
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">真实姓名：</label>
                <div class="controls">
                    <input type="text" name="name" class="input-xlarge" id="input01" value="{{ user.name }}" />
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">性别：</label>
                <div class="controls">
                    <input type="radio" name="sex" class="input-xlarge" id="input01" value="1" 
                    {% if user.sex == 1 %}checked{% endif %} /> 男
                    <input type="radio" name="sex" class="input-xlarge" id="input01" value="0"
                    {% if user.sex == 0 %}checked{% endif %} /> 女
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">地址：</label>
                <div class="controls">
                    <input type="text" name="address" class="input-xlarge" id="input01" value="{{ user.address }}" />
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">邮编：</label>
                <div class="controls">
                    <input type="text" name="code" class="input-xlarge" id="input01" value="{{ user.code }}" />
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">电话：</label>
                <div class="controls">
                    <input type="text" name="phone" class="input-xlarge" id="input01" value="{{ user.phone }}" />
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">email邮箱：</label>
                <div class="controls">
                    <input type="text" name="email" class="input-xlarge" id="input01" value="{{ user.email }}" />
                </div>
            </div>
            <div class="control-group">
                <label class="control-label" for="input01">状态：</label>
                <div class="controls">
                    <input type="radio" name="state" class="input-xlarge" id="input01" value="0" 
                    {% if user.state == 0 %}checked{% endif %} /> 管理员 
                    <input type="radio" name="state" class="input-xlarge" id="input01" value="1" 
                    {% if user.state == 1 %}checked{% endif %} /> 启用会员
                    <input type="radio" name="state" class="input-xlarge" id="input01" value="2" 
                    {% if user.state == 2 %}checked{% endif %} /> 禁用用户
                </div>
            </div>                        
            <div class="form-actions">
                <button type="submit" class="btn btn-primary">保存</button> <button type="reset" class="btn">重置</button>
            </div>
        </fieldset>
    </form>
{% endblock %}

```
