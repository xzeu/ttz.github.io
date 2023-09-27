---
title: "使用 Ingress-Nginx 进行灰度(金丝雀)发布"
date: 2023-08-30T10:07:40+08:00
lastmod: 2023-08-30T10:07:40+08:00
draft: false
keywords: [ingress,nginx]
description: ""
tags: [教程笔记,docker,linux,nginx]
categories: [k8s]
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
- [1. Ingress-Nginx Canary介绍](#1-ingress-nginx-canary介绍)
- [2. ingress-nginx Canary实现](#2-ingress-nginx-canary实现)
  - [2.1. 基于客户端请求的流量切分场景](#21-基于客户端请求的流量切分场景)
- [3. 金丝雀发布的高级功能](#3-金丝雀发布的高级功能)
- [4. 阿里开源ingress-nginx实现](#4-阿里开源ingress-nginx实现)
  - [4.1. 基础理论](#41-基础理论)
  - [4.2. 案例](#42-案例)

<!--more-->

# 1. Ingress-Nginx Canary介绍

Nginx Ingress Controller 作为项目对外的流量入口和项目中各个服务的反向代理。

官方文档概述：Annotations - Ingress-Nginx Controller (kubernetes.github.io)

Nginx Annotations 的几种 Canary 规则：

| Annotation                                           | 说明                                                                                                                                                                                                                                                                |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| nginx.ingress.kubernetes.io/canary                   | 必须设置该Annotation值为 `true`，否则其它规则将不会生效。取值：`true`：启用 `canary`功能。`false`：不启用 `canary`功能。                                                                                                                                            |
| nginx.ingress.kubernetes.io/canary-by-header         | 表示基于请求头的名称进行灰度发布。请求头名称的特殊取值：`always`：无论什么情况下，流量均会进入灰度服务。 `never`：无论什么情况下，流量均不会进入灰度服务。 若没有指定请求头名称的值，则只要该头存在，都会进行流量转发。                                             |
| nginx.ingress.kubernetes.io/canary-by-header-value   | 表示基于请求头的值进行灰度发布。 需要与 `canary-by-header`头配合使用。                                                                                                                                                                                              |
| nginx.ingress.kubernetes.io/canary-by-header-pattern | 表示基于请求头的值进行灰度发布，并对请求头的值进行正则匹配。 需要与 `canary-by-header`头配合使用。 取值为用于匹配请求头的值的正则表达式。                                                                                                                           |
| nginx.ingress.kubernetes.io/canary-by-cookie         | 表示基于Cookie进行灰度发布。例如，`nginx.ingress.kubernetes.io/canary-by-cookie: foo`。 Cookie内容的取值：  `always`：当 `foo=always`，流量会进入灰度服务。 `never`：当 `foo=never`，流量不会进入灰度服务。 只有当Cookie存在，且值为 `always`时，才会进行流量转发。 |
| nginx.ingress.kubernetes.io/canary-weight            | 表示基于权重进行灰度发布。 取值范围：0~权重总值。 若未设定总值，默认总值为100。                                                                                                                                                                                     |
| nginx.ingress.kubernetes.io/canary-weight-total      | 表示设定的权重总值。 若未设定总值，默认总值为100。                                                                                                                                                                                                                  |

注意：不同灰度方式的优先级 由高到低 为：

> canary-by-header --> canary-by-cookie --> canary-weight

# 2. ingress-nginx Canary实现

## 2.1. 基于客户端请求的流量切分场景

需求：

假设当前线上环境，您已经有一套服务Service V1对外提供7层服务，此时上线了一些新的特性，需要发布上线一个新的版本Service V2。

希望将请求头中包含foo=bar或者Cookie中包含foo=bar的客户端请求转发到Service V2服务中。

待运行一段时间稳定后，可将所有的流量从Service V1切换到Service V2服务中，再平滑地将Service V1服务下线。

![Alt text](../images/image-20230830101233948.png)

通过上面的annotation来实现灰度发布，其 思路如下：

1. 在集群中部署两套系统，一套是stable版本(old-nginx)，一套是canary版本(new-nginx)，两个版本都有自己的service；
2. 定义两个ingress配置，一个正常提供服务，一个增加canary的annotation；
3. 待canary版本无误后，将其切换成stable版本，并且将旧的版本下线，流量全部接入新的stable版本

old-nginx 创建Deployment、Service、Ingress。

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
name: old-nginx
spec:
replicas: 2
selector:
  matchLabels:
    run: old-nginx
template:
  metadata:
    labels:
      run: old-nginx
  spec:
    containers:
    - image: registry.cn-hangzhou.aliyuncs.com/acs-sample/old-nginx
      imagePullPolicy: Always
      name: old-nginx
      ports:
      - containerPort: 80
        protocol: TCP
    restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
name: old-nginx
spec:
ports:
- port: 80
  protocol: TCP
  targetPort: 80
selector:
  run: old-nginx
sessionAffinity: None
type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: nginx-ingress
namespace: default
spec:
ingressClassName: nginx
rules:
  - host: nginx.kubernets.cn
    http:
      paths:
        - pathType: Prefix
          backend:
            service:
              name: old-nginx
              port:
                number: 80
          path: /
```

测试验证：（预期输出：old）

```sh
$ curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
```

灰度发布新版本服务：

new-nginx 创建Deployment、Service。

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
name: new-nginx
spec:
replicas: 1
selector:
  matchLabels:
    run: new-nginx
template:
  metadata:
    labels:
      run: new-nginx
  spec:
    containers:
    - image: registry.cn-hangzhou.aliyuncs.com/acs-sample/new-nginx
      imagePullPolicy: Always
      name: new-nginx
      ports:
      - containerPort: 80
        protocol: TCP
    restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
name: new-nginx
spec:
ports:
- port: 80
  protocol: TCP
  targetPort: 80
selector:
  run: new-nginx
sessionAffinity: None
type: ClusterIP
```

设置访问新版本服务的路由规则。

- 设置满足特定规则的客户端才能访问新版本服务。以下示例仅请求头中满足foo=bar的客户端请求才能路由到新版本服务。

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 # 修改名称
name: new-nginx-ingress
namespace: default
annotations:
   # 开启Canary。
  nginx.ingress.kubernetes.io/canary: "true"
   # 请求头为foo。
  nginx.ingress.kubernetes.io/canary-by-header: "foo"
   # 请求头foo的值为bar时，请求才会被路由到新版本服务new-nginx中。
  nginx.ingress.kubernetes.io/canary-by-header-value: "bar"
spec:
ingressClassName: nginx
rules:
  - host: nginx.kubernets.cn
    http:
      paths:
        - pathType: Prefix
          backend:
            service:
               # 选择新pod的svc
              name: new-nginx
              port:
                number: 80
          path: /
```

按照header头信息转发流量：

```sh
# 正常访问：
$ curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
old

# 访问新的服务：
$ curl -H "Host: nginx.kubernets.cn" -H "foo: bar" http://nginx.kubernets.cn
new
```

可以看到，仅请求头中满足foo=bar的客户端请求才能路由到新版本服务。

---

需求：请求头中满足foo=bar的客户端请求，若不包含该请求头，再将50%的流量路由到新版本服务中。

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: new-nginx-ingress
namespace: default
annotations:
   # 开启Canary。
  nginx.ingress.kubernetes.io/canary: "true"
   # 请求头为foo。
  nginx.ingress.kubernetes.io/canary-by-header: "foo"
   # 请求头foo的值为bar时，请求才会被路由到新版本服务new-nginx中。
  nginx.ingress.kubernetes.io/canary-by-header-value: "bar"
   # 在满足上述匹配规则的基础上仅允许50%的流量会被路由到新版本服务new-nginx中。
  nginx.ingress.kubernetes.io/canary-weight: "50"
spec:
ingressClassName: nginx
rules:
  - host: nginx.kubernets.cn
    http:
      paths:
        - pathType: Prefix
          backend:
            service:
              name: new-nginx
              port:
                number: 80
          path: /
```

测试验证：(几乎可以达到50%请求分布)

```sh
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
new
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
old
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
old
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
old
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
new
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
new
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
old
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
old
```

系统运行一段时间后，当新版本服务已经稳定并且符合预期后，需要下线老版本的服务 ，仅保留新版本服务在线上运行。

为了达到该目标，需要将旧版本的Service指向新版本服务的Deployment，并且删除旧版本的Deployment和新版本的Service。

```yml
apiVersion: v1
kind: Service
metadata:
name: old-nginx
spec:
ports:
- port: 80
  protocol: TCP
  targetPort: 80
selector:
   # 指向新版本服务。
  run: new-nginx
sessionAffinity: None
type: ClusterIP
```

预期输出：

```sh
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
new
```

# 3. 金丝雀发布的高级功能

如上只简单介绍了一些ingress开源默认支持的Annotation。

日常工作中基于开源ingress-nginx实线的高级功能：

通过修改 `nginx.ingress.kubernetes.io/configuration-snippet`配置，并且配置正则实现：

- 当header头中有关键字（foo 或 new）字段的时候，自动将流量转发至new-nginx；
- nginx.ingress.kubernetes.io/configuration-snippet （用于插入 location 块代码段）;
- nginx.ingress.kubernetes.io/server-snippet （用于插入 server 块中的代码段）;

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: new-nginx-ingress
namespace: default
annotations:
  nginx.ingress.kubernetes.io/configuration-snippet: |
    if ($http_name ~ "^.*foo$|^.*new$") {
        proxy_pass http://new-nginx.default:80;
        break;
    }
spec:
ingressClassName: nginx
rules:
  - host: nginx.kubernets.cn
    http:
      paths:
      - path: /
        backend:
          service:
            name: old-nginx
            port:
              number: 80
        pathType: Prefix
      - path: /
        backend:
          service:
            name: new-nginx
            port:
              number: 80
        pathType: Prefix
```

测试验证：

```sh
# curl -H "name: new" http://nginx.kubernets.cn
new
# curl -H "name: foo" http://nginx.kubernets.cn
new
# curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
old
```

# 4. 阿里开源ingress-nginx实现

## 4.1. 基础理论

Nginx Ingress Controller通过Annotation来支持应用服务的灰度发布机制。

`nginx.ingress.kubernetes.io/service-match：`该注解用来配置新版本服务的路由匹配规则。

```sh
nginx.ingress.kubernetes.io/service-match: |
  <service-name>: <match-rule>
# 参数说明：
# service-name：服务名称，满足match-rule的请求会被路由到该服务中。
# match-rule：路由匹配规则
#
# 路由匹配规则：
# 1. 支持的匹配类型
# - header：基于请求头，支持正则匹配和完整匹配。
# - cookie：基于cookie，支持正则匹配和完整匹配。
# - query：基于请求参数，支持正则匹配和完整匹配。
#
# 2. 匹配方式
# - 正则匹配格式：/{regular expression}/，//表明采用正则方式匹配。
# - 完整匹配格式："{exact expression}"，""表明采用完整方式匹配。

```

路由匹配规则配置示例：

```sh
# 请求头中满足foo正则匹配^bar$的请求被转发到新版本服务new-nginx中。
new-nginx: header("foo", /^bar$/)
# 请求头中满足foo完整匹配bar的请求被转发到新版本服务new-nginx中。
new-nginx: header("foo", "bar")
# cookie中满足foo正则匹配^sticky-.+$的请求被转发到新版本服务new-nginx中。
new-nginx: cookie("foo", /^sticky-.+$/)
# query param中满足foo完整匹配bar的请求被转发到新版本服务new-nginx中。
new-nginx: query("foo", "bar")
```

`nginx.ingress.kubernetes.io/service-weight：`该注解用来配置新旧版本服务的流量权重。

```sh
nginx.ingress.kubernetes.io/service-weight: |
  <new-svc-name>:<new-svc-weight>, <old-svc-name>:<old-svc-weight>
参数说明：
new-svc-name：新版本服务名称
new-svc-weight：新版本服务权重
old-svc-name：旧版本服务名称
old-svc-weight：旧版本服务权重

配置示例：
nginx.ingress.kubernetes.io/service-weight: |
  new-nginx: 20, old-nginx: 60
```

## 4.2. 案例

创建老服务的：Deployment、Service。

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
name: old-nginx
spec:
replicas: 2
selector:
  matchLabels:
    run: old-nginx
template:
  metadata:
    labels:
      run: old-nginx
  spec:
    containers:
    - image: registry.cn-hangzhou.aliyuncs.com/acs-sample/old-nginx
      imagePullPolicy: Always
      name: old-nginx
      ports:
      - containerPort: 80
        protocol: TCP
    restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
name: old-nginx
spec:
ports:
- port: 80
  protocol: TCP
  targetPort: 80
selector:
  run: old-nginx
sessionAffinity: None
type: ClusterIP
```

Ingress资源创建：

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: gray-release
namespace: default
spec:
ingressClassName: nginx
rules:
  - host: nginx.kubernets.cn
    http:
      paths:
        - pathType: Prefix
          backend:
            service:
              name: old-nginx
              port:
                number: 80
          path: /
```

灰度发布新版本服务

发布一个新版本的Nginx服务并配置路由规则。

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
name: new-nginx
spec:
replicas: 1
selector:
  matchLabels:
    run: new-nginx
template:
  metadata:
    labels:
      run: new-nginx
  spec:
    containers:
    - image: registry.cn-hangzhou.aliyuncs.com/acs-sample/new-nginx
      imagePullPolicy: Always
      name: new-nginx
      ports:
      - containerPort: 80
        protocol: TCP
    restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
name: new-nginx
spec:
ports:
- port: 80
  protocol: TCP
  targetPort: 80
selector:
  run: new-nginx
sessionAffinity: None
type: ClusterIP
```

设置满足特定规则的客户端才能访问新版本服务。以下示例仅请求头中满足foo=bar的客户端请求才能路由到新版本服务。

- 修改如上步骤创建的Ingress。

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: gray-release
namespace: default
annotations:
   # 请求头中满足正则匹配foo=bar的请求才会被路由到新版本服务new-nginx中。
  nginx.ingress.kubernetes.io/service-match: |
    new-nginx: header("foo", /^bar$/)
spec:
ingressClassName: nginx
rules:
  - host: nginx.kubernets.cn
    http:
      paths:
       # 老版本服务
      - path: /
        backend:
          service:
            name: old-nginx
            port:
              number: 80
        pathType: ImplementationSpecific
       # 新版本服务
      - path: /
        backend:
          service:
            name: new-nginx
            port:
              number: 80
        pathType: ImplementationSpecific
```

测试验证：

执行以下命令，访问当前服务：

```sh
$ curl -H "Host: nginx.kubernets.cn" http://nginx.kubernets.cn
old
```

执行以下命令，请求头中满足foo=bar的客户端请求访问服务：

```
$ curl -H "Host: nginx.kubernets.cn" -H "foo: bar" http://nginx.kubernets.cn
new
```
