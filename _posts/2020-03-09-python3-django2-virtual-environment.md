---
layout: post
title: Python3.6安装Django2.0虚拟环境
categories: Python Django
tags: Python Django
---

* content
{:toc}





### Python3.6安装

```sh
[root@centostest ~]# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
[root@centostest ~]# yum install ntpdate wget vim tree -y
[root@centostest ~]# ntpdate -u  ntp.aliyun.com
[root@centostest ~]# wget  https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tgz
[root@centostest ~]# tar -xzvf Python-3.6.2.tgz -C /opt/
[root@centostest ~]# cd /opt/Python-3.6.2/
[root@centostest Python-3.6.2]# yum install gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y
[root@centostest Python-3.6.2]# ./configure --prefix=/opt/python36
[root@centostest Python-3.6.2]# make && make install
[root@centostest Python-3.6.2]# vim /etc/profile
export PATH=$PATH:/opt/python362/bin

[root@centostest Python-3.6.2]# python3
-bash: python3: command not found
[root@centostest Python-3.6.2]# source /etc/profile
[root@centostest Python-3.6.2]# python3
Python 3.6.2 (default, Jan 10 2020, 02:08:50) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
[root@centostest Python-3.6.2]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple virtualenv
Collecting virtualenv
  Downloading https://pypi.tuna.tsinghua.edu.cn/packages/05/f1/2e07e8ca50e047b9cc9ad56cf4291f4e041fa73207d000a095fe478abf84/virtualenv-16.7.9-py2.py3-none-any.whl (3.4MB)
    100% |████████████████████████████████| 3.4MB 401kB/s 
Installing collected packages: virtualenv
Successfully installed virtualenv-16.7.9
You are using pip version 9.0.1, however version 19.3.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
[root@centostest Python-3.6.2]# cd  /opt
[root@centostest opt]# virtualenv --no-site-packages --python=python3 env_1

--no-site-packages：表示使用一个只有Python3的环境，而不导入原来Python3中安装模块。
--python=python3：指定要被虚拟的解释器环境。
env_1：表示虚拟的Python环境目录。

Running virtualenv with interpreter /opt/python362/bin/python3
Already using interpreter /opt/python362/bin/python3
Using base prefix '/opt/python362'
New python executable in /opt/env_1/bin/python3
Also creating executable in /opt/env_1/bin/python
Installing setuptools, pip, wheel...
done.
[root@centostest opt]# source env_1/bin/activate
创建好虚拟环境后，需要激活虚拟目录

(env_1) [root@centostest opt]# tree -L 1 env_1/
env_1/
├── bin
├── include
└── lib

3 directories, 0 files
(env_1) [root@centostest opt]# tree env_1/bin/
env_1/bin/
├── activate
├── activate.csh
├── activate.fish
├── activate.ps1
├── activate_this.py
├── activate.xsh
├── easy_install
├── easy_install-3.6
├── pip
├── pip3
├── pip3.6
├── python -> python3
├── python3
├── python3.6 -> python3
├── python-config
└── wheel

0 directories, 16 files

(env_1) [root@centostest opt]# pip3 list
Package    Version
---------- -------
pip        19.3.1 
setuptools 44.0.0 
wheel      0.33.6

(env_1) [root@centostest opt]# python -V
Python 3.6.2

(env_1) [root@centostest opt]# deactivate
[root@centostest ~]# source /opt/env_1/bin/activate
(env_1) [root@centostest ~]# deactivate


[root@centostest ~]# source /opt/env_1/bin/activate
(env_1) [root@centostest ~]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple django==1.11.11
(env_1) [root@centostest ~]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple pymysql==0.8.0
(env_1) [root@centostest ~]# pip3 list
Package    Version
---------- -------
Django     1.11.11
Pillow     5.0.0  
pip        19.3.1 
PyMySQL    0.8.0  
pytz       2019.3 
setuptools 44.0.0 
wheel      0.33.6

```




### Django 2.0虚拟环境
```sh
[root@centostest opt]# virtualenv --no-site-packages --python=python3 env_2
[root@centostest opt]# source /opt/env_2/bin/activate
(env_2) [root@centostest opt]# cd env_2
(env_2) [root@centostest env_2]# ll
total 0
drwxr-xr-x. 2 root root 288 Mar  7 04:28 bin
drwxr-xr-x. 2 root root  24 Mar  7 04:24 include
drwxr-xr-x. 3 root root  23 Mar  7 04:24 lib
(env_2) [root@centostest env_2]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple django==2.0

(env_2) [root@centostest env_2]# pip3 list
Package    Version
---------- -------
Django     2.0    
pip        20.0.2 
pytz       2019.3 
setuptools 45.2.0 
wheel      0.34.2

(env_2) [root@centostest env_2]# python --version
Python 3.6.2
(env_2) [root@centostest env_2]# django-admin startproject view_func_demo
(env_2) [root@centostest env_2]# cd view_func_demo/
(env_2) [root@centostest view_func_demo]# python manage.py startapp book
(env_2) [root@centostest view_func_demo]# vim view_func_demo/settings.py
ALLOWED_HOSTS = ['*']

(env_2) [root@centostest view_func_demo]# nohup python manage.py runserver 192.168.220.131:8000 &
[1] 121058
(env_2) [root@centostest view_func_demo]# nohup: ignoring input and appending output to ‘nohup.out’

(env_2) [root@centostest view_func_demo]# tail -f nohup.out 

```

