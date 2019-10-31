---
title: Background Services
date: 2019-10-30 14:19:57
categories:
    - h5
    - background
    - service
tags:
    - h5
    - background services
    - service worker
---
##前言
  随着h5中service worker的出现，开发者已经可以实现一些数据缓存及简单的后台服务功能，但是早期的service worker在后台获取接口及延迟发送数据方面仍然不足，因此扩展了两个Background Services来实现这两个功能。
  - Background Fetch
    + 解决问题：
        - 当service worker在下载大文件到缓存中时，如果用户离开了，即使使用原来的`event.waitUntil`，这个service worker仍然可能会被杀死
        - 当service worker在上传问件时，如果用户离开则上传文件会中断
    + 作用：
        background Fetch允许开发者在当前上下文之外操作和控制文件列表，提高了上传和下载的成功率，同时可以让service worker缓存请求结果。
    + Example:
    ```typescript
        // 业务代码
        navigator.serviceWorker.register('/sw.js').then(function(reg) {
            const button = document.getElementById('download');
            if ('backgroundFetch' in reg) {
                button.addEventListener('click', function(event) {
                reg.backgroundFetch.fetch('large-file', [new Request('/images/twilio.png')], {
                    title: 'Downloading large image'
                }).then(function(backgroundFetch) { console.log(backgroundFetch) });
                })
            }
        });
    
        // service worker
        self.addEventListener('backgroundfetched', function(event) {
            event.waitUntil(caches.open('downloads').then(function(cache) {
                event.updateUI('Large file downloaded');
                registration.showNotification('File downloaded!');
                const promises = event.fetches.map(({ request, response }) => {
                    if (response && response.ok) {
                        return cache.put(request, response.clone());
                    }   
                });
                return Promise.all(promises);
            })
          );
        });
    ```
  - Background Sync
    + 解决问题：
       - 延迟发送请求，直到用户网络稳定
       - form 提交或者文件上传必须要等到提交完成，才能离开
    + 作用：
        可以在发送数据时使用调度，聊天，消息，邮件，文档更新，设置更改时，上传照片时，任何想要发送给服务器的数据都可以使用。即使页面关闭或用户跳转到其他页面，该页面也会将数据存到Indexed DB，并且Service Worker会检索到这些信息，并且发送。
    + Example：
    ```typescript
        navigator.serviceWorker.register('/sw.js');
        navigator.serviceWorker.ready.then(function(swRegistration) {
            return swRegistration.sync.register('myFirstSync');
        });
        self.addEventListener('sync', function(event) {
            if (event.tag == 'myFirstSync') {
                event.waitUntil(doSomeStuff());
            }
        });
    ```
