---
title: cacheStorage
tags:
  - h5
  - cache
  - storage
  - cacheStorage
categories:
  - h5
  - cache
  - storage
top: true
abbrlink: f688fea9
date: 2019-11-02 16:56:04
---
## 前言
h5中本地存储除了LocalStorage、SessionStorage外还有一个鲜为人知的CacheStorage，顾名思义，其就是用来做缓存的storage，虽然通常很多东西都用localStorage和sessionStorage去进行存储，但用作缓存的话还是cacheStorage最佳。

## 介绍
CacheStorage是浏览器提供的用作缓存存储的一个API类，通常配合service worker使用，但独立在主进程中使用也是可以的，其提供的丰富的api进行缓存的设置，清除，下载操作。当然，新的API都有其兼容性问题，详细兼容性可以[查看](https://caniuse.com/#search=cacheStorage)。

## API
1. `open(cacheName:string):Promise`：打开或创建某个缓存区，是其他方法的执行前提。
2. `add(path:string)`：下载某个资源并缓存，通常用来做预缓存
3. `addAll(paths:Array<string>)`：批量下载资源并缓存，通常用来做预缓存
4. `delete(key:string)`：删除某个缓存
5. `match(requestInfo:request|string)`：查找、命中缓存
6. `put(requestInfo:request|string，response: Response)`：添加缓存


## Examples
1. 预缓存
```typescript
window.caches.open("cache_name").then(cache=>{
    // 预缓存
    cache.addAll([
        "/a.css"
    ]);
    cache.add("/b.css");
});
```
2. 命中
```typescript
window.caches.open("cache_name").then(cache=>{
    return cache.match("/a.css");
});
```
3. 删除
```typescript
window.caches.open("cache_name").then(cache=>{
    cache.delete("/a.css");
});
```
4. 添加
```typescript
window.caches.open("cache_name").then(cache=>{
    cache.put("/a.css",new Response());
});
```
