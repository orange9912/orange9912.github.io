---
title: CSS选择器优先级笔记
date: 2021-06-24 15:18:14
tags:
categories:
- 前端
---

在css中，我们可以使用多种不同的方法选择元素。实际上，我们总会碰到同一个元素被选中多次的情况，而且选择器不尽相同，这个时候到底哪一个选择器选中的规则生效呢，就需要看选择器的优先级。

本文参考了我之前大二看的一本书，《CSS权威指南》。

# 各类选择器

css中的选择器实际上是非常多的，这里就说一些我平常用的比较多的选择器

- 类选择器，不用多说，这个基本上是我最常用的选择器之一
- ID选择器
- 属性选择器
- 通用选择器：*
- 文档结构的选择器，例如后代选择器、直接子代的选择器、相邻兄弟选择器等等
- 伪类、伪元素选择器，如hover、active等等

# 特指度

## 概念

特指度就是一个充当优先级的一个东西。它由四个数值部分组成：**0，0，0，0**

它是从左到右比较的，只要从左到右比较相同位时哪个大，就是哪个大，不管后面的位数。

如：0,1,0,0>0,0,13,0

## 规则

它计算的规则如下：

- 选择器中的每个ID属性值加0，1，0，0
- 选择器中的每个类选择器、属性选择或伪类加0，0，1，0
- 选择器中的每个元素和伪元素加0，0，0，1。(css2.1明确指出伪元素有特指度)
- 连接符和通用选择器不增加特指度。通用选择器特指度为0，0，0，0
- 行内样式的特指度为1，0，0，0

## 例子

计算下列各个选择器中的特指度

```css
h1{color:blue;}		//0,0,0,1
p em{...}					//0,0,0,2
.orange{...}			//0,0,1,0
*.bright{...}			//0,0,1,0
p.bright em.dark{...}//0,0,2,2
#id216{...}				//0,1,0,0
div#sidebar *[href]{...}//0,1,1,1
```

## 注意边界

ID选择符不等于选择id属性的属性选择符，一定要注意。

# 重要声明

前面说的比较前提都建立在属于非重要声明之中。

对于重要声明，可以在声明末尾分号之前插入!important，如：

```css
p.dark{color:#333 !important;background:white;}
```

带有！important的声明对特指度没有影响。

但是，[重要声明和非重要声明冲突时，重要声明始终胜出]{.pink}

# 继承值

css很多属性会发生继承，但是要注意，继承得来的值是==没有特指度的==。

:::primary

没有特指度不等于零特指度，要注意，零特指度是大于没有特指度的。

:::

# 当特指度相同的时候...

总有特指度相同的时候，这个时候还需要结合其他的规则进行判断。

1. 按权重和来源排序
2. 按特指度排序
3. 按前后位置排序

下面展开说。

## 按权重和来源排序

显式权重：看是否是重要声明，重要声明胜出

来源：行内样式>内部样式>外部样式

## 按特指度排序

这个没什么好说，前面已经说过了

## 按前后位置排序

如果显式权重、来源、特指度都相同，则样式表中位置靠后的规则胜出。

:::primary

来自不同样式表的，则某个样式表的位置相当于import（导入）中的位置

:::

