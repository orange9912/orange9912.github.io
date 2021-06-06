---
title: CSS布局知识笔记1
date: 2021-06-04 18:18:32
tags:
categories:
- 前端
---

之前学习css还有开发的时候其实接触过很多知识点，只是一直没有时间去进行一个总结整合，今天特地来记录一下。

# BFC模式及应用

BFC，全称Block Formatting Context，块级格式化上下文。

按我的理解，一旦某个元素开启这个模式，它内部就会有自己的一套渲染规则，**它内部的元素不会影响到外部其他元素的渲染**

## 形成BFC模式的条件

- float的值不为none
- position的值为absolute或fixed
- overflow的值不为visible
- display的值为inline-block或table-cell或table-caption
- 根元素

## BFC模式的渲染规则

- 每个box在水平方向上的左边缘和BFC的左边缘相对齐，就算存在浮动。
- 内部的box将会独占宽度
- ==BFC区域不会和浮动元素重叠==
- BFC区域是一个独立的渲染容器，内部不干扰外部的渲染
- 浮动元素的高度也参与BFC的高度计算

## BFC模式的应用场景

### 实现自适应布局

用于实现一列/n列定宽，最右边那一列自适应。

将左边的列向左浮动，浮动会导致从文档流中删除，使得最右边的一列和左边的列重叠，这时对最右边一列启用BFC模式，就可以实现这个布局。

```html
<style>
    body{
        width: 600px;
        position: relative;
    }
    .left{
        width:80px;
        height:200px;
        float: left;
        background-color: blue;
    }
    .right{
        height:200px;
        background-color: red;
        overflow: hidden;
    }
</style>
<body>
    <div class="left"></div>
    <div class="right"></div>
</body>
```

![BFC1](BFC1.png "上图代码的效果")

### 解决高度塌陷问题

浮动会导致元素从文档流之中删除，从而无法撑起父元素的高度。

**但浮动元素也参与BFC的高度计算，利用这一点我们可以简单的解决高度塌陷问题**

```html
<style>
    .root{
        border: 5px solid blue;
        width:300px;
    }
    .child{
        border:5px solid red;
        width:100px;
        height:100px;
        float: left;
    }
</style>
<body>
    <div class="root">
        <div class="child child1"></div>
        <div class="child child2"></div>
    </div>
</body>
```

![BFC2](BFC2.png "高度塌陷")

这个时候，我们在父元素中添加

```css
.root{
  overflow: hidden;
}
```

![BFC3](BFC3.png "利用BFC解决高度塌陷")

### 解决边距折叠问题

正常来说，同一个BFC的两个相邻box的margin会出现折叠现象。

只要让两个相邻元素不在同属于一个BFC，就可以解决边距折叠问题。

# 多种方式实现居中

## 已知居中元素宽高

### 绝对定位+负margin

- 父元素设置定位relative（因为绝对定位相对于最近设置了定位的父元素）
- 居中元素设置绝对定位，top和left各50%，左和上margin设置负值（宽度的一半）

### 绝对定位+自动margin

- 父元素设置定位relative
- 居中元素设置绝对定位，上下左右偏移全部设置为0，且设置margin：auto。

## 未知居中元素宽高

其实这种相对来说更通用，用的更多。

### 绝对定位+transform

- 父元素设置定位relative
- 居中元素设置绝对定位，top和left各50%，且使用transform使自身偏移回宽高的一半。

### 设置为行内元素+text-align+vertical-align

- 父元素设置text-align为center
- 子元素设置display为inline-block，vertical-align为middle

### table-cell

```css
.parent{
  display:table-cell;
  text-align:center;
  vertical-align:middle;
}
.son{
  display:inline-block;
}
```

### flex

我最喜欢的flex布局，省事代码也少。

```css
.parent{
  display:flex;
  justify-content:center;
  align-items:center;
}
```

# 两列布局

以左边定宽右边自适应为例

## float+margin

```html
<style>
    .left{
        width:200px;
        height:200px;
        background-color: blue;
        float: left;
    }
    .right{
        height:200px;
        margin-left:200px;
        background-color: red;
    }
</style>
<body>
    <div class="left"></div>
    <div class="right"></div>
</body>
```

## float+overflow

```html
<style>
    .left{
        width:200px;
        height:200px;
        background-color: blue;
        float: left;
    }
    .right{
        height:200px;
        overflow: hidden;
        background-color: red;
    }
</style>
<body>
    <div class="left"></div>
    <div class="right"></div>
</body>
```

## flex

```html
<style>
    .root{
        display: flex;
    }
    .left{
        width:200px;
        height:200px;
        background-color: blue;
        float: left;
    }
    .right{
        height:200px;
        flex-grow: 1;
        background-color: red;
    }
</style>
<body>
    <div class="root">
        <div class="left"></div>
        <div class="right"></div>
    </div>
</body>
```

# 三列布局

## 左边两列定宽右边自适应

### float+margin

实现方法类似两列布局，多出来一列定宽的那列css样式和上面左边列一样

### float+overflow

同上

### flex

同上，类似

## 两侧定宽中间自适应

### float+margin

```html
<style>
    .root{
        /* display: flex; */
    }
    .left{
        width:200px;
        height:200px;
        background-color: blue;
        float: left;
    }
    .center{
        height:200px;
        margin:0px 200px;
        background-color: aquamarine;
    }
    .right{
        height:200px;
        width: 200px;
        float: right;
        background-color: red;
    }
</style>
<body>
    <div class="root">
        <div class="left"></div>
        <div class="right"></div>
        <div class="center"></div>
    </div>
</body>
```

:::primary

注意，在html内center反而放后面，因为块级元素都是会独立占据一行的，如果放在前面，会使right列被顶下去。

:::

### float+overflow

只需将上面center列的css中margin属性替换成overflow：hidden即可

### flex

```html
<style>
    .root{
        display: flex;
    }
    .left{
        width:200px;
        height:200px;
        background-color: blue;
        float: left;
    }
    .center{
        height:200px;
        flex-grow: 1;
        background-color: aquamarine;
    }
    .right{
        height:200px;
        width: 200px;
        float: right;
        background-color: red;
    }
</style>
<body>
    <div class="root">
        <div class="left"></div>
        <div class="center"></div>
        <div class="right"></div>
    </div>
</body>
```

