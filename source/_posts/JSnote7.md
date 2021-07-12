---
title: 响应式框架学习笔记
date: 2021-06-07 14:52:01
tags:
categories:
- 前端
---

本文是根据我读的书《前端开发核心知识进阶》做出的提炼总结，仅作为个人学习、加深记忆所用

之前我学习vue的响应式原理的时候，就大概了解了一下它底层原理所用的API，如Object.defineProperty、Proxy等，现在来做一个深入学习和总结。

# 响应式框架基本原理

响应式框架的概念是指，==数据变化时不需要开发者手动去更新视图，视图会自动根据变化的数据去进行自动更新==

想要做到这一点，用通俗的话来说，需要做到以下几点

- 知道这个视图依赖了哪些数据
- 能够及时知道哪些数据变化了
- 变化的时候，通知对应的视图去更新

用专业术语来说，也就是

- 依赖收集
- 订阅数据变化——数据劫持、数据代理
- 通知更新（发布）——发布订阅模式

## 数据劫持

数据劫持，在es6之前，往往使用的是Object.defineProperty这个方法来实现操作。使用这个方法去定义数据的getter和setter，从而在获取数据/修改数据的时候监听到这一操作而达到订阅数据变化的目的。

```javascript
let data = {
  name:'orange',
  hobby:{
    title:'JavaScript',
    since:'2019-01-01',
  }
}

Object.keys(data).forEach(key => {
  let currentValue = data[key];

  Object.defineProperty(data,key,{
    enumerable:true,
    configurable:false,
    get(){
      //感知被获取
      console.log(`正在获取${key},值为：${currentValue}`);
      return currentValue;
    },
    set(newValue){
      currentValue = newValue;
      //通知改变
      console.log(`改变${key},改为${newValue}`);
    }
  });
});
data.name;//正在获取name，值为：orange
data.name = 'me';//改变name，改为me
data.hobby.since;//正在获取hobby,值为{title:'JavaScript,since:'2019-01-01'}
```

写到这里，很明显，获取和设置data.hobby的时候，它输出的不是我们想要的结果。

查资料发现，Object.defineProperty实际上只能监听一层，如果属性是引用类型，则需要递归下去进行深层拦截。

```javascript
const observe = data => {
  if(!data || typeof data !== 'object'){
    //保证对象存在才递归
    return;
  }
  Object.keys(data).forEach(key => {
    let currentValue = data[key];

    observe(currentValue);//递归进去，若是引用类型则深层拦截，否则直接返回

    Object.defineProperty(data,key,{
      enumerable:true,
      configurable:false,
      get(){
        console.log(`获取${key}，值为`,currentValue);
        return currentValue;
      },
      set(newValue){
        currentValue = newValue;
        console.log(`设置${key}，改为`,newValue);
      }
    });

  });
}
observe(data);
```

但是这样子改还有一个很大的问题，就是Object.defineProperty是基于改动对象属性时触发getter、setter方法的前提下才能够感知到，首先要满足这个大前提，而且实际情况下数据会复杂的多，还可能会存在newValue下为复杂类型的情况，我们都没有考虑到，只是写了一个简单的demo来举个例子。

==像数组的变化，数组变化其实是不触发setter的，导致使用Object.defineProperty无法监听到数组的变化==。如数组的push方法就不能被拦截。

:::primary

Vue2也是存在着这样的问题，但Vue的解决方法是将数组中常用的方法进行重写覆盖，以达到监听的效果

:::

## 数据代理

es6之中有一个新特性，Proxy

Proxy可以理解成，在和目标对象之间的操作加一层拦截，对这个对象做的任何操作，都必须先通过这层拦截。所以我们可以在这一层拦截中，对访问进行监听。

```javascript
let data = {
  name:'orange',
  hobby:{
    title:'JavaScript',
    since:'2019-01-01',
    sex:{
      name:'boy'
    },
    skills:['JavaScript','Vue','React']
  }
}

const observe = data => {
  if(!data || Object.prototype.toString.call(data)!== '[object Object]'){
    return;
  }

  Object.keys(data).forEach(key =>{
    let currentValue = data[key];
    if(typeof currentValue === 'object'){
      observe(currentValue);
      data[key] = new Proxy(currentValue,{
        set(target,property,value,receiver){
          if(property !== 'length'){
            console.log(`设置${key},改为`,currentValue);
          }
          return Reflect.set(target,property,value,receiver);
        }
      })
    }
  });
}
observe(data);
data.hobby.sex.name = 'girl';
data.hobby.name = 'orange2';
```

这是一个简单的例子，实际情况肯定比这个复杂的多。

## 数据劫持和数据代理的区别

- Object.defineProperty不能监听数组的变化，需要对数组方法进行重写。
- Object.defineProperty必须遍历对象的每个属性，且需要对嵌套结构进行深层便利。
- Proxy的代理是针对整个对象的，一层代理可以监听同级结构下的所有属性变化。（深层结构仍然需要递归）
- Proxy支持代理数组的变化

Vue中监听数据的变化，当数据变化之后，重新进行模板编译。现在实现了单向的变化，即数据变化引起视图变化。

## 双向绑定

以vue中的v-model为例。

假设页面中有一个输入框

```vue
<input v-model="inputData" type="text"/>
```

实际上，v-model只是一个语法糖，它做了这么一些操作

1. 将输入框的值与变量inputData绑定。
2. 监听输入框输入事件比如说input事件，触发事件时去修改数据源变量（这里是inputData）

这个其实和我学习React之中受控组件的概念有点类似，数据源负责渲染视图，修改输入框内容实质上触发事件修改数据源。

一旦修改了数据源，就会走一遍数据劫持/数据代理+模版编译的过程，重新渲染。这样就完成了响应式。

## 发布/订阅模式

实际上上面还少了一个事情，就是要精准的知道哪个视图需要订阅哪个数据，哪个数据修改需要向哪个视图发布通知。这个就利用到了发布订阅模式。

简单的发布订阅模式大概是这样的

```javascript
class Notify{
  constructor(){
    this.subscribers = [];
  }
  add(handler){
    this.subscriber.push(handler);
  }
  emit(){
    this.subscribers.forEach(subscriber => subscriber());
  }
}

let notify = new Notify();

notify.add(()=>{
  console.log('emit');
});
notify.emit();
```

## 虚拟DOM

在框架里面还有一个重要的概念就是虚拟DOM。所谓虚拟DOM就是使用数据结构来表示DOM结构，先操作虚拟DOM，再把有必要的变更应用到真实DOM树上。

==因为DOM操作其实是比较耗费资源和时间的，操作数据结构远比操作DOM要快，通过去比较、精确获取最小必要操作，可以提升渲染的性能==。

用户进行特定操作后，会产生一个虚拟DOM，通过对比前后两份虚拟DOM的差异，来得出最小变更，这个过程就是DOM diff算法。

## Vue框架响应式的原理

Vue2之中，首先对数据进行拦截（使用Object.defineProperty），转化为getter、setter，使得他们在被访问和变更时让Vue能够追踪，通知。每个组件实例都对应一个 **watcher** 实例，它会在组件渲染的过程中把“接触”过的数据 property 记录为依赖。之后当依赖项的 setter 触发时，会通知 watcher，从而使它关联的组件重新渲染。

![data](data.png)

