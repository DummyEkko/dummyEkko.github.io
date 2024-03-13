---
title: "TTFB"
date: 2024-03-13T15:47:14+08:00
lastmod: 2024-03-13T15:47:14+08:00
author: ["lizhi"]
keywords: 
- 
categories: # 没有分类界面可以不填写
- performance
tags: # 标签
- performance
description: ""
weight:
slug: ""
draft: true # 是否为草稿
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

# 优化加载第一个字节所需时间

> 首字节时间 (TTFB) 是一项基础的网页性能指标，早于所有其他有意义的用户体验指标，例如首次内容绘制 (FCP) 和 Largest Contentful Paint (LCP)。这意味着，较高的 TTFB 值会延长其后面的指标的时间。
> 尽量将 TTFB 控制在 0.8 秒或更短

## Server-Timing响应头

后端使用Server-Timing响应头可以提供许多数据进行测量后端延迟时间， 例如，数据库读/写、CPU 时间、文件系统访问等）。

```js
// Get the serverTiming entry for the first navigation request:
performance.getEntries("navigation")[0].serverTiming.forEach(entry => {
    // Log the server timing data:
    console.log(entry.name, entry.description, entry.duration);
});
```

## 优化

1. 使用CDN

2. 避免多次网页重定向

3. 流式传输

4. Service Worker


