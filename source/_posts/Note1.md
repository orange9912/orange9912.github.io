---
title: 开发小知识笔记一
date: 2021-06-03 12:02:28
tags:
categories:
- 前端
---

# 移动端产生300ms点击延迟现象

可以禁止混用touch、click，或者增加一层透明遮罩

# 点击元素时禁止产生背景或边框

```css
-webkit-tap-highlight-color:rgba(0,0,0,0);
```

# 禁止用户选中文字

```css
-webkit-user-select:none;
user-select:none;
```

# 开启硬件加速

```css
transform:translate3d(0,0,0);
```

# CSS一行文本超出显示省略号

```css
overflow:hidden;
text-overflow:ellipsis;
white-space:nowrap;
```

# CSS多行文本超出显示省略号

```css
display:-webkit-box;
-webkit-box-orient:vertical;
-webkit-line-clamp:3;//行数，自由控制
overflow:hidden;
```

# 修改滚动条样式

查看div::-webkit-scrollbar相关样式

# CSS画三角形

```css
#triangle{
  width: 0;
  height: 0;
  border-top:50px solid blue;
  border-right:0px;
  border-left:0px;
  border-bottom:0px;
}
```

