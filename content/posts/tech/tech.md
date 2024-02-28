---
title: "字体最佳实践"
date: 2024-02-27T19:25:21+08:00
author: ["lizhi"]
keywords: 
- 
categories: 
- 
tags: 
- tech
- font
- html
- web
description: "介绍字体使用到优化"
weight:
slug: ""
draft: false # 是否为草稿
comments: true
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://www.sulvblog.cn/posts/blog/how_to_write_a_blog/1.png" #图片路径例如：posts/tech/123/123.png
    caption: "" #图片底部描述
    alt: ""
    relative: false
---

### 字体加载

#### @font-face

`@font-face` 用来申明网络字体，它至少要声明将用于引用**字体的名称**，并指明相应**字体文件的位置**。

```css
@font-face {
  font-family: "Open Sans";
  src: url("/fonts/OpenSans-Regular-webfont.woff2") format("woff2");
}

h1 {
    /* 这个时候才会去下载 font-face 声明的字体 */
  font-family: "Open Sans" 
}
```

`@font-face` 声明本身不会触发**字体下载**, 只有通过在网页上使用的样式引用某个字体时，系统才会下载该字体


`unicode-range` 使用介绍：

todo


### 字体渲染

当面临尚未加载的网络字体时，浏览器将面临一个困境：是否应该在网络字体到达之前暂停渲染文本？还是应该在网页字体到达之前以后备字体呈现文本？

默认情况下，如果关联的网页字体尚未加载，基于 `Chromium` 和 `Firefox` 的浏览器将会阻止文本呈现，最长可持续**3秒**钟; `Safari` 将无限期地阻止文本呈现。

> 概括就是**文本不可见在字体还没加载前**

可以使用 `font-display` 属性来配置此行为。

#### `font-display`

`font-display` 属性决定了一个 `@font-face` 在不同的下载时间和可用时间下是如何展示的.

`font-display` 可选值：

 | 值    | 屏蔽期 | 交换期 |
 | :-----| ----: | :----: |
 | auto  | 因浏览器而异 | 因浏览器而异 |
 | block | 2-3 秒	 | 无限 |
 | swap | 0 毫秒	 | 无限  |
 | fallback | 100 毫秒	 | 3 秒  |
 | optional | 100 毫秒	 | 无  |

- 屏蔽期：屏蔽期从浏览器请求网页字体时开始计算。在屏蔽期间，如果网页字体不可用，相应字体将以不可见的后备字体呈现，因此用户将看不到相应文本。如果在屏蔽期结束后该字体不可用，它将以后备字体呈现。
- 交换期：交换期在屏蔽期之后。如果网页字体在交换期内可用，则会被“交换”。

#### 如何设置合理的 `font-display` 

##### 如果性能是我们的头等大事

那么使用 font-display: optional。

优点: 文本渲染的延迟不超过 **100** 毫秒, 并且能够保证不会发生与字体交换相关的布局偏移(优化`CLS`)

缺点: 如果网页字体延迟显示，将无法使用网页字体。

##### 快速显示文本是首要任务，仍想确保使用网页字体

使用 font-display: swap

优点: 先会以默认字体展示，等网页字体加载完成后，进行交换。

缺点：会导致布局偏移。

总结: 尽早加载字体，例如 `link preload` 进行预加载。


##### 确保文本以网页字体显示是首要任务

使用 `font-display: block`。

缺点：字体没加载完文本将不可见。

总结: 尽早加载字体，例如 `link preload` 进行预加载。


### 对字体优化总结

#### 字体加载时机

通过上面文章可以知道 只有读到网络字体被使用时，才会触发`@font-face`下载逻辑。

优化方案：

- **关键字体**通过 `preload` 进行加载。
  - 前提条件：如果明确字体的情况下。
  - 注意点：字体文件必须通过 `CORS` 连接，预加载情况下，`preload` 会忽略 `unicode-range` 声明，如果使用得当，应仅用于加载一种字体格式。
- 预先连接到关键的第三方源 `link rel="preconnect"`
  - 前提条件：可能多个字体，不明确哪个是关键，
  - 注意点：字体文件必须通过 `CORS` 连接

#### 字体加载速度

字体加载速度主要是网络优化，一个是字体大小一个是合理的网络传输

- 采用 `CDN` 托管字体和 `HTTP/2`。
- 使用 `WOFF2` （广泛的浏览器支持，并提供最佳的压缩效果， WOFF2 使用 Brotli，因此其压缩效果比 WOFF 高 30%）
- 子集 通过使用 `unicode-range` 会告知浏览器某种字体可用于哪些字符, 将一个大的字体分成几个子集，需要时加载子集从而减小体积


### 补充

#### 为什么 字体的预加载需要通过 `CORS` 连接？

todo




