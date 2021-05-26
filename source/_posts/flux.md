---
title: flux
date: 2021-05-26 14:36:07
tags:
categories:
- JavaScript
---

# 前言

最近在实习做项目的时候，使用的是react，发现项目的数据管理使用的架构是flux。虽然我很久之前也看过一下flux的架构思想，不过过了这么久又没有应用，早就忘光了，趁着这次机会，再来复习一下，顺便也记录下以前对redux的结构的思考。

# Flux是什么？

![flux](flux.png)

flux是一种架构思想。在flux里，一个应用分为四个部分，分别是：

```text
View:视图层
Action：动作，视图层发送的信息，比如用户点击了某个按钮。
Dispatcher：发布者、派发器，用来接收action并执行回调函数。
Store：数据层，用来存放应用的状态，一旦发生变动，就提醒Views要更新页面
```

![struct](struct.png "图摘自阮一峰老师的博客‘Flux架构入门教程’")

看到这个图，很明显，Flux的特点是==数据的单项流动==。

## View（视图）

视图是用来展现数据的，当视图上某些事件发生的时候，他就会调用一个方法，向Dispatcher发出一个Action，告诉他是什么类型的变化。

## Action

每一个Action都是一个对象，它通常包含动作的类型、数据属性。

## Dispatcher

它的作用是将Action派发到Store中。

## Store

它保存着整个应用的状态，所有数据都存放在那里

:::default

一般来说，View从Store处获得数据并渲染，用户从View进行操作、触发一个Action发送到Dispatcher中进行相应的更改计算，再更改Store，并且将这个变化“告知”View，再重新渲染，大概是这么一个过程。

:::

# Redux

Redux是2015年出现的，它将Flux和函数式编程相结合。

## 应用场景

用户使用方式复杂、不同身份用户有不同的使用方式（比如普通用户和管理员），或者使用了websocket。

它是为了解决react中组件通信问题的。

## 设计思想

Redux的设计思想其实很简单，两句话就能总结下来

++应用是一个状态的集合，状态和视图一一对应++

++所有的状态保存在一个对象里面++

## Redux的基本概念

### Store

​	保存数据的地方，整个应用只有一个

​	使用createStore生成

```javascript
import {createStore} form 'redux';
const store = createStore(fn);//参数为一个函数（这个函数为Reducer），返回值为一个store对象
```

### State

​	store对象包含所有数据，Store的快照——某个时点的数据，就叫做State

​	通过store.getState()获取

​	一个State对应一个View。

### Action

State改变时，View会发出一个通知——就是Action

### Action Creator

用于批量生成Action

### Store.dispatch()

View发出Action的唯一方法

该方法参数为一个对象或一个Action Creator

它其实做了两件事，将传入的action传递给reducer方法，计算出新的state、并逐一执行订阅了store的函数。

### Reducer

计算新State的过程——Reducer，一个函数，参数为Action与当前State，返回一个新State。

store.dispatch会触发reducer的自动执行。该函数在生成store的时候传入createStore方法。

==它是一个纯函数，不得改写参数，同样的输入必定得到同样的输出==

### store.subscribe()

store允许使用store.subscribe方法设置监听函数，一旦State发生变化，就自动执行该函数。

参数为一个函数（++View的更新函数，一般对于react项目就是组件的render或者setState方法++）

Store.subscribe方法返回一个函数，调用这个返回的函数可以解除监听

## Store的实现

```javascript
const createStore = (reducer) =>{
	let state;//某一时刻数据的状态
	let listeners = [];//监听数据者
	
	const getState = () => state;
	const dispatch = (action) => {
		state = reducer(state,action);
		listeners.forEach(listener => listener());//一个个执行更新函数
	};
	
	const subscribe = (listener) => {
		listeners.push(listener);
		return () => {
			listeners = listeners.filter(l => l!==listener);//执行函数将监听者数组去除这个listener
		}
	};
	dispatch({});//第一次先初始化一下
	return { getState, dispatch, subscribe };
};
```

## Redux工作流程的梳理总结

1. 先通过createStore(fn)生成一个store。fn为reducer，通过action和state计算新的state
2. 通过getState（）获取Store某一时刻的State
3. 通过State渲染View，一个state对应一个view
4. View通过store.dispatch来发出一个action（通知更新store）
5. 更新后循环234

