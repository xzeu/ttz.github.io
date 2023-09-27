---
title: "数据建模方法论及实施步骤"
date: 2023-08-18T15:42:30+08:00
lastmod: 2023-08-18T15:42:30+08:00
draft: false
keywords: [Logical，Conceptual，Physical]
description: ""
tags: [数据建模,datamodel,model,Conceptual]
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
- [1. 概要：数据建模简介](#1-概要数据建模简介)
  - [1.1. 概念数据模型（Conceptual Data Model）](#11-概念数据模型conceptual-data-model)
  - [1.2. 逻辑数据模型（Logical Data Model）](#12-逻辑数据模型logical-data-model)
  - [1.3. 物理数据模型（Physical Data Model）](#13-物理数据模型physical-data-model)
- [2. 方法：数据建模常用模型](#2-方法数据建模常用模型)
  - [2.1. D-R模型](#21-d-r模型)
  - [2.2. 多维模型](#22-多维模型)
  - [2.3. 星型模型](#23-星型模型)
  - [2.4. 雪花模型](#24-雪花模型)
- [3. 方案：数据建模六步骤](#3-方案数据建模六步骤)
  - [3.1. 收集业务需求与数据实现](#31-收集业务需求与数据实现)
  - [3.2. 选择业务过程](#32-选择业务过程)
  - [3.3. 声明粒度](#33-声明粒度)
  - [3.4. 确认维度](#34-确认维度)
  - [3.5. 确认事实](#35-确认事实)
  - [3.6. 部署方式](#36-部署方式)
<!--more-->
<!-- more -->

了解数据建模之前首先要知道的是什么是数据模型。数据模型（Data Model）是数据特征的抽象，它从抽象层次上描述了系统的静态特征、动态行为和约束条件，为数据库系统的信息表示与操作提供一个抽象的框架。

## 1. 概要：数据建模简介

数据基本用于两种目的：

1. 操作型记录的保存
2. 分析型决策的制定。

简单地说就是操作型系统保存数据，分析型系统使用数据；前者反映数据的最新状态，后者反映数据一段时间的状态变化。操作型系统简称为OLTP（On-Line Transaction Processing）联机事务处理，分析型系统简称为OLAP（On-Line Analytical Processing）联机分析处理。

- 在OLTP场景中，常用的是使用实体关系模型（ER）来存储，从而在事务处理中解决数据的冗余和一致性问题。
- 在OLAP场景中，有多种建模方式有：ER模型、星型模型和多维模型。

数据建模是一种用于定义和分析数据的要求和其需要的相应支持的信息系统的过程。从需求到实际的数据库，有三种不同的类型。用于信息系统的数据模型作为一个概念数据模型，本质上是一组记录数据要求的最初的规范技术。数据首先用于讨论适合企业的最初要求，然后被转变为一个逻辑数据模型，该模型可以在数据库中的数据结构概念模型中实现。一个概念数据模型的实现可能需要多个逻辑数据模型。数据建模中的最后一步是确定逻辑数据模型到物理数据模型中到对数据访问性能和存储的具体要求。数据建模定义的不只是数据元素，也包括它们的结构和它们之间的关系。

### 1.1. 概念数据模型（Conceptual Data Model）

简称概念模型 ，主要用来描述世界的概念化结构。概念数据模型是最终用户对数据存储的看法，反映了最终用户综合性的信息需求，它以数据类的方式描述企业级的数据需求，数据类代表了在业务环境中自然聚集成的几个主要类别数据。概念数据模型的目标是统一业务概念，作为业务人员和技术人员之间沟通的桥梁，确定不同实体之间的最高层次的关系。

### 1.2. 逻辑数据模型（Logical Data Model）

简称数据模型，这是用户从数据库所看到的模型，是具体的DBMS所支持的数据模型，如网状数据模型(Network Data Model)、 层次数据模型 (Hierarchical Data Model)等等。 此模型既要面向用户，又要面向系统 ，主要用于 数据库管理系统 （DBMS）的实现。逻辑数据模型的内容包括所有的实体和关系，确定每个实体的属性，定义每个实体的主键，指定实体的外键，需要进行范式化处理。逻辑数据模型的目标是尽可能详细的描述数据，但并不考虑数据在物理上如何来实现。逻辑数据建模不仅会影响数据库设计的方向，还间接影响最终数据库的性能和管理。

### 1.3. 物理数据模型（Physical Data Model）

简称物理模型 ，是面向计算机物理表示的模型，描述了数据在储存介质上的组织结构，它不但与具体的DBMS 有关，而且还与操作系统和硬件有关。每一种逻辑数据模型在实现时都有起对应的物理数据模型。DBMS为了保证其独立性与可移植性，大部分物理数据模型的实 现工作又系统自动完成，而设计者只设计索引、聚集等特殊结构。物理结构图显示物理数据模型是在逻辑数据模型的基础上，考虑各种具体的技术实现因素，进行数据库体系结构设计，真正实现数据在数据库中的存放。

## 2. 方法：数据建模常用模型

### 2.1. D-R模型

D-R模型（Entity-relationship model）实体关系模型，E-R模型的构成成分是实体集、属性和联系集。其表示方法如下：  
（1） 实体集用矩形框表示，矩形框内写上实体名。  
（2） 实体的属性用椭圆框表示，框内写上属性名，并用无向边与其实体集相连。  
（3） 实体间的联系用菱形框表示，联系以适当的含义命名，名字写在菱形框中，用无向连线将参加联系的实体矩形框分别与菱形框相连，并在连线上标明联系的类型，即1—1、1—N或M—N。如图1-1所示。

![Alt text](../images/dataModel.png)

### 2.2. 多维模型

它是维度模型的另一种实现。当数据被加载到OLAP多维数据库时，对这些数据的存储的索引，采用了为维度数据涉及的格式和技术。性能聚集或预计算汇总表通常由多维数据库引擎建立并管理。由于采用预计算、索引策略和其他优化方法，多维数据库可实现高性能查询。这种模型可以以星型模式，雪花模式，或事实星座模式的形式存在。

### 2.3. 星型模型

它是维度模型在关系型数据库上的一种实现。它是多维的数据关系，它由事实表（Fact Table）和维表（Dimension Table）组成。每个维表中都会有一个维作为主键，所有这些维的主键结合成事实表的主键。事实表的非主键属性称为事实，它们一般都是数值或其他可以进行计算的数据。该模型表示每个业务过程包含事实表，事实表存储事件的数值化度量，围绕事实表的多个维度表，维度表包含事件发生时实际存在的文本环境。这种类似于星状的结构通常称为'星型连接'。其重点关注用户如何更快速地完成需求分析，同时具有较好的大规模复杂查询的响应性能。如图1-2所示。
![图1-2](../images/startModel.png)  
图1-2

### 2.4. 雪花模型

它是当有一个或多个维表没有直接连接到事实表上，而是通过其他维表连接到事实表上时，其图解就像多个雪花连接在一起，故称雪花模型。雪花模型是对星型模型的扩展。如图1-3所示。

![图1-2](../images/snowModel.png)  
图1-3

## 3. 方案：数据建模六步骤

数据建模，通俗地说，就是通过建立数据科学模型的手段解决现实问题的过程。数据建模也可以称为数据科学项目的过程，并且这个过程是周期性循环的。具体可分为六大步骤，如图2-1所示。

![图2-1](../images/stepAndStep.png)  
图2-1

### 3.1. 收集业务需求与数据实现

在开始维度建模工作之前，需要理解业务需求，以及作为底层源数据的实际情况。通过与业务方沟通交流、查看现有报表等来发现需求，用于理解他们的基于关键性能指标、竞争性商业问题、决策制定过程、支持分析需求的目标。同时，数据实际情况可通过与数据库系统专家交流，了解访问数据可行性等。

### 3.2. 选择业务过程

业务过程是组织完成的操作型活动。业务过程时间建立或获取性能度量，并转换为事实表中的事实。多数事实表关注某一业务过程的结果。过程的选择非常重要的，因为过程定义了特定的设计目标以及对粒度、维度、事实的定义。

### 3.3. 声明粒度

声明粒度是维度设计的重要步骤。粒度用于确定某一事实表中的行表示什么。在选择维度或事实前必须声明粒度，因为每个候选维度或事实必须与定义的粒度保持一致。在从给定的业务过程获取数据时，原子粒度是最低级别的粒度。强烈建议从关注原子级别粒度数据开始设计，因为原子粒度数据能够承受无法预期的用户查询。

### 3.4. 确认维度

维度提供围绕某一业务过程事件所涉及的'谁、什么、何处、何时、为什么、如何'等背景。维度表包含分析应用所需要的用于过滤及分类事实的描述性属性。牢牢掌握事实表的粒度，就能够将所有可能存在的维度区分开来。

### 3.5. 确认事实

事实，涉及来自业务过程事件的度量，基本上都是以数据值表示。一个事实表行与按照事实表粒度描述的度量事件之间存在一对一关系，因此事实表对应一个物理可观察的事件。在事实表内，所有事实只允许与声明的粒度保持一致。

### 3.6. 部署方式

选择一种维度模型的落地方式。既可以选择星型模型，部署在关系数据库上，通过事实表及通过主外键关联的维度表；也可以选择多维模型，落地于多维数据库中。