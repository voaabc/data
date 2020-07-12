---
layout: post
title: IT技术总结(一)
categories: IT
tags: IT
---

* content
{:toc}

> 近期将陆续更新

## 网络
#### Scientific Internet

#### trojan
>trojan是较新的代理软件，其将通信流量伪装成互联网上最常见的https流量，从而能有效防止流量被检测和干扰。与v2ray相比，trojan功能更专注、从而更加轻量级，性能更好。

部署：略（请科学出国）

Google Cloud：台湾 g1-small（1 个 vCPU，1.7 GB 内存）centos-7

Android: igniter

>实测
![Scientific Internet](https://www.aiops.work/images/Scientific/001.jpg)
![Scientific Internet](https://www.aiops.work/images/Scientific/002.jpg)





#### OpenVPN
    OpenVpn的技术核心是虚拟网卡，其次是SSL协议实现：
    虚拟网卡是使用网络底层编程技术实现的一个驱动软件，安装后在主机上多出现一个网卡，可以像其它网卡一样进行配置。服务程序可以在应用层打开虚拟网卡，如果应用软件（如IE）向虚拟网卡发送数据，则服务程序可以读取到该数据，如果服务程序写合适的数据到虚拟网卡，应用软件也可以接收得到。虚拟网卡在很多的操作系统下都有相应的实现，这也是OpenVpn能够跨平台一个很重要的理由。
    在OpenVpn中，如果用户访问一个远程的虚拟地址（属于虚拟网卡配用的地址系列，区别于真实地址），则操作系统会通过路由机制将数据包（TUN模式）或数据帧（TAP模式）发送到虚拟网卡上，服务程序接收该数据并进行相应的处理后，通过SOCKET从外网上发送出去，远程服务程序通过SOCKET从外网上接收数据，并进行相应的处理后，发送给虚拟网卡，则应用软件可以接收到，完成了一个单向传输的过程，反之亦然。

部署：略（请科学回国）

服务端华为云：上海 2vCPUs 4GB kc1.large.2 CentOS 7.6 64bit 

客户端微软云：美国 Windows 10

>实测
![Scientific Internet](https://www.aiops.work/images/Scientific/004.jpg)
![Scientific Internet](https://www.aiops.work/images/Scientific/005.jpg)



## CI/CD
#### GitLab
#### Ansible
#### Jenkins

## 高可用负载均衡
#### LVS
#### Nginx
[Nginx实现反向代理负载均衡与静态缓存](http://www.aiops.work/2020/06/18/Nginx-Reverse-Proxy-Load-Balance-Cache/)

[Nginx+Keepalived实现双机热备的高可用](http://www.aiops.work/2020/06/19/Nginx-Keepalived-Active-Standby-high-availability/)



## 数据库
#### MySQL

[Docker搭建MySQL集群(PXC)](http://www.aiops.work/2020/07/08/Docker-MySQL-Cluster-PXC/)

#### Redis


## 容器
#### Docker
#### Kubernetes

## 监控
#### Prometheus
#### Zabbix

## 脚本编程
#### Shell
#### Python