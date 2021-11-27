---
title: 深入浅出Vue笔记————实例方法篇
date: 2021-11-08 14:28:00
tags:
categories:
- Vue
---

感觉一章节让我对Vue内部的原理更加深入的了解了，如果说前面的响应式系统、虚拟DOM的实现是Vue的精髓之一的话，这部分实例代码的实现让我对Vue组件实例的理解更为深刻，深入Vue的代码骨架之中。

# 概述

Vue实例中有许多方法，本博客记录这些实例方法的实现原理

在Vue源码中的src/core/instance/index.js中，有这么一段代码：

```javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)//初始化相关
stateMixin(Vue)//数据相关
eventsMixin(Vue)//事件相关
lifecycleMixin(Vue)//生命周期相关
renderMixin(Vue)//渲染相关

export default Vue
```

这里其实是定义了Vue的构造函数，然后分别调用initMixin、stateMixin、eventsMixin等函数，实际上就是向Vue的原型中挂载方法。

例如：

```javascript
export function initMixin (Vue) {
  Vue.prototype._init = function (options){
    //...
  }
}
```

# 与事件相关的实例方法

在上面我们说到，在上面我们调用了eventsMixin(Vue)来在Vue的原型上挂载方法。使得每个Vue实例都能够调用这些方法：

vm为某Vue实例

- vm.$on
- vm.$off
- vm.$once
- vm.$emit

```javascript
export function eventsMixin(Vue){
  Vue.prototype.$on = function(event,fn){
    ...
  }
  ...
}
```

想了下，还是不放实现代码了，只说说我读到的一些想法。

这四个方法的实现实际上有点类似于Node中的EventEmitter，实现方法也有点点类似，但是细节上有些许出入。

思路大概是这样的，在新建Vue实例的时候，会初始化一个对象用来存放事件相关的东西：

```javascript
vm._events = Object.create(null);
```

## vm.$on

这个方法用于监听当前实例上的自定义事件，实现代码如下：

```javascript
    /**
     * 
     * @param {string | Array<string>} event 
     * @param {Function} fn callback
     * @description 给传入的event注册事件回调
     */
    function $on(event,fn){
        if(Array.isArray(event)){
            for(let i = 0,j=event.length;i<j;i++){//防止中途长度改变了
                this.$on(event[i],fn);
            }
        }else{
            (this._events[event] || (this._events[event] = [])).push(fn);
        }
    }
```

其实就是，以事件名为_events的属性名，将函数注册进去。$on方法也支持多个事件注册回调。

## vm.$off

这个方法用于移除自定义事件监听器。

- 如果没有提供参数，移除所有事件监听器
- 只提供了事件参数，移除该事件所有监听器
- 同时提供两个参数，则移除对应的

```javascript
    /**
     * 
     * @param {string | Array<string>} event 
     * @param {Function} fn 
     * @description 移除自定义事件监听器
     */
    function $off(event,fn){ 
        //第一种情况
        if(!arguments.length){
            this._events = Object.create(null);
            return this;
        }
        if(Array.isArray(event)){
            for(let i = 0,l = event.length;i<l;i++){
                this.$off(event[i],fn);
            }
            return this;
        }
        //第二种情况
        const cbs = this._events[event];
        if(!cbs){
            return this;
        }
        if(arguments.length === 1){
            this._events[event] = null;
            return this;
        }
        //第三种情况
        if(fn){
            const cbs = this._events[event];
            let cb;
            let i = cbs.length;
            while(i--){
                cb = cbs[i];
                if(cb === fn || cb.fn === fn){
                    //fn属性，用来下面once特殊执行off，因为once注册的事件监听器并不是原来的函数
                    cbs.splice(i,1);
                    break;
                }
            }
        }
        return this;
    }
```

这里为什么我们还要检测注册的回调的fn属性呢？

因为下面我们就要讲到，once的实现方式其实是在外**包装**了一下原来传入的回调函数，我们通过将fn函数设置为包装后的回调函数的fn属性，用来对比是否是对应的监听器。

## vm.$once

和on方法的效果差不多，但是只触发一次，触发之后移除。

```javascript
    /**
     * 
     * @param {string | Array<string>} event 
     * @param {Function} fn 
     * @description 监听一个自定义事件，但是只触发一次后就移除
     */
    $once(event,fn){
        function on(){
            this.$off(event,on);//解除后执行
            fn.apply(this,arguments);
        }
        on.fn = fn;
        this.$on(event,on);
        return this;
    }
```

其实我们就是使用了一个函数来包装原回调监听器，执行后移除，并且设置该包装函数的fn属性为原回调监听器，和上面off的实现原理相呼应。

## vm.$emit

这个函数触发当前实例上的事件。附加参数都会传给监听器回调。

```javascript
    /**
     * 
     * @param {string} event 
     * @param {...args} 
     * @description 触发当前实例上的某个事件,附加参数都会传给监听器回调
     */
    $emit(event){
        let cbs = this._events[event];
        if(cbs){
            const args = [...arguments].slice(1);
            for(let i = 0,i = cbs.length;i<l;i++){
                try{
                    cbs[i].apply(this,args);
                }catch(e){
                    handleError(e,this,`event handler for ${event}`);
                }
            }
        }
        return this;
    }
```

# 生命周期相关的实例方法

主要是四个方法：vm.$mount、vm.$forceUpdate、vm.$nextTick、vm.$destroy。

其中vm.$forceUpdate和vm.$destroy是在lifecycleMixin中挂载到Vue构造函数的原型上的。

## vm.$forceUpdate

这个方法的作用是迫使Vue实例重新渲染。前面我们也说过Vue2的响应式系统，在Vue2之中，更新粒度为组件级别，一个组件实例对应一个组件级别Watcher实例，实际上，只要调用组件对应的watcher的update方法即可。

## vm.$destroy

这个方法的作用很明显，就是完全销毁一个实例。

它做的事情大概有以下几个：

1. 先判断是否已经销毁，已经销毁不需重复销毁
2. 触发Vue实例的生命周期beforeDestroy钩子。
3. 清理当前组件实例和父组件之间的联系（Vue实例的$children属性存储了所有子组件）
4. 销毁实例上的所有watcher（包括组件级别的watcher和用户通过$watch方法创建的watcher）
5. 给Vue实例添加_isDestroyed属性来表示已经被销毁
6. 触发Vue实例的生命周期destroyed钩子。
7. 移除实例上的所有事件监听器。

具体就是这七步，然后我们结合源代码来看：

```javascript
Vue.prototype.$destroy = function () {
    const vm: Component = this
    //1.判断
    if (vm._isBeingDestroyed) {
      return
    }
  	//2.触发
    callHook(vm, 'beforeDestroy')
    vm._isBeingDestroyed = true
    //3. 清理自己和父组件的联系
    const parent = vm.$parent
    if (parent && !parent._isBeingDestroyed && !vm.$options.abstract) {
      remove(parent.$children, vm)
    }
    //4.清理组件级别的watcher（存在_watcher)
    if (vm._watcher) {
      vm._watcher.teardown()
    }
  	//清理用户创建的watcher
    let i = vm._watchers.length
    while (i--) {
      vm._watchers[i].teardown()
    }
    // remove reference from data ob
    // frozen object may not have observer.
    if (vm._data.__ob__) {
      vm._data.__ob__.vmCount--
    }
    // 表示已经被销毁
    vm._isDestroyed = true
    // 触发destroy钩子函数解绑指令
    vm.__patch__(vm._vnode, null)
    // fire destroyed hook
    callHook(vm, 'destroyed')
    // 移除所有事件监听器
    vm.$off()
    // remove __vue__ reference
    if (vm.$el) {
      vm.$el.__vue__ = null
    }
    // release circular reference (#6759)
    if (vm.$vnode) {
      vm.$vnode.parent = null
    }
  }
```

## vm.$nextTick

它接收一个回调函数作为参数，然后在下次DOM更新周期之后执行。

面向场景：更新了状态后有时需要对新DOM做一些操作时。

### 前置知识

在说这个API的原理之前，需要先说说Vue的一些特性。

在Vue之中，当状态发生变化，会通知依赖这个状态的所有watcher，然后触发虚拟DOM渲染流程。在watcher触发渲染这个操作并不是同步的，它是**异步的**。Vue在内部有一个队列——异步更新队列，每当需要渲染时，就将要渲染的watcher推送到这个队列，下一次事件循环再统一清空队列。

好处就是能够减少重复，组件的watcher要是再一轮事件循环中多次收到通知需要渲染，实际上只需一次渲染。

事件循环的话这里不再仔细说了，前面其他博客也讲过很多次，可以参考我之前的博客。

### API本身

这个API有几个特性：

- 回调执行前反复调用，也只会添加一个任务
- 当任务触发，依次执行

vm.$nextTick和Vue.nextTick是一样的，所以我们直接说Vue.nextTick的原理

### Vue2.4之前

```javascript
const callbacks = [];
let pending = false;
function flushCallbacks(){
	pending = false;
  const copies = callbacks.slice(0);
  callbacks.length = 0;
  for(let i = 0;i<copies.length;i++){
		copies[i]();
  }
}

let microTimerFunc;
const p = Promise.resolve();
microTimerFunc = () => {
  p.then(flushCallbacks);
}

export function nextTick(cb,ctx){//回调函数和执行环境
	callbacks.push(() => {
		if(cb)cb.call(ctx);
  });
  if(!pending){
		pending = true;
    microTimerFunc();
  }
}
```

这段代码有几个要点：

- pending变量用于防止反复添加任务到微任务队列中，一轮事件循环只会添加一次。
- Vue2.4版本之前，nextTick方法使用微任务，因为微任务优先级较高，可能会出现一些问题

### Vue2.4之后

正所谓在2.4之前都使用微任务，后来Vue提供了强制使用宏任务的方法。

具体代码就不贴了，和之前有几个区别：

- 利用了一个变量来判断是否使用宏任务
- 新增了一个函数withMacroTask，给回调函数做了一层包装，让更新DOM操作推到宏任务队列中。
- 优先使用setImmediate，然后MessageChannel、setTimeout
- 如果浏览器不支持Promise，则降级成宏任务添加

# 全局API

全局API和实例方法不太一样，后者是在Vue的原型上挂载方法，而前者是直接在Vue上挂载方法。

如：

```javascript
Vue.extend = function(options){
	...
}
```

