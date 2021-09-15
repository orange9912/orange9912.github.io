---
title: 简单的比较vue、react思想
date: 2021-06-27 19:17:11
tags:
categories:
- 前端
---

学习了React之后，我不自觉地会将vue和react相对比，而且也在思考使用框架的同时我学习的开发思想。

# 数据绑定

他们都是响应式的框架，但他们“实现响应式“的方法有所不同。

## Vue

vue在数据绑定上采用了双向绑定的策略。在vue2之中，采用了Object.defineProperty（Vue3使用了Proxy进行重写）来递归的将每个实例的数据转化成getter/setter，从而达到一个数据拦截的目的，加上发布订阅模式，视图感知到依赖的变化就会自动刷新，从而达到监听数据。

再对事件的监听，来修改数据，达到两方一致的效果。

## React

React之中并没有数据和视图的双向绑定，而是采用的**[局部刷新]{.pink}**。当数据发生变化，直接重新渲染组件，就可以得到最新的视图。

# 表单输入绑定

在React中有个概念为受控组件，Vue中有个v-model。学习到后来的时候就发现，其实这两个的思想有点相似。

受控组件的意思是将表单输入控件的数据源绑定为组件的state，然后监听更改事件，触发事件时修改数据源，达到一个受控的样子。

如：

```react
function Component(){
  const [name,setName]  = useState('');
  return (
  	<input type="text" value={name} onChange={(e) => setName(e.target.value)}></input>
  )
}
```

而vue中的v-model其实是个语法糖，其实就是相当于实现了像上面代码一样的效果。

# 组件化、单向数据流

每个vue组件本质上就是一个vue实例，react组件可以是函数、也可以是类。他们都有生命周期方法。

组件间通信他们也有点类似，都是提倡的单向数据流。

- 父->子，都是通过props向下传递数据。
- 子->父，vue中利用子组件触发events、父组件监听events来达到向上发送信息（携带数据）给父组件。React中是利用父组件通过props传给子组件的回调来进行传递数据。

# 状态管理

React中比较经典的比如redux，Vue中就是vuex，思想上也有一些类似的地方。比如vuex不允许组件直接修改state，而是需要通过触发Action来提交一个mutation从而更改数据。

在vuex中数据是可变的，但redux中提倡数据不可变性，Redux每次reducer计算出新的值后都会生成新的state代替旧的state

# 渲染、更新

在vue中，组件的依赖在渲染过程中自动追踪，系统能够精确知道那些组件需要被重渲染。

但React中某个组件状态发生变化后，react会以该组件为根，重新渲染整个组件子树，虽然过程中可以利用pure Component或should ComponentUpdate来避免不必要的渲染，但仍然没有Vue精确。