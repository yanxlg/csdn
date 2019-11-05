---
title: 图片懒加载
abbrlink: f0e261b6
date: 2019-11-04 16:10:35
categories:
    - h5
    - lib
tags:
    - h5
    - lib
    - lazyload
---
## 前言
随着H5页面图片资源的丰富，首屏页面加载效率受到明显限制，因此优化图片加载成为一个重要的需求，而图片的优化，除了压缩、缓存外，还有一个重要的方式是`懒加载`。本文主要介绍`懒加载`的实现方案及思路。

## 简介
`懒加载`从字面看，`就是我比较懒，你催我要的时候我才加载，不然我就罢工休息`，因此主要的做法就是一开始不显示图片，在需要的时候我才加载并显示图片。什么时候才是需要的时候呢？从UI交互上讲就是图片所在的元素容器显示在viewport上的时候。因此设计方案如下：

## 实现方案
1. 默认情况下`img`标签不设置`src`属性，仅做占位使用。
2. 当该`img`元素被移动到可视区域`viewport`中时设置其`src`属性为需要访问的图片地址。
3. 图片显示后回收对该图片的监听，释放资源

第一步比较简单，在js或者模板中，生成`img`时，使用`data-src`属性替代`src`属性即可，关键是第二步和第三步，如何去监听及释放。
从掌握的前端知识中可以发现，有两种方式可以实现上面的逻辑：

## 方案一：Observer
详细了解浏览器{% post_link h5/observer Observer %}的用法，我们可以发现有一个`Intersection Observer`能够观察元素位置，同时支持取消观察的API，满足当前需求，因此可以实现为：
```html
<img data-lazyload data-src="img-1.jpg">
<img data-lazyload data-src="img-2.jpg">
<img data-lazyload data-src="img-3.jpg">
<!-- more images -->
```
```typescript
const callback = (entries, observer)=> {
    entries.forEach(entry => {
        entry.target.src = entry.target.dataset.src;
        observer.unobserve(entry.target);// 取消观察
    });
};
const lazyLoadObserver = new IntersectionObserver(callback);
document.querySelectorAll('[data-lazyload]').forEach(img => { lazyLoadObserver.observe(img) });
```
到此，基本功能已经实现了，可以看到代码非常简单，配合相关的框架生成html的方式使用即可，但是经过测验会发现，图片在滚动到`viewport`中时，会存在较长时间的无图现象，对于用户感知来说，不太友好，因此需要将观察区域进行延伸，修改为如下：
```typescript
const callback = (entries, observer)=> {
    entries.forEach(entry => {
        entry.target.src = entry.target.dataset.src;
        observer.unobserve(entry.target);// 取消观察
    });
};
const lazyLoadObserver = new IntersectionObserver(callback,{rootMargin:"-50px 0 -50px 0"});
document.querySelectorAll('[data-lazyload]').forEach(img => { lazyLoadObserver.observe(img) });
```
添加`rootMargin`参数，将观察区域相对于`root`进行放大，滚动时，当图片元素具体可视区域上或者下50px的位置时即可触发观察回调，此时立即进行图片加载，在一定的程度上，当用户将该图片滚动到可视区域中时，可能已经加载完成；具体的偏移值需要根据图片服务情况及用户网络情况确定。
将上述代码用于项目中会发现，部分环境`IntersectionObserver`报错，这是由于兼容性造成，{% post_link h5/observer Observer %}中详细介绍了其兼容性并提供了`polyfill`方案，其`polyfill`和下面第二种方案类似。

## 方案二：onscroll + getComputedStyle
传统js中判断元素与可视区域之间的位置关系用`getBoundingClientRect`API 来实现，因此，我们可以使用`轮询判断`的方式去进行元素探测，使用`onscroll`+`getBoundingClientRect`实现该功能：
```typescript
const imgs = document.querySelectorAll('[data-lazyload]');
const callback = ()=> {
    imgs.forEach((img:HTMLImageElement) => {
        if(!img.src){
            const position = img.getBoundingClientRect();
            if(position.bottom>=-50||position<=window.innerHeight+50){
                img.src = img.dataset.src;
            }
        }
    });
};
window.addEventListener("scroll",callback);
```
代码相当简单，但是`并不实用`，主要存在以下问题：
    - `onscroll`触发太过频繁，每次触发都去进行dom List的遍历，开销很大
    - `getBoundingClientRect`容易造成浏览器重绘
    - `onscroll`在IOS中并不会频繁触发，只在开始和结束触发一次
    
根据以上问题，需要对此进行优化：
    - 通过`throttle`或`debounce`优化`onscroll`的频繁触发
    - 当`onscroll`触发不正常时，即判断IOS环境，使用`requestAnimationFrame`无限轮询代替`onscroll`
优化代码此处不作提现，经过上面的优化，该方案可用性才明显提升。

## 注意
- image 初始不要设置src属性，即使设置为`""`或者`undefined`也不要，可能会造成重复载入，不同浏览器行为不一致

## 其他优化
- 通过滚动距离进行节流
- 滚动暂停时才开始触发探测回调

## 总结
常见的一下lazyload插件主要采用的是方案二的方式，只是或多或少在此基础上做了进一步优化，但是其效率仍然比不上方案一，建议使用方案一，实在不兼容的情况下使用方案二。
