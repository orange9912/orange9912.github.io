---
title: 单页应用鉴权问题
date: 2021-07-20 18:24:01
tags:
categories:
- 前端
---

面试中被问到的时候，才发现这块是个知识盲区，于是专门花了一点时间去搜集资料，然后自己理解、再整理出来自己的一篇笔记。来总结一下一些鉴权的方法，还有它们的优缺点。

# 背景

Http本身是一个无状态的协议，这个不用多说了，单从网络连接上，服务器无法获知客户的身份。

但是，总不能进行与用户相关的操作就需要登录一次吧？所以就需要实现登录之后如何鉴权的问题。

# Cookie-Session实现鉴权

## Cookie

一般来说，在登录成功后，可以把某些用户信息存入客户端的Cookie中，每次发送请求时带着Cookie，这样，服务器就能够通过Cookie来知道对应的客户身份了。

**Cookie一般分为以下两种**

- session cookie。这种cookie会随着用户关闭浏览器而清除，不会被标记任何过期时间Expires或最大时限Max-Age
- Permanent cookie。和session cookie相反，在关闭浏览器后会被持久化存储。

:::primary

请求一个网站时只能发送该网站下的Cookie，而不能操作别的网站的Cookie

:::

cookie可以由JavaScript代码创建：

```javascript
document.cookie = 'my_cookie_name=my_cookie_value; expires=....'
```

也可以由服务端通过设置响应头创建

```http
Set-Cookie: my_cookie_name=my_cookie_value
```

浏览器会自动在每个请求中加入相关domain下的cookie。

## Session

Session是另外一种记录客户状态的机制，不同的是Cookie保存在客户端中，Session保存在服务器上（但其实也要利用到Cookie，后面的机制就是）。

客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上。这就是Session。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。

服务器用Session把客户的信息临时保存在服务器上，在用户离开服务器后，Session会被销毁。

但是这种机制有可能会对服务器造成相应的压力。

# Cookie-Session认证

一个经典的场景就是使用Cookie存储一个SessionID（SessionID由服务端管理，进行创建和计时）。

通过验证cookie和SessionID，服务器便能标记一个用户的访问信息。

认证过程大概是这样的：

1. 用户使用密码、或者别的方式登录系统。
2. 服务端验证后，创建一个Session信息，并且将SessionID存到cookie，发送回浏览器
3. 下次客户端再发起请求，自动带上cookie信息，服务端通过cookie获取Session信息进行校验。

## 缺点及安全隐患

- 只能在web端使用，在APP中，不能使用cookie就不能使用了。
- 如果服务器做了负载均衡，下一个操作到了另外一台服务器的时候可能就会丢失
- cookie不能跨域
- XSS攻击下，cookie中的认证信息可能会被盗取。
- 可能会被CSRF所利用。cookie的同源策略只是限制不同源的网站读取其他网站的cookie，但如果利用CSRF攻击，同源策略并不能限制他。

## 安全设置

- HttpOnly cookie。在浏览器端，JavaScript没有读取cookie的权限，防止XSS。
- Secure cookie，只有在特定安全通道（通常是HTTPS下）下，才传递cookie
- SameSite cookie：在跨域情况下，相关cookie无法被请求携带，主要是为了防止CSRF。

具体还有一些，在我那篇XSS和CSRF笔记中也有提到，就不细说了

# 改进版的Cookie-Session认证

把上述的改进为：

- 不用cookie做客户端验证了，用其他的。web下使用localStorage，APP中使用客户端数据库，这样也可以避免了CSRF（不会自动发送）。
- 服务端不使用Session，而是把Session信息拿出来存到数据库之中。

认证过程：

1. 用户登录
2. 服务器经过验证，把认证信息存入数据库，并把key值发送给浏览器
3. 客户端拿到key值后存localStorage
4. 下次客户端再次请求后，把key值附加到header或者请求体内。
5. 服务端通过获取的key来从数据库中读取信息。

++这样做其实还有一个好处，在用户登录后存key进入localStorage的时候，可以利用监听storage事件来实现其他同源页面也保持登录状态++。

# 基于JWT的token验证

JWT（JSON Web Token），即用于验证的信息是由JSON数据格式组成，这个Token是一个字符串。

上述的认证，还是需要服务端和客户端这边维持一个状态信息，但是基于JWT的token认证方案可以省去这个过程。

基于Token的身份验证是无状态的，我们不将用户信息存在服务器中。

## JWT的组成

- header（消息头）
- payload（消息体，储存用户id，用户角色等）+过期时间（可选）
- signature（签名）

JWT是JSON格式的数据，前两部分就是JSON数据，第三部分signature是基于前两部分header和payload生成的签名。

前两部分分别通过Base64URL算法生成两组字符串，再和signature结合，三部分结合后通过“ **.** ”分割，就是最终的token（如xxxxxx.yyyyy.zzzz）

:::primary

签名的作用就是保证**JWT没有被篡改过**。签名基于前两个编码过的字符串，以某种不可逆算法加密（如HMAC算法）比较。

:::

## 认证过程

1. 用户登录系统
2. 服务端验证，将认证信息通过指定的算法（如HS256）进行加密，将加密的结果发送到客户端
3. 客户端拿到返回的token，存到某个地方（可以是localStorage或SessionStorage或cookie，后面比较哪种好）
4. 下次客户端发送的时候，将token附加到header中（放url中也可以，但是放在header中更安全）
5. 服务端获取header中的token，通过相同的算法对token中的信息进行验证，验证token中的signature和payload是否等同，就可知道是否被中间人更改。

## 存储位置

一般JWT在客户端的存储有三种：

- localStorage
- SessionStorage
- cookie（不能设置Http only）

一般来说比较推荐存放在session cookie中（上面提到的临时cookie），前两种存在跨域读取限制。

:::primary

就算是存临时cookie，也和上面传统的cookie-session机制是不一样的，我们只是存在那，并不是利用它去鉴权，实际上还是需要我们手动将cookie加到header上

:::

当然其实也可以存在localStorage，不过有几点不好就是：

- 当用户关闭浏览器后，JWT仍然会被存储在本地存储，除非手动更新或清理。
- 任何JavaScript都可以轻易获取本地存储的内容
- 无法被Web Worker使用

并不是说一定要存在临时cookie，结合具体场景选择。

## 优点

- 使用json作为数据传输，轻量级。
- 服务端无需保存状态，节省服务端资源。也方便横向扩展
- 可以实现单点登录
- 防CSRF（token的优势）

## 总结

- token获取到后需要保存到某个地方待下次使用，最好是临时cookie，或者localstorage、sessionStorage
- token是有有效期的。
- localStorage和sessionStorage的跨域限制较为严格。
- token每次请求中都会编码到请求中，注意token大小。
- 存储敏感信息时记得加密

# 混合JWT和Cookie进行鉴权

晚点填坑

# OAuth认证

比较常见的就是用微信登录、QQ登录或其他比较权威的网站开放的API来实现用户登录。

:::primary

OAuth允许用户提供一个令牌，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的网站（例如，视频编辑网站)在特定的时段（例如，接下来的2小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。这样，OAuth让用户可以授权第三方网站访问他们存储在另外服务提供者的某些特定信息，而非所有内容。——维基百科

:::

# 参考信息

- [说一说几种常用的登录认证方式，你用的哪种 - 风的姿态 - 博客园 (cnblogs.com)](https://www.cnblogs.com/fengzheng/p/8416393.html)
- 《前端核心开发知识进阶》第33章，单页应用鉴权设计

- [前端应该学习的 Token 登录认证知识 (qq.com)](https://mp.weixin.qq.com/s/93FzBigxugfRKI5qN76mWA)
- [JWT 超详细分析 | Laravel China 社区 (learnku.com)](https://learnku.com/articles/17883?order_by=vote_count&)

