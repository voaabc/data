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

MySQL集群的常见方案
Replication

速度快，但仅能保证弱一致性，适用于保存价值不高的数据，比如日志、帖子、新闻等。
采用master-slave结构，在master写入会同步到slave，能从slave读出；但在slave写入无法同步到master。
采用异步复制，master写入成功就向客户端返回成功，但是同步slave可能失败，会造成无法从slave读出的结果。

PXC (Percona XtraDB Cluster)

速度慢，但能保证强一致性，适用于保存价值较高的数据，比如订单、客户、支付等。
数据同步是双向的，在任一节点写入数据，都会同步到其他所有节点，在任何节点上都能同时读写。
采用同步复制，向任一节点写入数据，只有所有节点都同步成功后，才会向客户端返回成功。事务在所有节点要么同时提交，要么不提交。


