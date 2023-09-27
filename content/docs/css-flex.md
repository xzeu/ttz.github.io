---
title: "flex的六个属性"
date: 2022-11-23T23:26:56+08:00
lastmod: 2022-11-23T23:26:56+08:00
draft: false
categories:
  - web
tags:
  - css
  - web
  - flex
author: "xzeu"
toc: true
autoCollapseToc: true
contentCopyright: '<a rel="license noopener" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0 / 转载文章请保留链接。</a>'
mathjax: true
---
- [1. flex的6个常用属性](#1-flex的6个常用属性)
  - [1.1. 示例代码](#11-示例代码)
    - [1.1.1. html示例](#111-html示例)
    - [1.1.2. css示例代码](#112-css示例代码)
    - [1.1.3. flex样例效果](#113-flex样例效果)
  - [1.2. 基本概念](#12-基本概念)
    - [1.2.1. 说明](#121-说明)
  - [1.3. 全属性一览](#13-全属性一览)
    - [1.3.1. 作用于容器](#131-作用于容器)
  - [1.4. flex-direction属性](#14-flex-direction属性)
    - [1.4.1. row（默认）横向从左到右排列。](#141-row默认横向从左到右排列)
    - [1.4.2. row-reverse 顾名思义，从右向左横向排列，反向。](#142-row-reverse-顾名思义从右向左横向排列反向)
    - [1.4.3. column 纵向从上往下排列。](#143-column-纵向从上往下排列)
    - [1.4.4. column-reverse 纵向从下往上排列。](#144-column-reverse-纵向从下往上排列)
  - [1.5. flew-wrap属性](#15-flew-wrap属性)
  - [1.6. flex-flow属性](#16-flex-flow属性)
  - [1.7. justify-content](#17-justify-content)
  - [1.8. align-items](#18-align-items)
  - [1.9. flex属性](#19-flex属性)
  - [1.10. order属性](#110-order属性)
  - [1.11. place-content属性](#111-place-content属性)
  - [1.12. place-item属性](#112-place-item属性)
  - [1.13. place-self属性](#113-place-self属性)

<!--more-->

# 1. flex的6个常用属性

为了更好的学习与了解各属性的可视化效果，先制做了以下示例代码

## 1.1. 示例代码

### 1.1.1. html示例

```html
base.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <div class="item">item1</div>
        <div class="item">item2</div>
        <div class="item">item3</div>
    </div>
</body>
</html>
```

### 1.1.2. css示例代码

```css
style.css代码
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
.container {
  /* 将元素转为flex属性 */
  display: flex;
  height: 20em;
  border: 1px solid lightseagreen;
  justify-content: end;
}
.container > .item {
  width: 10em;
  padding: 2em;
  background-color: Azure;
  border: 1px dashed lightblue;
}
```

### 1.1.3. flex样例效果

![](../images/2022-11-23-23-32-45.png)

## 1.2. 基本概念

### 1.2.1. 说明

- 采用 flex 布局的元素，称为 flex容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 flex 项目（flex item），简称"项目"。
- 容器默认存在两根轴：水平的主轴和垂直的交叉轴。
  - 主轴又分主轴的开始位置和结束位置
  - 交叉轴同样也分开始位置和结束位置
- 项目默认沿着主轴排列
  ![](../images/2022-11-24-18-39-33.png)

| 需要理解的名词 | 解释                 |
| -------------- | -------------------- |
| main-axis      | 主轴（默认是水平的） |
| cross-axis     | 侧轴（默认是垂直的） |
| main-start     | 主轴开始的位置       |
| main-end       | 主轴结束的位置       |
| cross-start    | 侧轴开始的位置       |
| cross-end      | 侧轴结束的位置       |

## 1.3. 全属性一览

### 1.3.1. 作用于容器

作用于容器的属性一共6个

| 属性            | 值                                                                                               | 作用                                                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| flex-direction  | row（默认值）: 从左到右                                                                          | 决定主轴的方向，即子项目flex item的排列方向                                                                                |
|                 | row-reverse: 从右到左                                                                            |                                                                                                                            |
|                 | column: 从上到下                                                                                 |                                                                                                                            |
|                 | column-reverse：从下到上                                                                         |                                                                                                                            |
| flex-wrap       | nowrap （默认值）：不换行                                                                        | 默认情况下，项目都排在一条线（又称"轴线"）上。                                                                             |
|                 | wrap: 换行                                                                                       | flex-wrap属性定义，如果一条轴线排不下，如何换行                                                                            |
|                 | wrap-reverse: 换行，第一行在下方，颠倒下顺序                                                     |                                                                                                                            |
| flex-flow       |                                                                                                  | flex-direction、flex-wrap的简写方式                                                                                        |
| justify-content | flex-start（默认值）：左对齐                                                                     | 定义了子项目flex item在主轴上的对齐方式。                                                                                  |
|                 | flex-end：右对齐                                                                                 |                                                                                                                            |
|                 | center： 居中                                                                                    |                                                                                                                            |
|                 | space-between（空格在中间）：两端对齐，项目之间的间隔都相等。                                    |                                                                                                                            |
|                 | space-around（空格环绕）：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。 |                                                                                                                            |
| align-items     | flex-start：交叉轴的起点对齐                                                                     | 定义子项目flex item在交叉轴上的对齐方式。                                                                                  |
|                 | flex-end：交叉轴的终点对齐                                                                       |                                                                                                                            |
|                 | center：交叉轴的中点对齐                                                                         |                                                                                                                            |
|                 | baseline: 项目的第一行文字的基线对齐                                                             |                                                                                                                            |
|                 | stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。                          |                                                                                                                            |
| align-content   | flex-start：与交叉轴的起点对齐。                                                                 | 属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。多根轴线出现的场景是，flex item 很多，折成多行。即多轴 |
|                 | flex-end：与交叉轴的终点对齐。                                                                   |                                                                                                                            |
|                 | center：与交叉轴的中点对齐。                                                                     |                                                                                                                            |
|                 | space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。                                        |                                                                                                                            |
|                 | space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。           |                                                                                                                            |
|                 | stretch（默认值）：轴线占满整个交叉轴。                                                          |                                                                                                                            |

## 1.4. flex-direction属性

`flex-direction`属性决定主轴的⽅向。

`flex-direction` 属性指定了主轴线的方向，子元素默认是按照主轴线排列的，所以 `flex-direction` 也指定了弹性子元素在弹性容器中的排列方式， 有以下四个值：

- row ：（默认）横向从左到右排列。
- row-reverse ：顾名思义，从右向左横向排列，反向。
- column ： 纵向从上往下排列。
- column-reverse ： 纵向从下往上排列。

| 值             | 描述                               |
| -------------- | ---------------------------------- |
| row            | （默认）横向从左到右排列。         |
| row-reverse    | 顾名思义，从右向左横向排列，反向。 |
| column         | 纵向从上往下排列。                 |
| column-reverse | 纵向从下往上排列。                 |

### 1.4.1. row（默认）横向从左到右排列。

```css
flex-direction:row(默认值) | row-reverse | column | column-reverse;
//该属性指定了Flex的项目怎样在flex容器中排列，设置flex容器的主轴方向，它们(项目)两个主要的方向排列，就像一行一样水平排列或者像一列一样垂直排列。

.flex-container {
  -webkit-flex-direction: row; /* Safari */
  flex-direction:         row;
  }
```

![](../images/2022-11-25-11-06-29.png)

### 1.4.2. row-reverse 顾名思义，从右向左横向排列，反向。

```css
 .flex-container {
            flex-direction: row-reverse;
        }
```

![](../images/2022-11-25-11-07-09.png)

### 1.4.3. column 纵向从上往下排列。

```css
 .flex-container {
            flex-direction: coulmn;
        }
```

![](../images/2022-11-25-11-07-35.png)

### 1.4.4. column-reverse 纵向从下往上排列。

> 行排列 `flex-direction: row`，与上面效果相同。
>
> 列排列 `flex-direction: column`，效果如下

```css
 .flex-container {
            flex-direction: coulmn-reverse;
        }
```

![](../images/2022-11-25-11-13-18.png)

## 1.5. flew-wrap属性

`flex-wrap`属性设置⼦项⽬的换⾏⽅式。

弹性盒子默认情况下所有子元素都是排在一行上的，但容器大小有限，很有可能出现子元素被截断的情况，这时候就需要换行，`flex-wrap` 就是用来操作子元素换行的属性。`flex-wrap` 有以下三个值：

- nowrap ：（默认） 不换行。
- wrap ：换行，第一行排满后自动换到第二行。
- wrap-reverse ：换行，与 wrap 不同的是如果出现换行，换行后的元素在第一行上方。

| 值           | 描述                                                             |
| ------------ | ---------------------------------------------------------------- |
| nowrap       | ：（默认） 不换行。                                              |
| wrap         | ：换行，第一行排满后自动换到第二行。                             |
| wrap-reverse | ：换行，与 wrap 不同的是如果出现换行，换行后的元素在第一行上方。 |

![](../images/2022-11-23-23-37-16.png)

压缩宽度，产生换行，效果如下：

![](../images/2022-11-23-23-37-42.png)

## 1.6. flex-flow属性

flex-flow 是 flex-direction 和 flex-wrap 的简写形式，默认值为 row nowrap。

- `flex-flow: row nowrap;`
  flex-direction、flex-wrap的简写方式,实现与前述同样的功能。

## 1.7. justify-content

## 1.8. align-items

## 1.9. flex属性

flex:放大 缩小 宽度

- `flex：0 1 auto;` 0 不允许放大，1 允许缩小，auto宽度自动计算（宽度默认是根据内容变化，如果设置了宽度按设置算，如果设置了 `min-width`则按此计算，即 `min-width > flex：width > element：width）`。
- `flex：0 1 auto;`，即flex的默认值，也可以写作 `flex：initial`。

> 注：`flex：0`，由于只写了第一个参数，系统默认为 `flex 0,1，auto`，与上述效果一样。

效果查看，浏览器当前大小：
![](../images/2022-11-23-23-46-52.png)
缩小浏览器视窗，阴影消失，元素也被压缩：
![](../images/2022-11-23-23-47-05.png)
放大浏览器视窗，阴影扩大，元素不能被放大：
![](../images/2022-11-23-23-47-23.png)

`flex：1 1 auto`，允许放大，允许缩小，宽度自动计算，可以简写为flex：auto。
`flex：0 0 auto`，禁止放大缩小，简写为flex：none。

## 1.10. order属性

```css
.container .item:nth-of-type(1){
    background-color: coral;
    order:2;
}
```

改变第一个项目的背景色，调整他的顺序为2：
![](../images/2022-11-23-23-48-23.png)
把项目3排在项目2之前，

```css
.container .item:last-of-type{
    background-color: green;
    /* 项目2原来在最前顺序值为0，要排在项目2前，值应该小于0。 */
    order:-2;
}
```

![](../images/2022-11-23-23-48-46.png)

## 1.11. place-content属性

内容位置，表现为内容对齐，利用剩余空间的分配完成内容的对齐。

- place-content: start
  ![](../images/2022-11-23-23-40-35.png)
- place-content: end
  ![](../images/2022-11-23-23-41-00.png)
- place-content: center
  ![](../images/2022-11-23-23-41-33.png)
- place-content: space-between
  ![](../images/2022-11-23-23-42-03.png)
- place-content: space-around（中间空间是两边的两倍）
  ![](../images/2022-11-23-23-42-15.png)
- place-content: space-evenly（所有间距相等）
  ![](../images/2022-11-23-23-42-32.png)
  纵向排列一样，就不列举了。

## 1.12. place-item属性

- place-items: stretch

![](../images/2022-11-23-23-43-30.png)

- place-items: start

![](../images/2022-11-23-23-43-37.png)

- place-items: end
  ![](../images/2022-11-23-23-43-46.png)

## 1.13. place-self属性

控制某一个属性的对齐方式

> 例如：place-self: start

```css
.container .item:last-of-type{
background-color: green;
/ 项目2原来在最前顺序值为0，要排在项目2前，值应该小于0。 /
order:-2;
place-self: start;
}
```

![](../images/2022-11-23-23-49-50.png)
