---
title: 一些封装的工具函数
date: 2021-05-30 16:23:15
tags:
categories:
- 前端
---

# 前言

趁着有空，把自己平时使用的一些函数抽离出来做个记录。

# 防抖

防抖的思想是：==当某个事件处理函数被短时间内高频出发时，只有触发完一定时间没有再触发才会执行==。

可以用于输入框监听等高频触发事件的地方。

```javascript
/**
* @params {function} fn 需要防抖的函数
*	@params {number} time 多久没有触发才执行
*/ 
function debounce(fn,time){
  var timer = null;
  return function(){
    clearTimeout(timer);
    
    timeout = setTimeout(() => {
      fn.apply(this,arguments);
    },time);
  }
}
```

# 节流

使用场景同防抖，只不过核心思想有些不一致。

节流是某个事件处理函数被短时间内高频触发时，一定时间内只执行一次，可以降低执行频率。

```javascript
/**
*	@params {function} fn 
*	@params {number} time
*/
function throttle(fn,time){
  let canRun = true;
  return function(){
    if(!canRun) return;
    canRun = false;
    setTimeout(() => {
      fn.apply(this,arguments);
      canRun = true;
    },time);
  }
}
```

# 只执行一次

```javascript
function once(fn) {
	let result, executed;
  return function () {
    if (!executed) {
      executed = true
      result = fn.apply(this, arguments)
    }
    return result
  }
}
```

# cookie相关

## 获取cookie

```javascript
function getCookie(name){
  var name = name + '=';
  var ca = document.cookie.split(';');
  for(let i = 0;i<ca.length;i++){
    let c = ca[i].trim();
    if(c.indexOf(name) == 0)return c.substring(name.length,c.length);
  }
  return "";
}
```

## 设置cookie

```javascript
function setCookie(name,value,day){
  var d = new Date();
  d.setTime(d.getTime()+(day*24*60*60*1000));
  let expires = "expires=" + d.toGMTString();
  document.cookie = name + '=' value + '; ' + expires;
}
```

# call函数

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

# apply函数

```javascript
Function.prototype.apple = function(context){
  if(typeof this !== 'function'){
    throw new TypeError('error');
  }
  context = context || window;
  context.fn = this;
  if(arguments[1]){
    result = context.fn(...arguments[1]);
  }else{
    result = context.fn();
  }
  delete context.fn;
  return result;
}
```

# bind函数

```javascript
Function.prototype.myBind = function(context){
  if(typeof this !== 'function'){
    throw new TypeError('error');
  }
  const _this = this;
  const args.= [...arguments].slice(1);
  return function F(){
    if(this instanceof F){
      return new _this(...args,...arguments);
    }
    return _this.apply(context,args.concat(...arguments));
  }
}
```

# 获取某个元素的style

```javascript
function getStyle(obj,attr){
	if(obj.currentStyle)return obj.currentStyle[attr];
  else return getComputedStyle(obj,false)[attr];
}
```

