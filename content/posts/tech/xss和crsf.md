---
title: "WEB网络安全"
date: 2024-02-29T20:21:00+08:00
lastmod: 2024-02-29T20:21:00+08:00
author: ["lizhi"]
keywords: 
- web
categories:
- web # 没有分类界面可以不填写
tags:
- web # 标签
- XSS
- CSP
- CSRF
description: "xss、csrf攻击、CSP内容安全怎么运用"
weight:
slug: ""
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---

## XSS

> Cross-Site Scripting 跨站脚本攻击，攻击者将恶意代码注入到受害者所访问的网站中。这些恶意代码通常是`JavaScript`脚本，但也可以包括`HTML`和其他类型的代码。

### ⚠️XSS危害

如果页面被注入了恶意`JavaScript`脚本，恶意脚本都能做哪些事情：

- 可以窃取`Cookie`信息: 恶意`JavaScript`可以通过`document.cookie`获取`Cookie`信息，然后通过`XMLHttpRequest`或者`Fetch`加上`CORS`功能将数据发送给恶意服务器；恶意服务器拿到用户的 `Cookie` 信息之后，就可以在其他电脑上模拟用户的登录，然后进行转账等操作。
- 可以通过修改 `DOM` 伪造假的登录窗口，用来欺骗用户输入用户名和密码等信息。
- 还可以在页面内生成浮窗广告，这些广告会严重地影响用户体验。
- 可以监听用户行为: 恶意 `JavaScript` 可以使用`addEventListener`接口来监听键盘事件，比如可以获取用户输入的信用卡等信息，将其发送到恶意服务器。黑客掌握了这些信息之后，又可以做很多违法的事情。

### 🔐防范措施

1. 对输入脚本进行过滤或转码：例如`v-html`插入的文本。
2. 充分利用 `CSP（Content Security Policy）`

**渲染引擎**还有一个安全检查模块叫 `XSSAuditor`，是用来检测**词法安全**的。在分词器解析出来`Token`之后，它会检测这些模块是否安全，比如是否引用了外部脚本，是否符合 `CSP` 规范，是否存在跨站点请求等。如果出现不符合规范的内容,`XSSAuditor`会对该脚本或者下载任务进行拦截。

   `CSP` 有如下几个功能：
   - 限制加载其他域下的资源文件，这样即使黑客插入了一个 `JavaScript` 文件，这个 `JavaScript` 文件也是无法被加载的；
   - 禁止向第三方域提交数据
   - 禁止执行内联脚本和未授权的脚本。
3. 使用 HttpOnly 属性

## CSRF

> 跨站请求伪造（Cross-Site Request Forgery，简称 CSRF）
>  攻击者通过诱使受害者访问恶意网站或点击**恶意链接**，从而在受害者的浏览器上执行未经授权的操作。由于受害者的浏览器会自动携带已登录网站的 `Cookie`，因此攻击者可以利用这一特性伪造受害者的身份执行操作，如转账、修改密码或其他敏感操作。


CSRF 攻击有以下特点：

1. 攻击者无法直接获取受害者的数据，但可以伪造受害者执行操作。
2. 攻击者需要诱使受害者访问恶意网站或点击恶意链接。
3. 攻击者通常会利用受害者已登录的状态来进行攻击。


### 🔐防范措施

1. 使用 `SameSite Cookie` 属性：将 `Cookie` 的 `SameSite` 属性设置为 `"strict"` 或 `"lax"` 可以限制浏览器在跨站请求时发送 `Cookie`。这样，即使攻击者诱导受害者访问恶意网站，由于`Cookie`不会被发送，攻击者也无法伪造受害者的身份执行操作。
2. 验证请求来源：对于敏感操作，可以检查 `HTTP` 请求头中的 `Referer` 或 `Origin` 属性，确保请求来自于可信的来源。这可以防止来自恶意网站的跨站请求。
3. 使用 `CSRF` 令牌：为每个表单或敏感操作生成一个随机令牌（`Token`），并将该令牌与用户的会话绑定。当用户提交表单或执行操作时，需要验证令牌是否有效和匹配。这样攻击者就无法伪造有效的令牌，从而降低`CSRF`攻击的风险。
4. 使用双重认证：对于重要操作（如转账、修改密码等），可以使用双重认证（如短信验证码、邮箱验证码等）来验证用户的身份。


## CSP(Content Security Policy)

> CSP 的实质就是**白名单制度**，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。

### 两种开启`CSP`方法:

#### `HTTP` 响应头的`Content-Security-Policy`的字段

`HTTP` 响应头 `Content-Security-Policy` 允许站点管理者控制用户代理能够为指定的页面加载哪些资源。除了少数例外情况，设置的政策主要涉及指定服务器的源和脚本结束点。

例如：

```js
Content-Security-Policy: script-src 'self'; object-src 'none'; 
style-src cdn.example.org third-party.org; child-src https:
```

字段:

- **child-src**: 为 `Web Workers` 和其他内嵌浏览器内容（例如用 `<frame>` 和 `<iframe>` 加载到页面的内容）定义合法的源地址。
- **connect-src**: 限制能通过脚本接口加载的`URL`。
- **default-src**: 其他取指令提供备用服务`fetch directives`。
- **font-src**: 设置允许通过`@font-face` 加载的字体源地址。
- **frame-src**：设置允许通过类似 `<frame>` 和 `<iframe>` 标签加载的内嵌内容的源地址。
- **img-src**：限制图片和图标的源地址
- **manifest-src**：限制应用声明文件的源地址。
- **media-src**：限制通过 `<audio>`、`<video>` 或 `<track>` 标签加载的媒体文件的源地址。
- **object-src**：限制 `<object>` 或 `<embed>` 标签的源地址。
- **prefetch-src**: 指定预加载或预渲染的允许源地址。
- **script-src**: 限制 `JavaScript` 的源地址。
- **style-src**: 限制层叠样式表文件源。

[mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy)

#### 通过网页的`<meta>`标签

```html
<meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none'; style-src cdn.example.org third-party.org; child-src https:">
```