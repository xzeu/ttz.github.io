---
title: "Dockerfile操作指令"
date: 2023-04-11T12:14:29+08:00
lastmod: 2023-04-11T12:14:29+08:00
draft: false
keywords: [dockerfile,docker]
description: ""
tags: [dockerfile,docker]
categories: [教程笔记]
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
- [1. DockerFile介绍](#1-dockerfile介绍)
- [2. DockerFile指令](#2-dockerfile指令)
  - [2.1. FROM指令](#21-from指令)
    - [2.1.1. 基本概述](#211-基本概述)
    - [2.1.2. 用法](#212-用法)
    - [2.1.3. 关于FROM scratch](#213-关于from-scratch)
  - [2.2. RUN指令](#22-run指令)
    - [2.2.1. 基本概述](#221-基本概述)
    - [2.2.2. 用法](#222-用法)
  - [2.3. ENV指令](#23-env指令)
    - [2.3.1. 基本概述](#231-基本概述)
    - [2.3.2. 用法](#232-用法)
  - [2.4. COPY指令](#24-copy指令)
    - [2.4.1. 基本概述](#241-基本概述)
    - [2.4.2. 用法](#242-用法)
  - [2.5. ADD指令](#25-add指令)
    - [2.5.1. 基本概述](#251-基本概述)
    - [2.5.2. 用法](#252-用法)
  - [2.6. CMD指令](#26-cmd指令)
    - [2.6.1. 基本概述](#261-基本概述)
    - [2.6.2. 用法](#262-用法)
  - [2.7. ENTRYPOINT指令](#27-entrypoint指令)
    - [2.7.1. 基本概述](#271-基本概述)
    - [2.7.2. 用法](#272-用法)
  - [2.8. ARG指令](#28-arg指令)
    - [2.8.1. 基本概述](#281-基本概述)
    - [2.8.2. 用法](#282-用法)
  - [2.9. VOLUME指令](#29-volume指令)
    - [2.9.1. 基本概述](#291-基本概述)
    - [2.9.2. 用法](#292-用法)
  - [2.10. EXPOSE](#210-expose)
    - [2.10.1. 基本概述](#2101-基本概述)
    - [2.10.2. 用法](#2102-用法)
  - [2.11. WORKDIR指令](#211-workdir指令)
    - [2.11.1. 基本概述](#2111-基本概述)
    - [2.11.2. 用法](#2112-用法)
- [3. 总结](#3-总结)
  - [3.1. ADD和copy区别](#31-add和copy区别)
  - [3.2. CMD和entrypoint区别](#32-cmd和entrypoint区别)

<!--more-->

# 1. DockerFile介绍

dockerfile是用来构建dokcer镜像的文件!命令参数脚本!
通过以下脚本可以生成镜像，镜像是一层一层的，脚本一个个的命令，每个命令都是一层！

```sh
# 创建一个dockerfile文件，名字可以自定义，建议Dockerfile
# 文件中的内容 指令（大写） 参数
FROM centos

VOLUME ["volume01","volume02"]

CMD echo “----end----”

CMD /bin/bash

# 这里的每个命令，就是镜像一层！
```

`docker build` 令用于从Dockerfile构建镜像。可以在 `docker build`命令中使用-f标志指向文件系统中任何位置的Dockerfile。

**构建步骤︰**
1、编写一个dockerfile 文件
2、docker build构建成为一个镜像
3、docker run运行镜像
4、docker push 发布镜像(DockerHub、阿里云镜像仓库!)

**基础知识∶**
1、每个保留关键字（指令）都是尽量是大写字母
2、执行从上到下顺序执行
3、#表示注释
4、每一个指令都会创建提交一个新的镜像层，并提交!
![](../images/docker.png)

`dockerfile`是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerfile文件，这个文件十分简单!

Docker镜像逐渐成为企业交付的标准，必须要掌握!

**步骤:**

1. DockerFile :构建文件，定义了一切的步骤，源代码
2. Dockerlmages :通过DockerFile构建生成的镜像，最终发布和运行的产品
3. Docker容器︰容器就是镜像运行起来提供服务器

# 2. DockerFile指令

| 序号 | 指令                                      | 含义                                                                                                                             |
| ---- | ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| 1    | FROM                                      | 镜像 指定新镜像所基于的镜像，第 条指必须为from指令，每创建一个镜像就需要一条from指令                                             |
| 2    | MAINTAINER                                | 名字 说明新镜像的维护人信息                                                                                                      |
| 3    | RUN                                       | 命令 在所基于的镜像上执行命令，并提交到新的镜像中；docker内每执行一条命令都是run开头                                             |
| 4    | CMD[“要运行的程序”,“参数1”,“参数2”] | 指令启动容器时要运行的命令或者脚本，Dockerfile只能有一条CMD命令， 如果指定多条则只能最后一条被执行                               |
| 5    | EXPOSE                                    | 端口号 指定新镜像加载到Docker时要开启的端口                                                                                      |
| 6    | ENV                                       | 环境变量 变量值 设置一个环境变量的值，会被后面的run使用                                                                          |
| 7    | ADD                                       | 源文件、目录 目标文件/目录 具体识别压缩格式并且自动解压，将源文件复制到目标文件，源文件要与dockerfile位于相同目录中，或者一个URL |
| 8    | COPY                                      | 源文件/目录 目标文件/目录 将本地主机上的文件/目录复制到目标地点，源文件/目录要与Dockerfile在相同的目录中                         |
| 9    | VOLUME                                    | [“目录”] 在容器中创建一个挂载点                                                                                                |
| 10   | USER                                      | 用户名/UID 指定运行容器时的用户                                                                                                  |
| 11   | WORKDIR                                   | 路径 为后续的RUN、CMD、ENTRYPOINT指定工作目录                                                                                    |
| 12   | ONBUILD                                   | 命令 指定所生成的镜像作为一个基础镜像时所要运行的命令                                                                            |
| 13   | HEALTHCHECK                               | 健康检查                                                                                                                         |

## 2.1. FROM指令

### 2.1.1. 基本概述

指定基础镜像，在基础镜像上进行定制
FROM就是指定基础镜像，此指令通常必需放在 `Dockerfile`文件第一个非注释行。后续的指令都是运行于此基准镜像所提供的运行环境

基础镜像可以是任何可用镜像文件，默认情况下，`docker build`会在docker主机上查找指定的镜像文件，在其不存在时，则会从 `Docker Hub Registry` 上拉取所需的镜像文件.如果找不到指定的镜像文件，`docker build`会返回一个错误信息

### 2.1.2. 用法

```sh
FROM [--platform=<platform>] <image> [AS <name>]  
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]  
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]  
```

```sh
FROM scratch 		# 所有镜像的起源镜像，相当于Object类
FROM ubuntu
FROM ubuntu:bionic
FROM debian:buster-slim
```

{{< admonition type=info title="说明" open=false >}}
–platform 指定镜像的平台，比如: linux/amd64, linux/arm64, or windows/amd64
tag 和 digest是可选项，如果不指定，默认为latest
{{< /admonition >}}

### 2.1.3. 关于FROM scratch

scratch是祖宗镜像，所有的镜像都源于此创建，也是一个空镜像，一般要从零开始构建镜像时，用此方法

**参考链接:**

https://hub.docker.com/_/scratch?tab=description
https://docs.docker.com/develop/develop-images/baseimages/
该镜像是一个空的镜像，可以用于构建busybox等超小镜像，可以说是真正的从零开始构建属于自己的镜像

该镜像在构建基础镜像（如debian和busybox）或超最小镜像（仅包含一个二进制文件及其所需内容，如:hello-world）的上下文中最有用

## 2.2. RUN指令

### 2.2.1. 基本概述

在构建镜像阶段需要执行FROM，指定镜像所支持的shell命令，一般各种基础镜像都支持普通shell命令

注意:

RUN可以写多个，每一个RUN指令都会建立一个镜像层，所以尽可能合并成一条指令,比如将多个shell命令通过“&&”符号连接一起成为在一条指令
每个RUN都是独立运行的,和前一个RUN无关

### 2.2.2. 用法

```sh
#shell格式： 支持环境变量
RUN <命令>
 
#exec格式：
RUN ["可执行文件", "参数1", "参数2"]
 
#exec格式指定其他shell：
RUN ["/bin/bash","-c","echo hello wang"]
```

```sh
RUN echo "<h1>web 123</h1>" >/var/www/html/index.html
RUN ["/bin/bash","-c","echo 123456"]
RUN yum install -y epel-release \
  && yum install -y nginx \
  && echo "hj's web page" > /usr/share/nginx/html/index.html
```

{{< admonition type=info title="说明" open=false >}}
shell格式中，通常是一个shell命令，且以 "/bin/sh -c” 来运行它，这意味着此进程在容器中的PID不为1，不能接收Unix信号，因此，当使用docker stop 命令停止容器时，此进程接收不到SIGTERM信号
exec格式中的参数是一个JSON格式的数组，其中为要运行的命令，后面的为传递给命令的选项或参数;然而，此种格式指定的命令不会以"/bin/sh -c"来发起，因此常见的shell操作如变量替换、通配符(?,*等)替换将不会进行;不过，如果要运行的命令依赖于此shell特性的话，可替换为类似下面的格式
{{< /admonition >}}

## 2.3. ENV指令

### 2.3.1. 基本概述

以定义环境变量和值，会被后续指令(如:ENV,ADD,COPY,RUN等)通过或KEY或{KEY}进行引用，并在容器运行时保持

### 2.3.2. 用法

**格式1：**

此格式只能对一个key赋值,之后的所有内容均会被视作其的内容

```sh
ENV <key> <value>
env a1 123
ENV a2 qwe
```

**格式2：**

此格式可以支持多个key赋值,定义多个变量建议使用,减少镜像层。还是 \ 位换行

```sh
ENV <key1>=<value1> <key2>=<value2> \
 <key3>=<value3> ...
```

```sh
ENV user=hj pswd=123 \
  xxx=123
```

## 2.4. COPY指令

### 2.4.1. 基本概述

复制本地宿主机的到容器中

### 2.4.2. 用法

```sh
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]	适用于路径有空白字符的情况
```

```sh
COPY test.txt /root/		本地文件到容器root目录
```

{{< admonition type=info title="说明" open=false >}}
可以是多个,可以使用通配符，通配符规则满足Go语言的filepath.Match规则
只能复制在Dockerfile文件同级目录下的文件，也就是以Dockerfile存放目录为根目录
如果是目录，则其内部文件或子目录会被递归复制，但目录自身不会被复制
如果指定了多个,或在中使用了通配符，则dest必须是一个目录，且必须以 / 结尾
可以是绝对路径或者是 WORKDIR 指定的相对路径
使用COPY指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。也就是说可以加速构建速度，一般可用此优化镜像构建
如果事先不存在，它将会被自动创建，这包括其父目录路径,即递归创建目录
{{< /admonition >}}

## 2.5. ADD指令

### 2.5.1. 基本概述

增强版的COPY，不仅支持COPY，还支持自动解缩。可以将复制指定的到容器中的

### 2.5.2. 用法

```sh
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] 
```

```sh
ADD a.tgz /opt/ 		文件传到容器的/opt/目录后自动解压
```

{{< admonition type=info title="说明" open=false >}}
可以是Dockerfile所在目录的一个相对路径；也可是一个 URL；还可是一个 tar 文件（自动解压）
可以是绝对路径或者是 WORKDIR 指定的相对路径
如果是目录，只复制目录中的内容，而非目录本身
如果是一个 URL ，下载后的文件权限自动设置为 600
如果为URL且不以/结尾，则指定的文件将被下载并直接被创建为,如果以“\”结尾，则文件名URL指定的文件将被直接下载并保存为/
如果是一个本地文件系统上的打包文件,如: gz, bz2 ,xz ，它将被解包 ，其行为类似于"tar -x"命令, 但是通过URL获取到的tar文件将不会自动展开
如果有多个，或其间接或直接使用了通配符，则必须是一个以/结尾的目录路径;如果不以/结尾，则其被视作一个普通文件，的内容将被直接写入到
{{< /admonition >}}

## 2.6. CMD指令

### 2.6.1. 基本概述

容器中需要持续运行的进程一般只有一个,CMD用来指定启动容器时默认执行的一个命令，且其运行结束后,容器也会停止,所以一般CMD 指定的命令为持续运行且为前台命令

RUN是构建时运行的指令，CMD是容器运行时运行的指令

### 2.6.2. 用法

```sh
#exec格式执行：推荐使用，且不支持环境变量  
CMD ["命令","param1","param2"]
 
#sh格式执行：交互式使用，支持环境变量  
CMD command param1 param2
 
#给entrypoint命令的传递参数：  
CMD ["param1","param2"]
```

```sh
CMD ["nginx","-g","daemon off;"]
CMD ["curl","-s","https://ip.cn"]	同一文件中，只有此命令生效
```

{{< admonition type=info title="说明" open=false >}}
如果docker run没有指定任何的执行命令或者dockerfile里面也没有ENTRYPOINT，那么开启容器时就会使用执行CMD指定的默认的命令

每个Dockerfile只能有一条CMD命令。如指定了多条，只有最后一条被执行
如果启动容器时用 docker run xxx cmd 指定运行的命令，则会覆盖 CMD 指定的命令
{{< /admonition >}}

## 2.7. ENTRYPOINT指令

### 2.7.1. 基本概述

类似CMD，配置容器启动后执行的命令及参数
如果与cmd命令共存，则cmd的内容当做参数传递给entrypoint

### 2.7.2. 用法

```sh
#exc执行
ENTRYPOINT ["executable", "param1", "param2"]
 
#sh执行
ENTRYPOINT command param1 param2
```

```sh
#CMD的内容做参数传给ENTRYPOINT的命令
CMD ["-g","daemon off;"]
ENTRYPOINT ["nginx"]
```

{{< admonition type=info title="说明" open=false >}}
docker run -e/–env-file 不能覆盖ENTRYPOINT的参数，而是把传递的参数追加在ENTRYPOINT原有的参数后面
如果docker run 后面没有额外参数，但是dockerfile中的CMD里有（即上面CMD的第三种用法），即Dockerfile中即有CMD也有ENTRYPOINT,那么CMD的全部内容会作为ENTRYPOINT的参数
如果docker run 后面有额外参数，同时Dockerfile中即有CMD也有ENTRYPOINT,那么docker run 后面的参数覆盖掉CMD参数内容,最终作为ENTRYPOINT的参数
可以通过docker run --entrypoint string 参数在运行时替换,注意string不要加空格
使用CMD要在运行时重新写命令本身,然后在后面才能追加运行参数，ENTRYPOINT则可以运行时无需重写命令就可以直接接受新参数
每个Dockerfile中只能有一个ENTRYPOINT，当指定多个时，只有最后一个生效
{{< /admonition >}}

## 2.8. ARG指令

### 2.8.1. 基本概述

在build阶段指定变量,和ENV不同的是，容器运行时不会存在这些环境变量，只在构建时生效

### 2.8.2. 用法

```sh
ARG <name>[=<default value>]
docker build --build-arg <参数名>=<值>
```

```sh
vim xx/Dockerfile
ARG version=1.0
FROM base:${version}
```

{{< admonition type=info title="说明" open=false >}}
如果变量名与ENV同名，则ENV覆盖ARG
支持做最开始的变量定义，也就是写在FROM之前
{{< /admonition >}}

## 2.9. VOLUME指令

### 2.9.1. 基本概述

VOLUME实现的是匿名数据卷,无法指定宿主机路径和容器目录的挂载关系

通过 `docker rm -fv <容器ID> `可以删除容器的同时删除VOLUME指定的卷

宿主机目录为：

/var/lib/docker/volumes/<volume_id>/_data

指定容器内的挂载目录后，物理机自动在该路径下创建目录，卷id是随机生成的，_data就是容器内挂载的目录名,卷id以后就是挂载点根目录

当容器删除时，容器内在/data/下存的文件，还保留在物理机的/var/lib/docker/volumes/卷id/_data/

### 2.9.2. 用法

```sh
VOLUME <容器内路径>
VOLUME ["<容器内路径1>", "<容器内路径2>"...]
docker rm -v 容器 		删除容器时，删除对应的匿名卷
```

```sh
VOLUME ["/data","/data2"]
```

## 2.10. EXPOSE

### 2.10.1. 基本概述

EXPOSE仅仅是声明容器打算使用什么端口而已,并不会真正暴露端口,即不会自动在宿主进行端口映射
启动容器是还是需要-p/-P来指明与物理机映射的端口才能真正分配端口

注意:

即使 Dockerfile没有EXPOSE 端口指令,也可以通过docker run -p 临时暴露容器内程序真正监听的端口,所以EXPOSE 相当于指定默认的暴露端口,可以通过docker run -P 进行真正暴露

### 2.10.2. 用法

```sh
EXPOSE <port>[/ <protocol>] [<port>[/ <protocol>] ..]
```

```sh
EXPOSE 80 443/tcp
EXPOSE 3306 
```

{{< admonition type=info title="说明" open=false >}}
可为tcp或udp二者之一，默认为TCP协议
{{< /admonition >}}

## 2.11. WORKDIR指令

### 2.11.1. 基本概述

为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录，当容器运行后，进入容器内WORKDIR指定的默认目录
如果目录不存在，会自动创建

### 2.11.2. 用法

```sh
WORKDIR /path/to/workdir
```

```sh
WORKDIR /web
RUN echo 123456 > ts.txt
WORKDIR /a
WORKDIR b		切换多级目录，/a/b 
```

# 3. 总结

## 3.1. ADD和copy区别

1. Dockerfile中的COPY指令和ADD指令都可以将主机上的资源复制或加入到容器镜像中，都是在构建镜像的过程中完成的
2. copy只能用于复制（节省资源）
3. ADD复制的同时，如果复制的对象时压缩包，ADD还可以解压（消耗资源）
4. COPY指令和ADD指令的唯一区别在于是否支持从远程URL获取资源。COPY指令只能从执行docker build所在的主机上读取资源并复制到镜像中。而ADD指令还支持通过URL从远程服务器读取资源并复制到镜像中
5. 满足同等功能的情况下，推荐使用COPY指令。ADD指令更擅长读取本地tar文件并解压缩

## 3.2. CMD和entrypoint区别

一般还是会用entrypoint的中括号形式作为docker 容器启动以后的默认执行命令，里面放的是不变的部分，可变部分比如命令参数可以使用cmd的形式提供默认版本，也就是run里面没有任何参数时使用的默认参数。如果我们想用默认参数，就直接run，否则想用其他参数，就run 里面加参数。
