---
layout: post
title: Docker搭建MySQL集群(PXC)
categories: Docker MySQL
tags: Docker MySQL
---

* content
{:toc}


### MySQL集群的目的

* 能满足处理高并发请求的高性能要求
* 能处理单节点故障，保证高可用性

### MySQL集群的常见方案
#### Replication

* 速度快，但仅能保证弱一致性，适用于保存价值不高的数据，比如日志、帖子、新闻等。
* 采用master-slave结构，在master写入会同步到slave，能从slave读出；但在slave写入无法同步到master。
* 采用异步复制，master写入成功就向客户端返回成功，但是同步slave可能失败，会造成无法从slave读出的结果。

#### PXC (Percona XtraDB Cluster)

* 速度慢，但能保证强一致性，适用于保存价值较高的数据，比如订单、客户、支付等。
* 数据同步是双向的，在任一节点写入数据，都会同步到其他所有节点，在任何节点上都能同时读写。
* 采用同步复制，向任一节点写入数据，只有所有节点都同步成功后，才会向客户端返回成功。事务在所有节点要么同时提交，要么不提交。

### MySQL集群搭建

在Docker中安装PXC集群，使用Docker仓库中的PXC官方镜像

#### 从docker官方仓库中拉下PXC镜像

```shell
[root@docker01 ~]# docker pull percona/percona-xtradb-cluster
Using default tag: latest
Trying to pull repository docker.io/percona/percona-xtradb-cluster ... 
latest: Pulling from docker.io/percona/percona-xtradb-cluster
ff144d3c0ab1: Pull complete 
eafdff1524b5: Pull complete 
c281665399a2: Pull complete 
c27d896755b2: Pull complete 
c43c51f1cccf: Pull complete 
6eb96f41c54d: Pull complete 
4966940ec632: Pull complete 
2bafadcea292: Pull complete 
3c2c0e21b695: Pull complete 
52a8c2e9228e: Pull complete 
f3f28eb1ce04: Pull complete 
d301ece75f56: Pull complete 
3d24904bec3c: Pull complete 
1053c2982c37: Pull complete 
Digest: sha256:78460483e99c093d2910d3667d928ed8c2165165554482058875bccafa4ccf0b
Status: Downloaded newer image for docker.io/percona/percona-xtradb-cluster:latest
```

#### 重命名镜像

```shell
[root@docker01 ~]# docker tag percona/percona-xtradb-cluster:latest pxc
```

#### 创建网段
出于安全考虑，给PXC集群创建Docker内部网络

```shell
[root@docker01 ~]# docker network create --subnet=172.18.0.0/24 net1
13dd609ce49335a614e6c20a5a7a4b4d36b0d27cdbff5a55540ef10486777a2a
```


#### 创建Docker卷

> 使用Docker时，业务数据应保存在宿主机中，采用目录映射，这样可以使数据与容器独立。但是容器中的PXC无法直接使用映射目录，解决办法是采用Docker卷来映射

```shell
[root@docker01 ~]# docker volume create --name v1
v1
[root@docker01 ~]# docker volume create --name v2
v2
[root@docker01 ~]# docker volume create --name v3
v3
[root@docker01 ~]# docker volume create --name v4
v4
[root@docker01 ~]# docker volume create --name v5
v5
```

#### 创建PXC容器

```shell
# 创建5个PXC容器构成集群 
# 第一个节点 
$ docker run -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -v v1:/var/lib/mysql --name=node1 --network=net1 --ip 172.18.0.2 pxc 
# 在第一个节点启动后要等待一段时间，等候mysql启动完成。 
# 第二个节点 
$ docker run -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v2:/var/lib/mysql --name=node2 --net=net1 --ip 172.18.0.3 pxc 
# 第三个节点 
$ docker run -d -p 3309:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v3:/var/lib/mysql --name=node3 --net=net1 --ip 172.18.0.4 pxc 
# 第四个节点 
$ docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v4:/var/lib/mysql --name=node4 --net=net1 --ip 172.18.0.5 pxc 
# 第五个节点 
$ docker run -d -p 3311:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v5:/var/lib/mysql --name=node5 --net=net1 --ip 172.18.0.6 pxc 
```
> 我按照教程的命令启动PXC容器时一直失败，报如下错误：
> mkdir: cannot create directory ‘’: No such file or directory
> 后来查找了半天，发现命令里面不能带 "–privileged"参数，去掉这个参数后节点启动正常

> 另外命令参数中的密码设置我一开始是"root"，结果会造成PXC容器启动一段时间就自动退出，或者SQL客户端无法连接。
> 后来我把密码改成"abc123456"，就好了。很玄妙，我暂时还不知道为什么，但感觉密码最好字母加数字，不要是"root" 










