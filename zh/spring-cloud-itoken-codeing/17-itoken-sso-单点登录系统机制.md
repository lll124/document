# 单点登录系统机制

## HTTP 无状态协议
`web` 应用采用 `browser/server` 架构，`http` 作为通信协议。`http` 是无状态协议，浏览器的每一次请求，服务器会独立处理，不与之前或之后的请求产生关联，这个过程用下图说明，三次请求/响应对之间没有任何联系

![sso](../../static/zh/spring-cloud-itoken-codeing/17-001.png)

但这也同时意味着，任何用户都能通过浏览器访问服务器资源，如果想保护服务器的某些资源，必须限制浏览器请求；要限制浏览器请求，必须鉴别浏览器请求，响应合法请求，忽略非法请求；要鉴别浏览器请求，必须清楚浏览器请求状态。既然 http 协议无状态，那就让服务器和浏览器共同维护一个状态吧！这就是会话机制

## 会话机制
浏览器第一次请求服务器，服务器创建一个会话，并将会话的 id 作为响应的一部分发送给浏览器，浏览器存储会话 id，并在后续第二次和第三次请求中带上会话 id，服务器取得请求中的会话 id 就知道是不是同一个用户了，这个过程用下图说明，后续请求与第一次请求产生了关联

![sso](../../static/zh/spring-cloud-itoken-codeing/17-002.png)

服务器在内存中保存会话的两种方式

- 请求参数
- Cookie
将会话 `id` 作为每一个请求的参数，服务器接收请求自然能解析参数获得会话 `id`，并借此判断是否来自同一会话，很明显，这种方式不靠谱。那就浏览器自己来维护这个会话 id 吧，每次发送 http 请求时浏览器自动发送会话 `id`，`cookie` 机制正好用来做这件事。`cookie` 是浏览器用来存储少量数据的一种机制，数据以 `key/value` 形式存储，浏览器发送 `http` 请求时自动附带 `cookie` 信息

`tomcat` 会话机制当然也实现了 `cookie`，访问 `tomcat` 服务器时，浏览器中可以看到一个名为 `JSESSIONID` 的 `cookie`，这就是 `tomcat` 会话机制维护的会话 `id`，使用了 `cookie` 的请求响应过程如下图

![sso](../../static/zh/spring-cloud-itoken-codeing/17-003.png)

## 登录状态
有了会话机制，登录状态就好明白了，我们假设浏览器第一次请求服务器需要输入用户名与密码验证身份，服务器拿到用户名密码去数据库比对，正确的话说明当前持有这个会话的用户是合法用户，应该将这个会话标记为“已授权”或者“已登录”等等之类的状态，既然是会话的状态，自然要保存在会话对象中，`tomcat` 在会话对象中设置登录状态如下
```
HttpSession session = request.getSession();
session.setAttribute("isLogin", true);
用户再次访问时，tomcat 在会话对象中查看登录状态

HttpSession session = request.getSession();
session.getAttribute("isLogin");
实现了登录状态的浏览器请求服务器模型如下图描述
```
![sso](../../static/zh/spring-cloud-itoken-codeing/17-004.png)

每次请求受保护资源时都会检查会话对象中的登录状态，只有 `isLogin=true` 的会话才能访问，登录机制因此而实现