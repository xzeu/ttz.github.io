---
title: "安装Docker"
date: 2023-09-25T16:50:25+08:00
lastmod: 2023-09-25T16:50:25+08:00
draft: false
keywords: [docker]
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
- [1. 安装docker引擎](#1-安装docker引擎)
- [2. 启动docker](#2-启动docker)
- [3. 守护进程](#3-守护进程)
- [4. 容器基本命令](#4-容器基本命令)
- [5. 日志配置](#5-日志配置)
- [6. 常见参数](#6-常见参数)

官方所有操作系统安装教程：[Install Docker Engine on CentOS | Docker Documentation](https://docs.docker.com/engine/install/centos/)，其中CentOS安装docker引擎的代码：

安装yum-utils，配置库的地址

```sh
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

<!--more-->

# 1. 安装docker引擎

```shell
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

# 2. 启动docker

```shell
sudo systemctl start docker
```

# 3. 守护进程

```bash
systemctl start docker     #运行Docker守护进程
systemctl stop docker      #停止Docker守护进程
systemctl restart docker   #重启Docker守护进程
systemctl enable docker    #设置Docker开机自启动
systemctl status docker    #查看Docker的运行状态
```

设置防火墙

```bash
systemctl status firewalld.service   #查看防火墙状态
systemctl stop firewalld.service     #暂停防火墙
systemctl disable firewalld.service  #永久关闭防火墙
```

镜像基本命令

```bash
docker images                                                       #查看所有镜像
docker pull nginx:latest                                            #拉取nginx镜像
docker rmi nginx                                                    #删除nginx镜像
 
docker save -o ***.tar ImageName:latest                             #导出镜像
docker load -i ***.tar                                              #导入镜像
 
docker image tag ImageName:latest NewImageName:latest               #打标签
docker push ImageName:latest                                        #推送镜像
```

# 4. 容器基本命令

```bash
docker ps                                                           #查看运行中容器
docker pa -a                                                        #查看所有容器

docker run ImageName:latest                                         #从镜像中运行容器
docker start ContainerId                                            #运行容器
docker stop ContainerId                                             #暂停容器
docker restart ContainerId                                          #重新运行容器
docker kill ContainerId                                             #强制暂停容器
docker rm ContainerId                                               #删除容器
docker rm -f ContainerId                                            #强制删除容器
docker logs ContainerId                                             #查看容器日志
docker exec -it ContainerId /bin/bash                               #进入容器
exit                                                                #退出容器
docker commit  -m "描述" ContainerId ImageName:latest               #从容器中生成新镜像
```

# 5. 日志配置

```bash
--log-opt max-size=100m                                             #日志文件最大100M
--log-opt max-file=5                                                #最多五个日志文件，默认值：1
```

# 6. 常见参数

```bash
-p 5000:5000                                                        #端口映射
-d                                                                  #后台运行
-it /bin/bash                                                       #交互式容器，进入容器的/bin/bash
--restart=always                                                    #容器重启策略
--name ContainerName                                                #容器名称
-v /usr/local/auth:/auth                                            #挂载文件
-e REGISTRY_AUTH=htpasswd                                           #配置容器的环境变量
示例：docker run --name my-custom-nginx-container -v /host/path/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx
```
