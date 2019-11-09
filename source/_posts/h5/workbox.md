---
title: workbox
abbrlink: d5dcded1
date: 2019-11-02 16:54:35
tags:
    - h5
    - service worker
---
## 前言
`workbox` 是 GoogleChrome 团队推出的一套 Web App 静态资源和请求结果的本地存储的解决方案，该解决方案包含一些 Js 库和构建工具，在 Chrome Submit 2017 上首次隆重面世。而在 `workbox` 背后则是 `Service Worker` 和 `Cache API` 等技术和标准在驱动。在 `Workebox` 之前，GoogleChrome 团队较早时间推出过 `sw-precache` 和 `sw-toolbox` 库，但是在 GoogleChrome 工程师们看来，`workbox` 才是真正能方便统一的处理离线能力的更完美的方案，所以停止了对 `sw-precache` 和 `sw-toolbox` 的维护。那`workbox`能解决什么问题呢？
在`service worker`中，如果我们要拦截并代理所有的请求，需要我们手动去维护一套缓存列表。但是现在前端开发，多数用`webpack`、`gulp`、`grant`来构建前端的代码，导致我们的文件名可能会经常发生，这个时候，特别是中大型的多页应用，缓存列表的内容可能会非常多，手动维护就显得非常麻烦，维护成本也变得很高。
这个时候，`workbox`的横空出世，就是为了解决上面的问题。

## 特性
- 不管你的站点是哪种方式构建的，都可以实现离线缓存的效果；
- 自动管理好缓存列表，包括更新、同步、删除旧的缓存等；
- 配置简单却不失灵活，可以完全自定义相关需求（支持 Service Worker 相关的特性如 Web Push, Background sync 等）。
- 针对各种应用场景的多种缓存策略。

## 使用
1. Register
```typescript
if ('serviceWorker' in navigator) {
    window.addEventListener('load', function() {
        navigator.serviceWorker.register(`./build.sw.js`)
        .then(function(registration) {
            // Registration was successful
            console.log('[success] register ')
        }, function(err) {
            // registration failed :(
            console.log('[fail]: ', err);
        });
    });
}
```
注册代码与最终生成的sw.js文件名有关，也可查看{% post_link h5/serviceWorkers Service Workers %}了解更多，通常在`webpack`、`gulp`、`grant`相关插件中注册代码会自动生成。

2. Sw
```typescript
// 首先引入 Workbox 框架
importScripts('https://storage.googleapis.com/workbox-cdn/releases/3.3.0/workbox-sw.js');
// Control all opened tabs ASAP
workbox.clientsClaim();
// workbox.core.clientsClaim(); // workbox@4.0+

workbox.skipWaiting();
// workbox.core.skipWaiting(); // workbox@4.0+

// 注册成功后要立即缓存的资源列表
workbox.precaching.precacheAndRoute([
  {
    "url": "css/index.css",
    "revision": "835ba5c3"
  },
  {
    "url": "images/xxx.png",
    "revision": "b1537bfs"
  },
  {
    "url": "index.html",
    "revision": "b331f695"
  },
  {
    "url": "js/index.js",
    "revision": "4d562866"
  }
]);

// 缓存策略
workbox.routing.registerRoute(
  new RegExp(''.*\.html'),
  workbox.strategies.networkFirst()
);

workbox.routing.registerRoute(
  new RegExp('.*\.(?:js|css)'),
  workbox.strategies.cacheFirst()
);

workbox.routing.registerRoute(
  new RegExp('https://your\.cdn\.com/'),
  workbox.strategies.staleWhileRevalidate()
);

workbox.routing.registerRoute(
  new RegExp('https://your\.img\.cdn\.com/'),
  workbox.strategies.cacheFirst({
    cacheName: 'example:img'
  })
);
```
- 首先引入workbox框架，可以相对路径引用本地资源，也可以引用服务器资源，可以将其缓存，但是sw.js`千万不能缓存`。
- 通常第二步调用`clientsClaim`获取所有客户端管理权限，`workbox@4.0+`以后该方法被放到了`workbox.core`中
- 第三步调用`skipWaiting`使新的sw立即生效，该API按照需要进行调用
- 第四步使用`precacheAndRoute`预缓存一些资源，预缓存列表通常是由`webpack`、`gulp`、`grant`插件生成，因为其revision和文件版本相关，甚至每次编译文件名会发生改变。
- 第五步使用`registerRoute`配置runtime缓存

## API
1. setConfig
```typescript
workbox.setConfig({
    debug:true,// 是否可调试
    modulePathPrefix:"/workbox-v3.6.3",// 重写workbox库位置，用于本地库
})
```
2. setCacheNameDetails
```typescript
workbox.core.setCacheNameDetails({
    "prefix": "app", // 缓存表名前缀
    "suffix": "v1",// 缓存表后缀，通常用来指定版本号
    "precache": "precache",// 预缓存表名
    "runtime": "runtime"// runtime缓存表名
})
```
3. cleanupOutdatedCaches：清除过期的预缓存，根据revision及文件名进行清除，不满足条件的全部清除
```typescript
workbox.precaching.cleanupOutdatedCaches()
```
4. registerNavigationRoute(cachedAssetUrl, options:{cacheName:string;blacklist:RegExp[];whitelist:RegExp[]})：注册导航请求路由，会返回预缓存文件，常用于`App Shell`中
```typescript
workbox.routing.registerNavigationRoute('/index.html');
```
5. registerRoute(capture:string|RegExp|Function, handler, method)：轻松注册`string`、`RegExp`、`func`路由
```typescript
workbox.routing.registerRoute(/\/api\//, workbox.strategies.networkFirst());
```
6. setCatchHandler(handler:(response:Promise<any>)=>void)：路由抛错回调函数

7. setDefaultHandler(handler)：定义没有路由匹配时，默认处理程序，默认无匹配则直接请求网络，跟无sw一样

8. unregisterRoute(route)：销毁路由，一般不会使用到


## 缓存策略
缓存策略是指对于匹配到的路由，采取何种方式进行缓存。 `workbox`提供了两种配置缓存策略的方式
- 通过 `workbox.strategies` API 提供的 缓存策略。
- 提供一个自定义返回带有返回结果的 Promise 的回调方法。

以下介绍`workbox`默认提供的几种缓存策略，包含有五种，分别是：
- Stale While Revalidate
- Network First
- Cache First
- Network Only
- Cache Only

1. Stale While Revalidate
这种策略的意思是当请求的路由有对应的 Cache 缓存结果就直接返回，在返回 Cache 缓存结果的同时会在后台发起网络请求拿到请求结果并更新 Cache 缓存，`如果本来就没有 Cache 缓存的话，直接就发起网络请求并返回结果`。 使用方式如下：
```typescript
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.staleWhileRevalidate()
);
```
注意：对于没有使用precache预缓存的资源，会直接发起网络请求并返回结果，与上面强调的一句规则匹配。因此staleWhileRevalidate存在一定的问题，它在请求后不会将其缓存到Cache中，一定程度上===`Network Only`

2. Network First：具体实现方式可以参考{% post_link h5/serviceWorkers Service Workers %}
这种策略就是当请求路由是被匹配的，就采用网络优先的策略，也就是优先尝试拿到网络请求的返回结果，如果拿到网络请求的结果，就将结果返回给客户端并且写入 Cache 缓存，如果网络请求失败，那最后被缓存的 Cache 缓存结果就会被返回到客户端 使用方式如下：
```typescript
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.networkFirst()
);
```

3. Cache First
这个策略的意思就是当匹配到请求之后直接从 Cache 缓存中取得结果，如果  缓存中没有结果，那就会发起网络请求，拿到网络请求结果并将结果更新至 Cache 缓存，并将结果返回给客户端。
```typescript
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.cacheFirst()
);
```

4. Network Only
比较直接的策略，直接强制使用正常的网络请求，并将结果返回给客户端，这种策略比较适合对实时性要求非常高的请求。
```typescript
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.networkOnly()
);
```

5. Cache Only `[danger]`
这个策略也比较直接，直接使用 Cache 缓存的结果，并将结果返回给客户端，这种策略比较适合一上线就不会变的静态资源请求。
```typescript
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.cacheOnly()
);
```

> `注意：`
> workbox@4.0+ 以后workbox.strategies中现有实例已废弃，需要使用`new`创建对应的实例，如：`new workbox.strategies.StaleWhileRevalidate()`、`new workbox.strategies.NetworkFirst()`、`new workbox.strategies.CacheFirst()`、`new workbox.strategies.NetworkOnly()`、`new workbox.strategies.CacheOnly()`
> workbox默认只针对GET请求，POST请求当作需要往服务端发送数据，自动放过
> 对于cdn中资源，google官网建议使用`Stale While Revalidate`策略，但是使用下来发现该策略没有缓存效果，因为最初本地根本没有其缓存，因此以后每次都会向服务端请求，不起任何作用
> corss-origin 跨域资源无法直接使用`Cache First`策略，使用后不起作用
> cdn图片，跨域资源可以在`Cache First`的基础上集成插件`cacheableResponse`使用，此时就可以进行`Cache First` 策略缓存
> ```typescript
    workbox.routing.registerRoute(new RegExp('^http://ec2-3-229-226-125.compute-1.amazonaws.com'), new workbox.strategies.CacheFirst ({
        plugins: [
            new workbox.cacheableResponse.Plugin({
                statuses: [0, 200, 404], // One or more status codes that a Response can have and be considered cacheable.
            }),
        ]
    }), 'GET')
  ```

## 效果
使用domContentLoaded做为时间参考点，对比有无service worker的情况：
以首页为例，在不同的网络环境下，发起10次网络请求，然后取平均值，作为它们的最终结果，测试结果如下：

网络环境 | 白屏时间（Service worker）/ms | 白屏时间（正常请求）/ms | 提升比例
-|-|-|-
wifi | 310.8 | 358.8 | 14%
4G | 311.2 | 875.2 | 181%
3G | 313.8 | 2320 | 641%
2G | 312.8 | 4752 | 1423%
离线 | 320 | - | 

通过上面的数据可以得出几个结论:
+ 在弱环境下，service worker的优势越发明显，
+ 即使在wifi环境下面，由于存在缓存的情况，浏览器加载的速度也比未使用service worker的时间要短。
+ 在无网络环境的情况，也可以做到离线缓存的效果，极大地提升页面的用户体验。

## 移动端兼容性测试
有些文档显示移动端webview中已经支持，有些文档显示不支持，经过测试，发现默认情况下不支持，是否需要相关配置这方面资料不足，部分移动端浏览器中已经支持，需要后期研究相关设置。
