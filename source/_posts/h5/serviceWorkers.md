---
title: service worker
date: 2019-11-01 11:43:55
tags:
    - h5
    - optimize
    - PWA
    - cache
    - Service Worker
    - To supplement
categories:
    - h5
    - optimize
    - PWA
    - cache
top: true
---
## 前言
随着前端可处理业务的复杂，页面中资源也越来越庞大，优化逐渐成为一个重要的关注点，而缓存在所有优化中占据重要的地位，本文主要介绍web缓存机制中的Service Worker。

## 基本介绍
H5标准中对于前端资源缓存也提出了相应的方案，早起的[application cache](../applicationCache)原本是W3C做为前端缓存主推的方案，但是随着该方案缺陷越来越明显，目前该方案已经被废弃，而最新的替代方案为service worker。  
Service worker是基于浏览器底层重新创建了一个单独的线程，独立于Web页面，做为一个脚本在浏览器Background默默运行，提供了丰富的离线体验，定期后台同步以及推送通知等功能。
本文主要介绍Service Worker的常用功能，其新增功能可以访问[Service Worker 扩展](../backgroundServices)阅读。

## 特性
1. `独立线程`，不可以直接操作页面DOM，可以和页面进行通信，由页面去更新DOM
2. `网络代理`，做为一个可编程的网络代理，允许你控制当前网页网络请求如何发出，有过Android Webview开发经验的童鞋应该很容易理解，跟`shouldInterceptRequest`的功能类似
3. `闲时等待，用时重启`，在不使用时会停止运行，需要时重启，不会一直占用资源，没有一个全局的状态标志
4. `支持数据库`，支持IndexDB的访问
5. `Promise语法`，会用到大量的Promise语法，要求熟练掌握Promise

## 生命周期
 {% asset_img img-left lifecycle.png lifycycle %}

## 使用
1. 兼容性判断
```typescript
if ('serviceWorker' in navigator) {
    // TODO register
}
```
2. 注册：`register(scriptURL: string, options?: {scope?:string;type?:"classic" | "module";updateViaCache?:"imports" | "all" | "none"})`
```typescript
window.addEventListener('load', function() {
    navigator.serviceWorker.register('/sw.js').then(function(registration) {
      console.log('ServiceWorker registration successful with scope: ', registration.scope);
    }, function(err) {
      console.log('ServiceWorker registration failed: ', err);
    });
});
```
{% label danger@注意 %}：页面在首次打开的时候就调用`register`进行缓存sw的资源，因为sw内预缓存资源是需要下载的，sw线程一旦在首次打开时下载资源，将会占用主线程的带宽，以及加剧对cpu和内存的使用，而且sw启动之前，它必须先向浏览器 UI 线程申请分派一个线程，再回到 IO 线程继续执行sw线程的启动流程，并且在随后多次在ui线程和io线程之间切换，所以在启动过程中会存在一定的性能开销，在手机端尤其严重；这将导致首屏性能更差，因此正确的做法是在页面加载完成后进行注册，即监听`load`事件中注册。
通过register API进行注册，支持注册选项配置：  
- `scope`：配置该sw接管那个path下的请求，默认接管sw文件所在目录那一层，如果sw放在项目子文件夹中，需要通过`scope:"/"`，将其指定接管根目录下所有请求；  
- `updateViaCache`：大部分浏览器（包括 Chrome 68 和更高版本）在检查已注册的sw脚本的更新时，默认情况下都会忽略缓存标头。在通过 importScripts() 提取sw内加载的资源时，它们仍会遵循缓存标头。可以在注册sw时，通过设置`updateViaCache`选项来替换此默认行为；
- `type`：目前理解为标识是传统的sw还是模块化的sw，目前未发现其作用，可能是后续扩展属性，`等查到相关资料后补充`。 
3. 注销：`unregister()`
如果发生问题，怎么注销该sw并清除缓存，这都是要在注册时考虑的：
```typescript
 window.addEventListener('load', function() {
    const sw = window.navigator.serviceWorker;
    const killSW = window.killSW || false;// 该标识应该是sw出现异常被kill后浏览器中接收到的状态
    if (!sw) {
        return
    }
    if (!!killSW) {
        // 注销代码
        sw.getRegistration('/serviceWorker').then(registration => {
            // 手动注销
            registration.unregister();
            // 清除缓存
            window.caches && caches.keys && caches.keys().then(function(keys) {
                keys.forEach(function(key) {
                    caches.delete(key);
                });
            });
        })
    } else {
        sw.register('/serviceWorker.js',{scope: '/'}).then(registration => {
            console.log('Registered events at scope: ', registration.scope);
        }).catch(err => {
            console.error(err)
        })
    }
  });
```
3. 安装
```typescript
self.addEventListener('install', function(event) {
    event.waitUntil(caches.open(CACHE_NAME).then(function(cache) {
        return cache.addAll([
            '/',
            "index.html",
            "main.css",
        ]);
    }));
})
```
在安装中需要处理一些初始化，例如preCache，IndexDB 初始化等，preCache会直接下载并缓存指定文件，占用网络资源，因此不建议register直接在首屏加载时调用。
- `cacheStorage api`：阅读[cacheStorage](../cacheStorage)
- `waitUntil`：函数内所有的promise,只要有一个promise的结果是reject，那么这次安装就会失败。比如说cache.addAll 时，有一个资源下载不回来，即视为整个安装失败，那么后面的操作都不会执行，只能等待sw下一次重新注册。另外waitUntil还有一个重要的特性，那就是延长事件生命周期的时间，由于浏览器会随时睡眠 sw，所以为了防止执行中断就需要使用 event.waitUntil 进行捕获，当所有加载都成功时，那么 sw 就可以下一步。
- `skipWaiting`:安装成功后并不是立即被使用，如果当前页面已经存在sw进程，那么需要等待下一次页面被打开时新的sw才会被激活，或者使用`self.skipWaiting()`跳过等待，直接进入`activate`状态。
4. 激活
```typescript
self.addEventListener('activate', event => {
    event.waitUntil(caches.keys().then(cacheNames => {
        return cacheNames.filter(cacheName => CACHE_NAME !== cacheName);
    }).then(cachesToDelete => {
        return Promise.all(cachesToDelete.map(cacheToDelete => {
            return caches.delete(cacheToDelete);
        }));
    }).then(() => {
      // 立即接管所有页面
        self.clients.claim()
    }));
});
```
在activate中通常我们要检查并删除旧缓存，例如当sw发生变化或者`CACHE_NAME`发生变化则需要清除原有缓存，虽然说缓存区空间足够大，但是无用的缓存存在会影响命中效率
`clients.claim()`： 接管所有页面，默认情况下，页面的请求（fetch）不会通过 sw，除非它本身是通过 sw 获取的，也就是说，在安装 sw 之后，需要刷新页面才能有效果，clients.claim可以强制接管，在激活后接管所有客户端。`并非必须，因为仅第一次加载不经过sw，刷新后会经过sw，有时加上会出现一些问题，例如，加载和正常的请求不同资源时，也就是当作资源代理使用时会出现不可控问题，可能不经过sw直接从服务器请求`。
5. 请求拦截
```typescript
self.addEventListener('fetch', function(event) {
    event.respondWith(caches.match(event.request).then(function(resp) {
        const {body,cache,credentials,headers,integrity,keepalive,method,mode,redirect,referrer,referrerPolicy,signal,window} = event.request.clone();
        return resp || fetch(event.request,{
                body,cache,credentials,headers,integrity,keepalive,method,mode,redirect,referrer,referrerPolicy,signal,window
            }).then(function(response) {
            return caches.open(CACHE_NAME).then(function(cache) {
                cache.put(event.request, response.clone());
                return response;
            }); 
        });
    }));
});
```
通过监听`fetch`事件进行拦截请求，通常在内部需要判断哪些请求需要被处理，未使用event.respondWith处理的请求会直接采用原有方式发送到服务端。
- `event.respondWith`：代理原有请求，将结果返回给初始请求
- 先判断缓存中是否存在，如果命中直接返回缓存的response，否则再发送fetch请求，发送成功后先存入缓存，然后再返回给主进程
- 此处需要注意如果要发送的请求，需要cookie或者跨域，即原有请求设置了`credential`和`mode`参数，则sw中也fetch也需要设置，即需要将request中的对应参数进行复用
- 请求流和响应流只能被读取一次，因此response需要使用`response.clone`进行复制

## 请求策略
目前根据service worker衍生出的请求策略包括以下几种：
1. `networkFirst`：首先尝试通过网络来处理请求，如果成功就将响应存储在缓存中，否则返回缓存中的资源来回应请求。它适用于以下类型的API请求，即你总是希望返回的数据是最新的，但是如果无法获取最新数据，则返回一个可用的旧数据。具体实现如下：
```typescript
function networkFirst(request){
    return fetch(request).then(response=>{
        const clone = response.clone();
        caches.open(CACHE_NAME).then(cache=>{
            cache.put(request,clone);
        })    
    }).catch(()=>{
        return caches.op(CACHE_NAME).then(cache=>cache.match(request));
    })
}
self.addEventListener('fetch', function(event) {
    event.respondWith(networkFirst(event.request));
});
```
2. `cacheFirst`：如果缓存中存在与网络请求相匹配的资源，则返回相应资源，否则尝试从网络获取资源。 同时，如果网络请求成功则更新缓存。此选项适用于那些不常发生变化的资源，或者有其它更新机制的资源。具体实现如下：
```typescript
function cacheFirst(request){
    return caches.open(CACHE_NAME).then(cache=>cache.match(request)).then(response=>{
        return response||fetch(request).then(function(res) {
            return caches.open(CACHE_NAME).then(function(cache) {
                cache.put(request, res.clone());
                return response;
            })
        });
    })
}
self.addEventListener('fetch', function(event) {
    event.respondWith(cacheFirst(event.request));
});
```
3. `fastest`：从缓存和网络并行请求资源，并以首先返回的数据作为响应，通常这意味着缓存版本则优先响应。一方面，这个策略总会产生网络请求，即使资源已经被缓存了。另一方面，当网络请求完成时，现有缓存将被更新，从而使得下次读取的缓存将是最新的。具体实现如下：
```typescript
function fastest(request){
    const TIMEOUT = 500;
    let timer;
    return Promise.race([new Promise((resolve,reject)=>{
        timer = setTimeout(()=>{
            caches.open(CACHE_NAME).then(cache=>cache.match(request)).then(response=>{
                if(response){
                    resolve(response);
                }
            })
        },TIMEOUT)
    }),fetch(request).then(function(res) {
        clearTimeout(timer);
        caches.open(CACHE_NAME).then(function(cache) {
           cache.put(request, res.clone());
       });
        return res;
    })])
}
self.addEventListener('fetch', function(event) {
    event.respondWith(fastest(event.request));
});
```
4. `cacheOnly`：从缓存中解析请求，如果没有对应缓存则请求失败。此选项适用于需要保证不会发出网络请求的情况，例如在移动设备上节省电量。具体实现如下：
```typescript
function cacheOnly(request){
    return caches.open(CACHE_NAME).then(cache=>cache.match(request));
}
self.addEventListener('fetch', function(event) {
    event.respondWith(cacheOnly(event.request));
});
```
5. `networkOnly`：尝试从网络获取网址来处理请求。如果获取资源失败，则请求失败，这基本上与不使用service worker的效果相同。具体实现如下：
```typescript
function networkOnly(request){
    return fetch(request);
}
self.addEventListener('fetch', function(event) {
    event.respondWith(networkOnly(event.request));
});
```
6. 通常静态资源采用缓存优先的方式进行加载，api可以使用网络优先或者networkOnly，具体的优先策略根据项目需要进行调整。

## 更新
- service worker 的更新虽然没有application cache 那样繁琐，但是也并不简单，如果处理不好仍然会存在一些问题，正常情况下建议service worker不去做改动。  
- 默认情况下，service worker 浏览器`至少每隔24小时`去检查一次是否更新，刷新页面会立即去进行下载比较，当然不刷新页面也可以通过`update`方法进行更新操作：
```typescript
window.addEventListener('load', function() {
    navigator.serviceWorker.register('/sw.js').then(function(registration) {
        // some time later ...
        registration.update();
    }, function(err) {
      console.log('ServiceWorker registration failed: ', err);
    });
});
```
- 浏览器下载sw脚本后会与原有脚本进行`字节`对比，如果发现不同则会进行安装新的sw，安装后默认情况下新的sw处于等待状态，原有的仍然处于运行状态，直到原有sw管理的所有窗口全部关闭时旧的sw才会停止，新的sw才会在接下来重新打开的页面里生效。  
- 如果不想要新的sw进行等待，可以在`install`中调用`skipWaiting`去跳过等待状态。直接进入`activate`状态，并接管当前窗口，多窗口可以在`activate`中使用`clients.claim()`方法强制接管所有关联窗口。
- 除了强制接管外还有以下方案进行更新：
    - `controllerchange`：主程序中通过该事件监测sw是否更新，如果更新则刷新页面，例如：
    ```typescript
    navigator.serviceWorker.addEventListener('controllerchange',()=>{
      window.location.reload();
    })
    ```
    该方法在sw出现更新时会造成用户刚进入又自动刷新的现象，用户体验较差，不推荐使用。
    - 同上，参考[lavas](https://lavas.baidu.com/)库监测到更新后浏览器顶部弹出一个提示气泡，提示用户去刷新sw，例如：
    ```typescript
    try {
      navigator.serviceWorker.getRegistration().then(reg => {
        reg.waiting.postMessage('skipWaiting');
      });
    } catch (e) {
      window.location.reload();
    }
    // sw.js
    self.addEventListener('message', event => {
       if (event.data === 'skipWaiting') {
         self.skipWaiting();
       }
    })
    ```

## 兼容性
目前service worker的兼容各大浏览器是从16年左右开始，也就是之前版本的浏览器都不兼容，因此兼容性相对于application cache来说比较差，移动端`Android5+`，`IOS11.3+`才兼容，具体兼容性可以[查看](https://caniuse.com/#search=service%20worker)。

## 注意点
- sw.js 从项目开始到最终项目下线，不可以修改其文件名，如果修改文件名会出现无法更新的问题，因为通常页面是被缓存的，页面中注册的sw.js任然是以前的sw服务，甚至如果原有的sw文件不存在的情况下，注册会失败，任然使用原有服务。
- 不要给sw.js设置缓存，同样的问题，影响网站更新，sw.js设置了缓存后可能永远都不会更新sw.js 

## 工程化
如果在sw.js中不是通过泛类型的方式过滤资源的缓存的话，每次在项目打包时静态资源文件名会出现变化，其后面带的[hash]值会不一样，此时如果想要通过sw缓存固定的某个资源，需要根据编译的结果去进行配置，因此工程化建议使用泛类型或者webpack sw相关插件进行配置。

## 第三方工具
- ~~sw-precache~~：google 早期的轮子，与2006年停止维护，有对应的webpack插件
- ~~sw-toolbox~~：google 早期的轮子，与2006年停止维护，有对应的webpack插件
- [workbox](../workbox)：google最新推荐的工具，百度的[lavas](https://lavas.baidu.com/)底层使用的就是这个。
