---
title: react Hooks简单笔记
date: 2021-06-25 16:41:29
tags:
categories:
- 前端
---

在实习的时候参与项目需求编写的时候就是使用React的函数式组件+Hooks进行编写，来总结一下我常常用到的hooks来进行进一步印象的加深。

在此之前，首先要说说React的组件发展史。

# createClass创建组件时期

在最开始的时候，React提供了一个API——createClass来创建组件。

它是一个函数，接收参数并返回组件实例。

# ES class声明组件时期

ES6出现之后，出现了class这一个语法糖。

然后就是现在的class组件。

现在的class组件随着发展变得越来越复杂，比如

- 生命周期编程。如果逻辑变得越来越复杂，组件的生命周期就会变得特别难以理解和维护。
- 本身组件写法的复杂，即使是完成一个非常简单的功能，都要写上constructor、render、还有一些功能函数。

# 函数式组件

函数式组件的写法相比class组件就简洁的多了，它只需要接收参数，根据参数渲染UI（返回值就是JSX），非常的简洁。

但是函数本身是无状态的（**所以以前的函数式组件又叫无状态组件**），为了引入状态和类似生命周期的东西，于是便有了hook

# class组件和函数式组件的比较

写一个计数器的组件，如果使用class组件应该是这么写：

```react
class counter extends React.Component{
  constructor(props){
    super(props);
    this.state = {
      count : 0
    };
  },
  render(){
    return (
    	<div>
      	<p>you had clicked {this.state.count} times</p>
        <button onClick={() => this.setState((state) => {count : state.count + 1} )}>click me</button>
      </div>
    );
  }
}
```

如果使用函数式组件+hook，就是这么写

```react
import React,{useState} from 'React';
function counter(){
  const [count,setCount] = useState(0);
  return (
  	<div>
    	<p>you had clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>CLick me</button>
    </div>
  );
}
```

现在还没有引入生命周期和更加复杂的逻辑，所以看起来差不多，但实际上随着复杂度的增加，他们的差别会越来越明显。

# Hook

==Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数==。

平时写代码时最常用的hook就是useState、useEffect，下面细说。

## useState

**如果我们需要向函数式组件中引入状态，就需要使用useState。**

```react
import React,{useState} from 'React';
function counter(){
  const [count,setCount] = useState(0);//此处定义了一个count状态
  return (
  	<div>
    	<p>you had clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>CLick me</button>
    </div>
  );
}
```

useState接收一个参数，这个参数为这个状态的初始值，返回一个数组，数组的第一项为状态本身，第二项为修改状态的方法。

useState只会在组件首次渲染的时候创建，后面都只会直接返回状态。

useState除了定义基本类型的状态之外，也可以很好的存放对象和数组的状态，尽管分离开来可能会更好，但如果有时候实在是需要存放对象的话，就要这样使用：

```react
function Box(){
  const [state,setState] = useState({left:0,top:0,width:100,height:100});
  ...
  //修改时这样修改
  setState(state => ({...state,left:100,....}));//把旧值传入，然后新值如此传入
}
```

## useEffect

**如果我们需要在函数中执行副作用（例如获取数据、手动更改DOM），就需要使用useEffect**。

```react
useEffect(一个副作用函数，依赖项);//当依赖项中任一个改变的时候，函数就会执行一次
useEffect(() => {
  //发起Ajax请求。
},[]);//如果依赖项设置为空，那么它只会执行一次。
```

## useContext

共享状态钩子，用于组件之间共享状态。

在组件外部建立一个Context（利用React的API）

然后在jsx中用AppContext.Provider元素包裹需要共享状态的组件（value赋值）

**这个元素会提供一个Context对象，这个对象可以被子组件共享**

在子组件中使用useContext引入Context对象（与useState相似，也利用解构赋值获取）

```react
const AppContext = React.createContext({});
//模版中：
<AppContext.Provider value={{username:'xxx'}}>
	<Navbar/>
  <Messages></Messages>
</AppContext.Provider>

//子组件中，如Navbar中
const {username} = useContext(AppContext);
```

# useCallback

```react
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

它返回一个memoized回调函数。它只在依赖项更新的时候才会更新，可以避免不必要的重新渲染。

## 自定义hook

自定义hook主要是利用在提取需要共享的逻辑，这部分还没怎么用过，先挖个坑以后补。

