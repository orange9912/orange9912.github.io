---
title: webpack学习
date: 2021-06-22 14:57:41
tags:
categories:
- 前端
---

webpack是我做前端开发时一个非常重要的工具，也是现在前端工程化一个非常重要的工具

之前使用的时候基本只是知道它大概怎么使用，但并没有深入探究。这次想深入总结一下自己对webpack的学习

# webpack是什么？

官网上是这么介绍webpack的：**本质上，webpack是一个用于现代Javascript应用程序的静态模块打包工具。当webpack处理应用程序时，它会在内部构建一个依赖图，这个依赖图对应映射到项目所需的每一个模块，并生成一个或多个bundle。**

![webpack](webpack.png "官方图")

现代前端开发技术不断的增长，为了提高开发效率，新的工具和技术也不断的出现，比如scss、typescript等，他们这些都有一个共同点，就是源代码是无法直接执行的，需要特定的工具进行转换才可以执行，而webpack正是做了这一件事情。

简单来说，就是能将项目中的各个不同类型的文件，通过构建、打包将其打包成可执行的js、css、html文件和静态资源文件。

发展到现在，webpack的功能越来越强大，它的功能有以下这些：

- 代码转换：将Scss文件编译成css、将typescript文件编译成JavaScript等
- 文件优化：压缩JS、CSS、HTML代码，压缩合并图片
- 代码分割：提取多个页面的公共代码，懒加载资源
- 模块合并：构建时将模块分类合并成一个文件
- 热重载：监听本地代码的变化，自动重新构建、刷新，便于开发。
- 自动发布：更新代码后，自动构建出线上发布代码并传输给发布系统。

# webpack核心概念

要了解webpack的工作原理和流程，首先需要了解一些基本的核心概念。

webpack虽然不配置也可以使用，但是我们经常需要用配置做一些调整，一般在项目根目录下新建一个webpack.config.js来添加配置项。

## 入口（entry）

既然是打包，总得有个打包的起点，**这个起点就称为入口**。入口是webpack构建的起点，它会从这个起点中构建一个内部依赖图，找出依赖的所有模块。

它的默认值是[./src/index.js]{.pink}，但是我们可以在webpack配置文件中修改它，比如

**webpack.config.js**

```javascript
module.exports = {
  entry: './src/entry.js';
}
```

而且，入口可以不止一个，按照经验，通常在多页应用中，一个HTML文档使用一个入口。

## chunk

chunk是代码块的意思，一个chunk由多个模块组合而成，一般一个入口及这个入口依赖的所有模块对应一个chunk。

## 出口（output）

打包肯定是有结果的，就是所谓的出口。output告诉webpack在哪里输出它所创建的bundle还有命名规范。

它的默认值是[./dist/main.js]{.pink}，但是也可以像入口一样进行配置

```javascript
const path = require('path');
module.exports = {
  ...
  output:{
    path:path.resolve(__dirname,'dist'),
    filename:'my.bundle.js'
  },
};
```

如果配置文件中设置了多个入口，那就需要修改filename为占位符来对应生成不同名字。

## loader

webpack原本只能理解JS文件和JSON文件，如果要处理其他类型，就需要用到loader。

简单来说，可以把loader比喻成一个翻译，它识别特定的文件类型，并将其转换成另外一种类型，供程序使用及添加到依赖图之中。

像我们刚刚说到的Scss、typescript源代码文件在webpack中都需要对应的loader进行转换。

在webpack的配置中，loader有两个属性：

- test：识别那些文件需要被转换
- use：那些文件要使用什么loader进行转换。

**webpack.config.js**

```javascript
const path = require('path');

module.exports = {
	//其他配置项省略
  module: {
    rules: [
			{
        test: /\.scss$/,
        use: ['style-loader','css-loader','scss-loader']
      }
    ],
  },
};
```

这里就是告诉webpack：“webpack，你遇到scss结尾的文件时使用scss-loader、css-loader、style-loader来帮我把它转换”

:::primary

需要注意的是，use的属性是一个数组，表示要使用的loader，而且执行顺序是由后到前。

:::

loader接收一个文件和配置项作为输入，然后将对应的文件转换后输出。

## plugin

loader用于转换文件，而plugin（插件）有着范围更广的任务，它是用来扩展webpack功能的，通过在webpack构建流程中注入钩子实现。

想要使用一个插件，只需要require它，然后把它添加到plugins数组中，比如这样：

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
module.exports = {
  ...//其他省略
  plugins:[
    new HtmlWebpackPlugin({template:'./src/index.html'});
  ]
}
```

:::primary

HtmlWebpackPlugin是用于生成html文件、并自动在这个文件中引入相关入口chunk的js文件和抽取出来的css文件的插件

:::

## compiler对象

它的实例包含了完整的webpack配置，全局只有一个compiler对象。可以通过这个对象访问webpack的内部环境。

## compilation对象

当webpack以开发模式运行时，每当监测到文件变化时，就会创建一个新的compilation对象。对象包含了构建过程中所有的构建数据。

# webpack编译结果

不同模块规范下的打包结果有一些细微的差别，但总体大致是一样的。

- webpack的打包结果就是一个IIFE（立即调用函数表达式），它接收一个对象modules作为参数，这个参数对象的key是依赖路径，value是简单处理后的脚本
- 打包结果中定义了一个重要的模块加载函数__webpack_require__ 
- 首先使用模块加载函数__webpack_require__去加载入口模块
- 加载函数使用了闭包变量installedModules，将已加载过的模块存在内存中。

# webpack工作流程

## 简单理解版本

Webpack工作时，会**首先读取项目根目录下的webpack.config.js来获取配置项**（当然，从4.0.0版本开始可以不用该文件配置，使用默认配置），配置项是一个对象，使用module.export导出，通常可以指定打包的入口文件（entry）、打包的输出相关（output），如输出文件名、位置等，以及指定文件转换使用的loader。

获取配置项后，会对每一个入口进行如下的工作：**从入口的模块开始递归解析该模块依赖的所有模块，每找到一个模块，会根据配置的loader进行文件转换（webpack原本只支持对js和json的解析，其他文件需要对应的loader提供支持），然后这些模块以入口为分组，每个入口及其所有依赖的模块会被分到一个组（chunk），而且每一个入口对应一个依赖图。最后将所有chunk转换成文件输出。**

这是一个正常的webpack工作流程，除此之外，还可以在webpack正常工作流程中使用插件，注入钩子在特定工作流程中对打包结果进行干预。

# 总结

Webpack是当下最流行的前端工程构建工具，它是用于打包一个工程项目静态模块的打包工具，它的出现大大方便了日常开发。

各种新特性、新语言、新框架的出现，提高了我们的开发效率，如typescript、sass、babel、vue等。但是我们开发出来的项目文件最终还是应该打包成一堆html、css、js文件，尽管这些新特性的创造者会提供一个工具来进行转换，但是一个个转换我们的开发文件的工作量是巨大的，这个时候webpack就开始发挥它的威力了。

Webpack的出现极大的简化了转换文件的流程，更好的去开发，现在各种新特性通常都会有可以集成到webpack开发的loader。

# 面试题

## loader和plugin的区别？

loader主要是提供了一个文件转换的能力。webpack原生只支持解析js和json文件，而loader让webpack有了加载其他模块的能力。

plugin主要是插件，用于扩展webpack的功能，比如抽离代码、压缩、配置开发工具等功能。

# 10.25补充

webpack构建依赖图的时候，遇到不同的模块就根据webpack配置文件中配置的loader进行转换。

plugin更像一个类，webpack会先把plugin实例化（new的时候对构造函数传入options），然后调用plugin的apply方法（传入complier）
编写plugin的时候只需要在apply方法中对webpack广播出来的各种事件钩子根据需要注册处理函数，如compiler.hooks.hookname.tap(className,(…) => {…})

## loader编写要点

loader实质上是一个**函数**，它接收前一个loader的返回值作为参数输入（如果是配置的第一个loader则就是源文件内容），一般是字符串or二进制buffer，经过一些处理后返回对应的类型。

简单情况下，就是这样：

```javascript
module.exports = function(source){
  //...一些操作
  return content;
}
```

当然实际上我们不止是需要做一些操作，我们经常会看到对一个loader进行配置，然后定制化的处理文件。

- 所以我们还需要获取开发者配置的options
- 以及对loader处理过程中的错误捕获、导出sourceMap等信息
- 还有些loader可能是异步的，比如less-loader就是异步的

对于前两点，可以这样

```javascript
const loaderUtils = require('loader-utils');
module.exports = function(source){
	const options = loaderUtils.getOptions(this);//获取开发者配置的配置对象
  //....一些工作
  this.callback(error,content,sourceMap,ast);
  //四个参数分别是：loader出错时向外抛出的错误，导出的内容，sourceMap，本次编译生成的抽象语法树
}
```

:::primary

获取配置传入的this实际指向一个叫loaderContext的loader-runner特有对象，详情请读webpack loader部分的源码

:::

对于异步loader的方案，可以使用简单的async/await，也可以使用webpack提供的this.async。调用this.async会返回一个回调，可以在异步操作完成后调用（这样就知道异步loader结束了），例如：

```javascript
const loaderUtils = require('loader-utils');
module.exports = async function(source){
  const options = loaderUtils.getOptions(this);
  function delay(){
    return new Promise((resolve,reject) => {
      setTimeout(() => resolve(source),1000);
    })
  }
  const callback = this.async();
  delay().then(res => callback(null,res));
}
```

## plugin编写要点

在webpack构建的生命周期中，webpack会广播许多事件，我们可以通过监听这些事件，注册事件处理函数去干预编译，改变输出结果。

简单来说，plugin更像是一个类（当然你用函数也不是不行，只要给它的原型定义一个以compiler对象为参数的apply方法即可）

webpack会先把插件实例化（传入配置信息），然后调用实例的apply方法。一般apply方法内是对webpack各种事件钩子注册函数，调用apply方法时就会将这些钩子注册好。然后在真正webpack工作各个阶段的时候，广播各种事件，并调用该事件的所有钩子（自然就会调用某个插件的该事件钩子函数）

```javascript
class myPlugin{
	constructor(options){
    this.options = options;
  }
  apply(compiler){
    compiler.hooks.具体hook的名字.tap('myPlugin',(compilation, callback) => {
      setTimeout(() => {
        callback();
      },1000);
    });//注册钩子,会在事件触发时执行操作,具体钩子去官网看
  }
}
```

- 编写一个带apply方法的class
- 插入指定事件钩子
- 使用compilation修改打包内容

## 与其他打包工具的对比

### Rollup.js

相比webpack，Rollup.js有以下几个特点：

- 仅支持ESNext模块
- 自带tree shaking功能
- 生成的冗余代码比较少

有那么一句名言：“建库用rollup，其他场景用webpack”，当然这并不是绝对的。

rollup和webpack不同的地方主要是rollup会将模块按照代码顺序引入同一个文件来解决模块依赖问题，所以无法做到拆包（模块已经完全透明了）。

### Vite

现在的Vite也非常的火，Vite的火主要也是webpack衬托起来的。前面也说道，webpack在冷启动的时候要遍历整个应用模块冷启动，在HMR（hot module replacement，模块热更新）的时候也会随着应用的增大而变慢。

相比之下，Vite做了几点让HMR的速度快了不少。比如将代码分析为源码和依赖，预构建依赖、源码按需提供。

具体特点就不说了，看官方文档就可以，补充资料：[vite多久后能干掉webpack？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/477139054/answer/2156019180)

