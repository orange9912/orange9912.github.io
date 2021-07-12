---
title: sass
date: 2021-06-21 16:40:16
tags:
categories:
- 前端
---

最近在实习的时候开发使用到Sass，虽然之前从未使用过，不过看了下文档也比较快的上手了。使用之后真香，给我最大的感受就是写父子关系的选择器不再这么麻烦了，可以通过嵌套来选择，而且变量、混合器的引入也让我减少了重复写冗余代码的工作。总结一下开发的时候经常用到的几个特性

# 变量的使用和引入

使用变量的好处非常的大，假设我们有一个屏幕宽度，很多地方的样式都需要根据这个宽度来调整，如果用传统的css写法，那就只能每个地方都：

```css
width:1470px;
```

这样子不仅非常不优雅，最重要的是修改的时候非常不方便。[试想一下如果我们屏幕宽度变化了，岂不是要每一处都要重新修改？]{.pink}

但是如果我们使用Sass，将这个宽度定义为变量

```scss
$screenwidth:1470px;
.selected {
  width:$screenwidth;
}
```

这样子统一引用这个变量，不仅修改方便，而且来源也很容易看出来。

# 嵌套CSS

正常css中如果要对某个元素下的元素给样式，要这样：

```css
#content div h1{
  ...
}
#content div p{
  ...
}
#content div:hover{
  ...
}
```

但如果我们使用Sass，就可以减少很多写选择器的工作量

```scss
#content{
  div{
    h1{...}
    p{...}
    &:hover{...}
  }
}
```

其他选择器也是这样，简直是方便多了

# 混合器mixin

在开发过程中实际上有很多地方需要重用大段代码，变量是没法做到的，这个时候就要引入混合器了。

比如说，我们经常需要引入图片，这个时候，每次都要编写：

```css
.bg{
  background-image: url(url);
  background-repeat: no-repeat;
  background-position: center;
  background-size: 100% 100%;
}
```

其实是非常冗余的，我们可以把它抽出来

```scss
@mixin bg($url) {
  background-image: url($url);
  background-repeat: no-repeat;
  background-position: center;
  background-size: 100% 100%;
}
```

使用的时候:

```scss
.bg{
 @include bg(url);
}
```

