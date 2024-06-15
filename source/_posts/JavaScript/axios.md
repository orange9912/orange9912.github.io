---
title: Axios源码解析
date: 2021-09-01 14:34:36
tags: 
categories:
- 前端
---

前几天面试，和面试官聊天时

面试官：“你有没有想过看一下源码，而不是看那些分析文章，可能直接看源码收益来的直接一点”

我：“以前尝试过看的框架源码，不过觉得好晦涩，看不懂，打算等水平提升一点再看”

——————————————————————————————————————————————

于是最近几天，我觉得自己的水平相比以前也有点提升，虽然看不懂框架源码，但是我应该勉强能看懂一些简单库的源码？

**于是就选中了Axios**，仔细查找一番资料+拉取github上的代码慢慢看，确实有了不少收获，还加深了理解。

于是就有了这篇文章，不过源码的内容实在太多，所以这篇文章只是简单对工作流程源码的一篇解析，其他的功能还请慢慢读源码

——————————————————————————————————————————————

第一次写，可能有点乱，求大佬轻喷

# Axios是什么

axios是一个基于Promise的HTTP库，可以用在浏览器和nodejs之中使用。

它的特点有：

- 从浏览器创建XMLHttpRequests
- 从nodejs创建http请求
- 支持Promise
- 拦截请求和响应
- 取消请求
- 自动转换JSON
- 客户端支持防御CSRF

# Axios使用方式

了解它的内部机制之前，我们先要知道该模块的功能、输入输出，才能更好的了解它。

## 方法1:直接使用axios构造函数

axios(config) || axios(url[,config])

```javascript
axios({
  method:'POST',
  url:'/user/12345',
  data:{
		firstName:'Orange',
    lastName:'juice'
  }
});
```

## 方法2:使用axios对象的方法

- axios.request(config)
- axios.get(url[,config])
- axios.post(url[,data[,config]])
- 等等如head、delete、put方法

## 方法3:axios.create...

# 从axios入口文件分析Axios工作流程

暴露在项目根目录下的：

```javascript
//index.js
//就这么一行代码，让我们把目光放在axios.js
module.exports = require('./lib/axios');
```

然后跑去找到axios.js

```javascript
//axios.js的局部核心代码
/**
 * 创建axios
 *
 * @param {Object} defaultConfig The default config for the instance
 * @return {Axios} A new instance of Axios
 */
function createInstance(defaultConfig) {
  //创建Axios实例，参数为默认配置
  var context = new Axios(defaultConfig);
  //将Axios.prototype.request的函数的this绑定指向到context（创建的axios实例)
  //类似于Axios.prototype.request.bind(context)
  var instance = bind(Axios.prototype.request, context);
  //将Axios.prototype上的方法和属性都扩展到instance上，并且将这些扩展的方法绑定this为context
  utils.extend(instance, Axios.prototype, context);
  //把context上的方法和属性扩展到新的Axios实例上，主要是配置和拦截器
  utils.extend(instance, context);
  return instance;
}

//创建一个axios实例导出，这个实例实际上指向的是Axios.prototype.request函数
var axios = createInstance(defaults);
// 新建Axios实例的工厂方法
axios.create = function create(instanceConfig) {
  //创建一个axios实例，配置为原先defaults.js中的配置+参数传入的配置
  //配置会以一个优先顺序进行合并，优先级为lib/default.js的默认值<实例的defaults<请求的config
  return createInstance(mergeConfig(axios.defaults, instanceConfig));
};
//导出axios
module.exports = axios;
```

在这里我们可以看到，初始化的时候已经新建了一个默认配置的实例，**这个实例指向Axios.prototype.request函数(绑定了一个Axios实例)**，当我们以axios()方式调用的时候，实际上是执行了createInstance返回的一个指向Axios.prototype.request的函数。

当然，也支持使用axios.create来新建一个自定义配置的实例（也是指向request函数），**但最终也是执行Axios.prototype.request方法**。

**从这里我们可以看出，axios指向发请求的函数，而Axios是保存实例默认配置的对象，当调用axios的时候，使用对应的Axios实例的默认配置+参数配置来发起请求**

:::primary

当我们没有特别的要求时，使用默认的axios实例即可，否则的话我们可以新建一个axios实例（传入定制的配置信息），并且通过调用这个新建的实例来发起请求。

:::

既然发送请求最终调用的都是Axios.prototype.request函数，那我们来[简单]{.pink}看看Axios.js文件内部的代码，涉及某功能模块的具体解析在后面再详细讲，这里只是为了分析Axios的工作流程。

```javascript
//Axios.js部分代码
//Axios构造函数，一个Axios实例里有实例配置和请求拦截器+响应拦截器
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}

Axios.prototype.request = function request(config) {
  //如果第一个参数是字符串，则是url，否则是配置对象,axios(url[,config]) || axios(config)
  if (typeof config === 'string') {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    config = config || {};
  }
  //合并Axios实例中保存的默认配置和参数配置，默认配置优先级更低
  config = mergeConfig(this.defaults, config);

  // 若传入配置中指定了方法（或者实例defaults配置指定了），则改为小写，否则默认为get方法
  if (config.method) {
    config.method = config.method.toLowerCase();
  } else if (this.defaults.method) {
    config.method = this.defaults.method.toLowerCase();
  } else {
    config.method = 'get';
  }

  //过滤跳过的拦截器
  var requestInterceptorChain = [];
  var synchronousRequestInterceptors = true;//是否同步执行
  //此方法对请求拦截器中每一项中执行函数unshiftRequestInterceptors（会自动排除被eject注销的handler）
  //把拦截器中每一项存入requestInterceptorChain
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    if (typeof interceptor.runWhen === 'function' && interceptor.runWhen(config) === false) {
      return;
    }
    //只要有一个异步执行，整个队列都异步执行
    synchronousRequestInterceptors = synchronousRequestInterceptors && interceptor.synchronous;
	  //注意这里是unshift，所以先定义的拦截器是后执行的（栈）
    requestInterceptorChain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  //基本同上，但这里是push，正序
  var responseInterceptorChain = [];
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    responseInterceptorChain.push(interceptor.fulfilled, interceptor.rejected);
  });

  var promise;

  //若需要异步执行，则异步执行拦截器数组
  if (!synchronousRequestInterceptors) {
    var chain = [dispatchRequest, undefined];
    //将派发请求放在请求拦截器数组异步执行完的最后一步执行
    Array.prototype.unshift.apply(chain, requestInterceptorChain);
    chain.concat(responseInterceptorChain);

    promise = Promise.resolve(config);
    while (chain.length) {
      promise = promise.then(chain.shift(), chain.shift());
    }
    
    return promise;
  }
  //否则同步执行请求拦截器，直到拦截器栈清空
  var newConfig = config;
  while (requestInterceptorChain.length) {
    var onFulfilled = requestInterceptorChain.shift();
    var onRejected = requestInterceptorChain.shift();
    try {
      newConfig = onFulfilled(newConfig);
    } catch (error) {
      onRejected(error);
      break;
    }
  }
  //请求拦截器执行完之后，派发请求
  try {
    promise = dispatchRequest(newConfig);
  } catch (error) {
    return Promise.reject(error);
  }
  //执行响应拦截器，异步执行
  while (responseInterceptorChain.length) {
    promise = promise.then(responseInterceptorChain.shift(), responseInterceptorChain.shift());
  }
  //最后返回一个promise对象
  return promise;
};
```

# Axios工作流程

根据上面源代码的分析，我们可以知道Axios的工作流程大概是这些

1. 创建Axios实例（createInstance），自定义创建配置或使用默认配置

2. 调用axios，传入配置，合并配置并做一些处理

3. 执行请求拦截器（requestInterceptorManager）。

   拦截器的handler会收到实例的配置作为参数，然后需要返回一份新的配置

4. 派发请求（dispatchRequest）

5. 转换请求数据（transformData）

6. 使用Adapter处理请求（xhr.js或http.js）

7. 转换响应数据（transformData）

8. 执行响应拦截器（responseInterceptorManager）

9. 结束，返回一个promise

# 拦截器模块

在Axios实例的代码中，每个Axios实例都有着请求拦截器和响应拦截器。

```javascript
//Axios.js
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}
```

让我们看看拦截器中的代码具体是怎么写的，

```javascript
//InterceptorManager.js
function InterceptorManager() {
  this.handlers = [];
}

//往栈里增加一个新的拦截器
InterceptorManager.prototype.use = function use(fulfilled, rejected, options) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected,
    //决定该拦截器是否同步执行
    synchronous: options ? options.synchronous : false,
    runWhen: options ? options.runWhen : null
  });
  //返回一个下标，方便之后注销该handler
  return this.handlers.length - 1;
};

InterceptorManager.prototype.eject = function eject(id) {
  if (this.handlers[id]) {
    this.handlers[id] = null;
    //其实就是用这个下标去注销handler，在下面的forEach方法中会跳过被注销的
  }
};

//对拦截器数组中的每一项执行fn
InterceptorManager.prototype.forEach = function forEach(fn) {
  //utils.forEach实际上是对第一个参数进行遍历，执行第二个参数函数
  utils.forEach(this.handlers, function forEachHandler(h) {
    //h是传入的每一项，忽略被注销的
    if (h !== null) {
      fn(h);
    }
  });
};
```

我们可以看到，一个InterceptorManager对象保存着一个handlers数组，用来保存一个对象（这个对象包含了不同情况下执行的拦截函数）。结合Axios.js，有几个信息：

- 使用use方法添加一个拦截器，use方法返回一个下标。

- 可以使用use方法返回的下标来调用eject注销拦截器

- 在Axios.prototype.request方法中，调用实例的forEach方法来复制一份拦截器队列执行

- 请求拦截器先定义的后执行，响应拦截器先定义的先执行

- 请求拦截器可同步或异步执行，响应拦截器异步执行

  请求拦截器的执行可以要求异步执行，也可以是同步执行，如果要异步执行，设置拦截器的时候给第三个参数传入一个options对象，设置options.synchronous = false即可（**只要有一个需要异步执行，那整个队列都异步执行**）

# 派发请求模块

源码：

```javascript
// lib/core/dispatchRequest.js 部分核心代码

module.exports = function dispatchRequest(config) {
  // 保证headers存在
  config.headers = config.headers || {};

  // 转换请求数据
  config.data = transformData.call(
    config,
    config.data,
    config.headers,
    config.transformRequest
  );

  // Flatten headers
  config.headers = utils.merge(
    config.headers.common || {},
    config.headers[config.method] || {},
    config.headers
  );

	//可以自定义适配器，否则就用默认的（浏览器下用xhr.js）
  var adapter = config.adapter || defaults.adapter;

  return adapter(config).then(function onAdapterResolution(response) {
    throwIfCancellationRequested(config);
  
    // 转换响应数据
    response.data = transformData.call(
      config,
      response.data,
      response.headers,
      config.transformResponse
    );
    return response;
  }, function onAdapterRejection(reason) {
    if (!isCancel(reason)) {
      throwIfCancellationRequested(config);

      // 转换响应数据
      if (reason && reason.response) {
        reason.response.data = transformData.call(
          config,
          reason.response.data,
          reason.response.headers,
          config.transformResponse
        );
      }
    }
		
    return Promise.reject(reason);
  });
};
```

可以看到，派发请求模块主要是以下几个流程：

1. 转换请求数据、设置头部、选择适配器
2. 发送请求
3. 得到响应后进行对应的处理，如成功后转换数据并返回

# 转换数据模块

在dispatchRequest.js中，我们可以看到对请求数据的转换和对响应数据的转换代码，如下

```javascript
// lib/core/dispatchRequest.js 转换数据局部代码
//对请求数据的转换
config.data = transformData.call(
  config,
  config.data,
  config.headers,
  config.transformRequest
);
...
//对响应数据的转换
response.data = transformData.call(
  config,
  response.data,
  response.headers,
  config.transformResponse
);
```

在这里我们可以看到，有两个关键点：**transformData和config.transformResponse**

## transformData

```javascript
// lib/core/transformData.js
//转换请求/响应数据
module.exports = function transformData(data, headers, fns) {
  var context = this || defaults;
  
  //注意这里的fns，是转换函数数组，为扩展转换埋下了伏笔
  utils.forEach(fns, function transform(fn) {
    data = fn.call(context, data, headers);
  });

  return data;
};
```

结合在派发请求时候的调用，我们可以知道以下几点

- 这个函数本身并不直接转换数据，而是调用defaults.js中的转换方法数组来进行一个转换
- 传入要转换的data和headers，然后返回一份转换好的data
- 转换函数可以扩展个数和重写

## config.transformResponse/Request

通过上面可以知道，真正转换数据的是axios.default.transformRequest(Response)方法

```javascript
// defaults.js
  transformRequest: [function transformRequest(data, headers) {
    normalizeHeaderName(headers, 'Accept');
    normalizeHeaderName(headers, 'Content-Type');
    //如果是特殊的数据，不需要进行转换直接返回
    if (utils.isFormData(data) ||
      utils.isArrayBuffer(data) ||
      utils.isBuffer(data) ||
      utils.isStream(data) ||
      utils.isFile(data) ||
      utils.isBlob(data)
    ) {
      return data;
    }
    if (utils.isArrayBufferView(data)) {
      return data.buffer;
    }
    if (utils.isURLSearchParams(data)) {
      setContentTypeIfUnset(headers, 'application/x-www-form-urlencoded;charset=utf-8');
      return data.toString();
    }
    //如果是对象且我们规定了json,则进行json.stringift转化
    if (utils.isObject(data) || (headers && headers['Content-Type'] === 'application/json')) {
      setContentTypeIfUnset(headers, 'application/json');
      return JSON.stringify(data);
    }
    return data;
  }],
```

这个模块并不是很复杂，基本就是按照headers的类型和数据本身类型做的一个转换处理，为了减少篇幅就不贴transformResponse部分的代码了，可以自行下载源码查看

## 扩展

我们可以看到，在我们没有刻意去重写转换方法的时候，它使用的是defaults.js中默认的转换方法，但其实我们也可以根据需要，去增加或者重写转换方法。

```javascript
import axios from 'axios';
//这里导入的是默认的axios实例，我们也可以axios.create创建一个自定义配置实例
let instance = axios.create({
  baseURL='https://xxx.com'
});
instance.default.transformRequest.push((data,headers) => {
	//进行一系列的处理
  return data;
})
//或者进行重写
instance.default.transformRequest = [(data,headers) => {
	//同上
  return data;
}]
```

## 转换模块总结

总的来说

1. 派发请求的时候，将请求模块config作为**transformData的this指向，调用transformData**

2. transformData中会根据传入的config.transformRequest方法，来进行一个转换

   如果自定义的axios实例有进行重写或扩展，就调用我们重写或扩展的，否则就用默认的default.js中的

3. 转换完成，返回转换完成的数据

# Axios对ajax封装的模块

接下来到了最核心的地方：**Axios是如何对将ajax请求promise化的？**

话不多说，我们来看源代码（浏览器环境下的封装）

```javascript
//xhr.js 部分核心代码
module.exports = function xhrAdapter(config) {
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    //分离出请求相关的配置、数据、头部
    var requestData = config.data;
    var requestHeaders = config.headers;
    var responseType = config.responseType;
    //新建一个XMLHttpRequest对象
    var request = new XMLHttpRequest();

    //路径为baseURL+configURL
    var fullPath = buildFullPath(config.baseURL, config.url);
    //ajax请求方法、url设置
    request.open(config.method.toUpperCase(), buildURL(fullPath, config.params, config.paramsSerializer), true);

    // 设置超时时间
    request.timeout = config.timeout;
    //请求结束的处理函数
    function onloadend() {
      // 处理响应数据，略
      ...
      //根据响应状态码设置Promise是resolve还是reject
      settle(resolve, reject, response);
      //加载完后置null
      request = null;
    }
    //onloadend是一个属性，请求结束时就会存在
    //不存在就监听状态变化
    if ('onloadend' in request) {
      request.onloadend = onloadend;
    } else {
      //readyState为4的时候代表已完成
      request.onreadystatechange = function handleLoad() {
        if (!request || request.readyState !== 4) {
          return;
        }
        
        //宏任务，当同步任务执行完毕后执行
        setTimeout(onloadend);
      };
    }

    // 当请求被丢弃或者手动停止时的处理
    request.onabort = ...;
    // 网络出错处理
    request.onerror = ...;
    // 超时处理函数
    request.ontimeout = ...;
    // 在标准浏览器环境里添加CSRF头部，来防御CSRF攻击
    if (utils.isStandardBrowserEnv()) {
      // 如果该请求携带cookie或者是同源网站就需要添加(用于鉴权)
      var xsrfValue = (config.withCredentials || isURLSameOrigin(fullPath)) && config.xsrfCookieName ?
        cookies.read(config.xsrfCookieName) :
        undefined;

      if (xsrfValue) {
        requestHeaders[config.xsrfHeaderName] = xsrfValue;
      }
    }
    // 添加配置到请求头，略

    if (!requestData) {
      requestData = null;
    }
    // 发送请求
    request.send(requestData);
  });
};
```

总的来说其实就是将ajax请求promise化，封装进去，先进行一波配置的处理，然后请求结束时再根据状态码决定该Promise是完成还是失败状态。

在这里还加了对csrf攻击的防御。

# 取消请求模块

## 使用方式

在文档上我们查看到取消请求的两种方式

- 使用CancelToken.source工厂方法产生cancel token

  ```javascript
  var CancelToken = axios.CancelToken;
  var source = CancelToken.source();
  
  axios.get('/user/12345', {
    cancelToken: source.token
  }).catch(function(thrown) {
    if (axios.isCancel(thrown)) {
      console.log('Request canceled', thrown.message);
    } else {
      // 处理错误
    }
  });
  
  // 取消请求（message 参数是可选的）
  source.cancel('Operation canceled by the user.');
  ```

- 或者传递一个executor函数到CancelToken的构造函数来创建cancel token

  ```javascript
  var CancelToken = axios.CancelToken;
  var cancel;
  
  axios.get('/user/12345', {
    cancelToken: new CancelToken(function executor(c) {
      // executor 函数接收一个 cancel 函数作为参数
      cancel = c;
    })
  });
  
  // 取消请求
  cancel();
  ```

从上述使用方式我们可以看到，要对请求产生一个cancelToken，然后在配置项中将请求的cancelToken设置为我们产生的cancelToken。

这个cancelToken具有一个cancel方法，通过调用它可以取消请求。

## 原理

```javascript
// dispatchRequest.js
/**
 * 如果被要求取消，就抛出cancel,这个方法在派发请求前和响应后都会调用一次
 */
function throwIfCancellationRequested(config) {
  if (config.cancelToken) {
    //下面一行干了这个事：if(token.reason) throw token.reason;
    config.cancelToken.throwIfRequested();
  }
}

// CancelToken.js 部分核心代码
function CancelToken(executor) {
	...
  var resolvePromise;
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });

  var token = this;
  //这里的cancel函数实际上就是取消请求函数，利用executor向外传cancel回调
  executor(function cancel(message) {
    if (token.reason) {
      // Cancellation has already been requested
      return;
    }

    token.reason = new Cancel(message);
    resolvePromise(token.reason);
  });
}

//用于检查是否需要取消并抛出
CancelToken.prototype.throwIfRequested = function throwIfRequested() {
  if (this.reason) {
    throw this.reason;
  }
};

//产生CancelToken的工厂方法
CancelToken.source = function source() {
  var cancel;
  var token = new CancelToken(function executor(c) {
    cancel = c;
  });
  return {
    token: token,
    cancel: cancel
  };
};
```

实际上构造cancelToken时，利用传进来的函数将构造函数中写好的取消请求函数的引用传递出去，我们在使用的时候就可以调用该传出来的函数进行取消（这个函数会设置token.reason为取消的信息）。

当我们调用cancel方法时，会将cancel的信息传递，并设置token.reason。

在进入dispatchRequest步骤时，如果存在cancelToken且token.reason存在，就会抛出

Xhr.js中发送请求前，如果配置项存在cancelToken，**就会给这个cancelToken.promise.then()设置一个回调来取消请求**。当我们调用了cancel方法时，token内部的一个promise就会从pending->fulfilled状态，然后通过该回调取消请求。

如果发送完请求得到响应后，在转换数据前，Axios也会检测，如果存在cancelToken且token.reason存在，也会抛出。

## 设计步骤

**运行步骤大概如下**：

1. 利用构造函数或工厂方法产生一个cancelToken，并且把它设置为对应请求的cancelToken。

2. 如果是利用构造函数产生，我们还需要保存一下executor中传递出来的cancel函数。

3. 在适当的时候取消请求，调用cancel函数（可以传一个字符串作为信息）

   此时cancel函数会新建一个Cancel实例赋给token.reason，并且将token内部的promise完成（fulfilled）

4. **派发请求前和响应后**由dispatchRequest.js中的方法**throwIfCancellationRequested**来处理取消（如果token.reason存在，则表示需要取消）

   若是已经在请求中，xhr.js中有代码监听token内部的promise，一旦该promise完成，就触发request.abort()来取消

5. 取消成功

```javascript
// xhr.js
//请求中取消请求的代码
if (config.cancelToken) {
  // Handle cancellation
  config.cancelToken.promise.then(function onCanceled(cancel) {
    if (!request) {
      return;
    }
    request.abort();
    reject(cancel);
    // Clean up request
    request = null;
  });
}
```

# 总结

大概花了几天的时间把axios中主要模块的源代码分析了一遍，确实是收益良多。

作者水平有限（只是个刚学两年的小前端），如果有不正确的地方，还请大佬轻喷。

参考：[axios/axios: Promise based HTTP client for the browser and node.js (github.com)](https://github.com/axios/axios)

