---
title: 深入浅出Vue笔记————生命周期篇
date: 2021-11-08 14:36:31
tags:
categories:
- Vue
---

# 开头

开头，我们问自己几个问题：

- Vue实例初始化阶段都经历了些什么？
- beforeCreate钩子调用之前都干了些什么？
- Vue实例的事件系统是如何初始化的？
- inject的原理？
- 访问Vue实例的props和data，我们实际上访问的是什么？

带着问题去看文章。

每个Vue实例在创建的时候都要经过一系列初始化，从创建到销毁，Vue的生命周期可以分为四个阶段：**初始化、模板编译、挂载、卸载阶段**。

![lifecycle](lifecycle.png "官方图")

接下来，**我们从Vue的构造函数深入了解Vue的生命周期，看看Vue是怎么建立并初始化的**。不知道的可以再看看前面的博客，我们前面说到，Vue有个构造函数，在某个文件内，会引入不同的Mixin，往Vue的原型上挂载方法，其中Vue构造函数就引用了initMixin的方法用于初始化。

```javascript
function Vue(options){
  if(!process.env.NODE_ENV !== 'production' && !(this instanceof Vue)){
		warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options);//就是这里
}
```

# this._init

这个方法是通过initMixin挂载到Vue原型上的，接下来我们来看看他的细节。

```javascript
Vue.prototype._init = function (options){
	const vm = this;
  vm.$options = mergeOptions(
  	resolveConstructorOptions(vm.constructor),
    options || {},
    vm
  );//这里会根据用户传递、父级实例的options进行合并
  
  initLifecycle(vm)
  initEvents(vm)
  initRender(vm)
  callHook(vm,'beforeCreate')//触发对应的生命周期事件
  initInjections(vm)
  initState(vm)
  initProvide(vm)
  callHook(vm,'created')
  
  if(vm.$options.el)vm.$mount(vm.$options.el)
}
```

不得不说，大佬写的代码就是逻辑清晰和易读，基本上函数名已经是解释了它做了什么。

在初始化流程很明显，先是初始化实例属性、事件、渲染，然后触发beforeCreate生命周期，然后继续下去。

## 初始化实例属性

Vue通过**initLifecycle**函数向实例中挂载属性，局部代码如下：

```javascript
export function initLifecycle(vm){
  const options = vm.$options;
  //找出第一个非抽象父类
  let parent = options.parent
  //options中有一个属性为abstarct，表示当前组件是否为抽象组件
  //找到第一个非抽象父组件，然后添加自身给父组件，并设置$parent
  if(parent && !options.abstract){
		while(parent.$options.abstract && parent.$parent){
      parent = parent.$parent
    }
    parent.$children.push(vm);
  }
  
  vm.$parent = parent;
  vm.$root = parent ? parent.$root : vm//表示当前组件树的根Vue实例
  vm.$children = [];
  vm.$refs = {}
  vm._watcher = null
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

这是局部的代码，其实就是给一些属性赋一个默认值。

## 初始化事件

初始化事件即将父组件在模板中使用的v-on注册的事件添加到子组件的事件系统中。我们都知道，Vue中父组件可以在使用子组件的地方用v-on来监听子组件触发的事件。

:::primary

如果v-on写在组件标签上，事件就会被注册到子组件的事件系统中。如果写在平台标签（如div），就会把事件注册到浏览器系统中。

:::

```javascript
export function initEvents(vm){
  vm._events = Object.create(null)
  //初始化父组件附加事件
  const listeners = vm.$options._parentListeners
  if(listeners)updateComponentListeners(vm, listeners)
}
```

首先初始化_events属性为空对象，用来存储事件。所有使用vm.$on注册的事件监听器都会保存到这个属性中

然后在模板编译阶段，如果解析到组件标签，就会实例化子组件，同时将标签上注册的事件解析并传递给子组件的$options._parentListeners中。

那如果是这样，存在listeners，就将它（父组件向子组件注册的事件）注册到子组件实例中。**调用updateComponentListeners**。

### updateComponentListeners

组件通过该方法将父组件中向子组件注册的事件注册，初始化的时候其实只要循环vm.$options._parentListeners并使用vm.$on方法注册即可

代码如下：

```javascript
let target

function add(event, fn, once){
  if(once){
    target.$once(event,fn)
  }else{
    target.$on(event,fn)
  }
}

function remove(event,fn){
  target.$off(event,fn)
}
export function updateComponentListeners(vm,listeners,oldListeners){
	target = vm
  updateListener(listeners,oldListeners || {} , add ,remove, vm)
}
```

这里还封装了两个函数，用于新增和删除事件。

### updateListeners

这个函数思路其实比较简单，如果listeners对象存在某个事件而oldListeners不存在，则说明需要新增，反之则移除。

它的功能其实就是比对listeners和oldListeners来分辨哪些事件需要add注册，哪些需要remove移除。

:::primary

读到这里的时候，我就感觉这个函数应该不止是在初始化时可以调用，在重新渲染/更新时也可以调用来更新事件

:::

代码就懒得贴了，简单说一下这个函数的内部过程吧：

主要分为两个循环，第一部分循环listeners，判断哪些事件不在oldListeners中，调用add注册这些事件。第二部分循环oldListeners，造出哪些事件不在listeners，调用remove移除这些事件。

### 注意

这里还要注意的是，实际上我们使用Vue在模板中注册事件的时候还可能会有一些事件修饰符，如capture、once之类的，写成像：

```html
<child @increment.once="a"/>
```

在Vue中有个函数叫**normalizeEvent**，它就是用来将这些修饰符解析出来的函数。

## 初始化render

```javascript
export function initRender (vm: Component) {
  vm._vnode = null // the root of the child tree
  vm._staticTrees = null // v-once cached trees
  const options = vm.$options
  const parentVnode = vm.$vnode = options._parentVnode // the placeholder node in parent tree
  const renderContext = parentVnode && parentVnode.context
  vm.$slots = resolveSlots(options._renderChildren, renderContext)
  vm.$scopedSlots = emptyObject
  // bind the createElement fn to this instance
  // so that we get proper render context inside it.
  // args order: tag, data, children, normalizationType, alwaysNormalize
  // internal version is used by render functions compiled from templates
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  // normalization is always applied for the public version, used in
  // user-written render functions.
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)

  // $attrs & $listeners are exposed for easier HOC creation.
  // they need to be reactive so that HOCs using them are always updated
  const parentData = parentVnode && parentVnode.data

  /* istanbul ignore else */
  if (process.env.NODE_ENV !== 'production') {
    defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, () => {
      !isUpdatingChildComponent && warn(`$attrs is readonly.`, vm)
    }, true)
    defineReactive(vm, '$listeners', options._parentListeners || emptyObject, () => {
      !isUpdatingChildComponent && warn(`$listeners is readonly.`, vm)
    }, true)
  } else {
    defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, null, true)
    defineReactive(vm, '$listeners', options._parentListeners || emptyObject, null, true)
  }
}
```

## 触发生命周期钩子

我们可以看得到，Vue中通过callHook函数来触发生命周期钩子。

我们先说说Vue所有的生命周期钩子：

- beforeCreate
- created
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeDestroy
- destroyed
- activated
- deactivated
- errorCaptured

其中前八个是比较正常的生命周期，倒数第二第三个按我自己的经验在keepalive中用的比较多。

接下来我们说说callHook的功能，callHook的作用是触发用户设置的生命周期钩子。

这个函数有一个需要注意的点，我们可以在vm.$options获得用户设置的生命周期函数（如vm.$options.created），**但是这里获取到的是一个数组**。（如果你看过Vue官方文档中的混入，应该会知道这个玩意）

```javascript
export function callHook(vm,hook){
	const handlers = vm.$options[hook]
  if(handlers){
    for(let i = 0,j=handlers.length;i<j;j++){
      try{
        handlers[i].call(vm)
      } catch(e){
        handleError(e,vm,`${hook} hook`)
      }
    }
  }
}
```

callHook会遍历对应生命周期钩子名称所有的函数，逐个执行。

## 初始化inject

### 使用方式

很多人可能包括我自己没怎么经常用过inject/provide，所以先说说这玩意干什么用吧。

inject/provide，也就是依赖注入，有时候某些深层子组件都需要使用一个方法，但我们很难将其传入，就可以使用依赖注入。

在Vue实例中，使用provide选项，指定我们想要提供给后代的数据/方法，例如：

```javascript
provide: function(){
  return {
    getMap: this.getMap
  }
}
```

然后在任何后代组件中，使用inject选项来接收指定的属性：

```javascript
inject: ['getMap']
```

有点像React中的Context吧。

### 实现原理

这两个玩意一般是成对出现的，但是在初始化中，我们看的到，先初始化inject，然后初始化状态，最后才到provide。

这样做也是有深意的，后初始化的可以依赖先初始化的，即在Vue实例中，**用户可以在data/props中使用inject注入的内容**。

我们知道通过provide注入的内容可以被所有子孙组件通过inject的得到。

```javascript
export function initInjections (vm: Component) {
  const result = resolveInject(vm.$options.inject, vm)//返回一个对象，每一项属性都是inject的值
  if (result) {
    //这里其实是通知defineReactive函数不要将内容转换成响应式的，保存后再调回来
    toggleObserving(false)
    Object.keys(result).forEach(key => {
      if (process.env.NODE_ENV !== 'production') {
        defineReactive(vm, key, result[key], () => {
          warn(
            `Avoid mutating an injected value directly since the changes will be ` +
            `overwritten whenever the provided component re-renders. ` +
            `injection being mutated: "${key}"`,
            vm
          )
        })
      } else {
        defineReactive(vm, key, result[key])
      }
    })
    toggleObserving(true)
  }
}
```

这里我们可以看到，在初始化inject的过程中，调用了resolveInject来获取实例中需要inject的值，然后遍历这些值，定义为当前实例的非响应式属性。

:::primary

上面那个警告的意思大体是不要直接改变注入的值，当提供这个值的组件重渲染时，这个值会被更改。

:::

好吧，那我们想知道它的实现原理，就得深入到resolveInject这个函数中去找它的代码了。

```javascript
export function resolveInject (inject: any, vm: Component): ?Object {
  if (inject) {
    // inject is :any because flow is not smart enough to figure out cached
    const result = Object.create(null)
    const keys = hasSymbol
      ? Reflect.ownKeys(inject)
      : Object.keys(inject)

    for (let i = 0; i < keys.length; i++) {
      const key = keys[i]
      // #6574 in case the inject object is observed...
      if (key === '__ob__') continue
      const provideKey = inject[key].from
      let source = vm
      while (source) {
        if (source._provided && hasOwn(source._provided, provideKey)) {
          result[key] = source._provided[provideKey]
          break
        }
        source = source.$parent
      }
      if (!source) {
        if ('default' in inject[key]) {
          const provideDefault = inject[key].default
          result[key] = typeof provideDefault === 'function'
            ? provideDefault.call(vm)
            : provideDefault
        } else if (process.env.NODE_ENV !== 'production') {
          warn(`Injection "${key}" not found`, vm)
        }
      }
    }
    return result
  }
}
```

这代码有一点点长哈，我们一个部分一个部分的来读。首先我先说一下它的基本思想，**它实际上就是使用inject配置的key从当前组件读取内容，读不到就找他的父组件，然后一层层往上找直到找到内容，x中（保存为非响应式的属性)**。我感觉有点像作用域链，或者说有点像原型链的查找方式。

## 初始化状态

初始化状态呢，自然就是初始化一些我们平时用到的，例如props、data、methods、computed、watch之类的东西。

我们来看代码：

```typescript
export function initState (vm: Component) {
  vm._watchers = []//用来保存当前组件的所有watcher
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    //否则直接观察空对象。
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

还是比较易读的，我们可以看到，配置中有什么，他就初始化什么，并且调用对应的初始化函数，代码非常精简。

初始化顺序为：

1. props
2. methods
3. data
4. computed
5. watch

**后初始化的可以操作前初始化的**。这个初始化顺序非常符合逻辑，例如我们就可以在data中使用props了，在computed中使用data。

接下来我们看一下各个初始化的代码。

### 初始化props

props呢，其实就是**父组件提供数据**，子组件在内部通过props字段选择自己需要哪些数据，然后Vue内部通过子组件的props选项将数据筛选出来、再添加到**子组件的上下文**之中。

像我们常常用数组来指定props，但其实最后我们需要**将它规格化成为对象**的形式。

#### 规格化props

在初始化props之前，实际上在this._init里面，在传递配置的时候，Vue中使用了：

```javascript
//init.js
vm.$options = mergeOptions(...);
```

而在mergeOptions函数中，有这么一行代码：

```javascript
//options.js
normalizeProps(child, vm)
```

所以，实际上我们在initProps之前，是先做了一步规格化props的操作的，我们需要先了解这个函数到底做了什么事情。

```typescript
/**
 * Ensure all props option syntax are normalized into the
 * Object-based format.
 */
function normalizeProps (options: Object, vm: ?Component) {
  const props = options.props
  if (!props) return
  
  const res = {}
  let i, val, name
  
  if (Array.isArray(props)) {
    i = props.length
    while (i--) {
      val = props[i]
      if (typeof val === 'string') {
        name = camelize(val)
        res[name] = { type: null }
      } else if (process.env.NODE_ENV !== 'production') {
        warn('props must be strings when using array syntax.')
      }
    }
  } else if (isPlainObject(props)) {
    for (const key in props) {
      val = props[key]
      name = camelize(key)
      res[name] = isPlainObject(val)
        ? val
        : { type: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "props": expected an Array or an Object, ` +
      `but got ${toRawType(props)}.`,
      vm
    )
  }
  options.props = res
}
```

这段代码其实干了以下的事情：

1. 首先判断是否有props属性，没有就**提前返回**，不用规格化，很好理解。

2. 声明一些用到的变量，如res用来**保存规格化后的结果**。

3. 接下来就是主要逻辑了。检查props是否为一个数组，是的话转步骤4，否则检查是否为对象类型，如果是则转步骤5，都不是则在非生产环境下的控制台打印警告。

4. 是数组，那就遍历数组每一项，检查是否为string类型，是的话就将这个key**从蛇形命名法转换为驼峰命名法**。并且转化为对象赋值给res。

   这里为什么要转呢？这是因为我们在模板中父组件向子组件传递数据的时候，在标签中用的是蛇形命名法。

   即<child user-name={name}/>。

5. 如果是对象，那就用for-in循环props，也进行key的命名转换、但对值的处理稍有不同。

6. 最后将结果覆盖options.props。

总的来说，它就是将props字段规格化为Object类型。

#### 本体

现在才来到我们的初始化props的本体。

**通过规格化后的props从其父组件传入的props数据中**或者**从new创建实例时传入的propsdata参数中**，筛选出需要的数据存入vm._props，再在vm实例上设置一个代理，实现通过vm.x访问vm._props.x。

```javascript
function initProps (vm, propsOptions) {
  const propsData = vm.$options.propsData || {};
  const props = vm._props = {};
  // 缓存props的key
  const keys = vm.$options._propKeys = [];
  const isRoot = !vm.$parent;
  if (!isRoot) {
    toggleObserving(false);// 这里是让下面的defineReactive不用转换成响应式数据
  }
  for(const key in propsOptions) {
    keys.push(key);
    //获取父组件传下来实际的值，进行边界条件的判断和处理
    const value = validateProp(key, propsOptions, propsData, vm);
    defineReactive(props, key, value);
    if (!(key in vm)) {
			proxy(vm, `_props`, key);// 设置代理
    }
  }
  toggleObserving(true);
}
```

这里顺带一提一下proxy，其实它就是通过定义对应key的getter/setter来使得它获取到实际上是另外一个地方的值，代码如下：

```javascript
const sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
}

export function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

### 初始化methods

初始化method比较简单，主要有两步：**校验方法是否合法和将方法挂载到vm中**。

```javascript
function initMethods (vm: Component, methods: Object) {
  const props = vm.$options.props
  for (const key in methods) {
    if (process.env.NODE_ENV !== 'production') {
      if (typeof methods[key] !== 'function') {
        warn(
          `Method "${key}" has type "${typeof methods[key]}" in the component definition. ` +
          `Did you reference the function correctly?`,
          vm
        )
      }
      //如果props中已存在这个key
      if (props && hasOwn(props, key)) {
        warn(
          `Method "${key}" has already been defined as a prop.`,
          vm
        )
      }
      //如果是内部已经有的实例方法
      if ((key in vm) && isReserved(key)) {
        warn(
          `Method "${key}" conflicts with an existing Vue instance method. ` +
          `Avoid defining component methods that start with _ or $.`
        )
      }
    }
    vm[key] = typeof methods[key] !== 'function' ? noop : bind(methods[key], vm)
  }
}
```

:::primary

isReserved是用来检测字符串是否以$或_开头。在Vue中，$开头的是对外放出的，\_开头一般只在内部使用。

:::

### 初始化data

data的逻辑其实相对来说还是比较简单的，就是执行data所指向的函数，从而得到一个对象，并且检测是否有重名prop和method，没有的话就设置代理并转换成响应式数据。

### 初始化computed

computed相对复杂一点，首先先说说它的特性。

我们都知道，计算属性的特点就是有缓存，在依赖的数据没有发生变化的情况下，反复读计算属性，而计算属性函数**并不会反复执行**。

 computed其实是定义在vm上的一个特殊的getter，它结合了Watcher来实现缓存和依赖收集的功能。

那计算属性是怎么知道自己的依赖值改变了呢？

它自己有一个Watcher，当依赖值发生改变，自己的Watcher会收到信息，并且将dirty属性设为true，下次读取的时候就重新计算。

#### 原理

先说计算属性的原理吧，

1. 使用watcher读取计算属性。
2. 读取计算属性中的数据、并且使用Watcher观察。如果在模板中，就是组件级的watcher负责观察，如果在用户自定义watch中，就是自定义生成的watcher观察。
3. 当数据发生了变化，则通知计算属性的watcher观察数据的变化，同时通知组件的Watcher数据发生了变化，准备重新渲染
4. 计算属性的Watcher把自己的dirty属性设为true。
5. 当重新读区计算属性的值时，如果dirty为true，则重新计算一次。

计算属性其实是一个getter，对应有一个watcher，读取别的数据的时候就会被别的数据收集依赖进Dep。

- 在SSR(Server Side Render)下，计算属性只是一个普通的getter，没有缓存效果。

### 初始化watch

### 初始化provide

