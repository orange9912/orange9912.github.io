---
title: 前端存储笔记
date: 2021-06-14 17:06:26
tags:
categories:
- 前端
---

# Cookie

## Cookie是什么？

cookie是一些数据，用于存储web端的用户数据。它用于解决“[如何记录客户端的用户信息]{.pink}“

**当浏览器从服务器上请求web页面时，属于该页面的cookie会被添加到该请求中。服务端通过这种方式来获取用户的信息。**

## Cookie的特点

- cookie始终在同源的http请求中携带，会在浏览器和服务器间来回传递。
- cookie有路径的概念，可以把它限制在某个路径下（最好能子域可见就不要主域）
- 一个cookie的大小一般在4K以内，具体看浏览器。
- 不同浏览器对一个域名下cookie的数量限制不一样，像ie6限制最多50个，firefox3.6限制150个

## Cookie使用方法

Cookie以名/值对的形式存储，如下所示：

username=Orange; expires=......; path=/

### 设置Cookie

```javascript
function setCookie(cname,cvalue,exdays)
{
  var d = new Date();
  d.setTime(d.getTime()+(exdays*24*60*60*1000));
  var expires = "expires="+d.toGMTString();
  document.cookie = cname + "=" + cvalue + "; " + expires;
}
```

### 读取指定的Cookie

```javascript
function getCookie(cname)
{
  var name = cname + "=";
  var ca = document.cookie.split(';');
  for(var i=0; i<ca.length; i++) 
  {
    var c = ca[i].trim();
    if (c.indexOf(name)==0) return c.substring(name.length,c.length);
  }
  return "";
}
```

# LocalStorage和SessionStorage

## 相同点

- 两者都属于浏览器端存储，前者为本地存储，后者为会话存储

```javascript
window.localStorage //直接访问localStorage
window.sessionStorage
```

- 两者的操作语法基本一致

```javascript
localStorage.setItem("name","orange");
localStorage.getItem("name");
localStorage.removeItem("name");
localStorage.clear();

//SessionStorage的操作语法一样。
```

- 两者都遵循同源策略，跨域无法访问。
- 两者各自拥有5MB的存储空间（[实际这个大小只是建议，不同浏览器的限制不一样，chrome下确实是5M左右，但像Safari中本地存储只有2551K，会话存储却超过10M]{.pink})，并且只能保存字符串类型的数据。

## 不同点

- 本地存储本身不会失效，除非手动删除。会话存储仅在关闭浏览器窗口之前有效（仅在会话页面失效时失效，即刷新页面和回退、恢复等时不会失效）
- 有效范围不同。不跨域情况下本地存储可以跨页签生效，而会话存储只在当前页签有效。

其实还有一个前端数据库IndexDB，不过现在的我暂时用不上，先不去看，等要用上的时候再去学习总结。