---
title: "安装最新VL版office快捷指南"
date: 2023-03-15T13:27:50+08:00
lastmod: 2023-03-15T13:27:50+08:00
draft: false
keywords: [office,kms,kms.ttz.pw]
description: ""
tags: [office,kms,kms.ttz.pw]
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
mathjax: false

# Uncomment to add to the homepage's dropdown menu; weight = order of article
# menu:
#   main:
#     parent: "docs"
#     weight: 1
---
- [1 下载ODT工具并解压](#1-下载odt工具并解压)
- [2 配置你想要的Office](#2-配置你想要的office)
- [3 安装你心仪的office](#3-安装你心仪的office)
- [附录](#附录)

此指南是[KMS激活Windows/Office使用指南](/post/kms/)的补充，由于微软在Office 2016之后的版本均不提供ISO镜像下载（指正版商业镜像），VL版本的Office需要管理员手动使用ODT工具进行部署。此教程对ODT工具部署安装Office进行简要描述。
<!--more-->
# 1 下载ODT工具并解压

ODT工具下载： [https://www.microsoft.com/en-us/download/details.aspx?id=49117](https://www.microsoft.com/en-us/download/details.aspx?id=49117)
点击`[Download]`按钮下载微软官网提供的ODT工具，下载下来的是一个自解压程序的压缩包，双击选择解压位置，你就会得到一个红色图标的`setup.exe`和一些示例的xml文件。我们只要其中的`setup.exe`即可。
如果上面的链接打不开，可以尝试复制下载以下链接：
[https://download.microsoft.com/download/2/7/A/27AF1BE6-DD20-4CB4-B154-EBAB8A7D4A7E/officedeploymenttool_15928-20216.exe](https://download.microsoft.com/download/2/7/A/27AF1BE6-DD20-4CB4-B154-EBAB8A7D4A7E/officedeploymenttool_15928-20216.exe)

# 2 配置你想要的Office

配置文件生成: [https://config.office.com/deploymentsettings](https://config.office.com/deploymentsettings)
打开上面的微软Office提供的网站，你可以自由DIY你想要的Office。此处仅介绍几个重点：

- 版本选择：请选择带`批量许可证`字样的版本，同时请不要选择带`SPLA`字样的版本。（注意：如果你要安装Visio之类的组件，LTSC版本会和个人版OFFICE冲突，如果你电脑上有个人版OFFICE导致安装冲突请不要选择带LTSC字样版本。）
- 你可以自由选择安装的组件和语言，比如仅安装Word，Excel和PowerPoint。
- 授权和激活：开启自动接受EULA，点选KMS选项。
- DIY好你心仪的配置之后，可以点击`导出`按钮，如有询问默认文件格式点击`[保留当前配置]`即可。

*你将会下载到一个xml格式的配置文件，为了后续操作方便，你可以重命名为`office.xml`。*

# 3 安装你心仪的office

把步骤1得到的`setup.exe`和步骤2得到的`office.xml`放在同一个目录，在当前目录的地址栏输入`cmd`回车，打开命令提示符窗口，输入以下命令即可执行安装：

```cmd
setup.exe /configure office.xml
```

当然，你可以把你的上面的命令写成cmd文件，跟`setup.exe`和`office.xml`保存在一起，下次直接双击你的cmd脚本即可安装你心仪的office。
你无需保留任何大体积的安装包和关心补丁，每次安装都是联网安装你设置的最新版的Office。

# 附录

如果你想对ODT工具有更多了解，比如使用ODT下载Office等，可参阅微软官方文档：
[https://learn.microsoft.com/zh-cn/deployoffice/overview-office-deployment-tool](https://learn.microsoft.com/zh-cn/deployoffice/overview-office-deployment-tool)
