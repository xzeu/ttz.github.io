---
title: "Docker部署Redis集群及集群伸缩"
date: 2023-09-26T21:15:34+08:00
lastmod: 2023-09-26T21:15:34+08:00
draft: false
keywords: [redis,docker]
description: ""
tags: [教程笔记,docker,linux,nginx]
categories: [教程笔记]
author: "xzeu"

# Uncomment to pin article to front page
# weight: 1
# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: true
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: '<a rel="license noopener" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0 / 转载文章请保留链接。</a>'
reward: false
mathjax: true

# Uncomment to add to the homepage's dropdown menu; weight = order of article
# menu:
#   main:
#     parent: "docs"
#     weight: 1
---
- [1. Docker部署Redis集群及集群伸缩](#1-docker部署redis集群及集群伸缩)
  - [1.1. 创建6个data目录](#11-创建6个data目录)
  - [1.2. 官网下载最新的redis包](#12-官网下载最新的redis包)
  - [1.3. docker拉取对应的redis镜像](#13-docker拉取对应的redis镜像)
  - [1.4. 修改配置文件](#14-修改配置文件)
  - [1.5. 启动6台redis容器](#15-启动6台redis容器)
  - [1.6. 查看各容器IP](#16-查看各容器ip)
  - [1.7. 进入任一容器配置集群(遇yes敲yes)](#17-进入任一容器配置集群遇yes敲yes)
  - [1.8. 查看集群信息](#18-查看集群信息)
  - [1.9. 集群取存测试](#19-集群取存测试)
  - [1.10. 分片高可用测试(HA)](#110-分片高可用测试ha)
  - [1.11. 思考](#111-思考)
  - [1.12. 集群存储原理](#112-集群存储原理)
  - [1.13. 集群扩容](#113-集群扩容)
  - [1.14. 集群缩容](#114-集群缩容)

<!--more-->

# 1. Docker部署Redis集群及集群伸缩

## 1.1. 创建6个data目录

```bash
mkdir /mnt/data/6381
mkdir /mnt/data/6382
mkdir /mnt/data/6383
mkdir /mnt/data/6384
mkdir /mnt/data/6385
mkdir /mnt/data/6386
```

## 1.2. 官网下载最新的redis包

```bash
wget http://download.redis.io/releases/redis-6.0.6.tar.gz
```

## 1.3. docker拉取对应的redis镜像

```bash
docker pull redis:6.0.6
```

## 1.4. 修改配置文件

```bash
[root@localhost ~]# cp redis-6.0.6/redis.conf /mnt/redis.conf
[root@localhost ~]# vim /mnt/redis.conf
# bind 127.0.0.1 "注释bind 127.0.0.1"
protected-mode no   #"将 protected-mode yes 改为 protected-mode no "
# daemonize yes  "注释daemonize yes, 或改为no"
cluster-enabled yes #"启用集群配置"
cluster-config-file nodes-6381.conf #"节点配置文件"
cluster-node-timeout 5000 # "节点通信超时时间5s"
```

## 1.5. 启动6台redis容器

```bash
docker run -p 6381:6379 --name redis01 -v /mnt/redis.conf:/etc/redis/redis.conf -v /mnt/data/6381:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
docker run -p 6382:6379 --name redis02 -v /mnt/redis.conf:/etc/redis/redis.conf -v /mnt/data/6382:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
docker run -p 6383:6379 --name redis03 -v /mnt/redis.conf:/etc/redis/redis.conf -v /mnt/data/6383:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
docker run -p 6384:6379 --name redis04 -v /mnt/redis.conf:/etc/redis/redis.conf -v /mnt/data/6384:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
docker run -p 6385:6379 --name redis05 -v /mnt/redis.conf:/etc/redis/redis.conf -v /mnt/data/6385:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
docker run -p 6386:6379 --name redis06 -v /mnt/redis.conf:/etc/redis/redis.conf -v /mnt/data/6386:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
```

## 1.6. 查看各容器IP

```bash
[root@localhost ~]# docker inspect redis01 | grep "172.17"
172.172.0.2
[root@localhost ~]# docker inspect redis02 | grep "172.17"
172.172.0.3
[root@localhost ~]# docker inspect redis03 | grep "172.17"
172.172.0.4
[root@localhost ~]# docker inspect redis04 | grep "172.17"
172.172.0.5
[root@localhost ~]# docker inspect redis05 | grep "172.17"
172.172.0.6
[root@localhost ~]# docker inspect redis06 | grep "172.17"
172.172.0.7
```

## 1.7. 进入任一容器配置集群(遇yes敲yes)

```bash
redis-cli --cluster create 172.17.0.2:6379 172.17.0.3:6379 172.17.0.4:6379 172.17.0.5:6379 172.17.0.6:6379 172.17.0.7:6379  --cluster-replicas 1
```

## 1.8. 查看集群信息

```bash
root@d5fb8214a10a:/data# cat nodes-6379.conf 
748eb947ae5ac0c721740870ad44cb356a4b0ca5 172.17.0.4:6379@16379 master - 0 1598101846000 3 connected 10923-16383
8e986c400c6b0d47064ee2fbf7a391574042dbc4 172.17.0.6:6379@16379 slave 092994cf7044995d41a876370af56c97894adea0 0 1598101846000 1 connected
edf1c66d381237ed63a6a6045c84942b7a7f9df9 172.17.0.5:6379@16379 slave 748eb947ae5ac0c721740870ad44cb356a4b0ca5 0 1598101847085 3 connected
092994cf7044995d41a876370af56c97894adea0 172.17.0.2:6379@16379 myself,master - 0 1598101844000 1 connected 0-5460
220fba07c2e345c8aa4578e8bb3b45451377d335 172.17.0.7:6379@16379 slave bb02a3720ff638ea7edb147580e9ffa1dce29726 0 1598101845000 2 connected
bb02a3720ff638ea7edb147580e9ffa1dce29726 172.17.0.3:6379@16379 master - 0 1598101846077 2 connected 5461-10922
vars currentEpoch 6 lastVoteEpoch 0
```

## 1.9. 集群取存测试

```bash
172.17.0.2:6379>cluster nodes      # 查看本机信息
172.17.0.2:6379>cluster info       # 查看集群信息
172.17.0.2:6379>set lan golang     # master插入数据(可能会报错,叫我们去别的实例存)
172.17.0.3:6379>get lan            # node查询数据
(error) MOVED 5798 172.17.0.2:6379 # 告诉我们该数据在172.17.0.2上
172.17.0.5:6386> keys *            # 查键        
1) "lan"
```

> **解决(error) MOVED问题**
>
> 连接时使用-c参数来启动集群模式，命令如下：
> redis-cli -c -p6381

## 1.10. 分片高可用测试(HA)

```bash
docker stop redis03 #停止master3容器
172.17.0.2:6379>cluster nodes       # 查看集群信息
```

![Alt text](../images/20200822222849906.png)

docker start redis03 #恢复节点

![Alt text](../images/20200922222324871.png)

## 1.11. 思考

如果6个容器分别在多台机上，使用172内网端如何解决该问题？

- 解决思路
  - 方法1: 多态物理机配置成docker集群，etcd已确保了ip的唯一性。
  - 方法2: 集群中各个redis实例通过port+10000为集群通信端口，所有可以分别配置6个redis.conf的port端口分别为6381、6382、6383、6384、6385、6386，那么他们的集群通信端口为16382、16382、16383、16384、16385、16386；创建集群的时候使用真机ip+端口即可(如下)。

```bash
redis-cli --cluster create 192.168.1.1:6381 192.168.1.2:6382 192.168.1.3:6383 192.168.1.4:6384 192.168.1.5:6385 192.168.1.6:6386 --cluster-replicas 1
```

## 1.12. 集群存储原理

配置完集群后显示 hash slot的个数，每台机的slot范围=总数/主节点数，用户在任一台主节点上存储key值时，会将key值与CRC16进行相除，再与hash slot求余(%)，余数落在哪个范围就存在哪台机上。

## 1.13. 集群扩容

- 添加主节点

```bash
!创建存储卷
[root@localhost ~]# mkdir /mnt/data/6387
!启动redis实例
[root@localhost ~]# docker run -p 6387:6379 --name redis07 -v /mnt/redis.conf:/etc/redis/redis.conf -v /mnt/data/6387:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
!进入redis01容器
[root@localhost ~]# docker exec -it redis01 /bin/bash
!添加到集群
root@da8d8sew8486a# redis-cli --cluster add-node 172.17.0.8:6379 172.17.0.3:6379
!获取新节点ID
root@da8d8sew8486a# redis-cli --cluster check 172.17.0.8:6379 | grep 0.8:6379$ |awk '{print $2}'
37becf5c7faca5986a313c95bc23f5ab8c547210 
!重新分配哈希槽，一共16384/4=4096(4个主节点)
root@da8d8sew8486a# redis-cli --cluster reshard 172.17.0.3:6379
第一个填4096
第二个填新节点ID
第三个填all
第四个填yes
!检查集群
root@da8d8sew8486a# redis-cli --cluster check 172.17.0.8:6379 
```

- 添加从节点

```bash
!创建存储卷
[root@localhost ~]# mkdir /mnt/data/6388
!启动redis实例
[root@localhost ~]# docker run -p 6388:6379 --name redis08 -v /mnt/redis.conf:/etc/redis/redis.conf -v /mnt/data/6388:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
!进入redis01容器
[root@localhost ~]# docker exec -it redis01 /bin/bash
!添加到集群(两种方式，指定给谁添加[--master-id]或者自动为最少salve的主节点添加)
root@da8d8sew8486a# redis-cli --cluster add-node --cluster-slave 172.17.0.9:6379 172.17.0.3:6379
```

## 1.14. 集群缩容

- 删除从节点

```bash
redis-cli --cluster del-node IP:端口 ID
```

- 删除主节点

```bash
!重新分片
redis-cli --cluster reshard 172.17.0.8:6379
第一个填4096(表示要迁移的哈希槽)
第二个填任一master(表示把哈希槽给谁)
第三个填要移除的master ID
第四个填要移除的master ID2,没有的话填done
第五个填yes(表示确认修改)
!移除节点(跟移除从节点一样)
redis-cli --cluster del-node 172.17.0.8:6379 8c18466f6ab920e459267f878ec71f0f19016015
```

市面上K8s部署redis集群都是通过第三台机来手动配置集群，每次扩缩容都需要人工操作。
