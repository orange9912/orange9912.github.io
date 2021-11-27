---
title: 深入浅出Vue笔记————响应式系统篇
date: 2021-10-15 17:30:54
tags:
categories:
- Vue
---

参考资料——《深入浅出Vue.js》刘博文著

前四章，讲述变化侦测的笔记。

# 概述

Vue的变化侦测属于“推”，当状态变化时，Vue可以知道是哪些状态变化了。当状态变化了，它会向所有依赖这个状态的视图（或者说组件/DOM节点）发出更新通知

在Vue1.x之中，Vue对每个状态都进行依赖追踪，更新粒度相当的细，但是也带来了比较大的内存开销。

在Vue2.x之中，引入了虚拟DOM并且把更新粒度调整到了组件级别（比状态级别要粗），但是也大大降低了依赖数量及依赖追踪所消耗的内存。

:::primary

尽管Vue调整到了组件级别，也仍然要比React更细：React的做法是粗放的，它使用虚拟DOM，若检查到该组件类型变化或数据变化，它会把以该组件为根的整个子树销毁重建或更新渲染。由于更新过程是同步的，于此同时也带来了性能问题，后来React16便引入了React fiber来解决这个问题。

至于为什么Vue不用，因为Vue更新DOM是走的异步更新队列（与React的this.setState异曲同工），暂时还没有出现严重的性能问题。

:::

# 具体追踪

追踪对象变化的方式有二：

- Object.defineProperty
- Proxy（ES6）

在Vue1.x及Vue2.x之中，用的是前者。而在Vue3.x之中，用的是后者。

**在getter中收集依赖，在setter中触发依赖**。就是说，先收集依赖，知道某个属性都在哪些地方被用上，然后当属性发生变化时，通知这些地方进行更新。

## Dep——存储依赖的地方

既然说到依赖收集，那么我们就需要一个地方用来管理依赖。

**Vue之中封装了一个Dep类，专门用于管理依赖**。这个类可以用来收集、删除依赖、或者向依赖发出通知。

## Watcher——依赖类型

依赖收集好了，当属性发生变化时，就要向对应的依赖发出通知。但是依赖有很多种，他们不是统一的，可能是开发者写的一个watch，也可能是在模板里面用，为了统一这个依赖，我们抽象出一个处理这个问题的类，这个类就叫做==watcher==

依赖收集阶段，我们只收集这个封装好的类的实例，通知也只通知这个实例，然后它再负责通知具体依赖。

```javascript
export default class Watcher{
  constructor(vm,expOrFn,cb){
    this.vm = vm;//对应的vue实例
    this.getter = parsePath(expOrFn);//执行this.getter()，可以读取data.a.b.c的内容
    this.cb = cb;//更新时要调用的方法
    this.value = this.get();
  }
  get(){
    //利用window.target作中转，触发对象属性的getter，getter中自然会将window.target收集到对应的dep实例
    window.target = this;
    let value = this.getter.call(this.vm,this.vm);
    window.target = undefined;
    return value;
  }
  update(){
    const oldValue = this.value
    this.value = this.get();
    this.cb.call(this.vm,this.value,oldValue);//更新，传入新值和旧值
  }
}
```



当某个属性的getter被触发时，会执行Watcher实例的get方法进行依赖收集。

当某个属性改变（触发setter时），会将对应的Dep实例中收集到的依赖逐个通知，即通知Watch实例触发update方法。

## Observer——定义响应式对象

通过前面的API，可以侦测到数据的变化，不过有时候会存在对象嵌套的现象，于是Vue封装一个Observer类。

**这个类的作用是将一个对象内所有属性都转换成getter/setter的形式，然后追踪他们的变化**。

```javascript
export class Observer{
  constructor(value){
    //传入一个对象
    this.value = value;
    //数组的监听解决方案下面会说
    if(!Array.isArray(value))this.walk(value);
  }
  /**
  * 当参数为object时被调用
  */
  walk(obj){
    const keys = Object.keys(obj);
    for(let i = 0;i<keys.length;i++){
      defineReactive(obj,keys[i],obj[keys[i]]);
    }
  }
}

function defineReactive(data,key,val){
  if(typeof val === 'object')new Observer(val);//递归对象子属性
  let dep = new Dep();
  Object.defineProperty(data,key,{
    enumerable: true,
    configurable: true,
    get: function(){
      dep.depend();
      return val;
    },
    set:function(newVal){
      if(val === newVal)return;
      val = newVal;
      dep.notify();
    }
  })
}
```

当data中的属性发生变化时，这个属性对应的依赖就会收到通知。

而由于Object.defineProperty这个API本身的问题，**无法追踪新增的属性和删除的属性，也无法追踪到数组的修改**，而且它对于深层对象需要递归进行遍历，性能上不是那么的好。因此后续便使用的ES6的Proxy来进行替代。

# Vue2中响应式的缺点及解决方案

### 缺点

Vue2使用的是Object.defineProperty来进行数据劫持，也就是说，**变化侦测的方式是通过getter/setter实现的**。

那么就会存在以下两个问题（刚刚也提到了）：

- 无法检测属性的添加与移除

  因为在初始化时Vue对每个属性进行getter/setter转化，初始化时不存在的话就无法转化，自然也不是响应式的。

- 无法检测数组的变动，如使用push/pop方法改变，或者利用索引直接设置、修改数组长度之类的变动。

  因为这些方法并不会触发setter，自然也就无从监听了。

### 解决方案

前者的解决方案较简单：**Vue提供了一个Vue.set(object,propertyName,value)方法用于向嵌套对象添加响应式属性**。

后者的解决方案就稍微复杂一点。Vue2的解决办法是：**将常用的数组方法进行重写，从而覆盖原生的数组方法(push,pop,shift,unshift,splice,sort,reverse)**。

### 侦测数组的解决方案细节

像上面所说的一样，**首先新建一个以Array.prototype为原型的对象（伪数组原型，下面我们称之为拦截器），然后将其覆盖到要定义为响应式数组的原型上**。

```javascript
const arrayProto = Array.prototype;
export const arrayMethods = Object.create(arrayProto);

['push','pop','shift','unshift','splice','sort','reverse']
.forEach(function(method){
  //缓存原始方法
	const original = arrayProto[method]
  Object.defineProperty(arrayMethods, method ,{
    value:function mutator(...args){
      return original.apply(this,args);
    }
    enumerable:false,
    writable:true,
    configurable:true
  });
})
```

```javascript
import {arrayMethods} from './array';
const hasProto = '__proto__' in {};
const arrayKeys = Object.getOwnPropertyNames(arrayMethods);

export class Observer{
  constructor(value){
    this.value = value;
    
    if(Array.isArray(value)){
      const augment = hasProto ? protoAugment : copyAugment;
      augment(value, arrayMethods, arrayKeys);
    }else{
      this.walk(value);
    }
  }
}
function protoAugment(target, src, keys){
	target.__proto__ = src;
}
function copyAugment(target, src, keys){
  for(let i = 0 ,l = keys.length;i<l;i++){
		const key = keys[i];
    def(target, key , src[key]);
  }
}
```

:::primary

这里为什么不直接覆盖Array.prototype呢？因为我们不希望直接覆盖全局，尽可能的不向外污染内部的设计，所以使用覆盖响应式对象的原型来代替。

:::

顺带一提，如果浏览器不支持_ _proto__来访问对象的原型，就**直接将这些重写方法设置到被侦测的数组上**（因为原型屏蔽的原因，只有对象自身不存在该方法才按照原型链向上查找）。

**然后，还需要进行依赖收集。Array在getter中收集依赖，在拦截器中触发依赖**（当你调用方法修改数组时，实际上是调用拦截器的方法）。

在Vue.js中，Array的依赖存放在Observer中（对象是存在定义响应式对象的那个函数中，即defineReactive），然后将Observer实例挂载到对应数组实例上，拦截器就能够访问Observer实例中的dep了。

:::primary

之所以存在那里，是为了让getter和拦截器都可以访问到依赖。

:::

通过覆盖，我们就也能知道数组自身的变化了，不过光是这样还不够，对于数组之中的元素我们也需要进行侦测。于是Vue中还会进行一次循环，将数组的每个元素通过Observer进行转化。同时对新增的元素也进行转化。

# Vue3中的更改

后来Vue3使用了ES6的Proxy进行重写响应式系统，不仅能够监听到对象属性的增加与删除，同时也能够监听数组的变化。

# 总结

总的来说，分为几个关键词：**数据劫持（Vue3是数据代理）、依赖收集、发布/订阅**。

- 数据劫持。Vue之中使用Observer类，把一个对象的所有属性转换成getter/setter。

- 依赖收集。

  简单来说，**对象和数组都是在getter中进行依赖收集，但是对象在setter中触发依赖，而数组在拦截器中触发依赖**。因此，对象将依赖保存在defineReactive中，而数组将其保存在Observer实例中，并且将该Observer实例存放到数组上。

  依赖收集在Dep之中，依赖的类型则是Watcher实例。当外界某个地方依赖某个数据，通过Watcher读取数据时。新建的时候触发对应数据的getter，通过**window.target**作为中转，在触发数据getter时收集到Dep实例中。

- 发布/订阅模式。依赖收集的时候建立**数据-Watcher**的对应关系其实就是订阅，当数据发生变化（数据的setter方法被调用），则会通过dep.notify通知对应的所有依赖（对应的所有Watcher实例），然后Watcher实例再通过自己的update方法通知外界进行更新。

前面说过，在Vue1.x之中，更新粒度是属性级别的，即一个属性对应一个Watcher实例。

但在Vue2.x之中，更新粒度是组件级别的，**即一个组件实例对应一个组件级的Watcher实例(实际上在组件内部用户可以通过$watch方法创建watcher)**。一个组件可能依赖很多数据，这些数据对应的Dep都收集了这个Watcher实例，在数据变化的过程中，只要一个数据变化，都会引起这个组件的重新渲染。

:::primary

不同数据的Dep可能收录同一个Watcher实例，数据的Dep可收录多个Watcher实例。（多对多）

:::

![Vue变化侦测关系图](Vue变化侦测关系图.jpg "如图所示")

# Q&A

## 简述Vue的响应式原理？

主要是三个关键词，数据劫持、依赖收集、发布订阅模式。Vue通过Observer类将对象转化成getter/setter，将数组的原型用拦截器覆盖，都在getter中收集依赖存放到Dep实例中。对象在setter中通知依赖更新，而数组在拦截器中通知依赖。依赖就是Watcher实例，外界通过依赖读取数据，当数据发生变化时通知依赖，依赖再通知外界进行更新。

## Vue2响应式原理的缺点？

Vue2中使用的是Object.defineProperty进行数据劫持，这个API对对象的每个属性进行遍历转化，因此Vue2无法监听新增/删除的属性。同时该响应式基于数据改变触发setter的基础，而数组的修改方法不触发setter，需要进行重写覆盖。