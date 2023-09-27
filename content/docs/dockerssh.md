---
title: "Docker基于Alpine构建SSH服务"
date: 2023-04-11T15:54:40+08:00
lastmod: 2023-04-11T15:54:40+08:00
draft: false
keywords: []
description: ""
tags: [docker]
categories: []
author: "xzeu"

# Uncomment to pin article to front page
# weight: 1
# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
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
- [1. SSH （Secure Shell） 简介](#1-ssh-secure-shell-简介)
- [2. SSH服务适用场景](#2-ssh服务适用场景)
	- [2.1. sshd 服务启动方式](#21-sshd-服务启动方式)
- [3. Docker构建SSH服务镜像](#3-docker构建ssh服务镜像)
	- [3.1. SSH服务镜像说明](#31-ssh服务镜像说明)
	- [3.2. 创建Dockerfile目录](#32-创建dockerfile目录)
	- [3.3. 编辑Dockerfile 文件](#33-编辑dockerfile-文件)
	- [3.4. 构建SSH服务镜像](#34-构建ssh服务镜像)
	- [3.5. 通过创建的SSH服务镜像创建SSH服务容器](#35-通过创建的ssh服务镜像创建ssh服务容器)
	- [3.6. 通过ssh 客户端连接SSH服务容器](#36-通过ssh-客户端连接ssh服务容器)
- [4. 总结：](#4-总结)
<!--more-->
# 1. SSH （Secure Shell） 简介
SSH是最为常见的远程连接协议，其是可以帮助我们在互联网中使用Shell 的程序和协议。SSH 为我们在互联网中传递对服务器的操作，并对服务器返回的结果进行加密，以确保远程操作服务器时的安全；对服务器的远程操作，Shell程序比必然是输入指令和展示结果最有效的工具，每一位开发者或多或少都会对Shell程序有所了解；对于为运维人员，使用Shell程序更是家常便饭，对服务器的所有操作，以及查看监测服务器的状态都需要通过Shell程序。

# 2. SSH服务适用场景
通过SSH服务连接和访问到容器内部

通过SSH服务管理应用配置文件等，使用和配置隔离，防止因为服务被入侵后的安全问题；

## 2.1. sshd 服务启动方式
在Linux服务器系统中习惯以系统服务的方式启动Linux，但在Docker环境下，我们需要让容器绑定到sshd 服务的进程上，因此需要直接使用SSHD程序启动；
默认SSHD程序会被安装在/usr/sbin 下，并以守护进程的方式启动SSH 服务端程序；因此，我们需要通过 -D 参数使它转换到前台运行；
```sh
# /usr/sbin/sshd -D 
```
当服务端程序启动之后，程序会监听默认端口 22 ，接收并响应客户端的请求，即通过客户端远程连接访问服务器。也可以在连接时，通过-p 参数连接自定义的端口。

# 3. Docker构建SSH服务镜像
##  3.1. SSH服务镜像说明
Alpine 开源，轻量linux 系统，适合轻量服务开发  
OpenSSH ，SSHD服务程序、ssh 客户端等，远程连接访问工具  
## 3.2. 创建Dockerfile目录
```sh
 #  mkdir alpine_ssh && cd alpine_ssh && touch Dockerfile-ssh
```
## 3.3. 编辑Dockerfile 文件
```sh
# 指定创建的基础镜像
FROM alpine
 
 # 作者描述信息
LABEL MAINTAINER "xzeu admin<@xzeu.com>"
LABEL DESC "alpine_sshd_service"
 
# 替换阿里云的并更新源、安装openssh 并修改配置文件和生成key 并且同步时间
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \    
	&& apk update \    
	&& apk add --no-cache openssh tzdata \
	&& cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
	&& sed -i "s/#PermitRootLogin.*/PermitRootLogin yes/g" /etc/ssh/sshd_config \
	&& ssh-keygen -t dsa -P "" -f /etc/ssh/ssh_host_dsa_key \
	&& ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key \ 
	&& ssh-keygen -t ecdsa -P "" -f /etc/ssh/ssh_host_ecdsa_key \
	&& ssh-keygen -t ed25519 -P "" -f /etc/ssh/ssh_host_ed25519_key \
	&& echo "root:admin" | chpasswd

# 开放22端口
EXPOSE 22
 
# 容器启动时执行ssh启动命令
CMD ["/usr/sbin/sshd", "-D"]
```

## 3.4. 构建SSH服务镜像
通过Docker build 构建SSH服务：
```sh
[root@localhost SSHD]# docker build -t mysshd:v2 .  
Sending build context to Docker daemon  31.23kB
Step 1/5 : FROM alpine:3.11
 ---> 3ddbc1193bef
Step 2/5 : MAINTAINER alpine_sshd_service
 ---> Running in b6b66b0ccf32
Removing intermediate container b6b66b0ccf32
 ---> 6e6bb1e10a62
Step 3/5 : RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories     && apk update   && apk add --no-cache openssh tzdata     && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  && sed -i "s/#PermitRootLogin.*/PermitRootLogin yes/g" /etc/ssh/sshd_config      && ssh-keygen -t dsa -P "" -f /etc/ssh/ssh_host_dsa_key  && ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key         && ssh-keygen -t ecdsa -P "" -f /etc/ssh/ssh_host_ecdsa_key      && ssh-keygen -t ed25519 -P "" -f /etc/ssh/ssh_host_ed25519_key         && echo "root:admin" | chpasswd
 ---> Running in 34d9ac38d4de
 (1/10) Installing openssh-keygen (8.1_p1-r0)
(2/10) Installing ncurses-terminfo-base (6.1_p20200118-r4)
(3/10) Installing ncurses-libs (6.1_p20200118-r4)
(4/10) Installing libedit (20191211.3.1-r0)
(5/10) Installing openssh-client (8.1_p1-r0)
(6/10) Installing openssh-sftp-server (8.1_p1-r0)
(7/10) Installing openssh-server-common (8.1_p1-r0)
(8/10) Installing openssh-server (8.1_p1-r0
(9/10) Installing openssh (8.1_p1-r0)
(10/10) Installing tzdata (2020a-r0)
Executing busybox-1.31.1-r9.trigger
OK: 15 MiB in 24 packages
........
chpasswd: password for 'root' changed
Removing intermediate container 34d9ac38d4de
 ---> 86b646b9401a
Step 4/5 : EXPOSE 22
 ---> Running in 51f292956b6e
Removing intermediate container 51f292956b6e
 ---> aef65d0f8331
Step 5/5 : CMD ["/usr/sbin/sshd", "-D"]
 ---> Running in 6abd00112f70
Removing intermediate container 6abd00112f70
 ---> 5c8359952fca
'Successfully' built 5c8359952fca
Successfully tagged mysshd:v2
```
## 3.5. 通过创建的SSH服务镜像创建SSH服务容器
```sh
[root@localhost SSHD]# docker run -d --name sshdocker -P mysshd:v2
609dd1c2f1f97098a244af988d805a16fb2ac8d8045dd48f45f2bb9a35bd68b4
[root@localhost SSHD]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                        NAMES
609dd1c2f1f9        mysshd:v2           "/usr/sbin/sshd -D"      3 seconds ago       Up 2 seconds        0.0.0.0:32769->22/tcp                        sshdocker
```
## 3.6. 通过ssh 客户端连接SSH服务容器
通过ssh 客户端连接SSH服务容器，来进行容器内的访问和管理；
```sh
[root@localhost SSHD]# ssh -p 32769 root@192.168.5.128
The authenticity of host '[192.168.5.128]:32769 ([192.168.5.128]:32769)' can't be established.
ECDSA key fingerprint is SHA256:6WCdoR+cRGc7qHuIqInuIa0XQ1XnaaXUJsLHrIjwVyw.
ECDSA key fingerprint is MD5:d4:b7:82:43:8e:90:01:df:c7:29:85:89:ff:98:59:27.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.5.128]:32769' (ECDSA) to the list of known hosts.
root@192.168.5.128's password:
Welcome to Alpine!

The Alpine Wiki contains a large amount of how-to guides and general
information about administrating Alpine systems.
See <http://wiki.alpinelinux.org/>.

You can setup the system with the command: setup-alpine

You may change this message by editing /etc/motd.

609dd1c2f1f9:~# cat /etc/issue
Welcome to Alpine Linux 3.11
Kernel \r on an \m (\l)

# 查看当前目录
609dd1c2f1f9:~# pwd  
/root
```

# 4. 总结：

到此一个简单的SSH服务容器的镜像生成到完成远程的链接访问就完成，接下来，我会通过数据卷管理容器，以实现将程序拆分到不同的容器中，形成不同容器中的微服务。利用数据卷容器封装特性，更好的将动态代码、程序的配置管理、输出日志的导出等文件，实现快速迁移的便利。为数据迁移提供良好的适应性与项目中对Docker的使用提供了一致性保障。