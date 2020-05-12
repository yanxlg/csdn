---
title: pwa
tags:
  - h5
  - pwa
categories:
  - h5
abbrlink: f17b0fbd
date: 2019-01-29 14:58:59
---
## 前言
什么是`PWA`？`PWA`全称`Progressive Web App`，即渐进式WEB应用。是Google 在2016年提出的概念，2017年落地的web技术。目的就是在移动端利用提供的标准化框架，在网页应用中实现和原生应用相近的用户体验的渐进式网页应用。
>Reliable - Load instantly and never show the downasaur, even in uncertain network conditions.
>Fast - Respond quickly to user interactions with silky smooth animations and no janky scrolling.
>Engaging - Feel like a natural app on the device, with an immersive user experience.


## 简介
引用上述官方介绍，即可以做到：
1. 可靠
即时加载，即使在不确定的网络条件下也不会受到影响。当用户从主屏幕启动时，`service work`可以立即加载渐进式Web应用程序，完全不受网络环境的影响。`service work`就像一个客户端代理，它控制缓存以及如何响应资源请求逻辑，通过预缓存关键资源，可以消除对网络的依赖，确保为用户提供即时可靠的体验。
2. 快速
据统计，如果站点加载时间超过 3s，53% 的用户会放弃等待。页面展现之后，用户期望有平滑的体验，过渡动画和快速响应。
3. 沉浸式体验
感觉就像设备上的原生应用程序，具有沉浸式的用户体验。渐进式Web应用程序可以安装并在用户的主屏幕上，无需从应用程序商店下载安装。他们提供了一个沉浸式的全屏幕体验，甚至可以重新与用户接触的Web推送通知。

Web应用程序中，可以通过`manifest.json`控制应用程序的显示方式和启动方式，指定主屏幕图标、启动应用程序时要加载的页面、屏幕方向，甚至可以指定是否显示浏览器Chrome。
根据官方的介绍，不难看出，`PWA`的目标直指原生app，那接下来我们就来了解下`PWA`到底是个怎么样的何方神圣。

## 核心功能
`PWA`并不是指单一的技术，它是一种思想、一种概念，目的是对比原生App，通过一系列的技术去优化Web网站，提升其安全性、性能、流畅性、用户体验等方面指标，最后达到让用户像使用原生App的感觉。

### `PWA`包含的核心技术如下：
{% note %}
1、Web App Manifest
2、Service Worker
3、Cache API
4、Push&Notification
5、Background Sync
6、响应式设计
{% endnote %}

## `PWA`如何弥补与原生之间的差距
1. 性能差距
`PWA` 使用`App Shell`架构模型 
```text
1. 快速加载
2. 尽可能使用较少的数据
3. 使用本机缓存中的静态资产
4. 将内容与导航分离开来
5. 检索和显示特定页面的内容（HTML、JSON 等）
6. 缓存动态内容 App Shell 可保证 UI 的本地化以及从 API 动态加载内容，但同时不影响网络的可链接性和可检测性。 用户下次访问您的应用时，应用会自动显示最新版本。无需在使用前下载新版本。
为了保证首屏的加载，在内容请求完成之前，可以优先保证 App Shell 的渲染，做到和 Native App 一样的体验，App Shell 是 PWA 界面展现所需的最小资源。
```

2. 离线使用
```text
Service Worker + HTTPS +Cache Api + indexedDB 等一系列web技术实现离线加载和缓存
```

3. 数据更新
```text
Background Sync后台同步技术
```

4. 推送功能
```text
Push & Notification API 实现推送通知
```

5. 独立启动入口
```text
Manifest 使得可以直接添加到手机的桌面上。
```

## 优势
1. 无需安装，无需下载，只要你输入网址访问一次，然后将其添加到设备桌面就可以持续使用。
2. 发布不需要提交到app商店审核
3. 更新迭代版本不需要审核，不需要重新发布审核
4. 现有的web网页都能通过改进成为PWA， 能很快的转型，上线，实现业务、获取流量
5. 不需要开发Android和IOS两套不同的版本

## 劣势
1. 游览器对技术支持还不够全面， 不是每一款游览器都能100%的支持所有PWA
2. 需要通过第三方库才能调用底层硬件（如摄像头）
3. PWA现在还没那么火，国内一些手机生产上在Android系统上做了手脚，似乎屏蔽了PWA, 但是相信当PWA火起来以后，这个问题就不会是问题

## 详细介绍
1. Manifest
[参考文档](https://developer.mozilla.org/zh-CN/docs/Web/Manifest)
使用方式：
```html
<link rel="manifest" href="manifest.json">
```
```json5
{
  "name": "一个PWA示例", // 必填 显示的插件名称
  "short_name": "PWA Demo", // 可选  在APP launcher和新的tab页显示，如果没有设置，则使用name
  "description": "The app that helps you understand PWA", //用于描述应用
  "display": "standalone", // 定义开发人员对Web应用程序的首选显示模式。standalone模式会有单独的
  "start_url": "/", // 应用启动时的url
  "theme_color": "#313131", // 桌面图标的背景色
  "background_color": "#313131", // 为web应用程序预定义的背景颜色。在启动web应用程序和加载应用程序的内容之间创建了一个平滑的过渡。
  "icons": [ // 桌面图标，是一个数组
    {
      "src": "/public/img/logo.png",
      "sizes": "144x144",
      "type": "image/png"
    }]
}
```
Manifest在m站中使用较多，或者PC端，在App内置h5中一般不会使用。

2. service worker
见文章{% post_link h5/serviceWorker serviceWorker%}

3. Push & Notification
不同浏览器需要用不同的推送消息服务器。以 Chrome 上使用 Google Cloud Messaging<GCM> 作为推送服务为例，第一步是注册 applicationServerKey(通过 GCM 注册获取)，并在页面上进行订阅或发起订阅。每一个会话会有一个独立的端点（endpoint），订阅对象的属性(PushSubscription.endpoint) 即为端点值。将端点发送给服务器后，服务器用这一值来发送消息给会话的激活的 Service Worker （通过 GCM 与浏览器客户端沟通）。
service worker中捕获用户点击消息：
```typescript
 self.addEventListener('notificationclick', function (event) {
    event.notification.close(); //关闭通知
    event.waitUntil(
        // 获取所有clients
        self.clients.matchAll().then(function (clients) {
            if (!clients || clients.length === 0) {
                return;
            }
            clients.forEach(function (client) {
                // 使用postMessage进行通信
                client.postMessage(action);
                console.log("post a message");
            });
        })
    );
});
```

4. App Shell 模型
App Shell 架构是构建 PWA 应用的一种方式，它通常提供了一个最基本的 Web App 框架，包括应用的头部、底部、菜单栏等结构。顾名思义，我们可以把它理解成应用的一个「空壳」，这个「空壳」仅包含页面框架所需的最基本的 HTML 片段，CSS 和 javaScript，这样一来，用户重复打开应用时就能迅速地看到 Web App 的基本界面，只需要从网络中请求、加载必要的内容。我们使用 Service Worker 对 App Shell 做离线缓存，以便它可以在离线时正常展现，达到类似 Native App 的体验。
