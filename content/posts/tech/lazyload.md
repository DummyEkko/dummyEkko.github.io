---
title: "Lazyload"
date: 2024-02-29T22:14:41+08:00
lastmod: 2024-02-29T22:14:41+08:00
author: ["lizhi"]
keywords: 
- 懒加载 性能优化
categories: # 没有分类界面可以不填写
- performance
tags: # 标签
- performance
description: "什么是懒加载？为什么需要懒加载？怎么实现懒加载？"
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


## 懒加载

懒加载也叫**延迟加载**，延迟加载这一解决方案可以减少初始网页载荷和加载时间。

我们通常会延迟加载 **非关键资源**，在需要的时候再加载这些**非关键资源**。就图片来说，“非关键”通常指的是“屏幕外”。


## 为何要延迟加载图片或视频？

- 减少初始网页加载时间、初始页面重量以及系统资源用量。
- 用户可能永远不查看该内容，这样避免浪费流量，费处理时间、电量和其他资源（下载媒体资源后，浏览器必须对其进行解码，并在视口中呈现其内容）。

效果如：

## 延迟加载图片

### loading 属性

`Chrome` 和 `Firefox` 都支持通过 `loading` 属性进行延迟加载。

可选值：

- eager: 立即加载图像，不管它是否在可视视口（visible viewport）之外（默认值）
- lazy: 延迟加载图像，直到它和视口接近到一个计算得到的距离（由浏览器定义）。目的是在需要图像之前，避免加载图像所需要的网络和存储带宽。


如果浏览器不支持延迟加载，系统会忽略该属性，并照常立即加载图片。

判断是否支持 loading

```js
if ('loading' in HTMLImageElement.prototype) {
  // supported in browser
} else {
  // fetch polyfill/third-party library
}
```

### `scroll` 或 `resize`


```html
<body>
  <img loading="lazy" class='img-item' src="https://img.zcool.cn/community/011e0e5c231dcaa80121df9063b03f.jpg@1280w_1l_2o_100sh.jpg" data-original="https://img.zcool.cn/community/011e0e5c231dcaa80121df9063b03f.jpg@1280w_1l_2o_100sh.jpg" alt="">
  <img loading="lazy" class='img-item' src="" data-original="https://img.zcool.cn/community/01427a5c63c7dba801213f26fa8f1f.jpg@1280w_1l_2o_100sh.jpg" alt="">
  <img loading="lazy" class='img-item' src="" data-original="https://img.zcool.cn/community/0132fb60a7171011013f472074ee82.jpg@1280w_1l_2o_100sh.jpg" alt="">
  <img loading="lazy" class='img-item' src="" data-original="https://pic2.zhimg.com/v2-ac79ec66a08cdbbfd36c3f9e4f307077_1440w.jpg" alt="">
  <img loading="lazy" class='img-item' src="" data-original="https://img.zcool.cn/community/01702360a7171511013e3b7d2249ef.jpg@1280w_1l_2o_100sh.jpg" alt="">
  <img loading="lazy" class='img-item' src="" data-original="https://img.zcool.cn/community/01c9e05da91bcea8012163bae9a7ea.jpg@2o.jpg" alt="">
  <img loading="lazy" class='img-item' src="" data-original="https://img.zcool.cn/community/010c155dcec2b0a8012053c0802a12.jpg@3000w_1l_0o_100sh.jpg" alt="">
  <img loading="lazy" class='img-item' src="" data-original="https://pic2.zhimg.com/v2-b2ab50d677e6f8a9101dae446dd99acd_r.jpg" alt="">
  <img loading="lazy" class='img-item' src="" data-original="https://img.3dmgame.com/uploads/images/thumbpicfirst/20180824/1535099402_352803.jpg" alt="">
  <img loading="lazy" class='img-item' src="" data-original="https://img.zcool.cn/community/0193a55b9df41ba8012099c8aedcd0.jpg@3000w_1l_0o_100sh.jpg" alt="">
  <script>
    // 拿到可视觉区域的高度
    let viewHeight = document.documentElement.clientHeight;
    function lazyLoad(){
        // 拿到所有的img元素
        let imgs = document.querySelectorAll('img[data-original]');
        imgs.forEach(el=>{
            // getBoundingClientRect()专门获取容器的几何信息
            let rect = el.getBoundingClientRect()
            if(rect.top < viewHeight){
                // img元素自带一个构造函数，可以创建一个图片对象
                let image = new Image()
                // js专有写法dataset.original; = data-original
                image.src = el.dataset.original;
                image.onload = function(){
                    el.src = image.src
                }
                // 图片加载完毕就移除属性
                el.removeAttribute('data-original')
            }
        })
    }
    lazyLoad()
    // 添加滚动事件监听器
    document.addEventListener('scroll',lazyLoad)
</script>
```


### Intersection Observer

性能更佳且更高效

```html
<script>
document.addEventListener("DOMContentLoaded", function() {
  var lazyImages = [].slice.call(document.querySelectorAll("img.img-item"));

  if ("IntersectionObserver" in window) {
    let lazyImageObserver = new IntersectionObserver(function(entries, observer) {
      entries.forEach(function(entry) {
        if (entry.isIntersecting) {
          let lazyImage = entry.target;
          lazyImage.src = lazyImage.dataset.original;
          // lazyImage.srcset = lazyImage.dataset.srcset;
          // lazyImage.classList.remove("lazy");
          lazyImageObserver.unobserve(lazyImage);
        }
      });
    });

    lazyImages.forEach(function(lazyImage) {
      lazyImageObserver.observe(lazyImage);
    });
  } else {
}
</script>
```


### 总结

1. 可以通过 loading 和 `scroll` 或 `resize` 和 Intersection Observer 方式支持延迟加载
2. 延迟加载可以减少**启动期间的总体流量消耗**和**网络争用**，**减少图像解码所需的时间**来减少主线程上的处理。
3. 避免延迟加载可视区域内的，以免对 `lcp` 产生负面影响
