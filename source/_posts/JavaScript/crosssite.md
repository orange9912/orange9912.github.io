---
title: 跨域方法笔记
date: 2021-06-21 19:55:13
tags:
categories:
- 前端
---

以前开发的时候在本地请求接口总是会遇到跨域问题，当时基础也不扎实，也不懂的怎么解决，现在趁机会总结一下跨域的概念和几种方法。

# 同源策略

**要知道什么是跨域，首先得了解一下浏览器的同源策略**

为了保证用户信息的安全，防止恶意网站窃取数据，所以有了同源策略。

## 什么情况算是同源？

- 协议相同
- 域名相同
- 端口相同

```markdown
https://zyczxq.com:443
```

对于这个网站来说，它的协议是==https://==，域名是：==zyczxq.com==，端口是443

如果协议、域名、端口有任意一个不相同，都算是非同源。

## 非同源不能干哪些事？

- Cookie、LocalStorage、IndexDB无法读取
- 无法获取DOM
- 不能发送AJAX请求

# 跨域的办法

跨域的方法非常多，这里只说几种我平时用到过的几种

## JSONP

JSONP，全称为JSON with padding，它的做法是通过动态添加一个脚本，因为页面调用JS文件不受同源策略限制，所以可以通过script标签进行跨域请求。

- 网页动态插入一个script元素，由它向目标发出请求，同时这个请求需要设置好回调函数，作为url的参数
- 服务器收到请求后，通过该参数获取到回调函数的名字，并将数据放在回调函数的参数中返回。
- 收到结果后因为是script标签，所以浏览器会执行

```javascript
//假设我们有一个回调函数foo，需要数据ip
function foo(ip);

//添加一个script标签，设置src为接口地址+callback=foo
```

然后服务器返回的内容写：

```javascript
foo({ip:'....'});
```

很明显，JSONP只能是GET请求，无法支持比较复杂的POST请求和其他请求。

## CORS

CORS是一个W3C标准，全称为Cross-origin resource sharing(**跨域资源共享**)。

它允许浏览器向跨源服务器发出XMLHttpRequest请求。

实现CORS通信的关键在于服务器。

在CORS之中分为两类请求：简单请求和非简单请求

### 什么算是简单请求？

请求方法是下述三种之一：

- HEAD
- GET
- POST

HTTP的头信息不超过以下几种：

- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

### 简单请求

对于简单请求，浏览器直接发出CORS请求，就是在头部信息中增加一个origin字段，表示请求的“源”（协议+端口+域名），服务器根据这个值，来决定是否同意本次请求。

如果服务器同意，在返回来的请求中，会多出几个头信息：

- Access-Control-Allow-Origin：表示允许的源，该字段是必须的。它的值要么是请求时`Origin`字段的值，要么是一个`*`，表示接受任意域名的请求。
- Access-Control-Allow- Credentials：该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。
- Access-Control-Expose-Headers

不同意的话，就看头信息有没有这几个字段就可以了

:::primary

实际上不同意HTTP返回的状态码也有可能是200

:::

如果要发送Cookie，开发者这边还必须指定xmlHttpRequest对象的withcredentials属性为true。同时需要注意，如果要发送cookie，Access-Control-Allow-Origin的值就不能设置为“*”

### 非简单请求

对于非简单请求，在正式通信之前，浏览器会发出一个预检请求，询问服务器是否允许该域等信息。

如果不同意，服务器会返回一个正常的回应，但不带有任何CORS相关字段。

### 优化非简单请求

优化OPTIONS预检请求的发送，CORS中Access-Control-Max-age可以设置缓存的时间，表示多少秒内不会对同一个非简单请求去发送预检请求，这样的话就能够减少重复多次发送options请求的往返时间。

# 设置代理服务器进行转发

比如说cli工具中的代理

## webpack中的proxy

在webpack配置项中加入如下：

```javascript
devServer: {
    port: 8000,
    proxy: {
      "/api": {
        target: "http://localhost:8080"//服务器地址
      }
    }
  },
```

同时，在请求的时候不要带url，现在已经是同源了

## vuecli2.x

在vuecli的配置文件中：

```javascript
proxyTable: {
  '/api': {
     target: 'http://localhost:8080',
  }
},
```

## vuecli3.x

我用的比较多的方式就是这种，在vue.config.js中添加

```javascript
module.exports = {
  devServer: {
    port: 8000,
    proxy: {
      "/api": {
        target: "http://localhost:8080",
        changeOrigin:true,
        pathReWrite:{
          '/api':""
        }
      }
    }
  }
};
```

# 还有利用iframe进行跨域的

这个暂时用的还比较少，感觉查资料难以理解，先留个坑，等具体用到的时候再补