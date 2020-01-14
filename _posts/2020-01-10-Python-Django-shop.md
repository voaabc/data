---
layout: post
title: Django商城项目开发 
categories: Python Django
tags: Python Django
---

* content
{:toc}


## 项目架构的程序设计

### 项目使用技术

    基于Python语言，版本：>=3.5及以上。
    使用Django框架，版本：1.11.11的LTS版本。
    MySQL数据库，版本：
    连接数据库：pymysql=0.8.0
    图像处理： Pillow=5.0.0
    Web前端技术：HTML、CSS、JavaScript和Jquery等


### Python&Django虚拟环境

```
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

### Docker&MySQL部署

```sh
yum remove docker docker-client docker-client-latest docker-common   docker-latest docker-latest-logrotate docker-logrotate docker-selinux  docker-engine-selinux docker-engine

yum install -y yum-utils   device-mapper-persistent-data   lvm2

#yum-config-manager     --add-repo     https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum makecache fast

yum list docker-ce --showduplicates | sort -r

#yum install docker-ce-18.03.1.ce

yum install docker-ce

service docker start
systemctl enable docker 
systemctl stop docker
systemctl start docker
docker version
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40


systemctl status docker


(env_1) [root@centostest ~]# mkdir -p /opt/docker-mysql/conf.d
(env_1) [root@centostest ~]# vim /opt/docker-mysql/conf.d/config-file.cnf
[mysqld]
# 表名不区分大小写
lower_case_table_names=1 
#server-id=1
datadir=/var/lib/mysql
#socket=/var/lib/mysql/mysqlx.sock
#symbolic-links=0
# sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid


(env_1) [root@centostest ~]# mkdir -p /opt/docker-mysql/var/lib/mysql

(env_1) [root@centostest ~]# docker run --name mysql \
>     --restart=always \
>     -p 3306:3306 \
>     -v /opt/docker-mysql/conf.d:/etc/mysql/conf.d \
>     -v /opt/docker-mysql/var/lib/mysql:/var/lib/mysql \
>     -e MYSQL_ROOT_PASSWORD=123456-abc \
>     -d mysql:8.0.16
6e3920ff73522bec641c016d5f359d4dce45fd5533b8685e33a061e7daab271f

(env_1) [root@centostest ~]# yum install netstat -y
(env_1) [root@centostest ~]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      9308/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      10719/master        
tcp6       0      0 :::3306                 :::*                    LISTEN      63886/docker-proxy  
tcp6       0      0 :::22                   :::*                    LISTEN      9308/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      10719/master

参考：https://www.jianshu.com/p/d6febf6f95e0

```

```sql
(env_1) [root@centostest ~]# more /tmp/shopdb.sql 
-- 会员信息表（后台管理员信息也在此标准，通过状态区分）
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
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- 商品类别表
CREATE TABLE `type`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL,
  `pid` int(11) unsigned DEFAULT '0',
  `path` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- 商品信息表
CREATE TABLE `goods`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `typeid` int(11) unsigned NOT NULL,
  `goods` varchar(32) NOT NULL,
  `company` varchar(50) DEFAULT NULL,
  `content` text,
  `price` double(6,2) unsigned NOT NULL,
  `picname` varchar(255) DEFAULT NULL,
  `store` int(11) unsigned NOT NULL DEFAULT '0', 
  `num` int(11) unsigned NOT NULL DEFAULT '0',
  `clicknum` int(11) unsigned NOT NULL DEFAULT '0',
  `state` tinyint(1) unsigned NOT NULL DEFAULT '1',
  `addtime` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `typeid` (`typeid`)
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- 订单信息表
CREATE TABLE `orders`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `uid` int(11) unsigned DEFAULT NULL,
  `linkman` varchar(32) DEFAULT NULL,
  `address` varchar(255) DEFAULT NULL,
  `code` char(6) DEFAULT NULL,
  `phone` varchar(16) DEFAULT NULL,
  `addtime` datetime DEFAULT NULL,
  `total` double(8,2) unsigned DEFAULT NULL,
  `state` tinyint(1) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- 订单信息详情表
CREATE TABLE `detail`(
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `orderid` int(11) unsigned DEFAULT NULL,
  `goodsid` int(11) unsigned DEFAULT NULL,
  `name` varchar(32) DEFAULT NULL,
  `price` double(6,2) DEFAULT NULL,
  `num` int(11) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
)ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;


-- 在user上表中添加一条后台管理员账户数据
insert into users values(null,'admin','管理员',md5('admin'),1,'北京市中南海','1000','1888888888','10000@qq.com',0,now())
```


```sh
(env_1) [root@centostest ~]# docker exec -it mysql bash
root@6e3920ff7352:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.0.16 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE shopdb;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| shopdb             |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

```
