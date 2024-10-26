---
title: H5组件库及迁移工具的沉淀
date: 2023-1-24 22:33:00
tags:
sticky: true
categories:
- 工作沉淀
---

方便迁移，使用jscodeshift编写codemod

时间线：**工作半年（2022.10-2023.1）**

# 背景

2022年底时，抖音直播前端（H5层面）的C端组件库设计规范是遵循旧的**直播设计规范**开发，但后来设计统一对齐成了抖音设计规范（DUX，Delight Design），因此原有的直播组件库已不太适用于业务场景。

而Delight Design当时存在安卓、iOS、lynx组件库，由于人力问题一直未启动H5组件库建设，双方一拍即合，共建实现 C 端 H5 组件库。

**于是我们就开始共建DUX H5组件库**

耗时不到三个月，大家齐心协力产出21个基础组件、文档站、对应demo，并产出一个原直播组件库的全新版本（90%+兼容，少数break change）&更新codemod工具。 

# 组件库大致技术

React18 + Typescript + lodash-es + gulp构建

## 好用mixins沉淀

```scss
@mixin text-ellipsis {
  work-break: break-all;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

// 细边框
@mixin thin-border(
	$width: 1PX,
  $color: var(--dux-LinePrimary), // 主颜色,
  $style: solid,
  $border-radius: 0
) {
  &::before {
    content: "";
    pointer-events: none;
    display: block;
    position: absolute;
    left: 0;
    top: 0;
    transform-origin: 0 0;
		border-width: $width;
    border-style: $style;
    border-color: $color;
  }    
  
  @media (min-resolution: 2dppx) {
    
    &::before {
      width: 200%;
      height: 200%;
      transform: scale(.5);
      border-radius: calc(2 * $border-radius);
    }
  }
  
    @media (min-resolution: 3dppx) {
    
    &::before {
      width: 300%;
      height: 300%;
      transform: scale(.333);
      border-radius: calc(3 * $border-radius);
    }
  }
}

// 元素实际触区
@mixin touch-area (
	$width: 100%, // 宽度
  $height: 44px, // 设计要求
) {
  & {
    position: relative;
  }    
  
  &::before {
    width: $width;
    height: $height;
    content: "";
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
}

@mixin active-area (
    $width: 100%,
  $height: 100%,
  $borderRadius: 4px,
  $color: var(--dux-ShadowInverse),
  $left: 0,
  $top: 0
) {
  & {
    position: relative;
  } 
  
  &:active::after {
    background: var(--dux-active-area-backgroundColor, $color);
  }
  
  &::after {
    width: $width;
    height: $height;
    content: "";
    position: absolute;
    background: var(--active-area-background, transparent);
    left: $left;
    top: $top;
  }
}

```

## 不同主题控制

sass编写中，注意多层颜色token变量，暴露一层给到外层统一主题覆盖。

优先级：组件特殊token>主题token>默认兜底色。

# 任务

时间紧，任务不轻的情况下，其中我负责了以下三个组件的编写：

- Tag（标签）
- Input（单行文本框）
- TextArea（多行文本框）

# 问题

不太回忆起来了，仅列举部分还记得的问题

## Tag里渲染文字上浮的问题

经过一顿搜索后，原作者：[(99+ 封私信 / 80 条消息) Android浏览器下line-height垂直居中为什么会偏离？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/39516424/answer/274374076)

简而言之就是因为字体内部渲染的问题，无法通过前端的技术去根本上修复

只能进行缓解：让行高=字号，多余的行高通过间距去控制（即减小字体的渲染范围来缓解）

## TextArea、Input内在拼音输入态时多次触发无意义的onChange

中文比较特殊，有拼音输入态。所以这里选择使用onCompositionEnd来判断是否处于拼音输入状态，并暴露参数给onChange回调。

# codemod工具

本次我们的H5组件库在旧有的基础上，产生了一些API的break change（更名或参数变更）。为了方便旧有组件库用户一键迁移，于是使用jscodeshift编写了一个codemod：**通过AST，找出需要替换的代码片段。**

像React-codemod就是利用的jscodeshift实现。

jscodeshift提供了许多的上层功能封装（自带runner可对文件目录自动递归处理），用于当AST editor较为省事。

编写过程中，可以配合https://astexplorer.net工具来辅助快速找出节点对应类型去写匹配代码。

同样，我也负责了Tag、Input、TextArea三个组件对应的`transformer`文件编写。

参考资料：[GitHub - facebook/jscodeshift: A JavaScript codemod toolkit.](https://github.com/facebook/jscodeshift)