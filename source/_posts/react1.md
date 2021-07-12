---
title: react开发笔记1
date: 2021-06-20 15:38:05
tags:
categories:
- 前端
---

记录一下实习中使用React开发需求时候遇到的一些困难和解决方案。

# 条件渲染

开发的时候经常遇到根据某个条件进行组件的渲染

比如说：有数据的时候，没有数据的时候，正在加载的时候，如果要写，用最简单入门的方式就是这么写：

```jsx
const list = ({list}) => {
  const isEmpty = !list && !list.length;
  return (
  	isEmpty ? null :
    <Component1/>
  );
}
```

如果要再加上正在加载、加载是否错误的时候

```jsx
const list = ({isLoading,list,error}) => {
  return (
  	<div>
    {
    	isLoading ? <Component1/> :
      (
      	isError ? <Component2/> :
        (
        	isEmpty ? <Component3/> : <Component4/>
        )
      )
    }
    </div>
  )
}
```

简直就是地狱好吧！又难看，嵌套的又深，简直和当初的回调地狱一模一样。因为JSX本身没法使用if...else，只能用三目运算符。

于是我就在查如何破解这种嵌套地狱，查到了两种常用的手段。

## 抽离出渲染函数

可以把上面的例子，将其抽离出一个渲染函数

```jsx
const getList = (isLoading,list,error) => {
  ...使用ifelse进行条件渲染返回
  return ...
}
  
const list = ({isLoading,list,error}) => {
  return (
  	<div>
    	{
        getList(isLoading,list,error)
      }
    </div>
  )
}
```

## 直接在内部使用一个IIFE（立即执行函数表达式）

```jsx
const list = ({isLoading,list,error}) => {
  return (
  	<div>
    	{
        (
        	() => {
            ...使用ifelse进行渲染返回
          }
        )()
      }
    </div>
  )
}
```

使用这两种手段的话就可以破解这个嵌套，而且可以使用console.log在控制台进行简单调试。

## 为什么不能在jsx中使用if..else？

我在React官方文档读的时候，其实JSX是会变编译成为React.createElement，而React.createElement无法运行JavaScript代码，只能渲染一个结果，所以JSX只能写Javascript表达式。或者说JSX只是函数调用和表达式的语法糖

# Render prop

官方文档中：render prop是一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术。

它一点都不能称之为简单，一开始我查资料的时候感觉说的都有点“抽象”，说了一大堆，其实在我的理解里，它应用的场景应该是这样的。

**一般我们一些组件都是需要父组件来提供一些prop，然后根据这些prop进行渲染，但是如果要这样的话，子组件就必须在父组件内部返回的JSX中作为children嵌入编码**

比如说我们有个组件获取鼠标的位置，然后有个组件需要根据鼠标的位置来做一系列行为。

正常来说会编码在父子嵌套，但是这样就没有办法复用，于是引进了render prop。

简单来说，render prop的模式就是用于告知组件需要渲染什么内容的函数prop，我们不在里面写死它的子组件，而是把数据传递过去

[把需要某一类数据的组件需要的数据传递过去]{.pink}

```jsx
<DataProvider render={data => (<h1>Hello {data.target}</h1>)}/>

//当然也可以不写在render里面
<DataProvider>
	{
    (data) => (<h1>Hello {data.target}</h1>)
  }
</DataProvider>
```

比如说，要写出一个可复用的、提供窗口宽度数据的组件

```jsx
class WindowWidth extends React.Component{
  constructor(){
    super()
    this.state = {
      width:0
    }
  }
  
  componentDidMount(){
		this.setState(
    	{
        width:window.innerWidth
      },
    );
    window.addEventListener('resize',({target}) => {
      this.setState({width:target.innerWidth});
    });
  }
  
  render(){
    return this.props.children(this.state.width);//向下传递，将数据传到子组件的prop中
  }
}


<WindowWidth>
	{
    width => (width > 1080 ? <div>...</div> : null)
  }
</WindowWidth>
```

# 高阶组件

高阶组件是一个相对比较复杂的概念，它用于实现组件逻辑的抽象和复用。

简单来说，它是一个以React组件为参数，并且返回一个新的react组件的函数。

前面说过，高阶组件也可以和Render prop模式搭配着用，因为每次都要在外面包一层数据提供组件，看起来不够简洁。

```javascript
function ComponentWithWidth(component){
  return class extends Component{
    render(){
      return (
      	<WindowWidth>
        {
          (width) => <Component width={width}/>
        }
        </WindowWidth>
      )
    }
  }
}
```

就可以减少冗余。

**高阶组件实际上有点像一个装饰器模式，它增强了组件的功能。**它应该是适用于不同类组件有相似行为的的场景，比如说都要从localStorage读取数据、拖拽排序，或者说它分离了组件，使组件的功能更加“专一”

比如说把与服务器通讯的逻辑抽离出来，在高阶组件内部获取，通过props的方式传递给组件。

```react
import React,{Component} form 'react';

function withPersistentData(WrappedComponent){
  return class extends Component{
    componentWillMount(){
      let data = localStorage.getItem('data');
      this.setState({data});
    }
    
    render(){
      return <WrappedComponent data={this.state.data} {...props}/>
    }
  }
}

class MyComponent extends Component{
  render(){
    return <div>{this.props.data}</div>
  }
}

const MyComponentWithPersistentData = withPersistentData(MyComponent);
```

## 使用场景

1. **操作props**

   在被包装组件接收props之前，高阶组件可以先拦截到props，进行一些处理后（比如增加、删除、修改），再将处理后的props传递下去。

2. **通过ref访问组件实例**

   通过ref获取被包装组件实例的引用，然后高阶组件就具备了直接操作被包装组件的属性或方法的能力。

   ```react
   function withRef(wrappedComponent){
     return class extends React.Component{
       constructor(props){
         super(props);
         this.someMethod = this.someMethod.bind(this);
       }
       someMethod(){
         this.wrappedInstance.someMethodInWrappedComponent();
       }
       render(){
         return <WrappedInstance ref={(instance) => {this.wrappedInstance = instance}} {...this.props}/>
       }
     }
   }
   ```

3. **组件状态提升**

   高阶组件可以把**被包装组件的状态及相应的状态处理方法**提升到高阶组件中，从而使得被包装组件的无状态化（更好维护）。

   ```javascript
   function withControlledState(WrappedComponent){
     return class extends React.Component{
       constructor(props){
   			super(props);
         this.state = {
           value : '',
         };
         this.handleValueChange = this.handleValueChange.bind(this);
       }
       handleValueChange(e){
         this.setState({
           value:e.target.value
         });
       }
       render(){
         const newProps = {
           controlledProps:{
             value:this.state.value,
             onChange: this.handleValueChange
           }
         };
         return <WrappedComponent {...this.props} {...newProps}/>
       }
     }
   }
   
   function InputArea(props){
   	return <input name="input" {...this.props.controlledProps}/>
   }
   const ComponentWithControlledState = withControlledState(InputArea);
   ```

4. **用其他元素包裹组件，搭配render props使用**

# 那Render prop和高阶组件的区别？

像上面提到的功能，我觉得高阶组件可以做的场景，很多时候Render prop也能达到。比如说传递一些数据之类的。但是我就想，它们之间有什么区别呢？于是我查了一些资料，是这么说的：

- HOC高阶组件的控制权在于Wrapper组件，即包裹的组件，**最后渲染结果由父组件决定。**
- render props则是控制反转（IOC），控制权在子组件，父组件不过是提供了一些数据，但渲染结果仍然由子组件决定。

# this.setState

在使用this.setState的时候，经常会遇到这么一些场景：

根据上一次的state计算出下一次的state的值。

但是在官方文档内：state的更新可能是异步的，而且可能会将多次调用合并成一个调用。

于是好奇的我就去查，想了解这个API的特性，于是查到它的官方描述是这样的：

[setState() does not always immediately update the component. It may batch or defer the update until later. This makes reading this.state right after calling setState() a potential pitfall.]{.pink}

:::primary

就是说，它并不是每次都立即更新，可能会延迟更新，也就是说在通过this.state读取内容时，有可能获取不到最新的值

:::

## 既然不是每次都立即更新，那什么时候延迟什么时候不延迟呢？

带着这个疑问我又去查了查资料，结果是

**在React控制之内的事件处理过程中，setState不会同步更新，而在控制之外的地方，会同步更新。**

:::primary

React控制之内，即是给元素使用onClick={}这种方式的事件。

React控制之外，即是用如addEventListener这种API给元素添加事件监听

:::

## 那如果要根据上一状态计算出下一状态该怎么做？

这样的话，只要给setState参数传递一个函数而不是一个对象就行了，

```jsx
this.setState((state,props) => {  return {    data:state.data + props.data  };});
```

# react事件

React的事件并不是原生的，它是合成的，使用的时候有几点需要注意：

- 不能异步访问合成事件对象
- 不能阻止原生事件冒泡

