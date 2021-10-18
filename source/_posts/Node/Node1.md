---
title: Node学习笔记一
date: 2021-09-21 15:34:58
tags:
categories:
- Node
---

这段时间在学习Node，看书、看文档资料的时候概念感觉太乱了，于是打算写一篇文章来整理自己学到的知识，就有了这篇文章）

# Node简介

Node.js是一个开源、跨平台的JavaScript运行时环境。2009年的时候，Ryan Dahl想开发一个高性能的web服务器，经过一番深思熟虑，他最终选择了JavaScript作为Node的实现语言。时至今日，Node已经成为最火热的技术之一。

# Node的特点

- 事件驱动

  和浏览器端JavaScript类似，

- 异步I/O

- 单线程

  准确来说是JavaScript的执行是单线程的，其实Node中不同平台内部完成I/O任务另有线程池

  但其实类似前端浏览器中的**Web Workers**，Node也提供了创建子进程的方法来高效的利用CPU和I/O

- 跨平台

  Node基于**libuv**实现跨平台（libuv是一个跨平台的异步I/O库，它提供的内容不仅是I/O，还有进程、线程、定时器、线程池等。）

  Node用libuv作为抽象封装层，使得所有平台兼容性的判断由该层来完成。**Node在编译时会判断平台条件，选择性编译unix目录或win目录下的源文件到目标程序中**。

# Node与浏览器上的JavaScript不同之处

- 面向范围不同

  浏览器上的JavaScript是用于前端开发的，而Node是用于Web服务器开发的

- 语言组成不同

  在前端中，**JavaScript = ECMAScript + DOM + BOM**。

  Node中的语言（按我的理解）是等于**ECMAScript + 和操作系统交互的API（诸如读取文件等）**

- 运行环境不同

  前端的JavaScript跑在浏览器中的js引擎(不同浏览器的js引擎不同，如最经典的Chrome的引擎是V8，除此之外还有SpiderMonkey、Nitro等)，是被限制在浏览器中的沙箱的。

  Node.js在浏览器外运行**V8**（Chrome的JavaScript引擎）。基于V8的执行效率，Node的计算能力也是比较优秀的。

# Node应用场景

显然，异步I/O的使用场景最大就是**I/O密集型场景**。它的优势主要在于Node利用事件循环的处理能力。

# Node（零碎）基础知识

## 从命令行接收输入

利用readline模块创建一个接口从process.stdin读取

```javascript
const readline = require('readline').createInterface({
  input:process.stdin,
  output:process.stdout
});

readline.question('what\'s your name? \n',name => {
  console.log(`hello! ${name}!`);
  readline.close();
});
```

## Node模块规范

Node中模块规范用的是CommonJS规范，它的使用方式大致如下：

```javascript
//模块引用
const http = require('http');
//导出一
exports.add = (a,b) => a+b;
//导出二
module.exports = {
  ...
};
```

它的特点有以下几个：

1. CommonJS以同步的方式加载模块。这一点其实可以理解，因为Node通常是运行于服务端，而在服务端模块文件通常存放在本地磁盘，读取速度比起前端通过网络下载要快得多，所以一般没有什么问题（但是因此也不适用于前端）
2. CommonJS输出的是模块的拷贝，这一点与ESNext模块不同。模块一旦输出后便独立（即后续更改不影响）
3. Node中对引入过的模块都会进行缓存，减少后续引入的开销。

## 核心模块

1. util，提供常用函数的集合。如util.inspect(将任意对象转换为字符串)、util.isArray等等。

2. fs，文件系统API，用于操作文件。

   如fs.readFile(用于异步读取文件)、fs.open、

3. http，用于创建服务器。如http.createServer

4. url，提供一些操作url的方法，如获取url中的参数

5. path，用于处理和转换文件路径的工具。

还有等等非常多的模块。

## 全局对象

在浏览器中的JavaScript，全局对象通常是window。而在Node中，全局对象是global。

一般来说我们有几个常常使用的变量

1. process，用于描述当前Node进程状态的对象。经常用的有很多，如process.env.NODE_ENV用于描述是开发环境（development）或者生产环境（production）
2. __filename，它表示当前正在执行的文件名。它会输出文件所在位置的绝对路径。
3. __dirname，它表示当前执行脚本的目录。

## Buffer正确拼接

不要这样：

```javascript
var fs = require('fs');

let rs = fs.createReadStream('test.md');
var data = '';
rs.on('data',(chunk) => {
  data+=chunk;//这里相当于data = data.toString() + chunk.toString()
});
rs.on('end',() => {
  console.log(data);
});
```

这种情况下，宽字节的中文在Buffer转String时有可能会被截断、无法正常显示（出现乱码）

最好的办法就是将Buffer正确拼接成一段大Buffer后，再进行转String操作。

```javascript
var chunks = [];
var size = 0;
res.on('data',(chunk) => {
  chunks.push(chunk);
  size += chunk.length;
});
res.on('end', () => {
	var buf = Buffer.concat(chunks,size);
  var str = iconv.decode(buf,'utf8');
  console.log(str);
})
```

# Node重点知识

## Node事件循环

Node中的事件循环与浏览器中的事件循环不太一样，浏览器环境下的就不详细讲了，见我的其他博客：[JavaScript运行机制笔记 - 前端 | orange's blog = orange's blog = 橙子的博客 (zyczxq.com)](https://zyczxq.com/2021/06/06/JavaScript/JSnote6/)

事件循环是Node处理非阻塞I/O的机制。

Node中使用libuv来进行I/O处理，Node中的Event Loop也是基于libuv实现的。

Node中的Event loop共分为6个阶段，如下：

```tex
	 ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

1. timers（**定时器**）。执行setTimeout、setInterval中到期的callback。这里其实是由轮询阶段来控制定时器何时执行，而且不是精确的时间，有可能因为操作系统调度而被延迟。

2. pending callback（**待定回调**）。执行延迟到下一个循环迭代的 I/O 回调。

3. idle，prepare。仅内部使用

4. poll（**轮询**）。检索新的 I/O 事件;执行与 I/O 相关的回调（几乎所有情况下，除了关闭的回调函数，那些由计时器和 `setImmediate()` 调度的之外），其余情况 node 将在适当的时候在此阻塞。也就是说，如果执行的时间比较长，有可能定时器超时都还没返回timers阶段执行定时器。

   如果轮询队列不为空，事件循环将循环访问队列并同步执行直到空。

   如果轮询队列是空的，有两件事：

   - 如果脚本被setImmediate调度，则事件循环结束轮询，进入check（检查）阶段执行那些被调度的脚本
   - 如果没有被setImmediate调度，则事件循环会等待回调被添加到队列中，然后立即执行

   在这个过程中，一旦轮询队列为空，**事件循环还会检查已经到达时间阈值（或者超时）的计时器，如果有就回到定时器阶段执行对应的回调**。

5. check（**检测**）。执行setImmediate回调。

6. close callbacks（关闭的回调函数）

在每次运行的事件循环之间，Node.js 检查它是否在等待任何异步 I/O 或计时器，如果没有的话，则完全关闭。

:::primary

process.nextTick()不是事件循环的一部分，它在事件循环的每个阶段完成之后，和微任务一样去执行（**但它比其他的微任务优先级都要高**）

:::

### setImmediate()对比setTimeout()

主要是调用时机不同。

setImmediate在当前轮询阶段完成后，就执行脚本。而setTimeout在最小阈值过后运行脚本。

**执行计时器的顺序将根据调用他们的上下文而异。如果两者都从主模块内调用，则受进程性能的约束（顺序是非确定性的）**

**但是在异步I/O callback内部调用时，总是先执行setImmediate，再执行setTimeout**。因为I/O回调在poll阶段执行，当执行完后队列为空时，存在setImmediate回调的话会先跳转到check阶段去执行回调。

### process.nextTick()、setImmediate()

process.nextTick()比setImmediate()触发的更快。如果想设置立即异步执行一个任务，最好不要使用setTimeout(fn,0)，而是使用process.nextTick()或setImmediate()。

定时器不够准确，很多时候会超时，而且嵌套调用最小单位4ms、未激活页面最小间隔1000ms等都不够精确、底层红黑树的操作时间复杂度为O（lg(n)），而nextTick为O(1)，更高效。

### 与浏览器事件循环的差异？

浏览器会在每个宏任务执行完毕后清空微任务队列

而Node中微任务（microtask）在事件循环的各个阶段执行。即每个阶段执行完毕，就会去执行microtask队列的任务。



## 内存控制

Node的内存控制基于V8的内存控制，基本是一样的，不再重复，见：https://zyczxq.com/2021/09/23/JavaScript/v8-memoryManage/

# 参考资料

[Node.js 中文网 (nodejs.cn)](http://nodejs.cn/)

朴灵的《深入浅出nodejs》