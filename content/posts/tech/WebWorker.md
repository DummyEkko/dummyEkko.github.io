---
title: "WebWorker"
date: 2024-03-12T13:56:41+08:00
lastmod: 2024-03-12T13:56:41+08:00
author: ["lizhi"]
keywords: 
- 
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- 
description: ""
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

# Web Worker

> 就 JavaScript 而言，您通常只能在主线程上执行工作，但这只是默认操作。可以在 JavaScript 中注册和使用其他线程。允许在 JavaScript 中实现多线程的功能称为 Web Workers API。


## Web Worker 的限制

Web Worker 受到以下限制条件的约束：

- Web Worker 无法直接访问 DOM。
- Web Worker 可以通过消息传递流水线与 window 上下文进行通信，这意味着 Web Worker 可以通过某种方式间接访问 DOM。
- Web Worker 的作用域是 self，而不是 window。
- Web Worker 范围确实可以访问 JavaScript 基元和构造，以及 fetch 等

## 基本使用

### 启动、通信

```js
// Creates the web worker:
const myWebWorker = new Worker('/js/my-web-worker.js');

// Adds an event listener on the web worker instance that listens for messages:
myWebWorker.addEventListener("message", ({ data }) => {
  // Echoes "Hello, window!" to the console from the worker.
  console.log(data);
});
```

```js
// my-web-worker.js
self.addEventListener("message", () => {
  // Sends a message of "Hellow, window!" from the web worker:
  self.postMessage("Hello, window!");
});
```

## 使用场景

- 预取数据
- 处理加密

## demo

todo