---
title: this关键字的笔记
date: 2021-05-25 22:34:44
tags:
categories:
- JavaScript
---

# 使用this的原因

 在编写代码的时候，我们不得不经常使用当前的上下文对象来做一些事情，比如说对象赋值、调用方法。如果我们不使用this，就需要经常传入一个上下文对象，非常的繁杂，并且随着使用模式和代码量的增长，显式传递上下文对象会让代码变得越来越混乱。

但是this提供一种更“优雅”的方式来隐式传递一个上下文对象。

# this到底是什么

==当一个函数被调用时，会创建一个执行上下文。==这个上下文包含函数在哪里被调用、函数的调用方法、参数等信息，而this就是上下文的其中一个属性。

# this的绑定机制

通常来说，this通常时运行时绑定的，但在我的理解中，还有一种特殊情况，就是箭头函数。

:::default

箭头函数中的this会自动捕获定义时外层最近的上下文环境，而非运行时绑定。

:::

## 绑定规则

this是在函数被调用时绑定的，完全取决于函数的调用位置。

```javascript
function add(a,b){
  return a+b;
}
add(1,2);
```

++所谓的调用位置，就像上述代码中add被调用的地方，它所在的执行上下文，才决定this的绑定。++

### 默认绑定

==默认绑定，就是独立函数调用时候的绑定方式。==

来看这么一段代码

```javascript
function foo(){
  console.log(this.a);
}

var a = 2;
foo(); //会输出2
```

foo是不带任何修饰的函数调用，==在非严格模式下，这个情况会归为默认绑定，自动绑定到全局对象处。==

:::warning

在严格模式下，全局对象无法使用默认绑定，这个情况下this会绑定到undefined。

:::

### 隐式绑定

==看调用位置的上下文对象。==

某个函数可能被某个对象包裹。简单的说，这个函数可能定义在某个对象内部，是某个对象的方法。

例：

```javascript
function foo(){
  console.log(this.a);
}

var obj = {
  a : 2,
  foo: foo
};

obj.foo(); //2
```

这也是非常常见的一种绑定方式，但是这种绑定方式有一个地方一定要小心，就是异步（回调函数的绑定问题）。异步的隐式绑定有可能会丢失this，为了避免错误，后面会说到显式绑定的方法。

### 显式绑定

不多说，直接上代码

```javascript
function foo(){
  console.log(this.a);
}
var obj = {
  a:2
}

foo.call(obj);//2
```

显式绑定是一种硬绑定，它会将函数的this强制绑定到某个上下文环境（对象）。

利用call、apply方法可以将函数绑定到某个上下文对象中并执行，它们的区别只是接收参数的形式不同，具体实现方式见附录。

但有时候我们只是想强制绑定this，并不想马上执行，为了应对这一情况，es5中提供了内置的方法bind。

```javascript
var bar = foo.bind(obj);
```

这时候的bar就是foo强制绑定obj环境后的函数。

### new绑定

​	使用new来调用某个函数的时候，会构造一个新对象，并且这个新对象会绑定到函数的this。

## 优先级

上述四种绑定规则，分别有各自的优先级，按照这个顺序进行判断：

### new绑定：函数是否在new中调用？

### 显式绑定：是否通过call、apply绑定？

### 隐式绑定：是否在某个对象中调用？

### 默认绑定：都不是前三种情况，在非严格模式下为全局对象，严格模式下为undefined

# 总结

this是JavaScript学习者最重要的基本功之一，一定要切实理解并熟悉。

# 附录

## 1、手撕call、apply方法

```javascript
Function.prototype.call = function(context){
  if(typeof this !== 'function'){
		throw new TypeError('error');
  }
  context = context || window;
  context.fn = this;
  const args = [...arguments].slice(1);
  const result = context.fn(...args);
  delete context.fn;
  return result;
}
```

```javascript
Function.prototype.apply = function(context){
	if(typeof this !== 'function'){
    throw new TypeError('error');
  }
  context = context || window;
  context.fn = this;
  let result;
  if(arguments[1]){
    result = context.fn(...arguments[1]);
  }else{
    result = context.fn();
  }
  delete context.fn;
  return result;
}
```

