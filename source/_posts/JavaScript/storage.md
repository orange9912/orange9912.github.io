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

## Cookie相关字段

### Expires

这个值用于设置Cookie的过期时间，通常是一个绝对时间。如果不指定，则表示是一个临时会话的cookie。浏览器关闭时会失效

### Max-age

和Expires作用类似，但是他的值是一个相对时间（秒为单位）。并且他的优先级比expires更高。

当它的值为负数的时候表示是一个临时会话cookie。

### Domain

Domain指定cookie可以送达的域名。如果不指定，默认为当前文档访问地址的域名。

子域名一般可以使用指定为父域名的cookie。比如taobao.com下的cookie，a.taobao.com可以使用。

### Path

Path指定一个资源路径，请求的资源路径必须出现该路径才可以发送cookie。

如指定为/docs，则/docs/a可以使用，但/test不可以。

**Domain和Path共同指定了Cookie的作用域**。

### Http-only

设置 HTTPOnly 属性可以防止客户端脚本通过 document.cookie 等方式访问 Cookie，有助于避免 XSS 攻击。

### Secure

标记为 Secure 的 Cookie 只应通过被HTTPS协议加密过的请求发送给服务端。

### SameSite

它的值有三个：Strict,Lax,None.

在Strict下，会完全禁止第三方cookie。只在当前网页和请求目标网页的URL完全一致时才可发送请求网页的cookie

这个属性可以避免csrf，因为以往csrf在访问某一个网站时，这个网站向另外一个网站发送的请求，默认会发送请求的那个网站的cookie。

但是如果设置了samesite的strict，必须当前网页和请求网页的url完全一致，才能发送请求网页的cookie。

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

