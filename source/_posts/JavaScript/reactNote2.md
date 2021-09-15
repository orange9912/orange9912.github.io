---
title: React笔记2
date: 2021-07-01 20:05:17
tags:
categories:
- 前端
---

读《react进阶》时做的一些内容的笔记

# 组件生命周期

[类组件才有生命周期，函数组件是没有的！]{.pink}

## 挂载阶段

1. constructor。构造方法，一般用于初始化组件的state和绑定事件处理方法等。
2. componentWillMount。
3. render。必要方法，返回UI。他必须是个纯函数
4. componentDidMount。挂载后调用，一般在这个时候向服务端请求数据。在这里调用this.setState会引起组件重新渲染

## 更新阶段

1. componentWillReceiveProps（nextProps）。只在props引起的组件更新过程中才会被调用。	
2. should ComponentUpdate(nextProps,nextState)。决定组件是否继续执行更新过程，可以通过比较两个参数来决定返回值。用于减少组件不必要的渲染，来优化性能。
3. componentWillUpdate
4. componentDidUpdate(prevProps,prevState)。组件更新后被调用。

## 卸载阶段

1. componentWillUnmount。在组件被卸载前调用。用于执行一些清理操作，比如清除定时器，清除componentDidMount中手动创建的DOM

# React事件处理函数this问题

ES6class语法实质上是ES5组合继承的语法糖，这我们都知道。

在ES6class中，并不会为方法自动绑定this到当前对象，为了解决这个this指向问题，有几种解决方案。

## 使用箭头函数

我们都知道箭头函数的this是定义时绑定，它会拿外层中最靠近的this来套给自己。

## 使用组件方法

直接在事件监听中写this.handle

同时需要在构造函数中使用bind来绑定

```javascript
this.handle = this.handle.bind(this);
```

# React的ref

ref用于获取元素，他可以获取DOM元素，甚至也可获取React组件实例。

可以用来**控制元素的焦点、文本选择**等操作。

但是ref破坏了以props为数据传递介质的典型数据流，**应尽量避免使用**

## 在DOM元素上使用ref

在DOM元素上使用ref是最常见的使用场景。ref接收一个回调函数作为值，同时向这个回调函数中传入DOM元素。

在组件被挂载时，回调函数会被调用，同时接收当前DOM元素作为参数。

在组件被卸载时，会接受null作为参数。

```react
class AutoFocusTextInput extends React.Component {
  componentDidMount(){
    this.textInput.focus();
  }
  render(){
    return (
    	<div>
      	<input
          type="text"
          ref={(input) => {this.textInput = input;}}/>
      </div>
    )
  }
}
```

这个组件为input元素定义了ref，在组件挂载后，通过ref获取到这个input元素并存在textInput上，让input自动获取焦点。

## 在组件上使用ref

此时ref的回调函数接收的参数时当前组件的实例，提供了一种在组件外部操作组件的方式。

可以通过ref获取到组件的实例，并调用组件内部的方法。

先定义个内部组件

```react
class AutoFocusTextInput extends React.Component {
  constructor(props){
    super(props);
    this.blur = this.blur.bind(this);
  }
  componentDidMount(){
    this.textInput.focus();
  }
  blur(){
    this.textInput.blur();
  }
  render(){
    return (
    	<div>
      	<input
          type="text"
          ref={(input) => {this.textInput = input;}}/>
      </div>
    )
  }
}
```

然后定义一个组件

```react
class Container extends React.Component {
  constructor(props){
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick(){
    this.inputInstance.blur();
  }
  render(){
    return (
    	<div>
      	<AutoFocusTextInput ref={ (input) => {this.inputInstance = input}}/>
        <button onClick={this.handleClick}>失去焦点</button>
      </div>
    )
  }
}
```

:::warning

只能为类组件定义ref属性，不能为函数组件定义ref，因为函数组件没有实例。

:::

但可以在函数组件内部使用ref（利用useRef），只要它指向一个DOM元素或者class组件。

```react
function CustomTextInput(props) {
  // 这里必须声明 textInput，这样 ref 才可以引用它
  const textInput = useRef(null);

  function handleClick() {
    textInput.current.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={textInput} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );
}
```

:::primary

useRef返回一个可变的ref对象，其.current属性被初始化为传入的参数。返回的ref对象在组件的整个生命周期内保持不变

本质上，`useRef` 就像是可以在其 `.current` 属性中保存一个可变值的“盒子”。

当ref对象内容发生变化时，useRef不会发出通知。

:::

## 父组件访问子组件的DOM节点

其实和子组件修改父组件的数据有点相似

- 都是父组件传一个prop回调函数给子组件
- 然后子组件在内部调用这个函数，将值通过调用这个函数传给父组件，从而达到访问的目的

```react
function Children(props){
  return (
  	<div>
    	<input ref={props.inputRef}/>
    </div>
  )
}

class Parent extends React.Component{
  render(){
    return (
    	<Children 
        inputRef={el => this.inputElement = el}
        />
    );
  }
}
```

# 虚拟DOM

## 概念

虚拟DOM与真实DOM对应，就是使用JavaScript对象来表示DOM元素的一种技术。

在浏览器中，操作真实DOM是效率会非常慢，每次操作都可能引起浏览器的回流和重绘，所以要尽量减少。

## DIff算法

每次组件的状态或属性更新，组件的render方法都会返回一个新的虚拟DOM对象。

React会通过比较更新前后两份虚拟DOM（新的虚拟DOM和旧的虚拟DOM），来找出差异部分更新到真实DOM上，从而减少在真实DOM 上的操作。

**正常情况下，比较两个树型结构差异的算法时间复杂度是O(N的3次方)**，但React通过总结DOM的实际使用场景，提出了两个绝大多数场景下都成立的假设。通过这两个假设，React实现了[O(N)]{.pink}复杂下完成两颗虚拟DOM树的比较。

1. 如果两个元素的类型不同，那么它们将生成两棵不同的树。
2. 对DOM节点跨层级移动的情况忽略不计。

比较过程：

### 当根结点不同类型时

这时候React会认为新的树和旧的树完全不同，会将整棵树拆掉重建（包括虚拟DOM树和真实DOM树），拆除的时候会触发组件的卸载生命钩子，重建的时候会触发挂载钩子。

### 当根结点是相同的DOM元素类型

React会保留根结点，比较根结点的属性，只更新变化了的属性。

### 当根结点时相同的组件类型

这时对应的组件实例不会被销毁，只会执行更新操作，同步变化的属性到虚拟DOM树上。在这一过程中组件实例的componentWillReceiveProps和componentWillUpdate会被调用。

:::primary

对于组件类型的节点，这时候React无法知道如何更新真实DOm树，需要在组件更新并且render方法执行完毕后，根据render返回的虚拟DOM结构决定如何更新真实DOM树。

:::

[根据这三个假设，React递归遍历虚拟DOM树，比较完之后得到最终的差异，更新到DOM树中。]{.pink}

当一个节点有多个子节点的时候，默认情况下React会按照顺序逐一比较两棵树上对应的子节点。

```html
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

这个时候最终只会插入一个新的节点。

但如果我们在开始位置新增一个节点，或者把最后的节点调整位置到最开始，就会出现这种特殊情况。

React会以为每一个结点都变化了，从而每个节点都被修改。

### key

为了避免前面那种特殊情况，提出了key。key时为了帮助React提高Diff算法的效率，使用key来匹配子节点，只要渲染之后子节点的key没有变化，React就会认为这是同一个节点。

但是加上key也未必“性能最优”，需要避免使用索引来作为元素的key，如果使用索引，也会出现前面那个情况。

# 减少不必要组件渲染

之前大概知道，可以通过should ComponentUpdate钩子来决定是否更新组件。

最好的办法是通过比较当前prop和下次props看有没有属性变化，但是深比较的性能影响比较大。

所以提出了折中的办法，**只比较第一层级的属性**。

而React中提出了一个Pure Component组件，这个组件会使用浅比较来比较新旧props和state，因此可以通过让组件继承Pure Component来代替手写should ComponentUpdate的逻辑。

```react
class NumberList extends React.PureComponent{
  ...
}
```

# React Fiber

JavaScript是单线程的，如果调用栈运行一个很耗时的脚本，比如解析一个图片，可能会长时间阻塞，使得其他事件都无法得到响应。

为了突破这个瓶颈，React中试着将耗时高、易阻塞的长任务进行切片，分成子任务并异步执行他们。这样，在每个子任务的空隙间就有机会处理其他任务，如UI更新。


