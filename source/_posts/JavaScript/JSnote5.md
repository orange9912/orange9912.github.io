---
title: JavaScript异步学习笔记
date: 2021-05-31 21:33:01
tags:
categories:
- 前端
---

# 前言

异步是JavaScript里一个非常重要的基本功。因为js是单线程的，为了提高CPU利用率，异步是必要的，否则的话要是某一个任务耗时非常长（比如IO设备读写），后一个任务就不得不等待很久，浪费了处理器的能力。

# 异步方案一：回调

前面说到，异步其实是让一些非常耗时的任务释放CPU的占用，去让下一个任务先执行，但我们怎样才能知道那个耗时的任务执行完没有呢？这就要通过回调函数来通知了，++回调函数是我们给这个任务的参数，当这个任务结束时，它会调用这个回调函数++。

形象的比喻就是，有一个场景，我放衣服进洗衣机去洗，这期间我不用一直看着洗衣机看它洗完衣服没有，而是可以去忙自己的事情，洗衣机洗完就会自动发出“哔哔哔”的声音==提醒==我们它洗完了，要来处理衣服了，等我们腾出手来就可以去处理洗完的衣服了。==这个声音，就是回调函数的形象比喻==。

来看这么一个需求：

[假设有一个红绿灯，其中红灯3秒亮一次，绿灯1s亮一次，黄灯2s亮一次，如何让三个等不断交替重复亮呢？]{.pink}

```javascript
//已知三个亮灯函数已经存在，如下
function red(){
  console.log('red');
}
function green(){
  console.log('green');
}
function yellow(){
  console.log('yellow');
}
```

如果用回调方案的话，我们写出来的代码应该是这样的。

```javascript
/**
*	@params {number} time 延迟时间
* @params {string} light
* @params {function} callback
*/
const task = (time,light,callback) => {
  setTimeout(() => {
    if(light === 'red'){
      red();
    } else if(light === 'green'){
      green();
    } else if(light === 'yellow'){
      yellow();
    }
    callback();//结束调用回调函数
  },time);
}
```

每一个任务函数接收一个延迟时间、一个灯色、以及结束后调用的回调函数。

然后这样调用：

```javascript
const step = () => {
 task(3000,'red',()=> {
  task(1000,'green',()=> {
    task(2000,'yellow',step);
  	});
	});
}
step();
```

==但很明显，上面回调的写法会造成一种现象叫做“回调地狱”==。当我们有许多任务有关联的时候，它嵌套的写法不仅使得代码不美观，更重要的是，这样的代码非常难以理解和维护。

# 异步方案二：Promise

## Promise是什么？

Promise是一个对象，保存着未来将要结束的事件。它主要用来进行异步计算，可以将异步操作队列化。

### Promise特征

Promise对象是由构造函数构造出来的一个对象，每一个Promise对象代表着一个异步操作。

```javascript
let promise1 = new Promise((resolve,reject) => {
  //异步操作，成功后调用resolve()，失败后调用reject()
});
promise1.then((response) => {
  //成功后的回调函数操作
});
promise1.catch((error)=>{
  //失败后的回调函数操作
});
//也可以选择将传给catch的函数作为then方法的第二个参数，只是写法不同。如下
promise1.then((response)=>{
  //成功后，fulfill的操作
},(error)=> {
	//失败后
})
```

:::default

promise构造函数的执行是同步的，then和catch才是异步的。

:::

Promise对象由三个状态，从上述代码也可看出一二。分别是pending(进行中)，fulfill(已成功)，rejected(已失败)。

:::default

一旦promise的状态改变，状态就凝固了，不会再改变。也就是说，promise状态改变只有两种可能，从pending改成fulfill或者从pending改成rejected。

:::

promise状态一旦改变，就会触发then()中相应的函数进行处理。

### Promise.resolve

promise的resolve方法用于将promise对象从pending状态改成fulfill状态。

它可以接受参数，根据参数的类型进行处理并传递给then中相应处理的方法。

#### 参数是一个promise对象

Promise.resolve将不做任何修改，直接传递这个实例

#### 参数是一个thenable的对象

Promise.resolve会将该对象转化为一个promise对象，并立即执行这个对象的then方法。

#### 参数不是promise对象，也不是thenable对象

比如传递一个字符串，Promise.resolve会返回一个新的、状态为resolved的对象传递过去

```javascript
const p = Promise.resolve('hello');
p.then((s)=>{
  console.log(s);
});
//hello
```

#### 不带任何参数

==直接返回一个resolved状态的promise对象，而且then()函数内不返回值或者返回的不是promise，那么then会自动返回一个resolve状态的promise==

### promise的链式调用

promise的then方法也会返回一个promise对象，该对象行为也是一样的。

如果我们有一些异步任务是相互关联的，比如读完任务一后去读任务二，再读任务三，就可以使用promise的链式调用。

```javascript
//假设我们有一个工具函数ajax(url,callback);
function request(url){
  return new Promise((resolve,reject) => {
    ajax(url,resolve);
  });
}
request('xxx').then((response)=>request('xxx'+response)).then((response)=>{console.log(response2)});
```

这样就能把它们的回调嵌套调用改成链式调用，代码变美观了许多，而且更易读了。

## Promise改写红绿灯

先是红灯三秒一次，设置之后改变promise状态为fulfill，调用then中对应的方法，同时==then方法会自动将返回值封装为一个promise对象，供下次链式调用==。

```javascript
const task = (time,light) => new Promise((resolve,reject)=>{
  setTimeout(() => {
    if(light === 'red'){
      red();
    } else if(light === 'green'){
      green();
    } else if(light === 'yellow'){
      yellow();
    }
    resolve();
  },time);
});
const step = () =>{
  task(3000,'red').then(() => task(1000,'green')).then(()=>task(2000,'yellow'));
}
```

## Promise相关重点API

### Promise.all

Promise.all([p1, p2, p3])用于将多个promise实例，包装成一个新的Promise实例，返回的实例就是普通的promise。
它接收一个数组作为参数
数组里可以是Promise对象，也可以是别的值，只有Promise会等待状态改变
当所有的子Promise都完成，该Promise完成，返回值是全部值的数组（顺序和all方法内参数顺序一致）
有任何一个失败，该Promise失败，返回值是第一个失败的子Promise结果。

### Promise.race

Promise.race类似于Promise.all方法，区别在于[它有任意一个完成就算完成]{.pink}，

:::primary

当iterable参数里的任意一个子promise被成功或失败后，父promise马上也会用子promise的成功返回值或失败详情作为参数调用父promise绑定的相应句柄，并返回该promise对象。——MDN文档

:::

# 异步方案三：Generator

## Generator是什么

ES6中的Generator其实不是为了解决异步问题而生的，但是它又非常适合解决异步问题。

Generator函数与普通函数不同，它最大的特点，++就是可以交出函数的执行权、将函数分段进行++。

## Generator的用法

generator函数有两个区分于普通函数的部分。

Generator函数的定义，在function关键字后面，函数名之前有个"*"

函数内部有yield表达式

来看一个例子

```javascript
function* func(){
  console.log('one');
  yield '1';//根据yield关键字来分阶段
  console.log('two');
  yield '2';
  console.log('three');
  return '3';
}
```

generator函数被调用时，==会返回一个指向内部状态的指针==，我们可以使用next方法，来进行下一阶段的执行。

```javascript
//因此我们可以
let g = func();
//第一次调用next方法时，从函数头部开始执行
g.next();
//one
//{value:'1',done:false}
g.next();
//two
//{value:'2',done:false}
g.next();
//three
//{value:'3',done:true}
//如果已结束再次调用next，返回的value是undefined，done属性仍为true
```

:::primary

g.next()也会返回一个对象，格式为{value:xxx,done:true/false}，value用于函数体内外数据的交换，像返回值一样返回即可，done表示是否执行完毕

:::

## Generator的意义

利用Generator函数暂停执行的效果，可以把异步操作写在yield语句内，等到next方法后再往后执行。这实际上等同于不用再编写回调函数，因为异步操作的后续操作可以放在yield语句下面，等到调用next方法时再执行。

而且使用Generator函数写法写起来已经非常像同步了，但有一点，流程管理非常的不方便。

# 异步方案四：async/await

## async/await概念？

学习async/await的话，熟悉Promise是前提。而且async/await其实是Generator函数的语法糖。

==async函数的实现，就是将Generator函数和自动执行器，包装在一个函数内==。

async函数**返回一个Promise对象**，可以使用then方法添加回调函数。

await其实是默认创建一个Promise对象（如果修饰的代码不是返回一个Promise，则创建一个异步完成的Promise，并且将resolve传的结果作为返回值），然后等待该Promise完成，将完成的结果作为返回值。

```javascript
async function foo(){
	console.log('0');
  let a = await 1;
  console.log(a);
  console.log('2');
}
console.log('start');
foo();
console.log('end');
//start->0->end->1->2
```

当函数执行遇到await时，它会将主线程的控制权交出，先继续执行其他同步代码，等到Promise完成状态的时候通过事件循环（微任务那一套）来继续执行。

- await 仅能在 async 函数内部使用，否则会抛出语法错误
- async 函数也可以用bind二次绑定作用域
- 调用 async 函数时本质上返回的是一个 promise，可以进行 .then() .catch() 操作
- 在 async 函数中，可以在 while, for, for/in, for/of 等控制语句中循环执行 await

## async/await改写红绿灯

```javascript
const taskRunner = async () => {
  await task(3000,'red');
  await task(1000,'green');
  await task(2000,'yellow');
  taskRunner();
}
taskRunner();
```
